---
title: Esquema de metadatos de salida de Azure Media Services | Microsoft Docs
description: En este artículo se proporciona información general sobre el esquema de metadatos de salida de Azure Media Services.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/03/2020
ms.author: juliako
ms.openlocfilehash: ce3d0a5beb5903d29b1deec345cf4673e3492e5d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080931"
---
# <a name="output-metadata"></a>Metadatos de salida

Un trabajo de codificación está asociado a un recurso (o recursos) de entrada donde desea realizar algunas tareas de codificación. Por ejemplo, codificar un archivo MP4 en conjuntos MP4 de velocidad de bits adaptable H.264; crear una miniatura; crear superposiciones. Tras la finalización de una tarea, se produce un recurso de salida.  El recurso de salida contiene vídeo, audio, miniaturas y otros archivos. El recurso de salida también contiene un archivo con metadatos sobre el recurso de salida. El nombre del archivo JSON de metadatos tiene el siguiente formato: `<source_file_name>_manifest.json` (por ejemplo, `BigBuckBunny_manifest.json`). Deberá buscar cualquier archivo *_metadata.json y consultar la cadena de ruta de acceso que contiene para encontrar el nombre de archivo de origen (sin truncamiento).

Media Services no realiza un examen preventivo de los recursos de entrada para generar metadatos. Los metadatos de entrada se generan únicamente como artefactos cuando un recurso de entrada se procesa en un trabajo. Por tanto, estos artefactos se escriben en el recurso de salida. Para generar metadatos de los recursos de entrada y salida, se utilizan diferentes herramientas. Por tanto, los metadatos de entrada tienen un esquema ligeramente diferente al de los metadatos de salida.

En este artículo se describen los elementos y los tipos del esquema JSON en los que se basan los metadatos de salida (&lt;nombre_de_archivo_de_origen&gt;_manifest.json). <!--For information about the file that contains metadata about the input asset, see [Input metadata](input-metadata-schema.md).  -->

Puede encontrar el código del esquema completo y un ejemplo de JSON al final de este artículo.  

## <a name="assetfile"></a>AssetFile

Colección de entradas AssetFile para el trabajo de codificación.  

| Nombre | Descripción |
| --- | --- |
| **Sources** |Colección de archivos multimedia de entrada/origen, que se procesa para producir este AssetFile.<br />Ejemplo: `"Sources": [{"Name": "Ignite-short_1280x720_AACAudio_3551.mp4"}]`|
| **VideoTracks**|Cada AssetFile físico puede contener cero o más pistas de vídeo intercaladas en un formato de contenedor adecuado. <br />Consulte [VideoTracks](#videotracks). |
| **AudioTracks**|Cada AssetFile físico puede contener cero o más pistas de audio intercaladas en un formato de contenedor adecuado. Se trata de la colección de todas esas pistas de audio.<br /> Para más información, consulte [AudioTracks](#audiotracks). |
| **Nombre**<br />Obligatorio |Nombre del archivo de recursos multimedia. <br /><br />Ejemplo: `"Name": "Ignite-short_1280x720_AACAudio_3551.mp4"`|
| **Tamaño**<br />Obligatorio |Tamaño del archivo de recursos en bytes. <br /><br />Ejemplo: `"Size": 32414631`|
| **Duration**<br />Obligatorio |Duración de la reproducción del contenido. Para obtener más información, vea el formato [ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html). <br /><br />Ejemplo: `"Duration": "PT1M10.315S"`|

## <a name="videotracks"></a>VideoTracks 

Cada AssetFile físico puede contener cero o más pistas de vídeo intercaladas en un formato de contenedor adecuado. El elemento **VideoTracks** representa una colección de todas las pistas de vídeo.  

| Nombre | Descripción |
| --- | --- |
| **Id**<br /> Obligatorio |Índice de base cero de esta pista de vídeo. **Nota:**  este **identificador** no es necesariamente el objeto TrackID que se usa en un archivo MP4. <br /><br />Ejemplo: `"Id": 1`|
| **FourCC**<br />Obligatorio | Código FourCC del códec de vídeo que ffmpeg notifica.  <br /><br />Ejemplo: `"FourCC": "avc1"`|
| **Perfil** |Perfil de H264 (solo es aplicable al códec H264).  <br /><br />Ejemplo: `"Profile": "High"` |
| **Level** |Nivel de H264 (solo es aplicable al códec H264).  <br /><br />Ejemplo: `"Level": "3.2"`|
| **Width**<br />Obligatorio |Ancho del vídeo codificado en píxeles.  <br /><br />Ejemplo: `"Width": "1280"`|
| **Height**<br />Obligatorio |Alto del vídeo codificado en píxeles.  <br /><br />Ejemplo: `"Height": "720"`|
| **DisplayAspectRatioNumerator**<br />Obligatorio|Numerador de la relación de aspecto de pantalla de vídeo.  <br /><br />Ejemplo: `"DisplayAspectRatioNumerator": 16.0`|
| **DisplayAspectRatioDenominator**<br />Obligatorio |Denominador de la relación de aspecto de pantalla de vídeo.  <br /><br />Ejemplo: `"DisplayAspectRatioDenominator": 9.0`|
| **Framerate**<br />Obligatorio |Velocidad de fotogramas de vídeo medida en formato .3f.  <br /><br />Ejemplo: `"Framerate": 29.970`|
| **Bitrate**<br />Obligatorio |Velocidad de bits de vídeo media en bits por segundo, según los cálculos de AssetFile. Solo cuenta la carga de transmisión básica y no incluye la sobrecarga de empaquetado.  <br /><br />Ejemplo: `"Bitrate": 3551567`|
| **TargetBitrate**<br />Obligatorio |Velocidad de bits media de destino para esta pista de vídeo, tal como se ha solicitado mediante la codificación preestablecida, en bits por segundo. <br /><br />Ejemplo: `"TargetBitrate": 3520000` |

## <a name="audiotracks"></a>AudioTracks 

Cada AssetFile físico puede contener cero o más pistas de audio intercaladas en un formato de contenedor adecuado. El elemento **AudioTracks** representa una colección de todas las pistas de audio.  

| Nombre  | Descripción |
| --- | --- |
| **Id**<br />Obligatorio  |Índice de base cero de esta pista de audio. **Nota:**  no es necesariamente el objeto TrackID que se usa en un archivo MP4.  <br /><br />Ejemplo: `"Id": 2`|
| **Codec**  |Cadena de códec de pista de audio.  <br /><br />Ejemplo: `"Codec": "aac"`|
| **Lenguaje**|Ejemplo: `"Language": "eng"`|
| **Channels**<br />Obligatorio|Número de canales de audio.  <br /><br />Ejemplo: `"Channels": 2`|
| **SamplingRate**<br />Obligatorio |Velocidad de muestreo de audio en muestras/s o Hz.  <br /><br />Ejemplo: `"SamplingRate": 48000`|
| **Bitrate**<br />Obligatorio |Velocidad de bits de audio media en bits por segundo, según los cálculos de AssetFile. Solo cuenta la carga de transmisión básica y no incluye la sobrecarga de empaquetado.  <br /><br />Ejemplo: `"Bitrate": 128041`|

## <a name="json-schema-example"></a>Ejemplo de esquema JSON

```json
{
  "AssetFile": [
    {
      "Sources": [
        {
          "Name": "Ignite-short_1280x720_AACAudio_3551.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "3.2",
          "Width": "1280",
          "Height": "720",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 3551567,
          "TargetBitrate": 3520000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_1280x720_AACAudio_3551.mp4",
      "Size": 32414631,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_960x540_AACAudio_2216.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "3.1",
          "Width": "960",
          "Height": "540",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 2216326,
          "TargetBitrate": 2210000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_960x540_AACAudio_2216.mp4",
      "Size": 20680897,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_640x360_AACAudio_1150.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "3.0",
          "Width": "640",
          "Height": "360",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 1150440,
          "TargetBitrate": 1150000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_640x360_AACAudio_1150.mp4",
      "Size": 11313920,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_480x270_AACAudio_722.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "2.1",
          "Width": "480",
          "Height": "270",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 722682,
          "TargetBitrate": 720000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_480x270_AACAudio_722.mp4",
      "Size": 7554708,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_320x180_AACAudio_380.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "1.3",
          "Width": "320",
          "Height": "180",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 380655,
          "TargetBitrate": 380000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }`
      ],
      "Name": "Ignite-short_320x180_AACAudio_380.mp4",
      "Size": 4548932,
      "Duration": "PT1M10.315S"
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

[Creación de una entrada de trabajo desde una dirección URL de HTTPS](job-input-from-http-how-to.md).
