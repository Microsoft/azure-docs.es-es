---
title: Administración de recursos en Azure Media Services
titleSuffix: Azure Media Services
description: Un recurso en el que introduce elementos multimedia (por ejemplo, a través de la carga o la ingesta en vivo), genera elementos multimedia (a partir de la salida de un trabajo) y desde el que publica elementos multimedia (para streaming). Este tema le proporciona información general sobre cómo crear un nuevo recurso y cargar archivos.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 03/26/2020
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 6accd303ba11c4c1406c7a157fa8176972fc7a3a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87022914"
---
# <a name="manage-assets"></a>Administración de recursos

En Azure Media Services, un [recurso](/rest/api/media/assets) es donde 

* se cargan archivos multimedia en un recurso;
* se ingiere y archiva streaming en vivo en un recurso;
* se generan los resultados de la codificación de un trabajo analítico en un recurso;
* se publican elementos multimedia para streaming; y 
* se descargan archivos de un recurso.

En este tema se proporciona información general sobre la carga de archivos en un recurso y otras operaciones comunes. También se proporcionan vínculos a ejemplos de código y temas relacionados.

## <a name="prerequisite"></a>Requisito previo 

Antes de empezar a desarrollar, revise:

* [Conceptos](concepts-overview.md)
* [Desarrollo con las API de Media Services v3](media-services-apis-overview.md) (incluye información acerca de cómo acceder a las API, convenciones de nomenclatura, etc.) 

## <a name="upload-media-files-into-an-asset"></a>Carga de archivos multimedia en un recurso

Una vez que los archivos digitales se cargan en el almacenamiento y se asocian a un recurso, se pueden usar en los flujos de trabajo de codificación, transmisión y análisis de contenido de Media Services. Uno de los flujos de trabajo más comunes de Media Services es cargar, codificar y hacer streaming de un archivo. En esta sección se describen los pasos generales.

1. Use la API de Media Services v3 para crear un nuevo recurso de "entrada". Esta operación crea un contenedor en la cuenta de almacenamiento asociada a su cuenta de Media Services. La API devuelve el nombre del contenedor (por ejemplo, `"container": "asset-b8d8b68a-2d7f-4d8c-81bb-8c7bbbe67ee4"`).

    Si ya tiene un contenedor de blobs que quiera asociar a un recurso, puede especificar su nombre al crear ese recurso. Media Services actualmente solo admite blobs en la raíz del contenedor y que no tengan las rutas de acceso en el nombre de archivo. Por lo tanto, un contenedor con el nombre de archivo "input.mp4" funcionará perfectamente. Sin embargo, un contenedor con el nombre de archivo "videos/inputs/input.mp4" no funcionará.

    Puede usar la CLI de Azure para cargar contenido directamente en cualquier cuenta de almacenamiento y de contenedor sobre la que tenga derechos en su suscripción.

    El nombre del contenedor debe ser único y ha de seguir las directrices de nomenclatura de almacenamiento. El nombre no tiene que seguir el formato de nomenclatura para los contenedores de recursos de Media Services (GUID de recurso).

    ```azurecli
    az storage blob upload -f /path/to/file -c MyContainer -n MyBlob
    ```
2. Obtenga una dirección URL de SAS con permisos de lectura y escritura que podrá usar para cargar archivos digitales en el contenedor de recursos.

    Puede usar la API de Media Services para [enumerar las direcciones URL del contenedor de recursos](/rest/api/media/assets/listcontainersas).

    **AssetContainerSas.listContainerSas** toma un parámetro [ListContainerSasInput](/rest/api/media/assets/listcontainersas#listcontainersasinput) en el que se establece `expiryTime`. La hora debe establecerse en < 24 horas.

    [ListContainerSasInput](/rest/api/media/assets/listcontainersas#listcontainersasinput) devuelve varias direcciones URL de SAS, ya que hay dos claves de cuenta de almacenamiento para cada cuenta de almacenamiento. Una cuenta de almacenamiento tiene dos claves porque ayuda con la conmutación por error y la rotación integral de las claves de la cuenta de almacenamiento. De las dos direcciones URL de SAS, la primera representa la primera clave de la cuenta de almacenamiento y la segunda representa la segunda clave.
3. Use los SDK o las API de Azure Storage (por ejemplo, la [API de REST de Storage](../../storage/common/storage-rest-api-auth.md), el [SDK de .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)) para cargar archivos en el contenedor de recursos.
4. Use las API de Media Services v3 para crear una transformación y un trabajo para procesar el recurso de "entrada". Para obtener más información, consulte [Transformaciones y trabajos](./transforms-jobs-concept.md).
5. Haga streaming del contenido del recurso de "salida".

### <a name="create-a-new-asset"></a>Crear un recurso nuevo

> [!NOTE]
> Las propiedades del recurso del tipo Datetime siempre están en formato UTC.

#### <a name="rest"></a>REST

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Media/mediaServices/{amsAccountName}/assets/{assetName}?api-version=2018-07-01
```

Para obtener un ejemplo de REST, consulte el ejemplo para [crear un recurso con REST](/rest/api/media/assets/createorupdate#examples).

En el ejemplo se muestra cómo crear el **cuerpo de la solicitud**, donde puede especificar información útil, como la descripción, el nombre del contenedor, la cuenta de almacenamiento y otra información útil.

#### <a name="curl"></a>cURL

```cURL
curl -X PUT \
  'https://management.azure.com/subscriptions/00000000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/Microsoft.Media/mediaServices/amsAccountName/assets/myOutputAsset?api-version=2018-07-01' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "properties": {
    "description": "",
  }
}'
```

#### <a name="net"></a>.NET

```csharp
 Asset asset = await client.Assets.CreateOrUpdateAsync(resourceGroupName, accountName, assetName, new Asset());
```

### <a name="see-also"></a>Consulte también

* [Creación de una entrada de trabajo a partir de un archivo local](job-input-from-local-file-how-to.md).
* [Creación de una entrada de trabajo desde una dirección URL de HTTPS](job-input-from-http-how-to.md).

## <a name="ingest-and-archive-live-streams-into-an-asset"></a>Ingesta y archivo de streaming en vivo en un recurso

En Media Services, un objeto [Live Output](/rest/api/media/liveoutputs) es como una grabadora de vídeo digital que capta y graba el streaming en vivo en un recurso de su cuenta de Media Services. El contenido grabado se conserva en el contenedor definido por el recurso [Asset](/rest/api/media/assets).

Para más información, consulte:

* [Uso de un DVR en la nube](live-event-cloud-dvr.md)
* [Tutorial de streaming en vivo](stream-live-tutorial-with-api.md)

## <a name="output-the-results-of-a-job-to-an-asset"></a>Generación de los resultados de un trabajo en un recurso

En Azure Media Services, cuando se procesan los vídeos (por ejemplo, para su codificación o análisis), debe crear un [recurso](assets-concept.md) de salida para almacenar el resultado del [trabajo](transforms-jobs-concept.md).

Para más información, consulte:

* [Codificación de un vídeo](encoding-concept.md)
* [Creación de una entrada de trabajo a partir de un archivo local](job-input-from-local-file-how-to.md).

## <a name="publish-an-asset-for-streaming"></a>Publicación de un recurso para streaming

Para publicar un recurso para streaming, debe crear un [localizador de streaming](streaming-locators-concept.md). El localizador de streaming necesita saber el nombre del recurso que quiere publicar. 

Para más información, consulte:

[Tutorial: Carga, codificación y streaming de vídeos con Media Services v3](stream-files-tutorial-with-api.md)

## <a name="download-results-of-a-job-from-an-output-asset"></a>Descarga de los resultados de un trabajo desde un recurso de salida

Después, puede almacenar los resultados del trabajo en una carpeta local mediante las API de Media Service y Storage. 

Consulte los [archivos de descarga](download-results-howto.md) de ejemplo.

## <a name="filtering-ordering-paging"></a>Filtrado, ordenación, paginación

Consulte [Filtrado, ordenación y paginación de entidades de Media Services](entities-overview.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte los ejemplos de código completos que demuestran cómo cargar, codificar, analizar, hacer streaming en vivo y a petición: 

* [Java](/samples/azure-samples/media-services-v3-java/azure-media-services-v3-samples-using-java/), 
* [.NET](/samples/azure-samples/media-services-v3-dotnet/azure-media-services-v3-samples-using-net/), 
* [REST](/samples/azure-samples/media-services-v3-rest-postman/azure-media-services-postman-collection/).
