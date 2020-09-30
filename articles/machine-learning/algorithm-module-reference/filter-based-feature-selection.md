---
title: 'Selección de características basada en filtros: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo de selección de características basada en filtro de Azure Machine Learning para identificar las características de un conjunto de datos con la mayor eficacia predictiva.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/10/2019
ms.openlocfilehash: 1afa5df20c9bcbf63f8ad9f527e54f622eba3d19
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893790"
---
# <a name="filter-based-feature-selection"></a>Selección de características basada en filtro

En este artículo se describe cómo usar el módulo Selección de características basada en filtro en el diseñador de Azure Machine Learning. Este módulo ayuda a identificar las columnas del conjunto de datos de entrada con la máxima eficacia predictiva. 

En general, la *selección de características* hace referencia al proceso de aplicar pruebas estadísticas a las entradas, dada una salida especificada para determinar qué columnas son más predecibles de la salida. El módulo Filter Based Feature Selection (Selección de características basada en filtro) proporciona varios algoritmos de selección de características entre los que elegir. El módulo incluye métodos de correlación como la correlación de Pearson y valores de la χ2. 

Al usar el módulo Filter Based Feature Selection (Selección de características basada en filtro), se proporciona un conjunto de datos y se identifica la columna que contiene la etiqueta o la variable dependiente. A continuación, se especifica un único método que se va a usar para medir la importancia de la característica.

El módulo genera un conjunto de datos que contiene las mejores columnas de características, tal y como se clasifica por la potencia de predicción. También genera los nombres de las características y sus puntuaciones de la métrica seleccionada.  

## <a name="what-filter-based-feature-selection-is"></a>Qué es la selección de características basada en filtro  

Este módulo para la selección de características se denomina "basado en filtro" porque se usa la métrica seleccionada para identificar atributos irrelevantes y se filtran las columnas redundantes del modelo. Se elige una única medida estadística que se adapta a los datos y el módulo calcula una puntuación para cada columna de característica. Las columnas se devuelven según sus puntuaciones de características. 

Al elegir las características correctas, puede mejorar la precisión y la eficacia de la clasificación. 

Normalmente, solo se usan las columnas con las mejores puntuaciones para crear el modelo predictivo. Las columnas con puntuaciones de selección de características deficientes se pueden dejar en el conjunto de datos y omitirse al generar un modelo.  

## <a name="how-to-choose-a-feature-selection-metric"></a>Cómo elegir una métrica de selección de características

El módulo Filter Based Feature Selection (Selección de características basada en filtro) proporciona una variedad de métricas para evaluar el valor de la información de cada columna. En esta sección se proporciona una descripción general de cada métrica y cómo se aplica. Los requisitos adicionales para usar cada métrica se indican en las [notas técnicas](#technical-notes) y en las [instrucciones](#how-to-configure-filter-based-feature-selection) de configuración de cada módulo.

-   **Correlación de Pearson**  

    La estadística de correlación de Pearson o el coeficiente de correlación de Pearson, también se conoce en modelos estadísticos como el valor de `r`. Para dos variables cualesquiera, devuelve un valor que indica la fuerza de la correlación.

    El coeficiente de correlación de Pearson se calcula tomando la covarianza de dos variables y dividiendo por el producto de sus desviaciones estándar. Los cambios de escala en las dos variables no afecta al coeficiente.  

-   **χ2**  

    La prueba chi cuadrado bidireccional es un método estadístico que mide el modo en que los valores esperados cercanos son los resultados reales. El método supone que las variables son aleatorias y se dibujan a partir de un ejemplo adecuado de variables independientes. La estadística chi cuadrado resultante indica cuántos resultados provienen del resultado esperado (aleatorio).  


> [!TIP]
> Si necesita una opción diferente para el método de selección de características personalizado, use el módulo [Execute R Script](execute-r-script.md) (Ejecutar script R). 

## <a name="how-to-configure-filter-based-feature-selection"></a>Cómo configurar de la selección de características basada en filtros

Elija una métrica estadística estándar. El módulo calcula la correlación entre un par de columnas: la columna de etiquetas y una columna de características.

1.  Agregue el módulo Filter Based Feature Selection (Selección de características basada en filtro) a la canalización. Puede encontrarlo en la categoría **Selección de características** en el diseñador.

2. Conecte un conjunto de datos de entrada que contenga al menos dos columnas que sean características potenciales.  

    Para asegurarse de que se analiza una columna y se genera una puntuación de características, use el módulo [Edit Metadata](edit-metadata.md) (Editar metadatos) para establecer el atributo **IsFeature**. 

    > [!IMPORTANT]
    > Asegúrese de que las columnas que va a proporcionar como entrada son características potenciales. Por ejemplo, una columna que contiene un valor único no tiene ningún valor de información.
    >
    > Si sabe que hay columnas que crearían características incorrectas, puede quitarlas de la selección. También puede usar el módulo [Edit Metadata](edit-metadata.md) (Editar metadatos) para marcarlos como **Categorical** (Categórico). 
3.  Para el **Método de puntuación de características**, elija uno de los siguientes métodos estadísticos establecidos para usar en el cálculo de puntuaciones.  

    | Método              | Requisitos                             |
    | ------------------- | ---------------------------------------- |
    | Correlación de Pearson | La etiqueta puede ser de texto o numérica. Las características deben ser numéricas. |
    χ2| Las etiquetas y características pueden ser de texto o numéricas. Use este método para calcular la importancia de las características para dos columnas de categorías.|

    > [!TIP]
    > Si cambia la métrica seleccionada, se restablecerán todas las demás selecciones, así que asegúrese de establecer esta opción en primer lugar.
4.  Seleccione la opción **Operate on feature columns only** (Operar solo en las columnas de características) para generar una puntuación solo para las columnas que se marcaron anteriormente como características. 

    Si anula la selección de esta opción, el módulo creará una puntuación para cualquier columna que cumpla los criterios de otro modo, hasta el número de columnas especificado en **Number of desired features** (Número de características deseadas).  

5.  En **Target column** (Columna de destino), seleccione **Launch column selector** (Iniciar selector de columna) para elegir la columna de etiquetas por nombre o por su índice (los índices están basados en uno).  
    Se requiere una columna de etiqueta para todos los métodos que impliquen correlación estadística. El módulo devuelve un error en tiempo de diseño si no elige ninguna columna de etiqueta o varias columnas de etiqueta. 

6.  En **Number of desired features** (Número de características deseadas), escriba el número de columnas de características que desea que se devuelvan como resultado:  

    - El número mínimo de características que puede especificar es 1, pero se recomienda aumentar este valor.  

    - Si el número especificado de características deseadas es mayor que el número de columnas del conjunto de datos, se devuelven todas las características, incluso las que tienen puntuaciones de cero.  

    - Si especifica menos columnas de resultados que columnas de características, las características se clasifican por puntuación descendente y solo se devuelven las características principales. 

7.  Envíe la canalización o seleccione el módulo Selección de características basada en filtro y seleccione **Ejecutar seleccionadas**.


## <a name="results"></a>Results

Una vez completado el procesamiento:

+ Para ver una lista completa de las columnas de características que se han analizado y sus puntuaciones, haga clic con el botón derecho en el módulo y seleccione **Visualize** (Visualizar).  

+ Para ver el conjunto de datos en función de los criterios de selección de características, haga clic con el botón derecho en el módulo y seleccione **Visualize** (Visualizar). 

Si el conjunto de datos contiene menos columnas de las esperadas, compruebe la configuración del módulo y los tipos de datos de las columnas proporcionadas como entrada. Por ejemplo, si establece **Número de características deseadas** en 1, el conjunto de datos de salida solo contiene dos columnas: la columna de etiqueta y la columna de características con mayor grado de clasificación.


##  <a name="technical-notes"></a>Notas técnicas  

### <a name="implementation-details"></a>Detalles de la implementación

Si usa la correlación de Pearson en una característica numérica y una etiqueta de categoría, la puntuación de características se calcula de la manera siguiente:  

1.  Para cada nivel de la columna categórica, calcule la media condicional de la columna numérica.  

2.  Correlacione la columna de medias condicionales con la columna numérica.  

### <a name="requirements"></a>Requisitos  

-   No se puede generar una puntuación de selección de características para ninguna columna designada como **Label** (Etiqueta) o como columna **Score** (Puntuación).  

-   Si intenta usar un método de puntuación con una columna de un tipo de datos que el método no admite, el módulo generará un error. O bien, se le asignará una puntuación de cero a la columna.  

-   Si una columna contiene valores lógicos (verdadero/falso), se procesan como `True = 1` y `False = 0`.  

-   Una columna no puede ser una característica si se ha designado como **Label** (Etiqueta) o **Score** (Puntuación).  

### <a name="how-missing-values-are-handled"></a>Cómo se controlan los valores que faltan  

-   No se puede especificar como columna de destino (etiqueta) ninguna columna con todos los valores ausentes.  

-   Si una columna contiene valores ausentes, el módulo los omite al calcular la puntuación de la columna.  

-   Si una columna designada como columna de característica tiene todos los valores ausentes, el módulo le asigna una puntuación de cero.   


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 

