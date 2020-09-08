---
title: Esquema de Media Encoder Standard | Microsoft Docs
description: En este artículo se describen algunos de los elementos y tipos del esquema XML en los que se basan los valores preestablecidos de Media Encoder Standard.
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
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 35f79702b7aad51b764ce2edb37c3c76d1fe98e8
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89261519"
---
# <a name="media-encoder-standard-schema"></a>Esquema de Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este artículo se describen algunos de los elementos y tipos del esquema XML en los que se basan los [valores preestablecidos de Media Encoder Standard](media-services-mes-presets-overview.md). En el artículo se proporciona una explicación de los elementos y sus valores válidos.  

## <a name="preset-root-element"></a><a name="Preset"></a> Preset (elemento raíz)
Define un valor preestablecido de codificación.  

### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Encoding** |[Encoding](media-services-mes-schema.md#Encoding) |Elemento raíz, indica que los orígenes de entrada se van a codificar. |
| **Salidas** |[Salidas](media-services-mes-schema.md#Output) |Colección de archivos de salida deseados. |
| **StretchMode**<br/>minOccurs="0"<br/>default="AutoSize|xs:string|Controle el tamaño del fotograma de vídeo de salida, el relleno, los píxeles o la relación de aspecto de pantalla. **StretchMode** podría ser uno de los siguientes valores: **Ninguno**, **AutoSize** (valor predeterminado) o **AutoFit**.<br/><br/>**Ninguna**: Siga estrictamente la resolución de salida (por ejemplo, el **ancho** y **alto** del valor preestablecido) sin tener en cuenta la proporción de píxeles o relación de aspecto de pantalla del vídeo de entrada. Se recomienda en escenarios como los de [recorte](media-services-crop-video.md), donde el vídeo de salida tiene una relación de aspecto diferente en comparación con la entrada. <br/><br/>**AutoSize**: la resolución de los resultados cabrá dentro de la ventana (ancho * alto) especificado por el valor predeterminado. Sin embargo, el codificador genera un vídeo de salida que tiene la relación de aspecto de píxel cuadrada (1:1). Por lo tanto, el ancho o alto de salida se pueden invalidar para que coincida con la relación de aspecto de pantalla de la entrada, sin relleno. Por ejemplo, si la entrada es 1920 x 1080 y el valor preestablecido de codificación requiere 1280 x 1280, se invalida el valor del alto en el valor preestablecido y la salida será de 1280 x 720, que mantiene la relación de aspecto de entrada de 16:9. <br/><br/>**AutoFit**: si es necesario, rellene el vídeo de salida (con formato letterbox o pillarbox) para que admita la resolución de salida que quiera, asegurándose de que la región activa del vídeo en la salida tenga la misma relación de aspecto que la entrada. Por ejemplo, supongamos que la entrada es de 1920 x 1080 y el valor preestablecido de codificación requiere 1280 x 1280. A continuación, el vídeo de salida estará en 1280 x 1280, pero contendrá un rectángulo de 1280 x 720 interno de "vídeo activo" con la relación de aspecto de 16:9 y regiones con formato letterbox y 280 píxeles de alto en la parte superior e inferior. Otro ejemplo, si la entrada es de 1440 x 1080 y el valor preestablecido de codificación requiere 1280 x 720, la salida será de 1280 x 720, que contiene un rectángulo interno de 960 x 720 con relación de aspecto de 4:3, y regiones con formato pillarbox y 160 píxeles de ancho a la izquierda y derecha. 

### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Versión**<br/><br/> Obligatorio |**xs: decimal** |La versión del valor preestablecido. Se aplican las restricciones siguientes: xs:fractionDigits value="1" y xs:minInclusive value="1". Por ejemplo, **version="1.0"** . |

## <a name="encoding"></a><a name="Encoding"></a> Encoding
Contiene una secuencia de los elementos siguientes:  

### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **H264Video** |[H264Video](media-services-mes-schema.md#H264Video) |Configuración de la codificación H.264 de vídeo. |
| **AACAudio** |[AACAudio](media-services-mes-schema.md#AACAudio) |Configuración de la codificación AAC de audio. |
| **BmpImage** |[BmpImage](media-services-mes-schema.md#BmpImage) |Configuración de la imagen Bmp. |
| **PngImage** |[PngImage](media-services-mes-schema.md#PngImage) |Configuración de la imagen Png. |
| **JpgImage** |[JpgImage](media-services-mes-schema.md#JpgImage) |Configuración de la imagen Jpg. |

## <a name="h264video"></a><a name="H264Video"></a> H264Video
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **TwoPass**<br/><br/> minOccurs="0" |**xs:boolean** |Actualmente, solo se admite la codificación en un solo paso. |
| **KeyFrameInterval**<br/><br/> minOccurs="0"<br/><br/> **default="00:00:02"** |**xs:time** |Determina el espaciado fijo entre fotogramas IDR en unidades de segundos. También se conoce como duración de GOP. Vea **SceneChangeDetection** para controlar si el codificador puede desviarse de este valor. |
| **SceneChangeDetection**<br/><br/> minOccurs="0"<br/><br/> default=”false” |**xs: boolean** |Si está establecido en true, el codificador intenta detectar cambios de escena en el vídeo e inserta un fotograma IDR. |
| **Complexity**<br/><br/> minOccurs="0"<br/><br/> default="Balanced" |**xs:string** |Controla el equilibrio entre velocidad de codificación y calidad de vídeo. Podría ser uno de los siguientes valores: **Velocidad**, **Equilibrada**, o **Calidad**<br/><br/> Valor predeterminado: **Equilibrada** |
| **SyncMode**<br/><br/> minOccurs="0" | |La característica se expondrá en una versión futura. |
| **H264Layers**<br/><br/> minOccurs="0" |[H264Layers](media-services-mes-schema.md#H264Layers) |Colección de capas de vídeo de salida. |

### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Condition** |**xs:string** | Cuando la entrada no tiene ningún vídeo, puede forzar que el codificador inserte una pista de vídeo monocromática. Para ello, utilice Condition="InsertBlackIfNoVideoBottomLayerOnly" (para insertar un vídeo con la mínima velocidad de bits) o Condition="InsertBlackIfNoVideo" (para insertar un vídeo con todas las velocidades de bits de salida). Para obtener más información, consulte [este](media-services-advanced-encoding-with-mes.md#no_video) artículo.|

## <a name="h264layers"></a><a name="H264Layers"></a> H264Layers

De forma predeterminada, si envía una entrada al codificador que solo contenga audio, y no vídeo, el recurso de salida contiene archivos que, a su vez, solo contienen datos de audio. Algunos reproductores no puede controlar estos flujos de salida. Puede usar la configuración del atributo **InsertBlackIfNoVideo** de H264Video para forzar al codificador a agregar una pista de vídeo monocromática a la salida en ese escenario. Para obtener más información, consulte [este](media-services-advanced-encoding-with-mes.md#no_video) artículo.
              
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **H264Layer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[H264Layer](media-services-mes-schema.md#H264Layer) |Una colección de capas H264. |

## <a name="h264layer"></a><a name="H264Layer"></a> H264Layer
> [!NOTE]
> Los límites de vídeo se basan en los valores descritos en la tabla de [niveles de H264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Levels).  
> 
> 

### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Perfil**<br/><br/> minOccurs="0"<br/><br/> default=”Auto” |**xs: string** |Puede ser de uno de los siguientes valores **xs: string**: **Auto**, **Baseline**, **Main** o **High**. |
| **Level**<br/><br/> minOccurs="0"<br/><br/> default=”Auto” |**xs: string** | |
| **Bitrate**<br/><br/> minOccurs="0" |**xs:int** |La velocidad de bits usada para esta capa de vídeo, especificada en Kbps. |
| **MaxBitrate**<br/><br/> minOccurs="0" |**xs: int** |La velocidad de bits máxima usada para esta capa de vídeo, especificada en Kbps. |
| **BufferWindow**<br/><br/> minOccurs="0"<br/><br/> default="00:00:05" |**xs: time** |Longitud del búfer de vídeo. |
| **Width**<br/><br/> minOccurs="0" |**xs: int** |Ancho del fotograma de vídeo de salida, en píxeles.<br/><br/> Actualmente, debe especificar ancho y alto. El ancho y el alto deben ser números pares. |
| **Height**<br/><br/> minOccurs="0" |**xs:int** |Alto del fotograma de vídeo de salida, en píxeles.<br/><br/> Actualmente, debe especificar ancho y alto. El ancho y el alto deben ser números pares.|
| **BFrames**<br/><br/> minOccurs="0" |**xs: int** |Número de fotogramas B entre fotogramas de referencia. |
| **ReferenceFrames**<br/><br/> minOccurs="0"<br/><br/> default=”3” |**xs:int** |Número de fotogramas de referencia en un GOP. |
| **EntropyMode**<br/><br/> minOccurs="0"<br/><br/> default=”Cabac” |**xs: string** |Podría ser uno de los siguientes valores: **Cabac** y **Cavlc**. |
| **FrameRate**<br/><br/> minOccurs="0" |número racional |Determina la velocidad de fotogramas del vídeo de salida. Use el valor predeterminado "0/1" para permitir que el codificador use la misma velocidad de fotogramas que el vídeo de entrada. Se espera que los valores permitidos sean velocidades de fotogramas de vídeo habituales. No obstante, se admite cualquier número racional válido. Por ejemplo, 1/1 sería 1 fps y es válido.<br/><br/> - 12/1  (12 fps)<br/><br/> - 15/1 (15 fps)<br/><br/> - 24/1 (24 fps)<br/><br/> - 24000/1001 (23,976 fps)<br/><br/> - 25/1 (25 fps)<br/><br/>  - 30/1 (30 fps)<br/><br/> - 30000/1001 (29,97 fps) <br/> <br/>**NOTA** Si está creando un valor preestablecido personalizado para la codificación de velocidad de bits múltiple, todas las capas del valor preestablecido **deben** tener el mismo valor en FrameRate.|
| **AdaptiveBFrame**<br/><br/> minOccurs="0" |**xs: boolean** |Copia de Azure Media Encoder. |
| **Slices**<br/><br/> minOccurs="0"<br/><br/> default="0" |**xs:int** |Determina en cuántos segmentos se divide un fotograma. Se recomienda usar el valor predeterminado. |

## <a name="aacaudio"></a><a name="AACAudio"></a> AACAudio
 Contiene una secuencia de los elementos y grupos siguientes.  

 Para más información acerca de AAC, consulte [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding).  

### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Perfil**<br/><br/> minOccurs="0 "<br/><br/> default="AACLC" |**xs: string** |Podría ser uno de los siguientes valores: **AACLC**, **HEAACV1** o **HEAACV2**. |

### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Condition** |**xs: string** |Para forzar al codificador a producir un activo que contiene una pista de audio silenciosa cuando la entrada no tiene audio, especifique el valor de "InsertSilenceIfNoAudio".<br/><br/> De forma predeterminada, si envía una entrada al codificador que solo contenga vídeo, y no audio, el recurso de salida contendrá archivos que solo contienen datos de vídeos. Algunos reproductores no puede controlar estos flujos de salida. Puede usar este ajuste para forzar al codificador a agregar una pista de audio silenciosa a la salida en ese escenario. |

### <a name="groups"></a>Grupos

| Referencia | Descripción |
| --- | --- |
| [AudioGroup](media-services-mes-schema.md#AudioGroup)<br/><br/> minOccurs="0" |Consulte la descripción de [AudioGroup](media-services-mes-schema.md#AudioGroup) para saber el número adecuado de canales, la velocidad de muestreo y la velocidad de bits que se pueden establecer para cada perfil. |

## <a name="audiogroup"></a><a name="AudioGroup"></a> AudioGroup
Para detalles sobre qué valores son válidos para cada perfil, consulte la tabla "Detalles de códec de audio" más adelante.  

### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Channels**<br/><br/> minOccurs="0" |**xs: int** |El número de canales de audio codificados. Las opciones válidas son: 1, 2, 5, 6, 8.<br/><br/> Valor predeterminado: 2. |
| **SamplingRate**<br/><br/> minOccurs="0" |**xs: int** |La velocidad de muestreo de audio, especificada en Hz. |
| **Bitrate**<br/><br/> minOccurs="0" |**xs: int** |Velocidad de bits usada al codificar el audio, especificada en Kbps. |

### <a name="audio-codec-details"></a>Detalles de códec de audio

Códec de audio|Detalles  
-----------------|---  
**AACLC** |1:<br/><br/> - 11025: 8 &lt;= velocidad de bits &lt; 16<br/><br/> - 12000: 8 &lt;= velocidad de bits &lt; 16<br/><br/> - 16000: 8 &lt;= velocidad de bits &lt;32<br/><br/>- 22050: 24 &lt;= velocidad de bits &lt; 32<br/><br/> - 24000: 24 &lt;= velocidad de bits &lt; 32<br/><br/> - 32000: 32 &lt;= velocidad de bits &lt;= 192<br/><br/> - 44100: 56 &lt;= velocidad de bits &lt;= 288<br/><br/> - 48000: 56 &lt;= velocidad de bits &lt;= 288<br/><br/> - 88200: 128 &lt;= velocidad de bits &lt;= 288<br/><br/> - 96000 : 128 &lt;= velocidad de bits &lt;= 288<br/><br/> 2:<br/><br/> - 11025: 16 &lt;= velocidad de bits &lt; 24<br/><br/> - 12000: 16 &lt;= velocidad de bits &lt; 24<br/><br/> - 16000: 16 &lt;= velocidad de bits &lt; 40<br/><br/> - 22050: 32 &lt;= velocidad de bits &lt; 40<br/><br/> - 24000 : 32 &lt;= velocidad de bits &lt; 40<br/><br/> - 32000:  40 &lt;= velocidad de bits &lt;= 384<br/><br/> - 44100: 96 &lt;= velocidad de bits &lt;= 576<br/><br/> - 48000: 96 &lt;= velocidad de bits &lt;= 576<br/><br/> - 88200: 256 &lt;= velocidad de bits &lt;= 576<br/><br/> - 96000: 256 &lt;= velocidad de bits &lt;= 576<br/><br/> 5/6:<br/><br/> - 32000: 160 &lt;= velocidad de bits &lt;= 896<br/><br/> - 44100: 240 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 48000: 240 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 88200: 640 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 96000: 640 &lt;= velocidad de bits &lt;= 1024<br/><br/> 8:<br/><br/> - 32000: 224 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 44100: 384 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 48000: 384 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 88200: 896 &lt;= velocidad de bits &lt;= 1024<br/><br/> - 96000: 896 &lt;= velocidad de bits &lt;= 1024  
**HEAACV1** |1:<br/><br/> - 22050: velocidad de bits = 8<br/><br/> - 24000: 8 &lt;= velocidad de bits &lt;= 10<br/><br/> - 32000: 12 &lt;= velocidad de bits &lt;= 64<br/><br/> - 44100: 20 &lt;= velocidad de bits &lt;= 64<br/><br/> - 48000: 20 &lt;= velocidad de bits &lt;= 64<br/><br/> - 88200: velocidad de bits = 64<br/><br/> 2:<br/><br/> - 32000: 16 &lt;= velocidad de bits &lt;= 128<br/><br/> - 44100: 16 &lt;= velocidad de bits &lt;= 128<br/><br/> - 48000: 16 &lt;= velocidad de bits &lt;= 128<br/><br/> - 88200: 96 &lt;= velocidad de bits &lt;= 128<br/><br/> - 96000: 96 &lt;= velocidad de bits &lt;= 128<br/><br/> 5/6:<br/><br/> - 32000: 64 &lt;= velocidad de bits &lt;= 320<br/><br/> - 44100: 64 &lt;= velocidad de bits &lt;= 320<br/><br/> - 48000: 64 &lt;= velocidad de bits &lt;= 320<br/><br/> - 88200: 256 &lt;= velocidad de bits &lt;= 320<br/><br/> - 96000: 256 &lt;= velocidad de bits &lt;= 320<br/><br/> 8:<br/><br/> - 32000: 96 &lt;= velocidad de bits &lt;= 448<br/><br/> - 44100: 96 &lt;= velocidad de bits &lt;= 448<br/><br/> - 48000: 96 &lt;= velocidad de bits &lt;= 448<br/><br/> - 88200: 384 &lt;= velocidad de bits &lt;= 448<br/><br/> - 96000: 384 &lt;= velocidad de bits &lt;= 448  
**HEAACV2** |2:<br/><br/> - 22050: 8 &lt;= velocidad de bits &lt;= 10<br/><br/> - 24000: 8 &lt;= velocidad de bits &lt;= 10<br/><br/> - 32000: 12 &lt;= velocidad de bits &lt;= 64<br/><br/> - 44100: 20 &lt;= velocidad de bits &lt;= 64<br/><br/> - 48000: 20 &lt;= velocidad de bits &lt;= 64<br/><br/> - 88200: 64 &lt;= velocidad de bits &lt;= 64  
  
## <a name="clip"></a><a name="Clip"></a> Clip
### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **StartTime** |**xs:duration** |Especifica la hora de inicio de una presentación. El valor de StartTime debe coincidir con las marcas de tiempo absoluto de la entrada de vídeo. Por ejemplo, si el primer fotograma del vídeo de entrada tiene la marca de tiempo 12:00:10.000, StartTime debe ser al menos 12:00:10.000 o un valor superior. |
| **Duration** |**xs:duration** |Especifica la duración de una presentación (por ejemplo, la aparición de una superposición en el vídeo). |

## <a name="output"></a><a name="Output"></a> Output
### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **FileName** |**xs:string** |Nombre del archivo de salida.<br/><br/> Puede usar macros, descritas en la tabla siguiente, para generar los nombres de archivo de salida. Por ejemplo:<br/><br/> **"Outputs": [      {       "FileName": "{Basename} *{Resolution}* {Bitrate}.mp4",       "Format": {         "Type": "MP4Format"       }     }   ]** |

### <a name="macros"></a>Macros

| Macro | Descripción |
| --- | --- |
| **{Basename}** |Si está realizando la codificación de vídeo bajo demanda, {Basename} son los 32 primeros caracteres de la propiedad AssetFile.Name del archivo principal en el recurso de entrada.<br/><br/> Si el recurso de entrada es un archivo dinámico, {Basename} se deriva de los atributos de trackName en el manifiesto del servidor. Si va a enviar un trabajo de subclip con TopBitrate, como en "<VideoStream\>TopBitrate</VideoStream\>", y el archivo de salida contiene vídeo, {Basename} son los 32 primeros caracteres del elemento trackName de la capa de vídeo con la velocidad de bits más alta.<br/><br/> Si en su lugar envía un trabajo de subclip con todas las velocidades de bits de entrada, como "<VideoStream\>*</VideoStream\>", y el archivo de salida contiene vídeo, {Basename} son los 32 primeros caracteres del elemento trackName de la capa de vídeo correspondiente. |
| **{Codec}** |Se asigna a "H264" para el vídeo y a "AAC" para el audio. |
| **{Bitrate}** |La velocidad de bits de vídeo de destino si el archivo de salida contiene vídeo y audio, o la velocidad de bits de audio de destino si el archivo de salida solo contiene audio. El valor que se utiliza es la velocidad de bits en Kbps. |
| **{Channel}** |Número de canales de audio si el archivo contiene audio. |
| **{Width}** |Ancho del vídeo, en píxeles, en el archivo de salida, si el archivo contiene vídeo. |
| **{Height}** |Alto del vídeo, en píxeles, en el archivo de salida, si el archivo contiene vídeo. |
| **{Extension}** |Hereda de la propiedad "Type" del archivo de salida. El nombre del archivo de salida tiene una de estas extensiones: “mp4”, “ts”, “jpg”, “png” o “bmp”. |
| **{Index}** |Obligatorio para una miniatura. Solo debe estar presente una vez. |

## <a name="video-complex-type-inherits-from-codec"></a><a name="Video"></a> Video (tipo complejo que hereda de Codec)
### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Iniciar** |**xs:string** | |
| **Step** |**xs:string** | |
| **Range** |**xs:string** | |
| **PreserveResolutionAfterRotation** |**xs:boolean** |Consulte una explicación detallada en la sección siguiente: [PreserveResolutionAfterRotation](media-services-mes-schema.md#PreserveResolutionAfterRotation) |

### <a name="preserveresolutionafterrotation"></a><a name="PreserveResolutionAfterRotation"></a> PreserveResolutionAfterRotation
Se recomienda usar la marca **PreserveResolutionAfterRotation** en combinación con valores de resolución expresados en términos porcentuales (Width="100 %", Height = "100 %").  

De forma predeterminada, la configuración de resolución de codificación (Width, Height) en los valores preestablecidos de Media Encoder Standard (MES) va dirigida a vídeos con rotación de 0 grados. Por ejemplo, si el vídeo de entrada es 1280 × 720 con una rotación de cero grados, los valores preestablecidos garantizan que la salida tenga la misma resolución.  

![MESRoation1](./media/media-services-shemas/media-services-mes-roation1.png) 

Si el vídeo de entrada se ha capturado con una rotación que no sea cero (por ejemplo, una tableta o smartphone sostenidos verticalmente), MES aplicará de forma predeterminada la configuración de resolución de codificación (Width, Height) al vídeo de entrada y después compensará la rotación. Por ejemplo, vea la imagen siguiente. El valor predeterminado usa Width = “100%”, Height = “100%”, que MES interpreta como que es obligatorio que la salida tenga 1280 píxeles de ancho y 720 píxeles de alto. Después de girar el vídeo, reduce la imagen para ajustarla a esa ventana, lo que produce franjas negras a izquierda y derecha.  

![MESRoation2](./media/media-services-shemas/media-services-mes-roation2.png) 

Alternativamente, puede usar la marca **PreserveResolutionAfterRotation** y establecerla en “true” (el valor predeterminado es “false”). Por tanto, si su valor preestablecido tiene Width = “100 %”, Height = “100 %” y PreserveResolutionAfterRotation se ha establecido en "true", un vídeo de entrada que tenga 1280 píxeles de ancho y 720 de alto con una rotación de 90 grados genera una salida con rotación de cero grados, pero con 720 píxeles de ancho y 1280 de alto. Vea la siguiente imagen:  

![MESRoation3](./media/media-services-shemas/media-services-mes-roation3.png) 

## <a name="formatgroup-group"></a><a name="FormatGroup"></a> FormatGroup (grupo)
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **BmpFormat** |**BmpFormat** | |
| **PngFormat** |**PngFormat** | |
| **JpgFormat** |**JpgFormat** | |

## <a name="bmplayer"></a><a name="BmpLayer"></a> BmpLayer
### <a name="element"></a>Elemento

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="pnglayer"></a><a name="PngLayer"></a> PngLayer
### <a name="element"></a>Elemento

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="jpglayer"></a><a name="JpgLayer"></a> JpgLayer
### <a name="element"></a>Elemento

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |
| **Quality**<br/><br/> minOccurs="0" |**xs:int** |Valores válidos:  1(worst)-100(best) |

### <a name="attributes"></a>Atributos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="pnglayers"></a><a name="PngLayers"></a> PngLayers
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **PngLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[PngLayer](media-services-mes-schema.md#PngLayer) | |

## <a name="bmplayers"></a><a name="BmpLayers"></a> BmpLayers
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **BmpLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[BmpLayer](media-services-mes-schema.md#BmpLayer) | |

## <a name="jpglayers"></a><a name="JpgLayers"></a> JpgLayers
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **JpgLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[JpgLayer](media-services-mes-schema.md#JpgLayer) | |

## <a name="bmpimage-complex-type-inherits-from-video"></a><a name="BmpImage"></a> BmpImage (tipo complejo que hereda de Video)
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Capas Png |

## <a name="jpgimage-complex-type-inherits-from-video"></a><a name="JpgImage"></a> JpgImage (tipo complejo que hereda de Video)
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Capas Png |

## <a name="pngimage-complex-type-inherits-from-video"></a><a name="PngImage"></a> PngImage (tipo complejo que hereda de Video)
### <a name="elements"></a>Elementos

| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Capas Png |

## <a name="examples"></a>Ejemplos
Consulte ejemplos de valores preestablecidos de XML que se generan a partir de este esquema en [Task Presets for MES (Media Encoder Standard)](media-services-mes-presets-overview.md) (Valores preestablecidos de tareas para MES [Media Encoder Standard]).

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

