---
title: Datos adjuntos de Azure Cosmos DB
description: En este artículo se ofrece información general sobre los datos adjuntos de Azure Cosmos DB.
author: aliuy
ms.author: andrl
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/07/2020
ms.reviewer: sngun
ms.openlocfilehash: cb5dbe5101231963f45132fca58279b934c5e49c
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90055374"
---
# <a name="azure-cosmos-db-attachments"></a>Datos adjuntos de Azure Cosmos DB

Los datos adjuntos de Azure Cosmos DB son elementos especiales que contienen las referencias a los metadatos asociados con un archivo multimedia o blob externo.

Azure Cosmos DB admite dos tipos de datos adjuntos:

* Los **datos adjuntos no administrados** son un contenedor alrededor de una referencia de URI a un blob que se almacena en un servicio externo (por ejemplo, Azure Storage, OneDrive, etc.). Este enfoque es similar al almacenamiento de una propiedad URI en un elemento de Azure Cosmos DB estándar.
* Los **datos adjuntos administrados** son blobs administrados y almacenados internamente por Azure Cosmos DB y expuestos a través de un elemento mediaLink generado por el sistema.


> [!NOTE]
> Los datos adjuntos son una característica heredada. Su compatibilidad se limita a ofrecer una funcionalidad continua si ya está usando esta característica.
> 
> En lugar de usar datos adjuntos, se recomienda usar Azure Blob Storage como un servicio de almacenamiento de blobs creado específicamente para almacenar los datos de blob. Puede continuar almacenando los metadatos relacionados con los blobs, junto con los vínculos de URI de referencia, en Azure Cosmos DB como propiedades de elementos. El almacenamiento de estos datos en Azure Cosmos DB proporciona la capacidad de consultar los metadatos y los vínculos a los blobs almacenados en Azure Blob Storage.
> 
> Microsoft se compromete a proporcionar un aviso, como mínimo, 36 meses antes de que los datos adjuntos estén totalmente en desuso, lo que se anunciará en una fecha posterior.

## <a name="known-limitations"></a>Restricciones conocidas

Los datos adjuntos administrados de Azure Cosmos DB se distinguen de la compatibilidad con los elementos estándar, para los que ofrece escalabilidad ilimitada, distribución global e integración con otros servicios de Azure.

- Los datos adjuntos no se admiten en todas las versiones de los SDK de Azure Cosmos DB.
- Los datos adjuntos administrados se limitan a 2 GB de almacenamiento por cuenta de base de datos.
- Los datos adjuntos administrados no son compatibles con la distribución global de Azure Cosmos DB y no se replican entre regiones.

## <a name="migrating-attachments-to-azure-blob-storage"></a>Migración de datos adjuntos a Azure Blob Storage

Se recomienda migrar los datos adjuntos de Azure Cosmos DB a Azure Blob Storage siguiendo estos pasos:

1. Copie los datos adjuntos del contenedor de Azure Cosmos DB de origen en el contenedor de Azure Blob Storage de destino.
2. Valide los datos del blob cargado en el contenedor de Azure Blob Storage de destino.
3. Si es aplicable, agregue referencias de URI a los blobs contenidos en Azure Blob Storage como propiedades de cadena dentro del conjunto de datos de Azure Cosmos DB.
4. Refactorice el código de aplicación para leer y escribir blobs en el nuevo contenedor de Azure Blob Storage.

En el ejemplo de código de dotnet siguiente se muestra cómo copiar datos adjuntos de Azure Cosmos DB en Azure Blob Storage como parte de un flujo de migración mediante el SDK de .NET  de Azure Cosmos DB v2 y el SDK de .NET de Azure Blob Storage v12. Asegúrese de reemplazar `<placeholder values>` para la cuenta de Azure Cosmos DB de origen y el contenedor de Azure Blob Storage de destino.

```csharp

using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace attachments
{
    class Program
    {
        private static string cosmosAccount = "<Your_Azure_Cosmos_account_URI>";
        private static string cosmosKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>";
        private static string cosmosDatabaseName = "<Your_Azure_Cosmos_database>";
        private static string cosmosCollectionName = "<Your_Azure_Cosmos_collection>";
        private static string storageConnectionString = "<Your_Azure_Storage_connection_string>";
        private static string storageContainerName = "<Your_Azure_Storage_container_name>";
        private static DocumentClient cosmosClient = new DocumentClient(new Uri(cosmosAccount), cosmosKey);
        private static BlobServiceClient storageClient = new BlobServiceClient(storageConnectionString);
        private static BlobContainerClient storageContainerClient = storageClient.GetBlobContainerClient(storageContainerName);

        static void Main(string[] args)
        {
            CopyAttachmentsToBlobsAsync().Wait();
        }

        private async static Task CopyAttachmentsToBlobsAsync()
        {
            Console.WriteLine("Copying Azure Cosmos DB Attachments to Azure Blob Storage ...");

            int totalCount = 0;
            string docContinuation = null;

            // Iterate through each item (document in v2) in the Azure Cosmos DB container (collection in v2) to look for attachments.
            do
            {
                FeedResponse<dynamic> response = await cosmosClient.ReadDocumentFeedAsync(
                    UriFactory.CreateDocumentCollectionUri(cosmosDatabaseName, cosmosCollectionName),
                    new FeedOptions
                    {
                        MaxItemCount = -1,
                        RequestContinuation = docContinuation
                    });
                docContinuation = response.ResponseContinuation;

                foreach (Document document in response)
                {
                    string attachmentContinuation = null;
                    PartitionKey docPartitionKey = new PartitionKey(document.Id);

                    // Iterate through each attachment within the item (if any).
                    do
                    {
                        FeedResponse<Attachment> attachments = await cosmosClient.ReadAttachmentFeedAsync(
                            document.SelfLink,
                            new FeedOptions
                            {
                                PartitionKey = docPartitionKey,
                                RequestContinuation = attachmentContinuation
                            }
                        );
                        attachmentContinuation = attachments.ResponseContinuation;

                        foreach (var attachment in attachments)
                        {
                            // Download the attachment in to local memory.
                            MediaResponse content = await cosmosClient.ReadMediaAsync(attachment.MediaLink);

                            byte[] buffer = new byte[content.ContentLength];
                            await content.Media.ReadAsync(buffer, 0, buffer.Length);

                            // Upload the locally buffered attachment to blob storage
                            string blobId = String.Concat(document.Id, "-", attachment.Id);

                            Azure.Response<BlobContentInfo> uploadedBob = await storageContainerClient.GetBlobClient(blobId).UploadAsync(
                                new MemoryStream(buffer, writable: false),
                                true
                            );

                            Console.WriteLine("Copied attachment ... Item Id: {0} , Attachment Id: {1}, Blob Id: {2}", document.Id, attachment.Id, blobId);
                            totalCount++;

                            // Clean up attachment from Azure Cosmos DB.
                            // Warning: please verify you've succesfully migrated attachments to blog storage prior to cleaning up Azure Cosmos DB.
                            // await cosmosClient.DeleteAttachmentAsync(
                            //     attachment.SelfLink,
                            //     new RequestOptions { PartitionKey = docPartitionKey }
                            // );

                            // Console.WriteLine("Cleaned up attachment ... Document Id: {0} , Attachment Id: {1}", document.Id, attachment.Id);
                        }

                    } while (!string.IsNullOrEmpty(attachmentContinuation));
                }
            }
            while (!string.IsNullOrEmpty(docContinuation));

            Console.WriteLine("Finished copying {0} attachments to blob storage", totalCount);
        }
    }
}

```

## <a name="next-steps"></a>Pasos siguientes

- Introducción a [Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-dotnet)
- Obtener referencias para usar datos adjuntos mediante el [SDK de .NET de Azure Cosmos DB v2](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.attachment?view=azure-dotnet)
- Obtener referencias para usar datos adjuntos mediante el [SDK de Java de Azure Cosmos DB v2](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb.attachment?view=azure-java-stable)
- Obtener referencias para usar datos adjuntos mediante la [API de REST de Azure Cosmos DB](https://docs.microsoft.com/rest/api/cosmos-db/attachments)
