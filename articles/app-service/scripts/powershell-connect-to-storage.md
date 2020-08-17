---
title: 'PowerShell: Conexión a una cuenta de almacenamiento'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se indica cómo conectar una aplicación a una cuenta de almacenamiento.
tags: azure-service-management
ms.assetid: e4831bdc-2068-4883-9474-0b34c2e3e255
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc
ms.openlocfilehash: 2377c8536fa307f670e28f5ff3cf04f03b88325e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083326"
---
# <a name="connect-an-app-service-app-to-a-storage-account"></a>Conexión de una aplicación de App Service a una cuenta de almacenamiento

En este escenario aprenderá a crear una cuenta de Azure Storage y una aplicación de App Service. Después, vinculará la cuenta de almacenamiento a la aplicación mediante la configuración de la aplicación.

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/connect-to-storage/connect-to-storage.ps1 "Connect an app to a storage account")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos, la aplicación de App Service y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Crea un plan de App Service, |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Crea una aplicación de App Service. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Crea una cuenta de Storage. |
| [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) | Obtiene las claves de acceso de una cuenta de Azure Storage. |
| [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) | Modifica la configuración de una aplicación de App Service. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Se pueden encontrar ejemplos de Azure PowerShell para Azure App Service en los [ejemplos de PowerShell](../samples-powershell.md).
