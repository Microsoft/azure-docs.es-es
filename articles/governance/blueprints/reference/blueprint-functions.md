---
title: Funciones de Azure Blueprints
description: Describe las funciones disponibles que se pueden utilizar con artefactos de plano técnico en las definiciones y asignaciones de Azure Blueprints.
ms.date: 08/27/2020
ms.topic: reference
ms.openlocfilehash: fe984ee7664b0d50fb891d946f9f40a200ccce09
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050767"
---
# <a name="functions-for-use-with-azure-blueprints"></a>Funciones para usar con Azure Blueprints

Azure Blueprints proporciona funciones que aportan dinamismo a la definición de los planos técnicos. Estas funciones se usan con las definiciones y los artefactos de los planos técnicos. Un artefacto de la plantilla de Azure Resource Manager (plantilla de ARM) admite el uso completo de las funciones de Resource Manager, además de obtener un valor dinámico mediante un parámetro de plano técnico.

Se admiten las siguientes funciones:

- [artifacts](#artifacts)
- [concat](#concat)
- [parameters](#parameters)
- [resourceGroup](#resourcegroup)
- [resourceGroups](#resourcegroups)
- [suscripción](#subscription)

## <a name="artifacts"></a>artifacts

`artifacts(artifactName)`

Devuelve un objeto de propiedades rellenadas con los resultados de los artefactos de ese plano técnico.

> [!NOTE]
> No se puede usar la función `artifacts()` desde dentro de una plantilla de ARM. La función solo se puede usar en el JSON de definición de plano técnico o en el JSON del artefacto al administrar el plano técnico con Azure PowerShell o la API REST como parte de [planos técnicos como código](https://github.com/Azure/azure-blueprints/blob/master/README.md).

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| artifactName |Sí |string |El nombre de un artefacto de plano técnico. |

### <a name="return-value"></a>Valor devuelto

Un objeto de propiedades de salida. Las propiedades **outputs** dependen del tipo de artefacto de plano técnico al que se hace referencia. Todos los tipos tienen este formato:

```json
{
  "outputs": {collectionOfOutputProperties}
}
```

#### <a name="policy-assignment-artifact"></a>Artefacto de la asignación de directiva

```json
{
    "outputs": {
        "policyAssignmentId": "{resourceId-of-policy-assignment}",
        "policyAssignmentName": "{name-of-policy-assignment}",
        "policyDefinitionId": "{resourceId-of-policy-definition}",
    }
}
```

#### <a name="arm-template-artifact"></a>Artefacto de plantilla de ARM

Las propiedades **outputs** del objeto devuelto se definen en la plantilla de ARM y las devuelve la implementación.

#### <a name="role-assignment-artifact"></a>Artefacto de asignación de roles

```json
{
    "outputs": {
        "roleAssignmentId": "{resourceId-of-role-assignment}",
        "roleDefinitionId": "{resourceId-of-role-definition}",
        "principalId": "{principalId-role-is-being-assigned-to}",
    }
}
```

### <a name="example"></a>Ejemplo

Un artefacto de plantilla de ARM con el identificador _myTemplateArtifact_ que contenga la siguiente propiedad de salida de ejemplo:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    ...
    "outputs": {
        "myArray": {
            "type": "array",
            "value": ["first", "second"]
        },
        "myString": {
            "type": "string",
            "value": "my string value"
        },
        "myObject": {
            "type": "object",
            "value": {
                "myProperty": "my value",
                "anotherProperty": true
            }
        }
    }
}
```

Algunos ejemplos de recuperación de datos de la plantilla _myTemplateArtifact_ son los siguientes:

| Expression | Tipo | Value |
|:---|:---|:---|
|`[artifacts("myTemplateArtifact").outputs.myArray]` | Array | \["first", "second"\] |
|`[artifacts("myTemplateArtifact").outputs.myArray[0]]` | String | "first" |
|`[artifacts("myTemplateArtifact").outputs.myString]` | String | "my string value" |
|`[artifacts("myTemplateArtifact").outputs.myObject]` | Object | { "myproperty": "my value", "anotherProperty": true } |
|`[artifacts("myTemplateArtifact").outputs.myObject.myProperty]` | String | "my value" |
|`[artifacts("myTemplateArtifact").outputs.myObject.anotherProperty]` | Bool | True |

## <a name="concat"></a>concat

`concat(string1, string2, string3, ...)`

Combina varios valores de cadena y devuelve la cadena concatenada.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| string1 |Sí |string |El primer valor para la concatenación. |
| argumentos adicionales |No |string |Valores adicionales en orden secuencial para la concatenación |

### <a name="return-value"></a>Valor devuelto

Una cadena de valores concatenados.

### <a name="remarks"></a>Observaciones

La función de Azure Blueprints difiere de la función de la plantilla de ARM en que solo funciona con cadenas.

### <a name="example"></a>Ejemplo

`concat(parameters('organizationName'), '-vm')`

## <a name="parameters"></a>parámetros

`parameters(parameterName)`

Devuelve un valor de parámetro de plano técnico. El nombre del parámetro especificado debe estar definido en la definición o en los artefactos del plano técnico.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| parameterName |Sí |string |El nombre del parámetro que se va a devolver. |

### <a name="return-value"></a>Valor devuelto

El valor del parámetro del plano técnico o del artefacto de plano técnico especificado.

### <a name="remarks"></a>Observaciones

La función de Azure Blueprints difiere de la función de la plantilla de ARM en que solo funciona con parámetro de plano técnico.

### <a name="example"></a>Ejemplo

Defina el parámetro _principalIds_ en la definición del plano técnico:

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "parameters": {
            "principalIds": {
                "type": "array",
                "metadata": {
                    "displayName": "Principal IDs",
                    "description": "This is a blueprint parameter that any artifact can reference. We'll display these descriptions for you in the info bubble. Supply principal IDs for the users,groups, or service principals for the Azure role assignment.",
                    "strongType": "PrincipalId"
                }
            }
        },
        ...
    }
}
```

Después use _principalIds_ como argumento para `parameters()` en un artefacto de plano técnico:

```json
{
    "type": "Microsoft.Blueprint/blueprints/artifacts",
    "kind": "roleAssignment",
    ...
    "properties": {
        "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
        "principalIds": "[parameters('principalIds')]",
        ...
    }
}
```

## <a name="resourcegroup"></a>resourceGroup

`resourceGroup()`

Devuelve un objeto que representa el grupo de recursos actual.

### <a name="return-value"></a>Valor devuelto

El objeto devuelto está en el formato siguiente:

```json
{
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
}
```

### <a name="remarks"></a>Observaciones

La función de Azure Blueprints difiere de la función de la plantilla de ARM. La función `resourceGroup()` no se puede usar en un artefacto de nivel de suscripción ni en la definición del plano técnico. Solo puede usarse en artefactos de plano técnico que forman parte de un artefacto del grupo de recursos.

La función `resourceGroup()` acostumbra a usarse para crear recursos en la misma ubicación que artefacto del grupo de recursos.

### <a name="example"></a>Ejemplo

Para usar como ubicación de otro artefacto la ubicación del grupo de recursos, establecida en la definición del plano técnico o durante la asignación, indique un objeto de marcador de posición del grupo de recursos en la definición del plano técnico. En este ejemplo, _NetworkingPlaceholder_ es el nombre del marcador de posición del grupo de recursos.

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "resourceGroups": {
            "NetworkingPlaceholder": {
                "location": "eastus"
            }
        }
    }
}
```

Después utilice la función `resourceGroup()` en el contexto de un artefacto de plano técnico destinado a un objeto de marcador de posición de grupo de recursos. En este ejemplo, el artefacto de la plantilla se implementa en el grupo de recursos _NetworkingPlaceholder_ y proporciona el parámetro _resourceLocation_ que se rellena dinámicamente con la ubicación del grupo de recursos  _NetworkingPlaceholder_ en la plantilla. La ubicación del grupo de recursos _NetworkingPlaceholder_ puede haberse definido estáticamente en la definición del plano técnico o dinámicamente durante la asignación. En cualquier caso, el artefacto de la plantilla recibe esa información como un parámetro y lo usa para implementar los recursos en la ubicación correcta.

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "template",
  "properties": {
      "template": {
        ...
      },
      "resourceGroup": "NetworkingPlaceholder",
      ...
      "parameters": {
        "resourceLocation": {
          "value": "[resourceGroup().location]"
        }
      }
  }
}
```

## <a name="resourcegroups"></a>resourceGroups

`resourceGroups(placeholderName)`

Devuelve un objeto que representa el artefacto del grupo de recursos especificado. A diferencia de `resourceGroup()`, que requiere el contexto del artefacto, esta función se utiliza para obtener las propiedades de un marcador de posición de grupo de recursos específico cuando no esté en el contexto de ese grupo de recursos.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| placeholderName |Sí |string |El nombre de marcador de posición del artefacto del grupo de recursos que se va a devolver. |

### <a name="return-value"></a>Valor devuelto

El objeto devuelto está en el formato siguiente:

```json
{
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
}
```

### <a name="example"></a>Ejemplo

Para usar como ubicación de otro artefacto la ubicación del grupo de recursos, establecida en la definición del plano técnico o durante la asignación, indique un objeto de marcador de posición del grupo de recursos en la definición del plano técnico. En este ejemplo, _NetworkingPlaceholder_ es el nombre del marcador de posición del grupo de recursos.

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "resourceGroups": {
            "NetworkingPlaceholder": {
                "location": "eastus"
            }
        }
    }
}
```

Después utilice la función `resourceGroups()` en el contexto de un artefacto de plano técnico para obtener una referencia al objeto de marcador de posición de grupo de recursos. En este ejemplo, el artefacto de la plantilla se implementa fuera del grupo de recursos _NetworkingPlaceholder_ y proporciona el parámetro _artifactLocation_ que se rellena dinámicamente con la ubicación del grupo de recursos  _NetworkingPlaceholder_ en la plantilla. La ubicación del grupo de recursos _NetworkingPlaceholder_ puede haberse definido estáticamente en la definición del plano técnico o dinámicamente durante la asignación. En cualquier caso, el artefacto de la plantilla recibe esa información como un parámetro y lo usa para implementar los recursos en la ubicación correcta.

```json
{
  "kind": "template",
  "properties": {
      "template": {
          ...
      },
      ...
      "parameters": {
        "artifactLocation": {
          "value": "[resourceGroups('NetworkingPlaceholder').location]"
        }
      }
  },
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "name": "myTemplate"
}
```

## <a name="subscription"></a>subscription

`subscription()`

Devuelve detalles sobre la suscripción para la asignación actual del plano técnico.

### <a name="return-value"></a>Valor devuelto

El objeto devuelto está en el formato siguiente:

```json
{
    "id": "/subscriptions/{subscriptionId}",
    "subscriptionId": "{subscriptionId}",
    "tenantId": "{tenantId}",
    "displayName": "{name-of-subscription}"
}
```

### <a name="example"></a>Ejemplo

Utilice el nombre para mostrar de la suscripción y la función `concat()` para crear una convención de nomenclatura que pasa como el parámetro _resourceName_ al artefacto de la plantilla.

```json
{
  "kind": "template",
  "properties": {
      "template": {
          ...
      },
      ...
      "parameters": {
        "resourceName": {
          "value": "[concat(subscription().displayName, '-vm')]"
        }
      }
  },
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "name": "myTemplate"
}
```

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.