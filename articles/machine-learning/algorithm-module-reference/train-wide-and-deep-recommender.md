---
title: Uso del módulo Train Wide & Deep Recommender
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Train Wide & Deep Recommender.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/12/2020
ms.openlocfilehash: a548a1aa6b7c6382d00e218f1b61347002df2b38
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907781"
---
# <a name="train-wide--deep-recommender"></a>Train Wide & Deep Recommender
En este artículo se describe cómo usar el módulo **Aprenda a usar el módulo Train Wide & Deep Recommender** de Azure Machine Learning Designer para entrenar un modelo de recomendación. Este módulo se basa en el aprendizaje ancho y profundo que ofrece Google.

En el módulo **Train Wide & Deep Recommender** se estudia un conjunto de datos de triples usuario-elemento-calificación y, de manera opcional, algunas características de usuario y elemento. Devuelve un recomendador Wide & Deep entrenado.  Después, puede usar el modelo entrenado para generar predicciones o recomendaciones de clasificación mediante el uso del módulo [Score Wide and Deep Recommender](score-wide-and-deep-recommender.md).  

## <a name="more-about-recommendation-models-and-the-wide--deep-recommender"></a>Más información sobre los modelos de recomendación y el recomendador Wide & Deep  

El objetivo principal de un sistema de recomendación es recomendar uno o más *elementos* a *usuarios* del sistema. Algunos ejemplos de elementos pueden ser una película, un restaurante, un libro o una canción. Un usuario podría ser una persona, un grupo de personas u otra entidad con preferencias de elementos.  

Existen dos enfoques principales para los sistemas de recomendación. 

+ La primera es el **enfoque**  basado en el contenido, que hace uso de las características tanto para usuarios como para elementos. Los usuarios pueden describirse en propiedades como la edad y el sexo, y los elementos se pueden describir en propiedades como el autor y el fabricante. Los ejemplos típicos de sistemas de recomendación basados en contenido se pueden encontrar en los sitios de encuentros sociales. 
+ El segundo enfoque es de **filtrado de colaboración**, que solo usa los identificadores de usuarios y elementos, y obtiene información implícita sobre estas entidades a partir de una matriz (dispersa) de clasificaciones dadas a los elementos por los usuarios. Podemos obtener información sobre un usuario a partir de los elementos que han clasificado y de otros usuarios que han calificado los mismos elementos.  

El recomendador Wide & Deep combina estos enfoques que utilizan el filtrado de colaboración con un enfoque basado en contenido. Por tanto, se considera un **recomendador híbrido**. 

Funcionamiento: cuando se trata de un usuario relativamente nuevo en el sistema, para mejorar las predicciones se usa la información de características sobre el usuario, dando respuesta así al conocido problema del "arranque en frío". Sin embargo, una vez que ha recopilado clasificaciones suficientes de un usuario determinado, es posible crear predicciones completamente personalizadas para el usuario según sus clasificaciones específicas, en lugar de hacerlo solo según sus características. Por lo tanto, se trata de una transición sin problemas desde recomendaciones basadas en contenido a recomendaciones basadas en el filtrado de colaboración. Incluso si las características de elemento o usuario no están disponibles, el recomendador Wide & Deep funcionará de todos modos en su modalidad de filtrado de colaboración.  

Puede encontrar más detalles sobre el recomendador Wide & Deep y su algoritmo de probabilidad subyacente en el documento de investigación pertinente (en inglés): [Aprendizaje ancho y profundo de sistemas de recomendación](https://arxiv.org/pdf/1606.07792.pdf).  

## <a name="how-to-configure-train-wide--deep-recommender"></a>Configuración del entrenamiento del recomendador Wide & Deep  

+ [Preparación de los datos de entrenamiento](#prepare-data)
+ [Entrenamiento del modelo](#train-the-model)

### <a name="prepare-data"></a>Preparación de los datos

Antes de intentar usar el módulo, es esencial que los datos tengan el formato esperado por el modelo de recomendación. Se requiere un conjunto de datos de entrenamiento de **triples usuario-elemento-calificación**, pero también puede incluir características de usuario y características de elementos (si están disponibles) en conjuntos de datos independientes.

#### <a name="required-dataset-of-user-item-ratings"></a>Se requiere un conjunto de datos del usuario-elemento-clasificaciones

Los datos de entrada que se usan para el entrenamiento deben contener el tipo de datos correcto en el formato apropiado: 

+ La primera columna debe contener identificadores de usuario.
+ La segunda columna debe contener identificadores de elemento.
+ La tercera columna contiene la clasificación del par usuario-elemento. Los valores de clasificación deben ser numéricos. 

Por ejemplo, un conjunto típico de usuario-elemento-calificación puede tener el siguiente aspecto:

|UserId|MovieId|Rating|
|------------|-------------|------------|
|1|%#68646|10|
|223|31381|10|

#### <a name="user-features-dataset-optional"></a>Conjunto de datos de características del usuario (opcional)

El conjunto de datos de **características del usuario** debe contener identificadores para los usuarios y utilizar los mismos identificadores que se proporcionaron en la primera columna del conjunto de datos de usuario-elemento-calificación. Las demás columnas pueden contener cualquiera de las características que describen a los usuarios.  

Por ejemplo, un conjunto típico de características de usuario puede tener el siguiente aspecto: 

|UserId|Age|Sexo|Aficiones|Location|
|------------|--------------|-----------------------|---------------|------------|
|1|25|hombre| Teatro    |Europa|
|223|40|mujer|Historias románticas|Asia|

#### <a name="item-features-dataset-optional"></a>Conjunto de datos de características del elemento (opcional)

El conjunto de datos de características del elemento debe contener identificadores de elemento en la primera columna. Las demás columnas pueden contener cualquiera de las características descriptivas de los elementos.  

Por ejemplo, un conjunto típico de características del elemento puede tener el siguiente aspecto:  

|MovieId|Título|Idioma original|Géneros|Year|
|-------------|-------------|-------------------|-----------|---------------|
|%#68646|El padrino|Inglés|Teatro|1972|
|31381|Lo que el viento se llevó|Inglés|Historial|1939|

### <a name="train-the-model"></a>Entrenamiento del modelo

1.  Agregue el módulo **Train Wide and Deep Recommender** al experimento del diseñador y conéctelo al conjunto de datos de entrenamiento.  
  
2. Si tiene un conjunto de información independiente de características del usuario y/o características del elemento, conéctelo al módulo **Train Wide and Deep Recommender**.  
  
    - **Conjunto de datos de características del usuario**: conecte el conjunto de datos que describe a los usuarios en la segunda entrada.
    - **Conjunto de datos de características del elemento**: conecte el conjunto de datos que describe a los elementos en la tercera entrada.  
    
3.  **Épocas**: indican el número de veces que el algoritmo debe procesar todos los datos de entrenamiento. 

    Cuanto mayor sea este número, más adecuado será el entrenamiento; sin embargo, el entrenamiento conllevará más tiempo y puede provocar un sobreajuste.

4. **Tamaño de lote**: escriba el número de ejemplos de entrenamiento que se usan en un paso de entrenamiento. 

     Este hiperparámetro puede influir en la velocidad del entrenamiento. Un mayor tamaño de lote conduce a un costo de tiempo menor, pero puede aumentar el tiempo de convergencia. Y si el lote es demasiado grande para que se ajuste a la GPU o CPU, puede producirse un error de memoria.

5.  **Optimizador de la parte amplia**: seleccione un optimizador para que aplique los gradientes a la parte amplia del modelo.

6.  **Tasa de aprendizaje del optimizador de la parte amplia**: escriba un número entre 0,0 y 2,0 que defina la tasa de aprendizaje del optimizador de la parte amplia.

    Este hiperparámetro determina el tamaño del paso en cada paso del entrenamiento mientras se desplaza hacia una función de pérdida mínima. Una tasa de aprendizaje demasiado grande puede provocar que el aprendizaje pase por alto los mínimos, mientras que una tasa de aprendizaje demasiado pequeña puede provocar un problema de convergencia.

7.  **Dimensión de características cruzadas**: Escriba la dimensión indicando los identificadores de usuario y las características de identificador de elemento deseados. 

    El recomendador Wide & Deep realiza de forma predeterminada una transformación entre productos con las características de identificador de usuario y de elemento. Al resultado cruzado se le aplicará el algoritmo hash según este número para garantizar la dimensión.

8.  **Optimizador de parte profunda**: seleccione un optimizador para que aplique los gradientes a la parte profunda del modelo.

9.  **Tasa de aprendizaje del optimizador de la parte profunda**: escriba un número entre 0,0 y 2,0 que defina la tasa de aprendizaje del optimizador de la parte profunda.

10.  **Dimensión de inserción del usuario**: escriba un número entero para especificar la dimensión de la inserción del identificador de usuario.

     El recomendador Wide & Deep crea las inserciones compartidas de identificadores de usuarios e identificadores de elementos para la parte amplia y la parte profunda.

11.  **Dimensión de inserción del elemento**: escriba un número entero para especificar la dimensión de la inserción del identificador de elemento.

12.  **Dimensión de inserción de características de categorías**: escriba un número entero para especificar las dimensiones de las inserciones de características de categorías.

     En el componente profundo del recomendador Wide & Deep, se aprende un vector de inserción para cada característica de categoría. Y estos vectores de inserción comparten la misma dimensión.

13.  **Unidades ocultas**: escriba el número de nodos ocultos del componente profundo. El número de nodos de cada capa está separado por comas. Por ejemplo, si escribe "1000,500,100", especifica que el componente profundo tiene tres capas, donde cada capa tiene 1000, 500 y 100 nodos respectivamente.

14.  **Función de activación**: seleccione una función de activación que se aplique a cada capa; el valor predeterminado es ReLU.

15.  **Omisiones**: escriba un número entre 0,0 y 1,0 para determinar la probabilidad de que se omitan salidas en cada capa durante el entrenamiento.

     La omisión es un método regularización para evitar que las redes neuronales se sobreajusten. Una decisión habitual para este valor es comenzar con 0,5, que parece estar cerca de ser óptimo para una amplia gama de redes y tareas.

16.  **Normalización por lotes**: seleccione esta opción para usar la normalización por lotes después de cada capa oculta del componente profundo.

     La normalización por lotes es una técnica que se usa para combatir el problema interno de desplazamiento de la covariable durante el entrenamiento de las redes. En general, puede ayudar a mejorar la velocidad, rendimiento y estabilidad de las redes. 

17.  Ejecución de la canalización

##  <a name="technical-notes"></a>Notas técnicas

El recomendador Wide & Deep entrena de forma conjunta modelos lineales y redes neuronales profundas para combinar las ventajas de la memorización y la generalización. El componente amplio acepta un conjunto de características sin procesar y de transformaciones de características para memorizar las interacciones de estas. Y con menos ingeniería de características, el componente profundo generaliza las combinaciones de características no visibles a través de inserciones de características densas de baja dimensionalidad. 

En la implementación del recomendador Wide & Deep, el módulo usa una estructura de modelos predeterminada. El componente amplio toma las inserciones de usuario, las inserciones de elementos y la transformación entre productos de identificadores de usuario e identificadores de elemento como entrada. Para la parte profunda del modelo, se aprende un vector de inserción para cada característica de categoría. Al igual que otros vectores de características numéricos, estos vectores se alimentan en la red neuronal profunda prealimentada. La parte amplia y la profunda se combinan mediante la suma de sus logaritmos de probabilidades de salida finales como predicción, lo cual conduce finalmente a una función de pérdida común para el entrenamiento conjunto.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) de Azure Machine Learning. 
