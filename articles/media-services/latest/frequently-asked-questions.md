---
title: Preguntas más frecuentes de Azure Media Services v3 | Microsoft Docs
description: En este artículo se ofrecen respuestas a las preguntas más frecuentes de Azure Media Services v3.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 04/24/2019
ms.author: juliako
ms.openlocfilehash: 98e8c0ccd150776341e644f7565696e8fbd63e99
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556279"
---
# <a name="azure-media-services-v3-frequently-asked-questions"></a>Preguntas más frecuentes sobre Azure Media Services v3

En este artículo se ofrecen respuestas a las preguntas más frecuentes de Azure Media Services (AMS) v3.

## <a name="v3-apis"></a>API de la versión v3

### <a name="what-azure-roles-can-perform-actions-on-azure-media-services-resources"></a>¿Qué roles de Azure pueden realizar acciones en los recursos de Azure Media Services? 

Consulte [control de acceso basado en roles (RBAC) para cuentas de Media Services](rbac-overview.md).

### <a name="how-do-i-configure-media-reserved-units"></a>¿Cómo se pueden configurar las unidades reservadas de multimedia?

Para los trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda encarecidamente aprovisionar la cuenta con 10 MRU S3. Si necesita más de 10 MRU S3, abra una incidencia de soporte técnico desde [Azure Portal](https://portal.azure.com/).

Para más información, consulte [Escalado de procesamiento de medios con la CLI](media-reserved-units-cli-how-to.md).

### <a name="what-is-the-recommended-method-to-process-videos"></a>¿Cuál es el método recomendado para procesar vídeos?

Use [transformaciones](https://docs.microsoft.com/rest/api/media/transforms) para configurar tareas comunes para codificar o analizar vídeos. Cada **Transformación** describe una receta, o un flujo de trabajo simple de tareas para procesar los archivos de vídeo o audio. Un [trabajo](https://docs.microsoft.com/rest/api/media/jobs) es la solicitud real a Media Services para aplicar el **transformar** a un determinado contenido de vídeo o audio de entrada. Una vez creada la transformación, puede enviar trabajos mediante las API de Media Services o cualquiera de los SDK publicados. Para obtener más información, consulte [Transformaciones y trabajos](transforms-jobs-concept.md).

### <a name="how-does-pagination-work"></a>¿Cómo funciona la paginación?

Al usar la paginación, siempre debe usar el vínculo siguiente para enumerar la colección y no tener que depender de un tamaño de página determinado. Para obtener información detallada y ejemplos, consulte [Filtrado, ordenación y paginación](entities-overview.md).

### <a name="what-features-are-not-yet-available-in-azure-media-services-v3"></a>¿Qué características todavía no están disponibles en Azure Media Services v3?

Para obtener más información, consulte [característica vacíos con respecto a las API v2](migrate-from-v2-to-v3.md#feature-gaps-with-respect-to-v2-apis).

## <a name="live-streaming"></a>Streaming en vivo 

###  <a name="how-to-insert-breaksvideos-and-image-slates-during-live-stream"></a>¿Cómo insertar pausas o vídeos y caretas de imagen durante una transmisión en directo?

La codificación en directo de Media Services v3 no admite aún la inserción de caretas de vídeo o de imagen durante una transmisión en directo. 

Puede usar un [codificador local en directo](recommended-on-premises-live-encoders.md) para cambiar el vídeo de origen. Muchas aplicaciones proporcionan la posibilidad de cambiar los orígenes como, por ejemplo, Telestream Wirecast, Switcher Studio (en iOS), OBS Studio (aplicación gratuita), etc.

## <a name="content-protection"></a>Protección de contenido

### <a name="how-and-where-to-get-jwt-token-before-using-it-to-request-license-or-key"></a>¿Cómo y dónde se puede obtener el token JWT antes de usarlo para solicitar la licencia o la clave?

1. Para entornos de producción, deberá tener servicios de token seguro (STS) (servicio web) que emite el token JWT tras una solicitud HTTPS. Para la prueba, puede usar el código que se muestra en el método **GetTokenAsync** definido en [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs).
2. Una vez que se autentique un usuario, el reproductor deberá solicitar al STS dicho tipo de token y asignarlo como el valor del token. Puede usar la [API de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/).

* Para un ejemplo de cómo ejecutar STS, con claves simétricas y asimétricas, consulte [ https://aka.ms/jwt ](https://aka.ms/jwt). 
* Para un ejemplo de un reproductor basado en Azure Media Player usando dicho token JWT, consulte [ https://aka.ms/amtest ](https://aka.ms/amtest) (expanda el vínculo "player_settings" para ver la entrada de token).

### <a name="how-do-you-authorize-requests-to-stream-videos-with-aes-encryption"></a>¿Cómo se autorizan las solicitudes para transmitir vídeos con cifrado AES?

El enfoque correcto consiste en aprovechar STS (servicio de token seguro):

Según el perfil de usuario, agregue en STS notificaciones distintas (por ejemplo, "Usuario premium", "Usuario básico", "Usuario de evaluación gratuita"). Con notificaciones distintas en un token JWT, el usuario puede ver diferentes contenidos. Por supuesto, para otro contenido o recurso, la restricción ContentKeyPolicyRestriction tendrá el elemento RequiredClaims correspondiente.

Use Azure Media Services API para configurar/clave de licencia de entrega y cifrar los recursos (como se muestra en [este ejemplo](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs)).

Para más información, consulte:

- [Introducción a la protección de contenido](content-protection-overview.md)
- [Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](design-multi-drm-system-with-access-control.md)

## <a name="media-services-v2-vs-v3"></a>Comparación entre las versiones v2 y v3 de Media Services 

### <a name="can-i-use-the-azure-portal-to-manage-v3-resources"></a>¿Puedo usar Azure Portal para administrar los recursos de v3?

Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](media-services-apis-overview.md#sdks) admitidos.

### <a name="is-there-an-assetfile-concept-in-v3"></a>¿Existe el concepto de AssetFile en la versión v3?

Los AssetFiles se eliminaron de la API de AMS para separar la dependencia del SDK de Media Services de Storage. Ahora Storage y no Media Services mantiene la información que pertenece a Storage. 

Para más información, consulte [Migración a Media Services v3](migrate-from-v2-to-v3.md).

### <a name="where-did-client-side-storage-encryption-go"></a>¿Qué hay del cifrado del almacenamiento del lado del cliente?

Actualmente, es recomendable utilizar el cifrado de almacenamiento en el lado de servidor (que está activo de forma predeterminada). Para más información, consulte [Cifrado del servicio Azure Storage para datos en reposo](https://docs.microsoft.com/azure/storage/common/storage-service-encryption).

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services v3](media-services-overview.md)
