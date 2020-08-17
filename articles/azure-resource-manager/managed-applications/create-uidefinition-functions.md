---
title: Crear funciones de definición de interfaz de usuario
description: Describe las funciones que se usarán al crear definiciones de interfaz de usuario para aplicaciones administradas de Azure
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: e4255f0d42e28a72ad55d9b7f81d0dc49b2950cb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040996"
---
# <a name="createuidefinition-functions"></a>Funciones CreateUiDefinition

En este artículo se proporciona información general sobre las funciones admitidas en CreateUiDefinition.

## <a name="function-syntax"></a>Sintaxis de la función

Para usar una función, encierre la invocación entre corchetes. Por ejemplo:

```json
"[function()]"
```

Puede hacerse referencia a cadenas y otras funciones como parámetros para una función, pero las cadenas deben encerrarse entre comillas simples. Por ejemplo:

```json
"[fn1(fn2(), 'demo text')]"
```

Si corresponde, puede hacer referencia a propiedades de la salida de una función mediante el operador de punto. Por ejemplo:

```json
"[func().prop1]"
```

## <a name="collection-functions"></a>Funciones de colección

Estas funciones se pueden usar con las colecciones, como las cadenas JSON, matrices y objetos.

* [contains](create-ui-definition-collection-functions.md#contains)
* [empty](create-ui-definition-collection-functions.md#empty)
* [filter](create-ui-definition-collection-functions.md#filter)
* [first](create-ui-definition-collection-functions.md#first)
* [last](create-ui-definition-collection-functions.md#last)
* [length](create-ui-definition-collection-functions.md#length)
* [map](create-ui-definition-collection-functions.md#map)
* [skip](create-ui-definition-collection-functions.md#skip)
* [split](create-ui-definition-collection-functions.md#split)
* [take](create-ui-definition-collection-functions.md#take)

## <a name="conversion-functions"></a>Funciones de conversión

Estas funciones se pueden usar para convertir valores entre tipos de datos JSON y codificaciones.

* [bool](create-ui-definition-conversion-functions.md#bool)
* [decodeBase64](create-ui-definition-conversion-functions.md#decodebase64)
* [decodeUriComponent](create-ui-definition-conversion-functions.md#decodeuricomponent)
* [encodeBase64](create-ui-definition-conversion-functions.md#encodebase64)
* [encodeUriComponent](create-ui-definition-conversion-functions.md#encodeuricomponent)
* [float](create-ui-definition-conversion-functions.md#float)
* [int](create-ui-definition-conversion-functions.md#int)
* [parse](create-ui-definition-conversion-functions.md#parse)
* [string](create-ui-definition-conversion-functions.md#string)

## <a name="date-functions"></a>Funciones de fecha

* [addHours](create-ui-definition-date-functions.md#addhours)
* [addMinutes](create-ui-definition-date-functions.md#addminutes)
* [addSeconds](create-ui-definition-date-functions.md#addseconds)
* [utcNow](create-ui-definition-date-functions.md#utcnow)

## <a name="logical-functions"></a>Funciones lógicas

Estas funciones se pueden usar en los condicionales. Algunas funciones pueden no admitir todos los tipos de datos JSON.

* [and](create-ui-definition-logical-functions.md#and)
* [coalesce](create-ui-definition-logical-functions.md#coalesce)
* [equals](create-ui-definition-logical-functions.md#equals)
* [greater](create-ui-definition-logical-functions.md#greater)
* [greaterOrEquals](create-ui-definition-logical-functions.md#greaterorequals)
* [if](create-ui-definition-logical-functions.md#if)
* [less](create-ui-definition-logical-functions.md#less)
* [lessOrEquals](create-ui-definition-logical-functions.md#lessorequals)
* [not](create-ui-definition-logical-functions.md#not)
* [or](create-ui-definition-logical-functions.md#or)

## <a name="math-functions"></a>Funciones matemáticas

* [add](create-ui-definition-math-functions.md#add)
* [ceil](create-ui-definition-math-functions.md#ceil)
* [div](create-ui-definition-math-functions.md#div)
* [floor](create-ui-definition-math-functions.md#floor)
* [max](create-ui-definition-math-functions.md#max)
* [min](create-ui-definition-math-functions.md#min)
* [mod](create-ui-definition-math-functions.md#mod)
* [mul](create-ui-definition-math-functions.md#mul)
* [rand](create-ui-definition-math-functions.md#rand)
* [range](create-ui-definition-math-functions.md#range)
* [sub](create-ui-definition-math-functions.md#sub)

## <a name="referencing-functions"></a>Referencias a funciones

* [basics](create-ui-definition-referencing-functions.md#basics)
* [ubicación](create-ui-definition-referencing-functions.md#location)
* [resourceGroup](create-ui-definition-referencing-functions.md#resourcegroup)
* [steps](create-ui-definition-referencing-functions.md#steps)
* [suscripción](create-ui-definition-referencing-functions.md#subscription)

## <a name="string-functions"></a>Funciones de cadena

* [concat](create-ui-definition-string-functions.md#concat)
* [endsWith](create-ui-definition-string-functions.md#endswith)
* [guid](create-ui-definition-string-functions.md#guid)
* [indexOf](create-ui-definition-string-functions.md#indexof)
* [lastIndexOf](create-ui-definition-string-functions.md#lastindexof)
* [replace](create-ui-definition-string-functions.md#replace)
* [startsWith](create-ui-definition-string-functions.md#startswith)
* [substring](create-ui-definition-string-functions.md#substring)
* [toLower](create-ui-definition-string-functions.md#tolower)
* [toUpper](create-ui-definition-string-functions.md#toupper)

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).
