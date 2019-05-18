---
title: Recursos en Azure Media Services | Microsoft Docs
description: En este artículo se explica qué son los artículos y cómo los usa Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/11/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 2afcf2066238414cd08e32901ffccf2a44718b6d
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65551771"
---
# <a name="assets"></a>Recursos

En Azure Media Services, un [activos](https://docs.microsoft.com/rest/api/media/assets) contiene información acerca de los archivos digitales almacenados en Azure Storage (incluidos el vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y archivos de subtítulos). 

Un recurso se asigna a un contenedor de blobs en la [cuenta de Azure Storage](storage-account-concept.md) y los archivos del recurso se almacenan como blobs en bloques en ese contenedor. Azure Media Services admite los niveles de blob cuando la cuenta usa el almacenamiento de uso general v2 (GPv2). Con GPv2, puede mover los archivos al [almacenamiento de acceso esporádico o al almacenamiento en frío](https://docs.microsoft.com/azure/storage/blobs/storage-blob-storage-tiers). El almacenamiento de **archivos** es adecuado para guardar archivos de origen cuando ya no son necesarios (por ejemplo, una vez codificados).

El nivel de almacenamiento de **archivos** solo se recomienda para archivos de origen muy grandes que ya se hayan codificado y cuya salida del trabajo de codificación se haya colocado en un contenedor de blobs de salida. Los blobs del contenedor de salida que quiera asociar con un recurso y usar para hacer streaming o analizar contenido, deben existir en un nivel de almacenamiento **frecuente** o **esporádico**.

## <a name="upload-digital-files-into-assets"></a>Cargar los archivos digitales en recursos

Después de que los archivos digitales se cargan en el almacenamiento y asociados con un recurso, se puede usar en los servicios de multimedia, codificación, streaming, análisis de flujos de trabajo de contenido. Uno de los flujos de trabajo más comunes de Media Services es cargar, codificar y hacer streaming de un archivo. En esta sección se describen los pasos generales.

> [!TIP]
> Antes de empezar a desarrollar, revise [desarrollar con las API de Media Services v3](media-services-apis-overview.md) (incluye información sobre cómo acceder a las API, las convenciones de nomenclatura, etcetera.)

1. Use la API de Media Services v3 para crear un nuevo recurso de "entrada". Esta operación crea un contenedor en la cuenta de almacenamiento asociada a su cuenta de Media Services. La API devuelve el nombre del contenedor (por ejemplo, `"container": "asset-b8d8b68a-2d7f-4d8c-81bb-8c7bbbe67ee4"`).
   
    Si ya tiene un contenedor de blobs que quiera asociar a un recurso, puede especificar el nombre del contenedor al crear ese recurso. Media Services actualmente solo admite blobs en la raíz del contenedor y que no tengan las rutas de acceso en el nombre de archivo. Por lo tanto, un contenedor con el nombre de archivo "input.mp4" funcionará perfectamente. Sin embargo, un contenedor con el nombre de archivo "videos/inputs/input.mp4", no funcionará.

    Puede usar la CLI de Azure para cargar contenido directamente en cualquier cuenta de almacenamiento y de contenedor sobre la que tenga derechos en su suscripción. <br/>El nombre del contenedor debe ser único y ha de seguir las directrices de nomenclatura de almacenamiento. El nombre no tiene que seguir el formato de nomenclatura para los contenedores de recursos de Media Services (GUID de recurso). 
    
    ```azurecli
    az storage blob upload -f /path/to/file -c MyContainer -n MyBlob
    ```
2. Obtenga una dirección URL de SAS con permisos de lectura y escritura que podrá usar para cargar archivos digitales en el contenedor de recursos. Puede usar la API de Media Services para [enumerar las direcciones URL del contenedor de recursos](https://docs.microsoft.com/rest/api/media/assets/listcontainersas).
3. Use los SDK o API de Azure Storage (por ejemplo, la [API REST de Storage](../../storage/common/storage-rest-api-auth.md), el [SDK para JAVA](../../storage/blobs/storage-quickstart-blobs-java-v10.md) o el [SDK para .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)) para cargar archivos en el contenedor de recursos. 
4. Use las API de Media Services v3 para crear una transformación y un trabajo para procesar el recurso de "entrada". Para obtener más información, consulte [Transformaciones y trabajos](transform-concept.md).
5. Haga streaming del contenido del recurso de "salida".

Para obtener un ejemplo de .NET completo que muestre cómo crear el recurso, obtener una dirección URL de SAS que se pueda escribir en el contenedor del recurso de almacenamiento, cargar el archivo en el contenedor de almacenamiento mediante la dirección URL de SAS, consulte [Creación de una entrada de trabajo a partir de un archivo local](job-input-from-local-file-how-to.md).

### <a name="create-a-new-asset"></a>Crear un recurso nuevo

> [!NOTE]
> Las propiedades del recurso del tipo Datetime siempre están en formato UTC.

#### <a name="rest"></a>REST

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Media/mediaServices/{amsAccountName}/assets/{assetName}?api-version=2018-07-01
```

Para obtener un ejemplo de REST, consulte el ejemplo para [crear un recurso con REST](https://docs.microsoft.com/rest/api/media/assets/createorupdate#examples).

En el ejemplo se muestra cómo crear el **cuerpo de la solicitud** donde puede especificar información útil, como la descripción, el nombre del contenedor, la cuenta de almacenamiento y otros datos similares.

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

Para obtener un ejemplo completo, consulte [Creación de una entrada de trabajo a partir de un archivo local](job-input-from-local-file-how-to.md). En Media Services v3, también se puede crear una entrada de trabajo a partir de las direcciones URL de HTTPS (consulte [Creación de una entrada de trabajo a partir de un una dirección URL de HTTPS](job-input-from-http-how-to.md)).

## <a name="filtering-ordering-paging"></a>Filtrado, ordenación, paginación

Consulte [Filtrado, ordenación y paginación de entidades de Media Services](entities-overview.md).

## <a name="storage-side-encryption"></a>Cifrado del lado de almacenamiento

Para proteger los recursos en reposo, estos se deben cifrar mediante el cifrado del lado de almacenamiento. En la tabla siguiente se muestra cómo funciona el cifrado del lado de almacenamiento en Media Services:

|Opción de cifrado|DESCRIPCIÓN|Media Services v2|Media Services v3|
|---|---|---|---|
|Cifrado de almacenamiento en Media Services|Cifrado de AES-256, clave administrada por Media Services|Admitido<sup>(1)</sup>|No admitido<sup>(2)</sup>|
|[Storage Service Encryption para datos en reposo](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)|Cifrado en el lado del servidor que ofrece Azure Storage, clave administrada por Azure o por el cliente|Compatible|Compatible|
|[Cifrado en el lado de cliente de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-client-side-encryption)|Cifrado en el lado de cliente que ofrece Azure Storage, clave administrada por el cliente en Key Vault|No compatible|No compatible|

<sup>1</sup> Aunque Media Services admite el control del contenido de una forma clara o sin ninguna forma de cifrado, hacerlo así no es recomendable.

<sup>2</sup> En Media Services v3, el cifrado de almacenamiento (cifrado con AES-256) solo es compatible con versiones anteriores si los recursos se crearon con Media Services v2. Esto significa que la versión v3 funciona con los recursos cifrados de almacenamiento ya existentes pero no permitirá la creación de otros nuevos.

## <a name="next-steps"></a>Pasos siguientes

* [Streaming de un archivo](stream-files-dotnet-quickstart.md)
* [Uso de un DVR en la nube](live-event-cloud-dvr.md)
* [Diferencias entre la versión v2 y v3 de Media Services](migrate-from-v2-to-v3.md)
