---
title: 'Importancia de la característica de permutación: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo de importancia de la característica de permutación de Azure Machine Learning para calcular las puntuaciones de importancia de la característica de permutación de las variables de características dado un modelo entrenado y un conjunto de datos de prueba.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/24/2020
ms.openlocfilehash: ecf739ea838ad3742612eab7a1b42ac8c8ac455a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905263"
---
# <a name="permutation-feature-importance"></a>Importancia de la característica de permutación

En este artículo se describe cómo usar el módulo Permutation Feature Importance (Importancia de la característica de permutación) en el diseñador de Azure Machine Learning para calcular un conjunto de puntuaciones de importancia de las características del conjunto de datos. Estas puntuaciones se usan para ayudarle a determinar las mejores características que se deben usar en un modelo.

En este módulo, los valores de las características se ordenan aleatoriamente, una columna cada vez. El rendimiento del modelo se mide antes y después. Puede elegir una de las métricas estándar para medir el rendimiento.

Las puntuaciones que devuelve el módulo representan el *cambio* en el rendimiento de un modelo formado, después de la permutación. Las características importantes suelen ser más sensibles al proceso orden aleatorio y, por tanto, se obtendrán mayores puntuaciones de importancia. 

En este artículo se proporciona información general sobre la característica de permutación, su base teórica y sus aplicaciones en aprendizaje automático: [Permutation Feature Importance](https://blogs.technet.com/b/machinelearning/archive/2015/04/14/permutation-feature-importance.aspx) (Importancia de la característica de permutación).  

## <a name="how-to-use-permutation-feature-importance"></a>Cómo usar la Importancia de la característica de permutación

Para generar un conjunto de puntuaciones de características, es necesario tener un modelo ya entrenado, así como un conjunto de datos de prueba.  

1.  Agregue el módulo Permutation Feature Importance (Importancia de la característica de permutación) a su canalización. Puede encontrar este módulo en la categoría **Selección de características**. 

2.  Conecte un modelo formado a la entrada izquierda. El modelo tiene que ser un modelo de regresión o un modelo de clasificación.  

3.  En la entrada derecha, conecte un conjunto de datos. Preferiblemente, elija una que sea diferente del conjunto de datos que usó para entrenar el modelo. Este conjunto de datos se usa para puntuar en base al modelo entrenado. También se usa para evaluar el modelo después de que los valores de características hayan cambiado.  

4.  Para **Valor de inicialización aleatorio**, escriba un valor que se usará como valor de inicialización para la selección aleatoria. Si especifica 0 (el valor predeterminado), se genera un número basado en el reloj del sistema.

     Un valor de inicialización es opcional, pero debe proporcionar un valor si desea la reproducibilidad en ejecuciones de la misma canalización.  

5.  Para **Metric for measuring performance** (Métrica para medir el rendimiento), seleccione una sola métrica para usar al calcular la calidad del modelo después de la permutación.  

     El diseñador de Azure Machine Learning admite las siguientes métricas, en función de si está evaluando un modelo de clasificación o regresión:  

    -   **Clasificación**

        Exactitud, precisión, recuperación  

    -   **Regresión**

        Precisión, recuperación, error absoluto medio, error cuadrático medio raíz, error absoluto relativo, error cuadrado relativo, coeficiente de determinación  

     Para una descripción más detallada de estas métricas de evaluación y cómo se calculan, consulte [Evaluar modelo](evaluate-model.md).  

6.  Envíe la canalización.  

7.  El módulo genera una lista de columnas de características y las puntuaciones asociadas con ellas. La lista está en orden descendente de las puntuaciones.  


##  <a name="technical-notes"></a>Notas técnicas

Permutation Feature Importance (Importancia de la característica de permutación) funciona cambiando aleatoriamente los valores de cada columna de característica, una columna a la vez. A continuación, evalúa el modelo. 

Las clasificaciones que proporciona el módulo suelen ser diferentes de las que obtiene de [Selección de características basada en filtro](filter-based-feature-selection.md). Selección de características basada en filtro calcula las puntuaciones *antes de que* se cree un modelo. 

La razón de la diferencia es que Permutation Feature Importance (Importancia de la característica de permutación) no mide la asociación entre una característica y un valor de destino. En su lugar, captura el grado de influencia que tiene cada característica en las predicciones del modelo.
  
## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
