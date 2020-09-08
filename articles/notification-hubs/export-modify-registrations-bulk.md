---
title: Exportación e importación de registros de Azure Notification Hubs en masa | Microsoft Docs
description: Aprenda a usar el soporte técnico masivo de Notification Hubs para realizar un gran número de operaciones en un centro de notificaciones o para exportar todos los registros.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/18/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: c0771864229c8a3918da076de48fb6e033d2cf5a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018185"
---
# <a name="export-and-import-azure-notification-hubs-registrations-in-bulk"></a>Exportación e importación de registros de Azure Notification Hubs en masa

Existen escenarios en los que es necesario crear o modificar gran cantidad de registros en un centro de notificaciones. Algunos de estos escenarios son actualizaciones de etiquetas a continuación de los cálculos por lotes, o la migración de una implementación de inserción existente para usar Azure Notification Hubs.

En este artículo se explica cómo realizar un gran número de operaciones en un centro de notificaciones o cómo exportar todos los registros.

## <a name="high-level-flow"></a>Flujo de alto nivel

El soporte técnico por lotes está diseñado para respaldar los trabajos de ejecución prolongada que implican millones de registros. Para lograr esta escala, el soporte técnico por lotes usa Azure Storage para almacenar la salida y los detalles del trabajo. Para las operaciones de actualización masiva, el usuario debe crear un archivo en un contenedor de blobs, cuyo contenido sea la lista de las operaciones de actualización de registro. Al iniciar el trabajo, el usuario proporciona una dirección URL para el blob de entrada, junto con una dirección URL a un directorio de salida (también en un contenedor de blobs). Una vez iniciado el trabajo, el usuario puede comprobar el estado mediante la consulta de una ubicación de URL proporcionada al iniciar el trabajo. Un trabajo específico solo puede realizar operaciones de un tipo específico (creaciones, actualizaciones o eliminaciones). Las operaciones de exportación se realizan de forma análoga.

## <a name="import"></a>Importar

### <a name="set-up"></a>Configurar

En esta sección se supone que tiene las siguientes entidades:

- Un centro de notificaciones aprovisionado.
- Un contenedor de blobs de Azure Storage.
- Referencias a los paquetes [NuGet de Azure Storage](https://www.nuget.org/packages/windowsazure.storage/) y [NuGet de Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

### <a name="create-input-file-and-store-it-in-a-blob"></a>Crear el archivo de entrada y almacenarlo en un blob

Un archivo de entrada contiene una lista de registros que se serializan en XML, uno por fila. Mediante el SDK de Azure, en el ejemplo de código siguiente se muestra cómo serializar los registros y cargarlos en el contenedor de blobs:

```csharp
private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
{
    StringBuilder builder = new StringBuilder();
    foreach (var registrationDescription in descriptions)
    {
        builder.AppendLine(registrationDescription.Serialize());
    }

    var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
    using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
    {
        inputBlob.UploadFromStream(stream);
    }
}
```

> [!IMPORTANT]
> El código anterior serializa los registros en la memoria y, a continuación, carga toda la secuencia en un blob. Si ha cargado un archivo de más de unos cuantos megabytes, consulte las instrucciones de los blobs de Azure sobre cómo realizar estos pasos; por ejemplo, [blobs en bloques](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs).

### <a name="create-url-tokens"></a>Crear tokens de URL

Una vez cargado el archivo de entrada, genere las direcciones URL que se proporcionarán a su centro de notificaciones para el archivo de entrada y el directorio de salida. puede utilizar dos contenedores de blobs diferentes para la entrada y la salida.

```csharp
static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
    };

    string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + sasContainerToken);
}

static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Read
    };
    string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + "/" + filePath + sasToken);
}
```

### <a name="submit-the-job"></a>Enviar el archivo

Con las dos URL de entrada y de salida, ahora puede iniciar el trabajo por lotes.

```csharp
NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
var createTask = client.SubmitNotificationHubJobAsync(
new NotificationHubJob {
        JobType = NotificationHubJobType.ImportCreateRegistrations,
        OutputContainerUri = outputContainerSasUri,
        ImportFileUri = inputFileSasUri
    }
);
createTask.Wait();

var job = createTask.Result;
long i = 10;
while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
{
    var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
    getJobTask.Wait();
    job = getJobTask.Result;
    Thread.Sleep(1000);
    i--;
}
```

Además de las URL de entrada y de salida, este ejemplo crea un objeto `NotificationHubJob` que contiene un objeto `JobType`, que puede ser de uno de los tipos siguientes:

- `ImportCreateRegistrations`
- `ImportUpdateRegistrations`
- `ImportDeleteRegistrations`

Una vez finalizada la llamada, el centro de notificaciones continúa con el trabajo, y puede comprobar su estado con una llamada a [GetNotificationHubJobAsync](/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.getnotificationhubjobasync?view=azure-dotnet).

Al completar el trabajo, puede inspeccionar el trabajo. Para ello, consulte los archivos siguientes en el directorio de salida:

- `/<hub>/<jobid>/Failed.txt`
- `/<hub>/<jobid>/Output.txt`

Estos archivos contienen la lista de operaciones correctas y erróneas del lote. El formato de archivo es `.cvs`, donde cada fila tiene el número de línea del archivo de entrada original y la salida de la operación (por lo general, la descripción del registro creado o actualizado).

### <a name="full-sample-code"></a>Código de ejemplo completo

El siguiente código de ejemplo importa los registros en un centro de notificaciones.

```csharp
using Microsoft.Azure.NotificationHubs;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Runtime.Serialization;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Xml;

namespace ConsoleApplication1
{
    class Program
    {
        private static string CONNECTION_STRING = "---";
        private static string HUB_NAME = "---";
        private static string INPUT_FILE_NAME = "CreateFile.txt";
        private static string STORAGE_ACCOUNT = "---";
        private static string STORAGE_PASSWORD = "---";
        private static StorageUri STORAGE_ENDPOINT = new StorageUri(new Uri("---"));

        static void Main(string[] args)
        {
            var descriptions = new[]
            {
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMkUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMjUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMhUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMdUxREQFBlVTTkMwMQ"),
            };

            // Write to blob store to create an input file
            var blobClient = new CloudBlobClient(STORAGE_ENDPOINT, new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(STORAGE_ACCOUNT, STORAGE_PASSWORD));
            var container = blobClient.GetContainerReference("testjobs");
            container.CreateIfNotExists();

            SerializeToBlob(container, descriptions);

            // TODO then create Sas
            var outputContainerSasUri = GetOutputDirectoryUrl(container);
            var inputFileSasUri = GetInputFileUrl(container, INPUT_FILE_NAME);


            // Import this file
            NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
            var createTask = client.SubmitNotificationHubJobAsync(
                new NotificationHubJob {
                    JobType = NotificationHubJobType.ImportCreateRegistrations,
                    OutputContainerUri = outputContainerSasUri,
                    ImportFileUri = inputFileSasUri
                }
            );
            createTask.Wait();

            var job = createTask.Result;
            long i = 10;
            while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
            {
                var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
                getJobTask.Wait();
                job = getJobTask.Result;
                Thread.Sleep(1000);
                i--;
            }
        }

        private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
        {
            StringBuilder builder = new StringBuilder();
            foreach (var registrationDescription in descriptions)
            {
                builder.AppendLine(registrationDescription.Serialize());
            }

            var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
            using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
            {
                inputBlob.UploadFromStream(stream);
            }
        }

        static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + sasContainerToken);
        }

        static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + "/" + filePath + sasToken);
        }

        static string GetJobPath(string namespaceName, string notificationHubPath, string jobId)
        {
            return string.Format(CultureInfo.InvariantCulture, @"{0}//{1}/{2}/", namespaceName, notificationHubPath, jobId);
        }
    }
}
```

## <a name="export"></a>Exportación

La exportación del registro es similar a la importación, con las diferencias siguientes:

- Solo necesita la dirección URL de salida.
- Debe crear un elemento NotificationHubJob de tipo ExportRegistrations.

### <a name="sample-code-snippet"></a>Fragmento de código de ejemplo

A continuación, se incluye un fragmento de código de ejemplo para la exportación de registros en Java:

```java
// Submit an export job
NotificationHubJob job = new NotificationHubJob();
job.setJobType(NotificationHubJobType.ExportRegistrations);
job.setOutputContainerUri("container uri with SAS signature");
job = hub.submitNotificationHubJob(job);

// Wait until the job is done
while(true){
    Thread.sleep(1000);
    job = hub.getNotificationHubJob(job.getJobId());
    if(job.getJobStatus() == NotificationHubJobStatus.Completed)
        break;
}

```

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de los registros, consulte los siguientes artículos:

- [Administración de registros](notification-hubs-push-notification-registration-management.md)
- [Etiquetas para registros](notification-hubs-tags-segment-push-message.md)
- [Registros de plantilla](notification-hubs-templates-cross-platform-push-messages.md)
