---
title: Creación de suscripciones a temas y reglas de Service Bus mediante plantillas de Azure
description: Creación de un espacio de nombres de Service Bus con un tema, una suscripción y una regla mediante una plantilla de Azure Resource Manager
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3a53cadbdf7529a2690594617122e84f355e0e46
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88065665"
---
# <a name="create-a-service-bus-namespace-with-topic-subscription-and-rule-using-an-azure-resource-manager-template"></a>Creación de un espacio de nombres de Service Bus con un tema, una suscripción y una regla mediante una plantilla de Azure Resource Manager

En este artículo se muestra cómo utilizar una plantilla de Azure Resource Manager que crea un espacio de nombres de Service Bus con un tema, una suscripción y una regla (filtro). En el artículo se explica cómo especificar los recursos que se implementan y cómo definir los parámetros que se especifican cuando se ejecuta la implementación. Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades.

Para más información sobre la creación de plantillas, consulte [Creación de plantillas de Azure Resource Manager][Authoring Azure Resource Manager templates].

Para más información sobre procedimientos y patrones de convenciones de nomenclatura de recursos de Azure, vea [Convenciones de nomenclatura recomendadas para recursos de Azure][Recommended naming conventions for Azure resources].

Para ver la plantilla completa, consulte el artículo sobre el [espacio de nombres de Service Bus con un tema, suscripción y una regla][Service Bus namespace with topic, subscription, and rule].

> [!NOTE]
> Las siguientes plantillas de Azure Resource Manager están disponibles para su descarga e implementación.
> 
> * [Creación de un espacio de nombres de Service Bus con regla de autorización y cola](service-bus-resource-manager-namespace-auth-rule.md)
> * [Creación de un espacio de nombres de Service Bus con cola](service-bus-resource-manager-namespace-queue.md)
> * [Creación de un espacio de nombres de Service Bus](service-bus-resource-manager-namespace.md)
> * [Creación de un espacio de nombres de Service Bus con un tema y una suscripción](service-bus-resource-manager-namespace-topic.md)
> 
> Para buscar las últimas plantillas, visite la galería de [Plantillas de inicio rápido de Azure][Azure Quickstart Templates] y busque Service Bus.

## <a name="what-do-you-deploy"></a>¿Qué puede implementar?

Con esta plantilla, implementará un espacio de nombres de Service Bus con un tema, una suscripción y una regla (filtro).

Los [temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) proporcionan una o varias formas de comunicación en un patrón *publicación/suscripción*. Al usar temas y suscripciones, los componentes de una aplicación distribuida no se comunican directamente entre ellos, sino que intercambian mensajes a través de un tema que actúa como un intermediario. Una suscripción a un tema es similar a una cola virtual que recibe copias de mensajes que se enviaron al tema. Un filtro de una suscripción permite especificar qué mensajes enviados a un tema deben aparecer dentro de una suscripción a un tema determinado.

## <a name="what-are-rules-filters"></a>¿Cuáles son las reglas (filtros)?

En muchos escenarios, los mensajes que tienen características específicas deben procesarse de maneras diferentes. Para permitir este procesamiento personalizado, puede configurar suscripciones para buscar los mensajes que tengan propiedades específicas y, después, realizar modificaciones en dichas propiedades. Aunque las suscripciones de Service Bus ven todos los mensajes enviados al tema, solo se puede copiar un subconjunto de dichos mensajes en la cola de suscripción virtual. Esto se consigue mediante filtros de suscripción. Para más información sobre las reglas (filtros), vea [Reglas y acciones](service-bus-queues-topics-subscriptions.md#rules-and-actions).

Para ejecutar automáticamente la implementación, haga clic en el botón siguiente:

[![Implementación en Azure](./media/service-bus-resource-manager-namespace-topic/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-servicebus-create-topic-subscription-rule%2Fazuredeploy.json)

## <a name="parameters"></a>Parámetros

Con Azure Resource Manager, se definen los parámetros de los valores que quiere especificar al implementar la plantilla. La plantilla incluye una sección denominada `Parameters` que contiene todos los valores de los parámetros. Defina un parámetro para esos valores que variarán según el proyecto que vaya a implementar o según el entorno en el que vaya a realizar la implementación. No defina parámetros para valores que siempre permanezcan igual. Cada valor de parámetro se usa en la plantilla para definir los recursos que se implementan.

La plantilla define los parámetros siguientes:

### <a name="servicebusnamespacename"></a>serviceBusNamespaceName

El nombre del espacio de nombres de Service Bus que crear.

```json
"serviceBusNamespaceName": {
"type": "string"
}
```

### <a name="servicebustopicname"></a>serviceBusTopicName

El nombre del tema creado en el espacio de nombres de Service Bus.

```json
"serviceBusTopicName": {
"type": "string"
}
```

### <a name="servicebussubscriptionname"></a>serviceBusSubscriptionName

El nombre de la suscripción creada en el espacio de nombres de Service Bus.

```json
"serviceBusSubscriptionName": {
"type": "string"
}
```

### <a name="servicebusrulename"></a>serviceBusRuleName

El nombre de la regla (filtro) creada en el espacio de nombres de Service Bus.

```json
   "serviceBusRuleName": {
   "type": "string",
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

Crea un espacio de nombres de Service Bus estándar de tipo **Mensajería**con tema, suscripción y reglas.

```json
 "resources": [{
        "apiVersion": "[variables('sbVersion')]",
        "name": "[parameters('serviceBusNamespaceName')]",
        "type": "Microsoft.ServiceBus/Namespaces",
        "location": "[variables('location')]",
        "sku": {
            "name": "Standard",
        },
        "resources": [{
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusTopicName')]",
            "type": "Topics",
            "dependsOn": [
                "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
            ],
            "properties": {
                "path": "[parameters('serviceBusTopicName')]"
            },
            "resources": [{
                "apiVersion": "[variables('sbVersion')]",
                "name": "[parameters('serviceBusSubscriptionName')]",
                "type": "Subscriptions",
                "dependsOn": [
                    "[parameters('serviceBusTopicName')]"
                ],
                "properties": {},
                "resources": [{
                    "apiVersion": "[variables('sbVersion')]",
                    "name": "[parameters('serviceBusRuleName')]",
                    "type": "Rules",
                    "dependsOn": [
                        "[parameters('serviceBusSubscriptionName')]"
                    ],
                    "properties": {
                        "filterType": "SqlFilter",
                        "sqlFilter": {
                            "sqlExpression": "StoreName = 'Store1'",
                            "requiresPreprocessing": "false"
                        },
                        "action": {
                            "sqlExpression": "set FilterTag = 'true'"
                        }
                    }
                }]
            }]
        }]
    }]
```

Para conocer la sintaxis y las propiedades JSON, consulte [espacios de nombres](/azure/templates/microsoft.servicebus/namespaces), [temas](/azure/templates/microsoft.servicebus/namespaces/topics), [suscripciones](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions) y [reglas](/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions/rules).

## <a name="commands-to-run-deployment"></a>Comandos para ejecutar la implementación

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

```powershell-interactive
New-AzureResourceGroupDeployment -Name \<deployment-name\> -ResourceGroupName \<resource-group-name\> -TemplateUri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>
```

## <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>
```

## <a name="next-steps"></a>Pasos siguientes

Vea los siguientes artículos para aprender a administrar estos recursos:

* [Administración de Azure Service Bus](service-bus-management-libraries.md)
* [Administración de Service Bus con PowerShell](service-bus-manage-with-ps.md)
* [Administración de recursos de Service Bus con el Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/releases)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/?term=service+bus
[Learn more about Service Bus topics and subscriptions]: service-bus-queues-topics-subscriptions.md
[Using Azure PowerShell with Azure Resource Manager]: ../azure-resource-manager/management/manage-resources-powershell.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../azure-resource-manager/management/manage-resources-cli.md
[Recommended naming conventions for Azure resources]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging
[Service Bus namespace with topic, subscription, and rule]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-topic-subscription-rule/
[Service Bus queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md