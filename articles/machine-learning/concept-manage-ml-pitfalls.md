---
title: Evitar el sobreajuste y los datos desequilibrados con AutoML
titleSuffix: Azure Machine Learning
description: Identifique y administre los problemas comunes de los modelos de ML con las soluciones de aprendizaje automático automatizado de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 04/09/2020
ms.openlocfilehash: 77b5b52153c552008406b4b85083bcba5542cebe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87012729"
---
# <a name="prevent-overfitting-and-imbalanced-data-with-automated-machine-learning"></a>Evitar el sobreajuste y los datos desequilibrados con el aprendizaje automático automatizado

El sobreajuste y los datos desequilibrados son un problema común al crear modelos de Machine Learning. De forma predeterminada, el aprendizaje automático automatizado de Azure Machine Learning proporciona gráficos y métricas para facilitar la identificación de estos riesgos, e implementa procedimientos recomendados para ayudarle a mitigarlos. 

## <a name="identify-over-fitting"></a>Identificación del sobreajuste

El sobreajuste en el aprendizaje automático se produce cuando un modelo se ajusta demasiado bien a los datos de entrenamiento y, como resultado, no puede predecir con precisión sobre datos de prueba desconocidos. En otras palabras, el modelo simplemente ha memorizado patrones y ruido específicos de los datos de entrenamiento, pero no es lo suficientemente flexible como para realizar predicciones sobre datos reales.

Examine los siguientes modelos entrenados y sus correspondientes precisiones de entrenamiento y prueba.

| Modelo | Precisión del entrenamiento | Precisión de la prueba |
|-------|----------------|---------------|
| A | 99,9 % | 95 % |
| B | 87 % | 87 % |
| C | 99,9 % | 45 % |

Si observa el modelo **A**, hay una idea equivocada habitual de que si la precisión de la prueba en los datos desconocidos es inferior a la del entrenamiento, el modelo está sobreajustado. Pero la precisión de la prueba siempre debe ser menor que la del entrenamiento y la distinción entre el sobreajuste y el ajuste adecuado se convierte en *cuánto* menos preciso. 

Al comparar los modelos **A** y **B**, **A** es un modelo mejor porque tiene mayor precisión en las pruebas y, aunque es ligeramente inferior al 95 %, no es una diferencia significativa que sugiera la existencia de sobreajuste. No elegiría el modelo **B** simplemente porque las precisiones del entrenamiento y la prueba están más próximas.

El modelo **C** representa un caso claro de sobreajuste; la precisión del entrenamiento es muy alta, pero la de la prueba no tanto. Esta distinción es subjetiva, pero proviene del conocimiento del problema y los datos y de las magnitudes de error que son aceptables.

## <a name="prevent-over-fitting"></a>Prevención del sobreajuste

En los casos más notorios, un modelo sobreajustado supondrá que las combinaciones de valores de características que se han visto durante el entrenamiento siempre darán exactamente la misma salida para el destino.

La mejor manera de evitar el sobreajuste es seguir los procedimientos recomendados de ML, entre los que se incluyen los siguientes:

* Usar más datos de entrenamiento y eliminar el sesgo estadístico
* Evitar pérdidas de destino
* Usar menos características
* **Regularización y optimización de hiperparámetros**
* **Limitaciones de la complejidad del modelo**
* **Validación cruzada**

En el contexto del aprendizaje automático automatizado, los tres primeros elementos anteriores son **procedimientos recomendados que usted implementa**. Los tres últimos elementos en negrita son **procedimientos recomendados que el aprendizaje automático automatizado implementa** de forma predeterminada para protegerse frente al sobreajuste. En configuraciones distintas al aprendizaje automático automatizado, es conveniente seguir los seis procedimientos recomendados para evitar modelos con sobreajuste.

### <a name="best-practices-you-implement"></a>Procedimientos recomendados que usted implementa

El uso de **más datos** es la forma más sencilla y óptima de evitar el sobreajuste y, como ventaja adicional, suele aumentar la precisión. Cuando se usan más datos, resulta más difícil para el modelo memorizar patrones exactos y se ve obligado a obtener soluciones que son más flexibles para acomodar más condiciones. También es importante reconocer el **sesgo estadístico**, para asegurarse de que los datos de entrenamiento no incluyen patrones aislados que no existen en los datos de predicción en directo. Este escenario puede ser difícil de resolver, ya que es posible que no haya sobreajuste entre los conjuntos de entrenamiento y pruebas, pero puede haberlo cuando se comparan con los datos de pruebas en directo.

La **pérdida de destino** es un problema similar, en el que es posible que no vea sobreajuste entre los conjuntos de entrenamiento y pruebas, sino que aparece en tiempo de predicción. La pérdida de destino se produce cuando el modelo "hace trampa" durante el entrenamiento al tener acceso a datos que normalmente no debería tener en el momento de la predicción. Por ejemplo, si el problema es predecir el lunes cuál será el precio el viernes, pero una de las características incluye accidentalmente datos del jueves, serían datos que el modelo no tendrá en tiempo de predicción, ya que no puede ver el futuro. La pérdida de destino es un error sencillo de evitar, pero a menudo se caracteriza por una precisión anormalmente alta para el problema. Si intenta predecir el precio de las acciones y ha entrenado un modelo con una precisión del 95 %, es probable que se produzcan pérdidas de destino en alguna parte de las características.

La **eliminación de características** también puede ayudar con el sobreajuste, ya que impide que el modelo tenga demasiados campos para memorizar patrones específicos, lo que hace que sea más flexible. Puede ser difícil de medir cuantitativamente, pero si puede eliminar características y conservar la misma precisión, probablemente haya hecho que el modelo sea más flexible y haya reducido el riesgo de sobreajuste.

### <a name="best-practices-automated-ml-implements"></a>Procedimientos recomendados que implementa el aprendizaje automático automatizado

La **regularización** es el proceso de minimizar una función de costo para penalizar modelos complejos y sobreajustados. Hay diferentes tipos de funciones de regularización, pero en general todas penalizan el tamaño del coeficiente del modelo, la varianza y la complejidad. El aprendizaje automático automatizado usa L1 (Lasso), L2 (Ridge) y ElasticNet (L1 y L2 simultáneamente) en combinaciones diferentes con otras configuraciones de hiperparámetros del modelo que controlan el sobreajuste. En términos simples, el aprendizaje automático automatizado variará cuánto se regula un modelo y elegirá el mejor resultado.

El aprendizaje automático automatizado también implementa **limitaciones de complejidad del modelo** explícitas para evitar el sobreajuste. En la mayoría de los casos, esta implementación se aplica específicamente a árboles de decisión o algoritmos de bosque, donde la profundidad máxima del árbol individual está limitada y el número total de árboles que se usan en el bosque o las técnicas conjunto están limitados.

La **validación cruzada (CV)** es el proceso de tomar muchos subconjuntos de los datos de entrenamiento completos y entrenar un modelo en cada subconjunto. La idea es que un modelo podría tener "suerte" y conseguir una gran precisión con un subconjunto, pero al usar muchos subconjuntos el modelo no la alcanzará cada vez. Al realizar la CV, se proporciona un conjunto de datos de exclusión de la validación, se especifican los plegamientos de la CV (número de subconjuntos) y el aprendizaje automático automatizado entrena el modelo y ajusta los hiperparámetros para minimizar el error en el conjunto de validación. Un plegamiento de la CV podría estar sobreajustado, pero si se usan muchos de ellos, se reduce la probabilidad de que el modelo final esté sobreajustado. La contrapartida es que la CV genera tiempos de entrenamiento más largos y, por tanto, un costo mayor, porque en lugar de entrenar un modelo una vez, se entrena una vez por cada *n* subconjuntos de CV. 

> [!NOTE]
> La validación cruzada no está habilitada de forma predeterminada; se debe configurar en la configuración de aprendizaje automático automatizado. Pero después de configurar la validación cruzada y proporcionar un conjunto de datos de validación, el proceso es automático. Más información sobre la [configuración de validación cruzada en ML automatizado](how-to-configure-cross-validation-data-splits.md)

<a name="imbalance"></a>

## <a name="identify-models-with-imbalanced-data"></a>Identificación de modelos con datos no equilibrados

Los datos no equilibrados suelen encontrarse en los datos de escenarios de clasificación de aprendizaje automático y hacen referencia a los datos que contienen una relación desproporcionada de observaciones en cada clase. Este desequilibrio puede dar lugar a un efecto positivo percibido de manera falsa de la precisión de un modelo, ya que los datos de entrada tienen un sesgo hacia una clase, lo que da lugar a que el modelo entrenado imite ese sesgo. 

Además, las ejecuciones de ML automatizado generan automáticamente los gráficos siguientes, que pueden ayudar a entender la corrección de las clasificaciones del modelo e identificar los modelos potencialmente afectados por los datos no equilibrados.

Gráfico| Descripción
---|---
[Matriz de confusión](how-to-understand-automated-ml.md#confusion-matrix)| Evalúa las etiquetas clasificadas correctamente con respecto a las etiquetas reales de los datos. 
[Precisión-coincidencia](how-to-understand-automated-ml.md#precision-recall-chart)| Evalúa la proporción de etiquetas correctas con respecto a la de las instancias de etiquetas encontradas de los datos. 
[Curvas ROC](how-to-understand-automated-ml.md#roc)| Evalúa la proporción de etiquetas correctas con respecto a la de etiquetas de falsos positivos.

## <a name="handle-imbalanced-data"></a>Manejo de datos no equilibrados 

Como parte de su objetivo de simplificar el flujo de trabajo de aprendizaje automático, el **ML automatizado tiene funcionalidades integradas** que ayudan a tratar con datos no equilibrados como, por ejemplo: 

- Una **columna de peso**: el ML automatizado admite una columna de pesos como entrada, lo que provoca que las filas de los datos se puedan subir o bajar, que se puede usar para que una clase sea más o menos "importante".

- Los algoritmos que usa el ML automatizado detectan el desequilibrio cuando el número de muestras de la clase de minoría es igual o menor que el 20 % del número de muestras en la clase de mayoría, donde la clase de minoría se refiere a la que tiene menos muestras y la clase de mayoría, a la que tiene más muestras. Posteriormente, AutoML ejecutará un experimento con datos submuestreados para comprobar si el uso de pesos de clase solucionará este problema y mejorará el rendimiento. Si se garantiza un mejor rendimiento a través de este experimento, se aplica esta solución.

- Use una métrica de rendimiento que se ocupe mejor de los datos no equilibrados. Por ejemplo, AUC_weighted es una métrica primaria que calcula la contribución de cada clase en función del número relativo de muestras que representan esa clase, por lo que es más robusta frente al desequilibrio.

Las técnicas siguientes son otras opciones para controlar los datos no equilibrados **fuera del ML automatizado**. 

- Vuelva a muestrear para equilibrar el desequilibrio de clases, ya sea mediante el muestreo ascendente de las clases más pequeñas o el muestreo descendente de las más grandes. Estos métodos requieren conocimientos de proceso y análisis.

- Revise las métricas de rendimiento de los datos desequilibrados. Por ejemplo, la puntuación F1 es la media armónica de precisión y coincidencia. La precisión mide la exactitud de un clasificador, donde una precisión más alta indica menos falsos positivos, mientras que la coincidencia mide la integridad de un clasificador, donde una coincidencia más alta indica menos falsos negativos.

## <a name="next-steps"></a>Pasos siguientes

Vea ejemplos y aprenda cómo generar modelos mediante aprendizaje automático automatizado:

+ Lea el [Tutorial: Entrenamiento automático de un modelo de regresión con Azure Machine Learning](tutorial-auto-train-models.md).

+ Configure el experimento de entrenamiento automático:
  + En Azure Machine Learning Studio, [siga estos pasos](how-to-use-automated-ml-for-ml-models.md).
  + Con el SDK de Python, [siga estos pasos](how-to-configure-auto-train.md).


