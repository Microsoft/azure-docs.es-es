---
title: 'PowerShell: Creación de una copia de seguridad programada'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se indica cómo crear una copia de seguridad programada para una aplicación.
author: msangapu-msft
tags: azure-service-management
ms.assetid: a2a27d94-d378-4c17-a6a9-ae1e69dc4a72
ms.topic: sample
ms.date: 10/30/2017
ms.author: msangapu
ms.custom: mvc, seodec18
ms.openlocfilehash: 636e41afc4d04b6e34a8087302a5d2dcdc31e468
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083402"
---
# <a name="create-a-scheduled-backup-for-a-web-app-using-powershell"></a>Create a scheduled backup for a web app using PowerShell (Creación de una copia de seguridad programada para una aplicación web mediante PowerShell)

En este script de ejemplo se crea una aplicación web en App Service con sus recursos relacionados y, a continuación, se crea una copia de seguridad programada para ella. 

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure. 

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/backup-scheduled/backup-scheduled.ps1?highlight=1-4 "Create a scheduled backup for a web app")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos, la aplicación web y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Crea una cuenta de Storage. |
| [New-AzStorageContainer](/powershell/module/az.storage/new-AzStoragecontainer) | Crea un contenedor de Azure Storage. |
| [New-AzStorageContainerSASToken](/powershell/module/az.storage/new-AzStoragecontainersastoken) | Genera un token de SAS para un contenedor de Azure Storage. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Crea un plan de App Service, |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Crea una aplicación web. |
| [Edit-AzWebAppBackupConfiguration](/powershell/module/az.websites/edit-azwebappbackupconfiguration) | Edita la configuración de copia de seguridad para la aplicación web. |
| [Get-AzWebAppBackupList](/powershell/module/az.websites/get-azwebappbackuplist) | Obtiene una lista de copias de seguridad de una aplicación web. |
| [Get-AzWebAppBackupConfiguration](/powershell/module/az.websites/get-azwebappbackupconfiguration) | Obtiene la configuración de copia de seguridad para la aplicación web. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar ejemplos de Azure PowerShell para Azure App Service Web Apps en los [ejemplos de PowerShell](../samples-powershell.md).
