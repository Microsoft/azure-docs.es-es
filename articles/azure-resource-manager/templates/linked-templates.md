---
title: Plantillas de vínculo para la implementación
description: Describe cómo usar plantillas vinculadas en una plantilla del Administrador de recursos de Azure para crear una solución de plantilla modular. Muestra cómo pasar valores de parámetros y especificar un archivo de parámetros y las direcciones URL creadas dinámicamente.
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: 40da2443828a07f2171922fcc6d8976d464d0ad4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086819"
---
# <a name="using-linked-and-nested-templates-when-deploying-azure-resources"></a>Uso de plantillas vinculadas y anidadas al implementar recursos de Azure

Para implementar soluciones complejas, puede dividir la plantilla en numerosas plantillas relacionadas y, a continuación, implementarlas juntas mediante una plantilla principal. Las plantillas relacionadas pueden ser archivos independientes o una sintaxis de la plantilla que esté insertada en la plantilla principal. En este artículo se usa el término **plantilla vinculada** para hacer referencia a un archivo de plantilla independiente al que se hace referencia a través de un vínculo de la plantilla principal. Usa el término **plantilla anidada** para hacer referencia a la sintaxis de la plantilla insertada dentro de la plantilla principal.

En el caso de soluciones pequeñas o medianas, es más fácil entender y mantener una única plantilla. Puede ver todos los recursos y valores en un único archivo. Para los escenarios avanzados, las plantillas vinculadas le permiten desglosar la solución en componentes dirigidos. Estas plantillas se pueden reutilizar fácilmente para otros escenarios.

Para obtener un tutorial, consulte [Tutorial: creación de plantillas vinculadas de Azure Resource Manager](./deployment-tutorial-linked-template.md).

> [!NOTE]
> En cuanto a las plantillas vinculadas o anidadas, solo se puede usar el modo de implementación [incremental](deployment-modes.md).
>

## <a name="nested-template"></a>Plantilla anidada

Para anidar una plantilla, agregue un [recurso de implementaciones](/azure/templates/microsoft.resources/deployments) a la plantilla principal. En la propiedad **template**, especifique la sintaxis de la plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "nestedTemplate1",
      "properties": {
        "mode": "Incremental",
        "template": {
          <nested-template-syntax>
        }
      }
    }
  ],
  "outputs": {
  }
}
```

En el ejemplo siguiente se implementa una cuenta de almacenamiento mediante una plantilla anidada.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "nestedTemplate1",
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2019-04-01",
              "name": "[parameters('storageAccountName')]",
              "location": "West US",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "StorageV2"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
  }
}
```

### <a name="expression-evaluation-scope-in-nested-templates"></a>Ámbito de evaluación de expresiones en plantillas anidadas

Al utilizar una plantilla anidada, puede especificar si las expresiones de plantilla se evalúan dentro del ámbito de la plantilla principal o de la plantilla anidada. El ámbito determina cómo se resuelven los parámetros, las variables y las funciones como [resourceGroup](template-functions-resource.md#resourcegroup) y [subscription](template-functions-resource.md#subscription).

El ámbito se establece mediante la propiedad `expressionEvaluationOptions`. De forma predeterminada, la propiedad `expressionEvaluationOptions` está establecida en `outer`, lo que significa que usa el ámbito de plantilla principal. Establezca el valor en `inner` para que las expresiones se evalúen dentro del ámbito de la plantilla anidada.

```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2019-10-01",
  "name": "nestedTemplate1",
  "properties": {
  "expressionEvaluationOptions": {
    "scope": "inner"
  },
  ...
```

En la plantilla siguiente se muestra cómo se resuelven las expresiones de plantilla según el ámbito. Contiene una variable denominada `exampleVar` que se define en la plantilla principal y en la plantilla anidada. Devuelve el valor de la variable.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  },
  "variables": {
    "exampleVar": "from parent template"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "nestedTemplate1",
      "properties": {
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "variables": {
            "exampleVar": "from nested template"
          },
          "resources": [
          ],
          "outputs": {
            "testVar": {
              "type": "string",
              "value": "[variables('exampleVar')]"
            }
          }
        }
      }
    }
  ],
  "outputs": {
    "messageFromLinkedTemplate": {
      "type": "string",
      "value": "[reference('nestedTemplate1').outputs.testVar.value]"
    }
  }
}
```

El valor de `exampleVar` cambia según el valor de la `scope` propiedad en `expressionEvaluationOptions`. La siguiente tabla muestra los resultados para cada ámbito.

| Ámbito `expressionEvaluationOptions` | Output |
| ----- | ------ |
| interna | desde la plantilla anidada |
| outer (o predeterminado) | desde la plantilla principal |

En el ejemplo siguiente se implementa un servidor SQL Server y se recupera el secreto del almacén de claves que se va a usar para la contraseña. El ámbito se establece en `inner` porque crea dinámicamente el identificador de almacén de claves (consulte `adminPassword.reference.keyVault` en las plantillas externas `parameters`) y lo pasa como un parámetro a la plantilla anidada.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "The location where the resources will be deployed."
      }
    },
    "vaultName": {
      "type": "string",
      "metadata": {
        "description": "The name of the keyvault that contains the secret."
      }
    },
    "secretName": {
      "type": "string",
      "metadata": {
        "description": "The name of the secret."
      }
    },
    "vaultResourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "The name of the resource group that contains the keyvault."
      }
    },
    "vaultSubscription": {
      "type": "string",
      "defaultValue": "[subscription().subscriptionId]",
      "metadata": {
        "description": "The name of the subscription that contains the keyvault."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "dynamicSecret",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "parameters": {
          "location": {
            "value": "[parameters('location')]"
          },
          "adminLogin": {
            "value": "ghuser"
          },
          "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "[resourceId(parameters('vaultSubscription'), parameters('vaultResourceGroupName'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
              },
              "secretName": "[parameters('secretName')]"
            }
          }
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "adminLogin": {
              "type": "string"
            },
            "adminPassword": {
              "type": "securestring"
            },
            "location": {
              "type": "string"
            }
          },
          "variables": {
            "sqlServerName": "[concat('sql-', uniqueString(resourceGroup().id, 'sql'))]"
          },
          "resources": [
            {
              "type": "Microsoft.Sql/servers",
              "apiVersion": "2018-06-01-preview",
              "name": "[variables('sqlServerName')]",
              "location": "[parameters('location')]",
              "properties": {
                "administratorLogin": "[parameters('adminLogin')]",
                "administratorLoginPassword": "[parameters('adminPassword')]"
              }
            }
          ],
          "outputs": {
            "sqlFQDN": {
              "type": "string",
              "value": "[reference(variables('sqlServerName')).fullyQualifiedDomainName]"
            }
          }
        }
      }
    }
  ],
  "outputs": {
  }
}
```

> [!NOTE]
>
> Cuando el ámbito está establecido en `outer`, no puede usar la función `reference` en la sección de salidas de una plantilla anidada para un recurso que haya implementado en la plantilla anidada. Para devolver los valores de un recurso implementado en una plantilla anidada, use el ámbito `inner` o convierta la plantilla anidada en una plantilla vinculada.

## <a name="linked-template"></a>Plantilla vinculada

Para vincular una plantilla, agregue un [recurso de implementaciones](/azure/templates/microsoft.resources/deployments) a la plantilla principal. En la propiedad **templateLink**, especifique el URI de la plantilla que se va a incluir. El ejemplo siguiente está vinculado a una plantilla que implementa una nueva cuenta de almacenamiento.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "linkedTemplate",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

Al hacer referencia a una plantilla vinculada, el valor de `uri` no debe ser un archivo local o un archivo que solo esté disponible en la red local. Debe proporcionar un valor de URI que se puede descargar como **http** o **https**.

> [!NOTE]
>
> Puede hacer referencia a plantillas mediante parámetros que, en última instancia, se resuelven en algo que usa **http** o **https**, por ejemplo, usando el parámetro `_artifactsLocation` de esta manera: `"uri": "[concat(parameters('_artifactsLocation'), '/shared/os-disk-parts-md.json', parameters('_artifactsLocationSasToken'))]",`

Resource Manager debe tener acceso a la plantilla. Una opción es colocar la plantilla vinculada en una cuenta de almacenamiento y usar el URI para dicho elemento.

Las [especificaciones de plantilla](./template-specs.md) (actualmente en versión preliminar privada) le permiten compartir plantillas de Resource Manager con otros usuarios de la organización. Las especificaciones de plantillas también se pueden usar para empaquetar una plantilla principal y sus plantillas vinculadas. Para más información, consulte:

- [Tutorial: Creación de una especificación de plantilla con plantillas vinculadas](./template-specs-create-linked.md).
- [Tutorial: Implementación de una especificación de plantilla como una plantilla vinculada](./template-specs-deploy-linked-template.md).

### <a name="parameters-for-linked-template"></a>Parámetros de la plantilla vinculada

Puede proporcionar los parámetros de la plantilla vinculada en un archivo externo o alineados. Al proporcionar un archivo de parámetros externo, use la propiedad **parametersLink**:

```json
"resources": [
  {
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2019-10-01",
  "name": "linkedTemplate",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
      "contentVersion":"1.0.0.0"
    },
    "parametersLink": {
      "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.parameters.json",
      "contentVersion":"1.0.0.0"
    }
  }
  }
]
```

Para pasar los valores de parámetro alineados, use la propiedad **parameters**.

```json
"resources": [
  {
   "type": "Microsoft.Resources/deployments",
   "apiVersion": "2019-10-01",
   "name": "linkedTemplate",
   "properties": {
     "mode": "Incremental",
     "templateLink": {
      "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
      "contentVersion":"1.0.0.0"
     },
     "parameters": {
      "storageAccountName":{"value": "[parameters('storageAccountName')]"}
    }
   }
  }
]
```

No se pueden usar los parámetros alineados ni un vínculo a un archivo de parámetros. La implementación produce un error cuando ambos (`parametersLink` y `parameters`) se especifican.

## <a name="contentversion"></a>contentVersion

No tiene que proporcionar la propiedad `contentVersion` para la propiedad `templateLink` o `parametersLink`. Si no proporciona un `contentVersion`, se implementará la versión actual de la plantilla. Si proporciona un valor, este debe coincidir con la versión de la plantilla vinculada o, de lo contrario, se producirá un error durante la implementación.

## <a name="using-variables-to-link-templates"></a>Uso de variables para vincular plantillas

Los ejemplos anteriores mostraron valores de dirección URL codificadas de forma rígida para los vínculos de la plantilla. Este enfoque puede funcionar en una plantilla sencilla, pero no funciona bien para un gran conjunto de plantillas modulares. En su lugar, puede crear una variable estática que almacene una dirección URL base para la plantilla principal y, luego, crear dinámicamente direcciones URL para las plantillas vinculadas desde esa dirección URL base. La ventaja de este enfoque es que puede mover o bifurcar fácilmente la plantilla, ya que solo tiene que cambiar la variable estática en la plantilla principal. La plantilla principal pasa los URI correctos por toda la plantilla descompuesta.

En el ejemplo siguiente se muestra cómo usar una dirección URL base para crear dos direcciones URL para las plantillas vinculadas (**sharedTemplateUrl** y **vmTemplate**).

```json
"variables": {
  "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
  "sharedTemplateUrl": "[uri(variables('templateBaseUrl'), 'shared-resources.json')]",
  "vmTemplateUrl": "[uri(variables('templateBaseUrl'), 'database-2disk-resources.json')]"
}
```

También puede usar la función [deployment()](template-functions-deployment.md#deployment) para obtener la dirección URL base de la plantilla actual y usar esta información para obtener la dirección URL de otras plantillas en la misma ubicación. Este enfoque resulta útil si cambia la ubicación de la plantilla o desea evitar la codificación de forma rígida de las direcciones URL en el archivo de plantilla. La propiedad templateLink solo se devuelve al vincular una plantilla remota a una URL. Si utiliza una plantilla local, esa propiedad no está disponible.

```json
"variables": {
  "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
}
```

En última instancia, usaría la variable en la propiedad `uri` de una propiedad `templateLink`.

```json
"templateLink": {
 "uri": "[variables('sharedTemplateUrl')]",
 "contentVersion":"1.0.0.0"
}
```

## <a name="using-copy"></a>Uso de la copia

Para crear varias instancias de un recurso con una plantilla anidada, agregue el elemento de copia en el nivel del recurso **Microsoft.resources/Deployments**. O bien, si el ámbito es interno, puede agregar la copia dentro de la plantilla anidada.

En la plantilla de ejemplo siguiente se muestra cómo usar la función de copiar con una plantilla anidada.

```json
"resources": [
  {
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2019-10-01",
  "name": "[concat('nestedTemplate', copyIndex())]",
  // yes, copy works here
  "copy":{
    "name": "storagecopy",
    "count": 2
  },
  "properties": {
    "mode": "Incremental",
    "expressionEvaluationOptions": {
    "scope": "inner"
    },
    "template": {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
      {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(variables('storageName'), copyIndex())]",
      "location": "West US",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
      // Copy works here when scope is inner
      // But, when scope is default or outer, you get an error
      //"copy":{
      //  "name": "storagecopy",
      //  "count": 2
      //}
      }
    ]
    }
  }
  }
]
```

## <a name="get-values-from-linked-template"></a>Obtención de valores a partir de la plantilla vinculada

Para obtener un valor de salida de una plantilla vinculada, recupere el valor de propiedad con sintaxis de esta manera: `"[reference('deploymentName').outputs.propertyName.value]"`.

Al obtener una propiedad de salida de una plantilla vinculada, el nombre de la propiedad no debe incluir un guión.

Los ejemplos siguientes muestran cómo hacer referencia a una plantilla vinculada y recuperar un valor de salida. La plantilla vinculada devuelve un mensaje simple.  En primer lugar, la plantilla vinculada:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [],
  "outputs": {
    "greetingMessage": {
      "value": "Hello World",
      "type" : "string"
    }
  }
}
```

La plantilla principal implementa la plantilla vinculada y obtiene el valor devuelto. Observe que hace referencia al recurso de implementación por su nombre, y utiliza el nombre de la propiedad devuelta por la plantilla vinculada.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "linkedTemplate",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[uri(deployment().properties.templateLink.uri, 'helloworld.json')]",
          "contentVersion": "1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
    "messageFromLinkedTemplate": {
      "type": "string",
      "value": "[reference('linkedTemplate').outputs.greetingMessage.value]"
    }
  }
}
```

Como con otros tipos de recursos, puede establecer dependencias entre la plantilla vinculada y otros recursos. Cuando otros recursos requieren un valor de salida de la plantilla vinculada, asegúrese de que la plantilla vinculada se implemente antes que ellos. O bien, si la plantilla vinculada se basa en otros recursos, asegúrese de que los otros recursos se implementen antes que la plantilla vinculada.

En el ejemplo siguiente se muestra una plantilla que implementa una dirección IP pública y devuelve el identificador de recurso del recurso de Azure para esa IP pública:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "publicIPAddresses_name": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2018-11-01",
      "name": "[parameters('publicIPAddresses_name')]",
      "location": "eastus",
      "properties": {
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Dynamic",
        "idleTimeoutInMinutes": 4
      },
      "dependsOn": []
    }
  ],
  "outputs": {
    "resourceID": {
      "type": "string",
      "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
    }
  }
}
```

Para usar la dirección IP pública de la plantilla anterior al implementar un equilibrador de carga, vincule a la plantilla y declare una dependencia en el recurso `Microsoft.Resources/deployments`. La dirección IP pública del equilibrador de carga se establece en el valor de salida de la plantilla vinculada.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "loadBalancers_name": {
      "defaultValue": "mylb",
      "type": "string"
    },
    "publicIPAddresses_name": {
      "defaultValue": "myip",
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Network/loadBalancers",
      "apiVersion": "2018-11-01",
      "name": "[parameters('loadBalancers_name')]",
      "location": "eastus",
      "properties": {
        "frontendIPConfigurations": [
          {
            "name": "LoadBalancerFrontEnd",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                // this is where the output value from linkedTemplate is used
                "id": "[reference('linkedTemplate').outputs.resourceID.value]"
              }
            }
          }
        ],
        "backendAddressPools": [],
        "loadBalancingRules": [],
        "probes": [],
        "inboundNatRules": [],
        "outboundNatRules": [],
        "inboundNatPools": []
      },
      // This is where the dependency is declared
      "dependsOn": [
        "linkedTemplate"
      ]
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "linkedTemplate",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[uri(deployment().properties.templateLink.uri, 'publicip.json')]",
          "contentVersion": "1.0.0.0"
        },
        "parameters":{
          "publicIPAddresses_name":{"value": "[parameters('publicIPAddresses_name')]"}
        }
      }
    }
  ]
}
```

## <a name="deployment-history"></a>Historial de implementación

Resource Manager procesa cada plantilla como una implementación independiente en el historial de implementación. Una plantilla principal con tres plantillas vinculadas o anidadas aparece en el historial de implementación del modo siguiente:

![Historial de implementación](./media/linked-templates/deployment-history.png)

Puede utilizar estas entradas independientes en el historial para recuperar valores de salida después de la implementación. La siguiente plantilla crea una dirección IP pública y genera la dirección IP como salida:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "publicIPAddresses_name": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2018-11-01",
      "name": "[parameters('publicIPAddresses_name')]",
      "location": "southcentralus",
      "properties": {
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Static",
        "idleTimeoutInMinutes": 4,
        "dnsSettings": {
          "domainNameLabel": "[concat(parameters('publicIPAddresses_name'), uniqueString(resourceGroup().id))]"
        }
      },
      "dependsOn": []
    }
  ],
  "outputs": {
    "returnedIPAddress": {
      "type": "string",
      "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]"
    }
  }
}
```

La siguiente plantilla se vincula a la plantilla anterior. Crea tres direcciones IP públicas.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "[concat('linkedTemplate', copyIndex())]",
      "copy": {
        "count": 3,
        "name": "ip-loop"
      },
      "properties": {
        "mode": "Incremental",
        "templateLink": {
        "uri": "[uri(deployment().properties.templateLink.uri, 'static-public-ip.json')]",
        "contentVersion": "1.0.0.0"
        },
        "parameters":{
          "publicIPAddresses_name":{"value": "[concat('myip-', copyIndex())]"}
        }
      }
    }
  ]
}
```

Después de la implementación, puede recuperar los valores de salida con el siguiente script de PowerShell:

```azurepowershell-interactive
$loopCount = 3
for ($i = 0; $i -lt $loopCount; $i++)
{
  $name = 'linkedTemplate' + $i;
  $deployment = Get-AzResourceGroupDeployment -ResourceGroupName examplegroup -Name $name
  Write-Output "deployment $($deployment.DeploymentName) returned $($deployment.Outputs.returnedIPAddress.value)"
}
```

O bien, el script de la CLI de Azure en un shell de Bash:

```azurecli-interactive
#!/bin/bash

for i in 0 1 2;
do
  name="linkedTemplate$i";
  deployment=$(az deployment group show -g examplegroup -n $name);
  ip=$(echo $deployment | jq .properties.outputs.returnedIPAddress.value);
  echo "deployment $name returned $ip";
done
```

## <a name="securing-an-external-template"></a>Protección de una plantilla externa

Aunque la plantilla vinculada debe estar disponible externamente, no es necesario que esté generalmente disponible para el público. Puede agregar la plantilla a una cuenta de almacenamiento privada que sea accesible solo al propietario de la cuenta de almacenamiento. Ahora cree un token de Firma de acceso compartido (SAS) para permitir el acceso durante la implementación. Ese token SAS se agrega al identificador URI para la plantilla vinculada. Aunque el token se pasa como una cadena segura, el identificador URI de la plantilla vinculada, incluido el token de SAS, se registra en las operaciones de implementación. Para limitar la exposición, establezca una caducidad para el token.

También se puede limitar el acceso al archivo de parámetros a través de un token de SAS.

Actualmente, no se puede agregar un vínculo a una plantilla que se encuentre en una cuenta de almacenamiento detrás de un [firewall de Azure Storage](../../storage/common/storage-network-security.md).

En el ejemplo siguiente se muestra cómo pasar un token de SAS al vincular a una plantilla:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  "containerSasToken": { "type": "securestring" }
  },
  "resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2019-10-01",
    "name": "linkedTemplate",
    "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
      "contentVersion": "1.0.0.0"
    }
    }
  }
  ],
  "outputs": {
  }
}
```

En PowerShell, se obtiene un token para el contenedor y se implementan las plantillas con los comandos siguientes. Tenga en cuenta que el parámetro **containerSasToken** se define en la plantilla. No es un parámetro en el comando **New-AzResourceGroupDeployment**.

```azurepowershell-interactive
Set-AzCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
$token = New-AzStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
$url = (Get-AzStorageBlob -Container templates -Blob parent.json).ICloudBlob.uri.AbsoluteUri
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ($url + $token) -containerSasToken $token
```

En cuanto a la CLI de Azure en un shell de Bash, se obtiene un token para el contenedor y se implementan las plantillas con el código siguiente:

```azurecli-interactive
#!/bin/bash

expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
  --resource-group ManageGroup \
  --name storagecontosotemplates \
  --query connectionString)
token=$(az storage container generate-sas \
  --name templates \
  --expiry $expiretime \
  --permissions r \
  --output tsv \
  --connection-string $connection)
url=$(az storage blob url \
  --container-name templates \
  --name parent.json \
  --output tsv \
  --connection-string $connection)
parameter='{"containerSasToken":{"value":"?'$token'"}}'
az deployment group create --resource-group ExampleGroup --template-uri $url?$token --parameters $parameter
```

## <a name="example-templates"></a>Plantillas de ejemplo

En los ejemplos siguientes se muestran los usos más comunes de las plantillas vinculadas.

|Plantilla principal  |Plantilla vinculada |Descripción  |
|---------|---------| ---------|
|[Hello World](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworldparent.json) |[plantilla vinculada](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworld.json) | Devuelve una cadena de plantilla vinculada. |
|[Load Balancer con dirección IP pública](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) |[plantilla vinculada](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) |Devuelve una dirección IP pública de la plantilla vinculada y establece ese valor en el equilibrador de carga. |
|[Varias direcciones IP](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json) | [plantilla vinculada](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip.json) |Crea varias direcciones IP públicas en la plantilla vinculada.  |

## <a name="next-steps"></a>Pasos siguientes

* Para seguir los pasos de un tutorial, consulte [Tutorial: creación de plantillas vinculadas de Azure Resource Manager](./deployment-tutorial-linked-template.md).
* Para obtener información sobre cómo definir el orden de implementación de los recursos, consulte [Definición de dependencias en plantillas de Azure Resource Manager](define-resource-dependency.md).
* Para obtener información sobre cómo definir un recurso y crear numerosas instancias de este, consulte [Creación de varias instancias de recursos en Azure Resource Manager](copy-resources.md).
* Para obtener información sobre cómo configurar una plantilla en una cuenta de almacenamiento y generar un token de SAS, consulte [Deploy resources with Resource Manager templates and Azure PowerShell](deploy-powershell.md) (Implementación de recursos con plantillas de Resource Manager y Azure PowerShell) o [Deploy resources with Resource Manager templates and Azure CLI](deploy-cli.md) (Implementación de recursos con plantillas de Azure Manager y la CLI de Azure).
