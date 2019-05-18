---
title: 'Codificación en la nube con Media Services: Azure | Microsoft Docs'
description: Este tema describe el proceso de codificación con Azure Media Services
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/10/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 25b3209bed98ea217db9e414caa6f08cee6d8c89
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761880"
---
# <a name="encoding-with-media-services"></a>Codificación con Media Services

La codificación de término en Media Services se aplica al proceso de conversión de archivos que contienen vídeo digital y audio de un formato estándar a otro, con el objetivo de (a) lo que reduce el tamaño de los archivos, o (b) producen un formato que sea compatible con un amplia gama de dispositivos y aplicaciones. Este proceso también se conoce como compresión de vídeo o transcodificación. Consulte la [compresión de datos](https://en.wikipedia.org/wiki/Data_compression) y [¿qué es la codificación y transcodificación?](https://www.streamingmedia.com/Articles/Editorial/What-Is-/What-Is-Encoding-and-Transcoding-75025.aspx) para obtener más información sobre los conceptos.

Normalmente se entregan vídeos para dispositivos y aplicaciones por [descarga progresiva](https://en.wikipedia.org/wiki/Progressive_download) o a través [streaming de velocidad de bits adaptativa](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming). 

* Para la entrega mediante descarga progresiva, puede usar Azure Media Services para convertir un el archivo multimedia digital (intermedio) en un [MP4](https://en.wikipedia.org/wiki/MPEG-4_Part_14) archivo que contiene el vídeo que se ha codificado con el [H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC) códec, y audio que se ha codificado con el [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding) códec. Este archivo MP4 se escribe en un recurso de la cuenta de almacenamiento. Puede usar los SDK o API de Azure Storage (por ejemplo, [API de REST de almacenamiento](../../storage/common/storage-rest-api-auth.md), [SDK de JAVA](../../storage/blobs/storage-quickstart-blobs-java-v10.md), o [.NET SDK](../../storage/blobs/storage-quickstart-blobs-dotnet.md)) para descargar el archivo directamente. Si ha creado la salida de un recurso con un nombre de contenedor específico en el almacenamiento, usar esa ubicación. En caso contrario, puede usar Media Services para [mostrará las direcciones URL de contenedor de activos](https://docs.microsoft.com/rest/api/media/assets/listcontainersas). 
* Para preparar el contenido para la entrega por streaming de velocidad de bits adaptable, el archivo mezzanine debe codificarse en varias velocidades de bits (de mayor a menor). Para garantizar una transición correcta de calidad, según se reduce la velocidad de bits, por lo que es la resolución del vídeo. Esto da como resultado una escalera de codificación llamada: una tabla de resoluciones y velocidades de bits (consulte [escalera de velocidad de bits adaptativa generada automáticamente](autogen-bitrate-ladder.md)). Puede usar Media Services para codificar los archivos intermedios en varias velocidades de bits, al hacerlo, obtendrá un conjunto de archivos MP4 y streaming configuración archivos asociados, escritos en un recurso de la cuenta de almacenamiento. A continuación, puede usar el [empaquetado dinámico](dynamic-packaging-overview.md) capacidad en Media Services para entregar el vídeo a través de protocolos de streaming como [MPEG-DASH](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) y [HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming). Esto requiere crear un [localizador de Streaming](streaming-locators-concept.md) y generar direcciones URL correspondiente a los protocolos admitidos, que, a continuación, se pueden entregar a los dispositivos o aplicaciones basadas en sus funcionalidades de streaming.

El siguiente diagrama muestra el flujo de trabajo para la codificación y a petición con el empaquetado dinámico.

![Empaquetado dinámico](./media/dynamic-packaging-overview/media-services-dynamic-packaging.png)

En este tema se proporcionan instrucciones sobre cómo codificar el contenido con Media Services v3.

## <a name="transforms-and-jobs"></a>Transformaciones y trabajos

Para codificar con Media Services v3, debe crear una [transformación](https://docs.microsoft.com/rest/api/media/transforms) y un [trabajo](https://docs.microsoft.com/rest/api/media/jobs). La transformación define una receta para la configuración de codificación y las salidas; el trabajo es una instancia de la receta. Para más información, consulte [Transformaciones y trabajos](transforms-jobs-concept.md)

Al codificar con Media Services, se usan valores preestablecidos para indicar al codificador cómo se deben procesar los archivos multimedia de entrada. Por ejemplo, puede especificar la resolución de vídeo o el número de canales de audio que desea en el contenido codificado. 

Puede empezar a trabajar rápidamente con alguno de los valores preestablecidos integrados recomendados basados en los procedimientos recomendados del sector o puede optar por crear un valor preestablecido destinado a su escenario específico o los requisitos de dispositivos. Para más información, vea [Procedimiento de codificación con una transformación personalizada](customize-encoder-presets-how-to.md). 

A partir de enero de 2019, al codificar con Media Encoder Standard para generar archivos MP4, se genera un archivo .mpi nuevo y se agrega a la salida de activos. Este archivo MPI está diseñado para mejorar el rendimiento de escenarios de streaming y [empaquetado dinámico](dynamic-packaging-overview.md).

> [!NOTE]
> No debe modificar ni quitar el archivo MPI, así como tampoco tener ninguna dependencia en el servicio en la existencia (o no) de este tipo de archivo.

## <a name="built-in-presets"></a>Valores preestablecidos integrados

Media Services admite actualmente los siguientes valores preestablecidos de codificación integrados:  

### <a name="builtinstandardencoderpreset-preset"></a>Valor preestablecido BuiltInStandardEncoderPreset

[BuiltInStandardEncoderPreset](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) se usa para establecer un valor preestablecido integrado para codificar el vídeo de entrada con Standard Encoder. 

Actualmente se admiten los valores preestablecidos siguientes:

- **EncoderNamedPreset.AACGoodQualityAudio**: genera un solo archivo MP4 que contiene únicamente audio estéreo codificado a 192 kbps.
- **EncoderNamedPreset.AdaptiveStreaming** (recomendado). Para más información, consulte [Generación automática de una escala de velocidad de bits](autogen-bitrate-ladder.md).
- **EncoderNamedPreset.ContentAwareEncodingExperimental** -expone un ajuste preestablecido experimental para la codificación según el contenido. Dado que cualquier contenido de entrada, el servicio intenta determinar automáticamente el número óptimo de capas, velocidad de bits adecuado y configuración de resolución para la entrega mediante transmisión por secuencias adaptativa. Los algoritmos subyacentes continuará evolucionando con el tiempo. La salida contendrá archivos MP4 con vídeo y audio intercalados. Para obtener más información, consulte [Experimental valor preestablecido de codificación compatible con contenido](cae-experimental.md).
- **EncoderNamedPreset.H264MultipleBitrate1080p** genera un conjunto de 8 archivos MP4 alineados con GOP, que abarcan desde 6000 kbps hasta 400 kbps y audio AAC estéreo. La resolución comienza en 1080p y baja hasta 360p.
- **EncoderNamedPreset.H264MultipleBitrate720p** genera un conjunto de 6 archivos MP4 alineados con GOP, que abarcan desde 3400 kbps hasta 400 kbps y audio AAC estéreo. La resolución comienza en 720p y baja hasta 360p.
- **EncoderNamedPreset.H264MultipleBitrateSD** genera un conjunto de 5 archivos MP4 alineados con GOP, que abarcan desde 1600 kbps hasta 400 kbps y audio AAC estéreo. La resolución comienza en 480p y baja hasta 360p.
- **EncoderNamedPreset.H264SingleBitrate1080p** -genera un archivo MP4, donde el vídeo codificado con códecs H.264 a 6750 kbps y un alto de imagen de 1080 píxeles y el audio estéreo codificado con el códec AAC-LC a 64 kbps.
- **EncoderNamedPreset.H264SingleBitrate720p** -genera un archivo MP4, donde el vídeo codificado con códecs H.264 a 4500 kbps y un alto de imagen de 720 píxeles y el audio estéreo codificado con el códec AAC-LC a 64 kbps.
- **EncoderNamedPreset.H264SingleBitrateSD** -genera un archivo MP4, donde el vídeo codificado con códecs H.264 a 2200 kbps y un alto de 480 píxeles de la imagen y el audio estéreo codificado con el códec AAC-LC a 64 kbps.

Para ver la lista más actualizada de los valores preestablecidos, consulte [valores preestablecidos integrados que se utilizará para codificar vídeos](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#encodernamedpreset).

Para ver cómo se usan los valores preestablecidos, consulte [cargando, codificación y streaming de archivos](stream-files-tutorial-with-api.md).

### <a name="standardencoderpreset-preset"></a>Valor preestablecido StandardEncoderPreset

[StandardEncoderPreset](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#standardencoderpreset) describe la configuración que se utilizará al codificar el vídeo de entrada con Standard Encoder. Utilice este valor preestablecido al personalizar los valores preestablecidos de transformación. 

#### <a name="considerations"></a>Consideraciones

Al crear valores preestablecidos personalizados, se aplican las consideraciones siguientes:

- Todos los valores correspondientes al alto y ancho del contenido de AVC deben ser un múltiplo de 4.
- En Azure Media Services v3, todas las velocidades de bits de codificación son en bits por segundo. Esto es diferente de los valores preestablecidos con nuestras API v2, que usa kilobits por segundo como la unidad. Por ejemplo, si la velocidad de bits en la versión 2 se especificó como 128 (kilobits/segundo), en v3 se establecería en 128000 (bits/segundo).

#### <a name="examples"></a>Ejemplos

Media Services es totalmente compatible con la personalización de todos los valores preestablecidos para que cubran sus necesidades y requisitos de codificación específicos. Para obtener ejemplos que muestran cómo personalizar los valores preestablecidos del codificador, vea:

- [Personalizar valores preestablecidos con .NET](customize-encoder-presets-how-to.md)
- [Personalizar valores preestablecidos con CLI](custom-preset-cli-howto.md)
- [Personalizar valores preestablecidos con REST](custom-preset-rest-howto.md)

## <a name="preset-schema"></a>Valor preestablecido de esquema

En Media Services v3, los valores preestablecidos son entidades fuertemente tipadas en la propia API. Puede encontrar la definición de "schema" para estos objetos en [Open API Specification (o Swagger)](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01). También puede ver las definiciones preestablecidas (como **StandardEncoderPreset**) en el [API de REST](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#standardencoderpreset), [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.standardencoderpreset?view=azure-dotnet) (o en otra documentación de referencia SDK de Media Services v3).

## <a name="scaling-encoding-in-v3"></a>Escalado de la codificación en v3

Para escalar el procesamiento multimedia, consulte [escala con CLI](media-reserved-units-cli-how-to.md).

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* [Codificar desde una dirección URL de HTTPS con los valores preestablecidos integrados](job-input-from-http-how-to.md)
* [Codificar un archivo local con los valores preestablecidos integrados](job-input-from-local-file-how-to.md)
* [Crear un valor predeterminado para sus requisitos específicos de escenario o dispositivo de destino personalizado](customize-encoder-presets-how-to.md)
* [Carga, codificación y transmisión con Media Services](stream-files-tutorial-with-api.md)
