---
title: 'Enviar y recibir eventos mediante .NET Framework: Azure Event Hubs | Microsoft Docs'
description: En este artículo se ofrece un tutorial para crear una aplicación de .NET Framework que envía eventos a Azure Event Hubs.
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: c4974bd3-2a79-48a1-aa3b-8ee2d6655b28
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.custom: seodec18
ms.date: 04/15/2019
ms.author: shvija
ms.openlocfilehash: 96ce71a7b3076adec169f103060a167b61c42d5c
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65603513"
---
# <a name="send-events-to-or-receive-events-from-azure-event-hubs-using-net-framework"></a>Enviar eventos a o recibir eventos desde Azure Event Hubs mediante .NET Framework
Azure Event Hubs es una plataforma de streaming de macrodatos y servicio de ingesta de eventos de gran escalabilidad capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. Los datos enviados a un centro de eventos se pueden transformar y almacenar con cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento. Para más información sobre Event Hubs, consulte [Introducción a Event Hubs](event-hubs-about.md) y [Características de Event Hubs](event-hubs-features.md).

En este tutorial se muestra cómo crear aplicaciones de consola de .NET Framework en C# para enviar eventos a o recibir eventos desde un centro de eventos. 

## <a name="prerequisites"></a>Requisitos previos
Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- [Microsoft Visual Studio 2019](https://visualstudio.com).
- **Crear un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). A continuación, obtenga el **cadena de conexión para el espacio de nombres del centro de eventos** siguiendo las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Utilizará la cadena de conexión más adelante en el tutorial.

## <a name="send-events"></a>Envío de eventos 
En esta sección se muestra cómo crear una aplicación de consola de .NET Framework para enviar eventos a un centro de eventos. 

### <a name="create-a-console-application"></a>Creación de una aplicación de consola

En Visual Studio, cree un nuevo proyecto de aplicación de escritorio de Visual C# con la plantilla de proyecto **Aplicación de consola** . Asigne al proyecto el nombre **Remitente**.
   
![Creación de una aplicación de consola](./media/event-hubs-dotnet-framework-getstarted-send/create-sender-csharp1.png)

### <a name="add-the-event-hubs-nuget-package"></a>Incorporación del paquete NuGet de Event Hubs

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **Remitente** y luego haga clic en **Administrar paquetes NuGet para la solución**. 
2. Haga clic en la pestaña **Examinar** y luego busque `WindowsAzure.ServiceBus`. Haga clic en **Instalar**y acepte las condiciones de uso. 
   
    ![Instalación del paquete NuGet de Service Bus](./media/event-hubs-dotnet-framework-getstarted-send/create-sender-csharp2.png)
   
    Visual Studio descarga, instala y agrega una referencia al [paquete NuGet de la biblioteca de Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus).

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Escritura de código para enviar mensajes al centro de eventos

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :
   
    ```csharp
    using System.Threading;
    using Microsoft.ServiceBus.Messaging;
    ```
2. Agregue los siguientes campos a la clase **Program**; para ello, sustituya los valores del marcador de posición por el nombre del centro de eventos creado en la sección anterior y la cadena de conexión de nivel del espacio de nombres que ha guardado anteriormente. Puede copiar la cadena de conexión para el centro de eventos desde la clave **Connection string-primary** en **RootManageSharedAccessKey** en la página del centro de eventos en Azure Portal. Para ver los pasos detallados, consulte [Obtención de la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal).
   
    ```csharp
    static string eventHubName = "Your Event Hub name";
    static string connectionString = "namespace connection string";
    ```
3. Agregue el método siguiente a la clase **Program** :
   
      ```csharp
      static void SendingRandomMessages()
      {
          var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
          while (true)
          {
              try
              {
                  var message = Guid.NewGuid().ToString();
                  Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, message);
                  eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes(message)));
              }
              catch (Exception exception)
              {
                  Console.ForegroundColor = ConsoleColor.Red;
                  Console.WriteLine("{0} > Exception: {1}", DateTime.Now, exception.Message);
                  Console.ResetColor();
              }
       
              Thread.Sleep(200);
          }
      }
      ```
   
      Este método envía continuamente los eventos al centro de eventos con un retraso de 200 ms.
4. Por último, agregue las líneas siguientes al método **Main** :
   
      ```csharp
      Console.WriteLine("Press Ctrl-C to stop the sender process");
      Console.WriteLine("Press Enter to start now");
      Console.ReadLine();
      SendingRandomMessages();
      ```
5. Ejecute el programa y asegúrese de que no hay ningún error.
  
## <a name="receive-events"></a>Recepción de eventos
En esta sección, se escribirá una aplicación de consola de .NET Framework que recibe mensajes de un centro de eventos mediante el [Event Processor Host](event-hubs-event-processor-host.md). El [host de procesador de eventos](event-hubs-event-processor-host.md) es una clase de .NET que simplifica la recepción de eventos desde Event Hubs mediante la administración de puntos de control persistentes y recepciones paralelas desde dichas instancias de Event Hubs. Mediante el host de procesador de eventos, puede dividir eventos entre varios receptores, aunque estén hospedados en distintos nodos. 

[!INCLUDE [event-hubs-create-storage](../../includes/event-hubs-create-storage.md)]

### <a name="create-a-console-application"></a>Creación de una aplicación de consola

En Visual Studio, cree un nuevo proyecto de aplicación de escritorio de Visual C# con la plantilla de proyecto **Aplicación de consola**. Asigne al proyecto el nombre **Receptor**.
   
![Creación de una aplicación de consola](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-receiver-csharp1.png)

### <a name="add-the-event-hubs-nuget-package"></a>Incorporación del paquete NuGet de Event Hubs

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **Receptor** y luego haga clic en **Administrar paquetes NuGet para la solución**.
2. Haga clic en la pestaña **Examinar** y luego busque `Microsoft Azure Service Bus Event Hub - EventProcessorHost`. Haga clic en **Instalar**y acepte las condiciones de uso.
   
    ![Búsqueda del paquete NuGet del host del procesador de eventos](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-eph-csharp1.png)
   
    Visual Studio descarga, instala y agrega una referencia al [paquete de NuGet de Azure Service Bus - EventProcessorHost](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost), con todas sus dependencias.

### <a name="implement-the-ieventprocessor-interface"></a>Implementación de la interfaz de IEventProcessor

1. Haga clic con el botón derecho en el proyecto **Receptor**, haga clic en **Agregar** y, después, haga clic en **Clase**. Asigne a la nueva clase el nombre **SimpleEventProcessor** y después haga clic en **Agregar** para crear la clase.
   
    ![Incorporación de la clase SimpleEventProcessor](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-receiver-csharp2.png)
2. Agregue las siguientes instrucciones en la parte superior del archivo SimpleEventProcessor.cs:
    
      ```csharp
      using Microsoft.ServiceBus.Messaging;
      using System.Diagnostics;
      ```
    
3. Sustituya el código siguiente por el cuerpo de la clase:
    
      ```csharp
      class SimpleEventProcessor : IEventProcessor
      {
        Stopwatch checkpointStopWatch;
        
        async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
        {
            Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
            if (reason == CloseReason.Shutdown)
            {
                await context.CheckpointAsync();
            }
        }
        
        Task IEventProcessor.OpenAsync(PartitionContext context)
        {
            Console.WriteLine("SimpleEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
            this.checkpointStopWatch = new Stopwatch();
            this.checkpointStopWatch.Start();
            return Task.FromResult<object>(null);
        }
        
        async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
        {
            foreach (EventData eventData in messages)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
        
                Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
                    context.Lease.PartitionId, data));
            }
        
            //Call checkpoint every 5 minutes, so that worker can resume processing from 5 minutes back if it restarts.
            if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
            {
                await context.CheckpointAsync();
                this.checkpointStopWatch.Restart();
            }
        }
      }
      ```
    
      **EventProcessorHost** llama a esta clase para procesar los eventos recibidos del centro de eventos. La clase `SimpleEventProcessor` usa un cronómetro para llamar periódicamente al método de punto de control en el contexto **EventProcessorHost**. Este procesamiento garantiza que, si se reinicia el destinatario, no se pierden más de cinco minutos de trabajo de procesamiento.

### <a name="update-the-main-method-to-use-simpleeventprocessor"></a>Actualización del método Main para usar SimpleEventProcessor

1. En la clase **Program.cs**, agregue la siguiente instrucción `using` al principio del archivo:
    
      ```csharp
      using Microsoft.ServiceBus.Messaging;
      ```
    
2. Reemplace el método `Main` de la clase `Program` por el código siguiente, y sustituya el nombre del centro de eventos y la cadena de conexión de nivel del espacio de nombres que ha guardado anteriormente, y la cuenta de almacenamiento y la clave que ha copiado en las secciones anteriores. 
    
      ```csharp
      static void Main(string[] args)
      {
        string eventHubConnectionString = "{Event Hubs namespace connection string}";
        string eventHubName = "{Event Hub name}";
        string storageAccountName = "{storage account name}";
        string storageAccountKey = "{storage account key}";
        string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", storageAccountName, storageAccountKey);
        
        string eventProcessorHostName = Guid.NewGuid().ToString();
        EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
        Console.WriteLine("Registering EventProcessor...");
        var options = new EventProcessorOptions();
        options.ExceptionReceived += (sender, e) => { Console.WriteLine(e.Exception); };
        eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>(options).Wait();
        
        Console.WriteLine("Receiving. Press enter key to stop worker.");
        Console.ReadLine();
        eventProcessorHost.UnregisterEventProcessorAsync().Wait();
      }
      ```
    
3. Ejecute el programa y asegúrese de que no hay ningún error.
  
## <a name="next-steps"></a>Pasos siguientes
Lea los siguientes artículos: 

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Características y la terminología de Azure Event Hubs](event-hubs-features.md).
- [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)


<!-- Links -->
[EventProcessorHost]: /dotnet/api/microsoft.servicebus.messaging.eventprocessorhost
[Event Hubs overview]: event-hubs-about.md
[Event Hubs Programming Guide]: event-hubs-programming-guide.md
[Azure Storage account]:../storage/common/storage-create-storage-account.md
[Event Processor Host]: event-hubs-event-processor-host.md
[Azure portal]: https://portal.azure.com
