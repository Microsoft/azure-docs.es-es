---
title: 'Códecs y formatos de Media Encoder Standard: Azure'
description: En este artículo se ofrece información general sobre los códecs y formatos de Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.reviewer: anilmur
ms.openlocfilehash: 78236a334b6c75f823819c70c0cdbb75bb30191d
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89257439"
---
# <a name="media-encoder-standard-formats-and-codecs"></a>Códecs y formatos de Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Seleccione la versión de Media Services que se está usando:"]
> * [Versión 2](media-services-media-encoder-standard-formats.md)
> * [Versión 3](../latest/media-encoder-standard-formats.md)

Este documento contiene una lista de los formatos de archivo de importación y exportación más comunes que puede usar con Media Encoder Standard.

## <a name="input-containerfile-formats"></a>Formatos de archivo/contenedor de entrada
| Formatos de archivo (extensiones de archivo) | Compatible |
| --- | --- |
| FLV (con códecs H.264 y AAC) (.flv) |Sí |
| MXF    (.mxf) |Sí |
| GXF    (.gxf) |Sí |
| MPEG2-PS, MPEG2-TS, 3GP (.ts, .ps, .3gp, .3gpp, .mpg) |Sí |
| Windows Media Video (WMV)/ASF (.wmv, .asf) |Sí |
| AVI (sin comprimir de 8 bits/10 bits) (.avi) |Sí |
| MP4 (.mp4, .m4a, .m4v)/ISMV (.isma, .ismv) |Sí |
| [Microsoft Digital Video Recording(DVR-MS)](/previous-versions/windows/desktop/mstv/about-the-dvr-ms-file-format) (.dvr-ms) |Sí |
| Matroska/WebM (.mkv) |Sí |
| WAVE/WAV (.wav) |Sí |
| QuickTime (.mov) |Sí |

> [!NOTE]
> La lista anterior contiene las extensiones de archivo más habituales. El Codificador multimedia estándar admite muchos más (por ejemplo: .m2ts, .mpeg2video, .qt). Si intenta codificar un archivo y recibe un mensaje de error que indica que no se admite el formato, notifíquelo [aquí](https://feedback.azure.com/forums/169396-media-services/category/144411-encoding-and-processing/).
> 
> 

### <a name="audio-formats-in-input-containers"></a>Formatos de audio en contenedores de entrada
Media Encoder Standard admite el transporte de los siguientes formatos de audio en contenedores de entrada:

* Archivos MXF, GXF y QuickTime que tengan pistas de audio con muestras de estéreo entrelazado o 5.1

or

* Archivos MXF, GXF y QuickTime donde el audio se transporte como pistas PCM independientes, pero la asignación de canal (a estéreo o 5.1) se pueda deducir de los metadatos del archivo

## <a name="input-video-codecs"></a>Códecs de vídeo de entrada
| Códecs de vídeo de entrada | Compatible |
| --- | --- |
| AVC 8 bits/10 bits, hasta 4:2:2, incluido AVCIntra |8 bits: 4:2:0 y 4:2:2 |
| Avid DNxHD (en MXF) |Sí |
| DVCPro/DVCProHD (en MXF) |Sí |
| Vídeo digital (DV) (en archivos AVI) |Sí |
| JPEG 2000 |Sí |
| MPEG-2 (hasta 422 Perfil y Nivel alto; incluidas variantes como XDCAM, XDCAM HD, XDCAM IMX, CableLabs® y D10) |Hasta 422 Perfil |
| MPEG-1 |Sí |
| VC-1/WMV9 |Sí |
| Canopus HQ/HQX |No |
| MPEG-4, parte 2 |Sí |
| [Theora](https://en.wikipedia.org/wiki/Theora) |Sí |
| YUV420 sin comprimir o intermedio |Sí |
| Apple ProRes 422 |Sí |
| Apple ProRes 422 LT |Sí |
| Apple ProRes 422 HQ |Sí |
| Apple ProRes Proxy |Sí |
| Apple ProRes 4444 |Sí |
| Apple ProRes 4444 XQ |Sí |
| HEVC/H.265| Perfiles Main y Main 10 (&#42;)<br/>La compatibilidad con el perfil Main 10 está destinada a contenido 4:2:0 de 8 bits. |

## <a name="input-audio-codecs"></a>Códecs de audio de entrada
| Códecs de audio de entrada | Compatible |
| --- | --- |
| AAC (AAC-LC, AAC-HE y AAC-HEv2; hasta 5.1) |Sí |
| MPEG Layer 2 |Sí |
| MP3 (MPEG-1 Audio Layer 3) |Sí |
| Windows Media Audio |Sí |
| WAV/PCM |Sí |
| [FLAC](https://en.wikipedia.org/wiki/FLAC)</a> |Sí |
| [Opus](https://go.microsoft.com/fwlink/?LinkId=822667) |Sí |
| [Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a> |Sí |
| AMR (velocidad múltiple adaptable) |Sí |
| AES (SMPTE 331M y 302M, AES3-2003) |No |
| Dolby® E |No |
| Dolby® Digital (AC3) |No |
| Dolby® Digital Plus (E-AC3) |No |

## <a name="output-formats-and-codecs"></a>Códecs y formatos de salida
La siguiente tabla enumera los códecs y los formatos de archivo compatibles para exportación.

| Formato de archivo | Códec de vídeo | Códec de audio |
| --- | --- | --- |
| MP4 <br/><br/>(incluidos los contenedores de MP4 de velocidad de bits múltiple) |H.264 (perfil alto, perfil principal y perfil de base de línea) |AAC-LC, HE-AAC v1, HE-AAC v2 |
| MPEG2-TS |H.264 (perfil alto, perfil principal y perfil de base de línea) |AAC-LC, HE-AAC v1, HE-AAC v2 |

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Consulte también
[Codificación de contenido a petición con Azure Media Services](media-services-encode-asset.md)

[Codificación con Codificador multimedia estándar](media-services-dotnet-encode-with-media-encoder-standard.md)
