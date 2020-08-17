---
title: Creación de un espacio de nombres de Azure Service Bus con una plantilla
description: Utilice una plantilla de Azure Resource Manager para crear un espacio de nombres Messaging de Service Bus.
documentationcenter: .net
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.openlocfilehash: 1b7aafca331170100ce99c084a11c96c97df7781
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067399"
---
# <a name="create-a-service-bus-namespace-by-using-an-azure-resource-manager-template"></a>Creación de un espacio de nombres de Service Bus mediante una plantilla de Azure Resource Manager

Aprenda a implementar una plantilla de Azure Resource Manager para crear un espacio de nombres de Service Bus. Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades. Para más información sobre la creación de plantillas, consulte la [documentación de Azure Resource Manager](../azure-resource-manager/index.yml).

Las plantillas siguientes también están disponibles para crear espacios de nombres de Service Bus:

* [Creación de un espacio de nombres de Service Bus con cola](./service-bus-resource-manager-namespace-queue.md)
* [Creación de un espacio de nombres de Service Bus con un tema y una suscripción](./service-bus-resource-manager-namespace-topic.md)
* [Creación de un espacio de nombres de Service Bus con regla de autorización y cola](./service-bus-resource-manager-namespace-auth-rule.md)
* [Create a Service Bus namespace with topic, subscription, and rule](./service-bus-resource-manager-namespace-topic-with-rule.md) (Creación de un espacio de nombres de Service Bus con tema, suscripción y regla)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-service-bus-namespace"></a>Crear un espacio de nombres de Service Bus

En este inicio rápido, puede usar una [plantilla de Resource Manager de inicio rápido](https://github.com/Azure/azure-quickstart-templates/blob/master/101-servicebus-create-namespace/azuredeploy.json) de [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/):

[!code-json[create-azure-service-bus-namespace](~/quickstart-templates/101-servicebus-create-namespace/azuredeploy.json)]

Para buscar más ejemplos de plantillas, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Servicebus&pageNumber=1&sort=Popular).

Para crear un espacio de nombres de Service Bus mediante la implementación de una plantilla:

1. Seleccione **Probar** en el bloque de código siguiente y luego siga las instrucciones para iniciar sesión en Azure Cloud Shell.

    ```azurepowershell-interactive
    $serviceBusNamespaceName = Read-Host -Prompt "Enter a name for the service bus namespace to be created"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${serviceBusNamespaceName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -serviceBusNamespaceName $serviceBusNamespaceName

    Write-Host "Press [ENTER] to continue ..."
    ```

    El nombre del grupo de recursos es el nombre del espacio de nombres de Service Bus con **rg** anexado.

2. Seleccione **Copiar** para copiar el script de PowerShell.
3. Haga clic con el botón derecho en la consola del shell y seleccione **Pegar**.

Lleva algunos minutos crear un centro de eventos.

## <a name="verify-the-deployment"></a>Comprobar la implementación

Para ver el espacio de nombres de Service Bus implementación, puede abrir el grupo de recursos desde Azure Portal o usar el script de Azure PowerShell siguiente. Si Cloud Shell todavía está abierto, no es necesario copiar ni ejecutar la primera y la segunda líneas del script siguiente.

```azurepowershell-interactive
$serviceBusNamespaceName = Read-Host -Prompt "Enter the same service bus namespace name used earlier"
$resourceGroupName = "${serviceBusNamespaceName}rg"

Get-AzServiceBusNamespace -ResourceGroupName $resourceGroupName -Name $serviceBusNamespaceName

Write-Host "Press [ENTER] to continue ..."
```

En este tutorial se usa Azure PowerShell para implementar la plantilla. Para otros métodos de implementación de plantillas, consulte:

* [Mediante Azure Portal](../azure-resource-manager/templates/deploy-portal.md).
* [Mediante la CLI de Azure](../azure-resource-manager/templates/deploy-cli.md).
* [Mediante la API REST](../azure-resource-manager/templates/deploy-rest.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos. Si Cloud Shell todavía está abierto, no es necesario copiar ni ejecutar la primera y la segunda líneas del script siguiente.

```azurepowershell-interactive
$serviceBusNamespaceName = Read-Host -Prompt "Enter the same service bus namespace name used earlier"
$resourceGroupName = "${serviceBusNamespaceName}rg"

Remove-AzResourceGroup -ResourceGroupName $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se creó un espacio de nombres de Service Bus. Consulte las demás guías de inicio rápido para aprender a crear colas, temas y suscripciones, además de a utilizarlos:

* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)