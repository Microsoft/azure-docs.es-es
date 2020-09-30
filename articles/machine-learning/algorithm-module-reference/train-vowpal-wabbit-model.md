---
title: Entrenamiento del modelo de Vowpal Wabbit
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Entrenamiento del modelo de Vowpal Wabbit para crear un modelo de Machine Learning con una instancia de Vowpal Wabbit.
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/02/2020
ms.openlocfilehash: 6bc9f69440be772910ea8200b5ccf7d5a5122ae6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907807"
---
# <a name="train-vowpal-wabbit-model"></a>Entrenamiento del modelo de Vowpal Wabbit
En este artículo se describe cómo usar el módulo **Entrenamiento del modelo de Vowpal Wabbit** en el diseñador de Azure Machine Learning para crear un modelo de Machine Learning con Vowpal Wabbit.  

Para usar Vowpal Wabbit para el aprendizaje automático, dé formato a la entrada según los requisitos de Vowpal Wabbit y prepare los datos en función del formato requerido. Use este módulo para especificar los argumentos de la línea de comandos de Vowpal Wabbit. 

Cuando se ejecuta la canalización, se carga una instancia de Vowpal Wabbit en tiempo de ejecución del experimento, junto con los datos especificados. Una vez que se completa el entrenamiento, el modelo se vuelve a serializar en el área de trabajo. Puede usar el modelo de inmediato para puntuar los datos. 

Para entrenar de manera incremental un modelo existente sobre los datos nuevos, conecte un modelo guardado al puerto de entrada **Modelo de Vowpal Wabbit entrenado previamente** de **Entrenamiento del modelo de Vowpal Wabbit** y agregue los datos nuevos al otro puerto de entrada.  

## <a name="what-is-vowpal-wabbit"></a>¿Qué es Vowpal Wabbit?  

Vowpal Wabbit (VW) es un marco de aprendizaje automático rápido y en paralelo que desarrolló Yahoo! Research para la computación distribuida. Más adelante, John Langford (Microsoft Research) lo llevó a Windows y lo adaptó para la informática científica en arquitecturas paralelas.  

Las características de Vowpal Wabbit que son importantes para el aprendizaje automático incluyen el aprendizaje continuo (aprendizaje en línea), la reducción de dimensiones y el aprendizaje interactivo. Vowpal Wabbit también es una solución de problemas cuando no puede ajustar los datos del modelo en la memoria.  

Los usuarios primarios de Vowpal Wabbit son científicos de datos que han usado anteriormente el marco de trabajo para tareas de aprendizaje automático como la clasificación, la regresión, el modelado de temas o la factorización de matriz. El contenedor de Azure para Vowpal Wabbit tiene características de rendimiento muy similares a la versión local, por lo que puede usar las características y el rendimiento nativo eficaces de Vowpal Wabbit y publicar fácilmente el modelo entrenado como un servicio implementado.  

El módulo [hash de características](feature-hashing.md) también incluye la funcionalidad proporcionada por Vowpal Wabbit que permite transformar los conjuntos de datos de texto en características binarias mediante un algoritmo hash.  

## <a name="how-to-configure-vowpal-wabbit-model"></a>Configuración del modelo de Vowpal Wabbit  

En esta sección se describe cómo entrenar un modelo nuevo y cómo agregar datos nuevos a un modelo existente.

A diferencia de otros módulos del diseñador, este módulo especifica los parámetros del módulo y entrena el modelo. Si tiene un modelo existente, puede agregarlo como entrada opcional para entrenar el modelo de forma incremental.

+ [Preparación de los datos de entrada en uno de los formatos necesarios](#prepare-the-input-data)
+ [Entrenamiento de un modelo nuevo](#create-and-train-a-vowpal-wabbit-model)
+ [Entrenamiento incremental de un modelo existente](#retrain-an-existing-vowpal-wabbit-model)

### <a name="prepare-the-input-data"></a>Preparación de los datos de entrada

Para entrenar un modelo con este módulo, el conjunto de datos de entrada debe constar de una columna de texto única con uno de estos dos formatos compatibles: **SVMLight** o **VW**. Esto no significa que Vowpal Wabbit solo analice datos de texto, sino que solo que las características y los valores deben estar preparados en el formato de archivo de texto requerido.  

Los datos se pueden leer de dos tipos de conjuntos de datos: un conjunto de datos de archivo o un conjunto de datos tabulares. Estos conjuntos de datos deben tener el formato SVMLight o VW. El formato de datos de Vowpal Wabbit tiene la ventaja de que no requiere un formato de columnas, que ahorra espacio al tratar con datos dispersos. Para más información sobre este formato, consulte la [página wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format).  

### <a name="create-and-train-a-vowpal-wabbit-model"></a>Creación y entrenamiento de un modelo de Vowpal Wabbit

1. Agregue el módulo **Entrenamiento del modelo de Vowpal Wabbit** al experimento. 
  
2. Agregue el conjunto de datos de entrenamiento y conéctelo a **Datos de entrenamiento**. Si el conjunto de datos de entrenamiento es un directorio que contiene el archivo de datos de entrenamiento, especifique el nombre del archivo de datos de entrenamiento con **Nombre del archivo de datos de entrenamiento**. Si el conjunto de datos de entrenamiento es un archivo único, deje vacío el **Nombre del archivo de datos de entrenamiento**.

3. En el cuadro de texto **Argumentos de VW**, escriba los argumentos de la línea de comandos para el ejecutable de Vowpal Wabbit.

     Por ejemplo, puede agregar *`–l`* para especificar la velocidad de aprendizaje o *`-b`* para indicar el número de bits de hash.  

     Para más información, consulte la sección [Parámetros de Vowpal Wabbit](#supported-and-unsupported-parameters).  

4. **Nombre del archivo de datos de entrenamiento**: escriba el nombre del archivo que contiene los datos de entrada. Este argumento se usa solo cuando el conjunto de datos de entrenamiento es un directorio.

5. **Especificar el tipo de archivo**: indique el formato que usan los datos de entrenamiento. Vowpal Wabbit admite estos dos formatos de archivo de entrada:  

    - **VW** representa el formato interno que usa Vowpal Wabbit. Consulte la [página wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format) para más detalles. 
    - **SVMLight** es un formato que usan algunas de las otras herramientas de aprendizaje automático. 

6. **Archivo de modelo legible de salida**: seleccione la opción si quiere que el módulo guarde el modelo legible en los registros de ejecución. Este argumento corresponde al parámetro `--readable_model` en la línea de comandos de VW.  

7. **Archivo hash invertido de salida**: seleccione la opción si quiere que el módulo guarde la función hash invertida a un archivo en los registros de ejecución. Este argumento corresponde al parámetro `--invert_hash` en la línea de comandos de VW.  

8. Envíe la canalización.

### <a name="retrain-an-existing-vowpal-wabbit-model"></a>Reentrenamiento de un modelo de Vowpal Wabbit existente

Vowpal Wabbit admite el entrenamiento incremental al agregar datos nuevos a un modelo existente. Hay dos maneras de obtener un modelo existente para el reentrenamiento:

+ Use la salida de otro módulo **Entrenamiento del modelo de Vowpal Wabbit** en la misma canalización.  
  
+ Busque un modelo guardado en la categoría **Conjuntos de datos** del panel de navegación de la izquierda del diseñador y arrástrelo a la canalización.  

1. Agregue el módulo **Entrenamiento del modelo de Vowpal Wabbit** a la canalización.  
2. Conecte el modelo previamente entrenado al puerto de entrada **Modelo de Vowpal Wabbit entrenado previamente** del módulo.
3. Conecte los datos de entrenamiento nuevos al puerto de entrada **Datos de entrenamiento** del módulo.
4. En el panel de parámetros de **Entrenamiento del modelo de Vowpal Wabbit**, especifique el formato de los datos de entrenamiento nuevos y también el nombre del archivo de datos de entrenamiento si el conjunto de datos de entrada es un directorio.
5. Seleccione las opciones **Archivo de modelo legible de salida** y **Archivo hash invertido de salida** si los archivos correspondientes se deben guardar en los registros de ejecución.

6. Envíe la canalización.  
7. Seleccione el módulo y seleccione **Register dataset** (Registrar conjunto de datos) en la pestaña **Outputs+logs** (Salidas y registros) para conservar el modelo actualizado en el área de trabajo de Azure Machine Learning.  Si no especifica un nombre nuevo, el modelo actualizado sobrescribe el modelo guardado existente.

## <a name="technical-notes"></a>Notas técnicas

Esta sección contiene detalles de implementación, sugerencias y respuestas a las preguntas más frecuentes.

### <a name="advantages-of-vowpal-wabbit"></a>Ventajas de Vowpal Wabbit

Vowpal Wabbit proporciona un aprendizaje extremadamente rápido sobre características no lineales como n gramos.  

Vowpal Wabbit usa técnicas de *aprendizaje en línea*, como descenso de gradiente estocástico (SGD), para ajustar un modelo a un registro en cada momento. Por lo tanto, itera muy rápidamente sobre datos sin procesar y puede desarrollar un buen predictor más rápido que la mayoría de otros modelos. Este enfoque también evita tener que leer todos los datos de entrenamiento en la memoria.  

Vowpal Wabbit convierte todos los datos en hashes, no solo los datos de texto, sino otras variables de categorías. El uso de hashes hace que la búsqueda de pesos de regresión sea más eficaz, que es fundamental para el descenso de gradiente estocástico.  

###  <a name="supported-and-unsupported-parameters"></a>Parámetros compatibles y no compatibles 

En esta sección se describe la compatibilidad con los parámetros de la línea de comandos de Vowpal Wabbit en el diseñador de Azure Machine Learning. 

Por lo general, se admiten todos los argumentos, excepto un conjunto limitado. Para ver una lista completa de los argumentos, consulte la [página wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments).    

No se admiten los parámetros siguientes:

-   Las opciones de entrada/salida especificadas en [https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments)  
  
     El módulo ya configura automáticamente estas propiedades.  
  
-   Tampoco se permite ninguna opción que genere o acepte varias salidas. Estas incluyen *`--cbt`* , *`--lda`* y *`--wap`* .  
  
-   Solo se admiten algoritmos de aprendizaje supervisado. Por lo tanto, no se admiten estas opciones: *`–active`* , `--rank`, *`--search`* etc. 

### <a name="restrictions"></a>Restricciones

Como el objetivo del servicio es admitir usuarios experimentados de Vowpal Wabbit, los datos de entrada se deben preparar con anterioridad con el formato de texto nativo de Vowpal Wabbit en lugar del formato del conjunto de datos que usan otros módulos.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
