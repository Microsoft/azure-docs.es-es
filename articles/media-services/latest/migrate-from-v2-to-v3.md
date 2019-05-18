---
title: Migración de Azure Media Services v2 a v3 | Microsoft Docs
description: En este artículo se describen los cambios realizados en Azure Media Services v3 y se muestran las diferencias entre las dos versiones. En el artículo también se proporcionan instrucciones de migración para pasar de Media Services v2 a v3.
services: media-services
documentationcenter: na
author: Juliako
manager: femila
editor: ''
tags: ''
keywords: azure media services, transmisión, difusión, en vivo, sin conexión
ms.service: media-services
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 05/01/2019
ms.author: juliako
ms.openlocfilehash: da8d1000ebe2695dbc95d475027722962a3305d9
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65555889"
---
# <a name="migration-guidance-for-moving-from-media-services-v2-to-v3"></a>Guía de migración para mover de Media Services v2 a v3

En este artículo se describen los cambios realizados en Azure Media Services v3, se muestran las diferencias entre las dos versiones y se ofrece orientación sobre la migración.

Si tiene un servicio de vídeo desarrollado actualmente en la parte superior de las [API heredadas de Media Services v2](../previous/media-services-overview.md), debe revisar las siguientes directrices y consideraciones antes de migrar a las API v3. Hay muchas ventajas y características nuevas de la API v3 que mejoran la experiencia del desarrollador y las funcionalidades de Media Services. Sin embargo, como se indica en la sección [Problemas conocidos](#known-issues) de este artículo, también hay algunas limitaciones debido a cambios entre las versiones de API. Se hará el mantenimiento de esta página, ya que el equipo de Media Services realiza mejoras continuas de las API v3 y aborda las deficiencias entre las versiones. 

> [!NOTE]
> Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](media-services-apis-overview.md#sdks) admitidos.

## <a name="benefits-of-media-services-v3"></a>Ventajas de Media Services v3
  
### <a name="api-is-more-approachable"></a>La API es más cercana

*  v3 se basa en una superficie de API unificada que expone la funcionalidad de administración y operaciones creada en Azure Resource Manager. Las plantillas de Azure Resource Manager se pueden usar para crear e implementar transformaciones, puntos de conexión de streaming, Eventos en directo, etc.
* [Especificación de OpenAPI (anteriormente denominado Swagger)](https://aka.ms/ams-v3-rest-sdk) documento.
    Expone el esquema para todos los componentes de servicio, incluida la codificación basada en archivos.
* Diferentes SDK disponibles para [.NET](https://aka.ms/ams-v3-dotnet-ref), .NET Core, [Node.js](https://aka.ms/ams-v3-nodejs-ref), [Python](https://aka.ms/ams-v3-python-ref), [Java](https://aka.ms/ams-v3-java-ref), [Go](https://aka.ms/ams-v3-go-ref) y Ruby.
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

## <a name="changes-from-v2"></a>Cambios desde la versión v2

* En el caso de recursos creados con v3, Media Services solo admite el [cifrado de almacenamiento del lado servidor de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-service-encryption).
    * Puede usar las API v3 con recursos creados con las API v2 que tenían [cifrado de almacenamiento](../previous/media-services-rest-storage-encryption.md) (AES 256) proporcionado por Media Services.
    * No se pueden crear recursos con el [cifrado de almacenamiento](../previous/media-services-rest-storage-encryption.md) AES 256 heredado mediante las API v3.
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
*  Para obtener información acerca de un trabajo, debe conocer el nombre de la transformación en la que se creó el trabajo. 

## <a name="feature-gaps-with-respect-to-v2-apis"></a>Carencias de características con respecto a las API v2

La API v3 tiene las siguientes carencias de características con respecto a la API v2. El trabajo para la solución de deficiencias está en curso.

* El [codificador premium](../previous/media-services-premium-workflow-encoder-formats.md) y los [procesadores de análisis multimedia](../previous/media-services-analytics-overview.md) heredados (Azure Media Services Indexer 2 Preview, Face Redactor, etc.) no son accesibles mediante v3.<br/>Los clientes que deseen migrar de Media Indexer 1 o 2 Preview pueden usar inmediatamente el valor preestablecido de AudioAnalyzer en la API v3.  Este valor preestablecido nuevo contiene más características que la instancia anterior de Media Indexer 1 o 2. 
* Muchos de los [características avanzadas del Codificador multimedia estándar en la versión 2](../previous/media-services-advanced-encoding-with-mes.md) API actualmente no están disponibles en v3, tales como:
    * Recorte (para escenarios en directo y a petición)
    * Unión de recursos
    * Superposiciones
    * Recorte
    * Sprites de miniaturas
    * Insertar una pista de audio silenciosa cuando la entrada no tiene audio
    * Insertar una pista de vídeo cuando la entrada no tiene ningún vídeo
* Los Eventos en directo con transcodificación actualmente no admiten la inserción de careta durante la transmisión y la inserción de anuncios mediante la llamada API. 

> [!NOTE]
> Marque este artículo y siga buscando actualizaciones.
 
## <a name="code-differences"></a>Diferencias de código

En la tabla siguiente se muestran las diferencias de código entre v2 y v3 para escenarios comunes.

|Escenario|API V2|API V3|
|---|---|---|
|Crear un recurso y cargar un archivo |[Ejemplo de .NET v2](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L113)|[Ejemplo de .NET v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L169)|
|Enviar un trabajo|[Ejemplo de .NET v2](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L146)|[Ejemplo de .NET v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L298)<br/><br/>Se explica cómo crear una transformación y después enviar un trabajo.|
|Publicar un recurso con cifrado AES |1. Crear ContentKeyAuthorizationPolicyOption<br/>2. Crear ContentKeyAuthorizationPolicy<br/>3. Crear AssetDeliveryPolicy<br/>4. Crear Asset y cargar contenido o enviar trabajo y usar el recurso de salida<br/>5. Asociar AssetDeliveryPolicy con Asset<br/>6. Crear ContentKey<br/>7. Adjuntar ContentKey a Asset<br/>8. Crear AccessPolicy<br/>9. Crear Locator<br/><br/>[Ejemplo de .NET v2](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L64)|1. Crear ContentKeyPolicy<br/>2. Crear Asset<br/>3. Cargar contenido o usar Asset como JobOutput<br/>4. Creación de un localizador de streaming<br/><br/>[Ejemplo de .NET v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs#L105)|
|Obtener detalles del trabajo y administrar trabajos |[Administrar los trabajos con v2](../previous/media-services-dotnet-manage-entities.md#get-a-job-reference) |[Administrar los trabajos con v3](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L546)|

## <a name="known-issues"></a>Problemas conocidos

* Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API de REST](https://aka.ms/ams-v3-rest-sdk), la CLI o uno de los SDK admitidos.
* Debe aprovisionar unidades reservadas de multimedia (MRU) en su cuenta para controlar la simultaneidad y el rendimiento de sus trabajos, en particular aquellos que están relacionados con el análisis de audio o vídeo. Para más información, consulte [Escalado del procesamiento de elementos multimedia](../previous/media-services-scale-media-processing-overview.md). Puede administrar las MRU con la [CLI 2.0 para Media Services v3](media-reserved-units-cli-how-to.md), [Azure Portal](../previous/media-services-portal-scale-media-processing.md) o las [API de v2](../previous/media-services-dotnet-encoding-units.md). Tenga en cuenta que debe aprovisionar las MRU sin importar si está usando las API de Media Services de la versión 2 o 3.
* Las entidades de Media Services creadas con la API v3 no se pueden administrar con la API v2.  
* No se recomienda administrar las entidades que se crearon con las API v2 mediante las API v3. Estos son algunos ejemplos de las diferencias que hacen que las entidades de las dos versiones sean incompatibles:   
    * Los trabajos y las tareas creados en v2 no se muestran en v3, ya que no están asociados con una transformación. La recomendación es cambiar a transformaciones y trabajos de v3. Habrá un período de tiempo relativamente corto con la necesidad de supervisar los trabajos de v2 en proceso durante el cambio.
    * Los canales y programas creados con v2 (que se asignan a Eventos en directo y Salidas en vivo en v3) no se pueden continuar administrando con v3. La recomendación es cambiar a Eventos en directo y Salidas en vivo en v3 con una detención de canal cómoda.<br/>Actualmente, no puede migrar continuamente los canales en ejecución.  

> [!NOTE]
> Se hará el mantenimiento de esta página, ya que el equipo de Media Services realiza mejoras continuas de las API v3 y aborda las deficiencias entre las versiones.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

Para ver lo fácil que es empezar a codificar y hacer streaming de archivos de vídeo, consulte [Stream files](stream-files-dotnet-quickstart.md) (Streaming de vídeos). 

