---
title: 'Split Data: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Dividir datos en Azure Machine Learning para dividir un conjunto de datos en dos conjuntos distintos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/22/2019
ms.openlocfilehash: a4c93b12ad654e54a7f3c7ee0e75507d5cb45e90
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907813"
---
# <a name="split-data-module"></a>Módulo Split Data

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Use el módulo Split Data (Dividir datos) para dividir un conjunto de datos en dos conjuntos distintos.

Este módulo es especialmente útil cuando necesita separar los datos en conjuntos de entrenamiento y prueba. Además, puede personalizar la forma en que se dividen los datos. Algunas opciones admiten la selección aleatoria de datos. Otras se adaptan a un tipo de datos o tipo de modelo determinado.

## <a name="configure-the-module"></a>Configuración del módulo

> [!TIP]
> Antes de elegir el modo de división, lea todas las opciones para determinar el tipo de división que necesita.
> Si cambia el modo de división, se podrían restablecer todas las demás opciones.

1. Agregue el módulo **Split Data** a la canalización en el diseñador. Puede encontrar este módulo en **Transformación de datos**, en la categoría **Sample and Split** (Muestrear y dividir).

1. **Splitting mode** (Modo de división): elija uno de los modos siguientes, según el tipo de datos que tenga y cómo quiera dividirlos. Cada modo de división tiene distintas opciones.

   - **Split Rows** (Dividir filas): Use esta opción si quiere dividir los datos en dos partes. Puede especificar el porcentaje de datos que se van a colocar en cada división. De forma predeterminada, los datos se dividen en la proporción 50/50.

     También puede hacer la selección de filas de cada grupo de forma aleatoria y usar muestreo estratificado. En el muestreo estratificado, debe seleccionar una sola columna de datos para la que quiera que los valores se distribuyan por igual entre los dos conjuntos de datos de resultados.  

   - **Regular Expression Split** (División de expresión regular): elija esta opción cuando quiera dividir el conjunto de datos probando una sola columna para un valor.

     Por ejemplo, si analiza la opinión, puede comprobar la presencia del nombre de producto determinado en un campo de texto. Luego, puede dividir el conjunto de datos en filas con y sin el nombre del producto de destino.

   - **Relative Expression Split** (División de expresión relativa): Use esta opción siempre que quiera aplicar una condición a una columna de números. El número podría ser un campo de fecha y hora, una columna que contenga edades o importes monetarios, o incluso un porcentaje. Por ejemplo, podría dividir el conjunto de datos según el costo de los artículos, agrupar personas por intervalos de edad o separar datos por una fecha del calendario.

### <a name="split-rows"></a>División de filas

1. Agregue el módulo [Split Data](./split-data.md) (Dividir datos) a la canalización en el diseñador y conecte el conjunto de datos que quiera dividir.
  
1. En **Splitting mode** (Modo de división), seleccione **Split Rows** (Dividir filas). 

1. **Fraction of rows in the first output dataset** (Fracción de filas del primer conjunto de datos de salida): use esta opción para determinar cuántas filas irán en la primera salida (izquierda). Todas las demás filas irán en la segunda salida (derecha).

   El coeficiente representa el porcentaje de filas que se envían al primer conjunto de datos de salida, por lo que debe escribir un número decimal comprendido entre 0 y 1.
     
   Por ejemplo, si escribe **0,75** como valor, el conjunto de filas se dividirá en la proporción 75/25. En esta división, el 75 por ciento de las filas se enviarán al primer conjunto de datos de salida. El 25 por ciento restante se enviará al segundo conjunto de datos de salida.
  
1. Seleccione la opción **Randomized split** (División aleatoria) si quiere realizar la selección de datos en los dos grupos de forma aleatoria. Esta es la opción preferida cuando se crean conjuntos de datos de entrenamiento y prueba.

1. **Valor de inicialización aleatorio**: escriba un valor entero no negativo para iniciar la secuencia seudoaleatoria de instancias que se usará. Este valor de inicialización predeterminado se usa en todos los módulos que generan números aleatorios. 

   Especificar un valor de inicialización hace que los resultados sean reproducibles. Si tiene que repetir los resultados de una operación de división, debe especificar un valor de inicialización para el generador de números aleatorios. De lo contrario, el valor de inicialización aleatorio se establece de forma predeterminada en **0**, lo que significa que se obtiene del reloj del sistema. Como consecuencia, la distribución de datos puede ser ligeramente diferente cada vez que haga una división. 

1. **Stratified split** (División estratificada): Establezca esta opción en **True** para garantizar que los dos conjuntos de datos de resultados contengan una muestra representativa de los valores de la *columna de estratos* o la *columna de clave de estratificación*. 

   Con el muestreo estratificado, los datos se dividen de forma que cada conjunto de datos de salida obtiene aproximadamente el mismo porcentaje de cada valor de destino. Por ejemplo, es posible que quiera asegurarse de que sus conjuntos de entrenamiento y prueba estén más o menos compensados en relación con el resultado, o con respecto a alguna otra columna (como el género).

1. Envíe la canalización.


## <a name="select-a-regular-expression"></a>Selección de una expresión regular

1. Agregue el módulo [Split Data](./split-data.md) (Dividir datos) a la canalización y conéctelo como entrada al conjunto de datos que quiera dividir.  
  
1. Para **Splitting mode**, seleccione **Regular expression split**.

1. En el cuadro **Regular expression**(Expresión regular), escriba una expresión regular válida. 
  
   La expresión regular debe seguir la sintaxis de expresiones regulares de Python.

1. Envíe la canalización.

   En función de la expresión regular que proporcione, el conjunto de datos se divide en dos conjuntos de filas: filas con valores que coinciden con la expresión y todas las filas restantes. 

En los siguientes ejemplos se muestra cómo dividir un conjunto de datos mediante la opción **Regular expression** (Expresión regular). 

### <a name="single-whole-word"></a>Palabra completa única 

En este ejemplo todas las filas que contienen el texto `Gryphon` en la columna `Text` se colocan en el primer conjunto de datos. El resto de las filas van a la segunda salida de **Split Data** (Dividir datos).

```text
    \"Text" Gryphon  
```

### <a name="substring"></a>Substring

En este ejemplo se busca la cadena especificada en cualquier posición dentro de la segunda columna del conjunto de datos. La posición se indica aquí por el valor de índice de 1. La coincidencia distingue entre mayúsculas y minúsculas.

```text
(\1) ^[a-f]
```

El primer conjunto de datos de resultados contiene todas las filas en las que la columna de índice comienza con uno de estos caracteres: `a`, `b`, `c`, `d`, `e`, `f`. Todas las demás filas se dirigen a la segunda salida.

## <a name="select-a-relative-expression"></a>Selección de una expresión relativa

1. Agregue el módulo [Split Data](./split-data.md) (Dividir datos) a la canalización y conéctelo como entrada al conjunto de datos que quiera dividir.
  
1. En **Splitting mode** (Modo de división), seleccione **Relative Expression** (Expresión relativa).
  
1. En el cuadro de texto **Relational expression** (Expresión relacional), escriba una expresión que realice una operación de comparación en una sola columna.

   En **Numeric column** (Columna numérica):
   - La columna contiene números de cualquier tipo de datos numéricos, incluidos los tipos de datos de fecha y hora.
   - La expresión puede hacer referencia a un máximo de un nombre de columna.
   - Use el carácter de "Y" comercial `&` para la operación AND. Use el carácter de barra vertical `|` para la operación OR.
   - Se admiten los siguientes operadores: `<`, `>`, `<=`, `>=`, `==`, `!=`.
   - No puede agrupar las operaciones con `(` y `)`.
   
   En **String column** (Columna de cadena):
   - Se admiten los siguientes operadores: `==`, `!=`.

1. Envíe la canalización.

   La expresión divide el conjunto de datos en dos conjuntos de filas: las filas con valores que cumplen la condición y todas las filas restantes.

En los siguientes ejemplos se muestra cómo dividir un conjunto de datos mediante la opción **Relative Expression** (Expresión relativa) del módulo **Split Data** (Dividir datos).  

### <a name="calendar-year"></a>Año del calendario

Un escenario común consiste en dividir un conjunto de datos por años. La expresión siguiente selecciona todas las filas donde los valores de la columna `Year` son mayores que `2010`.

```text
\"Year" > 2010
```

La expresión de fecha debe tener en cuenta todas las partes de fecha que se incluyen en la columna de datos. El formato de las fechas en la columna de datos debe ser coherente. 

Por ejemplo, en una columna de fecha con el formato `mmddyyyy`, la expresión debe ser similar a la siguiente:

```text
\"Date" > 1/1/2010
```

### <a name="column-index"></a>Índice de columna

La siguiente expresión muestra cómo puede utilizar el índice de columna para seleccionar todas las filas de la primera columna del conjunto de datos que contienen valores menores o iguales a 30, pero no iguales a 20.

```text
(\0)<=30 & !=20
```


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
