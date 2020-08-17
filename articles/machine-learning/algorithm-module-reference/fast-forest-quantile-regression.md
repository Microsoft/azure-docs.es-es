---
title: Regresión rápida de bosque por cuantiles: Referencia del módulo
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Regresión rápida de bosque por cuantiles para crear un modelo de regresión que puede predecir valores para un número especificado de cuantiles.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/13/2020
ms.openlocfilehash: 5f00164e90d625c5343103290e9272f15d164cd8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095394"
---
# <a name="fast-forest-quantile-regression"></a>Regresión rápida de bosque por cuantiles

En este artículo se describe un módulo del diseñador de Azure Machine Learning (versión preliminar).

Utilice este módulo para crear un modelo de regresión rápida de bosque por cuantiles en una canalización. La regresión rápida de bosque por cuantiles es útil si desea saber más acerca de la distribución del valor previsto, en lugar de obtener un valor de predicción medio único. Este método tiene muchas aplicaciones, como:  
  
- Predecir los precios  
  
- Estimar el rendimiento de estudiantes o aplicar gráficos de crecimiento para evaluar el desarrollo del niño  
  
- Detectar relaciones predictivas en casos donde hay solo una relación débil entre variables  
  
Este algoritmo de regresión es un método de aprendizaje **supervisado**; es decir, requiere un conjunto de datos etiquetado, que incluya una columna de etiquetas. Dado que se trata de un algoritmo de regresión, la columna de etiquetas debe contener solo valores numéricos.

## <a name="more-about-quantile-regression"></a>Más sobre la regresión por cuantiles

Hay muchos tipos diferentes de regresión. En el sentido más básico, la regresión consiste en ajustar un modelo a un destino expresado como un vector numérico. Pero los estadísticos llevan tiempo desarrollando métodos de regresión cada vez más avanzados.

La definición más sencilla de *cuantil* es un valor que divide un conjunto de datos en grupos de igual tamaño, y los valores de cuantiles marcan los límites entre los grupos. Estadísticamente hablando, los cuantiles son valores tomados a intervalos regulares desde el inverso de la función de distribución acumulativa (CDF) de una variable aleatoria.

Mientras que los modelos de regresión lineal intentan predecir el valor de una variable numérica usando una sola estimación, la *media*, a veces es necesario predecir el intervalo o la distribución completa de la variable de destino. Para estos fines, se han desarrollado técnicas, como la regresión bayesiana y la regresión por cuantiles.

La regresión por cuantiles le ayuda a comprender la distribución del valor de predicción. Los modelos de regresión por cuantiles basados en árbol, como el que se usa en este módulo, tienen la ventaja adicional de que se pueden usar para predecir las distribuciones no paramétricas.

  
## <a name="how-to-configure-fast-forest-quantile-regression"></a>Cómo configurar la regresión rápida de bosque por cuantiles

1. Agregue el módulo **Regresión rápida de bosque por cuantiles** a la canalización del diseñador. Puede encontrar este módulo en **Machine Learning Algorithms** (Algoritmos de aprendizaje automático), en la categoría **Regresión**.

2. En el panel derecho del módulo **Regresión rápida de bosque por cuantiles**, especifique cómo quiere que se entrene el modelo. Para ello, establezca la opción **Create trainer mode** (Crear modo entrenador).  
  
    - **Single Parameter** (Parámetro único): Si sabe cómo quiere configurar el modelo, proporcione un conjunto específico de valores como argumentos. Al entrenar el modelo, use [Entrenar modelo](train-model.md).
  
    - **Parameter Range** (Intervalo de parámetros): Si no está seguro de los mejores parámetros, puede realizar un barrido de parámetros mediante el módulo [Optimizar los hiperparámetros del modelo](tune-model-hyperparameters.md). El instructor recorre en iteración varios valores que usted especifica para encontrar la configuración óptima.

3. **Número de árboles**, escriba el número máximo de árboles que se pueden crear en el conjunto. Si crea más árboles, en general conseguirá una mayor precisión, pero a costa de un tiempo de entrenamiento mayor.  

4. **Número de hojas**, indique el número máximo hojas, o nodos terminales, que se pueden crear en un árbol.  

5. En **Número mínimo de instancias de aprendizaje necesarias para formar una hoja**, indique el número mínimo de ejemplos casos que son necesarios para crear cualquier nodo terminal (hoja) en un árbol.  
  
     Al aumentar este valor, aumenta el umbral para crear reglas nuevas. Por ejemplo, con el valor predeterminado de 1, incluso un solo caso puede provocar que se cree una regla nueva. Si aumenta el valor a 5, los datos de entrenamiento tienen que contener, como mínimo, cinco casos que cumplan las mismas condiciones.

6. **Fracción de ensacado**, especifique un número entre 0 y 1 que represente la fracción de las muestras que se van a usar al generar cada grupo de cuantiles. Las muestras se eligen aleatoriamente, con reemplazo.

7. **Fracción de división**, escriba un número entre 0 y 1 que represente la fracción de las características que se van a usar en cada división del árbol. Las características usadas siempre se eligen aleatoriamente.

8. **Cuantiles que se calcularán**, escriba una lista separada por punto y coma de los cuantiles por los que desea entrenar al modelo y crear predicciones.
  
     Por ejemplo, si desea crear un modelo que calcule por cuantiles, escribiría `0.25; 0.5; 0.75`.  

9. De manera opcional, escriba un valor en **Valor de inicialización de números aleatorios** para inicializar el generador de números aleatorios que usa el modelo.  El valor predeterminado es 0, lo que significa que se elige un valor de inicialización aleatoria.
  
     Debe proporcionar un valor si necesita reproducir los resultados en ejecuciones sucesivas con los mismos datos.  

10. Conecte el conjunto de datos de entrenamiento y el modelo sin entrenar a uno de los módulos de entrenamiento: 

    - Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), use el módulo [Entrenar modelo](train-model.md).

    - Si establece **Create trainer mode** (Crear modo entrenador) para **Parameter Range** (Intervalo de parámetros), use el módulo [Optimizar los hiperparámetros del modelo](tune-model-hyperparameters.md).

    > [!WARNING]
    > 
    > - Si pasa un intervalo de parámetros a [Entrenar modelo](train-model.md), solo utiliza el primer valor en la lista del intervalo de parámetros.
    > 
    > - Si pasa un único conjunto de valores de parámetro al módulo [Optimizar los hiperparámetros del modelo](tune-model-hyperparameters.md), cuando espera un intervalo de valores para cada parámetro, omite los valores y usa los valores predeterminados para el aprendiz.
    > 
    > - Si selecciona la opción **Parameter Range** (Intervalo de parámetros) y especifica un valor único para algún parámetro, ese valor único que haya especificado se utilizará en todo el barrido, incluso si otros parámetros cambian en un intervalo de valores.

11. Envíe la canalización.

## <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para guardar una instantánea del modelo entrenado, seleccione el módulo de entrenamiento y cambie a la pestaña **Outputs+logs** (Salidas+registros) del panel derecho. Haga clic en el icono **Registro de un conjunto de datos**.  El modelo guardado se encuentra como un módulo más en el árbol de módulos.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning.
