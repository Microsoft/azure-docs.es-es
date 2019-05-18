---
title: Conceptos de Video Indexer
titlesuffix: Azure Media Services
description: En este tema se describen algunos conceptos del servicio Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: 156eceba856bf159d4821360639a0641d3ed02be
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799059"
---
# <a name="video-indexer-concepts"></a>Conceptos de Video Indexer
 
En este artículo se describen algunos conceptos del servicio de Video Indexer.
    
## <a name="summarized-insights"></a>Información resumida

La información resumida contiene una vista agregada de los datos: caras, temas y emociones. Por ejemplo, en lugar de pasar por cada uno de los miles de intervalos de tiempo y comprobar qué caras hay en ellos, la información resumida contiene todas las caras y, para cada una de ellas, los intervalos de tiempo en los que aparece y el porcentaje del tiempo en el que se muestra.

## <a name="time-range-vs-adjusted-time-range"></a>intervalo de tiempo frente a intervalo de tiempo ajustado

TimeRange es el intervalo de tiempo del vídeo original. AdjustedTimeRange es el intervalo de tiempo respecto a la lista de reproducción actual. Como puede crear una lista de reproducción a partir de diferentes líneas de vídeos distintas, puede grabar un vídeo de una hora y utilizar solo una línea de él, por ejemplo, de 10:00 a 10:15. En ese caso, tendrá una lista de reproducción con 1 línea, donde el intervalo de tiempo es 10:00-10:15, pero adjustedTimeRange es 00:00-00:15.
 
## <a name="blocks"></a>Blocks

Los bloques están pensados para que resulte más fácil el recorrido por los datos. Por ejemplo, el bloque puede dividirse en función de cuándo cambian los oradores o de si hay una pausa larga.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo empezar, consulte [Suscripción y carga del primer vídeo](video-indexer-get-started.md).

## <a name="see-also"></a>Vea también

[Introducción a Video Indexer](video-indexer-overview.md)
