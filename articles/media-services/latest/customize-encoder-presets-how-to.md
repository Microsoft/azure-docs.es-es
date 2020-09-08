---
title: Codificación de una transformación personalizada mediante Media Services v3 con .NET - Azure | Microsoft Docs
description: En este tema se explica cómo usar Azure Media Services v3 para codificar una transformación personalizada mediante .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: b5bb505df3cad7856e0b08f04949c2e56ccec1ca
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295621"
---
# <a name="how-to-encode-with-a-custom-transform---net"></a>Procedimiento de codificación con una transformación personalizada - .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Al codificar con Azure Media Services, puede empezar a trabajar rápidamente con cualquiera de los valores preestablecidos integrados recomendados basados en los procedimientos recomendados del sector, como se muestra en el tutorial [Transmisión de archivos](stream-files-tutorial-with-api.md). También puede compilar un valor preestablecido personalizado para sus requisitos específicos de escenario o dispositivo.

## <a name="considerations"></a>Consideraciones

Al crear valores preestablecidos personalizados, se aplican las consideraciones siguientes:

* Todos los valores de alto y ancho del contenido de AVC deben ser un múltiplo de 4.
* En Azure Media Services v3, todas las velocidades de bits de codificación se expresan en bits por segundo. Esto difiere de los valores preestablecidos en las API v2, en donde se usaban kilobits por segundo como unidad. Por ejemplo, si la velocidad de bits de v2 se especificaba como 128 (kilobits/segundo), en v3 se establecería en 128 000 (bits/segundo).

## <a name="prerequisites"></a>Prerequisites 

[Creación de una cuenta de Media Services](./create-account-howto.md)

## <a name="download-the-sample"></a>Descarga del ejemplo

Clone un repositorio GitHub que contenga el ejemplo de .NET Core completo en la máquina mediante el siguiente comando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials.git
 ```
 
El ejemplo de valor preestablecido personalizado se encuentra en la carpeta [EncodeCustomTransform](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/EncodeCustomTransform/).

## <a name="create-a-transform-with-a-custom-preset"></a>Creación de una transformación con un valor preestablecido personalizado 

Al crear una nueva [transformación](/rest/api/media/transforms), debe especificar qué desea originar como salida. El parámetro requerido es un objeto [TransformOutput](/rest/api/media/transforms/createorupdate#transformoutput), como se muestra en el código siguiente. Cada objeto **TransformOutput** contiene un **valor preestablecido**. El **valor preestablecido** describe las instrucciones paso a paso de las operaciones de procesamiento de vídeo o audio que se van a usar para generar el objeto **TransformOutput** deseado. El siguiente objeto **TransformOutput** crea el códec personalizado y la configuración de salida de la capa.

Al crear una [transformación](/rest/api/media/transforms), debe comprobar primero si ya existe una con el método **Get**, tal como se muestra en el código siguiente. En Media Services v3, los métodos **Get** en las entidades devuelven **null** si la entidad no existe (comprobación sin distinción entre mayúsculas y minúsculas en el nombre).

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se define un conjunto de salidas que queremos que se genere cuando se use esta transformación. En primer lugar se agrega una capa AacAudio para la codificación de audio y dos capas H264Video para la codificación de vídeo. En las capas de vídeo se asignan etiquetas para que se puedan usar en los nombres de archivo de salida. Luego queremos que la salida también incluya miniaturas. En el ejemplo siguiente se especifican imágenes en formato PNG generadas al 50 % de la resolución del vídeo de entrada, y en tres marcas de tiempo, {25 %, 50 %, 75 %}, de la longitud del vídeo de entrada. Por último, se especifica el formato de los archivos de salida: uno para vídeo y audio y otro para las miniaturas. Puesto que hay varias H264Layers, tenemos que usar macros que generen nombres únicos por capa. Se puede usar una macro `{Label}` o `{Bitrate}`; el ejemplo muestra la primera.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/EncodeCustomTransform/MediaV3ConsoleApp/Program.cs#EnsureTransformExists)]

## <a name="next-steps"></a>Pasos siguientes

[Streaming de archivos](stream-files-tutorial-with-api.md) 
