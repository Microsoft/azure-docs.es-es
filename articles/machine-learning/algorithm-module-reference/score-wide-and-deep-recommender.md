---
title: Uso del módulo Score Wide & Deep Recommender
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Score Wide & Deep Recommender (Puntuar el recomendador Wide & Deep) en Azure Machine Learning para puntuar las predicciones de recomendación de un conjunto de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/12/2020
ms.openlocfilehash: 43dca9cd9f9000faae701e618e9a5fdf21d31ee3
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192744"
---
# <a name="score-wide-and-deep-recommender"></a>Puntuación del recomendador ancho y profundo

En este artículo se describe cómo usar el módulo **Score Wide and Deep Recommender** (Puntuar el recomendador Wide and Deep) en Azure Machine Learning Designer (versión preliminar) para crear predicciones basadas en un modelo de recomendación entrenado, que se basa en el aprendizaje Wide & Deep de Google.

El recomendador Wide and Deep puede generar dos tipos diferentes de predicciones:

- [Predecir las valoraciones para un usuario determinado y un elemento](#predict-ratings)

- [Recomendar elementos a un usuario determinado](#recommend-items)


Al crear el segundo tipo de predicciones, puede trabajar en *modo de producción* o en *modo de evaluación*.

- **El modo de producción** considera todos los usuarios o elementos, y normalmente se usa en un servicio Web. Puede crear puntuaciones para los nuevos usuarios, no solo para los usuarios que se ven durante la formación. 

- **El modo de evaluación** funciona en un conjunto reducido de usuarios o elementos que se pueden evaluar y se usa normalmente durante la experimentación.

Puede encontrar más detalles sobre el recomendador Wide and Deep y su teoría subyacente en el documento de investigación pertinente:  [Aprendizaje Wide & Deep para los sistemas de recomendador](https://arxiv.org/pdf/1606.07792.pdf).  

## <a name="how-to-configure-score-wide-and-deep-recommender"></a>Procedimientos para configurar Score Wide & Deep Recommender

Este módulo admite distintos tipos de recomendaciones, cada una con requisitos diferentes. Haga clic en el vínculo correspondiente al tipo de datos que tiene y el tipo de recomendación que desea crear.

+ [Predecir clasificaciones](#predict-ratings)
+ [Elementos recomendados](#recommend-items)

### <a name="predict-ratings"></a>Predecir clasificaciones

Cuando se predicen las clasificaciones, el modelo calcula el modo en que un usuario determinado reaccionará a un elemento determinado, dados los datos de formación. Por lo tanto, los datos de entrada para la puntuación deben proporcionar tanto un usuario como el elemento a calificar.

1. Agregue un modelo de recomendación Wide & Deep al experimento y conéctelo a un **modelo de recomendación Wide and Deep entrenado**.  Tiene que crear el modelo con el módulo [Train Wide and Deep Recommender](train-wide-and-deep-recommender.md) (Entrenar recomendador Wide and Deep).

2. **Tipo de predicción recomendada**: Seleccione  **Predicción de clasificación**. No se requieren más parámetros.

3. Agregue los datos para los que quiere crear predicciones y conéctela al **Conjunto de datos para puntuar**.

    Para predecir las clasificaciones, el conjunto de datos de entrada debe contener pares de usuario-elemento.

    El conjunto de datos puede contener una tercera columna opcional de clasificaciones para el par de usuario-elemento en la primera y segunda columna, pero la tercera columna se omitirá durante la predicción.

4.  (Opcional). Si tiene un conjunto de datos de características de usuario, conéctelo a las **características de usuario**.

    El conjunto de elementos de las características de usuario debe contener el identificador de usuario en la primera columna. Las columnas restantes deben contener valores que caracterizan a los usuarios, como su sexo, preferencias, ubicación, etc.
  
    **Score Wide & Deep Recommender** (Puntuar el recomendador Wide & Deep) ignora las características de los usuarios que han calificado elementos en el conjunto de datos de entrenamiento, porque ya se han aprendido durante el entrenamiento. Por lo tanto, filtre el conjunto de datos de antemano para incluir solo a los *usuarios de arranque en frío* o a los usuarios que no hayan clasificado ningún elemento.

    > [!WARNING]
    > Si el modelo se entrenó sin usar características de usuario, no puede introducir características de usuario durante la puntuación.

5. Si tiene un conjunto de datos de características de elemento, puede conectarlo a las **características de elemento**.

    El conjunto de datos de características de elemento tiene que contener un identificador de elemento en la primera columna. El resto de las columnas deben contener valores que caracterizan a los elementos.

    **Score Wide & Deep Recommender** (Puntuar el recomendador Wide & Deep) ignora las características de los elementos clasificados en el conjunto de datos de entrenamiento, porque ya se han aprendido durante el entrenamiento. Por lo tanto, restrinja el conjunto de datos de puntuación a los *elementos de arranque en frío* o elementos que no hayan sido clasificados por ningún usuario.

    > [!WARNING]
    > Si el modelo se entrena sin usar características de elementos, no puede estas durante la puntuación.

7. Ejecute el experimento.

### <a name="results-for-rating-predictions"></a>Resultados de las predicciones de clasificación 

El conjunto de datos de salida contiene tres columnas, que contienen el usuario, el elemento y la clasificación de predicción para cada usuario de entrada y elemento.

Además, se aplican los siguientes cambios durante la puntuación:

-  Los valores omitidos de una columna de características de elemento o usuario numérico se reemplazarán automáticamente por el **promedio** de los valores de conjunto de entrenamiento no omitidos. En cuanto a la característica de categoría, los valores que faltan se reemplazan por el mismo valor de categoría distinto de los valores posibles de esta característica.
-  No se aplica ninguna traslación a los valores de característica para mantener su dispersión.

### <a name="recommend-items"></a>Elementos recomendados

Para recomendar elementos para los usuarios, debe proporcionar una lista de usuarios y elementos como entrada. A partir de estos datos, el modelo utiliza su conocimiento sobre los elementos y usuarios existentes para generar una lista de elementos con una posible apelación a cada usuario. Puede personalizar el número de recomendaciones devueltas y establecer un umbral para el número de recomendaciones anteriores que se requieren para generar una recomendación.

1. Agregue un modelo de recomendación Wide and Deep al experimento y conéctelo a un **modelo de recomendación Wide and Deep entrenado**.  Tiene que crear el modelo con el módulo [Train Wide and Deep Recommender](train-wide-and-deep-recommender.md) (Entrenar recomendador Wide and Deep).

2. Para recomendar elementos para una lista de usuarios determinada, establezca **tipo de recomendador de predicción** en **Recomendación de elemento**.

3. **Selección de recomendador de elemento**: Indique si está usando el módulo de puntuación en producción o para la evaluación de modelos; para ello, elija uno de estos valores:

    - **De los elementos clasificados (para la evaluación de modelos)** : Seleccione esta opción si va a desarrollar o probar un modelo. Esta opción habilita el **modo de evaluación** y el módulo hace recomendaciones solo de los elementos del conjunto de datos de entrada que se han clasificado.
    - **De todos los elementos**: Seleccione esta opción si va a configurar un experimento que se va a usar en un servicio web o en un entorno de producción.  Esta opción habilita el **modo de producción**y el módulo hace recomendaciones de todos los elementos que se han detectado durante la formación.
    - **De elementos sin clasificación (para sugerir nuevos elementos a los usuarios)** : Seleccione esta opción si quiere que el módulo haga recomendaciones solo de los elementos del conjunto de datos de entrenamiento que no se han clasificado. 
4. Agregue los conjuntos de datos para los que quiere crear predicciones y conéctela al **Conjunto de datos para puntuar**.

    - Para **From All Items** (De todos los elementos), el conjunto de datos de entrada debe constar de una sola columna que contenga los identificadores de los usuarios para los que se van a hacer recomendaciones.

        El conjunto de datos puede incluir dos columnas extra de identificadores de elementos y clasificaciones, pero estas dos columnas se omiten. 

    - Para **From Rated Items (for model evaluation)** (De los elementos calificados [para la evaluación de modelos]), el conjunto de datos de entrada debe constar de **pares de usuario-elemento**. La primera columna debe contener el identificador de **usuario**. La segunda columna debe contener los identificadores de **elemento** correspondientes.

        El conjunto de datos puede incluir una tercera columna de clasificaciones de usuario-elemento, pero esta columna se omite.
        
    - Para **From Unrated Items (to suggest new items to users)** (De elementos sin clasificación [para sugerir nuevos elementos a los usuarios]), el conjunto de datos de entrada debe constar de pares usuario-elemento. La primera columna debe contener el identificador de usuario. La segunda columna debe contener los identificadores de elemento correspondientes.

        El conjunto de datos puede incluir una tercera columna de clasificaciones de usuario-elemento, pero esta columna se omite.

5. (Opcional). Si tiene un conjunto de datos de **características de usuario**, conéctelo a **User features** (características de usuario).

    La primera columna del conjunto de datos de características de usuario debe contener el identificador de usuario. Las columnas restantes deben contener valores que caracterizan a los usuarios, como su sexo, preferencias, ubicación, etc.

    **Score Wide & Deep Recommender** (Puntuar el recomendador Wide & Deep) ignora las características de los usuarios que han calificado elementos, porque estas ya se han aprendido durante el entrenamiento. Por lo tanto, puede filtrar el conjunto de datos de antemano para incluir solo a los *usuarios de arranque en frío* o a los usuarios que no hayan clasificado ningún elemento.

    > [!WARNING]
    >  Si el modelo se entrenó sin usar características de usuario, no puede utilizar características aplicadas durante la puntuación.

6. (Opcional) Si tiene un conjunto de datos de **características de elemento**, puede conectarlo a las **características de elemento**.

    La primera columna del conjunto de datos de características de elemento tiene que contener el identificador del elemento. El resto de las columnas deben contener valores que caracterizan a los elementos.

    **Score Wide & Deep Recommender** (Puntuar el recomendador Wide & Deep) ignora las características de los elementos clasificados, porque estas ya se han aprendido durante el entrenamiento. Por lo tanto, puede restringir el conjunto de datos de puntuación a los *elementos de arranque en frío* o elementos que no hayan sido clasificados por ningún usuario.

    > [!WARNING]
    >  Si el modelo se entrenó sin usar características de elementos, no utilice características de elemento cuando se realice la puntuación.  

7. **Número máximo de elementos que se van a recomendar a un usuario**: Escriba el número de elementos que se devolverán para cada usuario. De forma predeterminada, se recomiendan cinco elementos.

8. **Tamaño mínimo del grupo de recomendaciones por usuario**: Escriba un valor que indique cuántas recomendaciones anteriores son necesarias.  De forma predeterminada, este parámetro se establece en 2, lo que significa que el elemento debe haber sido recomendado por al menos otros dos usuarios.

    Esta opción solo se debe usar si va a puntuar en modo de evaluación. La opción no está disponible si selecciona **De todos los elementos** o **De elementos sin clasificación (para sugerir nuevos elementos a los usuarios)** .

9. Para **De elementos Sin clasificación (para sugerir nuevos elementos a los usuarios)** , use el tercer puerto de entrada, denominado **Datos de aprendizaje**, para eliminar los elementos que ya se han clasificado desde los resultados de predicción.

    Para aplicar este filtro, conecte el conjunto de datos de aprendizaje original al puerto de entrada.

10. Ejecute el experimento.
### <a name="results-of-item-recommendation"></a>Resultados de la recomendación de elementos

El conjunto de datos puntuado devuelto por **Score Wide and Deep Recommender** (Puntuar el recomendador Wide & Deep) enumera los elementos recomendados para cada usuario.

- La primera columna debe contener identificadores de usuario.
- Se generan varias columnas adicionales, en función del valor establecido para **Número máximo de elementos para recomendar a un usuario**. Cada columna contiene un elemento recomendado (por identificador). Las recomendaciones se ordenan por afinidad de usuario-elemento, con el elemento con la mayor afinidad colocada en la columna, **Elemento 1**.

##  <a name="technical-notes"></a>Notas técnicas

Esta sección contiene respuestas a algunas preguntas comunes sobre el uso del recomendador Wide & Deep para crear predicciones.  

###  <a name="cold-start-users-and-recommendations"></a>Usuarios de inicio en frío y recomendaciones

Normalmente, para crear recomendaciones, el módulo **Score Wide & Deep Recommender** (Puntuar el recomendador Wide & Deep) requiere las mismas entradas que usó al entrenar el modelo, incluido un identificador de usuario. Esto se debe a que el algoritmo necesita saber si ha aprendido algo sobre este usuario durante el entrenamiento. 

De todas formas, en el caso de los nuevos usuarios, puede que no tenga un identificador de usuario, solo algunas características del mismo, como la edad, el sexo, etc.

Aun así, puede crear recomendaciones para los usuarios que son nuevos en el sistema, tratándolos como *usuarios de arranque en frío*. Para estos usuarios, el algoritmo de recomendación no utiliza el historial o las clasificaciones anteriores, solo las funciones de usuario.

Para fines de predicción, un usuario de arranque en frío se define como usuario con un identificador que no se ha utilizado para el entrenamiento. Para asegurarse de que los identificadores no coinciden con los usados en el entrenamiento, puede crear identificadores nuevos. Por ejemplo, puede crear identificadores aleatorios dentro de un intervalo especificado o asignar una serie de identificadores previamente para los usuarios de arranque en frío.

De todas formas, si no tiene datos de filtrado de colaboración, como un vector de características de usuario, es mejor usar un aprendiz de clasificación o de regresión.

###  <a name="production-use-of-the-wide-and-deep-recommender"></a>Uso de producción del recomendador Wide and Deep

Si ha experimentado con el recomendador Wide and Deep y, a continuación, pasa el modelo a producción, tenga en cuenta estas diferencias importantes entre utilizar el recomendador en modo de evaluación y en modo de producción:

- La evaluación, por definición, requiere predicciones que se pueden comprobar con el *terreno real* en un conjunto de pruebas. Por lo tanto, al evaluar el recomendador, debe predecir solo los elementos que se han clasificado en el conjunto de pruebas. Esto restringe necesariamente los valores posibles que se predicen.

    Sin embargo, cuando se pone en operación el modelo, normalmente se cambia el modo de predicción para realizar recomendaciones basadas en todos los elementos posibles, con el fin de obtener las mejores predicciones. Para muchas de estas predicciones, no hay ningún terreno real correspondiente, por lo que no se puede comprobar la precisión de la recomendación de la misma manera que durante la experimentación.

- Si no proporciona un identificador de usuario en producción, y proporciona únicamente un vector de característica, puede obtener como respuesta una lista de todas las recomendaciones para todos los usuarios posibles. Asegúrese de proporcionar un identificador de usuario.

    Para limitar el número de recomendaciones que se devuelven, también puede especificar el número máximo de elementos devueltos por el usuario. 



## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) de Azure Machine Learning. 
