---
title: Eliminación temporal de servidores SQL Server en máquinas virtuales de Azure y de instancias de SAP HANA en cargas de trabajo de máquinas virtuales de Azure
description: Obtenga información sobre cómo la eliminación temporal de servidores SQL Server en máquinas virtuales de Azure y de instancias de SAP HANA en cargas de trabajo de máquinas virtuales de Azure consigue que las copias de seguridad sean más seguras.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 26525ec758b3a27d6e0e1b9754b11041bd1fa0d2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022299"
---
# <a name="soft-delete-for-sql-server-in-azure-vm-and-sap-hana-in-azure-vm-workloads"></a>Eliminación temporal de servidores SQL Server en máquinas virtuales de Azure y de instancias de SAP HANA en cargas de trabajo de máquinas virtuales de Azure

Azure Backup ahora permite eliminar temporalmente servidores SQL Server en máquinas virtuales de Azure e instancias de SAP HANA en cargas de trabajo de máquinas virtuales de Azure. Esto se suma al [escenario de eliminación temporal de máquinas virtuales de Azure](soft-delete-virtual-machines.md) que ya se admite.

La [eliminación temporal](backup-azure-security-feature-cloud.md) es una característica de seguridad que ayuda a proteger los datos de copia de seguridad incluso después de la eliminación. Con la eliminación temporal, aunque un actor malintencionado elimine la copia de seguridad de una base de datos (o se eliminen accidentalmente los datos de copia de seguridad), dichos datos se conservan durante 14 días adicionales. De este modo, se puede recuperar ese elemento de copia de seguridad sin pérdida de datos. Esta retención adicional de 14 días de los datos de copia de seguridad en el estado de "eliminación temporal" no supone ningún costo para el cliente.

>[!NOTE]
>Una vez habilitada la versión preliminar en una suscripción, no es posible deshabilitar la eliminación temporal solo en servidores SQL Server o bases de SAP HANA si está habilitada al mismo tiempo en máquinas virtuales del mismo almacén. Puede crear almacenes independientes para llevar a cabo un control granular.

## <a name="steps-to-enroll-in-preview"></a>Pasos para inscribirse en la versión preliminar

1. Inicie sesión en la cuenta de Azure.

   ```powershell
   Login-AzureRmAccount
   ```

2. Seleccione la suscripción que desea inscribir en la versión preliminar:

   ```powershell
   Get-AzureRmSubscription –SubscriptionName "Subscription Name" | Select-AzureRmSubscription
   ```

3. Registre esta suscripción en el programa de la versión preliminar:

   ```powershell
   Register-AzureRMProviderFeature -FeatureName WorkloadBackupSoftDelete -ProviderNamespace Microsoft.RecoveryServices
   ```

4. Espere 30 minutos a que la suscripción se inscriba en la versión preliminar.

5. Para comprobar el estado, ejecute los siguientes cmdlets:

   ```powershell
   Get-AzureRmProviderFeature -FeatureName WorkloadBackupSoftDelete -ProviderNamespace Microsoft.RecoveryServices
   ```

6. Cuando la suscripción se muestre como registrada, ejecute el siguiente comando:

   ```powershell
   Register-AzureRmResourceProvider -ProviderNamespace Microsoft.RecoveryServices
   ```

>[!NOTE]
>Cada vez que se crean almacenes o almacenes en la suscripción habilitada para la eliminación temporal, hay que volver a ejecutar el siguiente comando para habilitar la característica en los almacenes recién creados.<BR>
> `Register-AzureRmResourceProvider -ProviderNamespace Microsoft.RecoveryServices`

## <a name="soft-delete-for-sql-server-in-azure-vm-using-azure-portal"></a>Eliminación temporal de servidores SQL Server en máquinas virtuales de Azure con Azure Portal

>[!NOTE]
>Estas instrucciones también se aplican a las instancias de SAP HANA en máquinas virtuales de Azure.

1. Para eliminar los datos de copia de seguridad de una base de datos de un servidor SQL Server, se debe detener la copia de seguridad. En Azure Portal, vaya al almacén de Recovery Services, haga clic con el botón derecho en el elemento de copia de seguridad y seleccione **Detener copia de seguridad**.

   ![Detener copia de seguridad](./media/soft-delete-sql-saphana-in-azure-vm/stop-backup.png)

2. En la ventana siguiente, se le ofrecerá la opción de eliminar o conservar los datos de copia de seguridad. Si elige **Eliminar datos de copia de seguridad**, la copia de seguridad de la base de datos no se eliminará de forma permanente. En su lugar, los datos de copia de seguridad se conservarán durante 14 días en el estado de eliminación temporal. La eliminación se aplaza hasta el día 15 y se envían mensajes de correo electrónico de alerta normales el primer día del mes, el 12 y el 15 con información sobre el estado de la copia de seguridad de la base de datos para el usuario.

   ![Eliminación de datos de copia de seguridad](./media/soft-delete-sql-saphana-in-azure-vm/delete-backup-data.png)

3. Durante esos 14 días, el elemento eliminado temporalmente aparecerá con un icono rojo de "eliminación temporal" junto a ella en el almacén de Recovery Services.

   ![Elementos eliminados temporalmente](./media/soft-delete-sql-saphana-in-azure-vm/soft-deleted-items.png)

4. Para restaurar la base de datos eliminada temporalmente, hay que eliminarla primero. Para ello, seleccione la base de datos eliminada temporalmente y, a continuación, elija la opción **Recuperar**.

   ![Recuperar base de datos](./media/soft-delete-sql-saphana-in-azure-vm/undelete-database.png)

   Aparecerá una ventana con una advertencia que indica que, si se selecciona Recuperar, todos los puntos de restauración de la base de datos se recuperarán y estarán disponibles para realizar una operación de restauración. La base de datos se conservará en el estado "Detener la protección con conservación de datos" con las copias de seguridad en pausa y los datos de copia de seguridad conservados indefinidamente sin ninguna directiva de copia de seguridad en vigor.

   ![Advertencia de recuperación](./media/soft-delete-sql-saphana-in-azure-vm/undelete-warning.png)

5. En este punto, también puede restaurar los datos seleccionando **Restaurar** en el elemento de copia de seguridad eliminado temporalmente seleccionado.

   ![Restaurar VM](./media/soft-delete-sql-saphana-in-azure-vm/restore-vm.png)

6. Una vez completado el proceso de recuperación, el estado volverá a "Detener copia de seguridad con retención de datos" y, a continuación, podrá seleccionar **Reanudar copia de seguridad**. La operación **Reanudar copia de seguridad** devuelve el elemento de copia de seguridad al estado activo, asociado a una directiva de copia de seguridad seleccionada por el usuario que define las programaciones de copia de seguridad y retención.

   ![Reanudar copia de seguridad](./media/soft-delete-sql-saphana-in-azure-vm/resume-backup.png)

## <a name="soft-delete-for-sql-server-in-vm-using-azure-powershell"></a>Eliminación temporal de servidores SQL Server en una máquina virtual con Azure PowerShell

>[!NOTE]
>La versión de Az.RecoveryServices necesaria para usar la eliminación temporal con Azure PowerShell es, como mínimo, la 2.2.0. Use `Install-Module -Name Az.RecoveryServices -Force` para obtener la versión más reciente.

La secuencia de pasos para utilizar Azure PowerShell es la misma que en Azure Portal, descrita anteriormente.

### <a name="delete-the-backup-item-using-azure-powershell"></a>Eliminación del elemento de copia de seguridad con Azure PowerShell

Puede eliminar el elemento de copia de seguridad mediante el cmdlet de PowerShell [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force
```

El elemento **DeleteState** del elemento de copia de seguridad cambiará de **NotDeleted** a **ToBeDeleted**. Los datos de la copia de seguridad se conservarán durante 14 días. Si desea revertir la operación de eliminación, se debe realizar la operación undo-delete.

### <a name="undoing-the-deletion-operation-using-azure-powershell"></a>Acción de deshacer la operación de eliminación mediante Azure PowerShell

En primer lugar, capture el elemento de copia de seguridad relevante que se encuentre en estado de eliminación temporal (es decir, a punto de eliminarse).

```powershell
Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType SQLDataBase -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType SQLDataBase -VaultId $myVaultID -Name AppVM1
```

A continuación, realice la operación para deshacer la eliminación con el cmdlet de PowerShell [Undo-AzRecoveryServicesBackupItemDeletion](/powershell/module/az.recoveryservices/undo-azrecoveryservicesbackupitemdeletion).

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force
```

El elemento **DeleteState** del elemento de copia de seguridad se revertirá a **NotDeleted**. Pero la protección sigue detenida. Reanude la copia de seguridad para volver a habilitar la protección.

## <a name="how-to-disable-soft-delete"></a>Deshabilitación de la eliminación temporal

No se recomienda deshabilitar esta característica. La única circunstancia en la que debe considerar la posibilidad de deshabilitar la eliminación temporal es si está planeando mover los elementos protegidos a un nuevo almacén y no puede esperar los 14 días necesarios para realizar la acción de eliminar y volver a proteger (por ejemplo, en un entorno de prueba). Para ver las instrucciones sobre cómo deshabilitar la eliminación temporal, consulte el artículo sobre cómo [habilitar y deshabilitar la eliminación temporal](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete).

## <a name="next-steps"></a>Pasos siguientes

- Lea las [preguntas más frecuentes](backup-azure-security-feature-cloud.md#frequently-asked-questions) sobre la eliminación temporal
- Consulte todas las [características de seguridad de Azure Backup](security-overview.md)
