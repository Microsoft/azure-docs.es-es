---
title: Codificación avanzada mediante la personalización de valores preestablecidos de MES | Microsoft Docs
description: En este tema se muestra cómo realizar codificación avanzada mediante la personalización de valores preestablecidos de tarea Media Encoder Estándar.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 2a4ade25-e600-4bce-a66e-e29cf4a38369
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 4288e2e955f8205f3b6551c83a5c883eecf02501
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89264817"
---
# <a name="perform-advanced-encoding-by-customizing-mes-presets"></a>Codificación avanzada mediante la personalización de valores preestablecidos de MES

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general

Este tema muestra cómo personalizar los valores preestablecidos de Media Encoder Standard. El tema [Encoding with Media Encoder Standard using custom presets](media-services-custom-mes-presets-with-dotnet.md) (Codificación con Media Encoder Standard mediante valores preestablecidos personalizados) muestra cómo usar .NET para crear una tarea de codificación y un trabajo que ejecuta esta tarea. Después de personalizar un valor preestablecido, proporcione los valores preestablecidos personalizados a la tarea de codificación. 

Si utiliza un ajuste preestablecido XML, asegúrese de conservar el orden de los elementos, como se muestra en los ejemplos XML siguientes (por ejemplo, KeyFrameInterval debe preceder a SceneChangeDetection).

> [!NOTE] 
> Muchas de las características avanzadas de Media Services v2 de Media Encoder Standard no están disponibles en v3: Para más información, consulte las [carencias de características](../latest/media-services-v2-vs-v3.md#feature-gaps-with-respect-to-v2-apis).

## <a name="support-for-relative-sizes"></a>Compatibilidad con tamaños relativos

Si se generan miniaturas, no será preciso especificar siempre la anchura y altura, en píxeles, de la salida. Se pueden en forma porcentual, en el intervalo [1 %,..., 100 %].

### <a name="json-preset"></a>Valor preestablecido JSON

```json
"Width": "100%",
"Height": "100%"
```

### <a name="xml-preset"></a>Valor preestablecido XML

```xml
<Width>100%</Width>
<Height>100%</Height>
```

## <a name="generate-thumbnails"></a>Generación de miniaturas

En esta sección se muestra cómo personalizar un valor preestablecido que genera vistas en miniatura. El valor preestablecido que se define a continuación contiene información sobre cómo se quiere codificar el archivo, así como la información necesaria para generar miniaturas. Puede usar cualquiera de los valores preestablecidos de MES que se documentan en [esta](media-services-mes-presets-overview.md) sección y agregar el código que genera miniaturas.  

> [!NOTE]
> La configuración de **SceneChangeDetection** en el siguiente valor preestablecido solo se puede establecer en true si va a codificar en vídeo de una única velocidad de bits. Si va a codificar un vídeo con velocidades de bits múltiple y establece **SceneChangeDetection** en true, el codificador devolverá un error.  
>
>

Para obtener información sobre el esquema, consulte [este](media-services-mes-schema.md) tema.

Asegúrese de revisar la sección [Consideraciones](#considerations) .

### <a name="json-preset"></a>Valor preestablecido JSON

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
          "Width": 640,
          "Height": 360
        }
      ],
      "Start": "{Best}",
      "Type": "JpgImage"
    },
    {
      "PngLayers": [
        {
          "Type": "PngLayer",
          "Width": 640,
          "Height": 360,
        }
      ],
      "Start": "00:00:01",
      "Step": "00:00:10",
      "Range": "00:00:58",
      "Type": "PngImage"
    },
    {
      "BmpLayers": [
        {
          "Type": "BmpLayer",
          "Width": 640,
          "Height": 360
        }
      ],
      "Start": "10%",
      "Step": "10%",
      "Range": "90%",
      "Type": "BmpImage"
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
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "PngFormat"
      }
    },
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "BmpFormat"
      }
    },
    {
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="xml-preset"></a>Valor preestablecido XML

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <SceneChangeDetection>true</SceneChangeDetection>
      <H264Layers>
        <H264Layer>
          <Bitrate>4500</Bitrate>
          <Width>1280</Width>
          <Height>720</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>4500</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <AACAudio>
      <Profile>AACLC</Profile>
      <Channels>2</Channels>
      <SamplingRate>48000</SamplingRate>
      <Bitrate>128</Bitrate>
    </AACAudio>
    <JpgImage Start="{Best}">
      <JpgLayers>
        <JpgLayer>
          <Width>640</Width>
          <Height>360</Height>
          <Quality>90</Quality>
        </JpgLayer>
      </JpgLayers>
    </JpgImage>
    <BmpImage Start="10%" Step="10%" Range="90%">
      <BmpLayers>
        <BmpLayer>
          <Width>640</Width>
          <Height>360</Height>
        </BmpLayer>
      </BmpLayers>
    </BmpImage>
    <PngImage Start="00:00:01" Step="00:00:10" Range="00:00:58">
      <PngLayers>
        <PngLayer>
          <Width>640</Width>
          <Height>360</Height>
        </PngLayer>
      </PngLayers>
    </PngImage>
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
      <MP4Format />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <JpgFormat />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <BmpFormat />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <PngFormat />
    </Output>
  </Outputs>
</Preset>
```

### <a name="considerations"></a>Consideraciones

Se aplican las siguientes consideraciones:

* El uso de marcas de tiempo explícitas para inicio/paso/intervalo asume que el origen de la entrada tiene al menos 1 minuto de duración.
* Los elementos Jpg/Png/BmpImage tienen atributos de cadena Start, Step y Range, que se pueden interpretar como:

  * Número de marco si son enteros no negativos, por ejemplo, "Start": "120",
  * Relativos a la duración de origen si se expresan como sufijo de %, por ejemplo, "Start": "15%", o bien
  * Marca de tiempo si se por ejemplo, "Start": "00:01:00"

    Puede mezclar y hacer coincidir notaciones a su conveniencia.

    Además, Start también admite una macro especial:{Best}, que intenta determinar el primer marco "interesante" del contenido. NOTA: (Step y Range se omiten cuando Start se establece en {Best}).
  * Valores predeterminados: Start:{Best}
* Es necesario proporcionar explícitamente el formato de salida para cada formato de imagen: Jpg, Png o BmpFormat. Cuando está presente, MES hará coincidir JpgVideo con JpgFormat y así sucesivamente. OutputFormat presenta una nueva macro específica de códec de imagen: {Index}, que debe estar presente (una vez y sólo una vez) para formatos de salida de imagen.

## <a name="trim-a-video-clipping"></a><a id="trim_video"></a>Recorte de un vídeo
En esta sección se habla sobre cómo modificar los valores preestablecidos del codificador para recortar el vídeo de entrada donde la entrada es un archivo denominado intermedio o a petición. El codificador también se puede usar para recortar un recurso que se captura o archiva desde una transmisión en directo; puede consultar los detalles en [este blog](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/).

Para recortar vídeos, puede usar cualquiera de los valores preestablecidos de MES que se documentan en [esta](media-services-mes-presets-overview.md) sección y modificar el elemento **Sources** (como se muestra a continuación). El valor de StartTime debe coincidir con las marcas de tiempo absoluto de la entrada de vídeo. Por ejemplo, si el primer fotograma del vídeo de entrada tiene una marca de tiempo de 12:00:10.000, StartTime debe ser al menos 12:00:10.000 o un valor superior. En el ejemplo siguiente, se supone que el vídeo de entrada tiene una marca de tiempo inicial de cero. **Sources** debe incluirse al comienzo del valor preestablecido.

### <a name="json-preset"></a><a id="json"></a>Valor preestablecido JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "StartTime": "00:00:04",
      "Duration": "00:00:16"
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "StretchMode": "AutoSize",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 3400,
          "MaxBitrate": 3400,
          "BufferWindow": "00:00:05",
          "Width": 1280,
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
          "Bitrate": 2250,
          "MaxBitrate": 2250,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 540,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1500,
          "MaxBitrate": 1500,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 540,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1000,
          "MaxBitrate": 1000,
          "BufferWindow": "00:00:05",
          "Width": 640,
          "Height": 360,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 650,
          "MaxBitrate": 650,
          "BufferWindow": "00:00:05",
          "Width": 640,
          "Height": 360,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 400,
          "MaxBitrate": 400,
          "BufferWindow": "00:00:05",
          "Width": 320,
          "Height": 180,
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

### <a name="xml-preset"></a>Valor preestablecido XML
Para recortar vídeos, puede usar cualquiera de los valores preestablecidos de MES que se documentan [aquí](media-services-mes-presets-overview.md) y modificar el elemento **Sources** (como se muestra a continuación).

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Sources>
    <Source StartTime="PT4S" Duration="PT14S"/>
  </Sources>
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <H264Layers>
        <H264Layer>
          <Bitrate>3400</Bitrate>
          <Width>1280</Width>
          <Height>720</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>3400</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>2250</Bitrate>
          <Width>960</Width>
          <Height>540</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>2250</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>1500</Bitrate>
          <Width>960</Width>
          <Height>540</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1500</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>1000</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1000</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>650</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>650</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>400</Bitrate>
          <Width>320</Width>
          <Height>180</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>400</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <AACAudio>
      <Profile>AACLC</Profile>
      <Channels>2</Channels>
      <SamplingRate>48000</SamplingRate>
      <Bitrate>128</Bitrate>
    </AACAudio>
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
      <MP4Format />
    </Output>
  </Outputs>
</Preset>
```

## <a name="create-an-overlay"></a><a id="overlay"></a>Creación de una superposición

Media Encoder Estándar permite superponer una imagen en un vídeo existente. Actualmente, se admiten los siguientes formatos: png, jpg, gif y bmp. El valor preestablecido que se definen a continuación es un ejemplo básico de superposición de vídeo.

Además de definir un archivo de valores preestablecidos, también tiene que permitir que Media Services sepa qué archivo del recurso es la imagen de superposición y qué archivo contiene el vídeo de origen en el que desea superponer la imagen. El archivo de vídeo debe ser el archivo **principal** .

Si usa. NET, agregue las dos funciones siguientes al ejemplo de .NET definido en [este](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet) tema. La función **UploadMediaFilesFromFolder** carga archivos desde una carpeta (por ejemplo, BigBuckBunny.mp4 e Image001.png) y establece el archivo mp4 como el archivo principal del recurso. La función **EncodeWithOverlay** usa el archivo preestablecido personalizado que se le pasó (por ejemplo, el archivo preestablecido que aparece a continuación) para crear la tarea de codificación.

```csharp
static public IAsset UploadMediaFilesFromFolder(string folderPath)
{
    IAsset asset = _context.Assets.CreateFromFolder(folderPath, AssetCreationOptions.None);

    foreach (var af in asset.AssetFiles)
    {
        // The following code assumes 
        // you have an input folder with one MP4 and one overlay image file.
        if (af.Name.Contains(".mp4"))
            af.IsPrimary = true;
        else
            af.IsPrimary = false;

        af.Update();
    }

    return asset;
}

static public IAsset EncodeWithOverlay(IAsset assetSource, string customPresetFileName)
{
    // Declare a new job.
    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
    // Get a media processor reference, and pass to it the name of the 
    // processor to use for the specific task.
    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

    // Load the XML (or JSON) from the local file.
    string configuration = File.ReadAllText(customPresetFileName);

    // Create a task
    ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
        processor,
        configuration,
        TaskOptions.None);

    // Specify the input assets to be encoded.
    // This asset contains a source file and an overlay file.
    task.InputAssets.Add(assetSource);

    // Add an output asset to contain the results of the job. 
    task.OutputAssets.AddNew("Output asset",
        AssetCreationOptions.None);

    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
    job.Submit();
    job.GetExecutionProgressTask(CancellationToken.None).Wait();

    return job.OutputMediaAssets[0];
}
```

> [!NOTE]
> Limitaciones actuales:
>
> No se admite el ajuste de opacidad de superposición.
>
> El archivo de vídeo de origen y el archivo de imagen de superposición tienen que estar en el mismo recurso y el archivo de vídeo debe establecerse como el archivo principal de este recurso.
>
>

### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "VideoOverlay": {
          "Position": {
            "X": 100,
            "Y": 100,
            "Width": 100,
            "Height": 50
          },
          "AudioGainLevel": 0.0,
          "MediaParams": [
            {
              "OverlayLoopCount": 1
            },
            {
              "IsOverlay": true,
              "OverlayLoopCount": 1
            }
          ],
          "Source": "Image001.png",
          "Clip": {
            "Duration": "00:00:05"
          },
          "FadeInDuration": {
            "Duration": "00:00:01"
          },
          "FadeOutDuration": {
            "StartTime": "00:00:03",
            "Duration": "00:00:04"
          }
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
          "Bitrate": 1045,
          "MaxBitrate": 1045,
          "BufferWindow": "00:00:05",
          "ReferenceFrames": 3,
          "EntropyMode": "Cavlc",
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "Width": "640",
          "Height": "360",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
    },
    {
      "Type": "CopyAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}{Extension}",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="xml-preset"></a>Valor preestablecido XML

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Sources>
    <Source>
      <Streams />
      <Filters>
        <VideoOverlay>
          <Source>Image001.png</Source>
          <Clip Duration="PT5S" />
          <FadeInDuration Duration="PT1S" />
          <FadeOutDuration StartTime="PT3S" Duration="PT4S" />
          <Position X="100" Y="100" Width="100" Height="50" />
          <Opacity>0</Opacity>
          <AudioGainLevel>0</AudioGainLevel>
          <MediaParams>
            <MediaParam>
              <IsOverlay>false</IsOverlay>
              <OverlayLoopCount>1</OverlayLoopCount>
            </MediaParam>
            <MediaParam>
              <IsOverlay>true</IsOverlay>
              <OverlayLoopCount>1</OverlayLoopCount>
            </MediaParam>
          </MediaParams>
        </VideoOverlay>
      </Filters>
      <Pad>true</Pad>
    </Source>
  </Sources>
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <H264Layers>
        <H264Layer>
          <Bitrate>1045</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>0</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cavlc</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1045</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <CopyAudio />
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}{Extension}">
      <MP4Format />
    </Output>
  </Outputs>
</Preset>
```

## <a name="insert-a-silent-audio-track-when-input-has-no-audio"></a><a id="silent_audio"></a>Inserción de una pista de audio silenciosa cuando la entrada no tiene audio
De forma predeterminada, si envía una entrada al codificador que solo contenga vídeo, y no audio, el recurso de salida contendrá archivos que solo contienen datos de vídeos. Algunos reproductores no puede controlar estos flujos de salida. Puede usar este ajuste para forzar al codificador a agregar una pista de audio silenciosa a la salida en ese escenario.

Para forzar al codificador a producir un activo que contiene una pista de audio silenciosa cuando la entrada no tiene audio, especifique el valor de "InsertSilenceIfNoAudio".

Puede usar cualquiera de los valores preestablecidos de MES que se documentan en [esta](media-services-mes-presets-overview.md) sección y realizar los cambios siguientes:

### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "Channels": 2,
  "SamplingRate": 44100,
  "Bitrate": 96,
  "Type": "AACAudio",
  "Condition": "InsertSilenceIfNoAudio"
}
```

### <a name="xml-preset"></a>Valor preestablecido XML

```xml
<AACAudio Condition="InsertSilenceIfNoAudio">
  <Channels>2</Channels>
  <SamplingRate>44100</SamplingRate>
  <Bitrate>96</Bitrate>
</AACAudio>
```

## <a name="disable-auto-de-interlacing"></a><a id="deinterlacing"></a>Deshabilitar el entrelazado automático
Los clientes no tienen que hacer nada si prefieren que el enlazado del contenido entrelazado se anule automáticamente. Cuando la anulación de entrelazado automática está activada (valor predeterminado), el MES realiza la detección automática de fotogramas entrelazados y solo se anula el entrelazado de los fotogramas marcados como entrelazados.

Puede desactivar la anulación de entrelazado automática. Esta opción no se recomienda.

### <a name="json-preset"></a>Valor preestablecido JSON

```json
"Sources": [
  {
    "Filters": {
      "Deinterlace": {
        "Mode": "Off"
      }
    },
  }
]
```

### <a name="xml-preset"></a>Valor preestablecido XML

```xml
<Sources>
  <Source>
    <Filters>
      <Deinterlace>
        <Mode>Off</Mode>
      </Deinterlace>
    </Filters>
  </Source>
</Sources>
```

## <a name="audio-only-presets"></a><a id="audio_only"></a>Valores preestablecidos de solo audio
En esta sección se muestran dos valores preestablecidos de MES de solo audio: AAC Audio y Audio AAC de buena calidad.

### <a name="aac-audio"></a>Audio AAC

```json
{
  "Version": 1.0,
  "Codecs": [
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
      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="aac-good-quality-audio"></a>Audio AAC de buena calidad

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "Profile": "AACLC",
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 192,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="concatenate-two-or-more-video-files"></a><a id="concatenate"></a>Concatenación de dos o más archivos de vídeo

En el ejemplo siguiente se muestra cómo generar un valor preestablecido para concatenar dos o más archivos de vídeo. El escenario más frecuente es cuando desea agregar un encabezado o un finalizador al vídeo principal. El uso previsto es cuando los archivos de vídeo que se están editando juntos comparten propiedades (resolución de vídeo, velocidad de fotogramas, número de pistas de audio, etc.). Procure no para combinar vídeos de distintas velocidades de fotogramas o con un número diferente de pistas de audio.

>[!NOTE]
>El diseño actual de la característica de concatenación espera que los clips de vídeo de entrada sean coherentes en cuanto a la resolución, la velocidad de fotogramas, etc. 

### <a name="requirements-and-considerations"></a>Requisitos y consideraciones

* Los vídeos de entrada solo deberían tener una pista de audio.
* Los vídeos de entrada deben tener la misma velocidad de fotogramas.
* Debe cargar los vídeos en recursos independientes y establecer los vídeos como archivo principal en cada recurso.
* Debe conocer la duración de los vídeos.
* En los siguientes ejemplos preestablecidos se supone que todos los vídeos de entrada empiezan con una marca de tiempo de cero. Debe modificar los valores StartTime si los vídeos tienen marca de tiempo de inicio diferente, como suele ser el caso de archivos dinámicos.
* El valor preestablecido de JSON hace referencia explícita a los valores de AssetID de los recursos de entrada.
* El código de ejemplo presupone que se ha guardado el valor preestablecido de JSON en un archivo local, como "C:\supportFiles\preset.json". También presupone que se han creado dos recursos mediante la carga de dos archivos de vídeo y de que conoce los valores de AssetID resultantes.
* El fragmento de código y el valor preestablecido de JSON muestran un ejemplo de concatenación de dos archivos de vídeo. Puede ampliarlo a más de dos vídeos mediante las siguientes tareas:

  1. Llamada de Calling task.InputAssets.Add() repetidamente para agregar más vídeos en orden.
  2. Realización de las ediciones correspondientes al elemento "Sources" de JSON, agregando más entradas en el mismo orden.

### <a name="net-code"></a>Código .NET

```csharp
IAsset asset1 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:606db602-efd7-4436-97b4-c0b867ba195b").FirstOrDefault();
IAsset asset2 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e").FirstOrDefault();

// Declare a new job.
IJob job = _context.Jobs.Create("Media Encoder Standard Job for Concatenating Videos");
// Get a media processor reference, and pass to it the name of the
// processor to use for the specific task.
IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

// Load the XML (or JSON) from the local file.
string configuration = File.ReadAllText(@"c:\supportFiles\preset.json");

// Create a task
ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
    processor,
    configuration,
    TaskOptions.None);

// Specify the input videos to be concatenated (in order).
task.InputAssets.Add(asset1);
task.InputAssets.Add(asset2);
// Add an output asset to contain the results of the job.
// This output is specified as AssetCreationOptions.None, which
// means the output asset is not encrypted.
task.OutputAssets.AddNew("Output asset",
    AssetCreationOptions.None);

job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
job.Submit();
job.GetExecutionProgressTask(CancellationToken.None).Wait();
```

### <a name="json-preset"></a>Valor preestablecido JSON

Actualice el valor preestablecido personalizado con los identificadores de los recursos que desee concatenar y con el segmento de tiempo adecuado para cada vídeo.

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "AssetID": "606db602-efd7-4436-97b4-c0b867ba195b",
      "StartTime": "00:00:01",
      "Duration": "00:00:15"
    },
    {
      "AssetID": "a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e",
      "StartTime": "00:00:02",
      "Duration": "00:00:05"
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "SceneChangeDetection": true,
      "H264Layers": [
        {
          "Level": "auto",
          "Bitrate": 1800,
          "MaxBitrate": 1800,
          "BufferWindow": "00:00:05",
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "Width": "640",
          "Height": "360",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
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
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="crop-videos-with-media-encoder-standard"></a><a id="crop"></a>Recorte de vídeos con Codificador multimedia estándar
Consulte el tema [Recorte de vídeos con Codificador multimedia estándar](media-services-crop-video.md) .

## <a name="insert-a-video-track-when-input-has-no-video"></a><a id="no_video"></a>Inserción de una pista de vídeo cuando la entrada cuando no tiene vídeo

De forma predeterminada, si envía una entrada al codificador que solo contenga audio, y no vídeo, el recurso de salida contendrá archivos que solo contienen datos de audio. Algunos reproductores, incluidos Azure Media Player (consulte [esta página](https://feedback.azure.com/forums/169396-azure-media-services/suggestions/8082468-audio-only-scenarios)) no pueden controlar estas secuencias. Puede usar este ajuste para forzar al codificador a agregar una pista de vídeo monocromática a la salida en ese escenario.

> [!NOTE]
> Al forzar que el codificador inserte una pista de vídeo de salida se aumenta el tamaño del recurso de salida, y, por tanto, el costo por la tarea de codificación. Debe ejecutar pruebas para comprobar que el aumento resultante solo tenga solo un leve impacto en los cargos mensuales.
>

### <a name="inserting-video-at-only-the-lowest-bitrate"></a>Inserción de vídeo solo con la velocidad de bits mínima

Supongamos que utiliza un valor preestablecido de codificación de varias velocidades de bits como [H264 Multiple Bitrate 720p](media-services-mes-preset-h264-multiple-bitrate-720p.md) con el fin de codificar el catálogo de entrada completo para el proceso de streaming, que contiene una combinación de archivos de vídeo y archivos de solo audio. En este escenario, cuando la entrada no tiene ningún vídeo, puede forzar que el codificador inserte una pista de vídeo monocromática solo con la velocidad de bits mínima, en lugar de hacerlo con cada velocidad de bits de salida. Para conseguir esto, debe usar la marca **InsertBlackIfNoVideoBottomLayerOnly**.

Puede usar cualquiera de los valores preestablecidos de MES que se documentan en [esta](media-services-mes-presets-overview.md) sección y realizar los cambios siguientes:

#### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "KeyFrameInterval": "00:00:02",
  "StretchMode": "AutoSize",
  "Condition": "InsertBlackIfNoVideoBottomLayerOnly",
  "H264Layers": [
    …
  ]
}
```

#### <a name="xml-preset"></a>Valor preestablecido XML

Al usar XML, utilice Condition="InsertBlackIfNoVideoBottomLayerOnly" como atributo del elemento **H264Video** y Condition="InsertSilenceIfNoAudio" como atributo de **AACAudio**.

```xml
. . .
<Encoding>
  <H264Video Condition="InsertBlackIfNoVideoBottomLayerOnly">
    <KeyFrameInterval>00:00:02</KeyFrameInterval>
    <SceneChangeDetection>true</SceneChangeDetection>
    <StretchMode>AutoSize</StretchMode>
    <H264Layers>
      <H264Layer>
        . . .
      </H264Layer>
    </H264Layers>
    <Chapters />
  </H264Video>
  <AACAudio Condition="InsertSilenceIfNoAudio">
    <Profile>AACLC</Profile>
    <Channels>2</Channels>
    <SamplingRate>48000</SamplingRate>
    <Bitrate>128</Bitrate>
  </AACAudio>
</Encoding>
. . .
```

### <a name="inserting-video-at-all-output-bitrates"></a>Inserción de vídeo con todas las velocidades de bits de salida
Supongamos que utiliza un valor preestablecido de codificación de varias velocidades de bits como [H264 Multiple Bitrate 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) con el fin de codificar el catálogo de entrada completo para el proceso de streaming, que contiene una combinación de archivos de vídeo y archivos de solo audio. En este escenario, cuando la entrada no tiene ningún vídeo, puede forzar que el codificador inserte una pista de vídeo monocromática con todas las velocidades de bits de salida. Esto garantiza que los recursos de salida sean homogéneos con respecto al número de pistas de vídeo y de audio. Para ello, debe especificar la marca InsertBlackIfNoVideo.

Puede usar cualquiera de los valores preestablecidos de MES que se documentan en [esta](media-services-mes-presets-overview.md) sección y realizar los cambios siguientes:

#### <a name="json-preset"></a>Valor preestablecido JSON

```json
{
  "KeyFrameInterval": "00:00:02",
  "StretchMode": "AutoSize",
  "Condition": "InsertBlackIfNoVideo",
  "H264Layers": [
    …
  ]
}
```

#### <a name="xml-preset"></a>Valor preestablecido XML

Al usar XML, utilice Condition="InsertBlackIfNoVideo" como atributo del elemento **H264Video** y Condition="InsertSilenceIfNoAudio" como atributo de **AACAudio**.

```xml
. . .
<Encoding>
  <H264Video Condition="InsertBlackIfNoVideo">
    <KeyFrameInterval>00:00:02</KeyFrameInterval>
    <SceneChangeDetection>true</SceneChangeDetection>
    <StretchMode>AutoSize</StretchMode>
    <H264Layers>
      <H264Layer>
        . . .
      </H264Layer>
    </H264Layers>
    <Chapters />
  </H264Video>
  <AACAudio Condition="InsertSilenceIfNoAudio">
    <Profile>AACLC</Profile>
    <Channels>2</Channels>
    <SamplingRate>48000</SamplingRate>
    <Bitrate>128</Bitrate>
  </AACAudio>
</Encoding>
. . .  
```

## <a name="rotate-a-video"></a><a id="rotate_video"></a>Girar un vídeo
[Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md) admite ángulos de rotación de 0, 90, 180 o 270 grados. El comportamiento predeterminado es "Auto", en el que se intentan detectar los metadatos de rotación del archivo de vídeo entrante y la compensación. Incluya el siguiente elemento **Sources** en uno de los valores preestablecidos definidos en [esta](media-services-mes-presets-overview.md) sección:

### <a name="json-preset"></a>Valor preestablecido JSON

```json
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "Rotation": "90"
      }
    }
  ],
  "Codecs": [

    ...
```

### <a name="xml-preset"></a>Valor preestablecido XML

```xml
<Sources>
  <Source>
    <Streams />
    <Filters>
      <Rotation>90</Rotation>
    </Filters>
  </Source>
</Sources>
```

Consulte también [esto](media-services-mes-schema.md#PreserveResolutionAfterRotation) tema para obtener más información sobre cómo el codificador interpreta la configuración de ancho y alto de los valores preestablecidos cuando se activa la compensación de rotación.

Puede utilizar el valor "0" para indicar al codificador que pase por alto los metadatos de rotación, si están presentes, en la entrada de vídeo.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Consulte también
[Información general sobre la codificación de Media Services](media-services-encode-asset.md)
