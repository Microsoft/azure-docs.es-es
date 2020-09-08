---
title: Preguntas más frecuentes sobre Azure Media Services
description: En este artículo se ofrecen respuestas a las preguntas frecuentes sobre Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: ff15206e2373ce6481cb65cbbeeb3c88cd2b7154
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269427"
---
# <a name="media-services-v2-frequently-asked-questions"></a>Preguntas más frecuentes sobre Media Services v2

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este artículo tratan las preguntas más frecuentes planteadas por la comunidad de usuarios de Azure Media Services (AMS).

## <a name="general-ams-faqs"></a>Preguntas más frecuentes generales sobre AMS

P: ¿Cómo se hace streaming a dispositivos de Apple iOS?

R: Agregue la ruta de acceso "(format=m3u8-aapl)" a la parte "/Manifest" de la dirección URL para indicar al servidor de origen de streaming que devuelva el contenido HLS para su consumo en dispositivos Apple iOS nativos (para obtener más información, lea la sección de [entrega de contenido](media-services-deliver-content-overview.md)).

P: ¿Cómo se escala la indización?

A. Las unidades reservadas son las mismas para las tareas de Encoding y de indización. Siga las instrucciones de [Escalado de unidades reservadas de Encoding](media-services-scale-media-processing-overview.md). **Tenga en cuenta** que el rendimiento del indexador no se ve afectado por el tipo de unidad reservada.

P: He cargado, codificado y publicado un vídeo. ¿Cuál es el motivo por el que el vídeo no se reproduce cuando intento transmitirlo?

A. Uno de los motivos más habituales es que no tiene el punto de conexión de streaming desde el que está intentando reproducir en estado **Running** (en ejecución).  

P: ¿Puedo realizar una composición en una secuencia en directo?

A. No se ofrece actualmente la composición en secuencias en vivo en Azure Media Services, por lo que tendrá que realizar una composición previa en el equipo.

P: ¿Puedo usar Azure CDN con secuencia en directo?

A. Media Services admite la integración con Azure CDN (para obtener más información, consulte [Administración de extremos de streaming en una cuenta de Media Services](media-services-portal-manage-streaming-endpoints.md)).  Puede usar el streaming en vivo con CDN. Azure Media Services proporciona salidas de Smooth Streaming, HLS y MPEG-DASH. Todos estos formatos usan HTTP para transferir datos y obtener beneficios del almacenamiento en caché de HTTP. En la transmisión en vivo, los datos de audio/vídeo reales se dividen en fragmentos y los fragmentos individuales se almacenan en caché en CDN. Solo tienen que actualizarse los datos de manifiesto. CDN actualiza periódicamente los datos de manifiesto.

P: ¿Los servicios multimedia de Azure admiten el almacenamiento de imágenes?

A. Si solo busca almacenar las imágenes JPEG o PNG, debe mantenerlas en Azure Blob Storage. No supone una ventaja si se colocan en la cuenta de Media Services a menos que desee mantenerlas asociadas a los recursos de vídeo o audio. O bien, si puede tener la necesidad de usar las imágenes como superposiciones en el codificador de vídeo, el Estándar de codificador multimedia admite la superposición de las imágenes encima de los vídeos, por eso muestra JPEG y PNG como formatos de entrada admitidos. Para obtener más información, consulte [Creación de superposiciones](media-services-advanced-encoding-with-mes.md#overlay).

P: ¿Cómo puedo copiar recursos de una cuenta de Media Services a otra?

A. Para copiar recursos de una cuenta de Media Services a otra mediante .NET, use el método de extensión [IAsset.Copy](https://github.com/Azure/azure-sdk-for-media-services-extensions/blob/dev/MediaServices.Client.Extensions/IAssetExtensions.cs#L354) disponible en el repositorio de [extensiones del SDK de .NET para Azure Media Services](https://github.com/Azure/azure-sdk-for-media-services-extensions/). Para obtener más información, consulte [esta](https://social.msdn.microsoft.com/Forums/azure/28912d5d-6733-41c1-b27d-5d5dff2695ca/migrate-media-services-across-subscription?forum=MediaServices) conversación del foro.

P: ¿Cuáles son los caracteres admitidos para nombrar los archivos cuando se trabaja con AMS?

A. Media Services usa el valor de la propiedad IAssetFile.Name al generar direcciones URL para el contenido de streaming (por ejemplo, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters.) Por esta razón, no se permite la codificación porcentual. El valor de la propiedad **Name**no puede tener ninguno de los siguientes [caracteres reservados para la codificación porcentual](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters):!*'();:@&=+$,/?%#[]" Además, solo puede haber un '.' para la extensión del nombre de archivo.

P: ¿Cómo se realiza la conexión con REST?

A. Para obtener más información sobre cómo conectarse a la API de Azure Media Services, consulte [Acceso a la API de Azure Media Services con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

P: ¿Cómo puedo girar un vídeo durante el proceso de codificación?

A. [Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md) admite ángulos de rotación de 90, 180 o 270 grados. El comportamiento predeterminado es "Auto", en el que se intentan detectar los metadatos de rotación del archivo MP4 o MOV entrante y la compensación. Incluya el siguiente elemento **Sources** en uno de los valores preestablecidos JSON definidos [aquí](media-services-mes-presets-overview.md):

```json
"Version": 1.0,
"Sources": [
{
  "Streams": [],
  "Filters": {
    "Rotation": "90"
  }
}
],
"Codecs": [

...
```


## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
