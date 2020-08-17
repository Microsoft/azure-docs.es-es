---
title: Creación de funciones de conversión de definición de interfaz de usuario
description: Describe las funciones que se van a usar al convertir valores entre los tipos de datos y las codificaciones.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: b69cd35b27b343da08727b4c4ee9b4fd025e1df7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095213"
---
# <a name="createuidefinition-conversion-functions"></a>Funciones de conversión CreateUiDefinition

Estas funciones se pueden usar para convertir valores entre tipos de datos JSON y codificaciones.

## <a name="bool"></a>bool

Convierte el parámetro en un booleano. Esta función admite parámetros de tipo number, string y booleano. Similar a los booleanos en JavaScript, cualquier valor excepto `0` o `'false'` devuelve `true`.

El siguiente ejemplo devuelve `true`:

```json
"[bool(1)]"
```

El siguiente ejemplo devuelve `false`:

```json
"[bool(0)]"
```

El siguiente ejemplo devuelve `true`:

```json
"[bool(true)]"
```

El siguiente ejemplo devuelve `true`:

```json
"[bool('true')]"
```

## <a name="decodebase64"></a>decodeBase64

Decodifica el parámetro de una cadena de codificación Base 64. Esta función admite parámetros solo de tipo string.

El siguiente ejemplo devuelve `"Contoso"`:

```json
"[decodeBase64('Q29udG9zbw==')]"
```

## <a name="decodeuricomponent"></a>decodeUriComponent

Decodifica el parámetro de una cadena de codificación con dirección URL. Esta función admite parámetros solo de tipo string.

El siguiente ejemplo devuelve `"https://portal.azure.com/"`:

```json
"[decodeUriComponent('https%3A%2F%2Fportal.azure.com%2F')]"
```

## <a name="encodebase64"></a>encodeBase64

Codifica el parámetro a una cadena de codificación Base 64. Esta función admite parámetros solo de tipo string.

El siguiente ejemplo devuelve `"Q29udG9zbw=="`:

```json
"[encodeBase64('Contoso')]"
```

## <a name="encodeuricomponent"></a>encodeUriComponent

Codifica el parámetro a una cadena de codificación con dirección URL. Esta función admite parámetros solo de tipo string.

El siguiente ejemplo devuelve `"https%3A%2F%2Fportal.azure.com%2F"`:

```json
"[encodeUriComponent('https://portal.azure.com/')]"
```

## <a name="float"></a>FLOAT

Convierte el parámetro en un número de punto flotante. Esta función admite parámetros de tipo number y string.

El siguiente ejemplo devuelve `1.0`:

```json
"[float('1.0')]"
```

El siguiente ejemplo devuelve `2.9`:

```json
"[float(2.9)]"
```

## <a name="int"></a>int

Convierte el parámetro en un número entero. Esta función admite parámetros de tipo number y string.

El siguiente ejemplo devuelve `1`:

```json
"[int('1')]"
```

El siguiente ejemplo devuelve `2`:

```json
"[int(2.9)]"
```

## <a name="parse"></a>parse

Convierte el parámetro en un tipo nativo. En otras palabras, esta función es el inverso de `string()`. Esta función admite parámetros solo de tipo string.

El siguiente ejemplo devuelve `1`:

```json
"[parse('1')]"
```

El siguiente ejemplo devuelve `true`:

```json
"[parse('true')]"
```

El siguiente ejemplo devuelve `[1,2,3]`:

```json
"[parse('[1,2,3]')]"
```

El siguiente ejemplo devuelve `{"type":"webapp"}`:

```json
"[parse('{\"type\":\"webapp\"}')]"
```

## <a name="string"></a>string

Convierte el parámetro en una cadena. Esta función admite parámetros de todos los tipos de datos JSON.

El siguiente ejemplo devuelve `"1"`:

```json
"[string(1)]"
```

El siguiente ejemplo devuelve `"2.9"`:

```json
"[string(2.9)]"
```

El siguiente ejemplo devuelve `"[1,2,3]"`:

```json
"[string([1,2,3])]"
```

El siguiente ejemplo devuelve `"{"type":"webapp"}"`:

```json
"[string({\"type\":\"webapp\"})]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).
