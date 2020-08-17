---
title: Ejemplo de PowerShell para supervisar y escalar un grupo elástico en Azure SQL Database
description: Script de ejemplo de Azure PowerShell para supervisar y escalar un grupo elástico en Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: juliemsft
ms.author: jrasnick
ms.reviewer: carlrab
ms.date: 03/12/2019
ms.openlocfilehash: 14c6193b4a4373d754e3e90d573eccd2735de56c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085037"
---
# <a name="use-powershell-to-monitor-and-scale-an-elastic-pool-in-azure-sql-database"></a>Uso de PowerShell para supervisar y escalar un grupo elástico en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Este ejemplo de script de PowerShell supervisa las métricas de rendimiento de un grupo elástico, lo escala a un tamaño de proceso superior y crea una regla de alertas en una de las métricas de rendimiento.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de Azure PowerShell o una posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/monitor-and-scale-pool/monitor-and-scale-pool.ps1?highlight=17-18 "Monitor and scale an elastic pool in SQL Database")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor que hospeda las bases de datos o los grupos elásticos. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crea un grupo elástico. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crea una base de datos en un servidor. |
| [Get-AzMetric](/powershell/module/az.monitor/get-azmetric) | Muestra la información de uso del tamaño de la base de datos.|
| [Add-AzMetricAlertRule](/powershell/module/az.monitor/add-azmetricalertrule) | Agrega o actualiza una regla de alerta basada en métricas. |
| [Set-AzSqlElasticPool](/powershell/module/az.sql/set-azsqlelasticpool) | Actualiza las propiedades del grupo elástico. |
| [Add-AzMetricAlertRule](/powershell/module/az.monitor/add-azmetricalertrule) | Establece una regla de alerta para supervisar automáticamente las métricas en el futuro. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar más ejemplos de scripts de PowerShell en [Scripts de Azure PowerShell](../powershell-script-content-guide.md).
