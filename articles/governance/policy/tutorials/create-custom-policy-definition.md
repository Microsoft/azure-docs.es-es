---
title: 'Tutorial: Creación de una definición de directiva personalizada'
description: En este tutorial, creará una definición de directiva personalizada para Azure Policy para aplicar reglas de negocio personalizadas a los recursos de Azure.
ms.date: 06/16/2020
ms.topic: tutorial
ms.openlocfilehash: 72282cbc5ed1877cf3f61b792235e8dc2f72fffe
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89649825"
---
# <a name="tutorial-create-a-custom-policy-definition"></a>Tutorial: Creación de una definición de directiva personalizada

Una definición de directiva personalizada permite a los clientes definir sus propias reglas para usar Azure. Estas reglas exigen a menudo:

- Prácticas de seguridad
- Administración de costos
- Reglas específicas de la organización (como nombres o ubicaciones)

Sea cual sea el factor empresarial para crear una directiva personalizada, los pasos para definir la nueva directiva personalizada son los mismos.

Antes de crear una directiva personalizada, consulte los [ejemplos de directivas](../samples/index.md) para ver si ya existe una directiva que coincida con sus necesidades.

El enfoque para crear una directiva personalizada sigue estos pasos:

> [!div class="checklist"]
> - Identificar los requisitos empresariales
> - Asignar a cada requisito una propiedad de recurso de Azure
> - Asignar a la propiedad un alias
> - Determinar qué efecto usar
> - Elaborar la definición de directiva

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="identify-requirements"></a>Identificación de los requisitos

Antes de crear la definición de directiva, es importante entender su intención. En este tutorial, se va a usar un requisito de seguridad empresarial común como objetivo para ilustrar los pasos implicados:

- Cada cuenta de almacenamiento debe estar habilitada para HTTPS.
- Cada cuenta de almacenamiento debe estar deshabilitada para HTTP.

Los requisitos deben identificar claramente los estados de los recursos "será" y "no será".

Aunque se ha definido el estado esperado del recurso, no se ha definido aún lo que se quiere hacer con los recursos no compatibles. Azure Policy admite una serie de [efectos](../concepts/effects.md). En este tutorial, el requisito empresarial se define de este modo: impedir la creación de recursos si no son compatibles con las reglas de negocio. Para alcanzar este objetivo, se usará el efecto [Denegar](../concepts/effects.md#deny). También se desea la opción para suspender la directiva en el caso de asignaciones específicas. Por lo tanto, se usará el efecto [Deshabilitado](../concepts/effects.md#disabled) y el efecto se convertirá en un [parámetro](../concepts/definition-structure.md#parameters) en la definición de directiva.

## <a name="determine-resource-properties"></a>Determinación de las propiedades de recursos

En función de las necesidades de la organización, el recurso de Azure que se va a auditar con Azure Policy puede ser una cuenta de almacenamiento. Sin embargo, se desconocen las propiedades que se usarán en la definición de directiva. Para realizar la evaluación, Azure Policy utiliza la representación JSON del recurso, por lo que es necesario conocer las propiedades disponibles en dicho recurso.

Hay muchas maneras de determinar las propiedades de un recurso de Azure. Se analizará cada una de ellas en este tutorial:

- Extensión de Azure Policy para VS Code
- Plantillas de Azure Resource Manager (plantillas de ARM)
  - Exportar un recurso existente
  - Experiencia de creación
  - Plantillas de inicio rápido (GitHub)
  - Documentos de referencia de plantilla
- Azure Resource Explorer

### <a name="view-resources-in-vs-code-extension"></a>Visualización de recursos en la extensión de VS Code

La [extensión de VS Code](../how-to/extension-for-vscode.md#search-for-and-view-resources) se puede usar para examinar los recursos de su entorno y ver las propiedades de Resource Manager en cada recurso.

### <a name="arm-templates"></a>Plantillas de ARM

Hay varias maneras de examinar una [plantilla de Resource Manager](../../../azure-resource-manager/templates/template-tutorial-use-template-reference.md) que incluye la propiedad que quiere administrar.

#### <a name="existing-resource-in-the-portal"></a>Recurso existente en el portal

La manera más sencilla de buscar propiedades es examinar un recurso existente del mismo tipo. Los recursos que ya haya configurado con el valor que quiere aplicar también proporcionan el valor con el que comparar.
Examine ese recurso concreto en la página **Exportar plantilla** (en **Configuración**) de Azure Portal.

:::image type="content" source="../media/create-custom-policy-definition/export-template.png" alt-text="Captura de pantalla de la página Exportar plantilla de un recurso existente en Azure Portal." border="false":::

Al hacer esto mismo con una cuenta de almacenamiento se revela una plantilla similar a la de este ejemplo:

```json
...
"resources": [{
    "comments": "Generalized from resource: '/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount'.",
    "type": "Microsoft.Storage/storageAccounts",
    "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
    },
    "kind": "Storage",
    "name": "[parameters('storageAccounts_mystorageaccount_name')]",
    "apiVersion": "2018-07-01",
    "location": "westus",
    "tags": {
        "ms-resource-usage": "azure-cloud-shell"
    },
    "scale": null,
    "properties": {
        "networkAcls": {
            "bypass": "AzureServices",
            "virtualNetworkRules": [],
            "ipRules": [],
            "defaultAction": "Allow"
        },
        "supportsHttpsTrafficOnly": false,
        "encryption": {
            "services": {
                "file": {
                    "enabled": true
                },
                "blob": {
                    "enabled": true
                }
            },
            "keySource": "Microsoft.Storage"
        }
    },
    "dependsOn": []
}]
...
```

En **propiedades** hay un valor llamado **supportsHttpsTrafficOnly** establecido en **false**. Esta propiedad se parece a la que se está buscando. Además, el **tipo** del recurso es **Microsoft.Storage/storageAccounts**. El tipo nos permite limitar la directiva a solo los recursos de este tipo.

#### <a name="create-a-resource-in-the-portal"></a>Creación de un recurso en el portal

Otra manera mediante el portal es la experiencia de creación de recursos. Al crear una cuenta de almacenamiento mediante el portal, una opción en la pestaña **Avanzadas** es **Security transfer required** (Transferencia de seguridad necesaria). Esta propiedad tiene las opciones _Disabled_ (Deshabilitado) y _Enabled_ (Habilitado). El icono de información tiene texto adicional que confirma que esta opción es probablemente la propiedad que busca. Sin embargo, el portal no dice el nombre de la propiedad en esta pantalla.

En la pestaña **Revisar y crear**, hay un vínculo en la parte inferior de la página a **Descargar una plantilla para la automatización**. Al seleccionar el vínculo se abre la plantilla que crea el recurso que se ha configurado. En este caso, se pueden ver dos trozos principales de información:

```json
...
"supportsHttpsTrafficOnly": {
    "type": "bool"
}
...
"properties": {
    "accessTier": "[parameters('accessTier')]",
    "supportsHttpsTrafficOnly": "[parameters('supportsHttpsTrafficOnly')]"
}
...
```

Esta información indica el tipo de propiedad y también confirma que **supportsHttpsTrafficOnly** es la propiedad que se busca.

#### <a name="quickstart-templates-on-github"></a>Plantillas de inicio rápido de GitHub

Las [plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates) en GitHub contienen cientos de plantillas de Resource Manager creadas para distintos recursos. Estas plantillas pueden ser una excelente manera de encontrar la propiedad de recurso que se busca. Puede que algunas propiedades parezcan ser lo que busca, sin embargo, es necesario controlar algo más.

#### <a name="resource-reference-docs"></a>Documentos de referencia de recursos

Para validar que **supportsHttpsTrafficOnly** es la propiedad correcta, compruebe la referencia de la plantilla de Resource Manager correspondiente al [recurso de cuenta de almacenamiento](/azure/templates/microsoft.storage/2018-07-01/storageaccounts) en el proveedor de almacenamiento. El objeto de propiedades tiene una lista de parámetros válidos. Al seleccionar el vínculo [StorageAccountPropertiesCreateParameters-object](/azure/templates/microsoft.storage/2018-07-01/storageaccounts#storageaccountpropertiescreateparameters-object) se muestra una tabla de propiedades aceptables. **supportsHttpsTrafficOnly** existe y la descripción coincide con lo que se busca para satisfacer los requisitos empresariales.

### <a name="azure-resource-explorer"></a>Azure Resource Explorer

Otra forma de explorar los recursos de Azure es mediante [Azure Resource Explorer](https://resources.azure.com) (versión preliminar). Esta herramienta usa el contexto de su suscripción, así que debe autenticarse en el sitio web con sus credenciales de Azure. Una vez autenticado, puede examinar por proveedores, suscripciones, grupos de recursos y recursos.

Busque un recurso de cuenta de almacenamiento y examine las propiedades. Aquí también se puede ver la propiedad **supportsHttpsTrafficOnly**. Al seleccionar la pestaña **Documentación**, se puede ver que la descripción de la propiedad coincide con lo que se ha encontrado en los documentos de referencia anteriormente.

## <a name="find-the-property-alias"></a>Búsqueda del alias de propiedad

Se ha identificado la propiedad de recurso, pero es necesario asignarle un [alias](../concepts/definition-structure.md#aliases).

Hay varias maneras de determinar los alias de un recurso de Azure. Se analizará cada una de ellas en este tutorial:

- Extensión de Azure Policy para VS Code
- Azure CLI
- Azure PowerShell
- Azure Resource Graph

### <a name="get-aliases-in-vs-code-extension"></a>Obtención de alias en la extensión de VS Code

La extensión de Azure Policy para la extensión de VS Code facilita el examen de los recursos y la [detección de alias](../how-to/extension-for-vscode.md#discover-aliases-for-resource-properties).

> [!NOTE]
> La extensión de VS Code solo expone las propiedades del modo Administrador de recursos y no muestra ninguna propiedad del modo [Proveedor de recursos](../concepts/definition-structure.md#mode).

### <a name="azure-cli"></a>Azure CLI

En la CLI de Azure, el grupo de comandos `az provider` se usa para buscar alias de recursos. Se va a filtrar por el espacio de nombres **Microsoft.Storage** según los detalles que se tienen anteriormente sobre el recurso de Azure.

```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Get Azure Policy aliases for type Microsoft.Storage
az provider show --namespace Microsoft.Storage --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
```

En los resultados, se puede ver un alias compatible con las cuentas de almacenamiento llamado **supportsHttpsTrafficOnly**. La existencia de este alias significa que se puede escribir la directiva para aplicar los requisitos empresariales.

### <a name="azure-powershell"></a>Azure PowerShell

En Azure PowerShell, el cmdlet `Get-AzPolicyAlias` se usa para buscar los alias de recurso. Se va a filtrar por el espacio de nombres **Microsoft.Storage** según los detalles que se tienen anteriormente sobre el recurso de Azure.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Use Get-AzPolicyAlias to list aliases for Microsoft.Storage
(Get-AzPolicyAlias -NamespaceMatch 'Microsoft.Storage').Aliases
```

Al igual que con la CLI de Azure, los resultados muestran un alias compatible con las cuentas de almacenamiento llamado **supportsHttpsTrafficOnly**.

### <a name="azure-resource-graph"></a>Azure Resource Graph

[Azure Resource Graph](../../resource-graph/overview.md) es un servicio que proporciona otro método para buscar las propiedades de los recursos de Azure. A continuación se describe una consulta sencilla para buscar una única cuenta de almacenamiento con Resource Graph:

```kusto
Resources
| where type=~'microsoft.storage/storageaccounts'
| limit 1
```

```azurecli-interactive
az graph query -q "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1"
```

Los resultados son similares a lo que se puede ver en las plantillas de Resource Manager y mediante Azure Resource Explorer. Sin embargo, los resultados de Azure Resource Graph también pueden incluir información del [alias](../concepts/definition-structure.md#aliases)_proyectando_ la matriz _aliases_:

```kusto
Resources
| where type=~'microsoft.storage/storageaccounts'
| limit 1
| project aliases
```

```azurecli-interactive
az graph query -q "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1 | project aliases"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1 | project aliases"
```

Esta es una salida de ejemplo de una cuenta de almacenamiento de alias:

```json
"aliases": {
    "Microsoft.Storage/storageAccounts/accessTier": null,
    "Microsoft.Storage/storageAccounts/accountType": "Standard_LRS",
    "Microsoft.Storage/storageAccounts/enableBlobEncryption": true,
    "Microsoft.Storage/storageAccounts/enableFileEncryption": true,
    "Microsoft.Storage/storageAccounts/encryption": {
        "keySource": "Microsoft.Storage",
        "services": {
            "blob": {
                "enabled": true,
                "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
            },
            "file": {
                "enabled": true,
                "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
            }
        }
    },
    "Microsoft.Storage/storageAccounts/encryption.keySource": "Microsoft.Storage",
    "Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyname": null,
    "Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyvaulturi": null,
    "Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyversion": null,
    "Microsoft.Storage/storageAccounts/encryption.services": {
        "blob": {
            "enabled": true,
            "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
        },
        "file": {
            "enabled": true,
            "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
        }
    },
    "Microsoft.Storage/storageAccounts/encryption.services.blob": {
        "enabled": true,
        "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
    },
    "Microsoft.Storage/storageAccounts/encryption.services.blob.enabled": true,
    "Microsoft.Storage/storageAccounts/encryption.services.file": {
        "enabled": true,
        "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
    },
    "Microsoft.Storage/storageAccounts/encryption.services.file.enabled": true,
    "Microsoft.Storage/storageAccounts/networkAcls": {
        "bypass": "AzureServices",
        "defaultAction": "Allow",
        "ipRules": [],
        "virtualNetworkRules": []
    },
    "Microsoft.Storage/storageAccounts/networkAcls.bypass": "AzureServices",
    "Microsoft.Storage/storageAccounts/networkAcls.defaultAction": "Allow",
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules": [],
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]": [],
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action": [],
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*]": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].action": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].id": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].state": [],
    "Microsoft.Storage/storageAccounts/primaryEndpoints": {
        "blob": "https://mystorageaccount.blob.core.windows.net/",
        "file": "https://mystorageaccount.file.core.windows.net/",
        "queue": "https://mystorageaccount.queue.core.windows.net/",
        "table": "https://mystorageaccount.table.core.windows.net/"
    },
    "Microsoft.Storage/storageAccounts/primaryEndpoints.blob": "https://mystorageaccount.blob.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.file": "https://mystorageaccount.file.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.queue": "https://mystorageaccount.queue.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.table": "https://mystorageaccount.table.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.web": null,
    "Microsoft.Storage/storageAccounts/primaryLocation": "eastus2",
    "Microsoft.Storage/storageAccounts/provisioningState": "Succeeded",
    "Microsoft.Storage/storageAccounts/sku.name": "Standard_LRS",
    "Microsoft.Storage/storageAccounts/sku.tier": "Standard",
    "Microsoft.Storage/storageAccounts/statusOfPrimary": "available",
    "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly": false
}
```

Azure Resource Graph se usa mediante [Cloud Shell](https://shell.azure.com), lo que hace que sea una manera rápida y fácil de explorar las propiedades de los recursos.

## <a name="determine-the-effect-to-use"></a>Determinación del efecto para usar

Decidir qué hacer con los recursos no compatibles es casi tan importante como decidir qué se debe evaluar en primer lugar. Cada posible respuesta a un recurso no compatible se conoce como [efecto](../concepts/effects.md). Los efectos controlan si se registra o bloquea el recurso no compatible, si tiene datos anexados o si tiene una implementación asociada para devolver el recurso a un estado de compatibilidad.

En nuestro ejemplo, la denegación es el efecto que se desea ya que no queremos tener recursos no compatibles creados en el entorno de Azure. Auditar es una buena primera opción de efecto de directiva para determinar de qué modo afecta una directiva antes de establecerla en Denegar. Una manera de facilitar el cambio del efecto por asignación es parametriza el efecto. Consulte a continuación los [parámetros](#parameters) para saber cómo hacerlo.

## <a name="compose-the-definition"></a>Elaboración de la definición

Ya se tienen los detalles de la propiedad y el alias de lo que se planea administrar. El siguiente paso es elaborar la propia regla de directiva. Si aún no está familiarizado con el lenguaje de directiva, consulte la [estructura de definición de directivas](../concepts/definition-structure.md) para saber cómo estructurar la definición de directiva. Esta es una plantilla vacía del aspecto de una definición de directiva:

```json
{
    "properties": {
        "displayName": "<displayName>",
        "description": "<description>",
        "mode": "<mode>",
        "parameters": {
                <parameters>
        },
        "policyRule": {
            "if": {
                <rule>
            },
            "then": {
                "effect": "<effect>"
            }
        }
    }
}
```

### <a name="metadata"></a>Metadatos

Los tres primeros componentes son metadatos de la directiva. Es fácil proporcionar valores para estos componentes dado que ya se sabe para lo que se está creando la regla. El [modo](../concepts/definition-structure.md#mode) consiste principalmente en las etiquetas y la ubicación del recurso. Puesto que no es necesario limitar la evaluación a los recursos que admiten etiquetas, se usará el valor _all_ para **mode**.

```json
"displayName": "Deny storage accounts not using only HTTPS",
"description": "Deny storage accounts not using only HTTPS. Checks the supportsHttpsTrafficOnly property on StorageAccounts.",
"mode": "all",
```

### <a name="parameters"></a>Parámetros

Aunque no se ha usado un parámetro para cambiar la evaluación, no queremos usar un parámetro para permitir cambiar el **efecto** para la solución de problemas. Se definirá un parámetro **effectType** y se limitará a solo **Deny** y **Disabled**. Estas dos opciones coinciden con nuestros requisitos empresariales. El bloque de parámetros finalizado es similar a este ejemplo:

```json
"parameters": {
    "effectType": {
        "type": "string",
        "defaultValue": "Deny",
        "allowedValues": [
            "Deny",
            "Disabled"
        ],
        "metadata": {
            "displayName": "Effect",
            "description": "Enable or disable the execution of the policy"
        }
    }
},
```

### <a name="policy-rule"></a>Regla de directiva

Elaborar la [regla de directiva](../concepts/definition-structure.md#policy-rule) es el paso final en la creación de la definición de directiva personalizada. Se han identificado dos instrucciones con las que realizar la prueba:

- Que el **tipo** de la cuenta de almacenamiento es **Microsoft.Storage/storageAccounts**
- Que la propiedad **supportsHttpsTrafficOnly** de la cuenta de almacenamiento no es **true**

Como es necesario que ambas instrucciones sean true, se va a usar el [operador lógico](../concepts/definition-structure.md#logical-operators) **allOf**. Se pasará el parámetro **effectType** al efecto en lugar de realizar una declaración estática. La regla finalizada se parece a la de este ejemplo:

```json
"if": {
    "allOf": [
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        },
        {
            "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
            "notEquals": "true"
        }
    ]
},
"then": {
    "effect": "[parameters('effectType')]"
}
```

### <a name="completed-definition"></a>Definición completada

Con las tres partes de la directiva definidas, esta es la definición completada:

```json
{
    "properties": {
        "displayName": "Deny storage accounts not using only HTTPS",
        "description": "Deny storage accounts not using only HTTPS. Checks the supportsHttpsTrafficOnly property on StorageAccounts.",
        "mode": "all",
        "parameters": {
            "effectType": {
                "type": "string",
                "defaultValue": "Deny",
                "allowedValues": [
                    "Deny",
                    "Disabled"
                ],
                "metadata": {
                    "displayName": "Effect",
                    "description": "Enable or disable the execution of the policy"
                }
            }
        },
        "policyRule": {
            "if": {
                "allOf": [
                    {
                        "field": "type",
                        "equals": "Microsoft.Storage/storageAccounts"
                    },
                    {
                        "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
                        "notEquals": "true"
                    }
                ]
            },
            "then": {
                "effect": "[parameters('effectType')]"
            }
        }
    }
}
```

La definición completada se puede usar para crear otra directiva. El portal y cada SDK (la CLI de Azure, Azure PowerShell y API REST) aceptan la definición de diferentes formas, así que revise los comandos de cada una para validar el uso correcto. A continuación, asígnela, con el efecto parametrizado, a los recursos adecuados para administrar la seguridad de sus cuentas de almacenamiento.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si terminó de trabajar con los recursos de este tutorial, use los pasos siguientes para eliminar todas las asignaciones y definiciones creadas anteriormente:

1. Seleccione **Definiciones** (o **Asignaciones** si trata de eliminar una asignación) en **Creación** en el lado izquierdo de la página de Azure Policy.

1. Busque la nueva definición de iniciativa o directiva (o asignación) que quiere quitar.

1. Haga clic con el botón derecho en la fila o seleccione los puntos suspensivos al final de la definición (o asignación) y elija **Eliminar definición** (o **Eliminar asignación** ).

## <a name="review"></a>Revisar

En este tutorial, ha realizado correctamente las tareas siguientes:

> [!div class="checklist"]
> - Ha identificado los requisitos empresariales
> - Ha asignado a cada requisito una propiedad de recurso de Azure
> - Ha asignado a la propiedad un alias
> - Ha determinado el efecto que se usará
> - Ha elaborado la definición de directiva

## <a name="next-steps"></a>Pasos siguientes

A continuación, usará su definición de directiva personalizada para crear y asignar una directiva:

> [!div class="nextstepaction"]
> [Creación y asignación de una definición de directiva](../how-to/programmatically-create.md#create-and-assign-a-policy-definition)
