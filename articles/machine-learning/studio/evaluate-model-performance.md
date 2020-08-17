---
title: 'ML Studio (clásico): Evaluación y validación cruzada de modelos: Azure'
description: Obtenga información sobre las métricas que puede usar para supervisar el rendimiento de un modelo en Azure Machine Learning Studio (clásico).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18, previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 03/20/2017
ms.openlocfilehash: e811d921d28ce33d7dbc9f2bb7996557838178ad
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87429862"
---
# <a name="evaluate-model-performance-in-azure-machine-learning-studio-classic"></a>Evaluación del rendimiento de un modelo en Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


En este artículo, puede obtener información sobre las métricas que puede usar para supervisar el rendimiento de un modelo en Azure Machine Learning Studio (clásico).  La evaluación del rendimiento de un modelo es una de las fases principales en el proceso de ciencia de datos. Indica el nivel de acierto de las puntuaciones (predicciones) de un conjunto de datos mediante un modelo entrenado. Azure Machine Learning Studio (clásico) admite la evaluación de modelos a través de dos de sus módulos principales de aprendizaje automático: 
+ [Evaluación de módulo][evaluate-model] 
+ [Modelo de validación cruzada][cross-validate-model]

Estos módulos permiten ver el rendimiento del modelo como un número de métricas que se usan habitualmente en estadísticas y aprendizaje automático.

Los modelos de evaluación deben tenerse en cuenta junto con:
+ [Optimización de parámetros para algoritmos](algorithm-parameters-optimize.md)
+ [Interoperabilidad del modelo](interpret-model-results.md)

Se presentan tres escenarios comunes de aprendizaje supervisado: 
* regresión
* clasificación binaria 
* clasificación multiclase


## <a name="evaluation-vs-cross-validation"></a>Evaluación frente a Validación cruzada
La evaluación y la validación cruzada son métodos estándares para medir el rendimiento de un modelo. Ambos generan métricas de evaluación que puede inspeccionar o comparar con las de otros modelos.

El módulo [Evaluar modelo][evaluate-model] espera un conjunto de datos puntuado como entrada (o dos en caso de que quiera comparar el rendimiento de dos modelos distintos). Por lo tanto, debe entrenar el modelo mediante el módulo [Entrenar modelo][train-model] y realizar predicciones sobre algún conjunto de datos con el módulo [Puntuar modelo][score-model], antes de poder evaluar los resultados. La evaluación se basa en las etiquetas y probabilidades puntuadas junto con las etiquetas verdaderas, las cuales son el resultado del módulo [Puntuar modelo][score-model].

De forma alternativa, es posible usar la validación cruzada para realizar automáticamente varias operaciones de entrenamiento, puntuación y evaluación (10 subconjuntos) en distintos subconjuntos de los datos de entrada. Los datos de entrada se dividen en 10 partes, donde una se reserva para las pruebas y las otras 9 para el entrenamiento. Este proceso se repite 10 veces y se calcula el promedio de las métricas de evaluación. Esto ayuda a determinar el nivel al que un modelo se podría generalizar para nuevos conjuntos de datos. El módulo [Validar modelo de forma cruzada][cross-validate-model] toma un modelo sin entrenar y algunos conjuntos de datos con etiquetas y genera los resultados de la evaluación de cada uno de los 10 subconjuntos, además de los resultados promediados.

En las siguientes secciones, se crearán modelos de clasificación y regresión simples, y se evaluará su rendimiento con los módulos [Evaluar modelo][evaluate-model] y [Validar modelo de forma cruzada][cross-validate-model].

## <a name="evaluating-a-regression-model"></a>Evaluación de un modelo de regresión
Supongamos que quiere predecir el precio de un automóvil mediante características, como sus dimensiones, caballos de potencia, especificaciones del motor, etc. Se trata de un problema de regresión típico, donde la variable objetivo (*price*) es un valor numérico continuo. Podemos generar un modelo de regresión lineal que, dados los valores de las características de un automóvil determinado, pueda predecir el precio de ese automóvil. Este modelo de regresión se puede usar para puntuar el mismo conjunto de datos con que se entrenó. Cuando se tienen los precios predichos del automóvil, se puede evaluar el rendimiento con una comparación de cuánto se desvían en promedio las predicciones de los precios reales. Para ilustrar esto, se usa el conjunto de datos *Información sobre los precios de los automóviles (datos sin procesar)* disponible en la sección **Conjuntos de datos almacenados** en Machine Learning Studio (clásico).

### <a name="creating-the-experiment"></a>Creación del experimento
Agregue los módulos siguientes al área de trabajo en Azure Machine Learning Studio (clásico):

* Información sobre los precios de los automóviles (datos sin procesar)
* [Regresión lineal][linear-regression]
* [Train Model][train-model] (entrenar modelo)
* [Score Model][score-model] (puntuar modelo)
* [Evaluación de módulo][evaluate-model]

Conecte los puertos, tal y como se muestra en la Ilustración 1 y establezca la columna de etiqueta del módulo [Entrenar modelo][train-model] en *price*.

![Evaluación de un modelo de regresión](./media/evaluate-model-performance/1.png)

Figura 1. Evaluación de un modelo de regresión.

### <a name="inspecting-the-evaluation-results"></a>Inspección de los resultados de la evaluación
Después de ejecutar el experimento, puede hacer clic en el puerto de salida del módulo [Evaluar modelo][evaluate-model] y seleccionar *Visualizar* para ver los resultados de la evaluación. Las métricas de evaluación disponibles para los modelos de regresión son: *Mean Absolute Error*, *Root Mean Absolute Error*, *Relative Absolute Error*, *Relative Squared Error* y *Coefficient of Determination*.

El término "error" representa aquí la diferencia entre el valor predicho y el valor verdadero. Normalmente, se calcula el valor absoluto o el cuadrado de esta diferencia para capturar la magnitud total de errores en todas las instancias, dado que la diferencia entre el valor real y el predicho puede ser negativa en algunos casos. Las métricas de error miden el rendimiento de predicción de un modelo de regresión en cuanto a la desviación media de sus predicciones a partir de los valores reales. Los valores de error más bajos implican que el modelo es más preciso a la hora de realizar predicciones. Una métrica de error general de cero significa que el modelo se ajusta a los datos perfectamente.

El coeficiente de determinación, que también se conoce como R cuadrado, es también una manera estándar de medir cuánto se adapta el modelo a los datos. Se puede interpretar como la proporción de la variación que explica el modelo. Una mayor proporción es mejor en este caso, donde 1 indica un ajuste perfecto.

![Métricas de evaluación de regresión lineal](./media/evaluate-model-performance/2.png)

Ilustración 2. Métricas de evaluación de regresión lineal.

### <a name="using-cross-validation"></a>Uso de la validación cruzada
Tal como se mencionó anteriormente, puede realizar procesos de entrenamiento, puntuación y evaluación de forma repetida y automática mediante el módulo [Validar modelo de forma cruzada][cross-validate-model]. Lo único que necesita en este caso es un conjunto de datos, un modelo sin entrenar y un módulo [Validar modelo de forma cruzada][cross-validate-model] (consulte la ilustración siguiente). Debe establecer la columna de etiqueta en *price* en las propiedades del módulo [Cross-Validate Model][cross-validate-model] (Modelo de validación cruzada).

![Validación cruzada de un modelo de regresión](./media/evaluate-model-performance/3.png)

Figura 3. Validación cruzada de un modelo de regresión.

Después de ejecutar el experimento, puede inspeccionar los resultados de la evaluación haciendo clic en el puerto de salida derecho del módulo [Validar modelo de forma cruzada][cross-validate-model]. Esto proporcionará una vista detallada de las métricas de cada iteración (subconjunto) y los resultados promediados de cada una de las métricas (Figura 4).

![Resultados de la validación cruzada de un modelo de regresión](./media/evaluate-model-performance/4.png)

Figura 4. Resultados de la validación cruzada de un modelo de regresión.

## <a name="evaluating-a-binary-classification-model"></a>Evaluación de un modelo de clasificación binaria
En un escenario de clasificación binaria, la variable objetivo tiene solo dos resultados posibles, por ejemplo: {0, 1} o {false, true}, {negative, positive}. Suponga que tiene un conjunto de datos de empleados adultos con algunas variables demográficas y de empleo, y se le pide que prediga el nivel de ingresos, una variable binaria con los valores {"<=50 K", ">50 K"}. En otras palabras, la clase negativa representa a los empleados que tienen un sueldo menor o igual a 50 000 al año y la clase positiva representa a los demás empleados. Al igual que en el escenario de regresión, se entrenaría un modelo, se puntuarían algunos datos y se evaluarían los resultados. La principal diferencia es la elección de las métricas que Azure Machine Learning Studio (clásico) calcula y da como resultado. Para ilustrar el escenario de predicción del nivel de ingresos, se usará el conjunto de datos [Adult](https://archive.ics.uci.edu/ml/datasets/Adult) para crear un experimento de Studio (clásico) y evaluar el rendimiento de un modelo de regresión logística de dos clases, un clasificador binario que se usa con frecuencia.

### <a name="creating-the-experiment"></a>Creación del experimento
Agregue los módulos siguientes al área de trabajo en Azure Machine Learning Studio (clásico):

* Conjunto de datos de clasificación binaria de ingresos en el censo de adultos
* [Regresión logística de dos clases][two-class-logistic-regression]
* [Train Model][train-model] (entrenar modelo)
* [Score Model][score-model] (puntuar modelo)
* [Evaluación de módulo][evaluate-model]

Conecte los puertos tal y como se muestra en la Ilustración 5 y establezca la columna de etiqueta del módulo [Entrenar modelo][train-model] en *income*.

![Evaluación de un modelo de clasificación binaria](./media/evaluate-model-performance/5.png)

Figura 5. Evaluación de un modelo de clasificación binaria.

### <a name="inspecting-the-evaluation-results"></a>Inspección de los resultados de la evaluación
Después de ejecutar el experimento, puede hacer clic en el puerto de salida del módulo [Evaluar modelo][evaluate-model] y seleccionar *Visualizar* para ver los resultados de la evaluación (ilustración 7). Las métricas de evaluación disponibles para los modelos de clasificación binaria son: *Accuracy*, *Precision*, *Recall*, *F1 Score* y *AUC*. Además, el módulo genera una matriz de confusión que muestra el número de positivos verdaderos, falsos negativos, falsos positivos y negativos verdaderos, así como curvas *ROC*, *Precision/Recall* y *Lift*.

La precisión es simplemente la proporción de instancias clasificadas correctamente. Suele ser la primera métrica que se comprueba al evaluar un clasificador. Sin embargo, si los datos de prueba están descompensados (en el caso en que la mayoría de las instancias pertenezcan a una de las clases) o está más interesado en el rendimiento de una de las clases, la precisión no captura realmente la eficacia de un clasificador. En el escenario de clasificación del nivel de ingresos, suponga que está realizando pruebas en datos donde el 99 % de las instancias representan personas con un sueldo menor o igual a 50.000 al año. Es posible conseguir una precisión de 0,99 al predecir la clase "<=50 K" para todas las instancias. En este caso, el clasificador parece hacer un buen trabajo global, pero en realidad no clasifica correctamente ninguno de las personas con ingresos elevados (1 %) correctamente.

Por ese motivo, es útil calcular métricas adicionales que capturen aspectos más específicos de la evaluación. Antes de entrar a los detalles de dichas métricas, es importante comprender la matriz de confusión de una evaluación de clasificación binaria. Las etiquetas de clase en el conjunto de entrenamiento pueden tomar solo dos valores posibles, a los que normalmente podemos referirnos como positivo o negativo. Las instancias positivas y negativas que un clasificador predice correctamente se denominan positivos verdaderos (TP) y negativos verdaderos (TN), respectivamente. De forma similar, las instancias clasificadas incorrectamente se denominan falsos positivos (FP) y falsos negativos (FN). La matriz de confusión es simplemente una tabla que muestra el número de instancias que se encuentran bajo cada una de estas cuatro categorías. Azure Machine Learning Studio (clásico) decide automáticamente cuál de las dos clases en el conjunto de datos es la clase positiva. Si las etiquetas de clase son valores booleanos o enteros, se asignan las instancias etiquetadas como "true" o "1" a la clase positiva. Si las etiquetas son cadenas, como en el conjunto de datos de ingresos, las etiquetas se ordenan alfabéticamente y se elige que el primer nivel sea la clase negativa, mientras que el segundo nivel es la clase positiva.

![Matriz de confusión de la clasificación binaria](./media/evaluate-model-performance/6a.png)

Figura 6. Matriz de confusión de la clasificación binaria.

Volviendo al problema de clasificación de ingresos, existen varias preguntas de evaluación que querríamos preguntar para ayudarnos a comprender el rendimiento del clasificador utilizado. Una pregunta natural es: "De las personas que el modelo predijo que ganan >50 000 (TP+FP), ¿cuántos se han clasificado correctamente (TP)?". Puede responder esta pregunta observando la **precisión** del modelo, que es la proporción de positivos que se han clasificado correctamente: TP/(TP+FP). Otra pregunta común es "De todos los empleados con ingresos >50 000 (TP+FN), ¿cuántos predijo el clasificador correctamente (TP)?". Esto es en realidad la **recuperación** o la tasa de positivos verdaderos: TP/(TP+FN) del clasificador. Observará que hay una evidente compensación entre la precisión y la recuperación. Por ejemplo, dado un conjunto de datos relativamente equilibrado, un clasificador que prediga principalmente instancias positivas tendría una recuperación alta, pero una precisión más baja, ya que muchas de las instancias negativas se clasificarían incorrectamente y se produciría un número mayor de falsos positivos. Para ver un gráfico de cómo varían estas dos métricas, haga clic en la curva de **PRECISIÓN/RECUPERACIÓN** en la página de salida de resultados de evaluación (parte superior izquierda de la Figura 7).

![Resultados de la evaluación de clasificación binaria](./media/evaluate-model-performance/7.png)

Ilustración 7. Resultados de la evaluación de clasificación binaria.

Otra métrica relacionada que se usa con frecuencia es **F1 Score**, que tiene en cuenta la precisión y la recuperación. Es la media armónica de estas dos métricas y se calcula como tal: F1 = 2 (precisión x recuperación) / (precisión + recuperación). La puntuación de F1 es una buena forma de resumir la evaluación en un número único, pero siempre es recomendable comprobar la precisión y la recuperación juntas para comprender mejor cómo se comporta un clasificador.

Además, es posible inspeccionar la tasa de positivos verdaderos frente a la de falsos positivos en la curva **Característica operativa del receptor (ROC)** y el valor del **área bajo la curva (AUC)** correspondiente. Cuanto más se acerque esta curva a la esquina superior izquierda, mejor será el rendimiento del clasificador (es decir, se maximiza la tasa de positivos verdaderos a la vez que se minimiza la de falsos positivos). Las curvas que están cerca de la diagonal del gráfico son el resultado de los clasificadores que tienden a realizar predicciones que se acercan a una estimación aleatoria.

### <a name="using-cross-validation"></a>Uso de la validación cruzada
Como en el ejemplo de regresión, podemos realizar una validación cruzada para entrenar, puntuar y evaluar de forma repetida y automática diferentes subconjuntos de datos. De manera similar, es posible usar el módulo [Validar modelo de forma cruzada][cross-validate-model], un modelo de regresión logística sin entrenar y un conjunto de datos. La columna de etiqueta debe establecerse en *income* en las propiedades del módulo [Cross-Validate Model][cross-validate-model] (Modelo de validación cruzada). Después de ejecutar el experimento y hacer clic en el puerto de la salida derecha del módulo [Validar modelo de forma cruzada][cross-validate-model], es posible ver los valores de métricas de clasificación binaria de cada subconjunto, además de las desviaciones media y estándar de cada uno. 

![Validación cruzada de un modelo de clasificación binaria](./media/evaluate-model-performance/8.png)

Figura 8. Validación cruzada de un modelo de clasificación binaria.

![Resultados de la validación cruzada de un clasificador binario](./media/evaluate-model-performance/9.png)

Figura 9. Resultados de la validación cruzada de un clasificador binario.

## <a name="evaluating-a-multiclass-classification-model"></a>Evaluación de un modelo de clasificación multiclase
En este experimento se usará el conocido conjunto de datos [Iris](https://archive.ics.uci.edu/ml/datasets/Iris "Iris"), que contiene las instancias de tres tipos (clases) distintos de la planta iris. Hay cuatro valores de características (longitud y ancho del sépalo y del pétalo) para cada instancia. En los experimentos anteriores se entrenaron y probaron los modelos con los mismos conjuntos de datos. Aquí usaremos el módulo [Dividir datos][split] para crear dos subconjuntos de los datos, con el fin de entrenar en el primero y puntuar y evaluar en el segundo. El conjunto de datos Iris está disponible públicamente en [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/index.html) (Repositorio de aprendizaje automático de UCI) y se puede descargar mediante un módulo [Importar datos][import-data].

### <a name="creating-the-experiment"></a>Creación del experimento
Agregue los módulos siguientes al área de trabajo en Azure Machine Learning Studio (clásico):

* [Import Data][import-data]
* [Bosque de decisión multiclase][multiclass-decision-forest]
* [División de datos][split]
* [Train Model][train-model] (entrenar modelo)
* [Score Model][score-model] (puntuar modelo)
* [Evaluación de módulo][evaluate-model]

Conecte los puertos tal como se muestra a continuación en la Figura 10.

Establezca el índice de la columna de etiqueta del módulo [Entrenar modelo][train-model] en 5. El conjunto de datos no tiene fila de encabezado, pero se sabe que las etiquetas de clase están en la quinta columna.

Haga clic en el módulo [Importar datos][import-data] y establezca la propiedad *Origen de datos* en la *dirección URL web a través de HTTP* y la *dirección URL* en http://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data.

Establezca la fracción de instancias que se usará para el entrenamiento en el módulo [Dividir datos][split] (por ejemplo, 0,7).

![Evaluar un clasificador multiclase](./media/evaluate-model-performance/10.png)

Figura 10. Evaluar un clasificador multiclase

### <a name="inspecting-the-evaluation-results"></a>Inspección de los resultados de la evaluación
Ejecute el experimento y haga clic en el puerto de salida de [Evaluar modelo][evaluate-model]. En este caso, los resultados de la evaluación se presentan en forma de una matriz de confusión. La matriz muestra las instancias reales frente a las predichas para las tres clases.

![Resultados de la evaluación de clasificación multiclase](./media/evaluate-model-performance/11.png)

Figura 11. Resultados de la evaluación de clasificación multiclase.

### <a name="using-cross-validation"></a>Uso de la validación cruzada
Tal como se mencionó anteriormente, puede realizar procesos de entrenamiento, puntuación y evaluación de forma repetida y automática mediante el módulo [Validar modelo de forma cruzada][cross-validate-model]. Necesitaría un conjunto de datos, un modelo sin entrenar y un módulo [Validar modelo de forma cruzada][cross-validate-model] (consulte la ilustración siguiente). De nuevo, debe establecer la columna de etiqueta del módulo [Evaluar modelo de forma cruzada][cross-validate-model] (en este caso, índice 5 de columna). Después de ejecutar el experimento y hacer clic en el puerto de salida derecho de [Validar modelo de forma cruzada][cross-validate-model], puede inspeccionar los valores de métricas de cada subconjunto, así como las desviaciones media y estándar. Las métricas que se muestran aquí son similares a las descritas en el caso de clasificación binaria. Sin embargo, en la clasificación multiclase, se realiza el cálculo de los positivos y negativos verdaderos, y de los falsos positivos y negativos con un recuento por clase, ya que no existe ninguna clase general positiva o negativa. Por ejemplo, al calcular la precisión o la recuperación de la clase 'Iris-setosa', se supone que se trata de la clase positiva y que todas las demás son negativas.

![Validación cruzada de un modelo de clasificación multiclase](./media/evaluate-model-performance/12.png)

Ilustración 12. Validación cruzada de un modelo de clasificación multiclase.

![Resultados de una validación cruzada de un modelo de clasificación multiclase](./media/evaluate-model-performance/13.png)

Ilustración 13. Resultados de una validación cruzada de un modelo de clasificación multiclase.

<!-- Module References -->
[cross-validate-model]: https://msdn.microsoft.com/library/azure/75fb875d-6b86-4d46-8bcc-74261ade5826/
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[linear-regression]: https://msdn.microsoft.com/library/azure/31960a6f-789b-4cf7-88d6-2e1152c0bd1a/
[multiclass-decision-forest]: https://msdn.microsoft.com/library/azure/5e70108d-2e44-45d9-86e8-94f37c68fe86/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[two-class-logistic-regression]: https://msdn.microsoft.com/library/azure/b0fd7660-eeed-43c5-9487-20d9cc79ed5d/
