---
title: 'PowerShell: Restauración de la copia de seguridad geográfica para Instancia administrada de Azure SQL'
description: Script de ejemplo de Azure PowerShell para restaurar una base de datos de Instancia administrada de Azure SQL a partir de una copia de seguridad con redundancia geográfica.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: PowerShell
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 07/03/2019
ms.openlocfilehash: 0c6ca03f535201059f107d6b53bc423b92d1c1c1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073368"
---
# <a name="use-powershell-to-restore-an-azure-sql-managed-instance-database-to-another-geo-region"></a>Uso de PowerShell para restaurar una base de datos de Instancia administrada de Azure SQL en otra región con replicación geográfica

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Este ejemplo de script de PowerShell restaura una base de datos de Instancia administrada de Azure SQL Database desde una región remota con replicación geográfica (georestauración).  

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 o posterior de Azure PowerShell. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

```azurepowershell-interactive
# Connect-AzAccount
# The SubscriptionId in which to create these objects
$SubscriptionId = '<put subscription_id here>'
# Set the information for your managed instance
$SourceResourceGroupName = "myResourceGroup-$(Get-Random)"
$SourceInstanceName = "myManagedInstance-$(Get-Random)"
$SourceDatabaseName = "myInstanceDatabase-$(Get-Random)"

# Set the information for your destination managed instance
$TargetResourceGroupName = "myTargetResourceGroup-$(Get-Random)"
$TargetInstanceName = "myTargetManagedInstance-$(Get-Random)"
$TargetDatabaseName = "myTargetInstanceDatabase-$(Get-Random)"

Connect-AzAccount
Set-AzContext -SubscriptionId $SubscriptionId

$backup = Get-AzSqlInstanceDatabaseGeoBackup `
-ResourceGroupName $SourceResourceGroupName `
-InstanceName $SourceInstanceName `
-Name $SourceDatabaseName

$backup | Restore-AzSqlInstanceDatabase -FromGeoBackup `
-TargetInstanceDatabaseName $TargetDatabaseName `
-TargetInstanceName $TargetInstanceName `
-TargetResourceGroupName $TargetResourceGroupName

```

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```azurepowershell-interactive
Remove-AzResourceGroup -ResourceGroupName $TargetResourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/New-AzResourceGroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [Get-AzSqlInstanceDatabaseGeoBackup](/powershell/module/az.sql/Get-AzSqlInstanceDatabaseGeoBackup) | Crea una copia de seguridad con redundancia geográfica de una base de datos de Instancia administrada de SQL. |
| [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/Restore-AzSqlInstanceDatabase) | Crea una base de datos en Instancia administrada de SQL a partir de una copia de seguridad con replicación geográfica. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Se pueden encontrar más ejemplos de scripts de PowerShell para Azure SQL Database en [Scripts de Azure PowerShell para Azure SQL Database](../../database/powershell-script-content-guide.md).
