---
title: Codificar la transformación personalizada mediante la CLI de Media Services v3 - Azure | Microsoft Docs
description: En este tema se muestra cómo usar Azure Media Services v3 para codificar una transformación personalizada mediante la CLI.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 05/14/2019
ms.author: juliako
ms.openlocfilehash: 42b7c2d86525c428253137b424fe58bb61edba70
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65762026"
---
# <a name="how-to-encode-with-a-custom-transform---cli"></a>Cómo codificar con una transformación personalizada - CLI

Al codificar con Azure Media Services, puede empezar a trabajar rápidamente con uno de los valores integrados preestablecidos recomendados según los procedimientos recomendados del sector, como se muestra en el [Streaming archivos](stream-files-cli-quickstart.md#create-a-transform-for-adaptive-bitrate-encoding) Guía de inicio rápido. También puede crear un valor predeterminado para sus requisitos específicos de escenario o dispositivo de destino personalizado.

## <a name="considerations"></a>Consideraciones

Al crear valores preestablecidos personalizados, se aplican las consideraciones siguientes:

* Todos los valores correspondientes al alto y ancho del contenido de AVC deben ser un múltiplo de 4.
* En Azure Media Services v3, todas las velocidades de bits de codificación son en bits por segundo. Esto es diferente de los valores preestablecidos con nuestras API v2, que usa kilobits por segundo como la unidad. Por ejemplo, si la velocidad de bits en la versión 2 se especificó como 128 (kilobits/segundo), en v3 se establecería en 128000 (bits/segundo).

## <a name="prerequisites"></a>Requisitos previos 

[Cree una cuenta de Media Services](create-account-cli-how-to.md). <br/>Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services. 

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="define-a-custom-preset"></a>Definir un valor preestablecido personalizado

El ejemplo siguiente define el cuerpo de solicitud de una nueva transformación. Definimos un conjunto de resultados que se va a generarse cuando se usa esta transformación. 

En este ejemplo, en primer lugar agregamos una capa AacAudio para la codificación de audio y dos capas de H264Video para la codificación de vídeo. En las capas de vídeo, asignamos las etiquetas para que se pueden usar en los nombres de archivo de salida. A continuación, queremos que la salida para incluir también las miniaturas. En el ejemplo siguiente se especifican las imágenes en formato PNG, generada al 50% de la resolución de la entrada de vídeo y en tres marcas de tiempo - {25%, 50%, 75} de la longitud de la entrada de vídeo. Por último, especificamos el formato de los archivos de salida: uno para vídeo y audio y otro para las vistas en miniatura. Puesto que tenemos varios H264Layers, tenemos que usar las macros que generan nombres únicos por nivel. Se puede usar un `{Label}` o `{Bitrate}` macro, el ejemplo muestra la primera de ellas.

Vamos a guardar esta transformación en un archivo. En este ejemplo, usamos el archivo `customPreset.json`. 

```json
{
    "@odata.type": "#Microsoft.Media.StandardEncoderPreset",
    "codecs": [
        {
            "@odata.type": "#Microsoft.Media.AacAudio",
            "channels": 2,
            "samplingRate": 48000,
            "bitrate": 128000,
            "profile": "AacLc"
        },
        {
            "@odata.type": "#Microsoft.Media.H264Video",
            "keyFrameInterval": "PT2S",
            "stretchMode": "AutoSize",
            "sceneChangeDetection": false,
            "complexity": "Balanced",
            "layers": [
                {
                    "width": "1280",
                    "height": "720",
                    "label": "HD",
                    "bitrate": 3400000,
                    "maxBitrate": 3400000,
                    "bFrames": 3,
                    "slices": 0,
                    "adaptiveBFrame": true,
                    "profile": "Auto",
                    "level": "auto",
                    "bufferWindow": "PT5S",
                    "referenceFrames": 3,
                    "entropyMode": "Cabac"
                },
                {
                    "width": "640",
                    "height": "360",
                    "label": "SD",
                    "bitrate": 1000000,
                    "maxBitrate": 1000000,
                    "bFrames": 3,
                    "slices": 0,
                    "adaptiveBFrame": true,
                    "profile": "Auto",
                    "level": "auto",
                    "bufferWindow": "PT5S",
                    "referenceFrames": 3,
                    "entropyMode": "Cabac"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Media.PngImage",
            "stretchMode": "AutoSize",
            "start": "25%",
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

```

## <a name="create-a-new-transform"></a>Crear una nueva transformación  

En este ejemplo, creamos un **transformar** que se basa en los valores preestablecidos personalizados que definimos anteriormente. Al crear una transformación, debe comprobar primero si ya existe uno. Si existe la transformación, volver a usarla. La siguiente `show` comando devuelve el `customTransformName` transformar si existe:

```cli
az ams transform show -a amsaccount -g amsResourceGroup -n customTransformName
```

El siguiente comando CLI crea la transformación según el valor preestablecido personalizado (definido anteriormente). 

```cli
az ams transform create -a amsaccount -g amsResourceGroup -n customTransformName --description "Basic Transform using a custom encoding preset" --preset customPreset.json
```

Media Services aplicar la transformación al audio o vídeo especificado, deberá enviar a un trabajo en esa transformación. Para obtener un ejemplo completo que muestra cómo enviar un trabajo en una transformación, consulte [inicio rápido: Archivos de vídeo: CLI de Stream](stream-files-cli-quickstart.md).

## <a name="see-also"></a>Vea también

[CLI de Azure](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
