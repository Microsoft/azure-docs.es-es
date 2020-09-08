---
title: Migración de Azure Media Services de la versión 2 a la 3
description: En este artículo se describen los cambios realizados en Azure Media Services v3 y se muestran las diferencias entre las dos versiones.
services: media-services
documentationcenter: na
author: IngridAtMicrosoft
manager: femila
editor: ''
tags: ''
keywords: azure media services, transmisión, difusión, en vivo, sin conexión
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 061ae48de9a73270ed499282c9fc9a4f8f1dba90
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89298953"
---
# <a name="media-services-v2-vs-v3"></a>Versiones 2 y 3 de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este artículo se describen los cambios realizados en Azure Media Services v3 y se muestran las diferencias entre las dos versiones.

## <a name="general-changes-from-v2"></a>Cambios generales en relación con la versión 2

* En el caso de los recursos creados con la versión 3, Media Services solo admite el [cifrado de almacenamiento del lado servidor de Azure Storage](../../storage/common/storage-service-encryption.md).
    * Puede usar las API v3 con recursos creados con las API v2 que tenían [cifrado de almacenamiento](../previous/media-services-rest-storage-encryption.md) (AES 256) proporcionado por Media Services.
    * No se pueden crear recursos con el [cifrado de almacenamiento](../previous/media-services-rest-storage-encryption.md) AES 256 heredado mediante las API v3.
* Las propiedades del [recurso](assets-concept.md) de la versión 3 difieren de las de la versión 2. Consulte [cómo se asignan las propiedades](#map-v3-asset-properties-to-v2).
* Los SDK v3 ahora se han desacoplado del SDK de Storage, lo que proporciona mayor control sobre la versión del SDK de Storage que desea usar y evita problemas de control de versiones. 
* En las API v3, todas las velocidades de bits de codificación se expresan en bits por segundo. Esto es diferente a los valores preestablecidos de Media Encoder Standard v2. Por ejemplo, la velocidad de bits en v2 se especificaría como 128 (kbps), mientras que en v3 sería 128000 (bits/segundo). 
* Las entidades AssetFiles, AccessPolicies y IngestManifests no existen en v3.
* La propiedad IAsset.ParentAssets no existe en v3.
* ContentKeys ya no es una entidad; ahora es una propiedad de Streaming Locator.
* La compatibilidad con Event Grid reemplaza los NotificationEndpoints.
* Se cambió el nombre de las siguientes entidades.
    * Salida de trabajo reemplaza a Tarea y ahora forma parte de un trabajo.
    * Localizador de streaming reemplaza a Localizador.
    * Evento en directo reemplaza a Canal.<br/>La facturación de Eventos en directo se basa en los medidores de Canal en vivo. Para más información, consulte la [facturación](live-event-states-billing.md) y los [precios](https://azure.microsoft.com/pricing/details/media-services/).
    * Salida en vivo reemplaza a Programa.
* Los objetos LiveOutput comienzan al crearlos y se detienen cuando se eliminan. Los programas funcionaban de forma diferente en las API v2; deben iniciarse después de la creación.
* Para obtener información sobre un trabajo, debe conocer el nombre de la transformación en la que se ha creado dicho trabajo. 
* En la versión v2, se generan archivos de metadatos de [entrada](../previous/media-services-input-metadata-schema.md) y [salida](../previous/media-services-output-metadata-schema.md) como resultado de un trabajo de codificación. En la versión v3, el formato de metadatos cambió de XML a JSON. 
* En Media Services v2, se puede especificar el vector de inicialización (VI). En Media Services v3, no se puede especificar el VI de FairPlay. Aunque no afecta a los clientes que usan Media Services tanto para el empaquetado como para la entrega de licencias, puede ser un problema al usar un sistema DRM de terceros para entregar las licencias de FairPlay (modo híbrido). En ese caso, es importante saber que el VI de FairPlay se deriva del id. de clave cbcs y se puede recuperar con esta fórmula:

    ```
    string cbcsIV =  Convert.ToBase64String(HexStringToByteArray(cbcsGuid.ToString().Replace("-", string.Empty)));
    ```

    con

    ``` 
    public static byte[] HexStringToByteArray(string hex)
    {
        return Enumerable.Range(0, hex.Length)
            .Where(x => x % 2 == 0)
            .Select(x => Convert.ToByte(hex.Substring(x, 2), 16))
            .ToArray();
    }
    ```

    Para obtener más información, consulte el [código de C# de Azure Functions para Media Services v3 en modo híbrido para las operaciones en directo y VOD](https://github.com/Azure-Samples/media-services-v3-dotnet-core-functions-integration/tree/master/LiveAndVodDRMOperationsV3).
 
> [!NOTE]
> Revise las convenciones de nomenclatura que se aplican a [los recursos de Media Services v3](media-services-apis-overview.md#naming-conventions). Revise también [los blobs de nomenclatura](assets-concept.md#naming).

## <a name="feature-gaps-with-respect-to-v2-apis"></a>Carencias de características con respecto a las API v2

La API v3 tiene las siguientes carencias de características con respecto a la API v2. El trabajo para la solución de deficiencias está en curso.

* El [codificador premium](../previous/media-services-premium-workflow-encoder-formats.md) y los [procesadores de análisis multimedia](../previous/media-services-analytics-overview.md) heredados (Azure Media Services Indexer 2 Preview, Face Redactor, etc.) no son accesibles mediante v3.<br/>Los clientes que deseen migrar de Media Indexer 1 o 2 Preview pueden usar inmediatamente el valor preestablecido de AudioAnalyzer en la API v3.  Este valor preestablecido nuevo contiene más características que la instancia anterior de Media Indexer 1 o 2. 
* Muchas de las [características avanzadas de Media Encoder Standard de las API v2](../previous/media-services-advanced-encoding-with-mes.md) actualmente no están disponibles en v3, como:
  
    * Unión de recursos
    * Superposiciones
    * Recorte
    * Sprites de miniaturas
    * Inserción de una pista de audio silenciosa cuando la entrada no tiene audio
    * Inserción de una pista de vídeo cuando la entrada no tiene vídeo
* Los Eventos en directo con transcodificación actualmente no admiten la inserción de careta durante la transmisión y la inserción de anuncios mediante la llamada API. 
* Consulte el código de ejemplo `https://github.com/Azure-Samples/media-services-v2-dotnet-core-restsharp-sample.git` para ver los procedimientos recomendados y los patrones relativos al uso de la API REST V2 en el .SDK de NETCore.

## <a name="asset-specific-changes"></a>Cambios específicos de recursos

### <a name="map-v3-asset-properties-to-v2"></a>Asignación de las propiedades de los recursos de v3 a v2

En la tabla siguiente se muestra cómo las propiedades de los [recursos](/rest/api/media/assets/createorupdate#asset) de v3 se asignan a las propiedades de los recursos de v2.

|Propiedades de v3|Propiedades de v2|
|---|---|
|`id`: (único), la ruta de acceso completa de Azure Resource Manager; consulte ejemplos en el artículo sobre [recursos](/rest/api/media/assets/createorupdate).||
|`name`: (único), consulte [Convenciones de nomenclatura](media-services-apis-overview.md#naming-conventions). ||
|`alternateId`|`AlternateId`|
|`assetId`|`nb:cid:UUID:`: (único), valor que empieza por el prefijo `Id`.|
|`created`|`Created`|
|`description`|`Name`|
|`lastModified`|`LastModified`|
|`storageAccountName`|`StorageAccountName`|
|`storageEncryptionFormat`| `Options` (opciones de creación)|
|`type`||

### <a name="storage-side-encryption"></a>Cifrado del lado de almacenamiento

Para proteger los recursos en reposo, estos se deben cifrar mediante el cifrado del lado de almacenamiento. En la tabla siguiente se muestra cómo funciona el cifrado del lado de almacenamiento en Media Services:

|Opción de cifrado|Descripción|Media Services v2|Media Services v3|
|---|---|---|---|
|Cifrado de almacenamiento en Media Services|Cifrado AES-256, clave administrada por Media Services|Admitido<sup>(1)</sup>|No admitido<sup>(2)</sup>|
|[Storage Service Encryption para datos en reposo](../../storage/common/storage-service-encryption.md)|Cifrado en el lado servidor que ofrece Azure Storage, clave administrada por Azure o por el cliente.|Compatible|Compatible|
|[Cifrado en el lado de cliente de almacenamiento](../../storage/common/storage-client-side-encryption.md)|Cifrado en el lado cliente que ofrece Azure Storage, clave administrada por el cliente en Key Vault.|No compatible|No compatible|

<sup>1</sup> Aunque Media Services admite el control del contenido sin cifrado, no se recomienda.

<sup>2</sup> En Media Services v3, el cifrado de almacenamiento (cifrado con AES-256) solo es compatible con versiones anteriores si los recursos se crearon con Media Services v2. Esto significa que la versión v3 funciona con los recursos cifrados de almacenamiento ya existentes pero no permitirá la creación de otros.

## <a name="code-differences"></a>Diferencias de código

En la tabla siguiente se muestran las diferencias de código entre v2 y v3 para escenarios comunes.

|Escenario|API V2|API V3|
|---|---|---|
|Crear un recurso y cargar un archivo |[Ejemplo de .NET v2](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L113)|[Ejemplo de .NET v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L169)|
|Enviar un trabajo|[Ejemplo de .NET v2](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L146)|[Ejemplo de .NET v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L298)<br/><br/>Se explica cómo crear una transformación y después enviar un trabajo.|
|Publicar un recurso con cifrado AES |1. Crear ContentKeyAuthorizationPolicyOption<br/>2. Crear ContentKeyAuthorizationPolicy<br/>3. Crear AssetDeliveryPolicy<br/>4. Crear Asset y cargar contenido o enviar trabajo y usar el recurso de salida<br/>5. Asociar AssetDeliveryPolicy con Asset<br/>6. Crear ContentKey<br/>7. Adjuntar ContentKey a Asset<br/>8. Crear AccessPolicy<br/>9. Crear Locator<br/><br/>[Ejemplo de .NET v2](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L64)|1. Crear ContentKeyPolicy<br/>2. Crear Asset<br/>3. Cargar contenido o usar Asset como JobOutput<br/>4. Creación de un localizador de streaming<br/><br/>[Ejemplo de .NET v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs#L105)|
|Obtener detalles de un trabajo y administrar trabajos |[Administrar trabajos con v2](../previous/media-services-dotnet-manage-entities.md#get-a-job-reference) |[Administrar trabajos con v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L546)|

> [!NOTE]
> Marque este artículo y siga buscando actualizaciones.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Guía de migración para mover de Media Services v2 a v3](migrate-from-v2-to-v3.md)
