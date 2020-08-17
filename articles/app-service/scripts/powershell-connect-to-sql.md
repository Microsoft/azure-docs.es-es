---
title: 'PowerShell: Conectarse a SQL Database'
description: Aprenda a usar Azure PowerShell para automatizar la implementación y administración de App Service. En este ejemplo se indica cómo conectar una aplicación a una base de datos SQL.
tags: azure-service-management
ms.assetid: 055440a9-fff1-49b2-b964-9c95b364e533
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc
ms.openlocfilehash: 70859701029eccf66035ed57354783b4c94fbfe8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083334"
---
# <a name="connect-an-app-service-app-to-sql-database"></a>Conexión de una aplicación de App Service a SQL Database

En este escenario aprenderá a crear una base de datos en Azure SQL Database y una aplicación de App Service. Luego, vinculará la base de datos a la aplicación mediante la configuración de la aplicación.

Si es necesario, instale Azure PowerShell con la instrucción que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/) y, luego, ejecute `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/connect-to-sql/connect-to-sql.ps1?highlight=13 "Connect an app to SQL Database")]

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
| [New-AzSQLServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crea una regla de firewall en el nivel de servidor. |
| [New-AzSQLDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crea una base de datos o una base de datos elástica. |
| [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) | Modifica la configuración de una aplicación de App Service. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Se pueden encontrar ejemplos de Azure PowerShell para Azure App Service en los [ejemplos de PowerShell](../samples-powershell.md).
