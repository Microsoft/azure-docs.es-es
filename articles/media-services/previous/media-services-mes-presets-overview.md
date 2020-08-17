---
title: Valores preestablecidos de las tareas de Media Encoder Standard (MES) | Microsoft Docs
description: En este tema se proporciona información general de los valores preestablecidos de ejemplo definidos por el servicio de Media Encoder Standard (MES).
author: Juliako
manager: femila
editor: johndeu
services: media-services
documentationcenter: ''
ms.assetid: f243ed1c-ac9c-4300-a5f7-f092cf9853b9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 1d264b0ae0b06079a45cd1e862144a6920133901
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038450"
---
# <a name="sample-presets-for-media-encoder-standard-mes"></a>Valores preestablecidos de ejemplo de Media Encoder Standard (MES)

**Media Encoder Standard** define un conjunto de valores preestablecidos de codificación predefinida del sistema que puede usar al crear trabajos de codificación. Se recomienda usar el valor predeterminado recomendado "Adaptive Streaming" si quiere codificar un vídeo para streaming con Media Services. Cuando se especifica este valor preestablecido, Media Encoder Standard [generará automáticamente una escalera de velocidad de bits](media-services-autogen-bitrate-ladder-with-mes.md). 

### <a name="creating-custom-presets-from-samples"></a>Creación de valores preestablecidos a partir de ejemplos
Media Services es totalmente compatible con la personalización de todos los valores preestablecidos para que cubran sus necesidades y requisitos de codificación específicos. Si necesita personalizar un valor preestablecido de codificación, debe empezar por seleccionar uno de los valores preestablecidos del sistema que se proporcionan en esta sección como plantilla para la configuración personalizada. Para obtener explicaciones de lo que cada elemento significa en estos nombres predeterminados y los valores válidos para cada elemento, vea el tema sobre el [esquema de Media Encoder Standard](media-services-mes-schema.md).  
  
> [!NOTE]
>  Cuando se usa un valor preestablecido para codificaciones de 4K, debe obtener el tipo de unidad reservada `S3`. Para obtener más información, consulte [Escalación de Encoding](./media-services-scale-media-processing-overview.md).  

#### <a name="video-rotation-default-setting-in-presets"></a>Configuración predeterminada de rotación de vídeo en valores preestablecidos:
Cuando se trabaja con Media Encoder Standard, la rotación de vídeo está habilitada de forma predeterminada. Si un vídeo se ha grabado en un dispositivo móvil en modo vertical, estos valores preestablecidos lo girará al modo horizontal antes de codificarlo.
 
## <a name="available-presets"></a>Valores predefinidos disponibles: 

 [H264 Multiple Bitrate 1080p Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-1080p-Audio-5.1.md) genera un conjunto de 8 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 6000 kbps y audio AAC 5.1.  
  
 [H264 Multiple Bitrate 1080p](media-services-mes-preset-H264-Multiple-Bitrate-1080p.md) genera un conjunto de 8 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 6000 kbps y audio AAC estéreo.  
  
 [H264 Multiple Bitrate 16x9 for iOS](media-services-mes-preset-H264-Multiple-Bitrate-16x9-for-iOS.md) genera un conjunto de 8 archivos MP4 alineados con GOP, que abarcan desde 200 kbps hasta 8500 kbps y audio AAC estéreo.  
  
 [H264 Multiple Bitrate 16x9 SD Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD-Audio-5.1.md) genera un conjunto de 5 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 1900 kbps y audio AAC 5.1.  
  
 [H264 Multiple Bitrate 16x9 SD](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD.md) genera un conjunto de 5 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 1900 kbps y audio AAC estéreo.  
  
 [H264 Multiple Bitrate 4K Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4K-Audio-5.1.md) genera un conjunto de 12 archivos MP4 alineados con GOP, que abarcan desde 1000 kbps hasta 20000 kbps y AAC 5.1.  
  
 [H264 Multiple Bitrate 4K](media-services-mes-preset-H264-Multiple-Bitrate-4K.md) genera un conjunto de 12 archivos MP4 alineados con GOP, que abarcan desde 1000 kbps hasta 20 000 kbps y audio AAC estéreo.  
  
 [H264 Multiple Bitrate 4x3 for iOS](media-services-mes-preset-H264-Multiple-Bitrate-4x3-for-iOS.md) genera un conjunto de 8 archivos MP4 alineados con GOP, que abarcan desde 200 kbps hasta 8500 kbps y audio AAC estéreo.  
  
 [H264 Multiple Bitrate 4x3 SD Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD-Audio-5.1.md) genera un conjunto de 5 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 1600 kbps y AAC 5.1.  
  
 [H264 Multiple Bitrate 4x3 SD](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD.md) genera un conjunto de 5 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 1600 kbps y audio AAC estéreo.  
  
 [H264 Multiple Bitrate 720p Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-720p-Audio-5.1.md) genera un conjunto de 6 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 3400 kbps y audio AAC 5.1.  
  
 [H264 Multiple Bitrate 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) genera un conjunto de 6 archivos MP4 alineados con GOP, que abarcan desde 400 kbps hasta 3400 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate 1080p Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-1080p-Audio-5.1.md) genera un único archivo MP4 con una velocidad de bits de 6750 kbps y audio AAC 5.1.  
  
 [H264 Single Bitrate 1080p](media-services-mes-preset-H264-Single-Bitrate-1080p.md) genera un único archivo MP4 con una velocidad de bits de 6750 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate 4K Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-4K-Audio-5.1.md) genera un único archivo MP4 con una velocidad de bits de 18 000 kbps y audio AAC 5.1.  
  
 [H264 Single Bitrate 4K](media-services-mes-preset-H264-Single-Bitrate-4K.md) genera un único archivo MP4 con una velocidad de bits de 18 000 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate 4x3 SD Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-4x3-SD-Audio-5.1.md) genera un único archivo MP4 con una velocidad de bits de 1800 kbps y audio AAC 5.1.  
  
 [H264 Single Bitrate 4x3 SD](media-services-mes-preset-H264-Single-Bitrate-4x3-SD.md) genera un único archivo MP4 con una velocidad de bits de 1800 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate 16x9 SD Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-16x9-SD-Audio-5.1.md) genera un único archivo MP4 con una velocidad de bits de 2200 kbps y audio AAC 5.1.  
  
 [H264 Single Bitrate 16x9 SD](media-services-mes-preset-H264-Single-Bitrate-16x9-SD.md) genera un único archivo MP4 con una velocidad de bits de 2200 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate 720p Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-720p-Audio-5.1.md) genera un único archivo MP4 con una velocidad de bits de 4500 kbps y audio AAC 5.1.  
  
 [H264 Single Bitrate 720p for Android](media-services-mes-preset-H264-Single-Bitrate-720p-for-Android.md) genera un único archivo MP4 con una velocidad de bits de 2000 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate 720p](media-services-mes-preset-H264-Single-Bitrate-720p.md) genera un único archivo MP4 con una velocidad de bits de 4500 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate High Quality SD for Android](media-services-mes-preset-H264-Single-Bitrate-High-Quality-SD-for-Android.md) genera un único archivo MP4 con una velocidad de bits de 500 kbps y audio AAC estéreo.  
  
 [H264 Single Bitrate Low Quality SD for Android](media-services-mes-preset-H264-Single-Bitrate-Low-Quality-SD-for-Android.md) genera un único archivo MP4 con una velocidad de bits de 56 kbps y audio AAC estéreo.  
  
 Para obtener más información relacionada con codificadores de Media Services, consulte [Información general y comparación de codificadores multimedia a petición de Azure](./media-services-encode-asset.md).
