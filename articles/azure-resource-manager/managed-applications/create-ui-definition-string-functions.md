---
title: Creación de funciones de cadena para definición de interfaz de usuario
description: Describe las funciones de cadena que se usarán al crear definiciones de interfaz de usuario para Azure Managed Applications
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: c662948542c36cd93f889ca045ee245c15c7bb11
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095206"
---
# <a name="createuidefinition-string-functions"></a>Funciones de cadena CreateUiDefinition

Estas funciones se usan con cadenas JSON.

## <a name="concat"></a>concat

Concatena una o varias cadenas.

Por ejemplo, si el valor de salida de `element1` si `"Contoso"`, a continuación, en este ejemplo se devuelve la cadena de `"Demo Contoso app"`:

```json
"[concat('Demo ', steps('step1').element1, ' app')]"
```

## <a name="endswith"></a>endsWith

Determina si una cadena termina con un valor.

En el ejemplo siguiente se devuelve true.

```json
"[endsWith('tuvwxyz', 'xyz')]"
```

## <a name="guid"></a>guid

Genera una cadena única global (GUID).

El ejemplo siguiente devuelve un valor como `"c7bc8bdc-7252-4a82-ba53-7c468679a511"`:

```json
"[guid()]"
```

## <a name="indexof"></a>indexOf

Devuelve la primera posición de un valor dentro de una cadena o -1 si no se encuentra.

En el ejemplo siguiente se devuelve 2.

```json
"[indexOf('abcdef', 'cd')]"
```

## <a name="lastindexof"></a>lastIndexOf

Devuelve la última posición de un valor en una cadena o -1 si no se encuentra.

En el ejemplo siguiente se devuelve 3.

```json
"[lastIndexOf('test', 't')]"
```

## <a name="replace"></a>replace

Devuelve una cadena en la que se reemplazan todas las instancias de la cadena especificada en la cadena actual con otra cadena.

El siguiente ejemplo devuelve `"Contoso.com web app"`:

```json
"[replace('Contoso.net web app', '.net', '.com')]"
```

## <a name="startswith"></a>startsWith

Determina si una cadena empieza con un valor.

En el ejemplo siguiente se devuelve true.

```json
"[startsWith('abcdefg', 'ab')]"
```

## <a name="substring"></a>substring

Devuelve la subcadena de la cadena especificada. La subcadena comienza en el índice especificado y tiene la longitud especificada.

El siguiente ejemplo devuelve `"web"`:

```json
"[substring('Contoso.com web app', 12, 3)]"
```

## <a name="tolower"></a>toLower

Devuelve una cadena convertida a minúsculas.

El siguiente ejemplo devuelve `"contoso"`:

```json
"[toLower('CONTOSO')]"
```

## <a name="toupper"></a>toUpper

Devuelve una cadena convertida a mayúsculas.

El siguiente ejemplo devuelve `"CONTOSO"`:

```json
"[toUpper('contoso')]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).

