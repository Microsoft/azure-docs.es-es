---
title: Puntos de restauración definidos por el usuario
description: Cómo crear un punto de restauración para el grupo de SQL.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 07/03/2019
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 66a2dad9396e8bf7c8ef49db529f7a5486cc8a8f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089214"
---
# <a name="user-defined-restore-points"></a>Puntos de restauración definidos por el usuario

En este artículo, aprenderá a crear un punto de restauración definido por el usuario para un grupo de SQL en Azure Synapse Analytics mediante PowerShell y Azure Portal.

## <a name="create-user-defined-restore-points-through-powershell"></a>Creación de puntos de restauración definidos por el usuario mediante PowerShell

Para crear un punto de restauración definido por el usuario, utilice el cmdlet de PowerShell [New-AzSqlDatabaseRestorePoint](/powershell/module/az.sql/new-azsqldatabaserestorepoint?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

1. Antes de empezar, asegúrese de [instalar Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Abra PowerShell.
3. Conéctese a su cuenta de Azure y enumere todas las suscripciones asociadas a su cuenta.
4. Seleccione la suscripción que contiene la base de datos que se va a restaurar.
5. Cree un punto de restauración para realizar una copia inmediata de la base de datos de almacenamiento de datos.

```Powershell

$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$DatabaseName="<YourDatabaseName>"
$Label = "<YourRestorePointLabel>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName

# Create a restore point of the original database
New-AzSqlDatabaseRestorePoint -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -RestorePointLabel $Label

```

6. Consulte la lista de todos los puntos de restauración existentes.

```Powershell
# List all restore points
Get-AzSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName
```

## <a name="create-user-defined-restore-points-through-the-azure-portal"></a>Creación de puntos de restauración definidos por el usuario mediante Azure Portal

Los puntos de restauración definidos por el usuario también se pueden crear mediante Azure Portal.

1. Inicie sesión en su cuenta de [Azure Portal](https://portal.azure.com/).

2. Vaya al grupo de SQL para el que desea crear un punto de restauración.

3. Seleccione **Información general** en el panel izquierdo y seleccione **Nuevo punto de restauración**. Si el botón Nuevo punto de restauración no está habilitado, asegúrese de que el grupo de SQL no está en pausa.

    ![Nuevo punto de restauración](./media/sql-data-warehouse-restore-points/creating-restore-point-01.png)

4. Especifique el nombre del punto de restauración definido por el usuario y haga clic en **Aplicar**. Los puntos de restauración definidos por el usuario tienen un período de retención predeterminado de siete días.

    ![Nombre de punto de restauración](./media/sql-data-warehouse-restore-points/creating-restore-point-11.png)

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de un grupo de SQL existente](sql-data-warehouse-restore-active-paused-dw.md)
- [Restauración de un grupo de SQL eliminado](sql-data-warehouse-restore-deleted-dw.md)
- [Restauración de un grupo de SQL a partir de una copia de seguridad de replicación geográfica](sql-data-warehouse-restore-from-geo-backup.md)

