---
title: Regresión de Poisson Referencia del módulo
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Regresión de Poisson para crear un modelo de regresión de Poisson.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/13/2020
ms.openlocfilehash: 6682d9426ed3fe011fe5c493ec34fcdf0a2b35b7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095143"
---
# <a name="poisson-regression"></a>Regresión de Poisson

En este artículo se describe un módulo del diseñador de Azure Machine Learning (versión preliminar).

Utilice este módulo para crear un modelo de regresión de Poisson en una canalización. La regresión de Poisson está pensada para predecir valores numéricos, normalmente recuentos. Por lo tanto, debe usar este módulo para crear el modelo de regresión solo si los valores que está intentando predecir se ajustan a las siguientes condiciones:

- La variable de respuesta tiene una [distribución de Poisson](https://en.wikipedia.org/wiki/Poisson_distribution).  

- Los recuentos no pueden ser negativos. El método fallará por completo si intenta utilizarlo con etiquetas negativas.

- Una distribución de Poisson es una distribución discreta; por lo tanto, no tiene sentido usar este método con números no enteros.

> [!TIP]
> Si el destino no es un recuento, la regresión de Poisson probablemente no sea un método adecuado. Pruebe [otros módulos de regresión en el diseñador](https://docs.microsoft.com/azure/machine-learning/algorithm-module-reference/module-reference#machine-learning-algorithms). 

Después de haber configurado el método de regresión, debe entrenar el modelo mediante un conjunto de datos que contenga ejemplos del valor que desea predecir. A continuación, el modelo entrenado podrá usarse para realizar predicciones.

## <a name="more-about-poisson-regression"></a>Más información sobre la regresión de Poisson

La regresión de Poisson es un tipo especial de análisis de regresión que se utiliza normalmente para modelar recuentos. Por ejemplo, la regresión de Poisson sería útil para:

- Modelar el número de resfriados asociados con vuelos en avión

- Calcular el número de llamadas al servicio de emergencia durante un evento

- Proyectar el número de consultas de clientes posteriores a una promoción

- Creación de tablas de contingencia

Dado que la variable de respuesta tiene una distribución de Poisson, el modelo hace diferentes suposiciones sobre los datos y su distribución de probabilidad que, por ejemplo, la regresión de mínimos cuadrados. Por lo tanto, los modelos de Poisson deben interpretarse de manera diferente a otros modelos de regresión.

## <a name="how-to-configure-poisson-regression"></a>Configuración de la regresión de Poisson

1. Agregue el módulo **Regresión de Poisson** a su canalización en el diseñador (versión preliminar). Puede encontrar este módulo en **Machine Learning Algorithms** (Algoritmos de aprendizaje automático), en la categoría **Regresión**.

2. Agregue un conjunto de datos que contenga datos de entrenamiento del tipo correcto. 

    Se recomienda usar [Normalize Data](normalize-data.md) (Normalizar datos) para normalizar el conjunto de datos de entrada antes de usarlo para entrenar el regresor.

3. En el panel derecho del módulo **Regresión de Poisson**, especifique cómo quiere que se entrene el modelo. Para ello, establezca la opción **Create trainer mode** (Crear modo entrenador).  
  
    - **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, proporcione un conjunto específico de valores como argumentos.
  
    - **Parameter Range** (Intervalo de parámetros): Si no está seguro de los mejores parámetros, puede realizar un barrido de parámetros mediante el módulo [Optimizar los hiperparámetros del modelo](tune-model-hyperparameters.md). El instructor recorre en iteración varios valores que usted especifica para encontrar la configuración óptima.
  
4. **Tolerancia de optimización**: Escriba un valor que defina el intervalo de tolerancia durante la optimización. Cuanto menor sea el valor, más lento y más preciso será el ajuste.

5. **L1 regularization weight** (Ponderación de regularización L1), **L2 regularization weight** (Ponderación de regularización L2): Escriba valores para la regularización L1 y L2. La *regularización* agrega restricciones al algoritmo sobre aspectos del modelo que son independientes de los datos de entrenamiento. La regularización se utiliza habitualmente para evitar el sobreajuste. 

    - La regularización L1 es útil si el objetivo es tener un modelo que sea tan disperso como sea posible.

        La regularización L1 se realiza restando el peso L1 del vector de peso de la expresión de pérdida que el aprendiz está intentando minimizar. La norma L1 es una buena aproximación a la norma L0, que es el número de coordenadas distintas de cero.

    - La regularización L2 impide que una única coordenada del vector de peso crezca demasiado en magnitud. La regularización L2 es útil si el objetivo es tener un modelo con pesos generales pequeños.

    En este módulo, puede aplicar una combinación de regularizaciones L1 y L2. Si combina regularizaciones L1 y L2, puede imponer una penalización en la magnitud de los valores de parámetro. El aprendiz intenta minimizar la penalización, buscando el equilibrio con la minimización de la pérdida.

    Para obtener una buena explicación de las regularizaciones L1 y L2, consulte [Regularización L1 y L2 para Machine Learning](https://msdn.microsoft.com/magazine/dn904675.aspx).

6. **Tamaño de memoria para L-BFGS**: Especifique la cantidad de memoria que se reservará para la optimización y el ajuste del modelo.

     L-BFGS es un método específico para la optimización, basado en el algoritmo Broyden-Fletcher-Goldfarb-Shanno (BFGS). El método utiliza una cantidad limitada de memoria (L) para calcular la dirección del siguiente paso.

     Al cambiar este parámetro, puede afectar el número de posiciones y gradientes pasados que se almacenan para el cálculo del siguiente paso.

7. Conecte el conjunto de datos de entrenamiento y el modelo sin entrenar a uno de los módulos de entrenamiento: 

    - Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), use el módulo [Entrenar modelo](train-model.md).

    - Si establece **Create trainer mode** (Crear modo entrenador) para **Parameter Range** (Intervalo de parámetros), use el módulo [Optimizar los hiperparámetros del modelo](tune-model-hyperparameters.md).

    > [!WARNING]
    > 
    > - Si pasa un intervalo de parámetros a [Entrenar modelo](train-model.md), solo utiliza el primer valor en la lista del intervalo de parámetros.
    > 
    > - Si pasa un único conjunto de valores de parámetro al módulo [Optimizar los hiperparámetros del modelo](tune-model-hyperparameters.md), cuando espera un intervalo de valores para cada parámetro, omite los valores y usa los valores predeterminados para el aprendiz.
    > 
    > - Si selecciona la opción **Parameter Range** (Intervalo de parámetros) y especifica un valor único para algún parámetro, ese valor único que haya especificado se utilizará en todo el barrido, incluso si otros parámetros cambian en un intervalo de valores.

8.  Envíe la canalización.

## <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para guardar una instantánea del modelo entrenado, seleccione el módulo de entrenamiento y cambie a la pestaña **Outputs+logs** (Salidas+registros) del panel derecho. Haga clic en el icono **Registro de un conjunto de datos**.  El modelo guardado se encuentra como un módulo más en el árbol de módulos. 

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
