---
title: Restauración de un almacenamiento de datos desde una copia de seguridad geográfica
description: Guía de procedimientos para la restauración geográfica de un grupo de SQL.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 07/12/2019
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 3c8d78c164cefbab03d9d3fa783c75ded9773d38
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075813"
---
# <a name="geo-restore-for-sql-pool"></a>Restauración geográfica del grupo de SQL

En este artículo, aprenderá a restaurar el grupo de SQL desde una copia de seguridad geográfica mediante Azure Portal y PowerShell.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**Compruebe la capacidad DTU**. Cada grupo de SQL está hospedado en un [servidor SQL lógico](../../azure-sql/database/logical-servers.md) (por ejemplo, myserver.database.windows.net) que tiene una cuota de DTU predeterminada. Compruebe que el servidor SQL Server tiene suficiente cuota de DTU restante para la base de datos en proceso de restauración. Para más información sobre cómo calcular la unidad DTU necesaria o solicitar más DTU, consulte cómo [solicitar un cambio en la cuota de DTU](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="restore-from-an-azure-geographical-region-through-powershell"></a>Uso de PowerShell para restaurar desde una región geográfica de Azure

Para realizar una restauración a partir de una copia de seguridad de replicación geográfica use el cmdlet [Get-AzSqlDatabaseGeoBackup](/powershell/module/az.sql/get-azsqldatabasegeobackup?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) y [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

> [!NOTE]
> Puede realizar una restauración geográfica en Gen2. Para ello, especifique Gen2 ServiceObjectiveName (p. ej., DW1000**c**) como parámetro opcional.
>

1. Antes de empezar, asegúrese de [instalar Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Abra PowerShell.
3. Conéctese a su cuenta de Azure y enumere todas las suscripciones asociadas a su cuenta.
4. Seleccione la suscripción que contiene el almacenamiento de datos que se va a restaurar.
5. Obtenga el almacenamiento de datos que quiere recuperar.
6. Cree la solicitud de recuperación para el almacenamiento de datos.
7. Compruebe el estado del almacenamiento de datos restaurado geográficamente.
8. Para configurar la base de datos una vez finalizada la restauración, consulte [Configuración de la base de datos después de realizar la recuperación]( ../../sql-database/sql-database-disaster-recovery.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

```Powershell
$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$TargetResourceGroupName="<YourTargetResourceGroupName>" # Restore to a different server.
$TargetServerName="<YourtargetServerNameWithoutURLSuffixSeeNote>"  
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"
$TargetServiceObjective="<YourTargetServiceObjective-DWXXXc>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName
Get-AzureSqlDatabase -ServerName $ServerName

# Get the data warehouse you want to recover
$GeoBackup = Get-AzSqlDatabaseGeoBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Recover data warehouse
$GeoRestoredDatabase = Restore-AzSqlDatabase –FromGeoBackup -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName $TargetServiceObjective

# Verify that the geo-restored data warehouse is online
$GeoRestoredDatabase.status
```

La base de datos recuperada estará habilitada para TDE si la base de datos de origen está habilitada para TDE.

## <a name="restore-from-an-azure-geographical-region-through-azure-portal"></a>Uso de Azure Portal para restaurar desde una región geográfica de Azure

Siga los pasos que se describen a continuación para restaurar un grupo de SQL desde una copia de seguridad geográfica:

1. Inicie sesión en su cuenta de [Azure Portal](https://portal.azure.com/).
2. Haga clic en **+ Crear un recurso**.

   ![Nuevo DW](./media/sql-data-warehouse-restore-from-geo-backup/georestore-new.png)

3. Haga clic en **Bases de datos** y, después, **Azure Synapse Analytics (anteriormente SQL DW)** .

   ![Nuevo DW 2](./media/sql-data-warehouse-restore-from-geo-backup/georestore-new-02.png)

4. Rellene la información solicitada en la pestaña **Aspectos básicos** y haga clic en **Siguiente: Configuración adicional**.

   ![Aspectos básicos](./media/sql-data-warehouse-restore-from-geo-backup/georestore-dw-1.png)

5. En el parámetro **Usar datos existentes**, haga clic en **Copia de seguridad** y luego seleccione la copia de seguridad adecuada en las opciones de desplazamiento. Haga clic en **Revisar y crear**.

   ![copia de seguridad](./media/sql-data-warehouse-restore-from-geo-backup/georestore-select.png)

6. Una vez que se haya restaurado el almacenamiento de datos, compruebe que el estado es **En línea**.

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de un grupo de SQL existente](sql-data-warehouse-restore-active-paused-dw.md)
- [Restauración de un grupo de SQL eliminado](sql-data-warehouse-restore-deleted-dw.md)
