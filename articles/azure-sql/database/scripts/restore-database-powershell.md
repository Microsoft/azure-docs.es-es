---
title: 'PowerShell: Restauración de una copia de seguridad automática de una base de datos en SQL Database'
description: Utilice un script de ejemplo de Azure PowerShell para restaurar una base de datos de SQL Database a un momento anterior a partir de copias de seguridad automáticas.
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: PowerShell
ms.topic: sample
author: mashamsft
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 03/27/2019
ms.openlocfilehash: ed66759ab2da28f004c704d4ea24ec4124355c98
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084984"
---
# <a name="use-powershell-to-restore-a-database-to-an-earlier-point-in-time"></a>Uso de PowerShell para restaurar una base de datos a un momento anterior

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

En este script de ejemplo de PowerShell se restaura una base de datos de SQL Database a un momento anterior.  

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de Azure PowerShell o posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/restore-database/restore-database.ps1?highlight=17-18 "Create SQL Database")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor que hospeda las bases de datos y los grupos elásticos. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crea una base de datos en un servidor. |
| [Get-AzSqlDatabaseGeoBackup](/powershell/module/az.sql/get-azsqldatabasegeobackup) | Obtiene una copia de seguridad con redundancia geográfica de una base de datos agrupada o independiente. |
| [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase) | Restaura una base de datos. |
| [Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase) | Quita una base de datos. |
| [Get-AzSqlDeletedDatabaseBackup](/powershell/module/az.sql/get-azsqldeleteddatabasebackup) | Obtiene una base de datos eliminada que se puede restaurar. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de SQL Database en los [scripts de PowerShell de Azure SQL Database](../powershell-script-content-guide.md).
