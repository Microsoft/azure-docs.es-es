---
title: Esquema de metadatos de entrada de Azure Media Services V3
description: En este artículo se proporciona información general sobre el esquema de metadatos de entrada de Azure Media Services v3.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 9ddfe3ea0d26a9032922423e7f2c2a2b6c3e411a
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295571"
---
# <a name="input-metadata"></a>Metadatos de entrada

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Un trabajo de codificación está asociado a un recurso (o recursos) de entrada donde desea realizar algunas tareas de codificación.  Tras la finalización de una tarea, se produce un recurso de salida. El recurso de salida contiene vídeo, audio, miniaturas, manifiestos y otros archivos. 

El recurso de salida también contiene un archivo con metadatos sobre el recurso de entrada. El nombre del archivo JSON de metadatos tiene un identificador aleatorio, no lo use para identificar el recurso de entrada al que pertenece el de salida. Para identificar el recurso de entrada al que pertenece, use el campo `Uri` (para más información, consulte [Otros elementos secundarios](#other-child-elements)).  

Media Services no realiza un examen preventivo de los recursos de entrada para generar metadatos. Los metadatos de entrada se generan únicamente como artefactos cuando un recurso de entrada se procesa en un trabajo. Por tanto, estos artefactos se escriben en el recurso de salida. Para generar metadatos de los recursos de entrada y salida, se utilizan diferentes herramientas. Por tanto, los metadatos de entrada tienen un esquema ligeramente diferente al de los metadatos de salida.

En este artículo se describen los elementos y los tipos del esquema JSON en que se basan los metadatos de entrada (&lt;id_de_recurso&gt;_metadata.json). Para información acerca del archivo que contiene metadatos sobre el recurso de salida, consulte [Metadatos de salida](output-metadata-schema.md).  

Puede encontrar el ejemplo de esquema JSON al final de este artículo.  

## <a name="assetfile"></a>AssetFile  

Contiene una colección de elementos AssetFile para el trabajo de codificación.  

> [!NOTE]
> Los cuatro elementos secundarios siguientes deben aparecer en secuencia.  
> 
> 

| Nombre  | Descripción |
| --- | --- | 
| **VideoTracks**|Cada archivo de recursos físico puede contener cero o más pistas de vídeos intercaladas en un formato de contenedor adecuado. Para más información, consulte [VideoTracks](#videotracks). |
| **AudioTracks**|Cada archivo de recursos físico puede contener cero o más pistas de audio intercaladas en un formato de contenedor adecuado. Para más información, consulte [AudioTracks](#audiotracks). |
| **Metadata**  |Metadatos del archivo de recursos representados como cadenas de clave-valor. <br />Por ejemplo: `<Metadata key="language" value="eng" />` |

### <a name="other-child-elements"></a>Otros elementos secundarios

| Nombre | Descripción |
| --- | --- |
| **Nombre**<br />Obligatorio |Nombre del archivo de recursos. <br /><br />Ejemplo: `"Name": "Ignite-short.mp4"` |
| **Uri**<br />Obligatorio |Dirección URL donde se encuentra el recurso de entrada. Para identificar el recurso de entrada al que pertenece el recurso de salida, use el campo `Uri` en lugar del identificador.|
| **Tamaño**<br />Obligatorio |Tamaño del archivo de recursos en bytes.  <br /><br />Ejemplo: `"Size": 75739259`|
| **Duration**<br />Obligatorio |Duración de la reproducción del contenido. <br /><br />Ejemplo: `"Duration": "PT1M10.304S"`. |
| **NumberOfStreams**<br />Obligatorio |Número de transmisiones en el archivo de recursos.  <br /><br />Ejemplo: `"NumberOfStreams": 2`|
| **FormatNames**<br />Obligatorio |Nombres de formatos.  <br /><br />Ejemplo: `"FormatNames": "mov,mp4,m4a,3gp,3g2,mj2"`|
| **FormatVerboseName**<br /> Obligatorio |Nombres detallados de formatos. <br /><br />Ejemplo: `"FormatVerboseName": "QuickTime / MOV"` |
| **StartTime** |Hora de inicio del contenido.  <br /><br />Ejemplo: `"StartTime": "PT0S"` |
| **OverallBitRate** |Velocidad de bits media en bits por segundo del archivo de recursos.  <br /><br />Ejemplo: `"OverallBitRate": 8618539`|

## <a name="videotracks"></a>VideoTracks

| Nombre |  | Descripción |
| --- | --- |
| **FourCC**<br />Obligatorio |Código FourCC del códec de vídeo que ffmpeg notifica.<br /><br />Ejemplo: `"FourCC": "avc1"` |
| **Perfil** |Perfil de la pista de vídeo. <br /><br />Ejemplo: `"Profile": "Main"`|
| **Level** |Nivel de la pista de vídeo. <br /><br />Ejemplo: `"Level": "3.2"`|
| **PixelFormat** |Formato de píxel de la pista de vídeo. <br /><br />Ejemplo: `"PixelFormat": "yuv420p"`|
| **Width**<br />Obligatorio |Ancho del vídeo codificado en píxeles. <br /><br />Ejemplo: `"Width": "1280"`|
| **Height**<br />Obligatorio |Alto del vídeo codificado en píxeles.<br /><br />Ejemplo: `"Height": "720"` |
| **DisplayAspectRatioNumerator**<br />Obligatorio |Numerador de la relación de aspecto de pantalla de vídeo.<br /><br />Ejemplo: `"DisplayAspectRatioNumerator": 16.0` |
| **DisplayAspectRatioDenominator**<br />Obligatorio |Denominador de la relación de aspecto de pantalla de vídeo. <br /><br />Ejemplo: `"DisplayAspectRatioDenominator": 9.0`|
| **SampleAspectRatioNumerator** |Numerador de la relación de aspecto de muestra de vídeo. <br /><br />Ejemplo: `"SampleAspectRatioNumerator": 1.0`|
| **SampleAspectRatioDenominator**|Ejemplo: `"SampleAspectRatioDenominator": 1.0`|
| **FrameRate**<br />Obligatorio |Velocidad de fotogramas de vídeo medida en formato .3f. <br /><br />Ejemplo: `"FrameRate": 29.970`|
| **Bitrate** |Velocidad de bits de vídeo media en bits por segundo, según los cálculos del archivo de recursos. Solo se cuenta la carga de transmisión básica; la sobrecarga de empaquetado no se incluye. <br /><br />Ejemplo: `"Bitrate": 8421583`|
| **HasBFrames** |Número de pista de vídeo de fotogramas B. <br /><br />Ejemplo: `"HasBFrames": 2`|
| **Metadata** |Cadenas genéricas de clave-valor que sirven para almacenar información de diversa índole. <br />Vea el ejemplo completo al final del artículo. |
| **Id**<br />Obligatorio |Índice de base cero de esta pista de audio o vídeo.<br /><br /> este **identificador** no es necesariamente el objeto TrackID que se usa en un archivo MP4. <br /><br />Ejemplo: `"Id": 2`|
| **Codec** |Cadena de códec de pista de vídeo. <br /><br />Ejemplo: `"Codec": "h264"`|
| **CodecLongName** |Nombre largo del códec de pista de vídeo o audio. <br /><br />Ejemplo: `"CodecLongName": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10"`|
| **Codec** |Cadena de códec de pista de vídeo. <br /><br />Ejemplo: `"Codec": "h264"`|
| **TimeBase**<br />Obligatorio |Base de tiempo.<br /><br />Ejemplo: `"TimeBase": "1/30000"`|
| **NumberOfFrames** |Número de fotogramas (presente para pistas de vídeo). <br /><br />Ejemplo: `"NumberOfFrames": 2107`|
| **StartTime** |Hora de inicio de la pista.<br /><br />Ejemplo: `"StartTime": "PT0.033S"` |
| **Duration** |Duración de la pista. <br /><br />Ejemplo: `"Duration": "PT1M10.304S"`|

## <a name="audiotracks"></a>AudioTracks

| Nombre  | Descripción |
| --- | --- | 
| **SampleFormat** |Formato de ejemplo. <br /><br />Ejemplo: `"SampleFormat": "fltp"`|
| **ChannelLayout** |Distribución de canales. <br /><br />Ejemplo: `"ChannelLayout": "stereo"`|
| **Channels**<br />Obligatorio |Número de canales de audio (0 o más). <br /><br />Ejemplo: `"Channels": 2`|
| **SamplingRate**<br />Obligatorio |Velocidad de muestreo de audio en muestras/s o Hz. <br /><br />Ejemplo: `"SamplingRate": 48000`|
| **Bitrate** |Velocidad de bits de audio media en bits por segundo, según los cálculos en el archivo de recursos. Solo se cuenta la carga de transmisión básica; la sobrecarga de empaquetado no se incluye en este recuento. <br /><br />Ejemplo: `"Bitrate": 192080`|
| **Metadata** |Cadenas genéricas de clave-valor que sirven para almacenar información de diversa índole.  <br />Vea el ejemplo completo al final del artículo. |
| **Id**<br />Obligatorio |Índice de base cero de esta pista de audio o vídeo.<br /><br /> No es necesariamente un elemento TrackID como se usa en un archivo MP4. <br /><br />Ejemplo: `"Id": 1`|
| **Codec** |Cadena de códec de pista de vídeo. <br /><br />Ejemplo: `"Codec": "aac"`|
| **CodecLongName** |Nombre largo del códec de pista de vídeo o audio. <br /><br />Ejemplo: `"CodecLongName": "AAC (Advanced Audio Coding)"`|
| **TimeBase**<br />Obligatorio |Base de tiempo.<br /><br />Ejemplo: `"TimeBase": "1/48000"` |
| **NumberOfFrames** |Número de fotogramas (presente para pistas de vídeo). <br /><br />Ejemplo: `"NumberOfFrames": 3294`|
| **StartTime** |Hora de inicio de la pista. Para más información, consulte [ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html). <br /><br />Ejemplo: `"StartTime": "PT0S"` |
| **Duration** |Duración de la pista. <br /><br />Ejemplo: `"Duration": "PT1M10.272S"` |

## <a name="metadata"></a>Metadatos

| Nombre | Descripción |
| --- | --- |
| **key**<br />Obligatorio |La clave del par clave-valor. |
| **value**<br /> Obligatorio |El valor del par clave-valor. |

## <a name="schema-example"></a>Ejemplo de esquema

```json
{
  "AssetFile": [
    {
      "VideoTracks": [
        {
          "FourCC": "avc1",
          "Profile": "Main",
          "Level": "3.2",
          "PixelFormat": "yuv420p",
          "Width": "1280",
          "Height": "720",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "SampleAspectRatioNumerator": 1.0,
          "SampleAspectRatioNumeratorSpecified": true,
          "SampleAspectRatioDenominator": 1.0,
          "SampleAspectRatioDenominatorSpecified": true,
          "FrameRate": 29.970,
          "Bitrate": 8421583,
          "BitrateSpecified": true,
          "HasBFrames": 2,
          "HasBFramesSpecified": true,
          "Disposition": {
            "Default": 1
          },
          "Metadata": [
            {
              "key": "creation_time",
              "value": "2018-02-21T21:42:08.000000Z"
            },
            {
              "key": "language",
              "value": "eng"
            },
            {
              "key": "handler_name",
              "value": "Video Media Handler"
            },
            {
              "key": "encoder",
              "value": "AVC Coding"
            }
          ],
          "Id": 2,
          "Codec": "h264",
          "CodecLongName": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
          "TimeBase": "1/30000",
          "NumberOfFrames": 2107,
          "NumberOfFramesSpecified": true,
          "StartTime": "PT0.033S",
          "Duration": "PT1M10.304S"
        }
      ],
      "AudioTracks": [
        {
          "SampleFormat": "fltp",
          "ChannelLayout": "stereo",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 192080,
          "BitrateSpecified": true,
          "BitsPerSampleSpecified": true,
          "Disposition": {
            "Default": 1
          },
          "Metadata": [
            {
              "key": "creation_time",
              "value": "2018-02-21T21:42:08.000000Z"
            },
            {
              "key": "language",
              "value": "eng"
            },
            {
              "key": "handler_name",
              "value": "Sound Media Handler"
            }
          ],
          "Id": 1,
          "Codec": "aac",
          "CodecLongName": "AAC (Advanced Audio Coding)",
          "TimeBase": "1/48000",
          "NumberOfFrames": 3294,
          "NumberOfFramesSpecified": true,
          "StartTime": "PT0S",
          "Duration": "PT1M10.272S"
        }
      ],
      "Metadata": [
        {
          "key": "major_brand",
          "value": "mp42"
        },
        {
          "key": "minor_version",
          "value": "19529854"
        },
        {
          "key": "compatible_brands",
          "value": "mp42isom"
        },
        {
          "key": "creation_time",
          "value": "2018-02-21T21:42:08.000000Z"
        }
      ],
      "Name": "Ignite-short.mp4",
      "Uri": "https://amsstorageacct.blob.core.windows.net/asset-00000000-0000-0000-000000000000/ignite.mp4",
      "Size": 75739259,
      "Duration": "PT1M10.304S",
      "NumberOfStreams": 2,
      "FormatNames": "mov,mp4,m4a,3gp,3g2,mj2",
      "FormatVerboseName": "QuickTime / MOV",
      "StartTime": "PT0S",
      "OverallBitRate": 8618539,
      "OverallBitRateSpecified": true
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

[Metadatos de salida](output-metadata-schema.md)
