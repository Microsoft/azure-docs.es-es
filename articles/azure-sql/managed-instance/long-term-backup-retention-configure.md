---
title: 'Instancia administrada de Azure SQL: Retención de copia de seguridad a largo plazo (PowerShell)'
description: Obtenga información sobre cómo almacenar y restaurar copias de seguridad automatizadas en contenedores de Azure Blob Storage independientes para una instancia administrada de Azure SQL mediante PowerShell.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab
ms.date: 04/29/2020
ms.openlocfilehash: ab4eaf5ad40b5ef8bee68ef0e56ab8f53db8a8a2
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533842"
---
# <a name="manage-azure-sql-managed-instance-long-term-backup-retention-powershell"></a>Administración de la retención de copias de seguridad a largo plazo de la instancia administrada de Azure SQL (PowerShell)
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En la instancia administrada de Azure SQL, puede configurar una directiva de [retención de copias de seguridad a largo plazo](../database/long-term-retention-overview.md#sql-managed-instance-support) (LTR) como característica en vista previa (GB) pública limitada. Esto permite conservar automáticamente copias de seguridad de bases de datos en contenedores de Azure Blob Storage independientes durante un máximo de 10 años. Posteriormente, puede recuperar una base de datos mediante estas copias de seguridad con PowerShell.

   > [!IMPORTANT]
   > LTR para instancias administradas se encuentra actualmente en versión preliminar limitada y está disponible para las suscripciones de EA y CSP en cada caso. Para solicitar la inscripción, cree un [vale de soporte de Azure](https://azure.microsoft.com/support/create-ticket/). En Tipo de problema, seleccione Problema técnico, en Servicio elija Instancia administrada de SQL Database y, en Tipo de problema, seleccione **Copia de seguridad, restauración y continuidad empresarial/Retención de copias de seguridad a largo plazo**. En la solicitud, indique que le gustaría inscribirse en la versión preliminar pública limitada de LTR para la instancia administrada.

En las siguientes secciones se explica cómo usar PowerShell para configurar la retención de copias de seguridad a largo plazo, ver las copias de seguridad en el almacén de Azure SQL y realizar una restauración a partir de una copia de seguridad del almacén de Azure SQL.

## <a name="azure-roles-to-manage-long-term-retention"></a>Roles de Azure para administrar la retención a largo plazo

En **Get-AzSqlInstanceDatabaseLongTermRetentionBackup** y **Restore-AzSqlInstanceDatabase**, deberá tener uno de los siguientes roles:

- Rol de propietario de la suscripción o
- rol de colaborador de instancia administrada o
- Rol personalizado con los permisos siguientes:
  - `Microsoft.Sql/locations/longTermRetentionManagedInstanceBackups/read`
  - `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionManagedInstanceBackups/read`
  - `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionDatabases/longTermRetentionManagedInstanceBackups/read`

En **Remove-AzSqlInstanceDatabaseLongTermRetentionBackup**, deberá tener uno de los roles siguientes:

- Rol de propietario de la suscripción o
- Rol personalizado con el permiso siguiente:
  - `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionDatabases/longTermRetentionManagedInstanceBackups/delete`

> [!NOTE]
> El rol de colaborador de instancia administrada no tiene permiso para eliminar las copias de seguridad de LTR.

Se pueden conceder permisos RBAC en el ámbito de la *suscripción* o del *grupo de recursos*. Sin embargo, para acceder a las copias de seguridad de LTR que pertenecen a una instancia descartada, se debe conceder el permiso en el ámbito de la *suscripción* de dicha instancia.

- `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionDatabases/longTermRetentionManagedInstanceBackups/delete`

## <a name="create-an-ltr-policy"></a>Creación de una directiva LTR

```powershell
# get the Managed Instance
$subId = "<subscriptionId>"
$instanceName = "<instanceName>"
$resourceGroup = "<resourceGroupName>"
$dbName = "<databaseName>"

Connect-AzAccount
Select-AzSubscription -SubscriptionId $subId

$instance = Get-AzSqlInstance -Name $instanceName -ResourceGroupName $resourceGroup

# create LTR policy with WeeklyRetention = 12 weeks. MonthlyRetention and YearlyRetention = 0 by default.
Set-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
   -DatabaseName $dbName -ResourceGroupName $resourceGroup -WeeklyRetention P12W

# create LTR policy with WeeklyRetention = 12 weeks, YearlyRetention = 5 years and WeekOfYear = 16 (week of April 15). MonthlyRetention = 0 by default.
Set-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
    -DatabaseName $dbName -ResourceGroupName $resourceGroup -WeeklyRetention P12W -YearlyRetention P5Y -WeekOfYear 16
```

## <a name="view-ltr-policies"></a>Visualización de directivas de LTR

Este ejemplo muestra cómo enumerar las directivas de LTR de una instancia

```powershell
# gets the current version of LTR policy for the database
$ltrPolicies = Get-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
    -DatabaseName $dbName -ResourceGroupName $resourceGroup
```

## <a name="clear-an-ltr-policy"></a>Borrado de una directiva LTR

Este ejemplo muestra cómo borrar una directiva LTR desde una base de datos

```powershell
Set-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
   -DatabaseName $dbName -ResourceGroupName $resourceGroup -RemovePolicy
```

## <a name="view-ltr-backups"></a>Visualización de copias de seguridad de LTR

Este ejemplo muestra cómo enumerar las copias de seguridad de LTR de una instancia.

```powershell
# get the list of all LTR backups in a specific Azure region
# backups are grouped by the logical database id, within each group they are ordered by the timestamp, the earliest backup first
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location

# get the list of LTR backups from the Azure region under the given managed instance
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -InstanceName $instanceName

# get the LTR backups for a specific database from the Azure region under the given managed instance
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -InstanceName $instanceName -DatabaseName $dbName

# list LTR backups only from live databases (you have option to choose All/Live/Deleted)
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -DatabaseState Live

# only list the latest LTR backup for each database
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -InstanceName $instanceName -OnlyLatestPerDatabase
```

## <a name="delete-ltr-backups"></a>Eliminación de copias de seguridad de LTR

En este ejemplo se muestra cómo eliminar una copia de seguridad de LTR desde la lista de copias de seguridad.

```powershell
# remove the earliest backup
$ltrBackup = $ltrBackups[0]
Remove-AzSqlInstanceDatabaseLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId
```

> [!IMPORTANT]
> La eliminación de la copia de seguridad de LTR no es reversible. Para eliminar una copia de seguridad de LTR una vez eliminada la instancia, debe tener el permiso del ámbito de la suscripción. Puede configurar notificaciones sobre cada eliminación en Azure Monitor filtrando por la operación "Elimina una copia de seguridad de retención a largo plazo". El registro de actividad contiene información sobre quién y cuándo realizó la solicitud. Consulte [Creación de alertas del registro de actividad](../../azure-monitor/platform/alerts-activity-log.md) para obtener instrucciones detalladas.

## <a name="restore-from-ltr-backups"></a>Restauración desde copias de seguridad de LTR

Este ejemplo muestra cómo restaurar desde una copia de seguridad de LTR. Observe que aunque esta interfaz no cambió, el parámetro ResourceId requiere ahora el identificador de recurso de la copia de seguridad de LTR.

```powershell
# restore a specific LTR backup as an P1 database on the instance $instanceName of the resource group $resourceGroup
Restore-AzSqlInstanceDatabase -FromLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId `
   -TargetInstanceName $instanceName -TargetResourceGroupName $resourceGroup -TargetInstanceDatabaseName $dbName
```

> [!IMPORTANT]
> Para restaurar a partir de una copia de seguridad de LTR una vez eliminada la instancia, debe tener los permisos de la suscripción de la instancia de SQL Server y dicha suscripción debe estar activa. También debe omitir el parámetro -ResourceGroupName opcional.

> [!NOTE]
> Desde aquí, puede conectarse a la base de datos restaurada mediante SQL Server Management Studio para realizar las tareas necesarias, como extraer un bit de datos de la base de datos restaurada para copiarlo en la base de datos existente o para eliminar la base de datos existente y cambiar el nombre de la base de datos restaurada por el nombre de la base de datos existente. Consulte la [restauración a un momento dado](../database/recovery-using-backups.md#point-in-time-restore).

## <a name="next-steps"></a>Pasos siguientes

- Para aprender sobre las copias de seguridad automáticas generadas por el servicio, consulte [copias de seguridad automáticas](../database/automated-backups-overview.md)
- Para más información sobre la retención de copia de seguridad a largo plazo, consulte sobre la [retención de copia de seguridad a largo plazo](../database/long-term-retention-overview.md).
