---
title: Restauración de un grupo de SQL eliminado
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
ms.openlocfilehash: fc21dc67dd6c686eedf307660a8d8b7afd60db71
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075846"
---
# <a name="restore-a-deleted-sql-pool-using-azure-synapse-analytics"></a>Restauración de un grupo de SQL eliminado mediante Azure Synapse Analytics

En este artículo, aprenderá a restaurar un grupo de SQL mediante Azure Portal o PowerShell.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**Compruebe la capacidad DTU**. Cada grupo de SQL está hospedado en un [servidor SQL lógico](../../azure-sql/database/logical-servers.md) (por ejemplo, myserver.database.windows.net) que tiene una cuota de DTU predeterminada.  Compruebe que el servidor tiene suficiente cuota de DTU restante para la base de datos en proceso de restauración. Para más información sobre cómo calcular la unidad DTU necesaria o solicitar más DTU, consulte cómo [solicitar un cambio en la cuota de DTU](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="restore-a-deleted-data-warehouse-through-powershell"></a>Restauración de una base de datos de almacenamiento de datos eliminada mediante PowerShell

Para restaurar una grupo de SQL eliminado, use el cmdlet [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Si también se ha eliminado el servidor correspondiente, la base de datos de almacenamiento de datos no se podrá restaurar.

1. Antes de empezar, asegúrese de [instalar Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Abra PowerShell.
3. Conéctese a su cuenta de Azure y enumere todas las suscripciones asociadas a su cuenta.
4. Seleccione la suscripción que contiene el grupo de SQL eliminado que se va a restaurar.
5. Obtenga la base de datos de almacenamiento de datos eliminada específica.
6. Restauración del grupo de SQL eliminado
    1. Para restaurar el grupo de SQL eliminado en un servidor diferente, asegúrese de especificar el nombre del otro servidor.  Este servidor también puede estar en un grupo de recursos y una región diferentes.
    1. Para realizar la restauración en otra suscripción, use el botón [Mover](../../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#use-the-portal) para mover el servidor a otra suscripción.
7. Compruebe que la base de datos de almacenamiento de datos restaurada está en línea.
8. Una vez finalizada la restauración, puede configurar la base de datos de almacenamiento de datos recuperada siguiendo la explicación que se proporciona en el apartado [Configuración de la base de datos después de realizar la recuperación](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

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

# Get the deleted database to restore
$DeletedDatabase = Get-AzSqlDeletedDatabaseBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Restore deleted database
$RestoredDatabase = Restore-AzSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Use the following command to restore deleted data warehouse to a different server
#$RestoredDatabase = Restore-AzSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Verify the status of restored database
$RestoredDatabase.status
```

## <a name="restore-a-deleted-database-using-the-azure-portal"></a>Uso de Azure Portal para restaurar una base de datos eliminada

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya al servidor en el que estaba alojada la base de datos de almacenamiento de datos eliminada.
3. Seleccione el icono **Bases de datos eliminadas** en la tabla de contenido.

    ![Bases de datos eliminadas](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-01.png)

4. Seleccione la instancia de SQL Data Warehouse eliminada que desea restaurar.

    ![Seleccionar bases de datos eliminadas](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-11.png)

5. Especifique un nuevo **nombre de base de datos** y haga clic en **Aceptar**

    ![Especificar nombre de base de datos](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-21.png)

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de un grupo de SQL existente](sql-data-warehouse-restore-active-paused-dw.md)
- [Restauración de un grupo de SQL a partir de una copia de seguridad de replicación geográfica](sql-data-warehouse-restore-from-geo-backup.md)
