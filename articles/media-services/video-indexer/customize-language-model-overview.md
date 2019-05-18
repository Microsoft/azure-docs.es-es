---
title: 'Personalización de un modelo de lenguaje en Video Indexer: Azure'
titlesuffix: Azure Media Services
description: En este artículo se proporciona información general sobre qué es un modelo de lenguaje en Video Indexer y cómo personalizarlo.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 516ecd8842e7b673201cc640b283c081a02d2b2f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799553"
---
# <a name="customize-a-language-model-with-video-indexer"></a>Personalización de un modelo de lenguaje con Video Indexer

Video Indexer es compatible con el reconocimiento de voz automático mediante la integración con [Custom Speech Service](https://azure.microsoft.com/services/cognitive-services/custom-speech-service/) de Microsoft. Puede personalizar el modelo de lenguaje mediante la carga de texto de adaptación, es decir, texto del dominio a cuyo vocabulario desea que se adapte el motor. Una vez entrenado el modelo, se reconocerán las nuevas palabras que aparezcan en el texto de adaptación, adoptando la pronunciación predeterminada, y el modelo de lenguaje aprenderá nuevas secuencias probables de palabras. Modelos de lenguaje personalizados se admiten para inglés, español, francés, alemán, italiano, chino (simplificado), japonés, ruso, portugués (Brasil), Hindi y coreano. 

Usemos como ejemplo una palabra muy específica, como "Kubernetes" (en el contexto de Azure Kubernetes Service). Dado que la palabra es nueva en Video Indexer, se reconoce como "comunidades". Es necesario entrenar al modelo para que la reconozca como "Kubernetes". En otros casos, las palabras existen, pero el modelo de lenguaje no espera que aparezcan en un contexto determinado. Por ejemplo, "servicio de contenedor" no es una secuencia de dos palabras que un modelo de lenguaje no especializado reconocería como un conjunto de palabras específico.

Tiene la opción de cargar palabras sin contexto en una lista en un archivo de texto. Se considera una adaptación parcial. También puede cargar archivos de texto de documentación u oraciones relacionadas con el contenido para una mejor adaptación.

Puede usar el sitio web o las API de Video Indexer para crear y editar modelos de lenguaje personalizados, como se describe en los temas de la sección [Pasos siguientes](#next-steps) de este tema.

## <a name="best-practices-for-custom-language-models"></a>Procedimientos recomendados para modelos de lenguaje personalizados

Video Indexer aprende según las probabilidades de combinaciones de palabras, por lo que para que aprenda de la mejor forma posible:

* Ofrézcale suficientes ejemplos de oraciones reales del modo en que se pronunciarían.
* Coloque solo una oración por línea, no más. De lo contrario, el sistema aprenderá probabilidades con las oraciones.
* No pasa nada por poner una palabra como oración para favorecerla frente a otras, pero el sistema aprende mejor de oraciones completas.
* Al introducir palabras o acrónimos nuevos, si es posible, proporcione tantos ejemplos de uso como pueda en una oración completa para ofrecer al sistema tanto contexto como sea posible.
* Intente colocar varias opciones de adaptación y vea cómo funcionan.
* Evite la repetición de la misma oración exacta varias veces. Puede crear un sesgo en el resto de la entrada.
* Evite incluir símbolos raros (~, # @ % &), ya que se descartarán. Las oraciones en que las que aparezcan también se descartarán.
* Evite colocar entradas demasiado grandes, como cientos de miles de oraciones, porque, al hacerlo, reducirá el efecto de impulso.

## <a name="next-steps"></a>Pasos siguientes

[Customize Language model using APIs](customize-language-model-with-api.md) (Personalización del modelo de lenguaje mediante las API)

[Customize Language model using website](customize-language-model-with-website.md) (Personalización del modelo de lenguaje mediante el sitio web)
