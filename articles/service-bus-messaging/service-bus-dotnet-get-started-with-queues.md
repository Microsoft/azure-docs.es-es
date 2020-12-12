---
title: Introducción a las colas de Azure Service Bus (Azure.Messaging.ServiceBus)
description: En este tutorial, creará una aplicación de consola de C# de .NET Core para enviar mensajes a una cola de Service Bus y recibir mensajes desde ella.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 11/13/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 15e5d257259bb4dfc98528cb726dbd2cc1f9a903
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498734"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-net"></a>Envío y recepción de mensajes con destino y origen en colas de Azure Service Bus (.NET)
En este tutorial, creará una aplicación de consola de .NET Core para enviar mensajes a una cola de Service Bus y recibir mensajes desde ella mediante el paquete **Azure.Messaging.ServiceBus**. 

> [!Important]
> Este inicio rápido usa el nuevo paquete Azure.Messaging.ServiceBus. Para ver un inicio rápido que use el paquete Microsoft.Azure.ServiceBus anterior, consulte [Envío y recepción de eventos mediante el paquete Microsoft.Azure.ServiceBus](service-bus-dotnet-get-started-with-queues-legacy.md).

## <a name="prerequisites"></a>Requisitos previos

- [Visual Studio 2019](https://www.visualstudio.com/vs)
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene una cola con la que trabajar, siga los pasos del artículo [Uso de Azure Portal para crear una cola de Service Bus](service-bus-quickstart-portal.md) para crear una. Anote la **cadena de conexión** para el espacio de nombres de Service Bus y el nombre de la **cola** que creó.

## <a name="send-messages-to-a-queue"></a>mensajes a una cola
En este inicio rápido, creará una aplicación de consola de C# de .NET Core para enviar mensajes a la cola.

### <a name="create-a-console-application"></a>Creación de una aplicación de consola
Inicie Visual Studio y cree otro proyecto **Aplicación de consola (.NET Core)** para C#. 

### <a name="add-the-service-bus-nuget-package"></a>Agregar el paquete NuGet de Service Bus

1. Haga clic con el botón derecho en el proyecto recién creado y seleccione **Administrar paquetes NuGet**.
1. Haga clic en **Examinar**. Busque y seleccione **[Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/)** .
1. Seleccione **Instalar** para completar la instalación y luego cierre el administrador de paquetes NuGet.

### <a name="add-code-to-send-messages-to-the-queue"></a>Adición de código para enviar mensajes a la cola

1. En *Program.cs*, agregue las siguientes instrucciones `using` en la parte superior de la definición del espacio de nombres, antes de la declaración de clase:

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    using Azure.Messaging.ServiceBus;
    ```

1. En la clase `Program`, declare las variables siguientes:

    ```csharp
        static string connectionString = "<NAMESPACE CONNECTION STRING>";
        static string queueName = "<QUEUE NAME>";
    ```

    Escriba la cadena de conexión para el espacio de nombres como la variable `ServiceBusConnectionString`. Escriba el nombre de la cola.

1. Reemplace el método `Main()` por el método `Main`**asincrónico**. Llama al método `SendMessagesAsync()` que se agregará en el paso siguiente para enviar mensajes a la cola. 

    ```csharp
    public static async Task Main(string[] args)
    {    
        const int numberOfMessages = 10;
        
        Console.WriteLine("======================================================");
        Console.WriteLine("Press ENTER key to exit after sending all the messages.");
        Console.WriteLine("======================================================");

        // Send messages.
        await SendMessagesAsync(numberOfMessages);

        Console.ReadKey();
    }
    ```
1. Inmediatamente después del método `Main()`, agregue el método `SendMessagesAsync()` siguiente, que realiza el trabajo de enviar el número de mensajes especificado por `numberOfMessagesToSend` (actualmente se establece en 10):

    ```csharp
        static async Task SendMessageAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the queue 
                ServiceBusSender sender = client.CreateSender(queueName);

                // create a message that we can send
                ServiceBusMessage message = new ServiceBusMessage("Hello world!");

                // send the message
                await sender.SendMessageAsync(message);
                Console.WriteLine($"Sent a single message to the queue: {queueName}");
            }
        }
    ```
1. Agregue un método denominado `CreateMessages` para crear una cola (cola de .NET) de mensajes a la clase `Program`. Normalmente, estos mensajes se obtienen de distintas partes de la aplicación. En este caso crearemos una cola de mensajes de ejemplo.

    ```csharp
        static Queue<ServiceBusMessage> CreateMessages()
        {
            // create a queue containing the messages and return it to the caller
            Queue<ServiceBusMessage> messages = new Queue<ServiceBusMessage>();
            messages.Enqueue(new ServiceBusMessage("First message"));
            messages.Enqueue(new ServiceBusMessage("Second message"));
            messages.Enqueue(new ServiceBusMessage("Third message"));
            return messages;
        }
    ```
1. Agregue un método denominado `SendMessageBatchAsync` a la clase `Program` y agréguele el código siguiente. Este método toma una cola de mensajes y prepara uno o varios lotes para enviarlos a la cola de Service Bus. 

    ```csharp
        static async Task SendMessageBatchAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the queue 
                ServiceBusSender sender = client.CreateSender(queueName);

                // get the messages to be sent to the Service Bus queue
                Queue<ServiceBusMessage> messages = CreateMessages();

                // total number of messages to be sent to the Service Bus queue
                int messageCount = messages.Count;

                // while all messages are not sent to the Service Bus queue
                while (messages.Count > 0)
                {
                    // start a new batch 
                    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

                    // add the first message to the batch
                    if (messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue once the message is added to the batch
                        messages.Dequeue();
                    }
                    else
                    {
                        // if the first message can't fit, then it is too large for the batch
                        throw new Exception($"Message {messageCount - messages.Count} is too large and cannot be sent.");
                    }

                    // add as many messages as possible to the current batch
                    while (messages.Count > 0 && messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue as it has been added to the batch
                        messages.Dequeue();
                    }
        
                    // now, send the batch
                    await sender.SendMessagesAsync(messageBatch);
        
                    // if there are any remaining messages in the .NET queue, the while loop repeats 
                }

                Console.WriteLine($"Sent a batch of {messageCount} messages to the queue: {queueName}");
            }
        }
    ```
1. Reemplace el método `Main()` por el método `Main`**asincrónico**. Llama a los métodos de envío para enviar un único mensaje y un lote de mensajes a la cola. 

    ```csharp
        static async Task Main()
        {
            // send a message to the queue
            await SendMessageAsync();

            // send a batch of messages to the queue
            await SendMessageBatchAsync();
        }
    ```
5. Ejecute la aplicación. Debería ver los mensajes siguientes. 

    ```console
    Sent a single message to the queue: myqueue
    Sent a batch of messages to the queue: myqueue
    ```       
1. En Azure Portal, haga lo siguiente:
    1. Vaya al espacio de nombres de Service Bus. 
    1. En la página **Información general**, seleccione la cola en el panel inferior central. 
    1. Observe los valores en la sección **Información esencial**.

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png" alt-text="Mensajes recibidos con el número y el tamaño" lightbox="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png":::

    Observe los valores siguientes:
    - El valor de **Recuento de mensajes activos** es ahora **4**. Cada vez que se ejecuta la aplicación de remitente sin recuperar los mensajes, este valor aumenta en 4.
    - El tamaño actual de la cola aumenta el valor de **ACTUAL** en la ventana **Essentials** cada vez que la aplicación agrega mensajes a la cola.
    - En el gráfico **Mensajes** de la sección **Métricas** inferior, puede ver que hay cuatro mensajes entrantes para la cola. 

## <a name="receive-messages-from-a-queue"></a>mensajes de una cola
En esta sección agregará código para recuperar mensajes de la cola.

1. Agregue los métodos siguientes a la clase `Program` que controlan los mensajes y los errores. 

    ```csharp
        // handle received messages
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        // handle any errors when receiving messages
        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Agregue un método denominado `ReceiveMessagesAsync` a la clase `Program` y agréguele el código siguiente para recibir mensajes. 

    ```csharp
        static async Task ReceiveMessagesAsync()
        {
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a processor that we can use to process the messages
                ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());

                // add handler to process messages
                processor.ProcessMessageAsync += MessageHandler;

                // add handler to process any errors
                processor.ProcessErrorAsync += ErrorHandler;

                // start processing 
                await processor.StartProcessingAsync();

                Console.WriteLine("Wait for a minute and then press any key to end the processing");
                Console.ReadKey();

                // stop processing 
                Console.WriteLine("\nStopping the receiver...");
                await processor.StopProcessingAsync();
                Console.WriteLine("Stopped receiving messages");
            }
        }
    ```
1. Agregue una llamada al método `ReceiveMessagesAsync` desde el método `Main`. Convierta en comentario el método `SendMessagesAsync` si solo desea probar la recepción de mensajes. Si no lo hace, verá otros cuatro mensajes enviados a la cola. 

    ```csharp
        static async Task Main()
        {
            // send a message to the queue
            await SendMessageAsync();

            // send a batch of messages to the queue
            await SendMessageBatchAsync();

            // receive message from the queue
            await ReceiveMessagesAsync();
        }
    ```

## <a name="run-the-app"></a>Ejecutar la aplicación
Ejecute la aplicación. Espere un minuto y, a continuación, presione cualquier tecla para dejar de recibir mensajes. Debería ver el resultado siguiente (barra espaciadora para la tecla). 

```console
Sent a single message to the queue: myqueue
Sent a batch of messages to the queue: myqueue
Wait for a minute and then press any key to end the processing
Received: Hello world!
Received: First message in the batch
Received: Second message in the batch
Received: Third message in the batch
Received: Hello world!
Received: First message in the batch
Received: Second message in the batch
Received: Third message in the batch

Stopping the receiver...
Stopped receiving messages
```

Vuelva a consultar el portal. 

- Los valores de **Recuento de mensajes activos** y **ACTUAL** ahora son **0**.
- En el gráfico **Mensajes** de la sección **Métricas** inferior, puede ver que hay ocho mensajes entrantes y otros ocho salientes para la cola. 

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png" alt-text="Mensajes activos y tamaño después de la recepción" lightbox="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png":::

## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes:

- [Biblioteca cliente de Azure Service Bus para .NET: léame](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Ejemplos en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Referencia de API de .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true&view=azure-dotnet-preview)
