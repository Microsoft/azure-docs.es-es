---
title: Creación de un centro de notificaciones de Azure mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un centro de notificaciones de Azure mediante una plantilla de Azure Resource Manager.
services: notification-hubs
author: sethmanheim
ms.service: notification-hubs
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: sethm
ms.date: 08/04/2020
ms.reviewer: thsomasu
ms.lastreviewed: 05/15/2020
ms.openlocfilehash: b9fe363da0fa5e451aaee83e895d539a0741a174
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830740"
---
# <a name="quickstart-create-a-notification-hub-using-an-arm-template"></a>Inicio rápido: Creación de un centro de notificaciones mediante una plantilla de Resource Manager

Azure Notification Hubs proporciona un motor de inserción fácil de usar y de escalabilidad horizontal que permite enviar notificaciones a cualquier plataforma (iOS, Android, Windows, Kindle, etc.) desde cualquier back-end (en la nube o local). Para más información sobre el servicio, consulte [¿Qué es Azure Notification Hubs?](notification-hubs-push-notification-overview.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

En este inicio rápido se usa una plantilla de Azure Resource Manager para crear un espacio de nombres de Azure Notification Hubs y un centro de notificaciones denominado **MyHub** en ese espacio de nombres.

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-notification-hub%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-notification-hub/).

:::code language="json" source="~/quickstart-templates/101-notification-hub/azuredeploy.json" range="1-45" highlight="22-40":::

* [Microsoft.NotificationHubs/namespaces](/azure/templates/microsoft.notificationhubs/namespaces)
* [Microsoft.NotificationHubs/namespaces/notificationHubs](/azure/templates/microsoft.notificationhubs/namespaces/notificationhubs)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla toma un nombre de espacio de nombres de Notification Hubs como parámetro. Luego, la plantilla crea un espacio de nombres con ese nombre y un centro de notificaciones llamado **MyHub** en ese espacio de nombres.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-notification-hub%2Fazuredeploy.json)

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar los recursos implementados, o bien usar un script de la CLI de Azure o Azure PowerShell script para enumerar el espacio de nombres de Notification Hubs y el centro implementados:

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzNotificationHub -Namespace "nhtestns123" -ResourceGroup "ContosoNotificationsGroup"
Get-AzNotificationHubsNamespace -Namespace "nhtestns123"
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
az notification-hub show --resource-group ContosoNotificationsGroup --namespace-name nhtestns123 --name MyHub
az notification-hub namespace show --resource-group ContosoNotificationsGroup --name nhtestns123
```

---
The output looks similar to: null
':::image type="content" source="media/create-notification-hub-template/verify-deploy.png" alt-text="Verify deployment"::': null
---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, que elimina los recursos que contiene.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener un tutorial paso a paso que le guíe en el proceso de creación de una plantilla, consulte:

> [!div class="nextstepaction"]
> [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
