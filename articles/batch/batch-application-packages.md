---
title: Implementación de paquetes de aplicación en nodos de proceso
description: Utilice la característica paquetes de aplicación de Azure Batch para administrar fácilmente varias aplicaciones y versiones para la instalación en nodos de proceso de Batch.
ms.topic: how-to
ms.date: 04/26/2019
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: d847dd926d157e455a6ef4e0c58c9dd204a1ecc7
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89146477"
---
# <a name="deploy-applications-to-compute-nodes-with-batch-application-packages"></a>Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch

La característica de paquetes de aplicación de Azure Batch permite administrar e implementar fácilmente aplicaciones de tareas en los nodos de proceso de un grupo. Con paquetes de aplicación, puede cargar y administrar fácilmente varias versiones de las aplicaciones que las tareas ejecutan, incluidos los archivos auxiliares. A continuación, se pueden implementar automáticamente una o varias de estas aplicaciones en los nodos de proceso del grupo.

En este artículo, aprenderá cómo cargar y administrar paquetes de aplicación en Azure Portal. A continuación, aprenderá a instalarlos en nodos de proceso de un grupo mediante la biblioteca de [.NET para Batch][api_net].

> [!NOTE]
> Los paquetes de aplicaciones se admiten en todos los grupos de Batch creados después del 5 de julio de 2017. Se admiten en los grupos de Batch creados entre el 10 de marzo de 2016 y el 5 de julio de 2017 únicamente si el grupo se creó mediante una configuración de Cloud Services. Los grupos de Batch creados antes del 10 de marzo de 2016 no admiten los paquetes de aplicaciones.
>
> Las API para crear y administrar paquetes de aplicaciones forman parte de la biblioteca [Batch Management .NET][api_net_mgmt]. Las API para la instalación de paquetes de aplicaciones en un nodo de proceso forman parte de la biblioteca de [Batch para .NET][api_net]. Las características comparables están en las API de Batch disponibles para otros idiomas. 
>
> La característica de paquetes de aplicaciones que se describe aquí reemplaza a la característica de aplicaciones de Batch disponible en versiones anteriores del servicio.

## <a name="application-package-requirements"></a>Requisitos de los paquetes de aplicación
Para utilizar paquetes de aplicación, primero se debe [vincular una cuenta de Azure Storage](#link-a-storage-account) a su cuenta de Batch.

## <a name="about-applications-and-application-packages"></a>Acerca de las aplicaciones y los paquetes de aplicación
En Azure Batch, una *aplicación* hace referencia a un conjunto de archivos binarios con versiones que se pueden descargar automáticamente en los nodos de proceso del grupo. Un *paquete de aplicación* hace referencia a un *conjunto específico* de dichos archivos binarios y representa una *versión* determinada de la aplicación.

![Diagrama de alto nivel de aplicaciones y paquetes de aplicación][1]

### <a name="applications"></a>APLICACIONES
Una aplicación en Batch contiene uno o más paquetes de aplicación y especifica las opciones de configuración de la aplicación. Por ejemplo, una aplicación puede especificar la versión predeterminada del paquete de aplicación que se instala en los nodos de proceso y si sus paquetes se pueden actualizar o eliminar.

### <a name="application-packages"></a>paquetes de aplicación
Un paquete de aplicación es un archivo .zip que contiene los archivos binarios de la aplicación y los archivos auxiliares que se requieren para que las tareas ejecuten la aplicación. Cada paquete de aplicación representa una versión específica de la aplicación.

Los paquetes de aplicaciones se pueden especificar a niveles de grupo y de tarea. Puede especificar uno o varios de estos paquetes y (opcionalmente) una versión cuando crea un grupo o tarea.

* **paquetes de aplicación del grupo** se implementan en *cada* nodo del grupo. Las aplicaciones se implementan cuando un nodo se une a un grupo y cuando se reinicia o se restablece la imagen inicial.
  
    Los paquetes de aplicación de grupo son adecuados cuando todos los nodos de un grupo ejecutan las tareas de un trabajo. Puede especificar uno o más paquetes de aplicación cuando se crea un grupo y puede agregar o actualizar los paquetes de un grupo ya existente. Si actualiza los paquetes de aplicaciones de un grupo existente, debe reiniciar los nodos para instalar el nuevo paquete.
* **paquetes de aplicación de tareas** solo se implementan en un nodo de proceso programado para ejecutar una tarea, justo antes de ejecutar la línea de comandos de la tarea. Si el paquete de aplicación y la versión especificados ya están en el nodo, este no se volverá a implementar y se utilizará el paquete existente.
  
    Los paquetes de aplicaciones de tareas son útiles en entornos de grupo compartido, donde los distintos trabajos se ejecutan en un grupo que no se elimina cuando se completa un trabajo. Si el trabajo tiene menos tareas que nodos en el grupo, los paquetes de aplicación de las tareas pueden minimizar la transferencia de datos, ya que la aplicación se implementa solo en los nodos que ejecutan tareas.
  
    Otros escenarios que pueden beneficiarse de los paquetes de aplicación de tareas son los trabajos que ejecutan una aplicación grande, pero solo para unas pocas tareas. Por ejemplo, en una fase previa al procesamiento o una tarea de combinación, donde la aplicación de procesamiento previo o combinación es pesada, puede ser útil usar paquetes de aplicación de tareas.

> [!IMPORTANT]
> Hay restricciones en el número de aplicaciones y paquetes de aplicación que puede haber en una cuenta de Batch, así como en el tamaño máximo del paquete de aplicación. Para más información sobre estos límites, consulte [Cuotas y límites del servicio Azure Batch](batch-quota-limit.md) .
> 
> 

### <a name="benefits-of-application-packages"></a>Ventajas de los paquetes de aplicación
Los paquetes de aplicación pueden simplificar el código de su solución de Batch y reducir la sobrecarga requerida para administrar las aplicaciones que ejecutan las tareas.

Con los paquetes de aplicación, la tarea de inicio del grupo no tiene que especificar una larga lista de archivos de recursos individuales que se deben instalar en los nodos. No es preciso administrar manualmente varias versiones de los archivos de la aplicación en Azure Storage ni en los nodos. Y tampoco es preciso preocuparse de generar [direcciones URL de SAS](../storage/common/storage-sas-overview.md) para proporcionar acceso a los archivos de su cuenta de Almacenamiento. Batch funciona en segundo plano con Azure Storage para almacenar paquetes de aplicación e implementarlos en los nodos de proceso.

> [!NOTE] 
> El tamaño total de una tarea de inicio debe ser menor o igual a 32 768 caracteres, incluidos los archivos de recursos y las variables de entorno. Si la tarea de inicio supera este límite, en este caso usar paquetes de aplicación es otra opción. Puede crear también un archivo comprimido que contiene los archivos de recursos, cargarlo como un blob en Azure Storage y. después, descomprímalo en línea de comandos de la tarea de inicio. 
>
>

## <a name="upload-and-manage-applications"></a>Carga y administración de aplicaciones
Puede usar [Azure Portal][portal] o las API de Batch Management para administrar los paquetes de aplicaciones en la cuenta de Batch. En las siguientes secciones, primero se muestra cómo vinculará primero una cuenta de Storage y, después, se analizará la incorporación de paquetes y aplicaciones y su administración con el portal.

### <a name="link-a-storage-account"></a>Vínculo a una cuenta de Almacenamiento
Para utilizar paquetes de aplicación, primero se debe vincular una [cuenta de Azure Storage](accounts.md#azure-storage-accounts) a su cuenta de Batch. Si aún no ha configurado ninguna cuenta de Storage, Azure Portal muestra una advertencia la primera vez que hace clic en **Aplicaciones** en la cuenta de Batch.



![Advertencia de que no hay cuentas de almacenamiento configuradas en Azure Portal][9]

El servicio Batch utiliza la cuenta de Storage asociada para almacenar los paquetes de aplicación. Una vez que haya vinculado las dos cuentas, Batch puede implementar automáticamente los paquetes almacenados en la cuenta de Almacenamiento vinculada en los nodos de proceso. Para vincular una cuenta de Storage a su cuenta de Batch, haga clic en **Cuenta de almacenamiento** en la ventana **Advertencia** y, después, en **Cuenta de almacenamiento** de nuevo.

![Hoja Elegir de cuenta de almacenamiento en Azure Portal][10]

Se recomienda crear una cuenta de Storage *específicamente* para su uso con la cuenta de Batch y seleccionarla aquí. Una vez que haya creado una cuenta de Storage, puede vincularla a su cuenta de Batch mediante la ventana **Cuenta de almacenamiento**.

> [!IMPORTANT] 
> - Actualmente no puede utilizar paquetes de aplicación con una cuenta de Azure Storage que esté configurada con [reglas de firewall](../storage/common/storage-network-security.md).
> - No se puede usar una cuenta de Azure Storage con el **espacio de nombres jerárquico** establecido en **Habilitado** para los paquetes de aplicación.

El servicio Batch utiliza Azure Storage para almacenar los paquetes de aplicación como blobs en bloques. Los datos de blob en bloques se [cobran al precio habitual][storage_pricing] y el tamaño de cada paquete no puede superar el tamaño máximo del blob en bloques. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage para las cuentas de almacenamiento](../storage/blobs/scalability-targets.md). Asegúrese de considerar el tamaño y número de los paquetes de aplicación y elimine periódicamente los paquetes en desuso para minimizar los costos.

### <a name="view-current-applications"></a>Visualización de las aplicaciones actuales
Para ver las aplicaciones en la cuenta de Batch, haga clic en el elemento de menú **Aplicaciones** del menú izquierdo mientras ve la **cuenta de Batch**.

![Icono Aplicaciones][2]

Al seleccionar esta opción de menú se abre la ventana **Aplicaciones**:

![Lista de aplicaciones][3]

Esta ventana muestra el identificador de cada aplicación en su cuenta y las propiedades siguientes:

* **Paquetes**: el número de versiones asociadas a la aplicación.
* **Versión predeterminada**: la versión de la aplicación que se instala si no indica ninguna versión al especificar la aplicación para un grupo. Esta configuración es opcional.
* **Permitir actualizaciones**: el valor que especifica si se permiten las actualizaciones, eliminaciones y adiciones en el paquete. Si se establece en **No**, las actualizaciones y eliminaciones se deshabilitan para la aplicación. Solo se pueden agregar versiones nuevas del paquete de aplicación. El valor predeterminado es **Sí**.

Si desea ver la estructura de archivos del paquete de aplicación en su nodo de ejecución, diríjase a su cuenta de Batch en el portal. En la cuenta de Batch, diríjase a **Grupos**. Seleccione el grupo que contiene los nodos de ejecución en los que está interesado.

![Nodos en el grupo][13]

Una vez que haya seleccionado el grupo, diríjase al nodo de ejecución que en el que está instalado el paquete de aplicación. Los detalles del paquete de aplicación se encuentran en la carpeta **aplicaciones**. Las carpetas adicionales del modo de ejecución contienen otros archivos, como las tareas de inicio, archivos de salida, salida de errores, etc.

![Archivos en el nodo][14]

### <a name="view-application-details"></a>Visualización de los detalles de una aplicación
Para consultar los detalles de una aplicación, selecciónela en la ventana **Aplicaciones**.

![Detalles de la aplicación][4]

En los detalles de la aplicación, puede configurar los siguientes valores de la aplicación.

* **Permitir actualizaciones**: especifica si se pueden actualizar o eliminar sus paquetes de aplicación. Consulte "Actualización o eliminación de un paquete de aplicación" más adelante en este artículo.
* **Versión predeterminada**: especifica el paquete de aplicación predeterminado que se implementa en los nodos de proceso.
* **Nombre para mostrar**: especifica un nombre descriptivo que la solución de Batch puede usar al mostrar información de la aplicación, por ejemplo, en la interfaz de usuario de un servicio que proporciona a los clientes a través de Batch.

### <a name="add-a-new-application"></a>Adición de una nueva aplicación
Para crear una aplicación, agregue un paquete de aplicación y especifique un identificador de aplicación nuevo y exclusivo. El primer paquete de aplicación que agregue con el nuevo identificador de aplicación también crea la nueva aplicación.

Haga clic en **Aplicaciones** > **Agregar**.

![Hoja Nueva aplicación en Azure Portal][5]

La ventana **Nueva aplicación** proporciona los siguientes campos para especificar la configuración de la nueva aplicación y del paquete de aplicación.

**Identificador de la aplicación**

Este campo especifica el identificador de la nueva aplicación, que está sujeto a las reglas de validación estándar de identificadores de Azure Batch. Las reglas para proporcionar un identificador de aplicación son las siguientes:

* En los nodos de Windows, el identificador puede contener cualquier combinación de caracteres alfanuméricos, guiones y caracteres de subrayado. En los nodos de Linux, solo se permiten caracteres alfanuméricos y caracteres de subrayado.
* No pueden contener más de 64 caracteres.
* Deben ser único en la cuenta de Batch.
* Conserva las mayúsculas y minúsculas, aunque no las distingue.

**Versión**

Este campo especifica la versión del paquete de aplicación que se carga. Las cadenas de la versión están sujetas a las siguientes reglas de validación:

* En los nodos de Windows, la cadena de versión puede contener cualquier combinación de caracteres alfanuméricos, guiones, caracteres de subrayado y puntos. En los nodos de Linux, la cadena de versión solo puede contener caracteres alfanuméricos y de subred.
* No pueden contener más de 64 caracteres.
* Deben ser únicas en la aplicación.
* Conservan las mayúsculas y minúsculas, aunque no las distinguen.

**Paquete de aplicación**

Este campo especifica el archivo .zip que contiene los archivos binarios y los archivos auxiliares necesarios para ejecutar la aplicación. Haga clic en el cuadro **Seleccionar un archivo** o en el icono de la carpeta a la que desee desplazarse y seleccione un archivo .zip que contenga los archivos de la aplicación.

Una vez que haya seleccionado un archivo, haga clic en **Aceptar** para iniciar la carga en Azure Storage. Una vez completada la operación de carga, el portal muestra una notificación. En función del tamaño del archivo que se va a cargar y de la velocidad de la conexión de red, esta operación puede tardar un tiempo.

> [!WARNING]
> No cierre la ventana **Nueva aplicación** antes de que se complete la operación de carga. Si lo hace, se detendrá el proceso de carga.
> 
> 

### <a name="add-a-new-application-package"></a>Adición de un nuevo paquete de aplicación
Para agregar una versión del paquete de aplicación de una aplicación existente, seleccione una aplicación en la ventana **Aplicaciones**, haga clic en **Paquetes** > **Agregar**.

![Hoja Agregar paquete de aplicación en Azure Portal][8]

Como puede ver, los campos coinciden con los de la ventana **Nueva aplicación**, excepto el cuadro de texto **Id. de aplicación**, que está deshabilitado. Como hizo para la nueva aplicación, especifique la **Versión** del paquete nuevo, vaya al archivo .zip de su **Paquete de aplicación** y haga clic en **Aceptar** para cargar el paquete.

### <a name="update-or-delete-an-application-package"></a>Actualización o eliminación de un paquete de aplicación
Para actualizar o eliminar un paquete de aplicación existente, abra los detalles de la aplicación, haga clic en **Paquetes** y en los **puntos suspensivos** de la fila del paquete de aplicación que desee modificar, y seleccione la acción que desee realizar.

![Actualizar o eliminar paquete en Azure Portal][7]

**Actualizar**

Al hacer clic en **Actualizar**, se muestra la ventana **Actualizar paquete**. Esta ventana es similar a la de **New application package** (Nuevo paquete de aplicación). Sin embargo, solo está habilitado el campo de selección de paquete, lo que permite especificar un nuevo archivo ZIP para cargarlo.

![Hoja Actualizar paquete en Azure Portal][11]

**Eliminar**

Al hacer clic en **Eliminar**, se le pedirá que confirme la eliminación de la versión del paquete y Batch eliminará el paquete de Azure Storage. Si elimina la versión predeterminada de una aplicación, la configuración de la **Versión predeterminada** se quita de la aplicación.

![Eliminar aplicación][12]

## <a name="install-applications-on-compute-nodes"></a>Instalación de aplicaciones en nodos de proceso
Ahora que ha aprendido cómo administrar paquetes de aplicación con Azure Portal, podemos analizar cómo implementarlos en los nodos de proceso y ejecutarlos con las tareas de Batch.

### <a name="install-pool-application-packages"></a>Instalación de paquetes de aplicación de grupos
Para instalar un paquete de aplicación en todos los nodos de proceso de un grupo, especifique una o varias *referencias* de paquetes de aplicación para el grupo. Los paquetes de aplicación que especifique para un grupo se instalan en cada nodo de proceso cuando este se una al grupo, además de cuando se reinicie o se restablezca la imagen inicial.

En el entorno de .NET para Batch, especifique una o varias [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref] al crear el grupo o para un grupo existente. La clase [ApplicationPackageReference][net_pkgref] especifica una versión y un identificador de la aplicación que se va a instalar en los nodos de proceso de un grupo.

```csharp
// Create the unbound CloudPool
CloudPool myCloudPool =
    batchClient.PoolOperations.CreatePool(
        poolId: "myPool",
        targetDedicatedComputeNodes: 1,
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Specify the application and version to install on the compute nodes
myCloudPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "litware",
        Version = "1.1001.2b" }
};

// Commit the pool so that it's created in the Batch service. As the nodes join
// the pool, the specified application package is installed on each.
await myCloudPool.CommitAsync();
```

> [!IMPORTANT]
> Si, por cualquier motivo, se produce un error en una implementación del paquete de aplicación, el servicio Batch marca el nodo como [inutilizable][net_nodestate] y no se programarán tareas de ejecución en ese nodo. En este caso, debería **reiniciar** el nodo para reiniciar la implementación del paquete. Al reiniciar el nodo, también se vuelve a habilitar la programación de tareas en el nodo.
> 
> 

### <a name="install-task-application-packages"></a>Instalación de paquetes de aplicación de tareas
De forma similar a un grupo, puede especificar las *referencias* de un paquete de aplicación para una tarea. Cuando una tarea está programada para ejecutarse en un nodo, el paquete se descarga y extrae justo antes de ejecutar la línea de comandos de la tarea. Si el paquete y versión especificados ya están instalados en el nodo, el paquete no se descarga y se utiliza el paquete existente.

Para instalar un paquete de aplicación de tareas, configure la propiedad [CloudTask][net_cloudtask].[ApplicationPackageReferences][net_cloudtask_pkgref]:

```csharp
CloudTask task =
    new CloudTask(
        "litwaretask001",
        "cmd /c %AZ_BATCH_APP_PACKAGE_LITWARE%\\litware.exe -args -here");

task.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference
    {
        ApplicationId = "litware",
        Version = "1.1001.2b"
    }
};
```

## <a name="execute-the-installed-applications"></a>Ejecución de las aplicaciones instaladas
Los paquetes que ha especificado para un grupo o tarea se descargan y extraen en un directorio con nombre dentro del nodo `AZ_BATCH_ROOT_DIR` . Batch también permite crear una variable de entorno que contiene la ruta de acceso al directorio con nombre. Las líneas de comando de la tarea usan esta variable de entorno al hacer referencia a la aplicación en el nodo. 

En los nodos de Windows, la variable está en el formato siguiente:

```
Windows:
AZ_BATCH_APP_PACKAGE_APPLICATIONID#version
```

En los nodos de Linux, el formato es ligeramente diferente. Los puntos (.), guiones (-) y signos de número (##) se convierten en caracteres de subrayado en la variable de entorno. Además, tenga en cuenta que se conserva el caso del identificador de la aplicación. Por ejemplo:

```
Linux:
AZ_BATCH_APP_PACKAGE_applicationid_version
```

`APPLICATIONID` y `version` son los valores que corresponden a la versión de la aplicación y del paquete que ha especificado para la implementación. Por ejemplo, si especificó que se debía instalar la versión 2.7 de la aplicación *blender* en los nodos de Windows, las líneas de comando de la tarea usarán esta variable de entorno para tener acceso a sus archivos:

```
Windows:
AZ_BATCH_APP_PACKAGE_BLENDER#2.7
```

En los nodos de Linux, especifique la variable de entorno con este formato. Acople los puntos (.), guiones (-) y almohadillas (#) a los caracteres de subrayado y conserve el caso del identificador de la aplicación:

```
Linux:
AZ_BATCH_APP_PACKAGE_blender_2_7
``` 

Cuando carga un paquete de aplicación, puede especificar una versión predeterminada que implementar en los nodos de proceso. Si ha especificado una versión predeterminada para una aplicación, puede omitir el sufijo de versión al hacer referencia a ella. Puede especificar la versión predeterminada de la aplicación en Azure Portal, en la ventana **Aplicaciones**, como se muestra en [Carga y administración de aplicaciones](#upload-and-manage-applications).

Por ejemplo, si especifica la versión "2.7" como la versión predeterminada de la aplicación *blender*, las tareas pueden hacer referencia a la siguiente variable de entorno y, después, los nodos de Windows ejecutarán la versión 2.7:

`AZ_BATCH_APP_PACKAGE_BLENDER`

El siguiente fragmento de código muestra una línea de comandos de una tarea de ejemplo que inicia la versión predeterminada de la aplicación *blender* :

```csharp
string taskId = "blendertask01";
string commandLine =
    @"cmd /c %AZ_BATCH_APP_PACKAGE_BLENDER%\blender.exe -args -here";
CloudTask blenderTask = new CloudTask(taskId, commandLine);
```

> [!TIP]
> Para más información sobre la configuración del entorno del nodo de proceso, consulte [Configuración del entorno para las tareas](jobs-and-tasks.md#environment-settings-for-tasks). 

## <a name="update-a-pools-application-packages"></a>Actualización de los paquetes de aplicación de un grupo
Si un grupo existente ya se ha configurado con un paquete de aplicación, se puede especificar un paquete nuevo para el grupo. Si especifica una nueva referencia de paquete para un grupo, se aplica lo siguiente:

* El servicio Batch instala el paquete recién especificado en todos los nodos nuevos que se unen al grupo y en cualquier nodo existente que se reinicie o cuya imagen inicial se restablezca.
* Los nodos de proceso que ya estén en el grupo cuando se actualicen las referencias del paquete no instalan automáticamente el paquete de aplicación nuevo. Estos nodos de proceso deben reiniciarse o se debe restablecer su imagen inicial para recibir el nuevo paquete.
* Cuando se implementa un nuevo paquete, las variables de entorno creadas reflejan las nuevas referencias del paquete de aplicación.

En este ejemplo, el grupo existente tiene la versión 2.7 de la aplicación *blender* configurada como una de sus propiedades [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref]. Para actualizar los nodos del grupo con la versión 2.76b, especifique una nueva clase [ApplicationPackageReference][net_pkgref] con la nueva versión y confirme el cambio.

```csharp
string newVersion = "2.76b";
CloudPool boundPool = await batchClient.PoolOperations.GetPoolAsync("myPool");
boundPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "blender",
        Version = newVersion }
};
await boundPool.CommitAsync();
```

Ahora que se ha configurado la nueva versión, el servicio Batch instala la versión 2.76b en todos los nodos *nuevos* que se unan al grupo. Para instalar la versión 2.76b en los nodos que *ya* están en el grupo, reinícielos o restablezca su imagen inicial. Tenga en cuenta que los nodos reiniciados conservan los archivos de las anteriores implementaciones del paquete.

## <a name="list-the-applications-in-a-batch-account"></a>Enumeración de las aplicaciones en una cuenta de Batch
Puede enumerar las aplicaciones y sus paquetes en una cuenta de Batch mediante el método [ApplicationOperations][net_appops].[ListApplicationSummaries][net_appops_listappsummaries].

```csharp
// List the applications and their application packages in the Batch account.
List<ApplicationSummary> applications = await batchClient.ApplicationOperations.ListApplicationSummaries().ToListAsync();
foreach (ApplicationSummary app in applications)
{
    Console.WriteLine("ID: {0} | Display Name: {1}", app.Id, app.DisplayName);

    foreach (string version in app.Versions)
    {
        Console.WriteLine("  {0}", version);
    }
}
```

## <a name="wrap-up"></a>Encapsulado
Con los paquetes de aplicación puede ayudar a los clientes a seleccionar las aplicaciones para sus trabajos y especificar la versión exacta que deben usar al procesar los trabajos con su servicio con Batch habilitado. También puede proporcionar a los clientes la capacidad de cargar y hacer un seguimiento de sus propias aplicaciones en su servicio.

## <a name="next-steps"></a>Pasos siguientes
* La [API de REST de Batch][api_rest] también proporciona compatibilidad para trabajar con paquetes de aplicación. Por ejemplo, consulte el elemento [applicationPackageReferences][rest_add_pool_with_packages] de [Agregar un grupo a una cuenta][rest_add_pool] para especificar los paquetes que se instalan mediante la API REST. Para ver detalles sobre cómo obtener información de la aplicación mediante la API de REST de Batch, consulte [Aplicaciones][rest_applications].
* Aprenda a [administrar mediante programación cuentas y cuotas de Azure Batch con Batch Management .NET](batch-management-dotnet.md). La biblioteca de [Batch Management .NET][api_net_mgmt] puede habilitar las características de creación y eliminación de cuentas de una aplicación o servicio de Batch.

[api_net]: /dotnet/api/overview/azure/batch/client
[api_net_mgmt]: /dotnet/api/overview/azure/batch/management
[api_rest]: /rest/api/batchservice/
[batch_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[github_samples]: https://github.com/Azure/azure-batch-samples
[storage_pricing]: https://azure.microsoft.com/pricing/details/storage/
[net_appops]: /dotnet/api/microsoft.azure.batch.applicationoperations
[net_appops_listappsummaries]: /dotnet/api/microsoft.azure.batch.applicationoperations
[net_cloudpool]: /dotnet/api/microsoft.azure.batch.cloudpool
[net_cloudpool_pkgref]: /dotnet/api/microsoft.azure.batch.cloudpool
[net_cloudtask]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_cloudtask_pkgref]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_nodestate]: /dotnet/api/microsoft.azure.batch.computenode
[net_pkgref]: /dotnet/api/microsoft.azure.batch.applicationpackagereference
[portal]: https://portal.azure.com
[rest_applications]: /rest/api/batchservice/application
[rest_add_pool]: /rest/api/batchservice/pool/add
[rest_add_pool_with_packages]: /rest/api/batchservice/pool/add#bk_apkgreference

[1]: ./media/batch-application-packages/app_pkg_01.png "Diagrama de alto nivel de paquetes de aplicación"
[2]: ./media/batch-application-packages/app_pkg_02.png "Icono Aplicaciones en Azure Portal"
[3]: ./media/batch-application-packages/app_pkg_03.png "Hoja Aplicaciones en Azure Portal"
[4]: ./media/batch-application-packages/app_pkg_04.png "Hoja Detalles de aplicación en Azure Portal"
[5]: ./media/batch-application-packages/app_pkg_05.png "Hoja Nueva aplicación en Azure Portal"
[7]: ./media/batch-application-packages/app_pkg_07.png "Lista desplegable Actualizar o eliminar paquetes en Azure Portal"
[8]: ./media/batch-application-packages/app_pkg_08.png "Hoja Nuevo paquete de aplicación en Azure Portal"
[9]: ./media/batch-application-packages/app_pkg_09.png "Alerta de ausencia de cuenta de almacenamiento vinculada"
[10]: ./media/batch-application-packages/app_pkg_10.png "Hoja Elegir de cuenta de almacenamiento en Azure Portal"
[11]: ./media/batch-application-packages/app_pkg_11.png "Hoja Actualizar paquete en Azure Portal"
[12]: ./media/batch-application-packages/app_pkg_12.png "Cuadro de diálogo de confirmación de eliminación de paquetes en Azure Portal"
[13]: ./media/batch-application-packages/package-file-structure.png "Información del nodo de ejecución en Azure Portal"
[14]: ./media/batch-application-packages/package-file-structure-node.png "Archivos del nodo de ejecución que aparecen en Azure Portal"
