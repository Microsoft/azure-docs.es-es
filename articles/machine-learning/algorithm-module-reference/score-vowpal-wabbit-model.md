---
title: Puntuación del modelo de Vowpal Wabbit
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Puntuación del modelo de Vowpal Wabbit para generar puntuaciones para un conjunto de datos de entrada, utilizando un modelo de Vowpal Wabbit entrenado existente.
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/02/2020
ms.openlocfilehash: 23dfee7b78f2606c54525391e1260af69a9b0779
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898398"
---
# <a name="score-vowpal-wabbit-model"></a>Puntuación del modelo de Vowpal Wabbit
En este artículo se describe cómo usar el módulo **Puntuación del modelo de Vowpal Wabbit** en el diseñador de Azure Machine Learning para generar puntuaciones para un conjunto de datos de entrada mediante un modelo de Vowpal Wabbit entrenado existente.  

Este módulo proporciona la versión más reciente del marco de Vowpal Wabbit, versión 8.8.1. Use este módulo para puntuar los datos mediante un modelo entrenado guardado con el formato VW, versión 8.  

## <a name="how-to-configure-score-vowpal-wabbit-model"></a>Configuración del módulo Puntuación del modelo de Vowpal Wabbit

1.  Agregue el módulo **Puntuación del modelo de Vowpal Wabbit** al experimento.  
  
2.  Agregue un modelo de Vowpal Wabbit entrenado y conéctelo al puerto de entrada de la izquierda. Puede usar un modelo entrenado creado en el mismo experimento o buscar un modelo guardado en la categoría **Conjuntos de datos** del panel de navegación izquierdo del diseñador. Sin embargo, el modelo debe estar disponible en el diseñador de Azure Machine Learning.  
  
    > [!NOTE]
    > Solo se admiten los modelos de Vowpal Wabbit 8.8.1; no puede conectar modelos guardados entrenados con otros algoritmos.
  
3.  Agregue el conjunto de datos de prueba y conéctelo al puerto de entrada derecho. Si el conjunto de datos de prueba es un directorio que contiene el archivo de datos de prueba, especifique el nombre del archivo de datos de prueba con **Nombre del archivo de datos de prueba**. Si el conjunto de datos de prueba es un archivo único, deje vacío el **Nombre del archivo de datos de prueba**.

4. En el cuadro de texto **Argumentos de VW**, escriba un conjunto de argumentos válidos de la línea de comandos para el ejecutable de Vowpal Wabbit.  

    Para información sobre cuáles son los argumentos de Vowpal Wabbit compatibles y no compatibles con Azure Machine Learning, consulte la sección [Notas técnicas](#technical-notes).  

5.  **Nombre del archivo de datos de prueba**: escriba el nombre del archivo que contiene los datos de entrada. Este argumento se usa solo cuando el conjunto de datos de prueba es un directorio.

6. **Especificar el tipo de archivo**: indique el formato que usan los datos de entrenamiento. Vowpal Wabbit admite estos dos formatos de archivo de entrada:  

   - **VW** representa el formato interno que usa Vowpal Wabbit. Consulte la [página wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format) para más detalles. 
   - **SVMLight** es un formato que usan algunas de las otras herramientas de aprendizaje automático. 

7. Seleccione la opción **Incluir una columna adicional que contiene etiquetas** si quiere generar etiquetas junto con las puntuaciones.  

   Por lo general, cuando se administran datos de texto, Vowpal Wabbit no requiere etiquetas y solo devolverá las puntuaciones para cada fila de datos.  

8. Seleccione la opción **Incluir una columna adicional que contiene puntuaciones sin formato** si quiere generar puntuaciones sin formato junto con los resultados.  

9. Envíe la canalización.

## <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para visualizar los resultados, haga clic con el botón derecho en la salida del módulo [Puntuación del modelo de Vowpal Wabbit](score-vowpal-wabbit-model.md). La salida indica una puntuación de predicción normalizada de 0 a 1. 

+ Para evaluar los resultados, el conjunto de datos de resultados debe contener nombres de columnas de puntuación específicos que cumplan con los requisitos del módulo Evaluación del modelo.

  + En la tarea de regresión, el conjunto de datos que se va a evaluar debe tener una columna, denominada `Regression Scored Labels`, que representa las etiquetas con puntuación.
  + En la tarea de clasificación binaria, el conjunto de datos que se va a evaluar debe tener dos columnas, denominadas `Binary Class Scored Labels` y `Binary Class Scored Probabilities`, que representan las etiquetas con puntuación y las probabilidades, respectivamente.
  + En la tarea de clasificación múltiple, el conjunto de datos que se va a evaluar debe tener una columna, denominada `Multi Class Scored Labels`, que representa las etiquetas con puntuación.

  Tenga en cuenta que los resultados del módulo Puntuación del modelo de Vowpal Wabbit no se pueden evaluar de manera directa. Antes de la evaluación, se debe modificar el conjunto de datos según los requisitos anteriores.

##  <a name="technical-notes"></a>Notas técnicas

Esta sección contiene detalles de implementación, sugerencias y respuestas a las preguntas más frecuentes.

### <a name="parameters"></a>Parámetros

Vowpal Wabbit tiene muchas opciones de la línea de comandos para elegir y ajustar algoritmos. Aquí no se puede obtener una descripción completa de estas opciones. Se recomienda consultar la [página wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments).  

Los parámetros siguientes no se admiten en Azure Machine Learning Studio (clásico).  

-   Las opciones de entrada/salida especificadas en [https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments)  
  
     El módulo ya configura automáticamente estas propiedades.  
  
-   Tampoco se permite ninguna opción que genere o acepte varias salidas. Estas incluyen *`--cbt`* , *`--lda`* y *`--wap`* .  
  
-   Solo se admiten algoritmos de aprendizaje supervisado. Esto no permite estas opciones: *`–active`* , `--rank`, *`--search`* , etc.  

Se permiten todos los argumentos distintos de los descritos anteriormente.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 