---
title: 'PowerShell: Incorporación de un grupo elástico a un grupo de conmutación por error automática'
description: Script de ejemplo de Azure PowerShell para crear un grupo elástico de Azure SQL Database, agregarlo a un grupo de conmutación por error automática y probar la conmutación por error.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: MashaMSFT
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 07/16/2019
ms.openlocfilehash: e509fc0f93c60d271409b5453aecdf793cd22ecb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040602"
---
# <a name="use-powershell-to-add-an-elastic-pool-to-a-failover-group"></a>Uso de PowerShell para agregar un grupo elástico a un grupo de conmutación por error
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

En este script de ejemplo de Azure PowerShell se crea una base de datos en Azure SQL Database, se agrega a un grupo elástico, se crea un grupo de conmutación por error y se prueba la conmutación por error.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de Azure PowerShell o posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-scripts"></a>Muestras de scripts

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/failover-groups/add-elastic-pool-to-failover-group-az-ps.ps1 "Add elastic pool to a failover group")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor que hospeda las bases de datos y los grupos elásticos. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crea una regla de firewall en el nivel de servidor para un servidor. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crea una nueva base de datos. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crea un grupo de bases de datos elásticas.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Establece las propiedades de una base de datos, o mueve una base de datos existente en un grupo elástico. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Crea un grupo de conmutación por error. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Obtiene una o más bases de datos. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Agrega una o varias bases de datos a un grupo de conmutación por error. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtiene o enumera grupos de conmutación por error de bases de datos. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Ejecuta una conmutación por error de un grupo de conmutación por error de bases de datos. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de SQL Database en los [scripts de PowerShell de Azure SQL Database](../powershell-script-content-guide.md).
