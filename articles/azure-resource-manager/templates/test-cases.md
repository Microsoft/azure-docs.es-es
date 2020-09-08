---
title: Casos de prueba del kit de herramientas para pruebas
description: En este artículo se describen las pruebas que se ejecutan en el kit de herramientas para pruebas de plantillas de Resource Manager.
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: dda8e92c17029126e7f473a6aee03acfc970e04b
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378124"
---
# <a name="default-test-cases-for-arm-template-test-toolkit"></a>Casos de prueba predeterminados del kit de herramientas para pruebas de plantillas de Resource Manager

En este artículo se describen las pruebas predeterminadas que se ejecutan con el [kit de herramientas para pruebas de plantillas](test-toolkit.md). Proporciona ejemplos que ilustran la superación y no superación de las pruebas. También incluye el nombre cada prueba.

## <a name="use-correct-schema"></a>Uso del esquema correcto

Nombre de la prueba: **Esquema DeploymentTemplate correcto**

En la plantilla, debe especificar un valor de esquema válido.

En el ejemplo siguiente **se supera** esta prueba.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": []
}
```

La propiedad de esquema de la plantilla debe establecerse en uno de los esquemas siguientes:

* `https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json`

## <a name="parameters-must-exist"></a>Deben existir parámetros

Nombre de la prueba: **Debe existir la propiedad de parámetros**

La plantilla debe tener un elemento de parámetros. Los parámetros son esenciales para que las plantillas se puedan volver a usar en distintos entornos. Agregue parámetros a la plantilla para los valores que cambian cuando se implementan en otros entornos.

En el ejemplo siguiente **se supera** esta prueba:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "vmName": {
          "type": "string",
          "defaultValue": "linux-vm",
          "metadata": {
            "description": "Name for the Virtual Machine."
          }
      }
  },
  ...
```

## <a name="declared-parameters-must-be-used"></a>Los parámetros declarados deben usarse

Nombre de la prueba: **Debe hacerse referencia a los parámetros**

Para reducir la confusión en la plantilla, elimine los parámetros definidos, pero que no se usan. Esta prueba encuentra cualquier parámetro que no se use en ninguna parte de la plantilla. La eliminación de parámetros no usados también facilita la implementación de la plantilla, ya que no hay que proporcionar valores innecesarios.

## <a name="secure-parameters-cant-have-hardcoded-default"></a>Los parámetros seguros no pueden tener un valor predeterminado codificado de forma rígida

Nombre de la prueba: **Los parámetros de cadena segura no pueden tener valores predeterminados**

No proporcione un valor predeterminado codificado de forma rígida para un parámetro seguro en la plantilla. Se puede usar una cadena vacía como valor predeterminado.

Use los tipos **SecureString** o **SecureObject** en los parámetros que contienen valores confidenciales, como las contraseñas. Cuando un parámetro usa un tipo seguro, el valor del parámetro no se registra ni se almacena en el historial de implementación. Esta acción evita que un usuario malintencionado detecte el valor confidencial.

Sin embargo, cuando se proporciona un valor predeterminado, cualquiera que pueda acceder a la plantilla o al historial de implementación puede detectarlo.

En el ejemplo siguiente **no se supera** esta prueba:

```json
"parameters": {
    "adminPassword": {
        "defaultValue": "HardcodedPassword",
        "type": "SecureString"
    }
}
```

En el ejemplo siguiente **se supera** esta prueba:

```json
"parameters": {
    "adminPassword": {
        "type": "SecureString"
    }
}
```

## <a name="environment-urls-cant-be-hardcoded"></a>Las direcciones URL de entorno no pueden estar codificadas de forma rígida

Nombre de la prueba: **DeploymentTemplate no debe contener un URI codificado de forma rígida**

No codifique de forma rígida las direcciones URL de entorno en la plantilla. En su lugar, use la [función de entorno ](template-functions-deployment.md#environment) para obtener estas direcciones URL de forma dinámica durante la implementación. Para obtener una lista de los hosts de URL que están bloqueados, consulte el [caso de prueba](https://github.com/Azure/arm-ttk/blob/master/arm-ttk/testcases/deploymentTemplate/DeploymentTemplate-Must-Not-Contain-Hardcoded-Uri.test.ps1).

En el siguiente ejemplo **no se supera** esta prueba porque la URL está codificada de forma rígida.

```json
"variables":{
    "AzureURL":"https://management.azure.com"
}
```

La prueba también **genera un error** cuando se usa con [concat](template-functions-string.md#concat) o [uri](template-functions-string.md#uri).

```json
"variables":{
    "AzureSchemaURL1": "[concat('https://','gallery.azure.com')]",
    "AzureSchemaURL2": "[uri('gallery.azure.com','test')]"
}
```

En el ejemplo siguiente **se supera** esta prueba.

```json
"variables": {
    "AzureSchemaURL": "[environment().gallery]"
},
```

## <a name="location-uses-parameter"></a>La ubicación usa el parámetro

Nombre de la prueba: **La ubicación no debe estar codificada de forma rígida**

Es posible que los usuarios de la plantilla tengan a su disposición un número limitado de regiones. Al establecer la ubicación del recurso en `"[resourceGroup().location]"`, puede que el grupo de recursos se haya creado en una región a la que otros usuarios no puedan acceder. A esos usuarios se le impide usar la plantilla.

Al definir la ubicación de cada recurso, use un parámetro que tenga como valor predeterminado la ubicación del grupo de recursos. Al proporcionar este parámetro, los usuarios pueden usar el valor predeterminado cuando sea conveniente, pero también especificar otra ubicación.

En el siguiente ejemplo **no se supera** esta prueba porque la ubicación en el recurso está establecida en `resourceGroup().location`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[resourceGroup().location]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ]
}
```

En el ejemplo siguiente se usa un parámetro de ubicación, pero **no se supera** esta prueba porque el parámetro de ubicación tiene como valor predeterminado una ubicación codificada de forma rígida.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "westus"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[parameters('location')]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

En su lugar, cree un parámetro que tenga como valor predeterminado la ubicación del grupo de recursos, pero que permita a los usuarios proporcionar otro valor. En el ejemplo siguiente **se supera** esta prueba.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for the resources."
            }
        }
     },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[parameters('location')]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

## <a name="resources-should-have-location"></a>Los recursos deben tener ubicación

Nombre de la prueba: **Los recursos deben tener ubicación**

La ubicación de un recurso debe establecerse en una [expresión de plantilla](template-expressions.md) o en `global`. La expresión de plantilla usaría normalmente el parámetro de ubicación descrito en la prueba anterior.

En el siguiente ejemplo **no se supera** esta prueba porque la ubicación no es una expresión o `global`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "westus",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

En el ejemplo siguiente **se supera** esta prueba.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Maps/accounts",
            "apiVersion": "2020-02-01-preview",
            "name": "demoMap",
            "location": "global",
            "sku": {
                "name": "S0"
            }
        }
    ],
    "outputs": {
    }
}
```

En el ejemplo siguiente también **se supera** esta prueba.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for the resources."
            }
        }
     },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[parameters('location')]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

## <a name="vm-size-uses-parameter"></a>El tamaño de la máquina virtual usa un parámetro

Nombre de la prueba: **El tamaño de la máquina virtual debe ser un parámetro**

No codifique de forma rígida el tamaño de la máquina virtual. Proporcione un parámetro para que los usuarios de la plantilla puedan modificar el tamaño de la máquina virtual que implementan.

En el ejemplo siguiente **no se supera** esta prueba.

```json
"hardwareProfile": {
    "vmSize": "Standard_D2_v3"
}
```

En cambio, proporcione un parámetro.

```json
"vmSize": {
    "type": "string",
    "defaultValue": "Standard_A2_v2",
    "metadata": {
        "description": "Size for the Virtual Machine."
    }
},
```

Luego, establezca el tamaño de la máquina virtual en ese parámetro.

```json
"hardwareProfile": {
    "vmSize": "[parameters('vmSize')]"
},
```

## <a name="min-and-max-values-are-numbers"></a>Los valores mínimo y máximo son números

Nombre de la prueba: **Los valores mínimo y máximo son números**

Si define los valores mínimo y máximo de un parámetro, especifíquelos como números.

En el ejemplo siguiente **no se supera** esta prueba:

```json
"exampleParameter": {
    "type": "int",
    "minValue": "0",
    "maxValue": "10"
},
```

En su lugar, proporcione los valores como números. En el ejemplo siguiente **se supera** esta prueba:

```json
"exampleParameter": {
    "type": "int",
    "minValue": 0,
    "maxValue": 10
},
```

También recibirá esta advertencia si proporciona un valor mínimo o máximo, pero no el otro.

## <a name="artifacts-parameter-defined-correctly"></a>Parámetro de artefactos definido correctamente

Nombre de la prueba : **parámetro de artefactos**

Cuando incluya parámetros para `_artifactsLocation` y `_artifactsLocationSasToken`, utilice los valores predeterminados y los tipos correctos. Deben cumplirse las condiciones siguientes para superar la prueba:

* Si proporciona uno de los parámetros, debe proporcionar el otro
* `_artifactsLocation` debe ser de tipo **string**
* `_artifactsLocation` debe tener un valor predeterminado en la plantilla principal
* `_artifactsLocation` no puede tener un valor predeterminado en una plantilla anidada 
* `_artifactsLocation` debe tener `"[deployment().properties.templateLink.uri]"` o la dirección URL del repositorio sin formato como valor predeterminado
* `_artifactsLocationSasToken` debe ser de tipo **secureString**
* `_artifactsLocationSasToken` solo puede tener una cadena vacía como valor predeterminado
* `_artifactsLocationSasToken` no puede tener un valor predeterminado en una plantilla anidada 

## <a name="declared-variables-must-be-used"></a>Se deben usar variables declaradas

Nombre de la prueba: **Debe hacerse referencia a las variables**

Para reducir la confusión en la plantilla, elimine las variables definidas, pero que no se utilicen. Esta prueba busca las variables que no se usan en ninguna parte de la plantilla.

## <a name="dynamic-variable-should-not-use-concat"></a>La variable dinámica no debe utilizar concat

Nombre de la prueba: **Las referencias de variables dinámicas no deben usar concat**

A veces es necesario crear dinámicamente una variable que se base en el valor de otra variable u otro parámetro. No utilice la función [concat](template-functions-string.md#concat) al establecer el valor. En su lugar, use un objeto que incluya las opciones disponibles y obtenga dinámicamente una de las propiedades del objeto durante la implementación.

En el ejemplo siguiente **se supera** esta prueba. La variable **currentImage** se establece dinámicamente durante la implementación.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "osType": {
          "type": "string",
          "allowedValues": [
              "Windows",
              "Linux"
          ]
      }
  },
  "variables": {
    "imageOS": {
        "Windows": {
            "image": "Windows Image"
        },
        "Linux": {
            "image": "Linux Image"
        }
    },
    "currentImage": "[variables('imageOS')[parameters('osType')].image]"
  },
  "resources": [],
  "outputs": {
      "result": {
          "type": "string",
          "value": "[variables('currentImage')]"
      }
  }
}
```

## <a name="use-recent-api-version"></a>Uso de la versión de API reciente

Nombre de la prueba: **apiVersions debe ser reciente**

La versión de API de cada recurso debe usar una versión reciente. La prueba evalúa la versión que se usa con respecto a las versiones disponibles para ese tipo de recurso.

## <a name="use-hardcoded-api-version"></a>Uso de la versión de API codificada de forma rígida

Nombre de la prueba: **No está permitido el uso de apiVersions de proveedores**

La versión de API para un tipo de recurso determina las propiedades que están disponibles. Proporcione una versión de API codificada de forma rígida en la plantilla. No recupere ninguna versión de API que se determine durante la implementación. No sabrá qué propiedades están disponibles.

En el ejemplo siguiente **no se supera** esta prueba.

```json
"resources": [
    {
        "type": "Microsoft.Compute/virtualMachines",
        "apiVersion": "[providers('Microsoft.Compute', 'virtualMachines').apiVersions[0]]",
        ...
    }
]
```

En el ejemplo siguiente **se supera** esta prueba.

```json
"resources": [
    {
       "type": "Microsoft.Compute/virtualMachines",
       "apiVersion": "2019-12-01",
       ...
    }
]
```

## <a name="properties-cant-be-empty"></a>Las propiedades no pueden estar vacías

Nombre de la prueba: **La plantilla no debe contener espacios en blanco**

No codifique propiedades de forma rígida en un valor vacío. Los valores vacíos incluyen cadenas, objetos o matrices NULL y vacíos. Si ha establecido una propiedad en un valor vacío, quite esa propiedad de la plantilla. Sin embargo, es correcto establecer una propiedad en un valor vacío durante la implementación, por ejemplo, a través de un parámetro.

## <a name="use-resource-id-functions"></a>Uso de funciones de identificador de recurso

Nombre de la prueba: **Los identificadores se deben derivar de ResourceId**

Al especificar un identificador de recurso, utilice una de las funciones de identificador de recurso. Las funciones permitidas son las siguientes:

* [resourceId](template-functions-resource.md#resourceid)
* [subscriptionResourceId](template-functions-resource.md#subscriptionresourceid)
* [tenantResourceId](template-functions-resource.md#tenantresourceid)
* [extensionResourceId](template-functions-resource.md#extensionresourceid)

No utilice la función concat para crear un identificador de recurso. En el ejemplo siguiente **no se supera** esta prueba.

```json
"networkSecurityGroup": {
    "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', variables('networkSecurityGroupName'))]"
}
```

En el ejemplo siguiente **se supera** esta prueba.

```json
"networkSecurityGroup": {
    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
}
```

## <a name="resourceid-function-has-correct-parameters"></a>La función ResourceId tiene parámetros correctos

Nombre de la prueba: **ResourceId no debe contener**

Al generar identificadores de recursos, no use funciones innecesarias para parámetros opcionales. De forma predeterminada, la función [ResourceId](template-functions-resource.md#resourceid) usa la suscripción y el grupo de recursos actuales. No es necesario proporcionar esos valores.  

En el siguiente ejemplo **no se supera** esta prueba, porque no es necesario proporcionar el identificador de suscripción y el nombre del grupo de recursos actuales.

```json
"networkSecurityGroup": {
    "id": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
}
```

En el ejemplo siguiente **se supera** esta prueba.

```json
"networkSecurityGroup": {
    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
}
```

Esta prueba se aplica a:

* [resourceId](template-functions-resource.md#resourceid)
* [subscriptionResourceId](template-functions-resource.md#subscriptionresourceid)
* [tenantResourceId](template-functions-resource.md#tenantresourceid)
* [extensionResourceId](template-functions-resource.md#extensionresourceid)
* [reference](template-functions-resource.md#reference)
* [list*](template-functions-resource.md#list)

En `reference` y `list*`, la prueba **no se supera** cuando se usa `concat` para crear el identificador de recurso.

## <a name="dependson-best-practices"></a>Procedimientos recomendados de dependsOn

Nombre de la prueba: **Procedimientos recomendados de dependsOn**

Al establecer las dependencias de implementación, no use la función [si](template-functions-logical.md#if) para probar una condición. Si un recurso depende de un recurso que se [implementa condicionalmente](conditional-resource-deployment.md), establezca la dependencia como lo haría con cualquier recurso. Cuando un recurso condicional no está implementado, Azure Resource Manager lo quita automáticamente de las dependencias necesarias.

En el ejemplo siguiente **no se supera** esta prueba.

```json
"dependsOn": [
    "[if(equals(parameters('newOrExisting'),'new'), variables('storageAccountName'), '')]"
]
```

En el ejemplo siguiente **se supera** esta prueba.

```json
"dependsOn": [
    "[variables('storageAccountName')]"
]
```

## <a name="nested-or-linked-deployments-cant-use-debug"></a>Las implementaciones anidadas o vinculadas no pueden utilizar depuración

Nombre de la prueba: **Los recursos de implementación no deben depurarse**

Al definir una [plantilla anidada o vinculada](linked-templates.md) con el tipo de recurso **Microsoft.Resources/deployments**, se puede habilitar la depuración para esa plantilla. La depuración es correcta cuando hay que probar la plantilla, pero se debe desactivar cuando esté a punto para usar la plantilla en producción.

## <a name="admin-user-names-cant-be-literal-value"></a>Los nombres de usuario administrador no pueden ser valores literales

Nombre de la prueba: **adminUsername no debe ser un literal**

Al establecer un nombre de usuario administrador, no utilice un valor literal.

En el ejemplo siguiente **no se supera** esta prueba:

```json
"osProfile":  {
    "adminUserName":  "myAdmin"
},
```

En su lugar, use un parámetro. En el ejemplo siguiente **se supera** esta prueba:

```json
"osProfile": {
    "adminUsername": "[parameters('adminUsername')]"
}
```

## <a name="use-latest-vm-image"></a>Uso de la imagen de máquina virtual más reciente

Nombre de la prueba: **Las imágenes de máquina virtual deben usar la versión más reciente**

Si la plantilla incluye una máquina virtual con una imagen, asegúrese de que está usando la versión más reciente de dicha imagen.

## <a name="use-stable-vm-images"></a>Uso de imágenes de máquina virtual estables

Nombre de la prueba: **Virtual Machines no debe ser una versión preliminar**

Las máquinas virtuales no deben usar imágenes de vista previa.

En el ejemplo siguiente **no se supera** esta prueba.

```json
"imageReference": {
    "publisher": "Canonical",
    "offer": "UbuntuServer",
    "sku": "16.04-LTS",
    "version": "latest-preview"
}
```

En el ejemplo siguiente **se supera** esta prueba.

```json
"imageReference": {
    "publisher": "Canonical",
    "offer": "UbuntuServer",
    "sku": "16.04-LTS",
    "version": "latest"
},
```

## <a name="dont-use-managedidentity-extension"></a>No use la extensión ManagedIdentity

Nombre de la prueba: **No se debe usar ManagedIdentityExtension**

No aplique la extensión ManagedIdentity a una máquina virtual. Para más información, consulte [Dejar de usar la extensión de identidades administradas de máquina virtual y empezar a usar Azure Instance Metadata Service](../../active-directory/managed-identities-azure-resources/howto-migrate-vm-extension.md).

## <a name="outputs-cant-include-secrets"></a>Las salidas no pueden incluir secretos

Nombre de la prueba: **Las salidas no deben contener secretos**

No incluya ningún valor en la sección de salidas que pueda exponer secretos. La salida de una plantilla se almacena en el historial de implementación, por lo que un usuario malintencionado podría encontrar esa información.

En el siguiente ejemplo **no se supera** la prueba porque incluye un parámetro seguro en un valor de salida.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "secureParam": {
            "type": "securestring"
        }
    },
    "functions": [],
    "variables": {},
    "resources": [],
    "outputs": {
        "badResult": {
            "type": "string",
            "value": "[concat('this is the value ', parameters('secureParam'))]"
        }
    }
}
```

En el siguiente ejemplo **no se supera** porque utiliza una función [list*](template-functions-resource.md#list) en las salidas.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageName": {
            "type": "string"
        }
    },
    "functions": [],
    "variables": {},
    "resources": [],
    "outputs": {
        "badResult": {
            "type": "object",
            "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('storageName')), '2019-06-01')]"
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo ejecutar el kit de herramientas para pruebas, consulte [Uso del kit de herramientas para pruebas de plantillas de Resource Manager](test-toolkit.md).