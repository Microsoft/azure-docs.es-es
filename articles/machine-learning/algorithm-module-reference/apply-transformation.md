---
title: 'Aplicar la transformación: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Aplicar la transformación en Azure Machine Learning para modificar un conjunto de datos de entrada basado en una transformación calculada previamente.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/05/2020
ms.openlocfilehash: 7573abbbee479bfb0d1710beba3b95d084a5e657
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898882"
---
# <a name="apply-transformation-module"></a>Módulo Aplicar transformación

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para modificar un conjunto de datos de entrada basado en una transformación calculada previamente.

Por ejemplo, si usa puntuaciones z para normalizar los datos de aprendizaje utilizando el nódulo **Normalizar datos**, tendrá que usar el valor de puntuación z que se calculó para el aprendizaje durante la fase de puntuación. En Azure Machine Learning, puede guardar el método de normalización como una transformación y después usar **Aplicar transformación** para aplicar la puntuación z a los datos de entrada antes de la puntuación.

## <a name="how-to-save-transformations"></a>Procedimiento para guardar transformaciones

El diseñador le permite guardar transformaciones de datos como **conjuntos de datos** para poder usarlos en otras canalizaciones.

1. Seleccione un módulo de transformación de datos que se haya ejecutado correctamente.

1. Seleccione la pestaña **Outputs + logs** (Resultados y registros).

1. Busque la salida de la transformación y seleccione **Register dataset** (Registrar conjunto de datos) para guardarlo como un módulo en la categoría **Conjuntos de datos** de la paleta de módulos.

## <a name="how-to-use-apply-transformation"></a>Cómo se usa Aplicar transformación  
  
1. Agregue el módulo **Apply Transformation** (Aplicar transformación) a la canalización. Puede encontrar este módulo en la sección **Model Scoring & Evaluation** (Puntuación y evaluación de modelos) de la paleta de módulos. 
  
1. Busque la transformación guardada que quiera usar en **Conjunto de datos** en la paleta de módulos.

1. Conecte la salida de la transformación guardada al puerto de entrada izquierdo del módulo **Apply Transformation** (Aplicar transformación).

    El conjunto de datos debe tener exactamente el mismo esquema (número de columnas, nombres de columna, tipos de datos) que el conjunto de datos para el que la transformación se ha diseñó inicialmente.  
  
1. Conecte la salida del conjunto de datos del módulo deseado al puerto de entrada derecho del módulo **Apply Transformation** (Aplicar transformación).
  
1. Para aplicar una transformación al nuevo conjunto de datos, ejecute la canalización.  

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 