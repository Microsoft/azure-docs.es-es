---
title: Aprendizaje de refuerzo (Personalizer)
titleSuffix: Azure Cognitive Services
description: Personalizer usa información acerca de las acciones y del contexto para realizar sugerencias para mejorar la clasificación. La información acerca de estas acciones y del contexto son atributos o propiedades que se conocen como características.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 05/07/2019
ms.openlocfilehash: 8b97221de4921e06ddfab610618f37683b990181
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132745"
---
# <a name="what-is-reinforcement-learning"></a>¿Qué es el aprendizaje de refuerzo?

El aprendizaje de refuerzo es un enfoque hacia el aprendizaje automático que aprende los comportamientos mediante la obtención de comentarios de su uso.
 
Así funciona el aprendizaje de refuerzo:

* Proporciona una oportunidad o el grado de libertad necesarios para aplicar un comportamiento (como tomar decisiones o elegir entre distintas opciones).
* Proporciona información contextual acerca del entorno y de las distintas opciones.
* Proporciona comentarios acerca del grado hasta el que el comportamiento logra un objetivo específico.

Aunque hay muchos subtipos y estilos de aprendizaje reforzado, así es como funciona el concepto en Personalizer:

* La aplicación permite mostrar una parte del contenido de una lista de alternativas.
* La aplicación proporciona información acerca de cada alternativa y del contexto del usuario.
* La aplicación calcula una _puntuación de recompensa_.

A diferencia de algunos enfoques hacia el aprendizaje de refuerzo, Personalizer no requiere una simulación para que funcione. Sus algoritmos de aprendizaje están diseñados para reaccionar ante el mundo exterior (en lugar de controlarlo) y aprender de cada punto de datos sabiendo que es una oportunidad única que cuesta tiempo y dinero crear, y que si se produce un rendimiento que no llega a ser óptimo, el lamento (pérdida de una posible recompensa) no es cero.

## <a name="what-type-of-reinforcement-learning-algorithms-does-personalizer-use"></a>¿Qué tipo de algoritmos de aprendizaje de refuerzo usa Personalizer?

La versión actual de Personalizer usa **bandidos contextuales**, un enfoque hacia el aprendizaje de refuerzo enmarcado en torno a la toma de decisiones y la elección de opciones entre acciones discretas, en un contexto determinado.

La _memoria de las decisiones_, el modelo que se ha entrenado para capturar la mejor decisión posible, dado un contexto, usa un conjunto de modelos lineales. Estos han mostrado en repetidas ocasiones resultados empresariales y son un enfoque probado, en parte porque pueden aprender del mundo real muy rápidamente sin necesidad de entrenamiento en varias fases, y en parte porque pueden complementar tanto los modelos de aprendizaje supervisados como los modelos de red neuronal profunda.

La asignación de tráfico de exploración y explotación se realiza de forma aleatoria siguiendo el porcentaje establecido para la exploración, y el algoritmo predeterminado para la exploración es epsilon-greedy.

### <a name="history-of-contextual-bandits"></a>Historia de los bandidos contextuales

John Langford acuñó el nombre bandidos contextuales (Langford y Zhang [2007]) para describir un subconjunto manejable de aprendizaje de refuerzo y ha trabajado en media docena de documentos que nos permiten conocer mejor cómo aprender acerca de este paradigma:

* Beygelzimer et al. [2011]
* Dudík et al. [2011a,b]
* Agarwal et al. [2014, 2012]
* Beygelzimer y Langford [2009]
* Li et al. [2010]

John también ha creado anteriormente varios acerca de temas como la predicción conjunta (ICML 2015), la teoría de los bandidos contextuales (NIPS 2013), el aprendizaje activo (ICML 2009) y los límites de complejidad de los ejemplos (ICML 2003)

## <a name="what-machine-learning-frameworks-does-personalizer-use"></a>¿Qué marcos de aprendizaje automático utiliza Personalizer?

Actualmente, Personalizer usa [Vowpal Wabbit](https://github.com/VowpalWabbit/vowpal_wabbit/wiki) como base para el aprendizaje automático. Este marco permite el máximo rendimiento y la mínima latencia al realizar clasificaciones de personalización y entrenar el modelo con todos los eventos.

## <a name="references"></a>Referencias

* [Making Contextual Decisions with Low Technical Debt](https://arxiv.org/abs/1606.03966) (Toma de decisiones contextuales con una deuda técnica baja)
* [A Reductions Approach to Fair Classification](https://arxiv.org/abs/1803.02453) (Un enfoque de reducciones hacia una clasificación justa)
* [Efficient Contextual Bandits in Non-stationary Worlds](https://arxiv.org/abs/1708.01799) (Bandidos contextuales eficaces en mundos que no son fijos)
* [Residual Loss Prediction: Reinforcement: learning With No Incremental Feedback](https://openreview.net/pdf?id=HJNMYceCW) (Predicción de pérdida residual: refuerzo: aprendizaje son comentarios incrementales)
* [Mapping Instructions and Visual Observations to Actions with Reinforcement Learning](https://arxiv.org/abs/1704.08795) (Asignación de instrucciones y observaciones viduales a acciones con aprendizaje reforzado)
* [Learning to Search Better Than Your Teacher](https://arxiv.org/abs/1502.02206) (Aprendiendo a buscar mejor que el profesor)

## <a name="next-steps"></a>Pasos siguientes

[Evaluación sin conexión](concepts-offline-evaluation.md) 