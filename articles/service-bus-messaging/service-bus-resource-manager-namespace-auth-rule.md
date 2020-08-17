---
title: Creación de una regla de autorización de Service Bus con una plantilla de Azure
description: Creación de una regla de autorización de Service Bus para un espacio de nombres y una cola mediante una plantilla de Azure Resource Manager
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurecli
ms.openlocfilehash: b0ffe022f6c61cf7d06c510c4f8fa5c764d72b42
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067195"
---
# <a name="create-a-service-bus-authorization-rule-for-namespace-and-queue-using-an-azure-resource-manager-template"></a>Creación de una regla de autorización de Service Bus para un espacio de nombres y una cola mediante una plantilla de Azure Resource Manager

En este artículo se muestra cómo utilizar una plantilla de Azure Resource Manager que crea una [regla de autorización](service-bus-authentication-and-authorization.md#shared-access-signature) para una cola y un espacio de nombres de Service Bus. En el artículo se explica cómo especificar los recursos que se implementan y cómo definir los parámetros que se especifican cuando se ejecuta la implementación. Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades.

Para más información sobre la creación de plantillas, consulte [Creación de plantillas de Azure Resource Manager][Authoring Azure Resource Manager templates].

Para ver la plantilla completa, consulte la [plantilla de regla de autorización de Service Bus][Service Bus auth rule template] en GitHub.

> [!NOTE]
> Las siguientes plantillas de Azure Resource Manager están disponibles para su descarga e implementación.
> 
> * [Creación de un espacio de nombres de Service Bus](service-bus-resource-manager-namespace.md)
> * [Creación de un espacio de nombres de Service Bus con cola](service-bus-resource-manager-namespace-queue.md)
> * [Creación de un espacio de nombres de Service Bus con un tema y una suscripción](service-bus-resource-manager-namespace-topic.md)
> * [Create a Service Bus namespace with topic, subscription, and rule](service-bus-resource-manager-namespace-topic-with-rule.md) (Creación de un espacio de nombres de Service Bus con tema, suscripción y regla)
> 
> Para buscar las plantillas más recientes, visite la galería de [Plantillas de inicio rápido de Azure][Azure Quickstart Templates] y busque **Service Bus**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="what-will-you-deploy"></a>¿Qué va a implementar?

Con esta plantilla, implementa una regla de autorización de Service Bus para una entidad de mensajería y un espacio de nombres (una cola, en este caso).

Esta plantilla usa la [firma de acceso compartido (SAS)](service-bus-sas.md) para la autenticación. SAS permite a las aplicaciones autenticarse en Service Bus mediante una clave de acceso configurada en el espacio de nombres o en la entidad de mensajería (cola o tema) al que se asocian derechos específicos. A continuación, puede usar esta clave para generar un token SAS que a su vez, los clientes pueden usar para autenticarse en Service Bus.

Para ejecutar automáticamente la implementación, haga clic en el botón siguiente:

[![Implementación en Azure](./media/service-bus-resource-manager-namespace-auth-rule/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F301-servicebus-create-authrule-namespace-and-queue%2Fazuredeploy.json)

## <a name="parameters"></a>Parámetros

Con el Administrador de recursos de Azure, se definen los parámetros de los valores que desea especificar al implementar la plantilla. La plantilla incluye una sección denominada `Parameters` que contiene todos los valores de los parámetros. Debe definir un parámetro para los valores que variarán según el proyecto que vaya a implementar o según el entorno en el que vaya a realizar la implementación. No defina parámetros para valores que vayan a permanecer igual. Cada valor de parámetro se usa en la plantilla para definir los recursos que se implementan.

La plantilla define los parámetros siguientes.

### <a name="servicebusnamespacename"></a>serviceBusNamespaceName

El nombre del espacio de nombres de Service Bus que crear.

```json
"serviceBusNamespaceName": {
"type": "string"
}
```

### <a name="namespaceauthorizationrulename"></a>namespaceAuthorizationRuleName

El nombre de la regla de autorización para el espacio de nombres.

```json
"namespaceAuthorizationRuleName ": {
"type": "string"
}
```

### <a name="servicebusqueuename"></a>serviceBusQueueName

El nombre de la cola en el espacio de nombres de Service Bus.

```json
"serviceBusQueueName": {
"type": "string"
}
```

### <a name="servicebusapiversion"></a>serviceBusApiVersion

La versión de la API de Service Bus de la plantilla.

```json
"serviceBusApiVersion": { 
       "type": "string", 
       "defaultValue": "2017-04-01", 
       "metadata": { 
           "description": "Service Bus ApiVersion used by the template" 
       }
```

## <a name="resources-to-deploy"></a>Recursos para implementar

Crea un espacio de nombres de Service Bus estándar de tipo **Mensajería**y una regla de autorización de Service Bus para el espacio de nombres y la entidad.

```json
"resources": [
        {
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusNamespaceName')]",
            "type": "Microsoft.ServiceBus/namespaces",
            "location": "[variables('location')]",
            "kind": "Messaging",
            "sku": {
                "name": "Standard",
            },
            "resources": [
                {
                    "apiVersion": "[variables('sbVersion')]",
                    "name": "[parameters('serviceBusQueueName')]",
                    "type": "Queues",
                    "dependsOn": [
                        "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('serviceBusQueueName')]"
                    },
                    "resources": [
                        {
                            "apiVersion": "[variables('sbVersion')]",
                            "name": "[parameters('queueAuthorizationRuleName')]",
                            "type": "authorizationRules",
                            "dependsOn": [
                                "[parameters('serviceBusQueueName')]"
                            ],
                            "properties": {
                                "Rights": ["Listen"]
                            }
                        }
                    ]
                }
            ]
        }, {
            "apiVersion": "[variables('sbVersion')]",
            "name": "[variables('namespaceAuthRuleName')]",
            "type": "Microsoft.ServiceBus/namespaces/authorizationRules",
            "dependsOn": ["[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"],
            "location": "[resourceGroup().location]",
            "properties": {
                "Rights": ["Send"]
            }
        }
    ]
```

Para conocer la sintaxis y las propiedades JSON, consulte [espacios de nombres](/azure/templates/microsoft.servicebus/namespaces), [colas](/azure/templates/microsoft.servicebus/namespaces/queues) y [AuthorizationRules](/azure/templates/microsoft.servicebus/namespaces/authorizationrules).

## <a name="commands-to-run-deployment"></a>Comandos para ejecutar la implementación

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell

```powershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>
```

## <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado e implementado recursos con Azure Resource Manager, estos artículos le enseñarán como administrarlos:

* [Administración de Service Bus con PowerShell](./service-bus-manage-with-ps.md)
* [Administración de recursos de Service Bus con el Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/releases)
* [Autenticación y autorización de Service Bus](service-bus-authentication-and-authorization.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/?term=service+bus
[Using Azure PowerShell with Azure Resource Manager]: ../azure-resource-manager/management/manage-resources-powershell.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../azure-resource-manager/management/manage-resources-cli.md
[Service Bus auth rule template]: https://github.com/Azure/azure-quickstart-templates/blob/master/301-servicebus-create-authrule-namespace-and-queue/