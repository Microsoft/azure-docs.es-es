---
title: Creación de funciones de fecha para definición de interfaz de usuario
description: Describe las funciones que se van a usar al trabajar con valores de fecha.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 80484fd15e51bae7036c43bd3ca8fe8167387ede
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095212"
---
# <a name="createuidefinition-date-functions"></a>Funciones de fecha CreateUiDefinition

Las funciones que se van a usar al trabajar con valores de fecha.

## <a name="addhours"></a>addHours

Agrega un número entero de horas a la marca de tiempo especificada.

El siguiente ejemplo devuelve `"1991-01-01T00:59:59.000Z"`:

```json
"[addHours('1990-12-31T23:59:59Z', 1)]"
```

## <a name="addminutes"></a>addMinutes

Agrega un número entero de minutos a la marca de tiempo especificada.

El siguiente ejemplo devuelve `"1991-01-01T00:00:59.000Z"`:

```json
"[addMinutes('1990-12-31T23:59:59Z', 1)]"
```

## <a name="addseconds"></a>addSeconds
Agrega un número entero de segundos a la marca de tiempo especificada.

El siguiente ejemplo devuelve `"1991-01-01T00:00:00.000Z"`:

```json
"[addSeconds('1990-12-31T23:59:60Z', 1)]"
```

## <a name="utcnow"></a>utcNow

Devuelve una cadena en formato ISO 8601 de la fecha y hora actuales en el equipo local.

El siguiente ejemplo podría devolver `"1990-12-31T23:59:59.000Z"`:

```json
"[utcNow()]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).
