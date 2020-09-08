---
title: Eliminación temporal de máquinas virtuales
description: Aprenda sobre la eliminación temporal de máquinas virtuales aumenta la seguridad de las copias de seguridad.
ms.topic: conceptual
ms.date: 04/30/2020
ms.custom: references_regions
ms.openlocfilehash: a8b70d4c8240d096c19e5a8d7449921557b8896c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022248"
---
# <a name="soft-delete-for-virtual-machines"></a>Eliminación temporal de máquinas virtuales

La eliminación temporal de máquinas virtuales protege las copias de seguridad de las máquinas virtuales de una eliminación imprevista. Incluso después de que se eliminen las copias de seguridad, se conservan en estado de eliminación temporal durante 14 días adicionales.

> [!NOTE]
> La eliminación temporal solo protege los datos de copia de seguridad eliminados. Si se elimina una máquina virtual sin una copia de seguridad, la característica de eliminación temporal no conservará los datos. Todos los recursos deben protegerse con Azure Backup para garantizar una resistencia total.
>

## <a name="supported-regions"></a>Regiones admitidas

La eliminación temporal se admite actualmente en Centro-oeste de EE. UU., Este de Asia, Centro de Canadá, Este de Canadá, Centro de Francia, Sur de Francia, Centro de Corea del Sur, Sur de Corea del Sur, Sur de Reino Unido, Oeste de Reino Unido, Este de Australia, Sudeste de Australia, Norte de Europa, Oeste de EE. UU., Oeste de EE. UU. 2, Centro de EE. UU., Sudeste de Asia, Centro-norte de EE. UU., Centro-sur de EE. UU., Este de Japón, Oeste de Japón, Sur de India, Centro de la India, Oeste de la India, Este de EE. UU. 2, Norte de Suiza, Oeste de Suiza, Oeste de Noruega, Este de Noruega y todas las regiones nacionales.

## <a name="soft-delete-for-vms-using-azure-portal"></a>Eliminación temporal para máquinas virtuales con Azure Portal

1. Para eliminar los datos de copia de seguridad de una máquina virtual, se debe detener la copia de seguridad. En Azure Portal, vaya al almacén de Recovery Services, haga clic con el botón derecho en el elemento de copia de seguridad y seleccione **Detener copia de seguridad**.

   ![Captura de pantalla de elementos de copia de seguridad de Azure Portal](./media/backup-azure-security-feature-cloud/backup-stopped.png)

2. En la ventana siguiente, se le ofrecerá la opción de eliminar o conservar los datos de copia de seguridad. Si elige la opción **Eliminar datos de copia de seguridad** y, a continuación, **Detener copia de seguridad**, la copia de seguridad de la máquina virtual no se eliminará de forma permanente. En su lugar, los datos de copia de seguridad se conservarán durante 14 días en el estado de eliminación temporal. Si selecciona **Eliminar datos de copia de seguridad**, se enviará una alerta de eliminación por correo electrónico al identificador de correo electrónico configurado que informa al usuario de que quedan 14 días de retención ampliada para los datos de copia de seguridad. Además, a los doce días se envía una alerta por correo electrónico para informar de que quedan otros dos días para recuperar los datos eliminados. La eliminación se aplaza hasta el decimoquinto día, cuando se produce la eliminación permanente y se envía una alerta final por correo electrónico que informa de la eliminación permanente de los datos.

   ![Captura de pantalla de Azure Portal, pantalla Detener copia de seguridad](./media/backup-azure-security-feature-cloud/delete-backup-data.png)

3. Durante esos 14 días, en el almacén de Recovery Services, la máquina virtual eliminada temporalmente aparecerá con un icono rojo de "eliminación temporal" junto a ella.

   ![Captura de pantalla de Azure Portal, máquina virtual en estado de eliminación temporal](./media/backup-azure-security-feature-cloud/vm-soft-delete.png)

   > [!NOTE]
   > Si algún elemento de copia de seguridad eliminado temporalmente está presente en el almacén, no se podrá eliminar el almacén en ese momento. Pruebe a eliminar el almacén después de que se eliminen de forma permanente los elementos de copia de seguridad y no quede ningún elemento en estado de eliminación temporal en el almacén.

4. Para restaurar la máquina virtual eliminada temporalmente, hay que eliminarla primero. Para ello, seleccione la máquina virtual eliminada temporalmente y, a continuación, seleccione la opción **Recuperar**.

   ![Captura de pantalla de Azure Portal, recuperación de máquina virtual](./media/backup-azure-security-feature-cloud/choose-undelete.png)

   Aparecerá una ventana con una advertencia que indica que si se selecciona Recuperar, todos los puntos de restauración de la máquina virtual se recuperarán y estarán disponibles para realizar una operación de restauración. La máquina virtual se conservará en el estado "Detener la protección con conservación de datos" con las copias de seguridad en pausa y los datos de copia de seguridad conservados indefinidamente sin ninguna directiva de copia de seguridad en vigor.

   ![Captura de pantalla de Azure Portal, confirmación de recuperación de máquina virtual](./media/backup-azure-security-feature-cloud/undelete-vm.png)

   En este punto, también puede restaurar la máquina virtual seleccionando **Restaurar VM** desde el punto de restauración elegido.  

   ![Captura de pantalla de Azure Portal, opción Restaurar VM](./media/backup-azure-security-feature-cloud/restore-vm.png)

   > [!NOTE]
   > El recolector de elementos no utilizados ejecutará y limpiará los puntos de recuperación expirados solo después de que el usuario realice la operación de **reanudar copia de seguridad**.

5. Una vez completado el proceso de recuperación, el estado volverá a "Detener copia de seguridad con conservación de datos" y, a continuación, podrá seleccionar **Reanudar copia de seguridad**. La operación **Reanudar copia de seguridad** devuelve el elemento de copia de seguridad al estado activo, asociado a una directiva de copia de seguridad seleccionada por el usuario que define las programaciones de copia de seguridad y retención.

   ![Captura de pantalla de Azure Portal, opción Reanudar copia de seguridad](./media/backup-azure-security-feature-cloud/resume-backup.png)

## <a name="soft-delete-for-vms-using-azure-powershell"></a>Eliminación temporal para máquinas virtuales con Azure PowerShell

> [!IMPORTANT]
> La versión de Az.RecoveryServices necesaria para usar la eliminación temporal con Azure PowerShell es, como mínimo, la 2.2.0. Use ```Install-Module -Name Az.RecoveryServices -Force``` para obtener la versión más reciente.

Tal y como se ha descrito anteriormente para Azure Portal, la secuencia de pasos es la misma cuando también se usa Azure PowerShell.

### <a name="delete-the-backup-item-using-azure-powershell"></a>Eliminación del elemento de copia de seguridad con Azure PowerShell

Puede eliminar el elemento de copia de seguridad mediante el cmdlet de PowerShell [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           DeleteBackupData     Completed            12/5/2019 12:44:15 PM     12/5/2019 12:44:50 PM     0488c3c2-accc-4a91-a1e0-fba09a67d2fb
```

El elemento "DeleteState" del elemento de copia de seguridad cambiará de "NotDeleted" a "ToBeDeleted". Los datos de la copia de seguridad se conservarán durante 14 días. Si desea revertir la operación de eliminación, se debe realizar la operación undo-delete.

### <a name="undoing-the-deletion-operation-using-azure-powershell"></a>Acción de deshacer la operación de eliminación mediante Azure PowerShell

En primer lugar, capture el elemento de copia de seguridad relevante que se encuentre en estado de eliminación temporal (es decir, a punto de eliminarse).

```powershell

Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

Name                                     ContainerType        ContainerUniqueName                      WorkloadType         ProtectionStatus     HealthStatus         DeleteState
----                                     -------------        -------------------                      ------------         ----------------     ------------         -----------
VM;iaasvmcontainerv2;selfhostrg;AppVM1    AzureVM             iaasvmcontainerv2;selfhostrg;AppVM1       AzureVM              Healthy              Passed               ToBeDeleted

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID -Name AppVM1
```

A continuación, realice la operación para deshacer la eliminación con el cmdlet de PowerShell [Undo-AzRecoveryServicesBackupItemDeletion](/powershell/module/az.recoveryservices/undo-azrecoveryservicesbackupitemdeletion).

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           Undelete             Completed            12/5/2019 12:47:28 PM     12/5/2019 12:47:40 PM     65311982-3755-46b5-8e53-c82ea4f0d2a2
```

El elemento "DeleteState" del elemento de copia de seguridad se revertirá a "NotDeleted". Pero la protección sigue detenida. [Reanude la copia de seguridad](./backup-azure-vms-automation.md#change-policy-for-backup-items) para volver a habilitar la protección.

## <a name="soft-delete-for-vms-using-rest-api"></a>Eliminación temporal para máquinas virtuales con API REST

- Elimine las copias de seguridad mediante la API REST, como se mencionó [aquí](backup-azure-arm-userestapi-backupazurevms.md#stop-protection-and-delete-data).
- Si quiere deshacer estas operaciones de eliminación, consulte los pasos mencionados [aquí](backup-azure-arm-userestapi-backupazurevms.md#undo-the-deletion).

## <a name="how-to-disable-soft-delete"></a>Deshabilitación de la eliminación temporal

No se recomienda deshabilitar esta característica. La única circunstancia en la que debe considerar la posibilidad de deshabilitar la eliminación temporal es si está planeando mover los elementos protegidos a un nuevo almacén y no puede esperar los 14 días necesarios para realizar la acción de eliminar y volver a proteger (por ejemplo, en un entorno de prueba). Puede encontrar instrucciones sobre cómo deshabilitar la eliminación temporal en [Habilitación y deshabilitación de la eliminación temporal](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete).

## <a name="next-steps"></a>Pasos siguientes

- Lea las [preguntas frecuentes](backup-azure-security-feature-cloud.md#frequently-asked-questions) sobre la eliminación temporal
- Lea todas las [características de seguridad de Azure Backup](security-overview.md).
