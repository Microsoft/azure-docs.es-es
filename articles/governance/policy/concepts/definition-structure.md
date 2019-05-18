---
title: Detalles de la estructura de definición de directivas
description: Describe cómo Azure Policy usa la definición de directiva de recursos para establecer convenciones para los recursos de su organización al describir cuándo se aplica la directiva y qué efecto tiene.
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/13/2019
ms.topic: conceptual
ms.service: azure-policy
manager: carmonm
ms.custom: seodec18
ms.openlocfilehash: b9fe723ca13cbee0e31b14e60a6bd740d2a282df
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779287"
---
# <a name="azure-policy-definition-structure"></a>Estructura de definición de Azure Policy

Azure Policy usa las definiciones de directivas de recursos para establecer convenciones para los recursos. Cada definición describe la compatibilidad de recursos y el efecto que debe realizarse cuando un recurso no es compatible.
La definición de convenciones permite controlar los costes y administrar los recursos más fácilmente. Por ejemplo, puede especificar que se permitan solo determinados tipos de máquinas virtuales. O puede obligar a que todos los recursos tengan una etiqueta concreta. Todos los recursos secundarios heredan las directivas. Si una directiva se aplica a un grupo de recursos, será aplicable a todos los recursos de dicho grupo de recursos.

El esquema utilizado por Azure Policy puede encontrarse aquí: [https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json](https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json)

Para crear una definición de directiva se utiliza JSON. La definición de directiva contiene elementos para:

- mode
- parameters
- nombre para mostrar
- description
- regla de directiva
  - evaluación lógica
  - efecto

Por ejemplo, el siguiente JSON muestra una directiva que limita las ubicaciones donde se implementan los recursos:

```json
{
    "properties": {
        "mode": "all",
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                },
                "defaultValue": [ "westus2" ]
            }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

Todos los ejemplos de Azure Policy están en [ejemplos de Azure Policy](../samples/index.md).

[!INCLUDE [az-powershell-update](../../../../includes/updated-for-az.md)]

## <a name="mode"></a>Mode

El **modo** determina qué tipos de recurso se evaluarán para una directiva. Los modos admitidos son:

- `all`: evalúe los grupos de recursos y todos los tipos de recurso
- `indexed`: evalúe solo los tipos de recurso que admitan las etiquetas y la ubicación

Se recomienda que establezca **mode** en `all` en la mayoría de los casos. Todas las definiciones de directivas creadas a través del portal usan el modo `all`. Si usa PowerShell o la CLI de Azure, puede especificar el parámetro **mode** de forma manual. Si la definición de directiva no incluye un valor de **modo**, el valor predeterminado es `all` en Azure PowerShell y `null` en la CLI de Azure. Un modo `null` es lo mismo que usar `indexed` para la compatibilidad con versiones anteriores.

`indexed` debe usarse al crear directivas que apliquen etiquetas o ubicaciones. Aunque no es obligatorio, impide que los recursos que no son compatibles con etiquetas y ubicaciones aparezcan como no compatibles en los resultados de cumplimiento. La excepción son los **grupos de recursos**. Las directivas que aplican la ubicación o etiquetas en un grupo de recursos deben establecer **mode** en `all` y tener como destino específico el tipo `Microsoft.Resources/subscriptions/resourceGroups`. Para obtener un ejemplo, consulte [Aplicar etiqueta y su valor en grupos de recursos](../samples/enforce-tag-rg.md). Para obtener una lista de recursos que admiten etiquetas, consulte [compatibilidad para los recursos de Azure de etiquetas](../../../azure-resource-manager/tag-support.md).

## <a name="parameters"></a>Parámetros

Los parámetros ayudan a simplificar la administración de directivas mediante la reducción del número de definiciones de directiva. Piense en los parámetros como si fueran los campos de un formulario: `name`, `address`, `city`, `state`. Estos parámetros no cambian, pero sí sus valores en función del individuo que rellena el formulario.
Los parámetros funcionan del mismo modo al crear las directivas. Con la inclusión de parámetros en una definición de directiva, puede volver a usar esa directiva en distintos escenarios con valores diferentes.

> [!NOTE]
> Pueden añadirse parámetros a una definición existente y asignada. El nuevo parámetro debe incluir la propiedad **defaultValue**. Así se evita que las asignaciones existentes de la directiva o la iniciativa realizadas indirectamente no sean válidas.

### <a name="parameter-properties"></a>Propiedades del parámetro

Un parámetro tiene las siguientes propiedades que se usan en la definición de directiva:

- **name**: El nombre del parámetro. Lo utiliza la función de la implementación `parameters` dentro de la regla de directiva. Para más información, consulte [Uso de un valor de parámetro](#using-a-parameter-value).
- `type`: Determina si el parámetro es una **cadena** o una **matriz**.
- `metadata`: Define las subpropiedades que usa principalmente Azure Portal para mostrar información intuitiva:
  - `description`: La explicación de para qué se usa el parámetro. Puede utilizarse para proporcionar ejemplos de valores aceptables.
  - `displayName`: El nombre descriptivo que se muestra en el portal para el parámetro.
  - `strongType`: (Opcional) Se usa al asignar la definición de directiva mediante el portal. Proporciona una lista que tiene en cuenta el contexto. Para más información, consulte [strongType](#strongtype).
  - `assignPermissions`: (Opcional) Establecer como _true_ para que Azure portal cree asignaciones de roles durante la asignación de directiva. Esta propiedad es útil en caso de que desea asignar permisos fuera del ámbito de asignación. Hay una asignación de roles por cada definición de rol en la directiva (o por la definición de roles en todas las directivas en la iniciativa). El valor del parámetro debe ser un recurso válido o un ámbito.
- `defaultValue`: (Opcional) Establece el valor del parámetro en una asignación, si no se especifica ningún valor. Requerido cuando se actualiza una definición de directiva existente que está asignada.
- `allowedValues`: (Opcional) Proporciona una matriz de valores que acepta el parámetro durante la asignación.

Por ejemplo, podría definir una definición de directiva para limitar las ubicaciones en las que se pueden implementar los recursos. Un parámetro para esa definición de directiva podría ser **allowedLocations**. Este parámetro podría utilizarse por cada asignación de la definición de directiva para limitar los valores aceptados. El uso de **strongType** proporciona una experiencia mejorada al completar la asignación mediante el portal:

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="using-a-parameter-value"></a>Uso de un valor de parámetro

En la regla de directiva, se hace referencia a los parámetros con la siguiente sintaxis de función con el valor de implementación `parameters`:

```json
{
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

En este ejemplo se hace referencia al parámetro **allowedLocations** que se mostró en las [ propiedades del parámetro](#parameter-properties).

### <a name="strongtype"></a>strongType

Dentro de la propiedad `metadata`, puede usar **strongType** para proporcionar una lista de opciones de selección múltiple en Azure Portal. Los valores permitidos para **strongType** actualmente incluyen:

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`
- `omsWorkspace`
- `Microsoft.EventHub/Namespaces/EventHubs`
- `Microsoft.EventHub/Namespaces/EventHubs/AuthorizationRules`
- `Microsoft.EventHub/Namespaces/AuthorizationRules`
- `Microsoft.RecoveryServices/vaults`
- `Microsoft.RecoveryServices/vaults/backupPolicies`

## <a name="definition-location"></a>Ubicación de definición

Al crear una iniciativa o directiva, es necesario especificar la ubicación de la definición. La ubicación de la definición puede especificarse como un grupo de administración o una suscripción. La ubicación determina el ámbito al que pueden asignarse la directiva o la iniciativa. Los recursos deben ser miembros directos o elementos secundarios dentro de la jerarquía de la ubicación de la definición para que puedan ser destino de asignación.

Si la ubicación de la definición es:

- Una **suscripción**: solo se puede asignar la directiva a los recursos dentro de esa suscripción.
- Un **grupo de administración**: solo se puede asignar la directiva a los recursos dentro de grupos de administración secundarios y suscripciones secundarias. Si planea aplicar la definición de directiva a varias suscripciones, la ubicación debe ser un grupo de administración que contenga esas suscripciones.

## <a name="display-name-and-description"></a>Nombre para mostrar y descripción

Use los valores **displayName** y **description** para identificar la definición de directiva y proporcionar el contexto para su uso. **displayName** tiene una longitud máxima de _128_ caracteres y **description** tiene una longitud máxima de _512_ caracteres.

## <a name="policy-rule"></a>Regla de directivas

La regla de directiva se compone de los bloques **If** y **Then**. En el bloque **If**, defina una o varias condiciones que especifican cuándo se aplica la directiva. Puede aplicar operadores lógicos a estas condiciones para definir con precisión el escenario de una directiva.

En el bloque **Then**, defina el efecto que se produce cuando se cumplen las condiciones de **If**.

```json
{
    "if": {
        <condition> | <logical operator>
    },
    "then": {
        "effect": "deny | audit | append | auditIfNotExists | deployIfNotExists | disabled"
    }
}
```

### <a name="logical-operators"></a>Operadores lógicos

Los operadores lógicos admitidos son:

- `"not": {condition  or operator}`
- `"allOf": [{condition or operator},{condition or operator}]`
- `"anyOf": [{condition or operator},{condition or operator}]`

La sintaxis **not** invierte el resultado de la condición. La sintaxis **allOf** (similar a la operación lógica **And**) requiere que se cumplan todas las condiciones. La sintaxis **anyOf** (similar a la operación lógica **Or**) requiere que se cumplan una o varias condiciones.

Puede anidar los operadores lógicos. El ejemplo siguiente muestra una operación **not** que está anidada dentro de una operación **allOf**.

```json
"if": {
    "allOf": [{
            "not": {
                "field": "tags",
                "containsKey": "application"
            }
        },
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        }
    ]
},
```

### <a name="conditions"></a>Condiciones

Una condición evalúa si un **campo** o el descriptor de acceso **value** cumple determinados criterios. Estas son las condiciones que se admiten:

- `"equals": "value"`
- `"notEquals": "value"`
- `"like": "value"`
- `"notLike": "value"`
- `"match": "value"`
- `"matchInsensitively": "value"`
- `"notMatch": "value"`
- `"notMatchInsensitively": "value"`
- `"contains": "value"`
- `"notContains": "value"`
- `"in": ["value1","value2"]`
- `"notIn": ["value1","value2"]`
- `"containsKey": "keyName"`
- `"notContainsKey": "keyName"`
- `"less": "value"`
- `"lessOrEquals": "value"`
- `"greater": "value"`
- `"greaterOrEquals": "value"`
- `"exists": "bool"`

Cuando se usan las condiciones **like** y **notLike**, incluya un carácter comodín (`*`) en el valor.
El valor no debe contener más de un carácter comodín `*`.

Cuando se usa el **coincide con** y **notMatch** proporcionar condiciones, `#` para que coincida con un dígito, `?` por una letra, `.` para que coincida con cualquier carácter y cualquier otro carácter para que coincida con ese carácter en Sí.
**match** y **notMatch** distinguen mayúsculas de minúsculas. Las alternativas de distinción entre mayúsculas y minúsculas están disponibles en **matchInsensitively** y **notMatchInsensitively**. Por ejemplo, consulte [Permitir varios patrones de nombre](../samples/allow-multiple-name-patterns.md).

### <a name="fields"></a>Fields

Para crear condiciones se usan campos. Un campo coincide con las propiedades de la carga de solicitud de recursos y describe el estado del recurso.

Se admiten los siguientes campos:

- `name`
- `fullName`
  - Devuelve el nombre completo del recurso. El nombre completo de un recurso es el nombre del recurso precedido por los nombres de recurso principal (por ejemplo "myServer/myDatabase").
- `kind`
- `type`
- `location`
  - Use **global** para los recursos que son independientes de la ubicación. Para obtener un ejemplo, consulte [Ejemplos: ubicaciones permitidas](../samples/allowed-locations.md).
- `identity.type`
  - Devuelve el tipo de [identidad administrada](../../../active-directory/managed-identities-azure-resources/overview.md) habilitado en el recurso.
- `tags`
- `tags['<tagName>']`
  - Esta sintaxis con corchetes admite nombres de etiquetas que tienen signos de puntuación como guion, punto o espacio.
  - Donde **\<tagName\>** es el nombre de la etiqueta para validar la condición.
  - Ejemplos: `tags['Acct.CostCenter']` donde **Acct.CostCenter** es el nombre de la etiqueta.
- `tags['''<tagName>''']`
  - Esta sintaxis con corchetes admite nombres de etiquetas con apóstrofos mediante secuencias de escape con dobles apóstrofos.
  - Donde **\<tagName\>** es el nombre de la etiqueta para validar la condición.
  - Ejemplo: `tags['''My.Apostrophe.Tag''']` donde **'\<tagName\>'** es el nombre de la etiqueta.
- alias de propiedad: para obtener una lista, vea [Alias](#aliases).

> [!NOTE]
> `tags.<tagName>`, `tags[tagName]` y `tags[tag.with.dots]` son todavía formas aceptables de declarar un campo de etiquetas.
> Sin embargo, las expresiones preferidas son las mencionadas anteriormente.

#### <a name="use-tags-with-parameters"></a>Uso de etiquetas con parámetros

Un valor de parámetro se puede pasar a un campo de etiqueta. Al pasar un parámetro a un campo de etiqueta aumenta la flexibilidad de la definición de directiva durante la asignación de directivas.

En el ejemplo siguiente, `concat` se usa para crear una búsqueda de campos de etiquetas para la etiqueta denominada con el valor del parámetro **tagName**. Si esa etiqueta no existe, se usa el efecto **append** para agregarla con el valor de la misma etiqueta con nombre establecida en el grupo de recursos principal de los recursos auditados mediante la función de búsqueda `resourcegroup()`.

```json
{
    "if": {
        "field": "[concat('tags[', parameters('tagName'), ']')]",
        "exists": "false"
    },
    "then": {
        "effect": "append",
        "details": [{
            "field": "[concat('tags[', parameters('tagName'), ']')]",
            "value": "[resourcegroup().tags[parameters('tagName')]]"
        }]
    }
}
```

### <a name="value"></a>Valor

Las condiciones también se pueden formar mediante el uso de **value**. **value** comprueba las condiciones en [parámetros](#parameters), [funciones de plantilla admitidas](#policy-functions) o literales.
**value** se empareja con cualquier [condición](#conditions) admitida.

> [!WARNING]
> Si el resultado de una _función de plantilla_ es un error, produce un error de evaluación de directiva. Una evaluación con errores es implícita **denegar**. Para obtener más información, consulte [evitar los errores de plantilla](#avoiding-template-failures).

#### <a name="value-examples"></a>Ejemplos de value

En este ejemplo de regla de directiva se utiliza **value** para comparar el resultado de la función `resourceGroup()` y la propiedad **name** devuelta con una condición  **like** de `*netrg`. La regla niega cualquier recurso que no sea de **tipo** `Microsoft.Network/*` en cualquier grupo de recursos cuyo nombre termine en `*netrg`.

```json
{
    "if": {
        "allOf": [{
                "value": "[resourceGroup().name]",
                "like": "*netrg"
            },
            {
                "field": "type",
                "notLike": "Microsoft.Network/*"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

En este ejemplo de regla de directiva se utiliza **value** para comprobar si el resultado de varias funciones anidadas es **igual a** `true`. La regla deniega cualquier recurso que no tenga al menos tres etiquetas.

```json
{
    "mode": "indexed",
    "policyRule": {
        "if": {
            "value": "[less(length(field('tags')), 3)]",
            "equals": true
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

#### <a name="avoiding-template-failures"></a>Evitar los errores de plantilla

El uso de _funciones de plantilla_ en **valor** permite muchas funciones anidadas complejas. Si el resultado de una _función de plantilla_ es un error, produce un error de evaluación de directiva. Una evaluación con errores es implícita **denegar**. Un ejemplo de un **valor** se produce un error en ciertos escenarios:

```json
{
    "policyRule": {
        "if": {
            "value": "[substring(field('name'), 0, 3)]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

La regla de directiva del ejemplo anterior usa [substring()](../../../azure-resource-manager/resource-group-template-functions-string.md#substring) para comparar los tres primeros caracteres de **nombre** a **abc**. Si **nombre** tenga menos de tres caracteres, el `substring()` función produce un error. Este error hace que la directiva para convertirse en un **denegar** efecto.

En su lugar, use el [if()](../../../azure-resource-manager/resource-group-template-functions-logical.md#if) función para comprobar si los tres primeros caracteres de **nombre** igual **abc** sin permitir que un **nombre** menor que tres caracteres para producir un error:

```json
{
    "policyRule": {
        "if": {
            "value": "[if(greaterOrEquals(length(field('name')), 3), substring(field('name'), 0, 3), 'not starting with abc')]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

Con la regla de directiva revisada `if()` comprueba la longitud del **nombre** antes de intentar obtener un `substring()` en un valor con menos de tres caracteres. Si **nombre** es demasiado corto, se devuelve en su lugar y en comparación con el valor "no se inicia con abc" **abc**. Un recurso con un nombre corto que no comienza por **abc** sigue sin funcionar la regla de directivas, pero ya no se produce un error durante la evaluación.

### <a name="effect"></a>Efecto

Directiva de Azure admite los siguientes tipos de efecto:

- **Deny**: genera un evento en el registro de actividad y genera un error en la solicitud.
- **Audit**: genera un evento de advertencia en el registro de actividad pero no genera un error en la solicitud.
- **Append**: agrega el conjunto de campos definido a la solicitud.
- **AuditIfNotExists**: habilita la auditoría si un recurso no existe.
- **DeployIfNotExists**: implementa un recurso si todavía no existe.
- **Disabled**: no se evalúa el cumplimiento de la regla de directivas en los recursos.

En el caso de **append**, debe proporcionar los detalles tal y como se muestra a continuación:

```json
"effect": "append",
"details": [{
    "field": "field name",
    "value": "value of the field"
}]
```

El valor puede ser una cadena o un objeto con formato JSON.

**AuditIfNotExists** y **DeployIfNotExists** evalúan la existencia de un recurso relacionado y aplican una regla. Si el recurso no coincide con la regla, se implementa el efecto. Por ejemplo, puede requerir que un monitor de red se implemente para todas las redes virtuales. Para obtener más información, vea el ejemplo [Auditar si la extensión no existe](../samples/audit-ext-not-exist.md).

El efecto de **DeployIfNotExists** requiere la propiedad **roleDefinitionId** en la parte **details** de la regla de directivas. Para obtener más información, vea [Remediation - Configure policy definition](../how-to/remediate-resources.md#configure-policy-definition) (Corrección: configurar la definición de directiva).

```json
"details": {
    ...
    "roleDefinitionIds": [
        "/subscription/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
        "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
    ]
}
```

Para obtener información detallada sobre cada efecto, el orden de evaluación, propiedades y ejemplos, vea [Understanding Azure directiva efectos](effects.md).

### <a name="policy-functions"></a>Funciones de directiva

Todos los [funciones de plantilla de Resource Manager](../../../azure-resource-manager/resource-group-template-functions.md) están disponibles para su uso dentro de una regla de directiva, excepto las siguientes funciones y las funciones definidas por el usuario:

- copyIndex()
- deployment()
- lista*
- newGuid()
- pickZones()
- providers()
- reference()
- resourceId()
- variables()

Las siguientes funciones están disponibles para usar en una regla de directiva, pero difieren de uso en una plantilla de Azure Resource Manager:

- addDays(dateTime, numberOfDaysToAdd)
  - **fecha y hora**: cadena [obligatorio] - cadena con el formato de fecha y hora de Universal ISO 8601 "aaaa-MM-ddTHH:mm:ss.fffffffZ'
  - **numberOfDaysToAdd**: integer [obligatorio] - número de días que desea agregar
- utcNow() - a diferencia de un administrador de recursos de plantilla, se puede usar fuera defaultValue.
  - Devuelve una cadena que se establece en la fecha y hora actuales en formato de fecha y hora de Universal ISO 8601 "aaaa-MM-ddTHH:mm:ss.fffffffZ'

Además, la función `field` está disponible para las reglas de directiva. `field` se usa principalmente con **AuditIfNotExists** y **DeployIfNotExists** para hacer referencia a los campos del recurso que se van a evaluar. Este uso se puede observar en el [ejemplo de DeployIfNotExists](effects.md#deployifnotexists-example).

#### <a name="policy-function-example"></a>Ejemplo de función de directiva

Este ejemplo de regla de directiva usa la función de recurso `resourceGroup` para obtener la propiedad **nombre**, combinada con la matriz `concat` y la función de objeto para compilar una condición `like` que exige que el nombre del recurso empiece con el nombre del grupo de recursos.

```json
{
    "if": {
        "not": {
            "field": "name",
            "like": "[concat(resourceGroup().name,'*')]"
        }
    },
    "then": {
        "effect": "deny"
    }
}
```

## <a name="aliases"></a>Alias

Los alias de propiedad se usan para tener acceso a propiedades específicas de un tipo de recurso. Los alias le permiten restringir los valores o condiciones que se permiten para una propiedad en un recurso. Cada alias se asigna a las rutas de acceso en las diferentes versiones de la API para un tipo de recurso determinado. Durante la evaluación de directivas, el motor de directiva obtiene la ruta de acceso de propiedad para esa versión de la API.

La lista de alias siempre está en aumento. Para descubrir qué alias son compatibles actualmente con Azure Policy, use uno de los métodos siguientes:

- Azure PowerShell

  ```azurepowershell-interactive
  # Login first with Connect-AzAccount if not using Cloud Shell

  # Use Get-AzPolicyAlias to list available providers
  Get-AzPolicyAlias -ListAvailable

  # Use Get-AzPolicyAlias to list aliases for a Namespace (such as Azure Automation -- Microsoft.Automation)
  Get-AzPolicyAlias -NamespaceMatch 'automation'
  ```

- Azure CLI

  ```azurecli-interactive
  # Login first with az login if not using Cloud Shell

  # List namespaces
  az provider list --query [*].namespace

  # Get Azure Policy aliases for a specific Namespace (such as Azure Automation -- Microsoft.Automation)
  az provider show --namespace Microsoft.Automation --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
  ```

- API de REST/ARMClient

  ```http
  GET https://management.azure.com/providers/?api-version=2017-08-01&$expand=resourceTypes/aliases
  ```

### <a name="understanding-the--alias"></a>Descripción del alias [*]

Algunos de los alias disponibles tienen una versión que aparece como un nombre "normal" y otra que tiene agregado **[\*]**. Por ejemplo:

- `Microsoft.Storage/storageAccounts/networkAcls.ipRules`
- `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]`

El alias 'normal' representa el campo como un valor único. Este campo es para escenarios de comparación de coincidencia exacta cuando todo el conjunto de valores debe ser exactamente como se define, nada más y nada menos.

El **[\*]** alias hace posible que se compara con el valor de cada elemento de la matriz y propiedades específicas de cada elemento. Este enfoque permite comparar propiedades de elemento de 'Si ninguno de', 'Si cualquiera de', o ' si todos los de ' escenarios. Mediante **ipRules [\*]**, un ejemplo sería validar que cada _acción_ es _Deny_, pero no preocuparse por qué la dirección IP ocuántasreglasexisten_valor_ es. Esta regla de ejemplo busca las coincidencias de **ipRules [\*] .value** a **10.0.4.1** y aplica la **effectType** solo si no encuentra al menos una coincidencia:

```json
"policyRule": {
    "if": {
        "allOf": [
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
                "exists": "true"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value",
                "notEquals": "10.0.4.1"
            }
        ]
    },
    "then": {
        "effect": "[parameters('effectType')]"
    }
}
```

Para obtener más información, consulte [evaluar el [\*] alias](../how-to/author-policies-for-arrays.md#evaluating-the--alias).

## <a name="initiatives"></a>Iniciativas

Las iniciativas le permiten agrupan varias definiciones de directivas relacionadas para simplificar las asignaciones y la administración, porque se trabaja con un grupo como un elemento único. Por ejemplo, puede agrupar las definiciones de directivas de etiquetado relacionadas en una sola iniciativa. En lugar de asignar individualmente cada directiva, la aplica.

En el ejemplo siguiente se muestra cómo crear una iniciativa para controlar dos etiquetas: `costCenter` y `productName`. Usa dos directivas integradas para aplicar el valor de etiqueta predeterminado.

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                }
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                }
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    },
    "id": "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policySetDefinitions/billingTagsPolicy",
    "type": "Microsoft.Authorization/policySetDefinitions",
    "name": "billingTagsPolicy"
}
```

## <a name="next-steps"></a>Pasos siguientes

- Revise los ejemplos en [ejemplos de Azure Policy](../samples/index.md).
- Vea la [Descripción de los efectos de directivas](effects.md).
- Comprender cómo [crear mediante programación las directivas](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/getting-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- Compruebe que un grupo de administración con [organizar los recursos con grupos de administración de Azure](../../management-groups/overview.md).