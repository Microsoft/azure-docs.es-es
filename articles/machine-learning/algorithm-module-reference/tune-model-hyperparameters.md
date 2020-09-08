---
title: Optimizar los hiperparámetros del modelo
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo de Optimizar los hiperparámetros del modelo en Azure Machine Learning para realizar un barrido de parámetros en un modelo con el fin de determinar la configuración óptima de los parámetros.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/17/2020
ms.openlocfilehash: 916d23670e1db22efe43e2966d96aa1c82f1852a
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782165"
---
# <a name="tune-model-hyperparameters"></a>Optimizar los hiperparámetros del modelo

En este artículo se describe cómo usar el módulo Tune Model Hyperparameters (Optimizar hiperparámetros del modelo) en el diseñador de Azure Machine Learning (versión preliminar). El objetivo es determinar los hiperparámetros óptimos para un modelo de Machine Learning. El módulo compila y prueba varios modelos con diferentes combinaciones de configuraciones. Compara las métricas de todos los modelos para obtener las combinaciones de valores. 

Los términos *parámetro* e *hiperparámetro* pueden ser confusos. Los *parámetros* del modelo son los que establece en el panel derecho del módulo. Básicamente, este módulo realiza una *barrido de parámetros* en la configuración de parámetros especificada. Aprende un conjunto óptimo de _hiperparámetros_, que puede ser diferente para cada árbol de decisión, conjunto de datos o método de regresión específico. El proceso de búsqueda de la configuración óptima a veces se denomina de *optimización*. 

El módulo admite el siguiente método para encontrar la configuración óptima de un modelo: *entrenamiento y optimización integrados*. En este método, se configura el conjunto de parámetros que se van a utilizar. A continuación, se deja que el módulo recorra en iteración varias combinaciones. El módulo mide la precisión hasta que encuentra un modelo "mejor". Con la mayoría de los módulos de aprendizaje, se puede elegir qué parámetros se deben cambiar durante el proceso de entrenamiento y cuáles deben permanecer fijos.

En función del tiempo que desee que se ejecute el proceso de optimización, puede decidir probar exhaustivamente todas las combinaciones. También puede acortar el proceso estableciendo una cuadrícula de combinaciones de parámetros y probando un subconjunto aleatorio de esta cuadrícula.

Este método genera un modelo entrenado que se puede guardar para volver a usar.  

> [!TIP] 
> Puede llevar a cabo una tarea relacionada. Antes de iniciar la optimización, aplique la selección de características para determinar las columnas o variables que tienen el máximo valor informativo.

## <a name="how-to-configure-tune-model-hyperparameters"></a>Cómo configurar la optimización de los hiperparámetros del modelo  

El aprendizaje de los hiperparámetros óptimos de un modelo de Machine Learning requiere un uso considerable de canalizaciones.

### <a name="train-a-model-by-using-a-parameter-sweep"></a>Entrenamiento de un modelo mediante un barrido de parámetros  

En esta sección se describe cómo realizar un barrido de parámetros básico, que entrena un modelo mediante el módulo Tune Model Hyperparameters (Optimizar hiperparámetros del modelo).

1.  Agregue el módulo Tune Model Hyperparameters (Optimizar hiperparámetros del modelo) a la canalización en el diseñador.

2.  Conecte un modelo no entrenado a la entrada que se encuentra más a la izquierda. 

    > [!NOTE] 
    > **Tune Model Hyperparameters** (Optimizar hiperparámetros del modelo) solo se puede conectar a los módulos de algoritmo de aprendizaje automático integrados y no es compatible con el modelo personalizado integrado **Crear modelo de Python**.


3.  Agregue el conjunto de datos que desea usar para el entrenamiento y conéctelo a la entrada central de Tune Model Hyperparameters (Optimizar hiperparámetros del modelo).  

    Opcionalmente, si tiene un conjunto de datos etiquetado, puede conectarlo al puerto de entrada situado más a la derecha (**Conjunto de datos de validación opcional**). Esto le permite medir la precisión durante el entrenamiento y la optimización.

4.  En el panel derecho de Optimizar hiperparámetros del modelo, elija un valor para **Parameter sweeping mode** (Modo de barrido de parámetros). Esta opción controla la manera en la que se seleccionan los parámetros.

    - **Cuadrícula completa**: Al seleccionar esta opción, el módulo recorre en bucle una cuadrícula predefinida por el sistema para probar diferentes combinaciones e identificar el mejor aprendizaje. Esta opción es útil si no sabe cuál podría ser la mejor configuración de parámetros y desea probar todas las combinaciones posibles de valores.

    - **Barrido aleatorio**: Cuando seleccione esta opción, el módulo elegirá aleatoriamente los valores de parámetro en un intervalo definido por el sistema. Debe especificar el número máximo de ejecuciones que desea que realice el módulo. Esta opción es útil si desea aumentar el rendimiento del modelo con las métricas de su elección pero conservando los recursos de computación.    

5.  En **Label column** (Columna de etiqueta), abra el selector de columnas para elegir una sola columna de etiqueta.

6.  Elija el número de ejecuciones:

    - **Máximo número de ejecuciones en el barrido aleatorio**: si elige un barrido aleatorio, puede especificar el número de veces que se debe entrenar el modelo con una combinación aleatoria de valores de parámetros.

7.  En **Ranking** (Clasificación), elija una sola métrica que se usará para clasificar los modelos.

    Al ejecutar un barrido de parámetros, el módulo calcula todas las métricas aplicables para el tipo de modelo y las devuelve en el informe **Sweep results** (Resultados de barrido). El módulo usa métricas independientes para los modelos de regresión y de clasificación.

    Sin embargo, la métrica que se elija determinará la clasificación de los modelos. Solo el modelo superior, clasificado por la métrica elegida, se genera como un modelo entrenado que se usará para la puntuación.

8.  En **Random seed** (Inicialización aleatoria), escriba el número que se usará para iniciar el barrido de parámetros. 

9. Envíe la canalización.

## <a name="results-of-hyperparameter-tuning"></a>Resultados de la optimización de hiperparámetros

Cuando se complete el entrenamiento:

+ Para ver los resultados del barrido, haga clic con el botón derecho en el módulo y, a continuación, seleccione **Visualizar**.

    La salida incluye todas las métricas de barrido y precisión aplicables al tipo de modelo, pero es la métrica seleccionada para la clasificación la que determina qué modelo se considera "el mejor".

+ Para guardar una instantánea del modelo entrenado, seleccione la pestaña **Outputs+logs** (Salidas+registros) en el panel derecho del módulo **Entrenar modelo**. Seleccione el icono **Register dataset** (Registrar conjunto de datos) para guardar el modelo como un módulo reutilizable.


## <a name="technical-notes"></a>Notas técnicas

Esta sección contiene detalles y sugerencias de implementación.

### <a name="how-a-parameter-sweep-works"></a>Funcionamiento de un barrido de parámetros

Al configurar un barrido de parámetros, se define el ámbito de la búsqueda. La búsqueda puede usar un número finito de parámetros seleccionados aleatoriamente. También puede ser una búsqueda exhaustiva en un espacio de parámetros que se defina.

+ **Barrido aleatorio**: esta opción entrena un modelo con un número establecido de iteraciones. 

  Especifique un intervalo de valores para recorrer en iteración y el módulo usa un subconjunto elegido aleatoriamente de esos valores. Los valores se eligen con el reemplazo, lo que significa que los números elegidos previamente de forma aleatoria no se quitan del grupo de números disponibles. Por lo tanto, la posibilidad de que cualquier valor se seleccione sigue siendo la misma en todos los pases.  

+ **Cuadrícula completa**: la opción de usar toda la cuadrícula significa que se prueban todas las combinaciones. Esta opción es la más completa, pero requiere más tiempo. 

### <a name="controlling-the-length-and-complexity-of-training"></a>Controlar la longitud y la complejidad del entrenamiento

La iteración en muchas combinaciones de configuración puede tomar mucho tiempo, por lo que el módulo proporciona varias maneras de restringir el proceso:

+ Limitar el número de iteraciones usadas para probar un modelo
+ Limitar el espacio de parámetros
+ Limitar tanto el número de iteraciones como el espacio de parámetros

Se recomienda canalizar con la configuración para determinar el método de entrenamiento más eficaz en un conjunto de datos y un modelo determinados.

### <a name="choosing-an-evaluation-metric"></a>Elección de una métrica de evaluación

Al final de las pruebas, el modelo presenta un informe que contiene la precisión de cada modelo, para que se puedan revisar los resultados de la métrica:

- En todos los modelos de clasificación binaria se usa un conjunto uniforme de métricas.
- En todos los modelos de clasificación de varias clases se usa la precisión.
- En los modelos de regresión se usa un conjunto diferente de métricas. 

Sin embargo, durante el entrenamiento, debe elegir una *sola* métrica para que se use en la clasificación de los modelos que se generan durante el proceso de optimización. Es posible que la mejor métrica varíe, según el problema empresarial y el costo de los falsos positivos y los falsos negativos.

#### <a name="metrics-used-for-binary-classification"></a>Métricas usadas para la clasificación binaria

-   La **precisión** es la proporción de resultados verdaderos respecto al total de los casos.  

-   La **exactitud** es la proporción de resultados verdaderos respecto a los resultados positivos.  

-   La **coincidencia** es la fracción de todos los resultados correctos respecto a todos los resultados.  

-   La **puntuación F** es una medida que equilibra la exactitud y la coincidencia.  

-   **AUC** es un valor que representa el área bajo la curva cuando los falsos positivos se trazan en el eje X y los verdaderos positivos se trazan en el eje Y.  

-   La **pérdida logarítmica media** es la diferencia entre dos distribuciones de probabilidad: la verdadera y la del modelo.  

#### <a name="metrics-used-for-regression"></a>Métricas usadas para la regresión

-   El **error absoluto medio** calcula la media de todos los errores del modelo, donde *error* significa la distancia entre el valor predicho y el valor verdadero. A menudo se abrevia como *EAM*.  

-   **La raíz del error medio cuadrado** mide el promedio de los cuadrados de los errores, y luego toma la raíz de ese valor. A menudo se abrevia como *RMSE*.  

-   **El error absoluto relativo** representa el error como un porcentaje del valor verdadero.  

-   El **error cuadrático relativo** normaliza el error cuadrático total dividiéndolo por el error cuadrático total de los valores predichos.  

-   El **coeficiente de determinación** es un solo número que indica cómo se ajustan los datos a un modelo. Un valor de uno significa que el modelo coincide exactamente con los datos. Un valor de cero significa que los datos son aleatorios o que no se pueden ajustar al modelo. A menudo se denomina *r<sup>2</sup>* , *R<sup>2</sup>* o *r al cuadrado*.  

### <a name="modules-that-dont-support-a-parameter-sweep"></a>Módulos que no admiten un barrido de parámetros

Casi todos los aprendizajes en Azure Machine Learning admiten la validación cruzada con un barrido de parámetros integrado, lo que le permite elegir los parámetros con los que desea realizar la canalización. Si el usuario no admite la configuración de un intervalo de valores, puede usarlo en la validación cruzada. En este caso, se selecciona un intervalo de valores permitidos para el barrido. 


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 

