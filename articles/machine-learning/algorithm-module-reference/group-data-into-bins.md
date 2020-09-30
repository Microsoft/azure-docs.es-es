---
title: 'Agrupación de datos en rangos: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo agrupar datos en ubicaciones para agrupar números o cambiar la distribución de datos continuos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/19/2020
ms.openlocfilehash: de9f4517d134b4a42603291110c6b19f69403412
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907041"
---
# <a name="group-data-into-bins-module"></a>Módulo de agrupación de datos en rangos

En este artículo se describe cómo usar el módulo de agrupación de datos en el diseñador de Azure Machine Learning para agrupar números o cambiar la distribución de datos continuos.

El módulo de agrupación de datos en rangos admite varias opciones para discretizar datos. Puede personalizar el establecimiento de los extremo del intervalo y cómo se reparten los valores en ellos. Por ejemplo, puede:  

+ Escriba manualmente una serie de valores que sirvan como límites de la ubicación.  
+ Asigne valores a los depósitos mediante *cuantiles* o las clasificaciones de percentiles.  
+ Forzar una distribución uniforme de valores en las ubicaciones.  

## <a name="more-about-binning-and-grouping"></a>Más información sobre cuantificación y agrupación

La *discretización* o agrupación de datos (a veces denominado *cuantificación*) es una herramienta importante para la preparación de datos numéricos para el aprendizaje automático. Es útil en escenarios como estos:

+ Una columna de números continuos tiene demasiados valores únicos para modelar de forma eficaz. Por lo tanto, puede asignar los valores a los grupos de forma automática o manual para crear un conjunto más pequeño de rangos discretos.

+ Quiere reemplazar una columna de números por valores de categorías que representen rangos específicos.

    Por ejemplo, puede que desee agrupar los valores de una columna de edad especificando rangos personalizados, como 1-15, 16-22, 23-30, etc., para los datos demográficos de los usuarios.

+ Un conjunto de datos tiene algunos valores extremos, todos ellos fuera del rango esperado, y estos valores tienen una influencia insuficiente en el modelo entrenado. Para mitigar el sesgo en el modelo, puede transformar los datos en una distribución uniforme mediante el método de cuantiles.

    Con este método, el módulo de agrupaciones de datos en rangos determina las ubicaciones y los anchos de rango ideales para asegurarse de que aproximadamente en cada rango haya el mismo número de muestras. A continuación, según el método de normalización que elija, los valores de los rangos se transforman en percentiles o se asignan a un número de rango.

### <a name="examples-of-binning"></a>Ejemplos de cuantificación

En el diagrama siguiente se muestra la distribución de los valores numéricos anteriores y posteriores a discretización con el método de *cuantiles*. Observe que, en comparación con los datos sin procesar de la izquierda, los datos se han cuantificado y transformado en una escala de unidad normal.  

Aquí puede ver un [ejemplo del resultado de la ejecución de esta canalización](https://ml.azure.com/visualinterface/authoring/Normal/87270db9-4651-448e-bd28-8ef7428084dc?wsid=%2Fsubscriptions%2Fe9b2ec51-5c94-4fa8-809a-dc1e695e4896%2Fresourcegroups%2Fmodule-ws-rg%2Fworkspaces%2Fmodule-prerelease-119&flight=cm&tid=72f988bf-86f1-41af-91ab-2d7cd011db47&smtendpoint=https%3A%2F%2Fsmt-test1.azureml-test.net).

Dado que hay muchas maneras de agrupar los datos, todos los que se pueden personalizar, se recomienda experimentar con distintos métodos y valores. 

## <a name="how-to-configure-group-data-into-bins"></a>Configuración de datos de grupo en contenedores

1. Agregue el módulo de **agrupación de datos en rangos** a su canalización en el diseñador. Puede encontrar este módulo en la categoría **Transformación de datos**.

2. Conecte el conjunto de datos que contiene datos numéricos a contenedor. La cuantificación solo se puede aplicar a las columnas que contienen datos numéricos. 

    Si el conjunto de datos contiene columnas no numéricas, use el módulo [Seleccionar columnas del conjunto de datos](select-columns-in-dataset.md) para proyectar un subconjunto de columnas con el que trabajar.

3. Especifique el modo cuantificación. El modo de discretización determina otros parámetros, por lo que debe asegurarse de seleccionar primero la opción **Modo de discretización**. Se admiten los siguientes tipos de cuantificación:

    - **Cuantiles**: El método de cuantil asigna valores a cubos basándose en rangos de percentil. Este método también se conoce como "alto de discretización de igualdad".

    - **Mismo ancho**: Con esta opción, debe especificar el número total de contenedores. Los valores de la columna se colocarán en rangos de forma que cada rango tiene el mismo intervalo entre los valores de inicio y fin. Como resultado, algunos contenedores pueden tener más valores si los datos se agrupan en torno a un punto determinado.

    - **Bordes personalizados**: Puede especificar los valores que comienzan cada cubo. El valor del borde siempre es el límite inferior del contenedor. 
    
      Por ejemplo, suponga que quiere agrupar los valores en dos rangos. Uno tendrá valores mayores que 0, y el otro, valores menores o iguales que 0. En este caso, para los extremos del rango, escriba **0** en la **lista separada por comas de los extremos del rango**. La salida del módulo sería 1 y 2, que indica el índice del rango de cada valor de fila. Tenga en cuenta que la lista de valores separados por comas debe estar en orden ascendente; por ejemplo, 1, 3, 5, 7.

4. Si usa los modos de discretización **Cuantiles** y **Mismo ancho**, use la opción **Número de discretizaciones** para especificar el número de rangos, o de *cuantiles*, que desea crear.

5. En **Columnas para discretizar**, use el selector de columnas para elegir las columnas que tienen los valores que desea discretizar. Las columnas deben tener un tipo de datos numéricos.

    La misma regla de cuantificación se aplica a todas las columnas que elija. Por tanto, si necesita discretizar columnas con un método distinto, use una instancia independiente del módulo de agrupación de datos en rangos para cada conjunto de columnas.

    > [!WARNING]
    > Si elige una columna que no es un tipo permitido, se genera un error en runtime. El módulo devuelve un error en cuanto encuentra cualquier columna de un tipo no permitido. Si comete un error, revise todas las columnas seleccionadas. El error no muestra todas las columnas no válidas.

6. En **Modo de salida**, indique cómo desea que se generen los valores cuantificados:

    + **Anexión**: crea una nueva columna con los valores discretizados y se anexa a la tabla de entrada.

    + **Inplace**: Reemplaza los valores originales por los nuevos valores en el conjunto de datos.

    + **ResultOnly**: Devuelve solo las columnas de resultados.

7. Si selecciona el modo de cuantificación **Cuantiles**, use la opción **Normalización de cuantiles** para determinar cómo se normalizan los valores antes de la ordenación en cuantiles. Tenga en cuenta que la normalización de los valores transforma los valores, pero no afecta al número final de rangos.

    Se admiten los siguientes tipos de normalización:

    + **Percent**: los valores se normalizan en el rango [0,100].

    + **PQuantile**: los valores se normalizan en el rango [0,1].

    + **QuantileIndex**:  los valores se normalizan en el rango [1, número de rangos].

8. Si elige la opción **Bordes personalizados**, escriba una lista separada por comas de números que se usará como *extremo del rango* en el cuadro de texto de la **lista separada por comas de los extremos del rango**. 

    Los valores marcan el punto que divide los rangos. Por ejemplo, si escribe un valor de extremo del intervalo, se generarán dos rangos. Si escribe dos valores de extremo del rango, se generarán tres rangos.

    Los valores se deben ordenar en función del orden de creación de los rangos, de menor a mayor.

10. Seleccione la opción **Columnas de etiqueta como de categorías** para indicar que las columnas cuantificadas se deben controlar como variables de categorías.

11. Envíe la canalización.

## <a name="results"></a>Results

El módulo de agrupación de datos en rangos devuelve un conjunto de datos donde cada elemento se discretiza según el modo especificado. 

También devuelve una *transformación de discretización*. Esa función puede pasarse al módulo [Aplicar transformación](apply-transformation.md) para discretizar nuevos ejemplos de datos usando los mismos parámetros y el mismo modo de discretización.  

> [!TIP]
> Si usa la discretización en los datos de entrenamiento, debe usar el mismo método de discretización en los datos que usa para las pruebas y la predicción. También debe usar las mismas ubicaciones y anchos de rango. 
> 
> Para asegurarse de que los datos se transformen siempre con el mismo método de discretización, se recomienda guardar transformaciones de datos útiles. A continuación, aplíquelos a otros conjuntos de valores usando el módulo [Aplicar transformación](apply-transformation.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
