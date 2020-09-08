---
title: Copia de seguridad y restauración de Service Fabric
description: Documentación conceptual para la copia de seguridad y la restauración de Service Fabric, un servicio para configurar la copia de seguridad de servicios con estado de confianza y Reliable Actors.
author: mcoskun
ms.topic: conceptual
ms.date: 10/29/2018
ms.author: mcoskun
ms.custom: devx-track-csharp
ms.openlocfilehash: a60ebff06562c12415b2a106a9a11127feb94dab
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021993"
---
# <a name="backup-and-restore-reliable-services-and-reliable-actors"></a>Copia de seguridad y restauración de Reliable Services y Reliable Actors
Azure Service Fabric es una plataforma de alta disponibilidad que replica el estado entre varios nodos para mantener esta disponibilidad alta.  Por lo tanto, incluso si se produce un error en un nodo del clúster, los servicios siguen estando disponibles. Aunque esta redundancia integrada proporcionada por la plataforma puede ser suficiente para algunos casos, en otros es conveniente que el servicio haga una copia de seguridad de los datos (en un almacén externo).

> [!NOTE]
> Es fundamental hacer una copia de seguridad de los datos y restaurarlos (y comprobar que funcionan según lo esperado) para que se pueda recuperar de escenarios de pérdida de datos.
> 

> [!NOTE]
> Microsoft recomienda usar [copias de seguridad y restauraciones periódicas](service-fabric-backuprestoreservice-quickstart-azurecluster.md) para configurar la copia de seguridad de datos de servicios de confianza con estado y de Reliable Actors. 
> 


Por ejemplo, es posible que un servicio quiera realizar copias de seguridad de los datos como medida de protección en los escenarios siguientes:

- En caso de que se produzca una pérdida permanente de todo un clúster de Service Fabric.
- En caso de que se produzca una pérdida permanente de la mayoría de las réplicas de una partición del servicio.
- Errores administrativos por los cuales el estado se elimina o daña accidentalmente. Por ejemplo, esto puede ocurrir si un administrador con suficientes privilegios elimina el servicio por error.
- Errores en el servicio que causan daños en los datos. Por ejemplo, esto puede suceder cuando una actualización de código de servicio empieza a escribir datos defectuosos en una colección confiable. En tal caso, es posible que el código y los datos se tengan que revertir a un estado anterior.
- Procesamiento de datos sin conexión. Podría ser conveniente procesar los datos para inteligencia empresarial sin conexión de forma independiente del servicio que los genera.

La característica de copia de seguridad y restauración permite que los servicios basados en la API de Reliable Services creen y restauren copias de seguridad. Las API de copia de seguridad proporcionadas por la plataforma permiten hacer copias de seguridad del estado de una partición del servicio sin bloquear las operaciones de lectura o escritura. Las API de restauración permiten restaurar el estado de una partición de servicio a partir de una copia de seguridad elegida.

## <a name="types-of-backup"></a>Tipos de copia de seguridad
Hay dos opciones de copia de seguridad: completa e incremental.
Una copia de seguridad completa es aquella que contiene todos los datos necesarios para volver a crear el estado de la réplica: puntos de control y todos los registros.
Puesto que tiene los puntos de control y el registro, las copias de seguridad completas pueden restaurarse solas.

El problema con las copias de seguridad completas se produce cuando los puntos de control tienen un gran tamaño.
Por ejemplo, una réplica que tiene 16 GB de estado tendrá puntos de control que agreguen hasta aproximadamente 16 GB.
Si tenemos un objetivo de punto de recuperación de cinco minutos, debe realizarse una copia de seguridad de la réplica cada cinco minutos.
Cada vez que se haga una copia de seguridad, habrá que copiar 16 GB de puntos de control, además de 50 MB de registros (esto se puede configurar con `CheckpointThresholdInMB`).

![Ejemplo de copia de seguridad completa.](media/service-fabric-reliable-services-backup-restore/FullBackupExample.PNG)

La solución a este problema son las copias de seguridad incrementales, donde solo se incluyen los registros que han cambiado desde la última copia de seguridad.

![Ejemplo de copia de seguridad incremental.](media/service-fabric-reliable-services-backup-restore/IncrementalBackupExample.PNG)

Puesto que las copias de seguridad incrementales solo incluyen los cambios realizados desde la última copia de seguridad (no se incluyen los puntos de control), suelen ser más rápidas, pero no se pueden restaurar automáticamente.
Para restaurar una copia de seguridad incremental, se necesita toda la cadena de copia de seguridad.
Una cadena de copia de seguridad empieza por una copia de seguridad completa a la que sigue un número contiguo de copias de seguridad incrementales.

## <a name="backup-reliable-services"></a>Copias de seguridad de Reliable Services
El autor del servicio tiene control total sobre cuándo se realizan las copias de seguridad y dónde se almacenan.

Para iniciar una copia de seguridad, el servicio debe invocar la función de miembro heredada `BackupAsync`.  
Las copias de seguridad solo se pueden realizar desde réplicas principales y necesitan que se conceda el estado de escritura.

Tal y como se muestra a continuación, `BackupAsync` toma un objeto `BackupDescription`, donde se puede especificar una copia de seguridad completa o incremental, así como una función de devolución de llamada, `Func<< BackupInfo, CancellationToken, Task<bool>>>`, que se invoca cuando la carpeta de copia de seguridad se ha creado localmente y está lista para transferirse a un almacenamiento externo.

```csharp

BackupDescription myBackupDescription = new BackupDescription(BackupOption.Incremental,this.BackupCallbackAsync);

await this.BackupAsync(myBackupDescription);

```

Si la copia de seguridad solicitada es incremental, puede producirse la excepción `FabricMissingFullBackupException`. Esta excepción indica que se ha producido uno de los problemas siguientes:

- La réplica nunca ha realizado una copia de seguridad completa desde que se convirtió en la primaria.
- Algunas de las entradas del registro se han truncado desde la última copia de seguridad.
- La réplica ha superado el límite de `MaxAccumulatedBackupLogSizeInMB`.

Para que la probabilidad de realizar correctamente copias de seguridad incrementales sea mayor, los usuarios pueden configurar `MinLogSizeInMB` o `TruncationThresholdFactor`.
Si se aumentan estos valores, lo hará también el uso del disco por parte de cada réplica.
Para obtener más información, consulte el artículo de [configuración de Reliable Services](service-fabric-reliable-services-configuration.md).

`BackupInfo` proporciona información sobre la copia de seguridad, como la ubicación de la carpeta donde el entorno de tiempo de ejecución guardó la copia de seguridad (`BackupInfo.Directory`). La función de devolución de llamada puede transferir `BackupInfo.Directory` a un almacén externo u otra ubicación.  Además, esta función devuelve un valor booleano que indica si se pudo mover correctamente la carpeta de copia de seguridad a su ubicación de destino.

En el fragmento de código siguiente, se muestra cómo puede utilizarse el método `BackupCallbackAsync` para cargar la copia de seguridad en Azure Storage:

```csharp
private async Task<bool> BackupCallbackAsync(BackupInfo backupInfo, CancellationToken cancellationToken)
{
    var backupId = Guid.NewGuid();

    await externalBackupStore.UploadBackupFolderAsync(backupInfo.Directory, backupId, cancellationToken);

    return true;
}
```

En el ejemplo anterior, `ExternalBackupStore` es la clase de muestra que se usa para interactuar con la instancia de Azure Blob Storage, mientras que `UploadBackupFolderAsync` es el método que comprime la carpeta y la coloca en el almacén de Azure Blob Storage.

Observe lo siguiente:

  - Solo puede haber una operación de copia de seguridad en proceso por réplica en cualquier momento dado. Si se realizan varias llamadas de `BackupAsync` a la vez, se iniciará una excepción `FabricBackupInProgressException` que restringirá a una las copias de seguridad en proceso.
  - Si una réplica conmuta por error mientras se realiza una copia de seguridad, es posible que esta no se complete. Por lo tanto, una vez que finalice la conmutación por error, es responsabilidad del servicio reiniciar la copia de seguridad mediante `BackupAsync`, en caso de que sea necesario.

## <a name="restore-reliable-services"></a>Restauración de Reliable Services
En general, los casos en los que necesite realizar una operación de restauración se dividen en estas categorías:

  - La partición del servicio perdió datos. Por ejemplo, el disco de dos de las tres réplicas de una partición (incluida la réplica principal) sufre daños o se borra. Es posible que la nueva réplica principal necesite restaurar datos desde una copia de seguridad.
  - Se perdió la totalidad del servicio. Por ejemplo, un administrador quita todo el servicio y, por tanto, se deben restaurar tanto el servicio como los datos.
  - El servicio replicó datos dañados de la aplicación (por ejemplo, debido a un error de la aplicación). En este caso, el servicio se debe actualizar o revertir para quitar la causa de los daños y se deben restaurar los datos no dañados.

Aunque hay muchos enfoques posibles, ofrecemos algunos ejemplos acerca de cómo se puede utilizar `RestoreAsync` para recuperarse de los escenarios anteriores.

## <a name="partition-data-loss-in-reliable-services"></a>Pérdida de datos de la partición en Reliable Services
En este caso, el entorno de tiempo de ejecución detectaría automáticamente la pérdida de datos e invocaría la API `OnDataLossAsync`.

El autor del servicio debe realizar lo siguiente para recuperar:

  - Invalidar el método de clase base virtual `OnDataLossAsync`.
  - Busque la última copia de seguridad en la ubicación externa que contiene las copias de seguridad del servicio.
  - Descargue la última copia de seguridad (y descomprímala en la carpeta de copia de seguridad si estaba comprimida).
  - El método `OnDataLossAsync` proporciona un objeto `RestoreContext`. Debe llamar a la API `RestoreAsync` del objeto `RestoreContext` proporcionado.
  - Devuelve true si la restauración fue correcta.

A continuación, se muestra un ejemplo de la implementación del método `OnDataLossAsync`:

```csharp
protected override async Task<bool> OnDataLossAsync(RestoreContext restoreCtx, CancellationToken cancellationToken)
{
    var backupFolder = await this.externalBackupStore.DownloadLastBackupAsync(cancellationToken);

    var restoreDescription = new RestoreDescription(backupFolder);

    await restoreCtx.RestoreAsync(restoreDescription);

    return true;
}
```

El objeto `RestoreDescription` que se pasa en la llamada a `RestoreContext.RestoreAsync` contiene un miembro denominado `BackupFolderPath`.
Si se restaura una única copia de seguridad completa, este miembro `BackupFolderPath` debe establecerse en la ruta de acceso local de la carpeta que contiene la copia de seguridad completa.
Si se restaura una copia de seguridad completa y varias copias de seguridad incrementales, `BackupFolderPath` debe establecerse en la ruta de acceso local de la carpeta que contenga tanto la copia de seguridad completa como todas las copias de seguridad incrementales.
La llamada a `RestoreAsync` puede iniciar una excepción `FabricMissingFullBackupException` si el miembro `BackupFolderPath` proporcionado no contiene una copia de seguridad completa.
También puede iniciar una excepción `ArgumentException` si `BackupFolderPath` tiene una cadena de copias de seguridad incrementales rota.
Por ejemplo, si contiene la copia de seguridad completa, la primera y la tercera copia de seguridad incremental, pero no la segunda.

> [!NOTE]
> RestorePolicy se establece en Seguro de forma predeterminada.  Esto significa que la API `RestoreAsync` iniciará una excepción ArgumentException si detecta que la carpeta de copia de seguridad contiene un estado igual o más antiguo que el estado contenido en esta réplica.  Esta comprobación de seguridad se puede omitir con `RestorePolicy.Force`. Este objeto se especifica como parte de `RestoreDescription`.
> 

## <a name="deleted-or-lost-service"></a>Eliminación o pérdida del servicio
Si se quita un servicio, en primer lugar debe volver a crearlo para poder restaurar los datos.  Es importante crear el servicio con la misma configuración, por ejemplo, con el mismo esquema de partición, para que los datos se restauren sin ningún problema.  Una vez que el servicio está activo, la API `OnDataLossAsync` para recuperar los datos que se menciona más arriba debe invocarse en todas las particiones de este servicio. Una manera de conseguirlo es mediante el uso de [FabricClient.TestManagementClient.StartPartitionDataLossAsync](/dotnet/api/system.fabric.fabricclient.testmanagementclient?view=azure-dotnet#System_Fabric_FabricClient_TestManagementClient_StartPartitionDataLossAsync_System_Guid_System_Fabric_PartitionSelector_System_Fabric_DataLossMode_) en cada partición.  

Desde este punto, la implementación es igual que en el escenario anterior. Cada partición debe restaurar la última copia de seguridad pertinente desde el almacén externo. Una advertencia es que es posible que el identificador de partición sea diferente, ya que el tiempo de ejecución crea los identificadores de partición de forma dinámica. Por lo tanto, el servicio necesita almacenar el nombre de servicio y la información de partición adecuada para identificar la copia de seguridad correcta más reciente desde la que restaurar para cada partición.

> [!NOTE]
> No se recomienda usar `FabricClient.ServiceManager.InvokeDataLossAsync` en cada partición para restaurar todo el servicio, ya que puede dañar el estado del clúster.
> 

## <a name="replication-of-corrupt-application-data"></a>Replicación de datos de aplicación dañados
Si hay un error en la actualización de la aplicación recién implementada, puede que produzca daños en los datos. Por ejemplo, una actualización de la aplicación puede empezar a actualizar todos los registros de número de teléfono en un diccionario confiable con un código de área no válido.  En este caso, los números de teléfono no válidos se replicarán porque Service Fabric no conoce la naturaleza de los datos que se almacenan.

Lo primero que debe hacer después de detectar un error tan notorio que cause daños en los datos es inmovilizar el servicio en el nivel de aplicación y, si es posible, actualizar a la versión del código de aplicación que no contenga el error.  Sin embargo, incluso una vez corregido el código del servicio, es posible que los datos sigan dañados y que por tanto se deban restaurar.  En tales casos, puede que no baste con restaurar la última copia de seguridad, ya que las copias de seguridad más recientes también pueden estar dañadas.  Por lo tanto, tiene que buscar la última copia de seguridad realizada antes de que los datos resultaran dañados.

Si no está seguro de qué copias de seguridad están dañadas, podría implementar un nuevo clúster de Service Fabric y restaurar las copias de seguridad de las particiones afectadas de la misma forma que en el escenario de "Eliminación o pérdida del servicio" anterior.  Para cada partición, inicie la restauración de las copias de seguridad desde la más a la menos reciente. Una vez que encuentre una copia de seguridad sin los daños, mueva o elimine todas las copias de seguridad de esta partición que sean más recientes (que esa copia de seguridad). Repita este proceso para cada partición. Ahora, cuando se llame a `OnDataLossAsync` en la partición del clúster de producción, la última copia de seguridad que se encuentre en el almacén externo será la elegida por el proceso anterior.

Ahora se puede aplicar el procedimiento de la sección "Eliminación o pérdida del servicio" para restaurar el servicio en el estado que tenía antes de que el código con errores produjera los daños.

Observe lo siguiente:

  - Con cada restauración, es posible que la copia de seguridad que se restaura sea anterior al estado de la partición antes de la pérdida de datos. Por este motivo, la restauración se debe utilizar solo como último recurso para recuperar tantos datos como sea posible.
  - La cadena que representa la ruta de acceso de la carpeta de copia de seguridad, así como las rutas de acceso de los archivos dentro de la misma, puede tener más de 255 caracteres, según la ruta de acceso de FabricDataRoot y la longitud del nombre del tipo de aplicación. Esto puede hacer que algunos métodos .NET, como `Directory.Move`, inicien la excepción `PathTooLongException`. Como solución alternativa, puede llamarse directamente a las API de kernel32, como `CopyFile`.

## <a name="back-up-and-restore-reliable-actors"></a>Copia de seguridad y restauración de Reliable Actors


El marco de Reliable Actors está basado en Reliable Services. El servicio ActorService que hospeda los actores es un servicio con estado confiable. Por lo tanto, toda la funcionalidad de copia de seguridad y restauración disponible en Reliable Services también está disponible para Reliable Actors (excepto los comportamientos específicos para el proveedor de estado). Puesto que las copias de seguridad se realizan por partición, se llevará a cabo una copia de seguridad de los estados para todos los actores de esa partición. La restauración es similar y se realizará por partición. Para realizar la copia de seguridad y restauración, el propietario del servicio debe crear una clase de servicio de actor personalizada que derive de la clase ActorService y, luego, hacer la copia de seguridad y restauración de forma similar a Reliable Services, tal como se describió en las secciones anteriores.

```csharp
class MyCustomActorService : ActorService
{
    public MyCustomActorService(StatefulServiceContext context, ActorTypeInformation actorTypeInfo)
          : base(context, actorTypeInfo)
    {
    }
    
    //
    // Method overrides and other code.
    //
}
```

Cuando crea una clase de servicio de actor personalizada, también necesita registrar dicha clase cuando registra el actor.

```csharp
ActorRuntime.RegisterActorAsync<MyActor>(
    (context, typeInfo) => new MyCustomActorService(context, typeInfo)).GetAwaiter().GetResult();
```

El proveedor de estado predeterminado de Reliable Actors es `KvsActorStateProvider`. La copia de seguridad incremental no está habilitada de manera predeterminada para `KvsActorStateProvider`. Para habilitar la copia de seguridad incremental, puede crear `KvsActorStateProvider` con la configuración adecuada en el constructor y pasarla después al constructor de ActorService, tal y como se muestra en el fragmento de código siguiente:

```csharp
class MyCustomActorService : ActorService
{
    public MyCustomActorService(StatefulServiceContext context, ActorTypeInformation actorTypeInfo)
          : base(context, actorTypeInfo, null, null, new KvsActorStateProvider(true)) // Enable incremental backup
    {
    }
    
    //
    // Method overrides and other code.
    //
}
```

Una vez habilitada la copia de seguridad incremental, crear una copia de seguridad incremental puede presentar un error con la excepción FabricMissingFullBackupException por alguno de los siguientes motivos; por ello, deberá crear una copia de seguridad completa antes de crear copias de seguridad incrementales:

  - La réplica nunca ha realizado una copia de seguridad completa desde que se convirtió en la primaria.
  - Algunos de los registros se truncaron desde que se creó la última copia de seguridad.

Cuando la copia de seguridad incremental está habilitada, `KvsActorStateProvider` no usa un búfer circular para administrar los registros y los trunca de forma periódica. Si el usuario no crea copias de seguridad durante un período de 45 minutos, el sistema trunca automáticamente los registros. Puede configurar este intervalo especificando `logTruncationIntervalInMinutes` en el constructor `KvsActorStateProvider` (de forma similar a cuando se habilita la copia de seguridad incremental). También es posible que se trunquen los registros si la réplica principal debe crear otra réplica enviando todos sus datos.

Cuando se realiza la restauración a partir de una cadena de copias de seguridad, de manera similar a Reliable Services, BackupFolderPath debe contener subdirectorios con un subdirectorio que contenga la copia de seguridad completa y otros subdirectorios que contengan las copias de seguridad incrementales. La API de restauración iniciará la excepción FabricException con el mensaje de error correspondiente si se produce un error en la validación de la cadena de copias de seguridad. 

> [!NOTE]
> En la actualidad, `KvsActorStateProvider` omite la opción RestorePolicy.Safe. La compatibilidad con esta característica está pensada para un próximo lanzamiento.
> 

## <a name="testing-back-up-and-restore"></a>Prueba de copias de seguridad y restauraciones
Es importante asegurarse de que se hace una copia de seguridad de los datos y que se pueden restaurar desde dicha copia. Esto se puede hacer invocando el cmdlet `Start-ServiceFabricPartitionDataLoss` de PowerShell, que puede provocar la pérdida de datos en una partición determinada para comprobar si la funcionalidad de copia de seguridad y restauración de los datos del servicio funciona según lo esperado.  También es posible invocar la pérdida de datos mediante programación y restaurar a partir de ese evento también.

> [!NOTE]
> Puede encontrar una implementación de ejemplo de la funcionalidad de copia de seguridad y restauración en la aplicación de referencia web en GitHub. Consulte el servicio `Inventory.Service` para obtener más información.
> 
> 

## <a name="under-the-hood-more-details-on-backup-and-restore"></a>Bajo el capó: Más detalles sobre la copia de seguridad y la restauración
A continuación se proporcionan más detalles sobre la copia de seguridad y la restauración.

### <a name="backup"></a>Copia de seguridad
El Administrador de estado fiable proporciona la capacidad de crear copias de seguridad coherentes sin bloquear ninguna operación de lectura ni escritura. Para ello, usa un mecanismo de persistencia de registro y punto de control.  El Administrador de estado fiable instaura puntos de control aproximados (ligeros) en determinados puntos para aliviar la presión sobre el registro transaccional y mejorar los tiempos de recuperación.  Cuando se llama a `BackupAsync`, Reliable State Manager indica a todos los objetos Reliable que copien los archivos de punto de control más recientes en una carpeta local de copia de seguridad.  Después, el Administrador de estado confiable copia todas las entradas del registro desde el "puntero de inicio" hasta la entrada del registro más reciente en la carpeta de copia de seguridad.  Como todas las entradas del registro, de la primera a la última, se incluyen en la copia de seguridad y Reliable State Manager conserva el registro de escritura previa, Reliable State Manager garantiza que todas las transacciones confirmadas (aquellas en las que se devuelva `CommitAsync` correctamente) estarán incluidas en la copia de seguridad.

Cualquier transacción que se confirme después de la llamada a `BackupAsync` puede estar o no incluida en la copia de seguridad.  Una vez que la plataforma rellene la carpeta de copia de seguridad (es decir, el tiempo de ejecución completó la copia de seguridad local), se invoca la devolución de llamada de la copia de seguridad del servicio.  Esta devolución de llamada se encarga de mover la carpeta de copia de seguridad a una ubicación externa, como Azure Storage.

### <a name="restore"></a>Restauración
Reliable State Manager ofrece la posibilidad de restaurar los datos de una copia de seguridad mediante la API `RestoreAsync`.  
El método `RestoreAsync` de `RestoreContext` solo puede invocarse desde el método `OnDataLossAsync`.
El valor booleano devuelto por `OnDataLossAsync` indica si el servicio restauró su estado a partir de un origen externo.
Si el método `OnDataLossAsync` devuelve True, Service Fabric volverá a generar todas las demás réplicas a partir de esta principal. Service Fabric garantiza que las réplicas que reciben la llamada de `OnDataLossAsync` son las primeras en realizar la transición al rol principal y se asegura de que no se les concede el estado de lectura o escritura.
Esto supone que, en el caso de los implementadores de StatefulService, no se va a llamar a `RunAsync` hasta que `OnDataLossAsync` se complete correctamente.
Después, se invocará `OnDataLossAsync` en la nueva réplica principal.
Hasta que un servicio complete esta API correctamente (y devuelva true o false) y finalice la reconfiguración pertinente, se seguirá llamando a la API de uno en uno.

En primer lugar, `RestoreAsync` quita todos los estados existentes en la réplica principal donde se han realizado las llamadas. Después, el Administrador de estado confiable crea todos los objetos confiables que existen en la carpeta de copia de seguridad. A continuación, se indica a los objetos confiables que restauren a partir de sus puntos de control en la carpeta de copia de seguridad. Por último, el Administrador de estado confiable recupera su propio estado de las entradas del registro en la carpeta de copia de seguridad y realiza la recuperación. Como parte del proceso de recuperación, se reproducen en los objetos confiables las operaciones a partir del "punto de partida" que tengan entradas de registro de confirmación en la carpeta de copia de seguridad. Este paso garantiza que el estado recuperado sea coherente.

## <a name="next-steps"></a>Pasos siguientes
  - [Colecciones confiables](service-fabric-work-with-reliable-collections.md)
  - [Guía de inicio rápido de Reliable Services](service-fabric-reliable-services-quick-start.md)
  - [Notificaciones de Reliable Services](service-fabric-reliable-services-notifications.md)
  - [Configuración de Reliable Services](service-fabric-reliable-services-configuration.md)
  - [Referencia para desarrolladores de colecciones confiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
  - [Restauración y copia de seguridad periódicas de Azure Service Fabric](service-fabric-backuprestoreservice-quickstart-azurecluster.md)
