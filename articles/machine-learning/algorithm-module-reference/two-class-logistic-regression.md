---
title: 'Regresión logística de dos clases: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Regresión logística de dos clases en Azure Machine Learning para crear un modelo de regresión logística que pueda usarse para predecir dos resultados (y solo dos).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 525293c7ae3d0af49f6deaa0ce9d3cb037d9ba38
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907694"
---
# <a name="two-class-logistic-regression-module"></a>Módulo Regresión logística de dos clases

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para crear un modelo de regresión logística que pueda usarse para predecir dos resultados (y solo dos). 

La regresión logística es una técnica estadística conocida que se usa para el modelado de muchos tipos de problemas. Este algoritmo es un método de *aprendizaje supervisado*; por lo tanto, debe proporcionar un conjunto de datos que ya contengan los resultados para entrenar el modelo.  

### <a name="about-logistic-regression"></a>Acerca de la regresión logística  

La regresión logística es un método conocido en estadística que se usa para predecir la probabilidad de un resultado y es especialmente popular para las tareas de clasificación. El algoritmo predice la probabilidad de aparición de un evento ajustando los datos a una función logística.
  
En este módulo, el algoritmo de clasificación se optimiza para las variables dicotómicas o binarias. Si necesita clasificar varios resultados, use el módulo [Multiclass Logistic Regression](./multiclass-logistic-regression.md) (Regresión logística multiclase).

##  <a name="how-to-configure"></a>Cómo se configura  

Para entrenar este modelo, debe proporcionar un conjunto de datos que contenga una columna de etiqueta o clase. Dado que este módulo está pensado para problemas de dos clases, la columna de etiqueta o clase debe contener exactamente dos valores. 

Por ejemplo, la columna de etiqueta podría ser [Votado] con los valores posibles "Sí" o "No". O bien, podría ser [Riesgo de crédito,] con los valores posibles "Alto" o "Bajo". 
  
1.  Agregue el módulo **Two-Class Logistic Regression** (Regresión logística de dos clases) a la canalización.  
  
2.  Especifique cómo quiere que se entrene el modelo, estableciendo la opción **Create trainer mode** (Crear modo entrenador).  
  
    -   **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, puede proporcionar un conjunto específico de valores como argumentos.  

    -   **Parameter Range** (Intervalo de parámetros): si no está seguro de los mejores parámetros, puede encontrar los óptimos mediante el módulo [Optimización de hiperparámetros de un modelo](tune-model-hyperparameters.md). Proporcione un rango de valores y el entrenador itera varias combinaciones de ellos para determinar la combinación de valores que genera el mejor resultado.
  
3.  Para **Optimization tolerance** (Tolerancia de optimización), especifique un valor de umbral que se usará al optimizar el modelo. Si la mejora entre iteraciones cae por debajo del umbral especificado, se considera que el algoritmo ha convergido en una solución y el entrenamiento se detiene.  
  
4.  Para **L1 regularization weight** (Peso de regularización L1) y **L2 regularization weight** (Peso de regularización L2), escriba un valor que se usará para los parámetros de regularización L1 y L2. Se recomienda un valor distinto de cero para ambos.  
     La *regularización* es un método para evitar el sobreajuste mediante la penalización de modelos con valores de coeficiente extremos. La regularización funciona agregando la penalización asociada a los valores de coeficiente al error de la hipótesis. Por lo tanto, un modelo preciso con valores de coeficiente extremos se penalizaría más y un modelo menos preciso con valores más conservadores se penalizaría menos.  
  
     Las regularizaciones L1 y L2 tienen efectos y usos diferentes.  
  
    -   L1 puede aplicarse a modelos dispersos, lo que resulta útil cuando se trabaja con datos muy dimensionales.  
  
    -   En cambio, la regularización L2 es preferible para los datos que no están dispersos.  
  
     Este algoritmo es compatible con una combinación lineal de los valores de regularización L1 y L2: es decir, si <code>x = L1</code> y <code>y = L2</code>, <code>ax + by = c</code> define el intervalo lineal de los términos de regularización.  
  
    > [!NOTE]
    >  ¿Desea obtener más información sobre las regularizaciones L1 y L2? En el siguiente artículo se proporciona una explicación de las diferencias entre las regularizaciones L1 y L2 y cómo afectan al ajuste del modelo, con códigos de ejemplo para los modelos de red neuronal y regresión logística:  [Regularización L1 y L2 para Machine Learning](https://msdn.microsoft.com/magazine/dn904675.aspx)  
    >
    > Se han diseñado diferentes combinaciones lineales de los términos de L1 y L2 para los modelos de regresión logística: por ejemplo, [regularización elástica neta](https://wikipedia.org/wiki/Elastic_net_regularization). Se recomienda hacer referencia a estas combinaciones para definir una combinación lineal que sea efectiva en el modelo.
      
5.  Para **Memory size for L-BFGS** (Tamaño de memoria para L-BFGS), especifique la cantidad de memoria que se usará para la optimización de *L-BFGS*.  
  
     L-BFGS significa "limited memory Broyden-Fletcher-Goldfarb-Shanno". Es un algoritmo de optimización que es popular para la estimación de parámetros. Este parámetro indica el número de gradientes y posiciones anteriores que se deben almacenar para el cálculo del siguiente paso.  
  
     Este parámetro de optimización limita la cantidad de memoria que se usa para calcular el siguiente paso y la dirección. Cuando especifica menos memoria, el entrenamiento es más rápido, pero menos preciso.  
  
6.  Para **Random number seed** (Inicialización de número aleatorio), escriba un valor entero. Definir un valor de inicialización es importante si desea que los resultados se puedan reproducir en varias ejecuciones de la misma canalización.  
  
  
8. Agregue un conjunto de datos etiquetado a la canalización y entrene el modelo:

    + Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), conecte un conjunto de datos etiquetado y el módulo [Entrenar modelo](train-model.md).  
  
    + Si define **Create trainer mode** (Crear modo de entrenador) como **Parameter Range** (Rango de parámetros), conecte un conjunto de datos etiquetado y entrene el modelo mediante [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Hiperparámetros de modelo de ajuste).  
  
    > [!NOTE]
    > 
    > Si pasa un rango de parámetros a [Entrenar modelo](train-model.md), solo se usa el valor predeterminado en la lista de parámetros única.  
    > 
    > Si pasa un único conjunto de valores de parámetro al módulo de [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Hiperparámetros de modelo de ajuste), cuando se espera un rango de valores para cada parámetro, ignora los valores y usa los valores predeterminados para el aprendiz.  
    > 
    > Si selecciona la opción **Parameter Range** (Rango de parámetros) y especifica un valor único para cualquier parámetro, ese valor único que haya especificado se usará en todo el barrido, incluso si otros parámetros cambian en un rango de valores.  
  
9. Envíe la canalización.  
  
## <a name="results"></a>Results

Una vez completado el entrenamiento:
 
  
+ Para realizar predicciones sobre nuevos datos, use el modelo entrenado y los nuevos datos como entrada para el módulo [Puntuar modelo](./score-model.md). 


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 