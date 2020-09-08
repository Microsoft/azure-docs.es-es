---
title: 'Recepción de eventos mediante el host del procesador de eventos: Azure Event Hubs | Microsoft Docs'
description: En este artículo se describe el host del procesador de eventos de Azure Event Hubs, que simplifica la administración de los eventos de punto de comprobación, concesión y lectura en paralelo.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 41778425a0ec6ba1732c8e604dead2deb7c97f12
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936187"
---
# <a name="event-processor-host"></a>Host del procesador de eventos
> [!NOTE]
> Este artículo se aplica a la versión anterior del SDK de Azure Event Hubs. Para aprender a migrar el código a la versión más reciente del SDK, consulte estas guías de migración. 
> - [.NET](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md)
> - [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/migration-guide.md)
> - [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/migration_guide.md)
> - [Script de Java](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/event-hubs/migrationguide.md)
>
> Consulte también [Equilibrio de la carga de particiones entre varias instancias de la aplicación](event-processor-balance-partition-load.md).

Azure Event Hubs es un eficaz servicio de ingesta de telemetría que se puede usar para hacer streaming de millones de eventos a un bajo costo. En este artículo se describe cómo usar eventos ingeridos mediante el *host del procesador de eventos* (EPH), un agente de consumidor inteligente que simplifica la administración de la creación de puntos de comprobación, la concesión y los lectores de eventos paralelos.  

La clave del escalado de Event Hubs es el concepto de consumidores con particiones. En contraposición al patrón de [consumidores de la competencia](/previous-versions/msp-n-p/dn568101(v=pandp.10)), el patrón de consumidores con particiones permite una alta escalabilidad mediante la eliminación de cuellos de botella de contención y la facilitación del paralelismo de principio a fin.

## <a name="home-security-scenario"></a>Escenario de seguridad en el hogar

Como caso de ejemplo, considere una empresa de seguridad en el hogar que supervisa 100 000 casas. Cada minuto obtiene los datos de los diversos sensores como el detector de movimiento, el sensor de apertura de puertas y ventanas, el detector de rotura de cristales, etc, instalados en cada casa. La empresa proporciona un sitio web para que los residentes supervisen la actividad de su casa casi en tiempo real.

Cada sensor inserta datos en un centro de eventos. El centro de eventos está configurado con 16 particiones. En el extremo de consumo, necesita un mecanismo que pueda leer estos eventos, consolidarlos (filtrarlos, agregarlos, etc.) y volcar el agregado a un blob de almacenamiento que, a continuación, se proyecta en una página web fácil de usar.

## <a name="write-the-consumer-application"></a>Escritura de la aplicación de consumidor

Al diseñar el consumidor en un entorno distribuido, el escenario debe controlar los siguientes requisitos:

1. **Escalado:** cree varios consumidores y que cada consumidor tome posesión de la lectura desde varias particiones de Event Hubs.
2. **Equilibrio de carga:** aumente o reduzca dinámicamente los consumidores. Por ejemplo, si se agrega un nuevo tipo de sensor (por ejemplo, un detector de monóxido de carbono) a cada casa, aumenta el número de eventos. En ese caso, el operador (una persona) aumenta el número de instancias de consumidor. A continuación, el grupo de consumidores puede volver a equilibrar el número de particiones que poseen para compartir la carga con los consumidores recién agregados.
3. **Reanudación sin problemas después de los errores:** si un consumidor (**consumidor A**) genera un error (por ejemplo, la máquina virtual que hospeda al consumidor, de repente, se bloquea), otros consumidores deben poder recopilar las particiones que posee el **consumidor A** y continuar. Además, el punto de continuación, llamado *punto de comprobación* o de *desplazamiento*, debe estar en el punto exacto en el que se produjo el error del **consumidor A** o ligeramente antes.
4. **Consumo de eventos:** mientras que los tres puntos anteriores trataban sobre la administración del consumidor, también tiene que haber código para consumir los eventos y hacer algo útil con ellos como, por ejemplo, agregarlos y cargarlos en Blob Storage.

En lugar de compilar su propia solución para esto, Event Hubs ofrece esta funcionalidad mediante la interfaz de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) y la clase [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost).

## <a name="ieventprocessor-interface"></a>Interfaz de IEventProcessor

En primer lugar, las aplicaciones de consumo implementan la interfaz de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor), que tiene cuatro métodos: [OpenAsync, CloseAsync, ProcessErrorAsync y ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor?view=azure-dotnet#methods). Esta interfaz contiene el código real para consumir los eventos que envía Event Hubs. El código siguiente muestra una implementación sencilla:

```csharp
public class SimpleEventProcessor : IEventProcessor
{
    public Task CloseAsync(PartitionContext context, CloseReason reason)
    {
       Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
       return Task.CompletedTask;
    }

    public Task OpenAsync(PartitionContext context)
    {
       Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
       return Task.CompletedTask;
     }

    public Task ProcessErrorAsync(PartitionContext context, Exception error)
    {
       Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
       return Task.CompletedTask;
    }

    public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
       foreach (var eventData in messages)
       {
          var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
             Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
       }
       return context.CheckpointAsync();
    }
}
```

A continuación, cree una instancia de una instancia de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Según la sobrecarga, al crear la instancia de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) en el constructor, se usan los siguientes parámetros:

- **hostName:** el nombre de cada instancia de consumidor. Cada instancia de **EventProcessorHost** debe tener un valor único para esta variable dentro de un grupo de consumidores, así que no codifique de forma rígida este valor.
- **eventHubPath:** El nombre del centro de eventos.
- **consumerGroupName:** Event Hubs usa **$Default** como nombre del grupo de consumidores predeterminado, pero es recomendable crear un grupo de consumidores para sus necesidades específicas de procesamiento.
- **eventHubConnectionString:** la cadena de conexión al centro de eventos que se puede recuperar desde Azure Portal. Esta cadena de conexión debe tener permisos de **escucha** en el centro de eventos.
- **storageConnectionString:** la cuenta de almacenamiento que se usa para la administración de recursos internos.

Por último, los consumidores registran la instancia de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) con el servicio Event Hubs. Al registrar una clase de procesador de eventos con una instancia de EventProcessorHost, se inicia el procesamiento de eventos. El proceso de registro indica al servicio Event Hubs que debe esperar que la aplicación del consumidor consuma eventos de algunas de sus particiones y que debe invocar el código de implementación de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) siempre que envíe eventos para su consumo. 


### <a name="example"></a>Ejemplo

Por ejemplo, imagine que hay 5 máquinas virtuales dedicadas a consumir eventos y una aplicación de consola simple en cada máquina virtual que es la que realiza el trabajo de consumo real. Cada aplicación de consola crea una instancia de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) y la registra con el servicio Event Hubs.

En este escenario de ejemplo, supongamos que se asignan 16 particiones a las 5 instancias de **EventProcessorHost**. Algunas instancias de **EventProcessorHost** pueden disponer de algunas particiones más que otras. Para cada partición que posee una instancia de **EventProcessorHost** esta crea una instancia de la clase `SimpleEventProcessor`. Por tanto, hay 16 instancias de `SimpleEventProcessor` en total, con una de ellas asignada a cada partición.

En la siguiente lista se resume este ejemplo:

- 16 particiones de Event Hubs.
- 5 máquinas virtuales, 1 aplicación de consumidor (por ejemplo, Consumer.exe) en cada máquina virtual.
- 5 instancias del host del procesador de eventos, 1 en cada máquina virtual con Consumer.exe.
- 16 objetos `SimpleEventProcessor` que crean las 5 instancias del host del procesador de eventos.
- 1 aplicación Consumer.exe puede contener 4 objetos `SimpleEventProcessor`, ya que 1 instancia del host del procesador de eventos puede poseer 4 particiones.

## <a name="partition-ownership-tracking"></a>Seguimiento de la propiedad de una partición

Se puede realizar un seguimiento de la propiedad de una partición en una instancia del host del procesador de eventos (o un consumidor) mediante la cuenta de Azure Storage que se proporciona para tal fin. Puede visualizar el seguimiento en una sencilla tabla como se indica a continuación. Puede ver la implementación real examinando los blobs en la cuenta de Storage proporcionada:

| **Nombre del grupo de consumidores** | **Identificador de la partición** | **Nombre de host (propietario)** | **Tiempo adquirido de concesión (o propiedad)** | **Desplazamiento en la partición (punto de comprobación)** |
| --- | --- | --- | --- | --- |
| $Default | 0 | Consumer\_VM3 | 2018-04-15T01:23:45 | 156 |
| $Default | 1 | Consumer\_VM4 | 2018-04-15T01:22:13 | 734 |
| $Default | 2 | Consumer\_VM0 | 2018-04-15T01:22:56 | 122 |
| : |   |   |   |   |
| : |   |   |   |   |
| $Default | 15 | Consumer\_VM3 | 2018-04-15T01:22:56 | 976 |

En este caso, cada host adquiere la propiedad de una partición durante un determinado período de tiempo (la duración de la concesión). Si se produce un error en un host (se apaga la máquina virtual), la concesión expira. Otros hosts intentan obtener la propiedad de la partición y uno de ellos lo consigue. Este proceso restablece la concesión de la partición con un nuevo propietario. De este modo, solo un lector a la vez puede leer en una determinada partición de un grupo de consumidores.

## <a name="receive-messages"></a>Recepción de mensajes

Cada llamada a [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) ofrece una colección de eventos. Es su responsabilidad administrar estos eventos. Si desea asegurarse de que el host del procesador procesa cada mensaje al menos una vez, deberá escribir su propio código de reintento. Pero tenga cuidado con los mensajes dudosos.

Se recomienda que el proceso se realice relativamente rápido, es decir, con el menor procesamiento posible. En su lugar, utilice grupos de consumidores. Si tiene que escribir en el almacenamiento y hacer algo de enrutamiento, es mejor usar dos grupos de consumidores y tener dos implementaciones de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) que se ejecuten por separado.

En algún momento durante el procesamiento, es posible que desee realizar un seguimiento de lo que ha leído y completado. Es fundamental realizar un seguimiento si debe reiniciar la lectura, para no tener que volver al principio de la transmisión. [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) simplifica este seguimiento mediante el uso de *puntos de comprobación*. Un punto de comprobación es una ubicación o desplazamiento, de una partición determinada, dentro de un grupo de consumidores determinado, en el que está satisfecho con los mensajes que se han procesado. Para marcar un punto de comprobación en **EventProcessorHost** llame al método [CheckpointAsync](/dotnet/api/microsoft.azure.eventhubs.processor.partitioncontext.checkpointasync) en el objeto [PartitionContext](/dotnet/api/microsoft.azure.eventhubs.processor.partitioncontext). Esta operación se realiza en el método [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) pero también se puede realizar en [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.closeasync).

## <a name="checkpointing"></a>Puntos de control

El método [CheckpointAsync](/dotnet/api/microsoft.azure.eventhubs.processor.partitioncontext.checkpointasync) tiene dos sobrecargas: la primera, sin parámetros, crea un punto de comprobación en el desplazamiento del evento más alto de la colección que devuelve [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync). Este desplazamiento es una marca de "límite superior" que supone que ha procesado todos los eventos recientes cuando lo llama. Si usa este método de esta manera, tenga en cuenta que se espera que lo llame una vez que ha devuelto el otro código de procesamiento de eventos. La segunda sobrecarga le permite especificar una instancia de [EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata) para crear un punto de comprobación. Este método le permite usar un tipo diferente de marca de agua para crear un punto de comprobación Con esta marca de agua, puede implementar una marca "de límite inferior": el evento más bajo de la secuencia que está seguro de haber procesado. Esta sobrecarga se proporciona para ofrecer flexibilidad en la administración de desplazamientos.

Cuando se realiza la creación de un punto de comprobación, se escribe un archivo JSON con información específica de la partición en la cuenta de almacenamiento que se proporcionó en el constructor de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Este archivo se actualiza continuamente. Es fundamental considerar la creación de puntos de comprobación en contexto. No es recomendable crear un punto de comprobación en cada mensaje. La cuenta de almacenamiento que se utilizó para la creación de puntos de comprobación no podrá administrar esta carga y, lo que es aún más importante, la creación de puntos de comprobación para todos los eventos puede indicar un patrón de mensajería en cola para el que una cola de Service Bus sería una opción mejor que un centro de eventos. La ventaja de Event Hubs es que obtiene al menos una entrega a gran escala. Al hacer los sistemas de nivel final idempotentes, es fácil recuperarse de errores o reinicios que hacen que los eventos se reciban múltiples veces.

## <a name="thread-safety-and-processor-instances"></a>Seguridad para subprocesos e instancias de procesador

De forma predeterminada, [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) es un subproceso seguro y se comporta de una forma sincrónica con respecto a la instancia de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor). Cuando llegan eventos para una partición, se llama a [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) en la instancia de **IEventProcessor** de esa partición y se bloquean el resto de llamadas a **ProcessEventsAsync** de la partición. Los siguientes mensajes y llamadas a **ProcessEventsAsync** se ponen en cola en segundo plano ya que el suministro de mensajes continúa ejecutándose en segundo plano en otros subprocesos. Esta seguridad para subprocesos elimina la necesidad de colecciones seguras para subprocesos y aumenta considerablemente el rendimiento.

## <a name="shut-down-gracefully"></a>Cierre correcto

Por último, [EventProcessorHost.UnregisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.unregistereventprocessorasync) permite un cierre correcto de todos los lectores de partición y se le debería llamar siempre al cerrar una instancia de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Si no lo hace, puede provocar retrasos al iniciar otras instancias de **EventProcessorHost** debido a conflictos de época y de expiración de concesión. La administración de épocas se trata detalladamente en la sección [Época](#epoch) del artículo. 

## <a name="lease-management"></a>Administración de concesiones
Al registrar una clase de procesador de eventos con una instancia de EventProcessorHost, se inicia el procesamiento de eventos. La instancia de host obtiene las concesiones sobre algunas particiones del centro de eventos, posiblemente al tomar algunas de otras instancias, de una forma que converge en una distribución uniforme de particiones mediante todas las instancias de host. Para cada partición de la concesión, la instancia de host crea una instancia de la clase de procesador de eventos proporcionada, después, recibe eventos de esa partición y los pasa a la instancia de procesador de eventos. A medida que más casos se agregan y más concesiones se toman, EventProcessorHost equilibra finalmente la carga entre todos los consumidores.

Como se explicó anteriormente, la tabla de seguimiento simplifica en gran medida la naturaleza de escalabilidad automática de [EventProcessorHost.UnregisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.unregistereventprocessorasync). Cuando se inicia una instancia de **EventProcessorHost**, esta adquiere el mayor número de concesiones posible y empieza a leer eventos. A medida que la concesión se acerca a su fin, **EventProcessorHost** intenta renovarla realizando una reserva. Si la concesión está disponible para la renovación, el procesador continúa la lectura, pero si no es así, el lector se cierra y se realiza una llamada a [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.closeasync). **CloseAsync** es una buena oportunidad para realizar cualquier tarea de limpieza final para esa partición.

**EventProcessorHost** incluye una propiedad [PartitionManagerOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.partitionmanageroptions). Esta propiedad habilita el control sobre la administración de concesiones. Establezca estas opciones antes de registrar la implementación de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor).

## <a name="control-event-processor-host-options"></a>Control de las opciones del host del procesador de eventos

Además, una sobrecarga de [RegisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync?view=azure-dotnet#Microsoft_Azure_EventHubs_Processor_EventProcessorHost_RegisterEventProcessorAsync__1_Microsoft_Azure_EventHubs_Processor_EventProcessorOptions_) toma un objeto [EventProcessorOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync?view=azure-dotnet#Microsoft_Azure_EventHubs_Processor_EventProcessorHost_RegisterEventProcessorAsync__1_Microsoft_Azure_EventHubs_Processor_EventProcessorOptions_) como parámetro. Utilice este parámetro para controlar el comportamiento del propio [EventProcessorHost.UnregisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.unregistereventprocessorasync). [EventProcessorOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions) define cuatro propiedades y un evento:

- [MaxBatchSize](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.maxbatchsize): el tamaño máximo de la colección que desea recibir en una invocación de [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync). Este tamaño no incluye el tamaño mínimo, solo el máximo. Si hay menos mensajes que recibir, **ProcessEventsAsync** se ejecuta con tantos como haya disponibles.
- [PrefetchCount](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.prefetchcount): un valor utilizado por el canal AMQP subyacente para determinar el límite superior de mensajes que debe recibir el cliente. Este valor debe ser superior o igual a [MaxBatchSize](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.maxbatchsize).
- [InvokeProcessorAfterReceiveTimeout](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.invokeprocessorafterreceivetimeout): si este parámetro es **true**, se realizará una llamada a [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) cuando se agote el tiempo de espera de la llamada subyacente para recibir eventos en una partición. Este método es útil para adoptar acciones basadas en tiempo durante los períodos de inactividad de la partición.
- [InitialOffsetProvider](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.initialoffsetprovider): permite que se establezca un puntero de función o expresión lambda que se usa para proporcionar el desplazamiento inicial cuando un lector comienza la lectura de una partición. Si no se especifica este desplazamiento, el lector se iniciará en el evento más antiguo, a menos que ya se haya guardado un archivo JSON con un desplazamiento en la cuenta de almacenamiento que se proporcionó al constructor de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Este método es útil cuando desea cambiar el comportamiento de inicio del lector. Cuando se invoca este método, el parámetro de objeto contiene el identificador de partición para el que se inicia el lector.
- [ExceptionReceivedEventArgs](/dotnet/api/microsoft.azure.eventhubs.processor.exceptionreceivedeventargs): le permite recibir notificaciones sobre cualquier excepción subyacente que se produzca en [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Si las cosas no funcionan según lo esperado, este evento es un buen lugar para empezar a buscar.

## <a name="epoch"></a>Época

Así es cómo funciona la época de recepción:

### <a name="with-epoch"></a>Con época
La época es un identificador único (valor de tiempo) que usa el servicio para aplicar la propiedad de la partición o de la concesión. Para crear un receptor de época se usa el método [CreateEpochReceiver](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createepochreceiver?view=azure-dotnet). Este método crea un receptor basado en época. El receptor se crea para una partición de centro de eventos específica desde el grupo de consumidores especificado.

La característica de época ofrece a los usuarios la posibilidad de garantizar que solo hay un receptor en un grupo de consumidores en cualquier momento dado, con las siguientes reglas:

- Si no hay ningún receptor existente en un grupo de consumidores, el usuario puede crear uno con cualquier valor de época.
- Si hay un receptor con un valor de época de e1 y se crea un receptor con un valor de época de e2 donde e1 < = e2, el receptor con e1 se desconectará automáticamente si el receptor con e2 se crea correctamente.
- Si hay un receptor con un valor de época de e1 y se crea un receptor con un valor de época de e2 donde e1 > e2 y, a continuación, la creación de e2 generará un error que indica que ya existe un receptor con la época e1.

### <a name="no-epoch"></a>No hay época
Creará un receptor no basado en época mediante el método [CreateReceiver](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createreceiver?view=azure-dotnet). 

Hay algunos escenarios en el procesamiento de streaming en los que a los usuarios les gustaría crear varios receptores en un solo grupo de consumidores. Para admitir estos escenarios, tenemos la posibilidad de crear un receptor sin época y, en este caso, se permiten hasta cinco receptores simultáneos en el grupo de consumidores.

### <a name="mixed-mode"></a>Modo mixto
No recomendamos el uso de aplicaciones donde crea un receptor con época y, luego, cambia a sin época o viceversa, en el mismo grupo de consumidores. Sin embargo, cuando se produce este comportamiento, el servicio lo controla mediante las reglas siguientes:

- Si hay un receptor ya ha creado con época e1 y está recibiendo eventos activamente y se crea un receptor sin época, se producirá un error en la creación del receptor. Los receptores de época siempre tienen prioridad en el sistema.
- Si hubiera un receptor ya ha creado con época e1 y se desconectara y se creara un receptor sin época en una nueva instancia de MessagingFactory, la creación del receptor se realizaría correctamente. Hay una salvedad aquí y es que nuestro sistema detectará la "desconexión del receptor" al cabo de unos 10 minutos.
- Si hay uno o varios receptores creados sin época y se crea uno con época e1, se desconectan todos los receptores antiguos.


> [!NOTE]
> Se recomienda el uso de diferentes grupos de consumidores para las aplicaciones que usan épocas y para aquellos que no usan épocas para evitar errores. 


## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con el host del procesador de eventos, consulte los artículos siguientes para más información acerca de Event Hubs:

- Introducción a Event Hubs
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-node-get-started-send.md)
* [Guía de programación de Event Hubs](event-hubs-programming-guide.md)
* [Disponibilidad y coherencia en Event Hubs](event-hubs-availability-and-consistency.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)
* [Ejemplos de Event Hubs en GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples)
