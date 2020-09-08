---
title: Codificación de una transformación personalizada con la CLI de Azure Media Services v3 | Microsoft Docs
description: En este tema se explica cómo usar Azure Media Services v3 para codificar una transformación personalizada mediante la CLI de Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: devx-track-azurecli
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c61ac8c7cf85b1fae4c9b531f16b951dd2b80876
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295792"
---
# <a name="how-to-encode-with-a-custom-transform---azure-cli"></a>Procedimiento de codificación con una transformación personalizada: CLI de Azure

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Al codificar con Azure Media Services, puede empezar a trabajar rápidamente con cualquiera de los valores preestablecidos integrados recomendados basados en los procedimientos recomendados del sector, como se muestra en el inicio rápido [Transmisión de archivos](stream-files-cli-quickstart.md#create-a-transform-for-adaptive-bitrate-encoding). También puede compilar un valor preestablecido personalizado para sus requisitos específicos de escenario o dispositivo.

## <a name="considerations"></a>Consideraciones

Al crear valores preestablecidos personalizados, se aplican las consideraciones siguientes:

* Todos los valores de alto y ancho del contenido de AVC deben ser un múltiplo de 4.
* En Azure Media Services v3, todas las velocidades de bits de codificación se expresan en bits por segundo. Esto difiere de los valores preestablecidos en las API v2, en donde se usaban kilobits por segundo como unidad. Por ejemplo, si la velocidad de bits de v2 se especificaba como 128 (kilobits/segundo), en v3 se establecería en 128 000 (bits/segundo).

## <a name="prerequisites"></a>Prerrequisitos

[Cree una cuenta de Media Services](./create-account-howto.md).

Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services.

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="define-a-custom-preset"></a>Definir un valor preestablecido personalizado

En el ejemplo siguiente se define el cuerpo de la solicitud de una nueva transformación. Se define un conjunto de salidas que queremos que se genere cuando se use esta transformación.

En primer lugar, se agrega una capa AacAudio para la codificación de audio y dos capas H264Video para la codificación de vídeo. En las capas de vídeo se asignan etiquetas para que se puedan usar en los nombres de archivo de salida. Luego queremos que la salida también incluya miniaturas. En el ejemplo siguiente se especifican imágenes en formato PNG generadas al 50 % de la resolución del vídeo de entrada, y en tres marcas de tiempo, {25 %, 50 %, 75 %}, de la longitud del vídeo de entrada. Por último, se especifica el formato de los archivos de salida: uno para vídeo y audio y otro para las miniaturas. Puesto que hay varias H264Layers, tenemos que usar macros que generen nombres únicos por capa. Se puede usar una macro `{Label}` o `{Bitrate}`; el ejemplo muestra la primera.

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

En este ejemplo, creamos una **transformación** que se basa en los valores preestablecidos personalizados que definimos anteriormente. Al crear una transformación, primero debe comprobar si ya existe alguna. Si existe la transformación, vuelva a usarla. El siguiente comando `show` devuelve la transformación `customTransformName`, si existe:

```azurecli-interactive
az ams transform show -a amsaccount -g amsResourceGroup -n customTransformName
```

El siguiente comando de la CLI de Azure crea la transformación según el valor preestablecido personalizado (definido anteriormente).

```azurecli-interactive
az ams transform create -a amsaccount -g amsResourceGroup -n customTransformName --description "Basic Transform using a custom encoding preset" --preset customPreset.json
```

Para que Media Services aplique la transformación al audio o vídeo especificado, deberá enviar un trabajo en esa transformación. Para obtener un ejemplo completo que muestra cómo enviar un trabajo en una transformación, consulte [Inicio rápido: Hacer streaming de archivos de vídeo: CLI de Azure](stream-files-cli-quickstart.md).

## <a name="see-also"></a>Consulte también

[CLI de Azure](/cli/azure/ams)
