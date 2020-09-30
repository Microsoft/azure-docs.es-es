---
title: 'Recomendador puntuado SVD: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Obtenga más información sobre cómo usar el módulo Score SVD Recommender (Puntuar recomendador de SVD) en Azure Machine Learning para puntuar las predicciones de recomendación de un conjunto de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/10/2020
ms.openlocfilehash: bf62fa995724b8e1fff757e89945cc39db3d9842
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893706"
---
# <a name="score-svd-recommender"></a>Puntuación del recomendador SVD

En este artículo se describe cómo usar el módulo Score SVD Recommender (Puntuar recomendador de SVD) del diseñador de Azure Machine Learning. Use este módulo para crear predicciones con un modelo de recomendación entrenado basado en el algoritmo de descomposición de un solo valor (SVD).

El recomendador SVD puede generar dos tipos diferentes de predicciones:

- [Predecir las valoraciones para un usuario determinado y un elemento](#prediction-of-ratings)
- [Recomendar elementos a un usuario](#recommendations-for-users)

A la hora de crear el segundo tipo de predicciones, puede funcionar en uno de estos modos:

- **El modo de producción** que considera a todos los usuarios o elementos. Normalmente se usa en un servicio web.

  Puede crear puntuaciones para los nuevos usuarios, no solo para los usuarios que se ven durante la formación. Para más información, consulte las [notas técnicas](#technical-notes). 

- **El modo de evaluación** que funciona en un conjunto reducido de usuarios o elementos que se pueden evaluar. Normalmente se usa durante las operaciones de canalización.

Para más información sobre el algoritmo recomendador de SVD, consulte el artículo de investigación [técnicas de factorización de matriz para sistemas de recomendador](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf).

## <a name="how-to-configure-score-svd-recommender"></a>Cómo configurar el Recomendador puntuado SVD

Este módulo admite dos tipos distintos de predicciones, cada una con requisitos diferentes. 

###  <a name="prediction-of-ratings"></a>Predicción de clasificaciones

Cuando se predicen las clasificaciones, el modelo calcula el modo en que un usuario reaccionará a un elemento determinado, teniendo en cuenta los datos de entrenamiento. Los datos de entrada para la puntuación tienen que proporcionar tanto un usuario como el elemento a clasificar.

1. Agregue un modelo de recomendación capacitado a su canalización y conéctelo al **Recomendador capacitado SVD**. Tiene que crear el modelo con el módulo [Train SVD Recommender](train-SVD-recommender.md) (Entrenar recomendador de SVD).

2. En **Recommender prediction kind** (Tipo de predicción recomendada) seleccione **Rating Prediction** (Predicción de clasificación). No se necesita ningún otro parámetro.

3. Agregue los datos para los que quiera realizar las predicciones y conéctela al **Conjunto de datos para puntuar**.

   Para que el modelo pueda predecir las clasificaciones, el conjunto de datos de entrada tiene que contener pares de usuario-elemento.

   El conjunto de datos puede contener una tercera columna opcional de clasificaciones para el par de usuario-elemento en la primera y segunda columna. Pero la tercera columna se omitirá durante la predicción.

4. Envíe la canalización.

### <a name="results-for-rating-predictions"></a>Resultados de las predicciones de clasificación 

El conjunto de datos de salida contiene tres columnas: usuarios, elementos y la clasificación de predicción para cada elemento y usuario de entrada.

###  <a name="recommendations-for-users"></a>Recomendaciones para los usuarios 

Para recomendar elementos para los usuarios, debe proporcionar una lista de usuarios y elementos como entrada. A partir de estos datos, el modelo utiliza su conocimiento sobre los elementos y usuarios existentes para generar una lista de elementos con una posible apelación a cada usuario. Puede personalizar el número de recomendaciones devueltas. También puede establecer un umbral para el número de recomendaciones anteriores que son necesarias para generar una recomendación.

1. Agregue un modelo de recomendación capacitado a su canalización y conéctelo al **Recomendador capacitado SVD**.  Tiene que crear el modelo con el módulo [Train SVD Recommender](train-svd-recommender.md) (Entrenar recomendador de SVD).

2. Para recomendar elementos para una lista de usuarios, establezca **Recommender prediction kind** (Tipo de predicción recomendada) en **Item Recommendation** (Recomendación de elemento).

3. En **Recommended item selection** (Selección de elementos recomendados), indique si está usando el módulo de puntuación en producción o para la evaluación de modelos. Elija uno de los valores siguientes:

    - **De todos los elementos**: Seleccione esta opción si va a configurar una canalización que se va a usar en un servicio web o en un entorno de producción.  Esta opción habilita el *modo de producción*. El módulo hace recomendaciones de todos los elementos que se han visto durante el entrenamiento.

    - **De los elementos clasificados (para la evaluación de modelos)** : Seleccione esta opción si va a desarrollar o probar un modelo. Esta opción habilita el *modo de evaluación*. El módulo hace recomendaciones solo de esos elementos del conjunto de datos de entrada que se han clasificado.
    
    - **De elementos sin clasificación (para sugerir nuevos elementos a los usuarios)** : Seleccione esta opción si quiere que el módulo haga recomendaciones solo de los elementos del conjunto de datos de entrenamiento que no se han clasificado. 

4. Agregue los conjuntos de datos para los que quiere crear predicciones y conéctela al **Conjunto de datos para puntuar**.

    - Para **From All Items** (De todos los elementos), el conjunto de datos de entrada debe constar de una columna. Esta contiene los identificadores de los usuarios para los que se van a hacer recomendaciones.

      El conjunto de datos puede incluir dos columnas extra de identificadores de elementos y clasificaciones, pero estas dos columnas se omiten. 

    - Para **From Rated Items (for model evaluation)** (De los elementos calificados [para la evaluación de modelos]), el conjunto de datos de entrada debe constar de pares de usuario-elemento. La primera columna debe contener el identificador de usuario. La segunda columna debe contener los identificadores de elemento correspondientes.

      El conjunto de datos puede incluir una tercera columna de clasificaciones de usuario-elemento, pero esta columna se omite.

    - Para **From Unrated Items (to suggest new items to users)** (De elementos sin clasificación [para sugerir nuevos elementos a los usuarios]), el conjunto de datos de entrada debe constar de pares usuario-elemento. La primera columna debe contener el identificador de usuario. La segunda columna debe contener los identificadores de elemento correspondientes.

     El conjunto de datos puede incluir una tercera columna de clasificaciones de usuario-elemento, pero esta columna se omite.

5. **Número máximo de elementos que se van a recomendar a un usuario**: Escriba el número de elementos que se devolverán para cada usuario. De forma predeterminada, el módulo recomienda cinco elementos.

6. **Tamaño mínimo del grupo de recomendaciones por usuario**: Escriba un valor que indique cuántas recomendaciones anteriores son necesarias. De forma predeterminada, este parámetro se establece en 2, lo que significa que por lo menos otros dos usuarios han recomendado el elemento.

   Use esta opción solo si va a puntuar en modo de evaluación. La opción no está disponible si selecciona **De todos los elementos** o **De elementos sin clasificación (para sugerir nuevos elementos a los usuarios)** .

7.  Para **De elementos Sin clasificación (para sugerir nuevos elementos a los usuarios)** , use el tercer puerto de entrada, denominado **Datos de aprendizaje**, para eliminar los elementos que ya se han clasificado desde los resultados de predicción.

    Para aplicar este filtro, conecte el conjunto de datos de aprendizaje original al puerto de entrada.

8. Envíe la canalización.

### <a name="results-of-item-recommendation"></a>Resultados de la recomendación de elementos

El conjunto de datos puntuado devuelto por Score SVD Recommender (Puntuar recomendador de SVD) de SVD enumera los elementos recomendados para cada usuario:

- La primera columna debe contener identificadores de usuario.
- Se generan varias columnas adicionales, en función del valor establecido para **Maximum number of items to recommend to a user** (Número máximo de elementos para recomendar a un usuario). Cada columna contiene un elemento recomendado (por identificador). Las recomendaciones se ordenan por afinidad de elemento de usuario. El elemento con la afinidad más alta se coloca en la columna **Elemento 1**.


##  <a name="technical-notes"></a>Notas técnicas

Si tiene una canalización con el recomendador de SVD y mueve el modelo a producción, tenga en cuenta que hay diferencias clave entre usar el recomendador en modo de evaluación y su uso en modo de producción.

La evaluación, por definición, requiere predicciones que se pueden comprobar con el *terreno real* en un conjunto de pruebas. Al evaluar el recomendador, tiene que predecir solo los elementos que se han clasificado en el conjunto de pruebas. Esto restringe los valores posibles que se predicen.

Cuando se pone en operación el modelo, normalmente se cambia el modo de predicción para realizar recomendaciones basadas en todos los elementos posibles, con el fin de obtener las mejores predicciones. Para muchas de estas predicciones, no hay datos ciertos. Por tanto, no se puede comprobar la precisión de la recomendación de la misma manera que durante las operaciones de canalización.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
