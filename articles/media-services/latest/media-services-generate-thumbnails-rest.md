---
title: Generación de miniaturas mediante Encoder Standard de Azure Media Services con REST
description: En este artículo se muestra cómo se usa REST para codificar un recurso y generar miniaturas al mismo tiempo con Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: af320b94950d5999b6dd181b7a8e0eb198088e98
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267639"
---
# <a name="how-to-generate-thumbnails-using-encoder-standard-with-rest"></a>Generación de miniaturas mediante Encoder Standard con REST

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Puede utilizar Media Encoder Standard para generar una o varias vistas en miniatura de la entrada de vídeo en formatos de archivo de imagen [JPEG](https://en.wikipedia.org/wiki/JPEG), [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics), o [BMP](https://en.wikipedia.org/wiki/BMP_file_format).

## <a name="recommended-reading-and-practice"></a>Práctica y lectura recomendadas

Se recomienda que se familiarice con las transformaciones personalizadas leyendo [Cómo codificar con una transformación personalizada: REST](custom-preset-rest-howto.md).

## <a name="thumbnail-parameters"></a>Parámetros de miniatura

Debe establecer los siguientes parámetros:

- **start**: la posición en el vídeo de entrada desde donde comenzar a generar miniaturas. El valor puede estar en formato ISO 8601 (por ejemplo, PT05S para empezar en 5 segundos), ser un número de fotogramas (por ejemplo, 10 para comenzar en el décimo fotograma) o ser un valor relativo a la duración de la secuencia (por ejemplo, 10 % para empezar en el 10 % de la duración de la secuencia). También admite una macro {Best}, que indica al codificador que seleccione la mejor miniatura en los primeros segundos del vídeo y que solo se producirá una miniatura, independientemente de cuál sea la configuración para step y range. El valor predeterminado es la macro {Best}.
- **step**: los intervalos en los que se generan las miniaturas. El valor puede estar en formato ISO 8601 (por ejemplo, PT05S para una imagen cada 5 segundos), ser un número de fotogramas (por ejemplo, 30 para una imagen cada 30 fotogramas) o ser un valor relativo a la duración de la secuencia (por ejemplo, 10 % para una imagen cada 10 % de la duración de la secuencia). El valor de step afectará a la primera miniatura generada, que puede no ser exactamente la especificada en el momento de inicio preestablecido de la transformación. Esto se debe al codificador, que intenta seleccionar la mejor miniatura entre el momento de inicio y la posición del paso desde dicho momento como la primera salida. Dado que el valor predeterminado es 10 %, significa que si la secuencia tiene una duración larga, la primera miniatura generada podría estar lejos del momento de inicio especificado. Intente seleccionar un valor razonable para step si se espera que la primera miniatura esté próxima al momento de inicio, o establezca el valor de range en 1 si solo se necesita una miniatura en el momento de inicio.
- **range**: la posición relativa al momento de inicio preestablecido de la transformación en el vídeo de entrada en la que se dejan de generar miniaturas. El valor puede estar en formato ISO 8601 (por ejemplo, PT5M30S para detenerse a los 5 minutos y 30 segundos desde el momento de inicio), ser un número de fotogramas (por ejemplo, 300 para detenerse en el fotograma 300 desde el fotograma del momento de inicio. Si este valor es 1, significa que solo se genera una miniatura en el momento de inicio) o ser un valor relativo a la duración de la secuencia (por ejemplo, 50 % para detenerse a la mitad de la duración de la secuencia desde el momento de inicio). El valor predeterminado es 100 %, lo que significa que debe detenerse al final de la secuencia.
- **layers**: una colección de capas de imagen de salida que el codificador va a generar.

## <a name="example-of-a-single-png-file-preset"></a>Ejemplo de un valor preestablecido de archivo PNG único

El siguiente valor preestablecido en código JSON puede usarse para generar un único archivo PNG de salida fuera de los primeros segundos de la entrada de vídeo, donde el codificador realiza un intento de mejor esfuerzo para buscar un fotograma "interesante". Tenga en cuenta que las dimensiones de la imagen de salida se han establecido en 100 %, lo que significa que se corresponderán con las dimensiones de la entrada de vídeo. Tenga en cuenta también que es necesaria la configuración de "Format" en "Outputs" para que coincida con el uso de "PngLayers" en la sección "Codecs".  

```json
{
    "properties": {
        "description": "Basic Transform using a custom encoding preset for thumbnails",
        "outputs": [
            {
                "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                "preset": {
                    "@odata.type": "#Microsoft.Media.StandardEncoderPreset",
                    "codecs": [
                        {
                            "@odata.type": "#Microsoft.Media.PngImage",
                            "stretchMode": "AutoSize",
                            "start": "{Best}",
                            "step": "25%",
                            "range": "80%",
                            "layers": [
                                {
                                    "width": "50%",
                                    "height": "50%"
                                }
                            ]
                        }
                    ],
                    "formats": [
                        {
                            "@odata.type": "#Microsoft.Media.Mp4Format",
                            "filenamePattern": "Video-{Basename}-{Label}-{Bitrate}{Extension}",
                            "outputFiles": []
                        },
                        {
                            "@odata.type": "#Microsoft.Media.PngFormat",
                            "filenamePattern": "Thumbnail-{Basename}-{Index}{Extension}"
                        }
                    ]
                }
            }
        ]
    }
}

```

## <a name="example-of-a-series-of-jpeg-images-preset"></a>Ejemplo de un valor preestablecido de una serie de imágenes JPEG

El siguiente valor preestablecido en código JSON puede usarse para generar un conjunto de 10 imágenes en marcas de tiempo del 5 %, 15 %,..., 95 % de la escala de tiempo de entrada y donde se especifica que el tamaño de la imagen sea un cuarto del tamaño de la entrada de vídeo.

### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "25%",
          "Height": "25%"
        }
      ],
      "Start": "5%",
      "Step": "10%",
      "Range": "96%",
      "Type": "JpgImage"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "JpgFormat"
      }
    }
  ]
}
```

## <a name="example-of-a-one-image-at-a-specific-timestamp-preset"></a>Ejemplo de un valor preestablecido de una imagen en una marca de tiempo específica

El siguiente valor preestablecido en código JSON puede usarse para generar una única imagen JPEG en la marca de 30 segundos de la entrada de vídeo. Este valor preestablecido espera que el vídeo de entrada sea de más de 30 segundos de duración (de lo contrario, el trabajo producirá un error).

### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "25%",
          "Height": "25%"
        }
      ],
      "Start": "00:00:30",
      "Step": "1",
      "Range": "1",
      "Type": "JpgImage"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "JpgFormat"
      }
    }
  ]
}
```

## <a name="example-of-a-thumbnails-at-different-resolutions-preset"></a>Ejemplo de un valor preestablecido de miniaturas con distintas resoluciones

El siguiente valor preestablecido puede utilizarse para generar miniaturas con diferentes resoluciones en una tarea. En el ejemplo, en las posiciones 5 %, 15 %, …, 95 % de la escala de tiempo de entrada, el codificador genera dos imágenes: una en la posición 100 % de la resolución del vídeo de entrada y otro en la posición 50 %.

Tenga en cuenta el uso de macro {Resolution} en el nombre de archivo. Indica al codificador que utilice el ancho y alto que especificó en la sección de codificación del valor preestablecido al generar el nombre de archivo de las imágenes de salida. Esto también ayuda a distinguir fácilmente entre las distintas imágenes.

### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "JpgLayers": [
{
  "Quality": 90,
  "Type": "JpgLayer",
  "Width": "100%",
  "Height": "100%"
},
{
  "Quality": 90,
  "Type": "JpgLayer",
  "Width": "50%",
  "Height": "50%"
}

      ],
      "Start": "5%",
      "Step": "10%",
      "Range": "96%",
      "Type": "JpgImage"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Resolution}_{Index}{Extension}",
      "Format": {
"Type": "JpgFormat"
      }
    }
  ]
}
```

## <a name="example-of-generating-a-thumbnail-while-encoding"></a>Ejemplo de cómo generar una miniatura durante la codificación

Aunque todos los ejemplos anteriores han explicado cómo enviar una tarea de codificación que solo generará imágenes, también puede combinar la codificación de audio y vídeo con generación de miniaturas. El siguiente valor preestablecido JSON indica a Encoder Standard que genere una miniatura durante la codificación.

### <a name="json-preset"></a>Valor preestablecido JSON

Para obtener información sobre el esquema, consulte [este](/azure/media-services/previous/media-services-mes-schema) artículo.

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "SceneChangeDetection": "true",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 4500,
          "MaxBitrate": 4500,
          "BufferWindow": "00:00:05",
          "Width": 1280,
          "Height": 720,
          "ReferenceFrames": 3,
          "EntropyMode": "Cabac",
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"

        }
      ],
      "Type": "H264Video"
    },
    {
      "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "100%",
          "Height": "100%"
        }
      ],
      "Start": "{Best}",
      "Type": "JpgImage"
    },
    {
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 128,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "JpgFormat"
      }
    },
    {
      "FileName": "{Basename}_{Resolution}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes
[Generación de miniaturas con .NET](media-services-generate-thumbnails-dotnet.md)
