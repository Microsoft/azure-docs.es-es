---
title: Uso del método de evaluación sin conexión - Personalizer
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo usar la evaluación sin conexión para medir la eficacia de la aplicación y analizar el bucle de aprendizaje.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: 627f511bb12c16c8f54935d1f782cb7c2c962163
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132762"
---
# <a name="offline-evaluation"></a>Evaluación sin conexión

La evaluación sin conexión es un método que permite probar y evaluar la eficacia del servicio Personalizer sin cambiar el código ni afectar a la experiencia del usuario. La evaluación sin conexión usa datos anteriores, que se envían desde la aplicación a las API Rank y Reward, para comparar el funcionamiento de las diferentes clasificaciones.

La evaluación sin conexión se realiza en un rango de fechas. El rango puede finalizar, como muy tarde, en la hora actual. El principio del rango no puede superar el número de días especificado para la [retención de datos](how-to-settings.md).

La evaluación sin conexión puede ayudarle a responder las siguientes preguntas:

* ¿Son muy eficaces las clasificaciones de Personalizer para una personalización correcta?
    * ¿Cuál es el promedio de recompensas que se logra mediante la directiva de aprendizaje de automático en línea de Personalizer?
    * ¿Cómo se compara Personalizer a la eficacia de lo que la aplicación habría hecho de forma predeterminada?
    * ¿Cuál habría sido la eficacia comparativa de una selección aleatoria para Personalization?
    * ¿Cuál habría sido la eficacia comparativa de políticas de distintas directivas de aprendizaje especificadas manualmente?
* ¿Qué características del contexto son las que más o las que menos contribuyen a una personalización correcta?
* ¿Qué características de las acciones son las que más o las que menos contribuyen a una personalización correcta?

Además, la evaluación sin conexión se puede utilizar para detectar directivas de aprendizaje más optimizadas que Personalizer puede usar para mejorar los resultados en el futuro.

Las evaluaciones sin conexión no proporcionan orientaciones en cuanto al porcentaje de eventos que se usan para la exploración.

## <a name="prerequisites-for-offline-evaluation"></a>Requisitos previos para la evaluación sin conexión

Los siguientes son consideraciones importantes para una evaluación sin conexión representativa:

* Tener suficientes datos. El mínimo recomendado es de, al menos, 50 000 eventos.
* Recopilar datos de períodos con un comportamiento de los usuarios y un tráfico representativos.

## <a name="discovering-the-optimized-learning-policy"></a>Detección de la directiva de aprendizaje optimizada

Personalizer puede usar el proceso de evaluación sin conexión para detectar automáticamente una directiva de aprendizaje óptima.

Después de realizar la evaluación sin conexión, puede ver la efectividad comparativa de Personalizer con la nueva directiva, en comparación con la directiva en línea actual. Después puede aplicar esa directiva de aprendizaje para que sea efectiva de inmediato en Personalizer, al descargarla y cargarla en el panel de modelos y directiva. También puede descargarla para su futuro análisis o uso.

Directivas actuales incluidas en la evaluación:

| Configuración del aprendizaje | Propósito|
|--|--|
|**Directiva en línea**| la directiva de aprendizaje actual utilizada en Personalizer. |
|**Línea de base**|El valor predeterminado de la aplicación (según lo determinado por la primera acción que se envía en las llamadas de Rank).|
|**Directiva aleatoria**|un comportamiento de Rank imaginario que siempre devuelve la selección aleatoria de las acciones de las proporcionadas.|
|**Directivas personalizadas**|directivas de aprendizaje adicionales que se cargan al iniciar la evaluación.|
|**Directiva optimizada**|Si la evaluación se inició con la opción de detectar una directiva optimizada, también se comparará, y podrá descargarla o convertirla en la directiva de aprendizaje en línea, por el reemplazo de la actual.|

## <a name="understanding-the-relevance-of-offline-evaluation-results"></a>Información de la relevancia de los resultados de la evaluación sin conexión

Al ejecutar una evaluación sin conexión, es muy importante analizar los _límites de confianza_ de los resultados. Si son grandes, significa que la aplicación no ha recibido datos suficientes para que los cálculos de la recompensa sean precisos o importantes. A medida que el sistema acumula más datos y se ejecutan evaluaciones sin conexión durante períodos más largos, los intervalos de confianza son estrechan más.

## <a name="how-offline-evaluations-are-done"></a>Cómo se realizan las evaluaciones sin conexión

Las evaluaciones sin conexión se realizan mediante un método denominado **evaluación contrafactual**.

Personalizer se basa en la suposición de que el comportamiento (y por consiguiente las recompensas) de los usuarios es imposible de predecir retrospectivamente (Personalizer no puede saber lo que habría ocurrido si el usuario hubiera mostrado algo diferente de lo que vio) y solo para aprender de las recompensas medidas.

Este es el proceso conceptual que se usa para las evaluaciones:

```
[For a given _learning policy), such as the online learning policy, uploaded learning policies, or optimized candidate policies]:
{
    Initialize a virtual instance of Personalizer with that policy and a blank model;

    [For every chronological event in the logs]
    {
        - Perform a Rank call

        - Compare the reward of the results against the logged user behavior.
            - If they match, train the model on the observed reward in the logs.
            - If they don't match, then what the user would have done is unknown, so the event is discarded and not used for training or measurement.

    }

    Add up the rewards and statistics that were predicted, do some aggregation to aid visualizations, and save the results.
}
```

La evaluación sin conexión solo usa el comportamiento observado del usuario. Este proceso descarta grandes volúmenes de datos, sobre todo si la aplicación realiza llamadas a Rank con un gran número de acciones.


## <a name="evaluation-of-features"></a>Evaluación de las características

Las evaluaciones sin conexión pueden proporcionar información acerca de la cantidad de determinadas características para acciones o contexto que se ponderan para recompensas mayores. La información se calcula mediante la evaluación en un período de tiempo determinado y con respecto a unos datos, y puede variar con el paso del tiempo.

Se recomienda mirar las evaluaciones de las características y preguntar:

* ¿Qué otras características adicionales puede proporcionar la aplicación o sistema que sean similares a las que sean más eficaces?
* ¿Qué características se pueden quitar debido a su baja eficacia? Las características de poca eficacia agregan _ruido_ al aprendizaje automático.
* ¿Hay características que se incluyen de forma accidental? Estos son algunos ejemplos de ellas: información de identificación del usuario, identificadores duplicados, etc.
* ¿Hay características no deseadas que no se deben usar para personalizar debido a consideraciones legales o de uso responsable? ¿Hay características que se puedan redirigir mediante proxy (es decir, asemejarse mucho o correlacionarse) con características no deseadas?


## <a name="next-steps"></a>Pasos siguientes

[Configuración de Personalizer](how-to-settings.md)
[Ejecución de evaluaciones sin conexión](how-to-offline-evaluation.md)[Funcionamiento de Personalizer](how-personalizer-works.md).
