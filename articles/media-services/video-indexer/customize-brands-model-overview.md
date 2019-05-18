---
title: 'Personalización de un modelo de marcas en Video Indexer: Azure'
titlesuffix: Azure Media Services
description: En este artículo se proporciona información general sobre qué es un modelo de marcas en Video Indexer y cómo personalizarlo.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 863dbd9a6044ee33ae39ac9693a7d4f74382b9c7
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799678"
---
# <a name="customize-a-brands-model-in-video-indexer"></a>Personalización de un modelo de marcas en Video Indexer

Video Indexer permite la detección de marcas por voz y texto visual durante la indexación y la reindexación de contenido de audio y vídeo. La característica de detección de marcas identifica menciones de productos, servicios y compañías sugeridas por la base de datos de marcas de Bing. Por ejemplo, si Microsoft se menciona en un contenido de audio o vídeo, o si se muestra en texto visual en un vídeo, Video Indexer lo detectará como una marca en el contenido. Para eliminar la ambigüedad con otros términos, se usa el contexto.

La detección de marcas es útil en una amplia variedad de escenarios empresariales, como archivo y detección de contenido, publicidad contextual, análisis de medios sociales, análisis de la competencia en el comercio minorista, y mucho más. La detección de marcas de Video Indexer le permite indexar las menciones de marcas en voz y texto visual, mediante una base de datos de marcas de Bing, así como personalizarlas por medio de la creación de un modelo de marcas personalizado para cada cuenta de Video Indexer. La característica de modelo de marcas personalizado le permite seleccionar si Video Indexer detectará o no marcas de la base de datos de marcas de Bing; excluirá determinadas marcas de la detección (mediante la creación de una lista de marcas no permitidas); e incluirá las marcas que deben ser parte del modelo que podrían no estar en la base de datos de marcas de Bing (mediante la creación de una lista de marcas permitidas). El modelo personalizado de marcas que cree solo estará disponible en la cuenta en la que creó el modelo.

## <a name="out-of-the-box-detection-example"></a>Ejemplo de detección predeterminada

En la presentación [Microsoft Build 2017 Day 2](https://www.videoindexer.ai/media/ed6ede78ad/) la marca "Microsoft Windows" aparece varias veces. Una veces en la transcripción, otras como texto visual y nunca como textual. Video Indexer detecta con alta precisión que un término es realmente una marca según el contexto, y abarca más de 90 000 marcas predeterminadas que se actualizan constantemente. A las 02:25, Video Indexer detecta la marca por la voz y de nuevo a las 02:40 por el texto visual, que es parte del logotipo de Windows.

![Introducción a las marcas](./media/content-model-customization/brands-overview.png)

Hablar sobre ventanas en el contexto de la construcción no detectará la palabra "Windows" como marca, y lo mismo sucede con Box, Apple, Fox, etc., según los algoritmos avanzados de Machine Learning que saben como eliminar la ambigüedad por el contexto. La detección de marcas funciona con todos los idiomas admitidos. Haga clic aquí para ver el [vídeo de la conferencia completa de Microsoft Build 2017 Day 2](https://www.videoindexer.ai/media/ed6ede78ad/).

Para traer sus propias marcas, consulte Pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes

[Personalización del modelo de marcas mediante las API](customize-brands-model-with-api.md)

[Customize Brands model using the website](customize-brands-model-with-website.md) (Personalización del modelo de marcas mediante el sitio web)
