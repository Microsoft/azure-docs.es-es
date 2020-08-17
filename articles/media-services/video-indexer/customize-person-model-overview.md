---
title: 'Personalización de un modelo de persona en Video Indexer: Azure'
titleSuffix: Azure Media Services
description: En este artículo se proporciona información general sobre qué es un modelo de persona en Video Indexer y cómo personalizarlo.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: kumud
ms.openlocfilehash: 613ad2d0ab90bef016e80a33be12b71cba5e2f71
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87047045"
---
# <a name="customize-a-person-model-in-video-indexer"></a>Personalización de un modelo de persona en Video Indexer

Video Indexer es compatible con el reconocimiento de celebridades de los vídeos. La característica de reconocimiento de celebridades abarca aproximadamente un millón de caras basadas en orígenes de datos comúnmente solicitados, como IMDB, Wikipedia y personas con más influencia de LinkedIn. Las caras que no reconoce Video Indexer también se detectan, pero no se les pone nombre. Los clientes pueden compilar modelos de persona personalizados y habilitar Video Indexer para que reconozca caras que no se reconocen de forma predeterminada. Los clientes pueden compilar estos modelos de persona mediante el emparejamiento de un nombre de persona con archivos de imagen de su cara.  

Si su cuenta sirve a diferentes casos de uso, podría serle beneficioso poder crear varios modelos de persona por cuenta. Por ejemplo, si el contenido de su cuenta está pensado para su ordenación en distintos canales, puede crear un modelo de persona independiente para cada canal. 

> [!NOTE]
> Cada modelo admite hasta 1 millón de personas y cada cuenta tiene un límite de 50 modelos. 

Una vez que se crea un modelo, para usarlo, proporcione el identificador de modelo de un modelo de persona específico al cargar/indexar o volver a indexar un vídeo. Al entrenar una nueva cara para un vídeo, se actualiza el modelo personalizado específico con el que está asociado ese vídeo. 

Si no necesita compatibilidad con varios modelos de persona, no asigne un identificador de modelo de persona a su vídeo al cargar/indexar o volver a indexar. En este caso, Video Indexer usa el modelo de persona personalizado predeterminado de su cuenta. 

Puede usar el sitio web de Video Indexer para editar las caras detectadas en un vídeo y administrar varios modelos de persona personalizados en su cuenta, como se describe en el tema [Customize a Person model using a website](customize-person-model-with-website.md) (Personalización de un modelo de persona mediante un sitio web). También puede usar la API, como se describe en el artículo de  [Personalización de un modelo de persona mediante las API](customize-person-model-with-api.md).
