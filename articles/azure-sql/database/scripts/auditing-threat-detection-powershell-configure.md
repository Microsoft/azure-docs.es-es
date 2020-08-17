---
title: 'Ejemplo de PowerShell de auditoría y Advanced Threat Protection: Azure SQL Database'
description: Ejemplo de script de Azure PowerShell para configurar la auditoría y Advanced Threat Protection en una base de datos Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: security, sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: DavidTrigano
ms.author: datrigan
ms.reviewer: carlrab, vanto
ms.date: 04/28/2020
ms.openlocfilehash: d44370e17b0d38f1d19849e97b2a7f41b50a4119
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040571"
---
# <a name="use-powershell-to-configure-sql-database-auditing-and-advanced-threat-protection"></a>Uso de PowerShell para configurar la auditoría y Advanced Threat Protection en SQL Database
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Este ejemplo de script de PowerShell configura la auditoría y Advanced Threat Protection en Azure SQL Database.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de Azure PowerShell o una posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/database-auditing-and-threat-detection/database-auditing-and-threat-detection.ps1?highlight=15-16 "Configure auditing and threat detection")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crea una base de datos o un grupo elástico. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Crea una cuenta de almacenamiento. |
| [Set-AzSqlDatabaseAuditing](/powershell/module/az.sql/set-azsqldatabaseauditing) | Establece la directiva de auditoría para una base de datos. |
| [Set-AzSqlDatabaseThreatDetectionPolicy](/powershell/module/az.sql/set-azsqldatabasethreatdetectionpolicy) | Establece una directiva de Advanced Threat Protection en una base de datos. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de SQL Database en los [scripts de PowerShell de Azure SQL Database](../powershell-script-content-guide.md).
