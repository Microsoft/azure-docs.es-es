---
title: Creación de un centro de notificaciones de Azure mediante PowerShell | Microsoft Docs
description: Aprenda a usar un script de PowerShell para crear un centro de notificaciones de Azure.
author: dimazaid
manager: femila
services: notification-hubs
editor: sethmanheim
ms.service: notification-hubs
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/14/2020
ms.author: dimazaid
ms.openlocfilehash: 857210efdc5740defcde4187b7b04cce55215325
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071607"
---
# <a name="use-powershell-to-create-an-azure-notification-hub"></a>Uso de PowerShell para crear un centro de notificaciones de Azure

Este script de PowerShell de ejemplo crea un centro de notificaciones de Azure de ejemplo. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>Prerequisites

* **Suscripción a Azure**: si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/notification-hubs/create-notification-hub/create-notification-hub.ps1 "Create a notification hub")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él:

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzNotificationHubsNamespace](/powershell/module/az.notificationhubs/new-aznotificationhubsnamespace) | Crea un espacio de nombres para el centro de notificaciones. |
| [New-AzNotificationHub](/powershell/module/az.notificationhubs/new-aznotificationhub) | Crea un centro de notificaciones. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).
