---
title: Creación de directivas para propiedades de matriz en recursos
description: Aprenda a trabajar con parámetros de matriz y expresiones de lenguaje de matriz, evaluar el alias [*] y anexar elementos con las reglas de definición de Azure Policy.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 5b9392a943e264ae5eca989ee87eb9ff09b36972
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048489"
---
# <a name="author-policies-for-array-properties-on-azure-resources"></a>Creación de directivas para propiedades de matriz en recursos de Azure

Las propiedades de Azure Resource Manager se suelen definir como cadenas y valores booleanos. En cambio, cuando existe una relación uno a varios, las propiedades complejas se definen como matrices. En Azure Policy, las matrices se usan de varias maneras diferentes:

- Tipo de un [parámetro de definición](../concepts/definition-structure.md#parameters), para proporcionar varias opciones
- Parte de una [regla de directiva](../concepts/definition-structure.md#policy-rule) con las condiciones **in** o **notIn**
- Parte de una regla de directiva que evalúa el [alias \[\*\]](../concepts/definition-structure.md#understanding-the--alias) para evaluar:
  - Escenarios como **Ninguno**, **Cualquiera** o **Todo**
  - Escenarios complejos con **count**
- En el [efecto append](../concepts/effects.md#append), para reemplazar o agregar a una matriz existente

En este artículo se explica cada uso por parte de Azure Policy y proporciona varias definiciones de ejemplo.

## <a name="parameter-arrays"></a>Matrices de parámetros

### <a name="define-a-parameter-array"></a>Definir una matriz de parámetros

Definir un parámetro como una matriz, permite la flexibilidad de la directiva cuando se necesita más de un valor.
Esta definición de directiva permite cualquier ubicación única para el parámetro **allowedLocations** y el valor predeterminado es _eastus2_:

```json
"parameters": {
    "allowedLocations": {
        "type": "string",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": "eastus2"
    }
}
```

Como **type** es _string_, solo se puede establecer un único valor al asignar la directiva. Si se asigna esta directiva, los recursos del ámbito solo se permiten en una única región de Azure. La mayoría de las definiciones de directivas deben permitir una lista de opciones aprobadas, como permitir _eastus2_, _eastus_ y _westus2_.

Para crear la definición de directiva a fin de permitir varias opciones, utilice el elemento **type** _array_. La misma directiva se puede reescribir del siguiente modo:

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": "eastus2",
        "allowedValues": [
            "eastus2",
            "eastus",
            "westus2"
        ]

    }
}
```

> [!NOTE]
> Una vez que se guarda una definición de directiva, no se puede cambiar la propiedad **type** en un parámetro.

Esta nueva definición de parámetro toma más de un valor durante la asignación de directiva. Con la propiedad de matriz **allowedValues** definida, los valores disponibles durante la asignación se limitan más a la lista predefinida de opciones. El uso de **allowedValues** es opcional.

### <a name="pass-values-to-a-parameter-array-during-assignment"></a>Pasar valores a una matriz de parámetros durante la asignación

Al asignar la directiva a través de Azure Portal, un parámetro de **type** _array_ se muestra como un único cuadro de texto. La sugerencia indica "Use ; para separar los valores (por ejemplo, Londres;Nueva York)". Para pasar los valores de ubicación permitidos _eastus2_, _eastus_ y _westus2_ al parámetro, use la siguiente cadena:

`eastus2;eastus;westus2`

El formato para el valor del parámetro es distinto cuando se usa la CLI de Azure, Azure PowerShell o la API de REST. Los valores se pasan a través de una cadena JSON que también incluye el nombre del parámetro.

```json
{
    "allowedLocations": {
        "value": [
            "eastus2",
            "eastus",
            "westus2"
        ]
    }
}
```

Para usar esta cadena con cada SDK, use los siguientes comandos:

- CLI de Azure: Comando [az policy assignment create](/cli/azure/policy/assignment#az-policy-assignment-create) con el parámetro **params**
- Azure PowerShell: Cmdlet [New AzPolicyAssignment](/powershell/module/az.resources/New-Azpolicyassignment) con el parámetro **PolicyParameter**
- API REST: en la operación _PUT_ [create](/rest/api/resources/policyassignments/create) como parte del cuerpo de la solicitud, como el valor de la propiedad **properties.parameters**

## <a name="policy-rules-and-arrays"></a>Matrices y reglas de directiva

### <a name="array-conditions"></a>Condiciones de matriz

Las [condiciones](../concepts/definition-structure.md#conditions) de la regla de directiva con la que puede usarse _array_
**type** del parámetro están limitadas a `in` y `notIn`. Tome la siguiente definición de directiva con la condición `equals` como ejemplo:

```json
{
  "policyRule": {
    "if": {
      "not": {
        "field": "location",
        "equals": "[parameters('allowedLocations')]"
      }
    },
    "then": {
      "effect": "audit"
    }
  },
  "parameters": {
    "allowedLocations": {
      "type": "Array",
      "metadata": {
        "description": "The list of allowed locations for resources.",
        "displayName": "Allowed locations",
        "strongType": "location"
      }
    }
  }
}
```

Al intentar crear esta definición de directiva a través de Azure Portal genera un error, como este mensaje de error:

- "No se pudo parametrizar la directiva '{GUID}' por errores de validación. Compruebe si los parámetros de la directiva están definidos correctamente. La excepción interna 'El resultado de la evaluación de la expresión de lenguaje '[parameters('allowedLocations')]' es de tipo 'Matriz'. El tipo esperado es 'String'".

El elemento **type** esperado de la condición `equals` es _string_. Puesto que **allowedLocations** se define como **type** _array_, el motor de directiva evalúa la expresión de lenguaje y genera el error. Con la condición `in` y `notIn`, el motor de directiva espera el elemento **type** _array_ en la expresión de lenguaje. Para resolver este mensaje de error, cambie `equals` a `in` o `notIn`.

### <a name="evaluating-the--alias"></a>Evaluación del alias [*]

Los alias que tienen **\[\*\]** adjuntado a sus nombres indican que **type** es una _matriz_. En lugar de evaluar el valor de toda la matriz, **\[\*\]** permite evaluar los elementos de la matriz individualmente, con el operador lógico AND entre ellos. Hay tres escenarios estándar en que la evaluación por elemento es útil: la coincidencia de _ninguno_, _alguno_ o _todos_ los elementos. Para escenarios complejos, use [count](../concepts/definition-structure.md#count).

El motor de directiva desencadena **effect** en **then** solo cuando la regla **if** se evalúa como true.
Este hecho es importante para comprender en contexto la manera en que **\[\*\]** evalúa cada elemento individual de la matriz.

La regla de directiva siguiente es un ejemplo para la tabla de escenario:

```json
"policyRule": {
    "if": {
        "allOf": [
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
                "exists": "true"
            },
            <-- Condition (see table below) -->
        ]
    },
    "then": {
        "effect": "[parameters('effectType')]"
    }
}
```

La matriz **ipRules** es como sigue para la tabla de escenario:

```json
"ipRules": [
    {
        "value": "127.0.0.1",
        "action": "Allow"
    },
    {
        "value": "192.168.1.1",
        "action": "Allow"
    }
]
```

Para cada ejemplo de condición siguiente, reemplace `<field>` por `"field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value"`.

Los siguientes resultados se derivan de la combinación de la condición y la regla de directiva de ejemplo y la matriz de valores existentes anteriores:

|Condición |Resultado | Escenario |Explicación |
|-|-|-|-|
|`{<field>,"notEquals":"127.0.0.1"}` |Nada |Ninguna coincidencia |Un elemento de matriz se evalúa como false (127.0.0.1 != 127.0.0.1) y otro como true (127.0.0.1 != 192.168.1.1), por lo que la condición **notEquals** es _false_ y el efecto no se desencadena. |
|`{<field>,"notEquals":"10.0.4.1"}` |Efecto de la directiva |Ninguna coincidencia |Ambos elementos de matriz se evalúan como true (10.0.4.1 != 127.0.0.1 and 10.0.4.1 != 192.168.1.1), por lo que la condición **notEquals** es _true_ y el efecto se desencadena. |
|`"not":{<field>,"notEquals":"127.0.0.1" }` |Efecto de la directiva |Una o más coincidencias |Un elemento de matriz se evalúa como false (127.0.0.1 != 127.0.0.1) y otro como true (127.0.0.1 != 192.168.1.1), por lo que la condición **notEquals** es _false_. El operador lógico se evalúa como true (**not** _false_), por lo que se desencadena el efecto. |
|`"not":{<field>,"notEquals":"10.0.4.1"}` |Nada |Una o más coincidencias |Ambos elementos de matriz se evalúan como true (10.0.4.1 != 127.0.0.1 y 10.0.4.1 != 192.168.1.1), por lo que la condición **notEquals** es _true_. El operador lógico se evalúa como false (**not** _true_), por lo que no se desencadena el efecto. |
|`"not":{<field>,"Equals":"127.0.0.1"}` |Efecto de la directiva |Coincidencia parcial |Un elemento de matriz se evalúa como true (127.0.0.1 == 127.0.0.1) y otro como false (127.0.0.1 == 192.168.1.1), por lo que la condición **Equals** es _false_. El operador lógico se evalúa como true (**not** _false_), por lo que se desencadena el efecto. |
|`"not":{<field>,"Equals":"10.0.4.1"}` |Efecto de la directiva |Coincidencia parcial |Ambos elementos de matriz se evalúan como false (10.0.4.1 == 127.0.0.1 y 10.0.4.1 == 192.168.1.1), por lo que la condición **Equals** es _false_. El operador lógico se evalúa como true (**not** _false_), por lo que se desencadena el efecto. |
|`{<field>,"Equals":"127.0.0.1"}` |Nada |Coincidencia total |Un elemento de matriz se evalúa como true (127.0.0.1 == 127.0.0.1) y otro como false (127.0.0.1 == 192.168.1.1), por lo que la condición **Equals** es _false_ y el efecto no se desencadena. |
|`{<field>,"Equals":"10.0.4.1"}` |Nada |Coincidencia total |Ambos elementos de matriz se evalúan como false (10.0.4.1 == 127.0.0.1 y 10.0.4.1 == 192.168.1.1), por lo que la condición **Equals** es _false_ y el efecto no se desencadena. |

## <a name="the-append-effect-and-arrays"></a>El efecto de anexar y matrices

El [efecto append](../concepts/effects.md#append) se comporta de manera diferente en función de si **details.field** es un alias **\[\*\]** o no.

- Cuando no es un alias **\[\*\]** , append reemplaza toda la matriz por la propiedad **value**.
- Cuando se trata de un alias **\[\*\]** , append agrega la propiedad **value** a la matriz existente o crea una nueva matriz.

Para obtener más información, consulte [Ejemplos de append](../concepts/effects.md#append-examples).

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
