---
title: Envío y recepción de eventos desde Azure Event Hubs mediante .NET (versión anterior)
description: En este artículo se proporciona un tutorial para crear una aplicación de .NET Core que envíe o reciba eventos hacia y desde Azure Event Hubs mediante el paquete Microsoft.Azure.EventHubs anterior.
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: d7d33ebcabb728cf2fbf43b393de5eec6cb58af3
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89010705"
---
# <a name="send-events-to-or-receive-events-from-azure-event-hubs-using-net-core-microsoftazureeventhubs"></a>Envío o recepción de eventos desde Azure Event Hubs mediante .NET Core (Microsoft.Azure.EventHubs)
En este inicio rápido se explica cómo enviar eventos a un centro de eventos o recibirlos de este mediante la biblioteca de .NET Core **Microsoft.Azure.EventHubs**.

> [!WARNING]
> En este inicio rápido se usa el paquete **Microsoft.Azure.EventHubs** anterior. Para obtener un inicio rápido que use la biblioteca **Azure.Messaging.EventHubs** más reciente, consulte [Envío y recepción de eventos mediante la biblioteca Azure.Messaging.EventHubs](event-hubs-dotnet-standard-getstarted-send.md). Para que la aplicación use la biblioteca nueva en lugar de la anterior, consulte la [Guía para migrar desde Microsoft.Azure.EventHubs a Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).

## <a name="prerequisites"></a>Requisitos previos
Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido. 

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- [Microsoft Visual Studio 2019](https://www.visualstudio.com).
- [Herramientas de .NET Core Visual Studio 2015 o 2017](https://www.microsoft.com/net/core). 
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). A continuación, obtenga la **cadena de conexión para el espacio de nombres del centro de eventos** siguiendo las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). La utilizará más adelante en este inicio rápido.

## <a name="send-events"></a>Envío de eventos 
En esta sección se muestra cómo crear una aplicación de consola de .NET Core para enviar eventos a un centro de eventos. 

> [!NOTE]
> Puede descargar esta guía de inicio rápido como un ejemplo desde [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleSender), reemplazar las cadenas `EventHubConnectionString` y `EventHubName` por los valores del centro de eventos, y ejecutarlo. También puede seguir los pasos de este inicio rápido para crear la suya propia.


### <a name="create-a-console-application"></a>Creación de una aplicación de consola

Inicie Visual Studio. En el menú **Archivo**, haga clic en **Nuevo** y después haga clic en **Proyecto**. Cree una aplicación de consola de .NET Core.

![Nuevo proyecto](./media/event-hubs-dotnet-standard-getstarted-send/netcoresnd.png)

### <a name="add-the-event-hubs-nuget-package"></a>Incorporación del paquete NuGet de Event Hubs

Agregue el paquete NuGet de la biblioteca .NET Core [`Microsoft.Azure.EventHubs`](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) a su proyecto siguiendo estos pasos: 

1. Haga clic con el botón derecho en el proyecto recién creado y seleccione **Administrar paquetes NuGet**.
2. Haga clic en la pestaña **Examinar** y, después, busque "Microsoft.Azure.EventHubs" y seleccione el paquete **Microsoft.Azure.EventHubs**. Haga clic en **Instalar** para completar la instalación y, a continuación, cierre este cuadro de diálogo.

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Escritura de código para enviar mensajes al centro de eventos

1. Agregue las siguientes instrucciones `using` al principio del archivo Program.cs:

    ```csharp
    using Microsoft.Azure.EventHubs;
    using System.Text;
    using System.Threading.Tasks;
    ```

2. Agregue constantes a la clase `Program` para la cadena de conexión de Event Hubs y la ruta de la entidad (nombre de centro de eventos individual). Reemplace los marcadores de posición entre llaves por los valores adecuados obtenidos al crear el centro de eventos. Asegúrese de que `{Event Hubs connection string}` es la cadena de conexión en el nivel de espacio de nombres y no la cadena de Event Hub. 

    ```csharp
    private static EventHubClient eventHubClient;
    private const string EventHubConnectionString = "{Event Hubs connection string}";
    private const string EventHubName = "{Event Hub path/name}";
    ```

3. Agregue un nuevo método denominado `MainAsync` a la clase `Program`, de esta manera:

    ```csharp
    private static async Task MainAsync(string[] args)
    {
        // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
        // Typically, the connection string should have the entity path in it, but this simple scenario
        // uses the connection string from the namespace.
        var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
        {
            EntityPath = EventHubName
        };

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

        await SendMessagesToEventHub(100);

        await eventHubClient.CloseAsync();

        Console.WriteLine("Press ENTER to exit.");
        Console.ReadLine();
    }
    ```

4. Agregue un nuevo método denominado `SendMessagesToEventHub` a la clase `Program`, de esta manera:

    ```csharp
    // Uses the event hub client to send 100 messages to the event hub.
    private static async Task SendMessagesToEventHub(int numMessagesToSend)
    {
        for (var i = 0; i < numMessagesToSend; i++)
        {
            try
            {
                var message = $"Message {i}";
                Console.WriteLine($"Sending message: {message}");
                await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
            }
            catch (Exception exception)
            {
                Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
            }

            await Task.Delay(10);
        }

        Console.WriteLine($"{numMessagesToSend} messages sent.");
    }
    ```

5. Agregue el siguiente código al método `Main` de la clase `Program`:

    ```csharp
    MainAsync(args).GetAwaiter().GetResult();
    ```

   Este es el aspecto que debería tener Program.cs.

    ```csharp
    namespace SampleSender
    {
        using System;
        using System.Text;
        using System.Threading.Tasks;
        using Microsoft.Azure.EventHubs;

        public class Program
        {
            private static EventHubClient eventHubClient;
            private const string EventHubConnectionString = "{Event Hubs connection string}";
            private const string EventHubName = "{Event Hub path/name}";

            public static void Main(string[] args)
            {
                MainAsync(args).GetAwaiter().GetResult();
            }

            private static async Task MainAsync(string[] args)
            {
                // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
                // Typically, the connection string should have the entity path in it, but for the sake of this simple scenario
                // we are using the connection string from the namespace.
                var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
                {
                    EntityPath = EventHubName
                };

                eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

                await SendMessagesToEventHub(100);

                await eventHubClient.CloseAsync();

                Console.WriteLine("Press ENTER to exit.");
                Console.ReadLine();
            }

            // Uses the event hub client to send 100 messages to the event hub.
            private static async Task SendMessagesToEventHub(int numMessagesToSend)
            {
                for (var i = 0; i < numMessagesToSend; i++)
                {
                    try
                    {
                        var message = $"Message {i}";
                        Console.WriteLine($"Sending message: {message}");
                        await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
                    }
                    catch (Exception exception)
                    {
                        Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
                    }

                    await Task.Delay(10);
                }

                Console.WriteLine($"{numMessagesToSend} messages sent.");
            }
        }
    }
    ```

6. Ejecute el programa y asegúrese de que no hay ningún error.

## <a name="receive-events"></a>Recepción de eventos
En esta sección se muestra cómo escribir una aplicación de consola de .NET Core que reciba mensajes de un centro de eventos mediante el [host de procesador de eventos](event-hubs-event-processor-host.md). El [host de procesador de eventos](event-hubs-event-processor-host.md) es una clase de .NET que simplifica la recepción de eventos desde Event Hubs mediante la administración de puntos de control persistentes y recepciones paralelas desde dichas instancias de Event Hubs. Mediante el host de procesador de eventos, puede dividir eventos entre varios receptores, aunque estén hospedados en distintos nodos. En este ejemplo se muestra cómo usar el host de procesador de eventos en un solo receptor.
> [!NOTE]
> Puede descargar esta guía de inicio rápido como un ejemplo desde [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver), reemplazar las cadenas `EventHubConnectionString` y `EventHubName`, `StorageAccountName`, `StorageAccountKey` y `StorageContainerName` con los valores del centro de eventos y ejecutarlo. También puede seguir los pasos de este tutorial para crear el suyo propio.

[!INCLUDE [event-hubs-create-storage](../../includes/event-hubs-create-storage.md)]

### <a name="create-a-console-application"></a>Creación de una aplicación de consola

Inicie Visual Studio. En el menú **Archivo**, haga clic en **Nuevo** y después haga clic en **Proyecto**. Cree una aplicación de consola de .NET Core.

![Nuevo proyecto para recibir](./media/event-hubs-dotnet-standard-getstarted-receive-eph/netcorercv.png)

### <a name="add-the-event-hubs-nuget-package"></a>Incorporación del paquete NuGet de Event Hubs

Agregue los paquetes NuGet de la biblioteca de .NET Standard [**Microsoft.Azure.EventHubs**](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) y [**Microsoft.Azure.EventHubs.Processor**](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/) al proyecto mediante estos pasos: 

1. Haga clic con el botón derecho en el proyecto recién creado y seleccione **Administrar paquetes NuGet**.
2. Haga clic en la pestaña **Examinar**, busque **Microsoft.Azure.EventHubs** y seleccione el paquete **Microsoft.Azure.EventHubs**. Haga clic en **Instalar** para completar la instalación y, a continuación, cierre este cuadro de diálogo.
3. Repita los pasos 1 y 2 e instale el paquete **Microsoft.Azure.EventHubs.Processor**.

### <a name="implement-the-ieventprocessor-interface"></a>Implementación de la interfaz de IEventProcessor

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto, haga clic en **Agregar** y después en **Clase**. Asigne el nombre **SimpleEventProcessor** a la nueva clase.

2. Abra el archivo SimpleEventProcessor.cs y agregue las siguientes instrucciones `using` en la parte superior del archivo.

    ```csharp
    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.EventHubs.Processor;
    using System.Threading.Tasks;
    ```

3. Implemente la interfaz `IEventProcessor`. Reemplace todo el contenido de la clase `SimpleEventProcessor` con el código siguiente:

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

### <a name="update-the-main-method-to-use-simpleeventprocessor"></a>Actualización del método Main para usar SimpleEventProcessor

1. Agregue las siguientes instrucciones `using` al principio del archivo Program.cs.

    ```csharp
    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.EventHubs.Processor;
    using System.Threading.Tasks;
    ```

2. Agregue constantes a la clase `Program` para la cadena de conexión del centro de eventos, el nombre del centro de eventos, el nombre del contenedor de la cuenta de almacenamiento, y el nombre de la cuenta de almacenamiento y la clave de la cuenta de almacenamiento. Agregue el código siguiente, reemplazando los marcadores de posición con sus valores correspondientes:

    ```csharp
    private const string EventHubConnectionString = "{Event Hubs connection string}";
    private const string EventHubName = "{Event Hub path/name}";
    private const string StorageContainerName = "{Storage account container name}";
    private const string StorageAccountName = "{Storage account name}";
    private const string StorageAccountKey = "{Storage account key}";

    private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);
    ```   

3. Agregue un nuevo método denominado `MainAsync` a la clase `Program`, de esta manera:

    ```csharp
    private static async Task MainAsync(string[] args)
    {
        Console.WriteLine("Registering EventProcessor...");

        var eventProcessorHost = new EventProcessorHost(
            EventHubName,
            PartitionReceiver.DefaultConsumerGroupName,
            EventHubConnectionString,
            StorageConnectionString,
            StorageContainerName);

        // Registers the Event Processor Host and starts receiving messages
        await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

        Console.WriteLine("Receiving. Press ENTER to stop worker.");
        Console.ReadLine();

        // Disposes of the Event Processor Host
        await eventProcessorHost.UnregisterEventProcessorAsync();
    }
    ```

3. Agregue la siguiente línea de código al método `Main`:

    ```csharp
    MainAsync(args).GetAwaiter().GetResult();
    ```

    Este es el aspecto que debería tener el archivo Program.cs:

    ```csharp
    namespace SampleEphReceiver
    {

        public class Program
        {
            private const string EventHubConnectionString = "{Event Hubs connection string}";
            private const string EventHubName = "{Event Hub path/name}";
            private const string StorageContainerName = "{Storage account container name}";
            private const string StorageAccountName = "{Storage account name}";
            private const string StorageAccountKey = "{Storage account key}";

            private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);

            public static void Main(string[] args)
            {
                MainAsync(args).GetAwaiter().GetResult();
            }

            private static async Task MainAsync(string[] args)
            {
                Console.WriteLine("Registering EventProcessor...");

                var eventProcessorHost = new EventProcessorHost(
                    EventHubName,
                    PartitionReceiver.DefaultConsumerGroupName,
                    EventHubConnectionString,
                    StorageConnectionString,
                    StorageContainerName);

                // Registers the Event Processor Host and starts receiving messages
                await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

                Console.WriteLine("Receiving. Press ENTER to stop worker.");
                Console.ReadLine();

                // Disposes of the Event Processor Host
                await eventProcessorHost.UnregisterEventProcessorAsync();
            }
        }
    }
    ```

4. Ejecute el programa y asegúrese de que no hay ningún error.


## <a name="next-steps"></a>Pasos siguientes
Lea los siguientes artículos:

- [Control de acceso basado en rol (RBAC): ejemplos](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    En estos ejemplos se usa la biblioteca anterior **Microsoft.Azure.EventHubs**, pero se puede actualizar fácilmente para usar la biblioteca **Azure.Messaging.EventHubs** más reciente. Para que los ejemplos usen la biblioteca nueva en lugar de la anterior, consulte la [Guía para migrar de Microsoft.Azure.EventHubs a Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Características y terminología de Azure Event Hubs](event-hubs-features.md)
- [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)


