---
title: 'PowerShell: Actualización del esquema de sincronización de SQL Data Sync'
description: Script de ejemplo de Azure PowerShell para actualizar el esquema de sincronización de SQL Data Sync
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 03/12/2019
ms.openlocfilehash: 44a9b0b4d609013147f799ae19f750c34e81837c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084881"
---
# <a name="use-powershell-to-update-the-sync-schema-in-an-existing-sync-group"></a>Usar PowerShell para actualizar el esquema de sincronización en un grupo de sincronización existente
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

En este ejemplo de Azure PowerShell se actualiza el esquema de sincronización en un grupo de sincronización de SQL Data Sync existente. Cuando se sincronizan varias tablas, este script le ayuda a actualizar eficazmente el esquema de sincronización. En este ejemplo se muestra el uso del script de **UpdateSyncSchema**, que está disponible en GitHub como [UpdateSyncSchema.ps1](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/sql-data-sync/UpdateSyncSchema.ps1).

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de Azure PowerShell o posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

Para obtener información general acerca de SQL Data Sync, consulte [Sincronización de datos entre varias bases de datos locales y de la nube con Azure SQL Data Sync](../sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> SQL Data Sync no admite en este momento Instancia administrada de Azure SQL.

## <a name="examples"></a>Ejemplos

### <a name="add-all-tables-to-the-sync-schema"></a>Agregar todas las tablas al esquema de sincronización

En el ejemplo siguiente se actualiza el esquema de base de datos y agregan todas las tablas válidas de la base de datos central en el esquema de sincronización.

```powershell-interactive
UpdateSyncSchema.ps1 -SubscriptionId <subscriptionId> -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> `
    -SyncGroupName <syncGroupName> -RefreshDatabaseSchema $true -AddAllTables $true
```

### <a name="add-and-remove-tables-and-columns"></a>Agregar y quitar tablas y columnas

En el ejemplo siguiente se agregan `[dbo].[Table1]` y `[dbo].[Table2].[Column1]` al esquema de sincronización y se quita `[dbo].[Table3]`.

```powershell-interactive
UpdateSyncSchema.ps1 -SubscriptionId <subscriptionId> -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> `
    -SyncGroupName <syncGroupName> -TablesAndColumnsToAdd "[dbo].[Table1],[dbo].[Table2].[Column1]" -TablesAndColumnsToRemove "[dbo].[Table3]"
```

## <a name="script-parameters"></a>Parámetros de script

El script **UpdateSyncSchema** tiene los parámetros siguientes:

| Parámetro | Notas |
|---|---|
| $subscriptionId | Suscripción en la que se creó el grupo de sincronización. |
| $resourceGroupName | Grupo de recursos en el que se creó el grupo de sincronización.|
| $serverName | Nombre de servidor de la base de datos central.|
| $databaseName | Nombre de la base de datos central. |
| $syncGroupName | Nombre del grupo de sincronización. |
| $memberName | Especifique el nombre de miembro si quiere cargar el esquema de base de datos desde el miembro de sincronización en lugar de desde la base de datos central. Si quiere cargar el esquema de base de datos desde la central, deje vacío este parámetro. |
| $timeoutInSeconds | Tiempo de espera después del cual el script actualiza el esquema de base de datos. El valor predeterminado es 900 segundos. |
| $refreshDatabaseSchema | Permite especificar si el script debe actualizar el esquema de la base de datos. Si el esquema de la base de datos cambió desde la configuración anterior (por ejemplo, si agregó una tabla o una columna nueva), deberá actualizar el esquema antes de volver a configurarlo. El valor predeterminado es False. |
| $addAllTables | Si este valor es true, todas las tablas y columnas válidas se agregan al esquema de sincronización. Se omiten los valores de $TablesAndColumnsToAdd y $TablesAndColumnsToRemove. |
| $tablesAndColumnsToAdd | Permite especificar las tablas o columnas que se agregarán al esquema de sincronización. Cada nombre de tabla o columna debe delimitarse totalmente con el nombre del esquema. Por ejemplo: `[dbo].[Table1]`, `[dbo].[Table2].[Column1]`. Pueden especificarse varios nombres de tabla o columna y separarse por una coma (,). |
| $tablesAndColumnsToRemove | Permite especificar las tablas o columnas que se eliminarán del esquema de sincronización. Cada nombre de tabla o columna debe delimitarse totalmente con el nombre de esquema. Por ejemplo: `[dbo].[Table1]`, `[dbo].[Table2].[Column1]`. Pueden especificarse varios nombres de tabla o columna y separarse por una coma (,). |

## <a name="script-explanation"></a>Explicación del script

El script **UpdateSyncSchema** usa los comandos siguientes. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzSqlSyncGroup](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlsyncgroup) | Devuelve información sobre un grupo de sincronización. |
| [Update-AzSqlSyncGroup](https://docs.microsoft.com/powershell/module/az.sql/update-azsqlsyncgroup) | Actualiza un grupo de sincronización. |
| [Get-AzSqlSyncMember](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlsyncmember) | Devuelve información sobre un miembro de sincronización. |
| [Get-AzSqlSyncSchema](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlsyncschema) | Devuelve información sobre un esquema de sincronización. |
| [Update-AzSqlSyncSchema](https://docs.microsoft.com/powershell/module/az.sql/update-azsqlsyncschema) | Actualiza un esquema de sincronización. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de SQL Database en los [scripts de PowerShell de Azure SQL Database](../powershell-script-content-guide.md).

Para más información acerca de SQL Data Sync, consulte:

- Introducción: [Sincronización datos entre Azure SQL Database y SQL Server con Azure SQL Data Sync](../sql-data-sync-data-sql-server-sql-database.md)
- Configuración de Data Sync
    - Uso de Azure Portal: [Tutorial: Configuración de SQL Data Sync para sincronizar datos entre Azure SQL Database y SQL Server](../sql-data-sync-sql-server-configure.md)
    - Uso de PowerShell
        -  [Uso de PowerShell para sincronizar datos entre varias bases de datos en Azure SQL Database](sql-data-sync-sync-data-between-sql-databases.md)
        -  [Uso de PowerShell para sincronizar datos entre Azure SQL Database y SQL Server](sql-data-sync-sync-data-between-azure-onprem.md)
- Data Sync Agent: [Data Sync Agent para Azure SQL Data Sync](../sql-data-sync-agent-overview.md)
- Procedimientos recomendados: [Procedimientos recomendados para Azure SQL Data Sync](../sql-data-sync-best-practices.md)
- Supervisión: [Monitor SQL Data Sync with Azure Monitor logs](../sql-data-sync-monitor-sync.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
- Solución de problemas: [Solución de problemas con Azure SQL Data Sync](../sql-data-sync-troubleshoot.md)
- Actualización del esquema de sincronización
    - Uso de Transact-SQL: [Automatización de la replicación de los cambios de esquema en Azure SQL Data Sync](../sql-data-sync-update-sync-schema.md)

Para obtener más información sobre SQL Database, vea:

- [Información general de SQL Database](../sql-database-paas-overview.md)
- [Administración del ciclo de vida de las aplicaciones](https://msdn.microsoft.com/library/jj907294.aspx)
