---
title: Creación de funciones lógicas de definición de interfaz de usuario
description: Describe las funciones para realizar operaciones lógicas.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 00d2f0eeb5d353c8ebd7ad30f6866f890d6cb42e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095210"
---
# <a name="createuidefinition-logical-functions"></a>Funciones lógicas CreateUiDefinition

Estas funciones se pueden usar en las expresiones condicionales. Algunas funciones pueden no admitir todos los tipos de datos JSON.

## <a name="and"></a>y

Devuelve `true` si todos los parámetros se evalúan como `true`. Esta función admite dos o más parámetros solo de tipo booleano.

El siguiente ejemplo devuelve `true`:

```json
"[and(equals(0, 0), equals('web', 'web'), less(1, 2))]"
```

El siguiente ejemplo devuelve `false`:

```json
"[and(equals(0, 0), greater(1, 2))]"
```

## <a name="coalesce"></a>coalesce

Devuelve el valor del primer parámetro no nulo. Algunas funciones admiten todos los tipos de datos JSON.

Asuma que `element1` y `element2` son indefinidos. El siguiente ejemplo devuelve `"Contoso"`:

```json
"[coalesce(steps('demoStep').element1, steps('demoStep').element2, 'Contoso')]"
```

Esta función es especialmente útil en el contexto de la invocación opcional que se produce debido a una acción del usuario después de que se carga la página. Un ejemplo puede ser si las restricciones colocadas en un campo de la interfaz de usuario dependen del valor seleccionado actualmente en otro campo que **no es visible inicialmente**. En este caso, se puede usar `coalesce()` para permitir que la función sea sintácticamente válida en el momento de la carga de la página y tenga el efecto deseado cuando el usuario interactúa con el campo.

Tenga en cuenta este elemento `DropDown`, que permite al usuario elegir entre varios tipos de bases de datos diferentes:

```
{
    "name": "databaseType",
    "type": "Microsoft.Common.DropDown",
    "label": "Choose database type",
    "toolTip": "Choose database type",
    "defaultValue": "Oracle Database",
    "visible": "[bool(steps('section_database').connectToDatabase)]"
    "constraints": {
        "allowedValues": [
            {
                "label": "Azure Database for PostgreSQL",
                "value": "postgresql"
            },
            {
                "label": "Oracle Database",
                "value": "oracle"
            },
            {
                "label": "Azure SQL",
                "value": "sqlserver"
            }
        ],
        "required": true
    },
```

Para condicionar la acción de otro campo según el valor seleccionado actualmente en este campo, use `coalesce()`, como se muestra a continuación:

```
"regex": "[concat('^jdbc:', coalesce(steps('section_database').databaseConnectionInfo.databaseType, ''), '.*$')]",
```

Esto es necesario porque el campo `databaseType` no es visible inicialmente y, por lo tanto, no tiene un valor. Esto hace que toda la expresión no se evalúe correctamente.

## <a name="equals"></a>equals

Devuelve `true` si ambos parámetros tienen el mismo tipo y valor. Algunas funciones admiten todos los tipos de datos JSON.

El siguiente ejemplo devuelve `true`:

```json
"[equals(0, 0)]"
```

El siguiente ejemplo devuelve `true`:

```json
"[equals('web', 'web')]"
```

El siguiente ejemplo devuelve `false`:

```json
"[equals('abc', ['a', 'b', 'c'])]"
```

## <a name="greater"></a>greater

Devuelve `true` si el primer parámetro es estrictamente mayor que el segundo parámetro. Esta función admite parámetros solo de tipo number y string.

El siguiente ejemplo devuelve `false`:

```json
"[greater(1, 2)]"
```

El siguiente ejemplo devuelve `true`:

```json
"[greater('9', '10')]"
```

## <a name="greaterorequals"></a>greaterOrEquals

Devuelve `true` si el primer parámetro es estrictamente mayor o igual que el segundo parámetro. Esta función admite parámetros solo de tipo number y string.

El siguiente ejemplo devuelve `true`:

```json
"[greaterOrEquals(2, 2)]"
```

## <a name="if"></a>if

Devuelve un valor dependiendo de si una condición es verdadera o falsa. El primer parámetro es la condición que se va a probar. El segundo, es el valor que se va a devolver si la condición es verdadera. El tercero, es el valor que se va a devolver si la condición es falsa.

En el ejemplo siguiente se devuelve `yes`.

```json
"[if(equals(42, mul(6, 7)), 'yes', 'no')]"
```

## <a name="less"></a>less

Devuelve `true` si el primer parámetro es estrictamente menor que el segundo parámetro. Esta función admite parámetros solo de tipo number y string.

El siguiente ejemplo devuelve `true`:

```json
"[less(1, 2)]"
```

El siguiente ejemplo devuelve `false`:

```json
"[less('9', '10')]"
```

## <a name="lessorequals"></a>lessOrEquals

Devuelve `true` si el primer parámetro es estrictamente menor o igual que el segundo parámetro. Esta función admite parámetros solo de tipo number y string.

El siguiente ejemplo devuelve `true`:

```json
"[lessOrEquals(2, 2)]"
```

## <a name="not"></a>not

Devuelve `true` si el parámetro se evalúa como `false`. Esta función admite parámetros solo de tipo booleano.

El siguiente ejemplo devuelve `true`:

```json
"[not(false)]"
```

El siguiente ejemplo devuelve `false`:

```json
"[not(equals(0, 0))]"
```

## <a name="or"></a>or

Devuelve `true` si al menos uno de los parámetros se evalúa como `true`. Esta función admite dos o más parámetros solo de tipo booleano.

El siguiente ejemplo devuelve `true`:

```json
"[or(equals(0, 0), equals('web', 'web'), less(1, 2))]"
```

El siguiente ejemplo devuelve `true`:

```json
"[or(equals(0, 0), greater(1, 2))]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción a Azure Resource Manager, consulte [Información general sobre Azure Resource Manager](../management/overview.md).
