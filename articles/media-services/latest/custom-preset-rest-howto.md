---
title: Codificación de transformación personalizado con Media Services v3 REST - Azure | Microsoft Docs
description: En este tema se muestra cómo usar Azure Media Services v3 para codificar una transformación personalizada con REST.
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
ms.openlocfilehash: 30e22cb786e5dc2a667fe41ca8edf398cf0b7613
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761794"
---
# <a name="how-to-encode-with-a-custom-transform---rest"></a>Cómo codificar con una transformación personalizada: REST

Al codificar con Azure Media Services, puede empezar a trabajar rápidamente con uno de los valores integrados preestablecidos recomendados según los procedimientos recomendados del sector, como se muestra en el [Streaming archivos](stream-files-tutorial-with-rest.md#create-a-transform) tutorial. También puede crear un valor predeterminado para sus requisitos específicos de escenario o dispositivo de destino personalizado.

## <a name="considerations"></a>Consideraciones

Al crear valores preestablecidos personalizados, se aplican las consideraciones siguientes:

* Todos los valores correspondientes al alto y ancho del contenido de AVC deben ser un múltiplo de 4.
* En Azure Media Services v3, todas las velocidades de bits de codificación son en bits por segundo. Esto es diferente de los valores preestablecidos con nuestras API v2, que usa kilobits por segundo como la unidad. Por ejemplo, si la velocidad de bits en la versión 2 se especificó como 128 (kilobits/segundo), en v3 se establecería en 128000 (bits/segundo).

## <a name="prerequisites"></a>Requisitos previos 

- [Cree una cuenta de Media Services](create-account-cli-how-to.md). <br/>Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services. 
- [Configuración de Postman para llamadas API REST de Azure Media Services](media-rest-apis-with-postman.md).<br/>Asegúrese de seguir el último paso en el tema [Obtención del token de Azure AD](media-rest-apis-with-postman.md#get-azure-ad-token). 

## <a name="define-a-custom-preset"></a>Definir un valor preestablecido personalizado

El ejemplo siguiente define el cuerpo de solicitud de una nueva transformación. Definimos un conjunto de resultados que se va a generarse cuando se usa esta transformación. 

En este ejemplo, en primer lugar agregamos una capa AacAudio para la codificación de audio y dos capas de H264Video para la codificación de vídeo. En las capas de vídeo, asignamos las etiquetas para que se pueden usar en los nombres de archivo de salida. A continuación, queremos que la salida para incluir también las miniaturas. En el ejemplo siguiente se especifican las imágenes en formato PNG, generada al 50% de la resolución de la entrada de vídeo y en tres marcas de tiempo - {25%, 50%, 75} de la longitud de la entrada de vídeo. Por último, especificamos el formato de los archivos de salida: uno para vídeo y audio y otro para las vistas en miniatura. Puesto que tenemos varios H264Layers, tenemos que usar las macros que generan nombres únicos por nivel. Se puede usar un `{Label}` o `{Bitrate}` macro, el ejemplo muestra la primera de ellas.

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

En este ejemplo, creamos un **transformar** que se basa en los valores preestablecidos personalizados que definimos anteriormente. Al crear una transformación, debe usar [obtener](https://docs.microsoft.com/rest/api/media/transforms/get) para comprobar si ya existe una. Si existe la transformación, volver a usarla. 

En la colección de Postman que ha descargado, seleccione **transformaciones y trabajos**->**Create o Update transformar**.

El método **PUT** de solicitud HTTP es similar a:

```
PUT https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
```

Seleccione el **cuerpo** pestaña y reemplace el cuerpo con el json de código [definida anteriormente](#define-a-custom-preset). Media Services aplicar la transformación al audio o vídeo especificado, deberá enviar a un trabajo en esa transformación.

Seleccione **Enviar**. 

Media Services aplicar la transformación al audio o vídeo especificado, deberá enviar a un trabajo en esa transformación. Para obtener un ejemplo completo que muestra cómo enviar un trabajo en una transformación, consulte [Tutorial: Archivos de vídeo: REST de Stream](stream-files-tutorial-with-rest.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte [otras operaciones de REST](https://docs.microsoft.com/rest/api/media/)
