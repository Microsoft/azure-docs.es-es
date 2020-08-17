---
title: Expresiones de estilo basadas en datos en el SDK web de Azure Maps | Microsoft Azure Maps
description: En este artículo, aprenderá a usar expresiones de estilo basadas en datos en el SDK web de Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 4/4/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendleton
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 54477bd74df660edb12f6daffbaa2a7390f9516a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285720"
---
# <a name="data-driven-style-expressions-web-sdk"></a>Expresiones de estilo basadas en datos (SDK web)

Las expresiones permiten aplicar lógica de negocios a las opciones de estilo que respetan las propiedades definidas en cada forma de un origen de datos. Las expresiones pueden filtrar los datos de un origen o una capa de datos. Pueden contener lógica condicional, como instrucciones if. Además, se pueden usar para manipular los datos mediante operadores de cadena, operadores lógicos y operadores matemáticos.

Los estilos basados en datos reducen la cantidad de código necesario para implementar la lógica de negocios relacionada con los estilos. Cuando se usan con capas, las expresiones se evalúan en tiempo de representación en un subproceso independiente. Esta funcionalidad proporciona mayor rendimiento que cuando la lógica de negocios se evalúa en el subproceso de la interfaz de usuario.

En este vídeo, encontrará información general sobre la aplicación de estilos basados en datos en el SDK web de Azure Maps.

<br/>

<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Data-Driven-Styling-with-Azure-Maps/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>

Las expresiones se representan como matrices JSON. El primer elemento de una expresión de la matriz es una cadena que especifica el nombre del operador de expresión. Por ejemplo, "+" o "case". Los elementos siguientes (si existen) son los argumentos para la expresión. Cada argumento es un valor literal (cadena, número, booleano o `null`) u otra matriz de expresiones. El pseudocódigo siguiente define la estructura básica de una expresión. 

```javascript
[ 
    expression_operator, 
    argument0, 
    argument1, 
    …
] 
```

El SDK web de Azure Maps admite muchos tipos de expresiones. Las expresiones se pueden utilizar solas o junto con otras expresiones.

| Tipo de expresiones | Descripción |
|---------------------|-------------|
| [Expresión de agregado](#aggregate-expression) | Expresión que define un cálculo que se procesa en un conjunto de datos y que se puede utilizar con la opción `clusterProperties` de `DataSource`. |
| [Expresiones booleanas](#boolean-expressions) | Las expresiones booleanas proporcionan un conjunto de expresiones de operadores booleanos para evaluar las comparaciones booleanas. |
| [Expresiones de color](#color-expressions) | Las expresiones de color facilitan la creación y la manipulación de valores de color. |
| [Expresiones condicionales](#conditional-expressions) | Las expresiones condicionales proporcionan operaciones lógicas que son parecidas a las instrucciones if. |
| [Expresiones de datos](#data-expressions) | Proporcionan acceso a los datos de propiedad de una característica. |
| [Expresiones de interpolación y paso](#interpolate-and-step-expressions) | Se pueden usar para calcular valores a lo largo de una curva interpolada o una función de paso. |
| [Expresiones específicas de capa](#layer-specific-expressions) | Expresiones especiales que solo son aplicables a una capa. |
| [Expresiones matemáticas](#math-expressions) | Proporcionan operadores matemáticos para llevar a cabo cálculos basados en datos dentro del marco de la expresión. |
| [Expresiones de operador de cadena](#string-operator-expressions) | Las expresiones de operador de cadena realizan operaciones de conversión sobre cadenas, como concatenación y conversión de las mayúsculas y minúsculas. |
| [Expresiones de tipo](#type-expressions) | Las expresiones de tipo proporcionan herramientas para probar y convertir distintos tipos de datos, como cadenas, números y valores booleanos. |
| [Expresiones de enlace de variable](#variable-binding-expressions) | Las expresiones de enlace de variables almacenan los resultados de un cálculo en una variable y permiten utilizar referencias en otra parte de una expresión varias veces sin tener que volver a calcular el valor almacenado. |
| [Expresión de zoom](#zoom-expression) | Recupera el nivel de zoom actual del mapa en tiempo de representación. |

En todos los ejemplos de este documento, se utiliza la siguiente característica para mostrar diferentes maneras en que se pueden usar los diversos tipos de expresiones. 

```javascript
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.13284, 47.63699]
    },
    "properties": { 
        "id": 123,
        "entityType": "restaurant",
        "revenue": 12345,
        "subTitle": "Building 40", 
        "temperature": 72,
        "title": "Cafeteria", 
        "zoneColor": "red"
    }
}
```

## <a name="data-expressions"></a>Expresiones de datos

Las expresiones de datos proporcionan acceso a los datos de propiedad de una característica. 

| Expression | Tipo de valor devuelto | Descripción |
|------------|-------------|-------------|
| `['at', number, array]` | object | Recupera un elemento de una matriz. |
| `['geometry-type']` | string | Obtiene el tipo de geometría de la característica: Point, MultiPoint, LineString, MultiLineString, Polygon, MultiPolygon. |
| `['get', string]` | value | Obtiene el valor de propiedad de las propiedades de la característica actual. Devuelve null si no se encuentra la propiedad solicitada. |
| `['get', string, object]` | value | Obtiene el valor de propiedad de las propiedades del objeto proporcionado. Devuelve null si no se encuentra la propiedad solicitada. |
| `['has', string]` | boolean | Determina si las propiedades de una característica tienen la propiedad especificada. |
| `['has', string, object]` | boolean | Determina si las propiedades del objeto tienen la propiedad especificada. |
| `['id']` | value | Obtiene el identificador de la característica si lo tiene. |
| `['length', string | array]` | number | Obtiene la longitud de una cadena o matriz. |
| `['in', boolean | string | number, array]` | boolean | Determina si un elemento existe en una matriz. |
| `['in', substring, string]` | boolean | Determina si existe una subcadena en una cadena. |

**Ejemplos**

Se puede acceder directamente a las propiedades de una característica en una expresión mediante una expresión `get`. En este ejemplo, el valor de "zoneColor" de la característica se utiliza para especificar la propiedad de color de una capa de burbujas. 

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: ['get', 'zoneColor'] //Get the zoneColor value.
});
```

El ejemplo anterior funcionará bien si todas las características de punto tienen la propiedad `zoneColor`. De lo contrario, es probable que se utilice el color de reserva "black". Si desea modificar el color de reserva, utilice una expresión `case` junto con la expresión `has` para comprobar si existe la propiedad. Si la propiedad no existe, se devolverá un color de reserva.

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'case', //Use a conditional case expression.

        ['has', 'zoneColor'],   //Check to see if feature has a "zoneColor" property
        ['get', 'zoneColor'],   //If it does, use it.

        'blue'  //If it doesn't, default to blue.
    ]
});
```

De forma predeterminada, las capas de burbujas y símbolos representarán las coordenadas de todas las formas de un origen de datos. Este comportamiento puede resaltar los vértices de una línea o un polígono. La opción `filter` de la capa puede emplearse para limitar el tipo de geometría de las características que representa, utilizando una expresión `['geometry-type']` dentro de una expresión booleana. En el siguiente ejemplo se limita una capa de burbujas para que solo se representen las características `Point`.

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    filter: ['==', ['geometry-type'], 'Point']
});
```

El siguiente ejemplo permite que se representen las características `Point` y `MultiPoint`. 

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    filter: ['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]
});
```

Igualmente, el contorno de los polígonos se representará en capas de línea. Para deshabilitar este comportamiento en una capa de línea, agregue un filtro que solo permita las características `LineString` y `MultiLineString`.  

## <a name="math-expressions"></a>Expresiones matemáticas

Las expresiones matemáticas proporcionan operadores matemáticos para llevar a cabo cálculos basados en datos dentro del marco de la expresión.

| Expression | Tipo de valor devuelto | Descripción |
|------------|-------------|-------------|
| `['+', number, number, …]` | number | Calcula la suma de los números especificados. |
| `['-', number]` | number | Resta 0 al número especificado. |
| `['-', number, number]` | number | Resta las primeras cifras al segundo número. |
| `['*', number, number, …]` | number | Multiplica los números especificados juntos. |
| `['/', number, number]` | number | Divide el primer número entre el segundo. |
| `['%', number, number]` | number | Calcula el resto al dividir el primer número entre el segundo. |
| `['^', number, number]` | number | Calcula el valor del primer valor elevado a la potencia del segundo número. |
| `['abs', number]` | number | Calcula el valor absoluto del número especificado. |
| `['acos', number]` | number | Calcula el arcocoseno del número especificado. |
| `['asin', number]` | number | Calcula el arcoseno del número especificado. |
| `['atan', number]` | number | Calcula el arcotangente del número especificado. |
| `['ceil', number]` | number | Redondea el número al siguiente número entero. |
| `['cos', number]` | number | Calcula el coseno del número especificado. |
| `['e']` | number | Devuelve la constante matemática `e`. |
| `['floor', number]` | number | Redondea el número hacia abajo al número entero anterior. |
| `['ln', number]` | number | Calcula el logaritmo natural del número especificado. |
| `['ln2']` | number | Devuelve la constante matemática `ln(2)`. |
| `['log10', number]` | number | Calcula el logaritmo en base 10 del número especificado. |
| `['log2', number]` | number | Calcula el logaritmo en base dos del número especificado. |
| `['max', number, number, …]` | number | Calcula el número máximo en el conjunto especificado de números. |
| `['min', number, number, …]` | number | Calcula el número mínimo en el conjunto especificado de números. |
| `['pi']` | number | Devuelve la constante matemática `PI`. |
| `['round', number]` | number | Redondea el número al entero más cercano. Los valores a medio camino se redondean al valor más alejado de cero. Por ejemplo, `['round', -1.5]` se evalúa como -2. |
| `['sin', number]` | number | Calcula el seno del número especificado. |
| `['sqrt', number]` | number | Calcula la raíz cuadrada del número especificado. |
| `['tan', number]` | number | Calcula la tangente del número especificado. |

## <a name="aggregate-expression"></a>Expresión de agregado

Una expresión de agregado define un cálculo que se procesa en un conjunto de datos y que se puede utilizar con la opción `clusterProperties` de `DataSource`. La salida de estas expresiones debe ser un número o un valor booleano. 

Una expresión de agregado toma tres valores: un valor de operador, un valor inicial y una expresión para recuperar una propiedad de cada característica de los datos en los que se va a aplicar esta operación. La expresión tiene el siguiente formato:

```javascript
[operator: string, initialValue: boolean | number, mapExpression: Expression]
```

- operator: función de la expresión que se aplica después a todos los valores calculados por `mapExpression` en cada punto del clúster. Operadores admitidos: 
    - Para números `+`: `*`, `max`, `min`
    - Para valores booleanos: `all`, `any`
- initialValue: un valor inicial al que se agrega el primer valor calculado.
- mapExpression: expresión que se aplica a cada punto del conjunto de datos.

**Ejemplos**

Si todas las características de un conjunto de datos tienen una propiedad `revenue`, que es un número, se pueden calcular los ingresos totales de todos los puntos de un clúster, que se crean a partir del conjunto de datos. Este cálculo se realiza mediante la siguiente expresión de agregado: `['+', 0, ['get', 'revenue']]`

## <a name="boolean-expressions"></a>Expresiones booleanas

Las expresiones booleanas proporcionan un conjunto de expresiones de operadores booleanos para evaluar las comparaciones booleanas.

Cuando se comparan valores, la comparación debe estar estrictamente tipada. Los valores de distintos tipos siempre se consideran desiguales. Los casos en los que se sabe que los tipos son diferentes en tiempo de análisis se consideran no válidos y producirán un error de análisis. 

| Expression | Tipo de valor devuelto | Descripción |
|------------|-------------|-------------|
| `['! ', boolean]` | boolean | Negación lógica. Devuelve `true` si la entrada es `false`, y `false` si la entrada es `true`. |
| `['!= ', value, value]` | boolean | Devuelve `true` si los valores de entrada no son iguales; de lo contrario, `false`. |
| `['<', value, value]` | boolean | Devuelve `true` si la primera entrada es estrictamente menor que el segundo; de lo contrario, `false`. Es necesario que los argumentos sean ambos cadenas o ambos números. |
| `['<=', value, value]` | boolean | Devuelve `true` si la primera entrada es menor o igual que el segundo; de lo contrario, `false`. Es necesario que los argumentos sean ambos cadenas o ambos números. |
| `['==', value, value]` | boolean | Devuelve `true` si los valores de entrada son iguales; de lo contrario `false`. Es necesario que los argumentos sean ambos cadenas o ambos números. |
| `['>', value, value]` | boolean | Devuelve `true` si la primera entrada es estrictamente mayor que la segunda; de lo contrario, `false`. Es necesario que los argumentos sean ambos cadenas o ambos números. |
| `['>=' value, value]` | boolean | Devuelve `true` si la primera entrada es mayor o igual que la segunda; de lo contrario `false`. Es necesario que los argumentos sean ambos cadenas o ambos números. |
| `['all', boolean, boolean, …]` | boolean | Devuelve `true` si todas las entradas son `true`; de lo contrario, `false`. |
| `['any', boolean, boolean, …]` | boolean | Devuelve `true` si cualquiera de las entradas es `true`; de lo contrario, `false`. |

## <a name="conditional-expressions"></a>Expresiones condicionales

Las expresiones condicionales proporcionan operaciones lógicas que son parecidas a las instrucciones if.

Las expresiones siguientes realizan operaciones de lógica condicional sobre los datos de entrada. Por ejemplo, la expresión `case` proporciona la lógica "if/then/else" mientras que la expresión `match` es como una "instrucción swtich". 

### <a name="case-expression"></a>Expresión case

Una expresión `case` es un tipo de expresión condicional que proporciona la lógica "if/then/else". Este tipo de expresión recorre paso a paso una lista de condiciones booleanas. Devuelve el valor de salida de la primera condición booleana que se evalúa como true.

El pseudocódigo siguiente define la estructura de la expresión `case`. 

```javascript
[
    'case',
    condition1: boolean, 
    output1: value,
    condition2: boolean, 
    output2: value,
    ...,
    fallback: value
]
```

**Ejemplo**

En el ejemplo siguiente se recorren diferentes condiciones booleanas hasta encontrar una que se evalúe como `true` y, luego, se devuelve ese valor asociado. Si ninguna condición booleana se evalúa como `true`, se devolverá un valor de retroceso. 

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'case',

        //Check to see if the first boolean expression is true, and if it is, return its assigned result.
        ['has', 'zoneColor'],
        ['get', 'zoneColor'],

        //Check to see if the second boolean expression is true, and if it is, return its assigned result.
        ['all', ['has', ' temperature '], ['>', ['get', 'temperature'], 100]],
        'red',

        //Specify a default value to return.
        'green'
    ]
});
```

### <a name="match-expression"></a>Expresión match

Una expresión `match` es un tipo de expresión condicional que proporciona la instrucción switch como lógica. La entrada puede ser cualquier expresión, como `['get', 'entityType']` que devuelve una cadena o un número. Cada etiqueta debe ser un único valor literal o una matriz de valores literales, cuyos valores deben ser todos cadenas o todos números. La entrada coincide si coincide alguno de los valores de la matriz. Cada etiqueta debe ser única. Si el tipo de entrada no coincide con el tipo de las etiquetas, el resultado será el valor de retroceso.

El pseudocódigo siguiente define la estructura de la expresión `match`. 

```javascript
[
    'match',
    input: number | string,
    label1: number | string | (number | string)[], 
    output1: value,
    label2: number | string | (number | string)[], 
    output2: value,
    ...,
    fallback: value
]
```

**Ejemplos**

En el ejemplo siguiente se examina la propiedad `entityType` de una característica de punto en una capa de burbuja que busca una coincidencia. Si se encuentra una coincidencia, se devuelve ese valor especificado o se devuelve el valor de retroceso.

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'match',

        //Get the property to match.
        ['get', 'entityType'],

        //List the values to match and the result to return for each match.
        'restaurant', 'red',
        'park', 'green',

        //Specify a default value to return if no match is found.
        'black'
    ]
});
```

En el ejemplo siguiente se usa una matriz para enumerar un conjunto de etiquetas que deben devolver el mismo valor. Este enfoque es mucho más eficiente que enumerar cada etiqueta por separado. En este caso, si la propiedad `entityType` es "restaurant" o "grocery_store", se devolverá el color "red".

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'match',

        //Get the property to match.
        ['get', 'entityType'],

        //List the values to match and the result to return for each match.
        ['restaurant', 'grocery_store'], 'red',

        'park', 'green',

        //Specify a default value to return if no match is found.
        'black'
    ]
});
```

En el ejemplo siguiente, se usa una expresión de coincidencia para realizar un filtro de tipo "in array" (en la matriz) o "array contains" (la matriz contiene). En este caso, la expresión filtra los datos cuyo valor de identificador se encuentra en una lista de identificadores permitidos. Cuando se usan expresiones con filtros, el resultado debe ser un valor booleano.

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    filter: [
        'match',  

        //Get the property to match.
        ['get', 'id'],  

         //List of values to match.
        [24, 53, 98], 

        //If there is a match, return true.
        true,
    
        //Otherwise return false.
        false
    ]
});
```

### <a name="coalesce-expression"></a>Expresión coalesce

Una expresión `coalesce` recorre un conjunto de expresiones hasta que se obtiene el primer valor distinto de null y devuelve ese valor. 

El pseudocódigo siguiente define la estructura de la expresión `coalesce`. 

```javascript
[
    'coalesce', 
    value1, 
    value2, 
    …
]
```

**Ejemplo**

En el ejemplo siguiente se usa una expresión `coalesce` para establecer la opción `textField` de una capa de símbolos. Si falta la propiedad `title` de la característica o está establecida en `null`, la expresión intentará buscar la propiedad `subtitle` y, si falta o es `null`, retrocede a una cadena vacía. 

```javascript
var layer = new atlas.layer.SymbolLayer(datasource, null, {
    textOptions: {
        textField: [
            'coalesce',

            //Try getting the title property.
            ['get', 'title'],

            //If there is no title, try getting the subtitle. 
            ['get', 'subtitle'],

            //Default to an empty string.
            ''
        ]
    }
});
```

En el ejemplo siguiente se usa una expresión `coalesce` para recuperar el primer icono de imagen disponible en el objeto del sprite del mapa de una lista de nombres de imagen especificados.

```javascript
var layer = new atlas.layer.SymbolLayer(datasource, null, {
    iconOptions: {
        image: [
            'coalesce',

            //Try getting the image with id 'missing-image'.
            ['image', 'missing-image'],

            //Specify an image id to fallback to. 
            'marker-blue'
        ]
    }
});
``` 

## <a name="type-expressions"></a>Expresiones de tipo

Las expresiones de tipo proporcionan herramientas para probar y convertir distintos tipos de datos, como cadenas, números y valores booleanos.

| Expression | Tipo de valor devuelto | Descripción |
|------------|-------------|-------------|
| `['literal', array]`<br/><br/>`['literal', object]` | matriz \| objeto | Devuelve una matriz literal o un valor de objeto. Use esta expresión para impedir que una matriz o un objeto se evalúen como una expresión. Esto es necesario cuando una matriz o un objeto se deben devolver mediante una expresión. |
| `['image', string]` | string | Comprueba si un identificador de imagen especificado se carga en el sprite de imagen de los mapas. Si es así, se devuelve el identificador. De lo contrario, se devuelve NULL. |
| `['to-boolean', value]` | boolean | Convierte el valor de entrada en un valor booleano. El resultado es `false` cuando la entrada es una cadena vacía, `0`, `false``null` o `NaN`; de lo contrario, es `true`. |
| `['to-color', value]`<br/><br/>`['to-color', value1, value2…]` | color | Convierte el valor de entrada en un color. Si se proporcionan varios valores, cada uno de ellos se evalúa en orden hasta que se obtiene la primera conversión correcta. Si ninguna de las entradas se puede convertir, la expresión es un error. |
| `['to-number', value]`<br/><br/>`['to-number', value1, value2, …]` | number | Convierte el valor de entrada en un número, si es posible. Si la entrada es `null` o `false`, el resultado es 0. Si la entrada es `true`, el resultado es 1. Si la entrada es una cadena, se convierte en un número mediante la función de cadena [ToNumber](https://tc39.github.io/ecma262/#sec-tonumber-applied-to-the-string-type) de la especificación del lenguaje ECMAScript. Si se proporcionan varios valores, cada uno de ellos se evalúa en orden hasta que se obtiene la primera conversión correcta. Si ninguna de las entradas se puede convertir, la expresión es un error. |
| `['to-string', value]` | string | Convierte el valor de entrada en una cadena. Si la entrada es `null`, el resultado es `""`. Si la entrada es un valor booleano, el resultado es `"true"` o `"false"`. Si la entrada es un número, se convierte en una cadena mediante la función numérica [ToString](https://tc39.github.io/ecma262/#sec-tostring-applied-to-the-number-type) de la especificación del lenguaje ECMAScript. Si la entrada es un color, se convierte en la cadena de color `"rgba(r,g,b,a)"` de CSS RGBA. De lo contrario, la entrada se convierte en una cadena mediante la función [JSON.stringify](https://tc39.github.io/ecma262/#sec-json.stringify) de la especificación del lenguaje ECMAScript. |
| `['typeof', value]` | string | Devuelve una cadena que describe el tipo del valor especificado. |

> [!TIP]
> Si aparece un mensaje de error similar a `Expression name must be a string, but found number instead. If you wanted a literal array, use ["literal", [...]].` en la consola del explorador, significa que en algún lugar del código hay una expresión que tiene una matriz cuyo primer valor no contiene una cadena. Si quiere que la expresión para devolver una matriz, encapsule la matriz con la expresión `literal`. En el ejemplo siguiente se establece la opción `offset` de icono de una capa de símbolos, que debe ser una matriz que contenga dos números, mediante una expresión `match` para elegir entre dos valores de desplazamiento en función del valor de la propiedad `entityType` de la característica de punto.
>
> ```javascript
> var layer = new atlas.layer.SymbolLayer(datasource, null, {
>     iconOptions: {
>         offset: [
>             'match',
>
>             //Get the entityType value.
>             ['get', 'entityType'],
>
>             //If the entity type is 'restaurant', return a different pixel offset. 
>             'restaurant', ['literal', [0, -10]],
>
>             //Default to value.
>             ['literal', [0, 0]]
>         ]
>     }
> });
> ```

## <a name="color-expressions"></a>Expresiones de color

Las expresiones de color facilitan la creación y la manipulación de valores de color.

| Expression | Tipo de valor devuelto | Descripción |
|------------|-------------|-------------|
| `['rgb', number, number, number]` | color | Crea un valor de color a partir de los componentes *red*, *green* y *blue* que deben oscilar entre`0` y `255`, y un componente alfa de `1`. Si alguno de los componentes está fuera del intervalo, la expresión es un error. |
| `['rgba', number, number, number, number]` | color | Crea un valor de color a partir de los componentes *red*, *green* y *blue* que deben oscilar entre `0` y `255`, y un componente alfa dentro de un intervalo de `0` y `1`. Si alguno de los componentes está fuera del intervalo, la expresión es un error. |
| `['to-rgba']` | \[número, número, número, número\] | Devuelve una matriz de cuatro elementos que contiene los componentes *red*, *green*, *blue* y *alfa* componentes, en ese orden. |

**Ejemplo**

En el ejemplo siguiente, se crea un valor de color RGB que tiene el valor `255` en *red* y valores *green* y *blue* que se calculan multiplicando `2.5` por el valor de la propiedad `temperature`. A medida que cambia la temperatura, el color cambiará a diferentes tonalidades de *red*.

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'rgb', //Create a RGB color value.

        255,    //Set red value to 255.

        ['*', 2.5, ['get', 'temperature']], //Multiple the temperature by 2.5 and set the green value.

        ['*', 2.5, ['get', 'temperature']]  //Multiple the temperature by 2.5 and set the blue value.
    ]
});
```

## <a name="string-operator-expressions"></a>Expresiones de operador de cadena

Las expresiones de operador de cadena realizan operaciones de conversión sobre cadenas, como concatenación y conversión de las mayúsculas y minúsculas. 

| Expression | Tipo de valor devuelto | Descripción |
|------------|-------------|-------------|
| `['concat', string, string, …]` | string | Concatena varias cadenas juntas. Cada valor debe ser una cadena. Use la expresión de tipo `to-string` para convertir otros tipos de valor en cadena, si es necesario. |
| `['downcase', string]` | string | Convierte la cadena especificada a minúsculas. |
| `['upcase', string]` | string | Convierte la cadena especificada a mayúsculas. |

**Ejemplo**

En el ejemplo siguiente se convierte la propiedad `temperature` de la característica de punto en una cadena y, luego, se concatena "° F" al final de esta.

```javascript
var layer = new atlas.layer.SymbolLayer(datasource, null, {
    textOptions: {
        textField: ['concat', ['to-string', ['get', 'temperature']], '°F'],

        //Some additional style options.
        offset: [0, -1.5],
        size: 12,
        color: 'white'
    }
});
```

La expresión anterior representa una chincheta en el mapa con el texto "64° F" superpuesto sobre ella, como se muestra en la imagen siguiente.

<center>

![Ejemplo de expresión de operador de cadena](media/how-to-expressions/string-operator-expression.png) </center>

## <a name="interpolate-and-step-expressions"></a>Expresiones de interpolación y paso

Se pueden usar para calcular valores a lo largo de una curva interpolada o una función de paso. Estas expresiones toman una expresión que devuelve un valor numérico como entrada, por ejemplo `['get',  'temperature']`. El valor de entrada se evalúa con pares de valores de entrada y salida para determinar el valor que mejor se adapta a la curva interpolada o a la función de paso. Los valores de salida se denominan "paradas". Los valores de entrada de cada parada deben ser un número y estar en orden ascendente. Los valores de salida deben ser un número y una matriz de números, o un color.

### <a name="interpolate-expression"></a>Expresión de interpolación

Una expresión `interpolate` puede usarse para calcular un conjunto continuo y fluido de valores mediante la interpolación entre valores de parada. Una expresión `interpolate` que devuelve valores de color produce un degradado de color del que se seleccionan valores de resultados.

Hay tres tipos de métodos de interpolación que se pueden usar en una expresión `interpolate`:
 
* `['linear']`: interpola linealmente entre el par de paradas.
* `['exponential', base]`: interpola exponencialmente entre las paradas. El valor `base` controla la velocidad a la que aumenta la salida. Los valores más altos hacen que aumente la salida más hacia el extremo superior del intervalo. Un valor `base` cercano a 1 genera una salida que aumenta más linealmente.
* `['cubic-bezier', x1, y1, x2, y2]` interpola mediante una [curva Bézier cúbica](https://developer.mozilla.org/docs/Web/CSS/timing-function) definida por los puntos de control especificados.

Este es un ejemplo del aspecto de estos diferentes tipos de interpolaciones. 

| Lineal  | Exponencial | Bézier cúbica |
|---------|-------------|--------------|
| ![Gráfico de interpolación lineal](media/how-to-expressions/linear-interpolation.png) | ![Gráfico de interpolación exponencial](media/how-to-expressions/exponential-interpolation.png) | ![Gráfico de interpolación de Bézier cúbica](media/how-to-expressions/bezier-curve-interpolation.png) |

El pseudocódigo siguiente define la estructura de la expresión `interpolate`. 

```javascript
[
    'interpolate',
    interpolation: ['linear'] | ['exponential', base] | ['cubic-bezier', x1, y1, x2, y2],
    input: number,
    stopInput1: number, 
    stopOutput1: value1,
    stopInput2: number, 
    stopOutput2: value2, 
    ...
]
```

**Ejemplo**

En el ejemplo siguiente se usa una expresión `linear interpolate` para establecer la propiedad `color` de una capa de burbuja según la propiedad `temperature` de la característica de punto. Si el valor de `temperature` es inferior a 60, se devolverá "blue". Si es mayor que 60 y menor que 70, se devolverá "yellow". Si es mayor que 70 y menor que 80, se devolverá "orange". Si es 80 o superior, se devolverá "red".

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'interpolate',
        ['linear'],
        ['get', 'temperature'],
        50,        
        'blue',
        60,
        'yellow',
        70,
        'orange',
        80,
        'red'
    ]
});
```

En la siguiente imagen se muestra cómo se eligen los colores de la expresión anterior.
 
<center>

![Ejemplo de expresión de interpolación](media/how-to-expressions/interpolate-expression-example.png) </center>

### <a name="step-expression"></a>Expresión de paso

Una expresión `step` puede usarse para calcular valores de resultados escalonados discretos mediante la evaluación de una [función constante a trozos](http://mathworld.wolfram.com/PiecewiseConstantFunction.html) definida por paradas. 

El pseudocódigo siguiente define la estructura de la expresión `step`. 

```javascript
[
    'step',
    input: number,
    output0: value0,
    stop1: number, 
    output1: value1,
    stop2: number, 
    output2: value2, 
    ...
]
```

Las expresiones de paso devuelven el valor de salida de la parada justo antes del valor de entrada, o el primer valor de entrada si la entrada es menor que la primera parada. 

**Ejemplo**

En el ejemplo siguiente se usa una expresión `step` para establecer la propiedad `color` de una capa de burbuja según la propiedad `temperature` de la característica de punto. Si el valor de `temperature` es inferior a 60, se devolverá "blue". Si es mayor que 60 y menor que 70, se devolverá "yellow". Si es mayor que 70 y menor que 80, se devolverá "orange". Si es 80 o superior, se devolverá "red".

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        'step',
        ['get', 'temperature'],
        'blue',
        60,
        'yellow',
        70,
        'orange',
        80,
        'red'
    ]
});
```

En la siguiente imagen se muestra cómo se eligen los colores de la expresión anterior.
 
<center>

![Ejemplo de expresión de paso](media/how-to-expressions/step-expression-example.png)
</center>

## <a name="layer-specific-expressions"></a>Expresiones específicas de una capa

Expresiones especiales que solo se aplican a capas específicas.

### <a name="heat-map-density-expression"></a>Expresión de densidad de mapa térmico

Una expresión de densidad de mapa término recupera el valor de densidad de mapa término para cada píxel de una capa de mapa térmico y se define como `['heatmap-density']`. Este valor es un número comprendido entre `0` y `1`. Se usa en combinación con una expresión `interpolation` o `step` para definir el degradado de color empleado para colorear el mapa térmico. Esta expresión solo se puede usar en la [opción de color](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.heatmaplayeroptions?view=azure-iot-typescript-latest#color) de la capa de mapa térmico.

> [!TIP]
> El color del índice 0 de una expresión de interpolación o el color predeterminado de un paso definen el color del área donde no hay ningún dato. El color del índice 0 se puede usar para definir el color de fondo. Muchos prefieren establecer este valor en transparente o en un negro semitransparente.

**Ejemplo**

En este ejemplo se usa una expresión de interpolación lineal para crear un degradado de color suave para representar el mapa térmico. 

```javascript 
var layer = new atlas.layer.HeatMapLayer(datasource, null, {
    color: [
        'interpolate',
        ['linear'],
        ['heatmap-density'],
        0, 'transparent',
        0.01, 'purple',
        0.5, '#fb00fb',
        1, '#00c3ff'
    ]
});
```

Además de usar un degradado suave para colorear un mapa térmico, se pueden especificar colores dentro de un conjunto de intervalos mediante una expresión `step`. El uso de una expresión `step` para colorear el mapa térmico separa visualmente la densidad en intervalos, lo que recuerda a un mapa de estilo de contorno o radar.  

```javascript 
var layer = new atlas.layer.HeatMapLayer(datasource, null, {
    color: [
        'step',
        ['heatmap-density'],
        'transparent',
        0.01, 'navy',
        0.25, 'navy',
        0.5, 'green',
        0.75, 'yellow',
        1, 'red'
    ]
});
```

Para más información, consulte el artículo [Adición de una capa de mapa térmico](map-add-heat-map-layer.md).

### <a name="line-progress-expression"></a>Expresión de progreso lineal

Una expresión de progreso lineal recupera el progreso a lo largo de una línea de degradado en una capa de línea y se define como `['line-progress']`. Este valor es un número comprendido entre 0 y 1. Se usa en combinación con una expresión `interpolation` o `step`. Esta expresión solo puede usarse con la [opción strokeGradient]( https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.linelayeroptions?view=azure-iot-typescript-latest#strokegradient) de la capa de línea. 

> [!NOTE]
> La opción `strokeGradient` de la capa de línea requiere que la opción `lineMetrics` del origen de datos esté establecida en `true`.

**Ejemplo**

En este ejemplo, se usa la expresión `['line-progress']` para aplicar un degradado de color al trazo de una línea.

```javascript
var layer = new atlas.layer.LineLayer(datasource, null, {
    strokeGradient: [
        'interpolate',
        ['linear'],
        ['line-progress'],
        0, "blue",
        0.1, "royalblue",
        0.3, "cyan",
        0.5, "lime",
        0.7, "yellow",
        1, "red"
    ]
});
```

[Vea un ejemplo en vivo](map-add-line-layer.md#line-stroke-gradient).

### <a name="text-field-format-expression"></a>Expresión de formato de campo de texto

La expresión de formato de campo de texto puede usarse con la opción `textField` de la propiedad `textOptions` de las capas de símbolos para proporcionar formato de texto mixto. Esta expresión permite la especificación de un conjunto de cadenas de entrada y opciones de formato. Las siguientes opciones pueden especificarse para cada cadena de entrada de esta expresión.

 * `'font-scale'`: especifica el factor de escala del tamaño de fuente. Si se especifica, este valor invalidará la propiedad `size` de `textOptions` para la cadena individual.
 * `'text-font'`: especifica una o varias familias de fuentes que se deben usar para esta cadena. Si se especifica, este valor invalidará la propiedad `font` de `textOptions` para la cadena individual.
 * `'text-color'`: especifica el color que se va a aplicar a un texto al representarlo. 

El siguiente pseudocódigo define la estructura de la expresión de formato de campo de texto. 

```javascript
[
    'format', 
    input1: string, 
    options1: { 
        'font-scale': number, 
        'text-font': string[],
        'text-color': color
    },
    input2: string, 
    options2: { 
        'font-scale': number, 
        'text-font': string[] ,
        'text-color': color
    },
    …
]
```

**Ejemplo**

En el ejemplo siguiente se aplica formato al campo de texto, para lo cual se agrega una fuente negrita y se amplía el tamaño de fuente de la propiedad `title` de la característica. En este ejemplo también se agrega la propiedad `subtitle` de la característica en una nueva línea, con un tamaño de fuente reducido y el color rojo.

```javascript
var layer = new atlas.layer.SymbolLayer(datasource, null, {
    textOptions: {
        textField: [
            'format',

            //Bold the title property and scale its font size up.
            ['get', 'title'],
            {
                'text-font': ['literal', ['StandardFont-Bold']],
                'font-scale': 1.25
            },

            '\n', {},   //Add a new line without any formatting.

            //Scale the font size down of the subtitle property. 
            ['get', 'subtitle'],
            { 
                'font-scale': 0.75, 
                'text-color': 'red' 
            }
        ]
    }
});
```

Esta capa representará la característica punto, tal como se muestra en la imagen siguiente:
 
<center>

![Imagen de la característica de punto con el campo de texto con formato](media/how-to-expressions/text-field-format-expression.png) </center>

### <a name="number-format-expression"></a>Expresión de formato de número

La expresión `number-format` solo se puede usar con la opción `textField` de una capa de símbolos. Esta expresión convierte al número proporcionado en una cadena con formato. Esta expresión encapsula la función [Number.toLocalString](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString) de JavaScript y admite el siguiente conjunto de opciones.

 * `locale`: especifique esta opción para convertir números en cadenas de forma que se alineen con el idioma especificado. Pase una [etiqueta de idioma BCP 47](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Intl#Locale_identification_and_negotiation) en esta opción.
 * `currency`: para convertir el número en una cadena que representa una moneda. Los valores posibles son los [códigos de divisa ISO 4217](https://en.wikipedia.org/wiki/ISO_4217), como "USD" para el dólar estadounidense, "EUR" para el euro o "CNY" para el RMB chino.
 * `'min-fraction-digits'`: especifica el número mínimo de posiciones decimales que se incluyen en la versión de cadena del número.
 * `'max-fraction-digits'`: especifica el número máximo de posiciones decimales que se incluyen en la versión de cadena del número.

El siguiente pseudocódigo define la estructura de la expresión de formato de campo de texto. 

```javascript
[
    'number-format', 
    input: number, 
    options: {
        locale: string, 
        currency: string, 
        'min-fraction-digits': number, 
        'max-fraction-digits': number
    }
]
```

**Ejemplo**

En el ejemplo siguiente se usa una expresión `number-format` para modificar cómo se representa la propiedad `revenue` de la característica de punto en la opción `textField` de una capa de símbolos, de forma que aparece un valor de dólar estadounidense.

```javascript
var layer = new atlas.layer.SymbolLayer(datasource, null, {
    textOptions: {
        textField: [
            'number-format', 
            ['get', 'revenue'], 
            { ‘currency': 'USD' }
        ],

        offset: [0, 0.75]
    }
});
```

Esta capa representará la característica punto, tal como se muestra en la imagen siguiente:

<center>

![Ejemplo de expresión de formato de número](media/how-to-expressions/number-format-expression.png) </center>

### <a name="image-expression"></a>Expresión de imagen

Se puede usar una expresión de imagen con las opciones `image` y `textField` de una capa de símbolo y la opción `fillPattern` de la capa de polígono. Esta expresión comprueba que la imagen solicitada existe en el estilo y devuelve el nombre de la imagen resuelta o `null`, en función de si la imagen está actualmente en el estilo o no. Este proceso de validación es sincrónico y requiere que la imagen se haya agregado al estilo antes de solicitarla en el argumento de imagen.

**Ejemplo**

En el ejemplo siguiente se usa una expresión `image` para agregar un icono en línea con texto en una capa de símbolo. 

```javascript
 //Load the custom image icon into the map resources.
map.imageSprite.add('wifi-icon', 'wifi.png').then(function () {

    //Create a data source and add it to the map.
    datasource = new atlas.source.DataSource();
    map.sources.add(datasource);

    //Create a point feature and add it to the data source.
    datasource.add(new atlas.data.Point(map.getCamera().center));

    //Add a layer for rendering point data as symbols.
    map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
        iconOptions: {
            image: 'none'
        },
        textOptions: {
            //Create a formatted text string that has an icon in it.
            textField: ["format", 'Ricky\'s ', ["image", "wifi-icon"], ' Palace']
        }
    }));
});
```

Esta capa representará al campo de texto en la capa de símbolo, tal como se muestra en la imagen siguiente:

<center>

![Ejemplo de expresión de imagen](media/how-to-expressions/image-expression.png) </center>

## <a name="zoom-expression"></a>Expresión de zoom

Una expresión `zoom` se usa para recuperar el nivel de zoom actual del mapa e tiempo de representación y se define como `['zoom']`. Esta expresión devuelve un número entre el intervalo de nivel de zoom mínimo y máximo del mapa. Los controles de mapa interactivo de Azure Maps para web y Android admiten 25 niveles de zoom, numerados de 0 a 24. El uso de la expresión `zoom` permite modificar dinámicamente los estilos a medida que cambia el nivel de zoom del mapa. La expresión `zoom` solo puede usarse con expresiones `interpolate` y `step`.

**Ejemplo**

De forma predeterminada, los radios de los puntos de datos representados en la capa del mapa térmico tienen un radio de píxel fijo para todos los niveles de zoom. A medida que se amplía el mapa, los datos se agregan juntos y la capa de mapa térmico parece diferente. Una expresión `zoom` puede usarse para ampliar el radio de cada nivel de zoom de forma que cada punto de datos cubra el mismo área físico del mapa. Esto hará que la capa del mapa térmico parezca más estática y coherente. Cada nivel de zoom del mapa tiene dos veces tantos píxeles vertical y horizontalmente que nivel de zoom anterior. Si se amplía el radio de forma que se duplique con cada nivel de zoom, se creará un mapa térmico que parecerá coherente en todos los niveles de zoom. Puede realizarse mediante la expresión `zoom` con una expresión `base 2 exponential interpolation`, con el radio de píxel establecido para el nivel mínimo de zoom y un radio escalado para el nivel máximo de zoom calculado como `2 * Math.pow(2, minZoom - maxZoom)`, tal como se muestra a continuación.

```javascript 
var layer = new atlas.layer.HeatMapLayer(datasource, null, {
    radius: [
        'interpolate',
        ['exponential', 2],
        ['zoom'],
        
        //For zoom level 1 set the radius to 2 pixels.
        10, 2,

        //Between zoom level 1 and 19, exponentially scale the radius from 2 pixels to 2 * Math.pow(2, 19 - 1) pixels (524,288 pixels).
        19, 2 * Math.pow(2, 19 - 1)
    ]
};
```

[Vea un ejemplo en vivo](map-add-heat-map-layer.md#consistent-zoomable-heat-map).

## <a name="variable-binding-expressions"></a>Expresiones de enlace de variable

Las expresiones de enlace de variables almacenan los resultados de un cálculo en una variable. De ese modo, se pueden incluir varias veces referencias a esos resultados de cálculo en cualquier parte de una expresión. Esta optimización resulta muy útil para las expresiones en las que intervienen muchos cálculos.

| Expression | Tipo de valor devuelto | Descripción |
|--------------|---------------|--------------|
| \[<br/>&nbsp;&nbsp;&nbsp;&nbsp;'let',<br/>&nbsp;&nbsp;&nbsp;&nbsp;name1: string,<br/>&nbsp;&nbsp;&nbsp;&nbsp;value1: any,<br/>&nbsp;&nbsp;&nbsp;&nbsp;name2: string,<br/>&nbsp;&nbsp;&nbsp;&nbsp;value2: any,<br/>&nbsp;&nbsp;&nbsp;&nbsp;…<br/>&nbsp;&nbsp;&nbsp;&nbsp;childExpression<br/>\] | | Almacena uno o más valores como variables para su uso por la expresión `var` en la expresión secundaria que devuelve el resultado. |
| `['var', name: string]` | cualquiera | Hace referencia a una variable que se creó con la expresión `let`. |

**Ejemplo**

En este ejemplo se usa una expresión que calcula los ingresos relativos a la proporción de temperatura y, luego, usa una expresión `case` para evaluar diferentes operaciones booleanas sobre este valor. La expresión `let` se usa para almacenar los ingresos en relación con el índice de temperatura, de modo que solo debe calcularse una vez. La expresión `var` hace referencia a esta variable tantas veces como sean necesarias sin necesidad de volver a calcularla.

```javascript
var layer = new atlas.layer.BubbleLayer(datasource, null, {
    color: [
        //Divide the point features `revenue` property by the `temperature` property and store it in a variable called `ratio`.
        'let', 'ratio', ['/', ['get', 'revenue'], ['get', 'temperature']],
        //Evaluate the child expression in which the stored variable will be used.
        [
            'case',

            //Check to see if the ratio is less than 100, return 'red'.
            ['<', ['var', 'ratio'], 100],
            'red',

            //Check to see if the ratio is less than 200, return 'green'.
            ['<', ['var', 'ratio'], 200],
            'green',

            //Return `blue` for values greater or equal to 200.
            'blue'
        ]
    ]
});
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para ver más ejemplos de código que implementan expresiones:

> [!div class="nextstepaction"] 
> [Adición de una capa de símbolo](map-add-pin.md)

> [!div class="nextstepaction"] 
> [Adición de una capa de burbuja](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](map-add-shape.md)

> [!div class="nextstepaction"] 
> [Adición de una capa de mapa térmico](map-add-heat-map-layer.md)

Más información sobre las opciones de capa que admiten expresiones:

> [!div class="nextstepaction"] 
> [BubbleLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.bubblelayeroptions?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"] 
> [HeatMapLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.heatmaplayeroptions?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"] 
> [LineLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.linelayeroptions?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"] 
> [PolygonLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.polygonlayeroptions?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"] 
> [SymbolLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.symbollayeroptions?view=azure-iot-typescript-latest)
