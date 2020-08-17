---
title: Restauración de un almacenamiento de datos existente
description: Guía para la restauración de un grupo de SQL eliminado.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 08/29/2018
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: f8dc7be63506e586b90d1e389d26ac591dce9c3b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075886"
---
# <a name="restore-an-existing-sql-pool"></a>Restauración de un grupo de SQL existente

En este artículo, aprenderá a restaurar un grupo de SQL existente en Azure Synapse Analytics mediante Azure Portal y PowerShell.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**Compruebe la capacidad DTU**. Cada grupo está hospedado en un [servidor SQL lógico](../../azure-sql/database/logical-servers.md) (por ejemplo, myserver.database.windows.net) que tiene una cuota de DTU predeterminada. Compruebe que el servidor tiene suficiente cuota de DTU restante para la base de datos que se va a restaurar. Para más información sobre cómo calcular la unidad DTU necesaria o solicitar más DTU, consulte cómo [solicitar un cambio en la cuota de DTU](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="before-you-begin"></a>Antes de empezar

1. Asegúrese de [instalar Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Tenga un punto de restauración existente desde el que desee realizar la restauración. Si desea crear una restauración, consulte [el tutorial para crear un de restauración definido por el usuario](sql-data-warehouse-restore-points.md).

## <a name="restore-an-existing-sql-pool-through-powershell"></a>Restauración de un grupo de SQL existente mediante PowerShell

Para restaurar un grupo de SQL existente a partir de un punto de restauración, use el cmdlet [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) de PowerShell.

1. Abra PowerShell.

2. Conéctese a su cuenta de Azure y enumere todas las suscripciones asociadas a su cuenta.

3. Seleccione la suscripción que contiene la base de datos que se va a restaurar.

4. Enumere los puntos de restauración del grupo de SQL.

5. Elija el punto de restauración deseado mediante RestorePointCreationDate.

6. Restaure el grupo de SQL al punto de restauración deseado mediante el comando [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) de PowerShell.

    1. Para restaurar el grupo de SQL en un servidor diferente, asegúrese de especificar el nombre del otro servidor.  Este servidor también puede estar en un grupo de recursos y una región diferentes.
    2. Para realizar la restauración en otra suscripción, use el botón Mover para mover el servidor a otra suscripción.

7. Compruebe que el grupo de SQL restaurado está en línea.

8. Una vez finalizada la restauración, puede configurar el grupo de SQL recuperado siguiendo la guía [Configuración de la base de datos después de realizar la recuperación](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

```Powershell

$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
#$TargetResourceGroupName="<YourTargetResourceGroupName>" # uncomment to restore to a different server.
#$TargetServerName="<YourtargetServerNameWithoutURLSuffixSeeNote>"  
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName

# Or list all restore points
Get-AzSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Get the specific database to restore
$Database = Get-AzSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Pick desired restore point using RestorePointCreationDate "xx/xx/xxxx xx:xx:xx xx"
$PointInTime="<RestorePointCreationDate>"  

# Restore database from a restore point
$RestoredDatabase = Restore-AzSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $Database.ResourceID

# Use the following command to restore to a different server
#$RestoredDatabase = Restore-AzSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceTargetGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $Database.ResourceID

# Verify the status of restored database
$RestoredDatabase.status

```

## <a name="restore-an-existing-sql-pool-through-the-azure-portal"></a>Restauración de un grupo de SQL existente mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya al grupo de SQL desde el que desea restaurar.
3. En la parte superior de la hoja de información general, seleccione **Restaurar**.

    ![ Introducción a la restauración](./media/sql-data-warehouse-restore-active-paused-dw/restoring-01.png)

4. Seleccione **Puntos de restauración automáticos** o **Puntos de restauración definidos por el usuario**. Si el grupo de SQL no tiene ningún punto de restauración automática, espere unas horas o cree un punto de restauración definido por el usuario antes de llevar a cabo la restauración. En el caso de los puntos de restauración definidos por el usuario, seleccione uno existente o cree uno nuevo. En **Servidor**, puede elegir un servidor de otro grupo de recursos y región, o bien crear uno nuevo. Después de proporcionar todos los parámetros, haga clic en **Revisar y restaurar**.

    ![Puntos de restauración automáticos](./media/sql-data-warehouse-restore-active-paused-dw/restoring-11.png)

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de un grupo de SQL eliminado](sql-data-warehouse-restore-deleted-dw.md)
- [Restauración de un grupo de SQL a partir de una copia de seguridad de replicación geográfica](sql-data-warehouse-restore-from-geo-backup.md)
