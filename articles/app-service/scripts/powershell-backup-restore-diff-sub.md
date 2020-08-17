---
title: 'PowerShell: Restauración de la copia de seguridad en otra suscripción'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se muestra cómo restaurar una copia de seguridad en otra suscripción.
author: msangapu-msft
tags: azure-service-management
ms.assetid: a2a27d94-d378-4c17-a6a9-ae1e69dc4a72
ms.topic: sample
ms.date: 11/21/2018
ms.author: msangapu
ms.custom: mvc, seodec18
ms.openlocfilehash: c9dda1f3e011006d1b5072c5b71accb28963aedb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87004875"
---
# <a name="restore-a-web-app-from-a-backup-in-another-subscription-using-powershell"></a>Restauración de una aplicación web a partir de una copia de seguridad en otra suscripción mediante PowerShell

Este script de ejemplo recupera una copia de seguridad completada previamente desde una aplicación web existente y la restaura en una aplicación web de otra suscripción. 

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure. 

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/backup-restore-diff-sub/backup-restore-diff-sub.ps1?highlight=1-6 "Restore a web app from a backup in another subscription")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Cuando ya no necesite la aplicación web, use el comando siguiente para quitar el grupo de recursos, la aplicación web y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name $resourceGroupName -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Add-AzAccount](/powershell/module/az.accounts/connect-azaccount) | Agrega una cuenta autenticada para utilizarla con las solicitudes de cmdlet de Azure Resource Manager.  |
| [Get-AzWebAppBackupList](/powershell/module/az.websites/get-azwebappbackuplist) | Obtiene una lista de copias de seguridad de una aplicación web. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Crea una aplicación web. |
| [Restore-AzWebAppBackup](/powershell/module/az.websites/restore-azwebappbackup) | Restaura una aplicación web desde una copia de seguridad completada previamente. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar ejemplos de Azure PowerShell para Azure App Service Web Apps en los [ejemplos de PowerShell](../samples-powershell.md).
