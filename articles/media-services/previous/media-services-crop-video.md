---
title: Recorte de vídeos con Media Encoder Standard - Azure | Microsoft Docs
description: Recortar es el proceso que consiste en la selección de una ventana rectangular en el fotograma de vídeo y la codificación solo de los píxeles que están dentro de esa ventana. En este artículo se muestra cómo recortar vídeos con Media Encoder Standard.
services: media-services
documentationcenter: ''
author: anilmur
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: anilmur
ms.reviewer: juliako
ms.openlocfilehash: 4264d1c0d83f14da02b26107d336521250fcf13b
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89257864"
---
# <a name="crop-videos-with-media-encoder-standard"></a>Recorte de vídeos con Codificador multimedia estándar

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

Puede utilizar el Codificador multimedia estándar (MES) para recortar el vídeo de entrada. Recortar es el proceso que consiste en la selección de una ventana rectangular en el fotograma de vídeo y la codificación solo de los píxeles que están dentro de esa ventana. El diagrama siguiente puede ayudarle a ilustrar el proceso.

![Recortar un vídeo](./media/media-services-crop-video/media-services-crop-video01.png)

Suponga que tiene como entrada un vídeo con una resolución de 1920 x 1080 píxeles (relación de aspecto 16:9), pero que tiene barras negras (formato pillarbox) a izquierda y derecha, por lo que sólo una ventana de 4:3 o 1440 x 1080 píxeles contiene vídeo activo. Puede usar el MES para recortar o editar las barras negras y codificar la región de 1440 x 1080 píxeles.

Recortar en MES es una fase previa al procesamiento, por ello, los parámetros de recorte del valor predeterminado de codificación se aplican al vídeo de entrada original. La codificación es una fase posterior y los valores de anchura y altura se aplicarán al vídeo *previamente procesado* y no al vídeo original. Al diseñar el valor predeterminado debe hacer lo siguiente: (a) seleccione los parámetros de recorte basándose en el vídeo de entrada original y (b), seleccione la configuración de codificación basándose en el vídeo recortado. Si no coincide la configuración de la codificación con el vídeo recortado, el resultado no será el esperado.

El [siguiente](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet) tema muestra cómo crear un trabajo de codificación con el MES y cómo especificar un valor predeterminado personalizado para la tarea de codificación. 

## <a name="creating-a-custom-preset"></a>Creación de un valor predeterminado personalizado
En el ejemplo que aparece en el diagrama:

1. La entrada original es 1920 x 1080
2. Debe recortarse a una salida de 1440 x 1080, centrada en el marco de entrada
3. Esto significa un desplazamiento X (1920 – 1440) / 2 = 240 y un desplazamiento Y de cero
4. La anchura y altura del rectángulo de recorte son 1440 y 1080, respectivamente
5. En la fase de codificación, la ASK producirá tres capas, con resoluciones de 1440 x 1080, 960 x 720 y 480 x 360, respectivamente

### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "Crop": {
          "X": 240,
          "Y": 0,
          "Width": 1440,
          "Height": 1080
        }
      },
      "Pad": true
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 3400,
          "MaxBitrate": 3400,
          "BufferWindow": "00:00:05",
          "Width": 1440,
          "Height": 1080,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 2250,
          "MaxBitrate": 2250,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 720,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1250,
          "MaxBitrate": 1250,
          "BufferWindow": "00:00:05",
          "Width": 480,
          "Height": 360,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
    },
    {
      "Profile": "AACLC",
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 128,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="restrictions-on-cropping"></a>Restricciones en el recorte
La función de recorte está pensada para ser manual. Debe cargar el vídeo de entrada en una herramienta de edición apropiada que le permita seleccionar los fotogramas de interés, colocar el cursor para determinar los desplazamientos del rectángulo de recorte y determinar el valor predeterminado de codificación que se ajusta a ese determinado vídeo, etc. Esta característica no está pensada para habilitar cosas como: detección automática y eliminación de bordes negros en formato letterbox o pillarbox en el vídeo de entrada.

Se aplican las siguientes restricciones a la característica de recorte. Si estos no se cumplen, la tarea de codificación puede producir un error o generar una salida inesperada.

1. Las coordenadas y el tamaño del rectángulo de recorte deben ajustarse al vídeo de entrada
2. Como se mencionó anteriormente, la anchura y altura de la configuración de codificación se tienen que corresponder con el vídeo recortado
3. El recorte se aplica a vídeos capturados en modo horizontal (es decir, no es aplicable a los vídeos grabados con un smartphone sujeto verticalmente o en modo vertical)
4. Funciona mejor con vídeo progresivo capturado con píxeles cuadrados

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-step"></a>Paso siguiente
Consulte las rutas de aprendizaje de Azure Media Services para conocer las magníficas características ofrecidas por AMS.  

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
