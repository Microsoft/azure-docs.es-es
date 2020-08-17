---
title: Envío o recepción de eventos desde Azure Event Hubs mediante Java (heredados)
description: Este artículo es un tutorial para crear una aplicación de Java que envíe eventos a Azure Event Hubs, o los reciba de él, mediante el anterior paquete azure-eventhubs.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-java
ms.openlocfilehash: 6efa21f4869dddef9d54001a8669b4b039240910
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371981"
---
# <a name="use-java-to-send-events-to-or-receive-events-from-azure-event-hubs-azure-eventhubs"></a>Use Java para enviar eventos a Azure Event Hubs o recibir eventos de él (azure-eventhubs)

En este inicio rápido se muestra cómo enviar y recibir eventos desde un centro de eventos mediante el paquete de Java **azure-eventhubs**.

> [!WARNING]
> En este inicio rápido se usan los anteriores paquetes **azure-eventhubs** y **azure-eventhubs-eph**. Para ver un inicio rápido que use el paquete **azure-messaging-eventhubs**más reciente, consulte [Envío y recepción de eventos mediante azure-messaging-eventhubs](get-started-java-send-v2.md). Para que la aplicación deje de usar el paquete anterior y use el nuevo, consulte la [Guía para migrar desde azure-eventhubs a azure-messaging-eventhubs](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/migration-guide.md). 


## <a name="prerequisites"></a>Prerrequisitos

Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido. 

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- Un entorno de desarrollo de Java. Este inicio rápido utiliza [Eclipse](https://www.eclipse.org/).
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). Luego, para obtener el valor de la clave de acceso del centro de eventos, siga las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Utilice la clave de acceso en el código que escribirá más adelante en este inicio rápido. El nombre de la clave predeterminada es: **RootManageSharedAccessKey**.

## <a name="send-events"></a>Envío de eventos 
En esta sección se muestra cómo crear una aplicación de Java para enviar eventos a un centro de eventos. 

> [!NOTE]
> Puede descargar esta guía de inicio rápido como un ejemplo desde [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/SimpleSend), reemplazar las cadenas `EventHubConnectionString` y `EventHubName` por los valores del centro de eventos, y ejecutarlo. También puede seguir los pasos de este inicio rápido para crear la suya propia.

### <a name="add-reference-to-azure-event-hubs-library"></a>Incorporación de una referencia a la biblioteca de Azure Event Hubs

La biblioteca de cliente de Java para Event Hubs está disponible para su uso en proyectos de Maven desde el [repositorio central de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22). Puede hacer referencia a esta biblioteca con la siguiente declaración de dependencia en el archivo de proyecto de Maven:

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>2.2.0</version>
</dependency>
```

Para distintos tipos de entornos de compilación, puede obtener explícitamente los últimos archivos JAR publicados del [repositorio central de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22).  

Para un editor de eventos simples, importe el paquete *com.microsoft.azure.eventhubs* para las clases de cliente de Event Hubs y el paquete *com.microsoft.azure.servicebus* para las clases de utilidad, como las excepciones comunes que se comparten con el cliente de mensajería de Azure Service Bus. 

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Escritura de código para enviar mensajes al centro de eventos

Para el ejemplo siguiente, primero cree un nuevo proyecto de Maven para una aplicación de consola o shell en su entorno de desarrollo de Java favorito. Agregue una clase denominada `SimpleSend` y agréguele el código siguiente:

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
import com.microsoft.azure.eventhubs.EventData;
import com.microsoft.azure.eventhubs.EventHubClient;
import com.microsoft.azure.eventhubs.EventHubException;

import java.io.IOException;
import java.nio.charset.Charset;
import java.time.Instant;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public class SimpleSend {

    public static void main(String[] args)
            throws EventHubException, ExecutionException, InterruptedException, IOException {
            
            
    }
 }
```

### <a name="construct-connection-string"></a>Crear cadena de conexión

Use la clase ConnectionStringBuilder para construir un valor de cadena de conexión y pasarlo a la instancia de cliente de Event Hubs. Reemplace los marcadores de posición por los valores que obtuvo al crear el espacio de nombres y el centro de eventos:

```java
        final ConnectionStringBuilder connStr = new ConnectionStringBuilder()
                .setNamespaceName("<EVENTHUB NAMESPACE") 
                .setEventHubName("EVENT HUB")
                .setSasKeyName("RootManageSharedAccessKey")
                .setSasKey("SHARED ACCESS KEY");
```

### <a name="write-code-to-send-events"></a>Escritura de código para enviar eventos

Cree un evento singular transformando una cadena en su codificación de bytes UTF-8. Después cree una instancia de cliente de Event Hubs a partir de la cadena de conexión y envíe el mensaje:   

```java 
        final Gson gson = new GsonBuilder().create();

        // The Executor handles all asynchronous tasks and this is passed to the EventHubClient instance.
        // This enables the user to segregate their thread pool based on the work load.
        // This pool can then be shared across multiple EventHubClient instances.
        // The following sample uses a single thread executor, as there is only one EventHubClient instance,
        // handling different flavors of ingestion to Event Hubs here.
        final ScheduledExecutorService executorService = Executors.newScheduledThreadPool(4);

        // Each EventHubClient instance spins up a new TCP/TLS connection, which is expensive.
        // It is always a best practice to reuse these instances. The following sample shows this.
        final EventHubClient ehClient = EventHubClient.createSync(connStr.toString(), executorService);


        try {
            for (int i = 0; i < 10; i++) {

                String payload = "Message " + Integer.toString(i);
                byte[] payloadBytes = gson.toJson(payload).getBytes(Charset.defaultCharset());
                EventData sendEvent = EventData.create(payloadBytes);

                // Send - not tied to any partition
                // Event Hubs service will round-robin the events across all Event Hubs partitions.
                // This is the recommended & most reliable way to send to Event Hubs.
                ehClient.sendSync(sendEvent);
            }

            System.out.println(Instant.now() + ": Send Complete...");
            System.out.println("Press Enter to stop.");
            System.in.read();
        } finally {
            ehClient.closeSync();
            executorService.shutdown();
        }

``` 

Compile y ejecute el programa y asegúrese de que no haya errores.

Felicidades. Ha enviado mensajes a un centro de eventos.

### <a name="appendix-how-messages-are-routed-to-eventhub-partitions"></a>Apéndice: Cómo se enrutan los mensajes a las particiones de EventHub

Antes de que los consumidores puedan recuperar los mensajes, los editores tienen que publicarlos primero en las particiones. Cuando los mensajes se publican en el centro de eventos de manera sincrónica con el método sendSync() en el objeto com.microsoft.azure.eventhubs.EventHubClient, se pueden enviar a una partición específica o distribuirse a todas las particiones disponibles en modo round-robin, según si se especifica o no la clave de partición.

Cuando se especifica una cadena que representa la clave de partición, se aplicará un algoritmo hash a la clave para determinar la partición a la que desea enviar el evento.

Cuando no se establece la clave de partición, los mensajes se envían en modo round-robin a todas las particiones disponibles

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

// close the client at the end of your program
eventHubClient.closeSync();

```

## <a name="receive-events"></a>Recepción de eventos
El código de este tutorial se basa en el [código EventProcessorSample en GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/EventProcessorSample), que se puede examinar para ver toda la aplicación en funcionamiento.

### <a name="receive-messages-with-eventprocessorhost-in-java"></a>Recepción de mensajes con EventProcessorHost en Java

**EventProcessorHost** es una clase de Java que simplifica la recepción de eventos desde Event Hubs mediante la administración de puntos de control persistentes y recepciones paralelas desde tales Centros de eventos. Con EventProcessorHost, puede dividir eventos entre varios destinatarios, incluso cuando están hospedados en distintos nodos. Este ejemplo muestra cómo usar EventProcessorHost para un solo destinatario.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Para usar EventProcessorHost, debe tener una [cuenta de Azure Storage][cuenta de Azure Storage]:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Crear un recurso** en la parte izquierda de la pantalla.
2. Seleccione **Almacenamiento** y, luego, **Cuenta de almacenamiento**. En la ventana **Crear cuenta de almacenamiento**, escriba un nombre para la cuenta de almacenamiento. Complete el resto de los campos, seleccione la región deseada y, finalmente, seleccione **Crear**.
   
    ![Creación de una cuenta de almacenamiento en Azure Portal](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-azure-storage-account.png)

3. Seleccione la cuenta de almacenamiento recién creada y, a continuación, **Claves de acceso**:
   
    ![Obtención de las claves de acceso en Azure Portal](./media/event-hubs-dotnet-framework-getstarted-receive-eph/select-azure-storage-access-keys.png)

    Copie el valor key1 en una ubicación temporal. Lo usará más adelante en este tutorial.

### <a name="create-a-java-project-using-the-eventprocessor-host"></a>Creación de un proyecto de Java mediante EventProcessorHost

La biblioteca de cliente de Java para Event Hubs está disponible para su uso en proyectos de Maven en el [repositorio central de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22), y se puede hacer referencia a ella mediante la siguiente declaración de dependencia en el archivo de proyecto de Maven: 

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs-eph</artifactId>
    <version>2.4.0</version>
</dependency>
```

Para distintos tipos de entornos de compilación, puede obtener explícitamente los últimos archivos JAR publicados del [repositorio central de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22).

1. Para el ejemplo siguiente, primero cree un nuevo proyecto de Maven para una aplicación de consola o shell en su entorno de desarrollo de Java favorito. La clase se denomina `ErrorNotificationHandler`.     
   
    ```java
    import java.util.function.Consumer;
    import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   
    public class ErrorNotificationHandler implements Consumer<ExceptionReceivedEventArgs>
    {
        @Override
        public void accept(ExceptionReceivedEventArgs t)
        {
            System.out.println("SAMPLE: Host " + t.getHostname() + " received general error notification during " + t.getAction() + ": " + t.getException().toString());
        }
    }
    ```
2. Cree una clase nueva denominada `EventProcessorSample`con el código siguiente. Reemplace los siguientes marcadores de posición por los valores que usó al crear el centro de eventos y la cuenta de almacenamiento:
   
   ```java
   package com.microsoft.azure.eventhubs.samples.eventprocessorsample;

   import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
   import com.microsoft.azure.eventhubs.EventData;
   import com.microsoft.azure.eventprocessorhost.CloseReason;
   import com.microsoft.azure.eventprocessorhost.EventProcessorHost;
   import com.microsoft.azure.eventprocessorhost.EventProcessorOptions;
   import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   import com.microsoft.azure.eventprocessorhost.IEventProcessor;
   import com.microsoft.azure.eventprocessorhost.PartitionContext;

   import java.util.concurrent.ExecutionException;
   import java.util.function.Consumer;

   public class EventProcessorSample
   {
       public static void main(String args[]) throws InterruptedException, ExecutionException
       {
           String consumerGroupName = "$Default";
           String namespaceName = "----NamespaceName----";
           String eventHubName = "----EventHubName----";
           String sasKeyName = "----SharedAccessSignatureKeyName----";
           String sasKey = "----SharedAccessSignatureKey----";
           String storageConnectionString = "----AzureStorageConnectionString----";
           String storageContainerName = "----StorageContainerName----";
           String hostNamePrefix = "----HostNamePrefix----";
        
           ConnectionStringBuilder eventHubConnectionString = new ConnectionStringBuilder()
                .setNamespaceName(namespaceName)
                .setEventHubName(eventHubName)
                .setSasKeyName(sasKeyName)
                .setSasKey(sasKey);
        
           EventProcessorHost host = new EventProcessorHost(
                EventProcessorHost.createHostName(hostNamePrefix),
                eventHubName,
                consumerGroupName,
                eventHubConnectionString.toString(),
                storageConnectionString,
                storageContainerName);
        
           System.out.println("Registering host named " + host.getHostName());
           EventProcessorOptions options = new EventProcessorOptions();
           options.setExceptionNotification(new ErrorNotificationHandler());

           host.registerEventProcessor(EventProcessor.class, options)
           .whenComplete((unused, e) ->
           {
               if (e != null)
               {
                   System.out.println("Failure while registering: " + e.toString());
                   if (e.getCause() != null)
                   {
                       System.out.println("Inner exception: " + e.getCause().toString());
                   }
               }
           })
           .thenAccept((unused) ->
           {
               System.out.println("Press enter to stop.");
               try 
               {
                   System.in.read();
               }
               catch (Exception e)
               {
                   System.out.println("Keyboard read failed: " + e.toString());
               }
           })
           .thenCompose((unused) ->
           {
               return host.unregisterEventProcessor();
           })
           .exceptionally((e) ->
           {
               System.out.println("Failure while unregistering: " + e.toString());
               if (e.getCause() != null)
               {
                   System.out.println("Inner exception: " + e.getCause().toString());
               }
               return null;
           })
           .get(); // Wait for everything to finish before exiting main!
        
           System.out.println("End of sample");
       }
   }
   ```
3. Cree una clase final más llamada `EventProcessor`, con el código siguiente:
   
    ```java
    public static class EventProcessor implements IEventProcessor
    {
        private int checkpointBatchingCount = 0;

        // OnOpen is called when a new event processor instance is created by the host. 
        @Override
        public void onOpen(PartitionContext context) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is opening");
        }

        // OnClose is called when an event processor instance is being shut down. 
        @Override
        public void onClose(PartitionContext context, CloseReason reason) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is closing for reason " + reason.toString());
        }
        
        // onError is called when an error occurs in EventProcessorHost code that is tied to this partition, such as a receiver failure.
        @Override
        public void onError(PartitionContext context, Throwable error)
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " onError: " + error.toString());
        }

        // onEvents is called when events are received on this partition of the Event Hub. 
        @Override
        public void onEvents(PartitionContext context, Iterable<EventData> events) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " got event batch");
            int eventCount = 0;
            for (EventData data : events)
            {
                try
                {
                    System.out.println("SAMPLE (" + context.getPartitionId() + "," + data.getSystemProperties().getOffset() + "," +
                            data.getSystemProperties().getSequenceNumber() + "): " + new String(data.getBytes(), "UTF8"));
                    eventCount++;
                    
                    // Checkpointing persists the current position in the event stream for this partition and means that the next
                    // time any host opens an event processor on this event hub+consumer group+partition combination, it will start
                    // receiving at the event after this one. 
                    this.checkpointBatchingCount++;
                    if ((checkpointBatchingCount % 5) == 0)
                    {
                        System.out.println("SAMPLE: Partition " + context.getPartitionId() + " checkpointing at " +
                            data.getSystemProperties().getOffset() + "," + data.getSystemProperties().getSequenceNumber());
                        // Checkpoints are created asynchronously. It is important to wait for the result of checkpointing
                        // before exiting onEvents or before creating the next checkpoint, to detect errors and to ensure proper ordering.
                        context.checkpoint(data).get();
                    }
                }
                catch (Exception e)
                {
                    System.out.println("Processing failed for an event: " + e.toString());
                }
            }
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " batch size was " + eventCount + " for host " + context.getOwner());
        }
    }
    ```

Este tutorial usa una sola instancia de EventProcessorHost. Para aumentar la capacidad de procesamiento, se recomienda ejecutar varias instancias de EventProcessorHost, preferiblemente en máquinas independientes.  Esto proporciona redundancia también. En esos casos, las diferentes instancias se coordinan automáticamente entre sí con el fin de equilibrar la carga de los eventos recibidos. Si desea que varios destinatarios procesen *todos* los eventos, debe usar el concepto **ConsumerGroup** . Cuando se reciben eventos de distintos equipos, puede ser útil especificar nombres para las instancias de EventProcessorHost según los equipos (o roles) en que se implementan.

### <a name="publishing-messages-to-eventhub"></a>Publicación de mensajes en EventHub

Antes de que los consumidores puedan recuperar los mensajes, los editores tienen que publicarlos primero en las particiones. Merece la pena destacar que cuando los mensajes se publican en el centro de eventos de manera sincrónica con el método sendSync() en el objeto com.microsoft.azure.eventhubs.EventHubClient, se podrían enviar a una partición específica o distribuirse a todas las particiones disponibles en modo round-robin, según si se especifica o no la clave de partición.

Cuando se especifica una cadena que representa la clave de partición, se aplicará un algoritmo hash a la clave para determinar la partición a la que desea enviar el evento.

Cuando no se establece la clave de partición, los mensajes se envían en modo round-robin a todas las particiones disponibles

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

```

### <a name="implementing-a-custom-checkpointmanager-for-eventprocessorhost-eph"></a>Implementación del administrador de puntos de control personalizado para EventProcessorHost (EPH)

La API proporciona un mecanismo para implementar el administrador de puntos de control personalizado en escenarios donde la implementación predeterminada no es compatible con su caso de uso.

El administrador de puntos de control predeterminado usa el almacenamiento de blobs, pero si reemplaza el administrador de puntos de control que usa EPH por su propia implementación, puede usar cualquier almacén que quiera para respaldar la implementación del administrador de puntos de control.

Cree una clase que implemente la interfaz com.microsoft.azure.eventprocessorhost.ICheckpointManager.

Use la implementación personalizada del administrador de puntos de control (com.microsoft.azure.eventprocessorhost.ICheckpointManager).

Dentro de su implementación, puede reemplazar el mecanismo de puntos de control predeterminado e implementar nuestros propios puntos de control en función de su propio almacén de datos (SQL Server, CosmosDB y Azure Cache for Redis). Se recomienda que el almacén usado para respaldar la implementación del administrador de puntos de control sea accesible para todas las instancias de EPH que van a procesar eventos para el grupo de consumidores.

Puede usar cualquier almacén de datos que esté disponible en su entorno.

La clase com.microsoft.azure.eventprocessorhost.EventProcessorHost proporciona dos constructores que le permiten reemplazar el administrador de puntos de control por EventProcessorHost.


## <a name="next-steps"></a>Pasos siguientes
Lea los siguientes artículos: 

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Características y terminología de Azure Event Hubs](event-hubs-features.md)
- [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)

