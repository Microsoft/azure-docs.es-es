---
title: Uso de Azure Queue Storage para supervisar las notificaciones sobre trabajos de Media Services | Microsoft Docs
description: Descubra cómo usar el almacenamiento en cola de Azure para supervisar las notificaciones sobre trabajos de Media Services. El ejemplo de código está escrito en C# y utiliza el SDK de Media Services para .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: f535d0b5-f86c-465f-81c6-177f4f490987
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: c6c1dcb0af500c47aabbd8e8193c066bb4a921c5
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267775"
---
# <a name="use-azure-queue-storage-to-monitor-media-services-job-notifications-with-net"></a>Uso del almacenamiento en cola de Azure para supervisar las notificaciones sobre trabajos de Media Services con .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Al ejecutar trabajos de codificación, muchas veces se requiere una forma de hacer un seguimiento del progreso del trabajo. Puede configurar Media Services para entregar notificaciones a [Azure Queue Storage](../../storage/queues/storage-dotnet-how-to-use-queues.md). Puede supervisar el progreso del trabajo obteniendo notificaciones desde Queue Storage. 

Se puede obtener acceso a los mensajes entregados al almacenamiento de cola desde cualquier lugar del mundo. La arquitectura de mensajería de Queue Storage es fiable y altamente escalable. El sondeo de mensajes en Queue Storage es preferible a otros métodos.

Un escenario común para escuchar notificaciones de Media Services es si va a desarrollar un sistema de administración de contenido que necesita realizar alguna tarea adicional después de que se complete un trabajo de codificación (por ejemplo, para desencadenar el siguiente paso del flujo de trabajo o publicar contenido).

En este artículo se muestra cómo obtener mensajes de notificación de Queue Storage.  

## <a name="considerations"></a>Consideraciones
Tenga en cuenta lo siguiente al desarrollar aplicaciones de Media Services que usen Queue Storage:

* Queue Storage no ofrece ninguna garantía de entrega ordenada de tipo primero en entrar, primero en salir (FIFO). Para obtener más información, consulte [Colas de Azure y colas de Azure Service Bus: comparación y diferencias](/previous-versions/azure/hh767287(v=azure.100)).
* Queue Storage no es un servicio de inserción. Tiene que sondear la cola.
* Puede tener cualquier número de colas. Para obtener más información, consulte la [API de REST del servicio de cola](/rest/api/storageservices/queue-service-rest-api).
* Queue Storage tiene algunas limitaciones y particularidades que deben tenerse en cuenta. Estas se describen en [Colas de Azure y de Azure Service Bus: comparación y diferencias](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md).

## <a name="net-code-example"></a>Ejemplo de código .NET

El ejemplo de código de esta sección realiza lo siguiente:

1. Define la clase **EncodingJobMessage** que se asigna al formato de mensaje de notificación. El código deserializa los mensajes recibidos de la cola en objetos del tipo **EncodingJobMessage** .
2. Carga la información de cuenta de almacenamiento y Media Services del archivo app.config. El ejemplo de código usa esta información para crear los objetos **CloudMediaContext** y **CloudQueue**.
3. Crea la cola que va a recibir los mensajes de notificación sobre el trabajo de codificación.
4. Crea el extremo de notificación que se asigna a la cola.
5. Adjunta el extremo de notificación al trabajo y envía el trabajo de codificación. Puede tener varios extremos de notificación adjuntos a un trabajo.
6. Pasa **NotificationJobState.FinalStatesOnly** al método **AddNew** (en este ejemplo, solo nos interesan los estados finales del procesamiento del trabajo).

    ```csharp
    job.JobNotificationSubscriptions.AddNew(NotificationJobState.FinalStatesOnly, _notificationEndPoint);
    ```

7. Si se pasa **NotificationJobState.All**, se obtienen todas las notificaciones de cambio de estado siguientes: en cola, programado, procesando y finalizado. Sin embargo, como se indicó anteriormente, Queue Storage no garantiza la entrega ordenada. Para ordenar los mensajes, utilice la propiedad **Timestamp** (definida en el tipo **EncodingJobMessage** en el ejemplo siguiente). Los mensajes duplicados son posibles. Para comprobar si existen duplicados, utilice la **propiedad ETag** (definida en el tipo de **EncodingJobMessage**). También es posible que se omitan algunas notificaciones de cambio de estado.
8. Espera a que el trabajo llegue al estado Finalizado comprobando la cola cada 10 segundos. Elimina los mensajes una vez que se hayan procesado.
9. Elimina la cola y el extremo de notificación.

> [!NOTE]
> La manera recomendada de supervisar el estado de un trabajo es escuchar los mensajes de notificación, tal como se muestra en el ejemplo siguiente:
>
> Como alternativa, puede comprobar el estado de un trabajo mediante el uso de la propiedad **IJob.State** .  Es posible que llegue un mensaje de notificación sobre la finalización de un trabajo antes de que el estado en **IJob** se establezca en **Finalizado**. La propiedad **IJob.State** refleja el estado exacto con un ligero retraso.
>
>

### <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

1. Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 
2. Cree una carpeta nueva (la carpeta puede estar en cualquier lugar del disco duro) y copie un archivo .mp4 que quiere codificar para reproducirlo en streaming o descargarlo progresivamente. En este ejemplo, se usa la ruta de acceso "C:\Media".
3. Agregue una referencia a la biblioteca **System.Runtime.Serialization**.

### <a name="code"></a>Código

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using System.Collections.Generic;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Queue;
using System.Runtime.Serialization.Json;

namespace JobNotification
{
    public class EncodingJobMessage
    {
        // MessageVersion is used for version control.
        public String MessageVersion { get; set; }

        // Type of the event. Valid values are
        // JobStateChange and NotificationEndpointRegistration.
        public String EventType { get; set; }

        // ETag is used to help the customer detect if
        // the message is a duplicate of another message previously sent.
        public String ETag { get; set; }

        // Time of occurrence of the event.
        public String TimeStamp { get; set; }

        // Collection of values specific to the event.

        // For the JobStateChange event the values are:
        //     JobId - Id of the Job that triggered the notification.
        //     NewState- The new state of the Job. Valid values are:
        //          Scheduled, Processing, Canceling, Cancelled, Error, Finished
        //     OldState- The old state of the Job. Valid values are:
        //          Scheduled, Processing, Canceling, Cancelled, Error, Finished

        // For the NotificationEndpointRegistration event the values are:
        //     NotificationEndpointId- Id of the NotificationEndpoint
        //          that triggered the notification.
        //     State- The state of the Endpoint.
        //          Valid values are: Registered and Unregistered.

        public IDictionary<string, object> Properties { get; set; }
    }

    class Program
    {

        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static readonly string _StorageConnectionString = 
            ConfigurationManager.AppSettings["StorageConnectionString"];

        private static CloudMediaContext _context = null;
        private static CloudQueue _queue = null;
        private static INotificationEndPoint _notificationEndPoint = null;

        private static readonly string _singleInputMp4Path =
            Path.GetFullPath(@"C:\Media\BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            string endPointAddress = Guid.NewGuid().ToString();

            // Create the context.
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
            
            // Create the queue that will be receiving the notification messages.
            _queue = CreateQueue(_StorageConnectionString, endPointAddress);

            // Create the notification point that is mapped to the queue.
            _notificationEndPoint =
                    _context.NotificationEndPoints.Create(
                    Guid.NewGuid().ToString(), NotificationEndPointType.AzureQueue, endPointAddress);


            if (_notificationEndPoint != null)
            {
                IJob job = SubmitEncodingJobWithNotificationEndPoint(_singleInputMp4Path);
                WaitForJobToReachedFinishedState(job.Id);
            }

            // Clean up.
            _queue.Delete();
            _notificationEndPoint.Delete();
        }


        static public CloudQueue CreateQueue(string storageAccountConnectionString, string endPointAddress)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageAccountConnectionString);

            // Create the queue client
            CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

            // Retrieve a reference to a queue
            CloudQueue queue = queueClient.GetQueueReference(endPointAddress);

            // Create the queue if it doesn't already exist
            queue.CreateIfNotExists();

            return queue;
        }


        public static IJob SubmitEncodingJobWithNotificationEndPoint(string inputMediaFilePath)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("My MP4 to Smooth Streaming encoding job");

            //Create an encrypted asset and upload the mp4.
            IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.StorageEncrypted,
                inputMediaFilePath);

            // Get a media processor reference, and pass to it the name of the
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Create a task with the conversion details, using a configuration file.
            ITask task = job.Tasks.AddNew("My encoding Task",
                processor,
                "Adaptive Streaming",
                Microsoft.WindowsAzure.MediaServices.Client.TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            // Add a notification point to the job. You can add multiple notification points.  
            job.JobNotificationSubscriptions.AddNew(NotificationJobState.FinalStatesOnly,
                _notificationEndPoint);

            job.Submit();

            return job;
        }

        public static void WaitForJobToReachedFinishedState(string jobId)
        {
            int expectedState = (int)JobState.Finished;
            int timeOutInSeconds = 600;

            bool jobReachedExpectedState = false;
            DateTime startTime = DateTime.Now;
            int jobState = -1;

            while (!jobReachedExpectedState)
            {
                // Specify how often you want to get messages from the queue.
                Thread.Sleep(TimeSpan.FromSeconds(10));

                foreach (var message in _queue.GetMessages(10))
                {
                    using (Stream stream = new MemoryStream(message.AsBytes))
                    {
                        DataContractJsonSerializerSettings settings = new DataContractJsonSerializerSettings();
                        settings.UseSimpleDictionaryFormat = true;
                        DataContractJsonSerializer ser = new DataContractJsonSerializer(typeof(EncodingJobMessage), settings);
                        EncodingJobMessage encodingJobMsg = (EncodingJobMessage)ser.ReadObject(stream);

                        Console.WriteLine();

                        // Display the message information.
                        Console.WriteLine("EventType: {0}", encodingJobMsg.EventType);
                        Console.WriteLine("MessageVersion: {0}", encodingJobMsg.MessageVersion);
                        Console.WriteLine("ETag: {0}", encodingJobMsg.ETag);
                        Console.WriteLine("TimeStamp: {0}", encodingJobMsg.TimeStamp);
                        foreach (var property in encodingJobMsg.Properties)
                        {
                            Console.WriteLine("    {0}: {1}", property.Key, property.Value);
                        }

                        // We are only interested in messages
                        // where EventType is "JobStateChange".
                        if (encodingJobMsg.EventType == "JobStateChange")
                        {
                            string JobId = (String)encodingJobMsg.Properties.Where(j => j.Key == "JobId").FirstOrDefault().Value;
                            if (JobId == jobId)
                            {
                                string oldJobStateStr = (String)encodingJobMsg.Properties.
                                                            Where(j => j.Key == "OldState").FirstOrDefault().Value;
                                string newJobStateStr = (String)encodingJobMsg.Properties.
                                                            Where(j => j.Key == "NewState").FirstOrDefault().Value;

                                JobState oldJobState = (JobState)Enum.Parse(typeof(JobState), oldJobStateStr);
                                JobState newJobState = (JobState)Enum.Parse(typeof(JobState), newJobStateStr);

                                if (newJobState == (JobState)expectedState)
                                {
                                    Console.WriteLine("job with Id: {0} reached expected state: {1}",
                                        jobId, newJobState);
                                    jobReachedExpectedState = true;
                                    break;
                                }
                            }
                        }
                    }
                    // Delete the message after we've read it.
                    _queue.DeleteMessage(message);
                }

                // Wait until timeout
                TimeSpan timeDiff = DateTime.Now - startTime;
                bool timedOut = (timeDiff.TotalSeconds > timeOutInSeconds);
                if (timedOut)
                {
                    Console.WriteLine(@"Timeout for checking job notification messages,
                                        latest found state ='{0}', wait time = {1} secs",
                        jobState,
                        timeDiff.TotalSeconds);

                    throw new TimeoutException();
                }
            }
        }

        static private IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
        {
            var asset = _context.Assets.Create("UploadSingleFile_" + DateTime.UtcNow.ToString(),
                assetCreationOptions);

            var fileName = Path.GetFileName(singleFilePath);

            var assetFile = asset.AssetFiles.Create(fileName);

            Console.WriteLine("Created assetFile {0}", assetFile.Name);
            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading of {0}", assetFile.Name);

            return asset;
        }

        static private IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
                ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

            return processor;
        }
    }
}
```

El ejemplo anterior genera el siguiente resultado: Los valores variarán.

```output
Created assetFile BigBuckBunny.mp4
Upload BigBuckBunny.mp4
Done uploading of BigBuckBunny.mp4

EventType: NotificationEndPointRegistration
MessageVersion: 1.0
ETag: e0238957a9b25bdf3351a88e57978d6a81a84527fad03bc23861dbe28ab293f6
TimeStamp: 2013-05-14T20:22:37
    NotificationEndPointId: nb:nepid:UUID:d6af9412-2488-45b2-ba1f-6e0ade6dbc27
    State: Registered
    Name: dde957b2-006e-41f2-9869-a978870ac620
    Created: 2013-05-14T20:22:35

EventType: JobStateChange
MessageVersion: 1.0
ETag: 4e381f37c2d844bde06ace650310284d6928b1e50101d82d1b56220cfcb6076c
TimeStamp: 2013-05-14T20:24:40
    JobId: nb:jid:UUID:526291de-f166-be47-b62a-11ffe6d4be54
    JobName: My MP4 to Smooth Streaming encoding job
    NewState: Finished
    OldState: Processing
    AccountName: westeuropewamsaccount
job with Id: nb:jid:UUID:526291de-f166-be47-b62a-11ffe6d4be54 reached expected
State: Finished
```

## <a name="next-step"></a>Paso siguiente
Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
