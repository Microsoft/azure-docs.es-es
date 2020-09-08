---
title: Generación de un sprite de miniaturas con Azure Media Services | Microsoft Docs
description: En este tema se muestra cómo generar un sprite de miniaturas con Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 08/31/2020
ms.author: juliako
ms.openlocfilehash: 1d05e73bc9aa51bf20121790de94b1b6d0e26615
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268577"
---
# <a name="generate-a-thumbnail-sprite"></a>Generación de un sprite de miniaturas

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Puede usar Media Encoder Standard para generar un sprite de miniaturas, que es un archivo JPEG que contiene varias miniaturas de baja resolución unidas en una sola imagen más grande, junto con un archivo VTT. Este archivo VTT especifica el intervalo de tiempo en el vídeo de entrada que representa cada miniatura, junto con el tamaño y las coordenadas de esa miniatura en el archivo JPEG grande. Los reproductores de vídeo usan el archivo VTT y la imagen de sprite para mostrar una barra de búsqueda "visual" que proporciona un visor con comentarios visuales al arrastrar el cabezal de reproducción hacia adelante y hacia atrás en la escala de tiempo del vídeo.

Para poder usar Media Encoder Standard para generar el sprite de miniaturas, el valor predeterminado debe cumplir estas condiciones:

1. Debe usar el formato de imagen de miniatura JPG.
2. Debe especificar valores de inicio/paso/intervalo como marcas de tiempo o valores de % (y no el número de fotogramas). 
    
    1. Se pueden combinar marcas de tiempo y valores de %.

3. Debe tener el valor SpriteColumn, como un número no negativo mayor o igual que 1.

    1. Si SpriteColumn se establece en M> = 1, la imagen de salida es un rectángulo con M columnas. Si el número de miniaturas generadas mediante #2 no es un múltiplo exacto de M, la última fila estará incompleta y se dejará con píxeles negros.  

Este es un ejemplo:

```json
{
    "Version": 1.0,
    "Codecs": [
    {
      "Start": "00:00:01",
      "Type": "JpgImage",
      "Step": "5%",
      "Range": "100%",
      "JpgLayers": [
        {
          "Type": "JpgLayer",
          "Width": "10%",
          "Height": "10%",
          "Quality": 90
        }
      ],
      "SpriteColumn": 10
    }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Index}{Extension}",
          "Format": {
            "Type": "JpgFormat"
          }
        }
   ]
}
```

## <a name="known-issues"></a>Problemas conocidos

1.  No es posible generar una imagen de sprite con una sola fila de imágenes (SpriteColumn = 1 produce una imagen con una sola columna).
2.  No se admite aún la fragmentación de las imágenes de sprite en imágenes JPEG de tamaño moderado. Por lo tanto, es necesario tener cuidado al limitar el número de miniaturas y su tamaño, para que el sprite de miniaturas unidas resultante esté en torno a 8 megapíxeles o menos.
3.  Azure Media Player admite sprites en los exploradores Microsoft Edge, Chrome y Firefox. No se admite el análisis de VTT en IE11.

## <a name="next-steps"></a>Pasos siguientes

[Codificación de contenido](media-services-encode-asset.md)
