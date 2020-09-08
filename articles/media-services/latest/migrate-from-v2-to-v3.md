---
title: Migración de Azure Media Services v2 a v3 | Microsoft Docs
description: En este artículo se describen los cambios realizados en Azure Media Services v3 y se muestran las diferencias entre las dos versiones. En el artículo también se proporcionan instrucciones de migración para pasar de Media Services v2 a v3.
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
ms.openlocfilehash: b4e79a2aab5ca72ff8263bfc5734757bbff41005
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297746"
---
# <a name="migration-guidance-for-moving-from-media-services-v2-to-v3"></a>Guía de migración para mover de Media Services v2 a v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue esta URL (`https://docs.microsoft.com/api/search/rss?search=%22Migrate+from+Azure+Media+Services+v2+to+v3%22&locale=en-us`) en el lector de fuentes RSS.

En este artículo se proporcionan instrucciones de migración para pasar de Media Services v2 a v3.

Si tiene un servicio de vídeo desarrollado actualmente en la parte superior de las [API heredadas de Media Services v2](../previous/media-services-overview.md), debe revisar las siguientes directrices y consideraciones antes de migrar a las API v3. Hay muchas ventajas y características nuevas de la API v3 que mejoran la experiencia del desarrollador y las funcionalidades de Media Services. Sin embargo, como se indica en la sección [Problemas conocidos](#known-issues) de este artículo, también hay algunas limitaciones debido a cambios entre las versiones de API. Se hará el mantenimiento de esta página, ya que el equipo de Media Services realiza mejoras continuas de las API v3 y aborda las deficiencias entre las versiones. 

## <a name="prerequisites"></a>Prerrequisitos

* Revise [Versiones 2 y 3 de Media Services](media-services-v2-vs-v3.md).
* [!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="benefits-of-media-services-v3"></a>Ventajas de Media Services v3
  
### <a name="api-is-more-approachable"></a>La API es más cercana

*  v3 se basa en una superficie de API unificada que expone la funcionalidad de administración y operaciones creada en Azure Resource Manager. Las plantillas de Azure Resource Manager se pueden usar para crear e implementar transformaciones, puntos de conexión de streaming, Eventos en directo, etc.
* Documento [Especificación OpenAPI (anteriormente denominada Swagger)](https://aka.ms/ams-v3-rest-sdk).
    Expone el esquema para todos los componentes de servicio, incluida la codificación basada en archivos.
* Diferentes SDK disponibles para [.NET](https://aka.ms/ams-v3-dotnet-ref), .NET Core, [Node.js](/javascript/api/overview/azure/mediaservices/management), [Python](https://aka.ms/ams-v3-python-ref), [Java](https://aka.ms/ams-v3-java-ref), [Go](https://aka.ms/ams-v3-go-ref) y Ruby.
* La integración de la [CLI de Azure](https://aka.ms/ams-v3-cli-ref) para la compatibilidad de scripting sencilla.

### <a name="new-features"></a>Nuevas características

* Para el procesamiento de trabajos basados en archivos, puede usar una dirección URL de HTTP(S) como entrada.<br/>No es necesario tener contenido ya almacenado en Azure, ni es necesario crear recursos.
* Presenta el concepto de [transformaciones](transforms-jobs-concept.md) para el procesamiento de trabajos basados en archivos. Una transformación puede utilizarse para crear configuraciones reutilizables, crear plantillas de Azure Resource Manager y aislar los valores de procesamiento entre varios clientes o inquilinos.
* Un recurso puede tener varios [Localizadores de streaming](streaming-locators-concept.md) con diferentes configuraciones de [empaquetado dinámico](dynamic-packaging-overview.md) y cifrado dinámico.
* La [protección de contenido](content-key-policy-concept.md) es compatible con características de varias claves.
* Puede transmitir eventos en directo que tengan hasta 24 horas de duración al usar Media Services para transcodificar una fuente de contribución de velocidad de bits única en un flujo de salida que tiene varias velocidades de bits.
* Nueva compatibilidad de streaming en vivo de baja latencia en Eventos en directo. Para más información, consulte [latencia](live-event-latency.md).
* La versión preliminar de Evento en directo admite [empaquetado dinámico](dynamic-packaging-overview.md) y cifrado dinámico. Esto habilita la protección de contenido en la vista previa, así como el empaquetado DASH y HLS.
* Salida en directo es más fácil de usar que la entidad Program de las API v2. 
* Compatibilidad mejorada con RTMP (mayor estabilidad y mejor compatibilidad con codificadores de origen).
* Ingesta segura de RTMPS.<br/>Cuando se crea un evento en directo, el usuario recibe cuatro direcciones URL de ingesta. Las cuatro direcciones URL de ingesta son casi idénticas, tienen el mismo token de streaming (AppId) y solo se diferencian en componente de número de puerto. Dos de las direcciones URL son principal y de respaldo para RTMPS.   
* Tiene control de acceso basado en roles (RBAC) en las entidades. 

## <a name="known-issues"></a>Problemas conocidos

*  Actualmente, se puede usar [Azure Portal](https://portal.azure.com/) para:

    * administrar los [eventos en directo](live-events-outputs-concept.md) de la versión 3 de Media Services, 
    * ver (no administrar) los [recursos](assets-concept.md) de la versión 3, 
    * [obtener información sobre el acceso a las API](./access-api-howto.md). 

    Para las restantes tareas de administración (por ejemplo, [Transformaciones y trabajos](transforms-jobs-concept.md) y [Protección de contenido](content-protection-overview.md)), use la [API REST](/rest/api/media/), la [CLI](https://aka.ms/ams-v3-cli-ref), o uno de los[SDK](media-services-apis-overview.md#sdks) compatibles.
* Debe aprovisionar unidades reservadas de multimedia (MRU) en su cuenta para controlar la simultaneidad y el rendimiento de sus trabajos, en particular aquellos que están relacionados con el análisis de audio o vídeo. Para más información, consulte [Escalado del procesamiento de elementos multimedia](../previous/media-services-scale-media-processing-overview.md). Puede administrar las MRU con la [CLI 2.0 para Media Services v3](media-reserved-units-cli-how-to.md), [Azure Portal](../previous/media-services-portal-scale-media-processing.md) o las [API de v2](../previous/media-services-dotnet-encoding-units.md). Tenga en cuenta que debe aprovisionar las MRU sin importar si está usando las API de Media Services de la versión 2 o 3.
* Las entidades de Media Services creadas con la API v3 no se pueden administrar con la API v2.  
* No todas las entidades de la API V2 se muestran automáticamente en la API V3.  A continuación se muestran ejemplos de entidades de las dos versiones que no son compatibles:  
    * Los trabajos y las tareas creados en v2 no se muestran en v3, ya que no están asociados con una transformación. La recomendación es cambiar a transformaciones y trabajos de v3. Habrá un período de tiempo relativamente corto con la necesidad de supervisar los trabajos de v2 en proceso durante el cambio.
    * Los canales y programas creados con v2 (que se asignan a Eventos en directo y Salidas en vivo en v3) no se pueden continuar administrando con v3. La recomendación es cambiar a Eventos en directo y Salidas en vivo en v3 con una detención de canal cómoda.<br/>Actualmente, no puede migrar continuamente los canales en ejecución.  

> [!NOTE]
> Se hará el mantenimiento de esta página, ya que el equipo de Media Services realiza mejoras continuas de las API v3 y aborda las deficiencias entre las versiones.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: Codificación de un archivo remoto según una dirección URL y transmisión del vídeo: .NET](stream-files-dotnet-quickstart.md)
