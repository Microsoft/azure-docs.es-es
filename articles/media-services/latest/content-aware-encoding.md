---
title: 'Un valor predeterminado para la codificación que tiene en cuenta el contenido: Azure Media Services'
description: En este artículo se describe la codificación según el contenido en Microsoft Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 5d08e09905841f6068f2bac45680a8e5a011c158
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297372"
---
# <a name="use-the-content-aware-encoding-preset-to-find-the-optimal-bitrate-value-for-a-given-resolution"></a>Use el valor predeterminado de la codificación en función del contenido para encontrar el valor de velocidad de bits óptimo para una resolución dada

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Para poder preparar el contenido para la entrega mediante [streaming con velocidad de bits adaptable](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming), el vídeo debe codificarse en varias velocidades de bits (de alta a baja). Esto garantiza una degradación correcta de la calidad, ya que la resolución del vídeo disminuye a medida que lo hace la velocidad de bits. Esta codificación con velocidad de bits múltiple utiliza algo que se conoce como escalera de codificación (una table de resoluciones y velocidades de bits, consulte los [valores predeterminados de codificación integrados](/rest/api/media/transforms/createorupdate#encodernamedpreset) de Media Services).

Debe tener en cuenta el contenido que va a procesar y personalizar o ajustar la escalera de codificación a la complejidad del vídeo individual. A cada resolución le corresponde una velocidad de bits más allá de la cual no se percibe un aumento en la calidad: el codificador funciona en este valor de velocidad de bits óptimo. El siguiente nivel de optimización consiste en seleccionar las resoluciones en función del contenido: por ejemplo, un vídeo de una presentación de PowerPoint no saca ningún provecho al bajar de 720p. Más adelante, el codificador puede encargarse de optimizar la configuración de cada captura dentro del vídeo. 

El valor predeterminado de [streaming adaptable](autogen-bitrate-ladder.md) de Microsoft soluciona parcialmente el problema de la variabilidad de calidad y resolución de los vídeos de origen. El contenido de nuestros clientes es muy variado: algunos vídeos tienen una resolución de 1080p, otros 720p y algunos tienen resoluciones de SD e inferiores. Además, no todo el contenido de origen está en formato mezzanine de alta calidad procedente de estudios de televisión o cinematográficos. El valor preestablecido de streaming adaptable soluciona estos problemas garantizando que la escala de velocidad de bits nunca supere la resolución o la velocidad de bits media del archivo mezzanine de entrada. Sin embargo, las únicas propiedades del origen que examina este valor predeterminado son la resolución y la velocidad de bits.

## <a name="the-content-aware-encoding"></a>Codificación en función del contenido 

El valor predeterminado de codificación en función del contenido amplía el mecanismo de "streaming con velocidad de bits adaptable", para lo que incorpora una lógica personalizada que permite al codificador buscar el valor óptimo de velocidad de bits para una resolución determinada sin requerir un análisis de cálculo exhaustivo. Este valor predefinido produce un conjunto de archivos MP4s alineados con GOP. Dado cualquier contenido de entrada, el servicio realiza un análisis ligero inicial del mismo y usa los resultados para determinar automáticamente el número óptimo de capas, la velocidad de bits adecuada y la configuración de resolución para la entrega a través del streaming adaptable. Este valor predefinido resulta particularmente eficaz en los vídeos de complejidad media y baja, donde los archivos de salida tendrán velocidades de bits más lentas que el valor predefinido de streaming adaptable, pero una calidad que seguirá ofreciendo una buena experiencia a los visores. La salida contendrá archivos MP4 con el vídeo y audio intercalados.

En los siguientes gráficos de ejemplo se muestra la comparación usando métricas de calidad como [PSNR](https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio) y [VMAF](https://en.wikipedia.org/wiki/Video_Multimethod_Assessment_Fusion). El origen se creó mediante la concatenación de clips cortos de capturas de gran complejidad de películas y programas de TV con la finalidad de destacar el codificador. Por definición, este valor predeterminado genera resultados que varían de contenido en contenido. Esto también significa que, para cierto contenido, puede ser que no se produzca una reducción significativa de la velocidad de bits ni una mejora en la calidad.

![Curva de distorsión de la velocidad (RD) mediante PSNR](media/content-aware-encoding/msrv1.png)

**Figura 1: Curva de distorsión de la velocidad (RD) con la métrica de PSNR para un origen de gran complejidad**

![Curva de distorsión de la velocidad (RD) mediante VMAF](media/content-aware-encoding/msrv2.png)

**Figura 2: Curva de distorsión de la velocidad (RD) con la métrica de VMAF para un origen de gran complejidad**

A continuación encontrará los resultados de otra categoría de contenido de origen, en la que el codificador pudo determinar que la entrada era de baja calidad (muchos artefactos de compresión debido a una baja velocidad de bits). Tenga en cuenta que, con el valor preestablecido en función del contenido, el codificador decidió generar solo una capa de salida a una velocidad de bits lo suficientemente baja para que la mayoría de los clientes pudieran reproducir la transmisión sin sufrir un estancamiento.

![Curva de RD mediante PSNR](media/content-aware-encoding/msrv3.png)

**Figura 3: Curva de RD mediante PSNR para entradas de baja calidad (a 1080p)**

![Curva de RD mediante VMAF](media/content-aware-encoding/msrv4.png)

**Ilustración 4: Curva de RD mediante VMAF para entradas de baja calidad (a 1080p)**

## <a name="how-to-use-the-content-aware-encoding-preset"></a>Uso del valor predeterminado de codificación en función del contenido 

Puede crear transformaciones que usen este valor preestablecido como se indica a continuación. 

Consulte la sección [Pasos siguientes](#next-steps) para ver otros tutoriales que usan salidas para la transformación. El recurso de la salida se puede entregar desde los puntos de conexión de streaming de Media Services en protocolos como MPEG-DASH y HLS (como se muestra en los tutoriales).

> [!NOTE]
> Asegúrese de usar el valor preestablecido **ContentAwareEncoding**, en lugar de ContentAwareEncodingExperimental.

```csharp
TransformOutput[] output = new TransformOutput[]
{
   new TransformOutput
   {
      // The preset for the Transform is set to one of Media Services built-in sample presets.
      // You can customize the encoding settings by changing this to use "StandardEncoderPreset" class.
      Preset = new BuiltInStandardEncoderPreset()
      {
         // This sample uses the new preset for content-aware encoding
         PresetName = EncoderNamedPreset.ContentAwareEncoding
      }
   }
};
```

> [!NOTE]
> Los trabajos de codificación que usan el valor predeterminado `ContentAwareEncoding` se facturan en función de los minutos de salida. 
  
## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Carga, codificación y streaming de vídeos con Media Services v3](stream-files-tutorial-with-api.md)
* [Tutorial: Carga, codificación y transmisión de vídeos con REST](stream-files-tutorial-with-rest.md)
* [Tutorial: Codificación de un archivo remoto según una dirección URL y transmisión del vídeo: CLI](stream-files-cli-quickstart.md)
* [Tutorial: Codificación de un archivo remoto según una dirección URL y transmisión del vídeo: .NET](stream-files-dotnet-quickstart.md)
* [Tutorial: Codificación de un archivo remoto según una dirección URL y transmisión del vídeo: Node.js](stream-files-nodejs-quickstart.md)
