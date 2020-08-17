---
title: Creación de funciones matemáticas de definición de interfaz de usuario
description: Describe las funciones que se van a usar al realizar operaciones matemáticas.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 022e59d847a4d89b4243223637fc6fd1e73255a9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095208"
---
# <a name="createuidefinition-math-functions"></a>Funciones matemáticas CreateUiDefinition

Las funciones permiten realizar operaciones matemáticas.

## <a name="add"></a>add

Agrega dos números y devuelve el resultado.

El siguiente ejemplo devuelve `3`:

```json
"[add(1, 2)]"
```

## <a name="ceil"></a>ceil

Devuelve el valor entero más grande mayor o igual que el número especificado.

El siguiente ejemplo devuelve `4`:

```json
"[ceil(3.14)]"
```

## <a name="div"></a>div

Divide el segundo número por el primer número y devuelve el resultado. El resultado es siempre un número entero.

El siguiente ejemplo devuelve `2`:

```json
"[div(6, 3)]"
```

## <a name="floor"></a>floor

Devuelve el valor entero más grande menor o igual que el número especificado.

El siguiente ejemplo devuelve `3`:

```json
"[floor(3.14)]"
```

## <a name="max"></a>max

Devuelve el número más grande de los dos.

El siguiente ejemplo devuelve `2`:

```json
"[max(1, 2)]"
```

## <a name="min"></a>Min

Devuelve el más pequeño de dos números.

El siguiente ejemplo devuelve `1`:

```json
"[min(1, 2)]"
```

## <a name="mod"></a>mod

Divide el primer número por el segundo número y devuelve el resto.

El siguiente ejemplo devuelve `0`:

```json
"[mod(6, 3)]"
```

El siguiente ejemplo devuelve `2`:

```json
"[mod(6, 4)]"
```

## <a name="mul"></a>mul

Multiplica dos números y devuelve el resultado.

El siguiente ejemplo devuelve `6`:

```json
"[mul(2, 3)]"
```

## <a name="rand"></a>rand

Genera un número integral aleatorio en el intervalo especificado. Esta función no genera números aleatorios criptográficamente seguros.

El siguiente ejemplo podría devolver `42`:

```json
"[rand(-100, 100)]"
```

## <a name="range"></a>range

Genera una secuencia de números enteros en el intervalo especificado.

El siguiente ejemplo devuelve `[1,2,3]`:

```json
"[range(1, 3)]"
```

## <a name="sub"></a>sub

Resta el segundo número al primer número y devuelve el resultado.

El siguiente ejemplo devuelve `1`:

```json
"[sub(3, 2)]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).
