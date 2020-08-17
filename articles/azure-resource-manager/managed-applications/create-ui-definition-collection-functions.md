---
title: Creación de funciones de recopilación de definición de interfaz de usuario
description: Describe las funciones que se deben usar al trabajar con colecciones, como matrices y objetos.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 2a075c5c99f457681cd49e75014487bf9cca263c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095214"
---
# <a name="createuidefinition-collection-functions"></a>Funciones de recopilación CreateUiDefinition

Estas funciones se pueden usar con las colecciones, como las cadenas JSON, matrices y objetos.

## <a name="contains"></a>contains

Devuelve `true` si una cadena contiene la subcadena especificada, una matriz contiene el valor especificado o un objeto contiene la clave especificada.

### <a name="example-string-contains"></a>Ejemplo: contenido de string

El siguiente ejemplo devuelve `true`:

```json
"[contains('webapp', 'web')]"
```

### <a name="example-array-contains"></a>Ejemplo: contenido de matriz

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `false`:

```json
"[contains(steps('demoStep').element1, 4)]"
```

### <a name="example-object-contains"></a>Ejemplo: contenido de objeto

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

El siguiente ejemplo devuelve `true`:

```json
"[contains(steps('demoStep').element1, 'key1')]"
```

## <a name="empty"></a>empty

Devuelve `true` si la cadena, la matriz o el objeto es nulo o está vacío.

### <a name="example-string-empty"></a>Ejemplo: string vacía

El siguiente ejemplo devuelve `true`:

```json
"[empty('')]"
```

### <a name="example-array-empty"></a>Ejemplo: matriz vacía

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `false`:

```json
"[empty(steps('demoStep').element1)]"
```

### <a name="example-object-empty"></a>Ejemplo: objeto vacío

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

El siguiente ejemplo devuelve `false`:

```json
"[empty(steps('demoStep').element1)]"
```

### <a name="example-null-and-undefined"></a>Ejemplo: NULL y sin definir

Asuma que `element1` es `null` o no está definida. El siguiente ejemplo devuelve `true`:

```json
"[empty(steps('demoStep').element1)]"
```

## <a name="filter"></a>filter

Devuelve una nueva matriz después de aplicar la lógica de filtrado proporcionada como una función lambda. El primer parámetro es la matriz que se usará para el filtrado. El segundo parámetro es la función lambda que especifica la lógica de filtrado.

En el ejemplo siguiente se devuelve la matriz `[ { "name": "abc" } ]`.

```json
"[filter(parse('[{\"name\":\"abc\"},{\"name\":\"xyz\"}]'), (item) => contains(item.name, 'abc'))]"
```

## <a name="first"></a>first

Devuelve el primer carácter de la cadena especificada, el primer valor de la matriz especificada o la primera clave y valor del objeto especificado.

### <a name="example-string-first"></a>Ejemplo: primer elemento de string

El siguiente ejemplo devuelve `"c"`:

```json
"[first('contoso')]"
```

### <a name="example-array-first"></a>Ejemplo: primer elemento de matriz

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `1`:

```json
"[first(steps('demoStep').element1)]"
```

#### <a name="example-object-first"></a>Ejemplo: primer elemento de objeto

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

El siguiente ejemplo devuelve `{"key1": "Linux"}`:

```json
"[first(steps('demoStep').element1)]"
```

## <a name="last"></a>last

Devuelve el último carácter de la cadena especificada, el último valor de la matriz especificada o la última clave y valor del objeto especificado.

### <a name="example-string-last"></a>Ejemplo: último elemento de string

El siguiente ejemplo devuelve `"o"`:

```json
"[last('contoso')]"
```

### <a name="example-array-last"></a>Ejemplo: último elemento de matriz

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `3`:

```json
"[last(steps('demoStep').element1)]"
```

### <a name="example-object-last"></a>Ejemplo: último elemento de objeto

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

El siguiente ejemplo devuelve `{"key2": "Windows"}`:

```json
"[last(steps('demoStep').element1)]"
```

## <a name="length"></a>length

Devuelve el número de caracteres de una cadena, el número de valores de una matriz o el número de claves en un objeto.

### <a name="example-string-length"></a>Ejemplo: longitud de string

El siguiente ejemplo devuelve `7`:

```json
"[length('Contoso')]"
```

### <a name="example-array-length"></a>Ejemplo: longitud de matriz

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `3`:

```json
"[length(steps('demoStep').element1)]"
```

### <a name="example-object-length"></a>Ejemplo: longitud de objeto

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

El siguiente ejemplo devuelve `2`:

```json
"[length(steps('demoStep').element1)]"
```

## <a name="map"></a>map

Devuelve una nueva matriz después de llamar una función lambda en una matriz específica. El primer parámetro es la matriz que se usará para el la función lambda. El segundo parámetro es la función lambda.

En el ejemplo siguiente se devuelve una nueva matriz con todos los valores duplicados. El resultado es `[2, 4, 6]`.

```json
"[map(parse('[1, 2, 3]'), (item) => mul(2, item))]"
```

En el ejemplo siguiente se devuelve una nueva matriz `["abc", "xyz"]`.

```json
"[map(parse('[{\"name\":\"abc\"},{\"name\":\"xyz\"}]'), (item) => item.name)]"
```

## <a name="skip"></a>skip

Omite un número especificado de elementos de una colección y, a continuación, devuelve los elementos restantes.

### <a name="example-string-skip"></a>Ejemplo: omisión en string

El siguiente ejemplo devuelve `"app"`:

```json
"[skip('webapp', 3)]"
```

### <a name="example-array-skip"></a>Ejemplo: omisión en matriz

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `[3]`:

```json
"[skip(steps('demoStep').element1, 2)]"
```

### <a name="example-object-skip"></a>Ejemplo: omisión en objeto

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```
El siguiente ejemplo devuelve `{"key2": "Windows"}`:

```json
"[skip(steps('demoStep').element1, 1)]"
```

## <a name="split"></a>split

Devuelve una matriz de string que contiene los elementos substring de la entrada delimitada por el separador.

En el ejemplo siguiente se devuelve la matriz `[ "555", "867", "5309" ]`.

```json
"[split('555-867-5309', '-')]"
```

## <a name="take"></a>take

Devuelve un número especificado de caracteres contiguos desde el principio de la cadena, un número especificado de valores contiguos desde el principio de la matriz o un número de claves y valores contiguos desde el principio del objeto especificado.

### <a name="example-string-take"></a>Ejemplo: extracción en string

El siguiente ejemplo devuelve `"web"`:

```json
"[take('webapp', 3)]"
```

### <a name="example-array-take"></a>Ejemplo: extracción en matriz

Asuma que `element1` devuelve `[1, 2, 3]`. El siguiente ejemplo devuelve `[1, 2]`:

```json
"[take(steps('demoStep').element1, 2)]"
```

### <a name="example-object-take"></a>Ejemplo: extracción en objeto

Asuma que `element1` devuelve:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

El siguiente ejemplo devuelve `{"key1": "Linux"}`:

```json
"[take(steps('demoStep').element1, 1)]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).
