---
title: Comparación de codificadores multimedia a petición de Azure | Microsoft Docs
description: En esta tema se comparan las funcionalidades de codificación de **Media Encoder Standard** y **Flujo de trabajo premium de Media Encoder**.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: a79437c0-4832-423a-bca8-82632b2c47cc
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.reviewer: anilmur
ms.openlocfilehash: f42361df438a434548b3bc9394c007ef8d4c6eb0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038977"
---
# <a name="comparison-of-azure-on-demand-media-encoders"></a>Comparación de codificadores multimedia a petición de Azure  

En esta tema se comparan las funcionalidades de codificación de **Media Encoder Standard** y **Flujo de trabajo premium de Media Encoder**.

## <a name="video-and-audio-processing-capabilities"></a>Funcionalidades de procesamiento de vídeo y audio

En la tabla siguiente se compara la funcionalidad entre Media Encoder Standard (MES) y el flujo de trabajo de Media Encoder Premium (MEPW). 

|Capacidad|Media Encoder Estándar|Flujo de trabajo del Codificador multimedia|
|---|---|---|
|Aplicación de la lógica condicional durante la codificación<br/>(por ejemplo, si la entrada es HD, entonces, codifique audio 5.1)|No|Sí|
|Subtítulos (CC)|No|[Sí](media-services-premium-workflow-encoder-formats.md#closed_captioning)|
|[Corrección de volumen profesional Dolby&reg;](https://professional.dolby.com/product/broadcast/vm600/)<br/> con Dialogue Intelligence&trade;|No|Sí|
|Deshacer entrelazado, telecine inverso|Básica|Calidad de difusión|
|Detectar y quitar bordes negros <br/>(formato pillarbox y formato letterbox)|No|Sí|
|Generación de miniaturas|[Sí](media-services-dotnet-generate-thumbnail-with-mes.md)|[Sí](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4)|
|Recorte y unión de vídeos|[Sí](media-services-advanced-encoding-with-mes.md#trim_video)|Sí|
|Superposiciones de audio o vídeo|[Sí](media-services-advanced-encoding-with-mes.md#overlay)|[Sí](media-services-media-encoder-premium-workflow-multiplefilesinput.md#example-1--overlay-an-image-on-top-of-the-video)|
|Superposiciones de gráficos|De orígenes de imagen|De orígenes de imagen y texto|
|Varias pistas de idiomas de audio|Limitado|[Sí](media-services-media-encoder-premium-workflow-multiplefilesinput.md#example-2--multiple-audio-language-encoding)|

## <a name="billing-meter-used-by-each-encoder"></a><a id="billing"></a>Medidor de facturación usado por cada codificador
| Nombre de procesador multimedia | Precios aplicables | Notas |
| --- | --- | --- |
| **Media Encoder Standard** |ENCODER |Las tareas de codificación se cobrarán en función de la duración total, en minutos, de todos los archivos multimedia producidos como salida, según la tarifa especificada [aquí][1], bajo la columna ENCODER. |
| **Flujo de trabajo del Codificador multimedia** |CODIFICADOR PREMIUM |Las tareas de codificación se cobrarán en función de la duración total, en minutos, de todos los archivos multimedia producidos como salida, según la tarifa especificada [aquí][1], bajo la columna PREMIUM ENCODER. |

## <a name="input-containerfile-formats"></a>Formatos de archivo/contenedor de entrada
| Formatos de archivo/contenedor de entrada | Media Encoder Estándar | Flujo de trabajo del Codificador multimedia |
| --- | --- | --- |
| Adobe&reg; Flash&reg; F4V |Sí |Sí |
| MXF/SMPTE 377M |Sí |Sí |
| GXF |Sí |Sí |
| Secuencias de transporte MPEG-2 |Sí |Sí |
| Secuencias de programa MPEG-2 |Sí |Sí |
| MPEG-4/MP4 |Sí |Sí |
| Windows Media/ASF |Sí |Sí |
| AVI (sin comprimir de 8 bits/10 bits) |Sí |Sí |
| 3GPP/3GPP2 |Sí |No |
| Formato de archivo de streaming con velocidad de transmisión adaptable (PIFF 1.3) |Sí |No |
| [Microsoft Digital Video Recording(DVR-MS)](/previous-versions/windows/desktop/mstv/about-the-dvr-ms-file-format) |Sí |No |
| Matroska/WebM |Sí |No |
| QuickTime (.mov) |Sí |No |

## <a name="input-video-codecs"></a>Códecs de vídeo de entrada
| Códecs de vídeo de entrada | Media Encoder Estándar | Flujo de trabajo del Codificador multimedia |
| --- | --- | --- |
| AVC 8 bits/10 bits, hasta 4:2:2, incluido AVCIntra |8 bits: 4:2:0 y 4:2:2 |Sí |
| Avid DNxHD (en MXF) |Sí |Sí |
| DVCPro/DVCProHD (en MXF) |Sí |Sí |
| JPEG2000 |Sí |Sí |
| MPEG-2 (hasta 422 Perfil y Nivel alto; incluidas variantes como XDCAM, XDCAM HD, XDCAM IMX, CableLabs&reg; y D10) |Hasta 422 Perfil |Sí |
| MPEG-1 |Sí |Sí |
| Windows Media Video/VC-1 |Sí |Sí |
| Canopus HQ/HQX |No |No |
| MPEG-4, parte 2 |Sí |No |
| [Theora](https://en.wikipedia.org/wiki/Theora) |Sí |No |
| Apple ProRes 422 |Sí |No |
| Apple ProRes 422 LT |Sí |No |
| Apple ProRes 422 HQ |Sí |No |
| Apple ProRes Proxy |Sí |No |
| Apple ProRes 4444 |Sí |No |
| Apple ProRes 4444 XQ |Sí |No |
| HEVC/H.265|Perfil Main|Perfil Main y Main 10|

## <a name="input-audio-codecs"></a>Códecs de audio de entrada
| Códecs de audio de entrada | Media Encoder Estándar | Flujo de trabajo del Codificador multimedia |
| --- | --- | --- |
| AES (SMPTE 331M y 302M, AES3-2003) |No |Sí |
| Dolby&reg; E |No |Sí |
| Dolby&reg; Digital (AC3) |No |Sí |
| Dolby&reg; Digital Plus (E-AC3) |No |Sí |
| AAC (AAC-LC, AAC-HE y AAC-HEv2; hasta 5.1) |Sí |Sí |
| MPEG Layer 2 |Sí |Sí |
| MP3 (MPEG-1 Audio Layer 3) |Sí |Sí |
| Windows Media Audio |Sí |Sí |
| WAV/PCM |Sí |Sí |
| [FLAC](https://en.wikipedia.org/wiki/FLAC)</a> |Sí |No |
| [Opus](https://en.wikipedia.org/wiki/Opus_\(audio_format\)) |Sí |No |
| [Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a> |Sí |No |

## <a name="output-containerfile-formats"></a>Formatos de archivo/contenedor de salida
| Formatos de archivo/contenedor de salida | Media Encoder Estándar | Flujo de trabajo del Codificador multimedia |
| --- | --- | --- |
| Adobe&reg; Flash&reg; F4V |No |Sí |
| MXF (OP1a, XDCAM y AS02) |No |Sí |
| DPP (incluido AS11) |No |Sí |
| GXF |No |Sí |
| MPEG-4/MP4 |Sí |Sí |
| MPEG-TS |Sí |Sí |
| Windows Media/ASF |No |Sí |
| AVI (sin comprimir de 8 bits/10 bits) |No |Sí |
| Formato de archivo de streaming con velocidad de transmisión adaptable (PIFF 1.3) |No |Sí |

## <a name="output-video-codecs"></a>Códecs de vídeo de salida
| Códecs de vídeo de salida | Media Encoder Estándar | Flujo de trabajo del Codificador multimedia |
| --- | --- | --- |
| AVC (H.264; 8 bits; hasta Perfil alto, Nivel 5.2; 4K Ultra HD; AVC Intra) |Solo 8 bits 4:2:0 |Sí |
| HEVC (H.265; 8 bits y 10 bits;)  |No |Sí |
| Avid DNxHD (en MXF) |No |Sí |
| MPEG-2 (hasta 422 Perfil y Nivel alto; incluidas variantes como XDCAM, XDCAM HD, XDCAM IMX, CableLabs&reg; y D10) |No |Sí |
| MPEG-1 |No |Sí |
| Windows Media Video/VC-1 |No |Sí |
| Creación de miniaturas JPEG |Sí |Sí |
| Creación de miniaturas PNG |Sí |Sí |
| Creación de miniaturas BMP |Sí |No |

## <a name="output-audio-codecs"></a>Códecs de audio de salida
| Códecs de audio de salida | Media Encoder Estándar | Flujo de trabajo del Codificador multimedia |
| --- | --- | --- |
| AES (SMPTE 331M y 302M, AES3-2003) |No |Sí |
| Dolby&reg; Digital (AC3) |No |Sí |
| Dolby&reg; Digital Plus (E-AC3) hasta 7.1 |No |Sí |
| AAC (AAC-LC, AAC-HE y AAC-HEv2; hasta 5.1) |Sí |Sí |
| MPEG Layer 2 |No |Sí |
| MP3 (MPEG-1 Audio Layer 3) |No |Sí |
| Windows Media Audio |No |Sí |

>[!NOTE]
>Si utiliza la codificación en Dolby&reg; Digital (AC3), la salida solo puede escribirse en un archivo ISO MP4.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>Artículos relacionados
* [Realización de tareas de codificación avanzadas mediante la personalización de valores preestablecidos de Media Encoder Estándar](media-services-custom-mes-presets-with-dotnet.md)
* [Cuotas y limitaciones](media-services-quotas-and-limitations.md)

<!--Reference links in article-->
[1]: https://azure.microsoft.com/pricing/details/media-services/
