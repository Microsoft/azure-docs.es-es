---
title: 'Personalización de un modelo de persona en Video Indexer: Azure'
titlesuffix: Azure Media Services
description: En este artículo se proporciona información general sobre qué es un modelo de persona en Video Indexer y cómo personalizarlo.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: c74b913fc3ac35039d914fc97c9c438d2e4a3069
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799440"
---
# <a name="customize-a-person-model-in-video-indexer"></a>Personalización de un modelo de persona en Video Indexer

Video Indexer es compatible con reconocimiento de celebridades de los vídeos. La característica de reconocimiento de celebridades abarca aproximadamente un millón de caras basadas en orígenes de datos comúnmente solicitados, como IMDB, Wikipedia y personas con más influencia de LinkedIn. Caras que no son reconocidas por Video Indexer se siguió encontrando pero se dejan sin nombre. Los clientes pueden crear modelos de persona personalizados y habilitar Video Indexer reconocer caras que no se reconocen de forma predeterminada. Los clientes pueden crear estos modelos de persona mediante el emparejamiento de un nombre de persona con archivos de imagen de su cara.  

Si su cuenta se encarga de casos de uso diferentes, puede beneficiarse de poder crear varios modelos de persona por cuenta. Por ejemplo, si el contenido de su cuenta está pensado para ordenar en distintos canales, puede crear un modelo de persona independiente para cada canal. 

> [!NOTE]
> Cada modelo de la persona que admite hasta 1 millón de personas y cada cuenta tiene un límite de 50 modelos de persona. 

Una vez que se crea un modelo, para usarlo, proporcione el identificador de modelo de un modelo de persona específico al cargar/indexar o volver a indexar un vídeo. Un nuevo aspecto para ver un vídeo de formación, actualiza el modelo personalizado específico que se ha asociado el vídeo. 

Si no necesita compatibilidad con varios modelos de persona, no asigne un identificador de modelo de persona a su vídeo al cargar/indexar o volver a indexar. En este caso, Video Indexer usará el modelo de la persona de forma predeterminada en su cuenta. 

Puede usar el sitio Web de Video Indexer para editar las caras detectadas en un vídeo y para administrar varios modelos de persona personalizados en su cuenta, como se describe en el [personalizar un modelo de persona con un sitio Web](customize-person-model-with-website.md) tema. También puede usar la API, como se describe en [personalizar un modelo de la persona mediante las API de](customize-person-model-with-api.md).