---
title: 'PowerShell: Importación de un archivo BACPAC a una base de datos de Azure SQL Database'
description: Script de ejemplo de Azure PowerShell para importar un archivo BACPAC a una base de datos de SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: load & move data, sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 05/24/2019
ms.openlocfilehash: 7e182b44810a322a0cf3a26e7131a411f6259dc9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073542"
---
# <a name="use-powershell-to-import-a-bacpac-file-into-a-database-in-sql-database"></a>Uso de PowerShell para importar un archivo BACPAC a una base de datos de SQL Database
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Este script de ejemplo de Azure PowerShell importa una base de datos de un archivo BACPAC a una nueva base de datos en SQL Database.  

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de Azure PowerShell o posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/import-from-bacpac/import-from-bacpac.ps1?highlight=20-21 "Create SQL Database")]

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
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor que hospeda las bases de datos y los grupos elásticos. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crea una regla de firewall en el nivel de servidor para un servidor. |
| [New-AzSqlDatabaseImport](/powershell/module/az.sql/new-azsqldatabaseimport) | Importa un archivo BACPAC y crea una nueva base de datos en el servidor. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de SQL Database en los [scripts de PowerShell de Azure SQL Database](../powershell-script-content-guide.md).
