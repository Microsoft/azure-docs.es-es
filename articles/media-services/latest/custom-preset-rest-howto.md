---
title: 'Codificación de una transformación personalizada mediante Media Services v3 REST: Azure | Microsoft Docs'
description: En este tema se muestra cómo usar Azure Media Services v3 para codificar una transformación personalizada con REST.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: ''
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 2656bf93cb9c29ded4b9dde49f0caba91c1654b7
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295638"
---
# <a name="how-to-encode-with-a-custom-transform---rest"></a>Cómo codificar con una transformación personalizada: REST

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Al codificar con Azure Media Services, puede empezar a trabajar rápidamente con uno de los valores preestablecidos integrados recomendados según los procedimientos recomendados del sector, tal como se muestra en el tutorial [Streaming de archivos](stream-files-tutorial-with-rest.md#create-a-transform). También puede compilar un valor preestablecido personalizado para sus requisitos específicos de escenario o dispositivo.

## <a name="considerations"></a>Consideraciones

Al crear valores preestablecidos personalizados, se aplican las consideraciones siguientes:

* Todos los valores de alto y ancho del contenido de AVC deben ser un múltiplo de 4.
* En Azure Media Services v3, todas las velocidades de bits de codificación se expresan en bits por segundo. Esto difiere de los valores preestablecidos en las API v2, en donde se usaban kilobits por segundo como unidad. Por ejemplo, si la velocidad de bits de v2 se especificaba como 128 (kilobits/segundo), en v3 se establecería en 128 000 (bits/segundo).

## <a name="prerequisites"></a>Prerrequisitos 

- [Cree una cuenta de Media Services](./create-account-howto.md). <br/>Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services. 
- [Configuración de Postman para llamadas API REST de Azure Media Services](media-rest-apis-with-postman.md).<br/>Asegúrese de seguir el último paso en el tema [Obtención del token de Azure AD](media-rest-apis-with-postman.md#get-azure-ad-token). 

## <a name="define-a-custom-preset"></a>Definir un valor preestablecido personalizado

En el ejemplo siguiente se define el cuerpo de la solicitud de una nueva transformación. Se define un conjunto de salidas que queremos que se genere cuando se use esta transformación. 

En primer lugar, se agrega una capa AacAudio para la codificación de audio y dos capas H264Video para la codificación de vídeo. En las capas de vídeo se asignan etiquetas para que se puedan usar en los nombres de archivo de salida. Luego queremos que la salida también incluya miniaturas. En el ejemplo siguiente se especifican imágenes en formato PNG generadas al 50 % de la resolución del vídeo de entrada, y en tres marcas de tiempo, {25 %, 50 %, 75 %}, de la longitud del vídeo de entrada. Por último, se especifica el formato de los archivos de salida: uno para vídeo y audio y otro para las miniaturas. Puesto que hay varias H264Layers, tenemos que usar macros que generen nombres únicos por capa. Se puede usar una macro `{Label}` o `{Bitrate}`; el ejemplo muestra la primera.

```json
{
    "properties": {
        "description": "Basic Transform using a custom encoding preset",
        "outputs": [
            {
                "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                "preset": {
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
            }
        ]
    }
}

```

## <a name="create-a-new-transform"></a>Crear una nueva transformación  

En este ejemplo, creamos una **transformación** que se basa en los valores preestablecidos personalizados que definimos anteriormente. Al crear una transformación, primero debe usar el método [Get](/rest/api/media/transforms/get) para comprobar si ya existe una. Si existe la transformación, vuelva a usarla. 

En la colección de Postman que descargó, seleccione **Transforms and Jobs** (Transformaciones y trabajos)->**Create or Update Transform** (Crear o actualizar transformación).

El método **PUT** de solicitud HTTP es similar a:

```
PUT https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
```

Seleccione la pestaña **Cuerpo** y pegue el código json que [definió anteriormente](#define-a-custom-preset). Para que Media Services aplique la transformación al audio o vídeo especificado, deberá enviar un trabajo en esa transformación.

Seleccione **Enviar**. 

Para que Media Services aplique la transformación al audio o vídeo especificado, deberá enviar un trabajo en esa transformación. Para obtener un ejemplo completo que muestra cómo enviar un trabajo en una transformación, consulte [Tutorial: Hacer streaming de archivos de vídeo: REST](stream-files-tutorial-with-rest.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte [otras operaciones de REST](/rest/api/media/).
