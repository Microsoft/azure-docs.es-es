---
title: Eliminación temporal de Azure Backup
description: Aprenda a usar las características de seguridad de Azure Backup para que las copias de seguridad sean más seguras.
ms.topic: conceptual
ms.date: 04/30/2020
ms.openlocfilehash: 76928b98bdb21ffa79fce8435bfe4dda92e0c72d
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89179970"
---
# <a name="soft-delete-for-azure-backup"></a>Eliminación temporal de Azure Backup

Cada vez es mayor la preocupación que generan problemas de seguridad como malware, ransomware e intrusión. Estos problemas de seguridad pueden ser costosos, en términos de dinero y datos. Para protegerse contra dichos ataques, Azure Backup proporciona características de seguridad que protegen los datos de las copias de seguridad incluso después de su eliminación.

Una de estas características es la eliminación temporal. Con la eliminación temporal, aunque un actor malintencionado elimine una copia de seguridad (o se eliminen por accidente datos de copia de seguridad), los datos de copia de seguridad se conservan durante 14 días adicionales, lo que permite la recuperación de ese elemento de copia de seguridad sin pérdida de datos. Esta retención adicional de 14 días de los datos de copia de seguridad en el estado "eliminación temporal" no acarrea ningún costo para el cliente.

La protección de eliminación temporal está disponible para estos servicios:

- [Eliminación temporal de máquinas virtuales de Azure](soft-delete-virtual-machines.md)
- [Eliminación temporal de servidores SQL Server en máquinas virtuales de Azure y de instancias de SAP HANA en cargas de trabajo de máquinas virtuales de Azure](soft-delete-sql-saphana-in-azure-vm.md)

En este diagrama de flujo se explican los diferentes pasos y estados de un elemento de copia de seguridad cuando se habilita la eliminación temporal:

![Ciclo de vida del elemento de copia de seguridad eliminado temporalmente](./media/backup-azure-security-feature-cloud/lifecycle.png)

## <a name="enabling-and-disabling-soft-delete"></a>Habilitación y deshabilitación de la eliminación temporal

La eliminación temporal se habilita de forma predeterminada en los almacenes recién creados para proteger los datos de copia de seguridad de eliminaciones accidentales o malintencionadas.  No se recomienda deshabilitar esta característica. La única circunstancia en la que debe considerar la posibilidad de deshabilitar la eliminación temporal es si está planeando mover los elementos protegidos a un nuevo almacén y no puede esperar los 14 días necesarios para realizar la acción de eliminar y volver a proteger (por ejemplo, en un entorno de prueba). Solo el propietario del almacén puede deshabilitar esta característica. Si se deshabilita, todas las eliminaciones posteriores de elementos protegidos se convertirán en eliminaciones inmediatas, sin la posibilidad de restaurar. Los datos de copia de seguridad que se encuentren en estado de eliminación temporal antes de deshabilitar esta característica permanecerán en ese estado durante el período de 14 días. Si quiere eliminarlos permanentemente de inmediato, debe recuperarlos y eliminarlos de nuevo para eliminarlos de forma permanente.

 Es importante recordar que una vez deshabilitada la eliminación temporal, la característica se deshabilita para todos los tipos de cargas de trabajo, incluidas las cargas de trabajo de SQL Server y SAP HANA. Por ejemplo, una vez que la [versión preliminar de SQL Server / SAP HANA](./soft-delete-sql-saphana-in-azure-vm.md#steps-to-enroll-in-preview) esté habilitada en una suscripción, no es posible deshabilitar la eliminación temporal solo en servidores SQL Server o bases de SAP HANA si está habilitada al mismo tiempo en máquinas virtuales del mismo almacén. Puede crear almacenes independientes para llevar a cabo un control granular.

### <a name="disabling-soft-delete-using-azure-portal"></a>Deshabilitación de la eliminación temporal con Azure Portal

Para deshabilitar la eliminación temporal, siga estos pasos:

1. En Azure Portal, vaya a su almacén y luego a **Configuración** -> **Propiedades**.
2. En el panel Propiedades, seleccione **Configuración de seguridad** -> **Actualizar**.  
3. En el panel Configuración de seguridad, en **Eliminación temporal**, seleccione **Deshabilitar**.

![Deshabilitación de la eliminación temporal](./media/backup-azure-security-feature-cloud/disable-soft-delete.png)

### <a name="disabling-soft-delete-using-azure-powershell"></a>Deshabilitación de la eliminación temporal con Azure PowerShell

> [!IMPORTANT]
> La versión de Az.RecoveryServices necesaria para usar la eliminación temporal con Azure PowerShell es, como mínimo, la 2.2.0. Use ```Install-Module -Name Az.RecoveryServices -Force``` para obtener la versión más reciente.

Para deshabilitar, use el cmdlet de PowerShell [Set-AzRecoveryServicesVaultBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty).

```powershell
Set-AzRecoveryServicesVaultProperty -VaultId $myVaultID -SoftDeleteFeatureState Disable


StorageModelType       :
StorageType            :
StorageTypeState       :
EnhancedSecurityState  : Enabled
SoftDeleteFeatureState : Disabled
```

### <a name="disabling-soft-delete-using-rest-api"></a>Deshabilitación de la eliminación temporal con API REST

Para deshabilitar la funcionalidad de eliminación temporal mediante la API REST, consulte los pasos mencionados [aquí](use-restapi-update-vault-properties.md#update-soft-delete-state-using-rest-api).

## <a name="permanently-deleting-soft-deleted-backup-items"></a>Eliminar permanentemente los elementos de copia de seguridad eliminados temporalmente

Los datos de copia de seguridad con el estado de eliminación temporal antes de deshabilitar esta característica continuarán en ese estado. Si quiere eliminarlos de permanentemente de inmediato, restáurelos y vuelva a eliminarlos para eliminarlos de forma permanente.

### <a name="using-azure-portal"></a>Uso de Azure Portal

Siga estos pasos:

1. Siga los pasos para [deshabilitar la eliminación temporal](#enabling-and-disabling-soft-delete).

2. En Azure Portal, vaya al almacén, a **Elementos de copia de seguridad** y elija el elemento eliminado temporalmente.

   ![Selección de un elemento eliminado temporalmente](./media/backup-azure-security-feature-cloud/vm-soft-delete.png)

3. Seleccione la opción **Recuperar**.

   ![Elegir Recuperar](./media/backup-azure-security-feature-cloud/choose-undelete.png)

4. Se mostrará una ventana. Seleccione **Recuperar**.

   ![Seleccionar Recuperar](./media/backup-azure-security-feature-cloud/undelete-vm.png)

5. Elija **Eliminar datos de la copia de seguridad** para eliminar los datos de copia de seguridad de forma permanente.

   ![Elegir Eliminar datos de la copia de seguridad](/azure/backup/media/backup-azure-manage-vms/delete-backup-buttom.png)

6. Escriba el nombre del elemento de copia de seguridad para confirmar que desea eliminar los puntos de recuperación.

   ![Escritura del nombre del elemento de copia de seguridad](/azure/backup/media/backup-azure-manage-vms/delete-backup-data1.png)

7. Para eliminar los datos de copia de seguridad para el elemento, seleccione **Eliminar**. Un mensaje de notificación le confirma que se han eliminado los datos de copia de seguridad.

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

Si los elementos se eliminaron antes de deshabilitar la eliminación temporal, estarán en un estado de eliminación temporal. Para eliminarlos de inmediato, la operación de eliminación debe invertirse y volver a ejecutarse.

Identifique los elementos que se encuentran en estado de eliminación temporal.

```powershell

Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

Name                                     ContainerType        ContainerUniqueName                      WorkloadType         ProtectionStatus     HealthStatus         DeleteState
----                                     -------------        -------------------                      ------------         ----------------     ------------         -----------
VM;iaasvmcontainerv2;selfhostrg;AppVM1    AzureVM             iaasvmcontainerv2;selfhostrg;AppVM1       AzureVM              Healthy              Passed               ToBeDeleted

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID -Name AppVM1
```

A continuación, invierta la operación de eliminación que se realizó cuando la eliminación temporal estaba habilitada.

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           Undelete             Completed            12/5/2019 12:47:28 PM     12/5/2019 12:47:40 PM     65311982-3755-46b5-8e53-c82ea4f0d2a2
```

Dado que la eliminación temporal ahora está deshabilitada, la operación de eliminación producirá la eliminación inmediata de los datos de la copia de seguridad.

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           DeleteBackupData     Completed            12/5/2019 12:44:15 PM     12/5/2019 12:44:50 PM     0488c3c2-accc-4a91-a1e0-fba09a67d2fb
```

### <a name="using-rest-api"></a>Uso de la API de REST

Si los elementos se eliminaron antes de deshabilitar la eliminación temporal, estarán en un estado de eliminación temporal. Para eliminarlos de inmediato, la operación de eliminación debe invertirse y volver a ejecutarse.

1. En primer lugar, deshaga las operaciones de eliminación con los pasos mencionados [aquí](backup-azure-arm-userestapi-backupazurevms.md#undo-the-deletion).
2. Después, deshabilite la funcionalidad de eliminación temporal mediante la API REST siguiendo los pasos mencionados [aquí](use-restapi-update-vault-properties.md#update-soft-delete-state-using-rest-api).
3. Después, elimine las copias de seguridad mediante la API REST, como se mencionó [aquí](backup-azure-arm-userestapi-backupazurevms.md#stop-protection-and-delete-data).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="do-i-need-to-enable-the-soft-delete-feature-on-every-vault"></a>¿Es necesario habilitar la característica de eliminación temporal en cada almacén?

No, está integrada y se habilita de manera predeterminada para todos los almacenes de Recovery Services.

### <a name="can-i-configure-the-number-of-days-for-which-my-data-will-be-retained-in-soft-deleted-state-after-the-delete-operation-is-complete"></a>¿Se puede configurar el número de días durante los que se conservarán los datos en estado de eliminación temporal tras completar la operación de eliminación?

No, este periodo está fijado en 14 días de retención adicional después de la operación de eliminación.

### <a name="do-i-need-to-pay-the-cost-for-this-additional-14-day-retention"></a>¿Hay que pagar el costo de esta retención adicional de 14 días?

No, esta retención adicional de 14 días es gratuita como parte de la funcionalidad de eliminación temporal.

### <a name="can-i-perform-a-restore-operation-when-my-data-is-in-soft-delete-state"></a>¿Puedo realizar una operación de restauración si los datos están en estado de eliminación temporal?

No, debe recuperar el recurso eliminado temporalmente para poder restaurarlo. La operación de recuperación devolverá el recurso al estado **Detener la protección con conservación de datos**, donde puede realizar la restauración a cualquier momento dado. El recolector de elementos no utilizados permanece en pausa en este estado.

### <a name="will-my-snapshots-follow-the-same-lifecycle-as-my-recovery-points-in-the-vault"></a>¿Las instantáneas seguirán el mismo ciclo de vida que los puntos de recuperación en el almacén?

Sí.

### <a name="how-can-i-trigger-the-scheduled-backups-again-for-a-soft-deleted-resource"></a>¿Cómo puedo volver a desencadenar las copias de seguridad programadas para un recurso eliminado temporalmente?

La recuperación seguida de una operación de reanudación volverá a proteger el recurso. La operación de reanudación asocia una directiva de copia de seguridad para desencadenar las copias de seguridad programadas con el período de retención seleccionado. Además, el recolector de elementos no utilizados se ejecuta tan pronto como se completa la operación de reanudación. Si desea realizar una restauración desde un punto de recuperación que supere su fecha de expiración, le recomendamos que lo haga antes de desencadenar la operación de reanudación.

### <a name="can-i-delete-my-vault-if-there-are-soft-deleted-items-in-the-vault"></a>¿Puedo eliminar mi almacén si contiene elementos eliminados temporalmente?

No es posible eliminar el almacén de Recovery Services si contiene elementos de copia de seguridad en estado de eliminación temporal. Los elementos eliminados temporalmente se eliminan de forma permanente 14 días después de la operación de eliminación. Si no puede esperar 14 días, [deshabilite la eliminación temporal](#enabling-and-disabling-soft-delete), recupere los elementos eliminados temporalmente y elimínelos de nuevo para eliminarlos de forma permanente. Después de asegurarse de que no hay elementos protegidos o eliminados temporalmente en el almacén, puede eliminarlo.  

### <a name="can-i-delete-the-data-earlier-than-the-14-days-soft-delete-period-after-deletion"></a>¿Puedo eliminar los datos antes del período de eliminación temporal de 14 días posterior a la eliminación?

No. No se puede forzar la eliminación de los elementos eliminados temporalmente. Se eliminarán de manera automática después de 14 días. Esta característica de seguridad está habilitada para proteger los datos de copia de seguridad de eliminaciones accidentales o malintencionadas.  Debe esperar 14 días antes de realizar cualquier otra acción en el elemento.  Los elementos eliminados temporalmente no se cobrarán.  Si tiene que volver a proteger los elementos marcados para eliminación temporal en un plazo de 14 días en un almacén nuevo, póngase en contacto con Soporte técnico de Microsoft.

### <a name="can-soft-delete-operations-be-performed-in-powershell-or-cli"></a>¿Se pueden realizar operaciones de eliminación temporal en PowerShell o la CLI?

Las operaciones de eliminación temporal se pueden realizar mediante PowerShell. Actualmente, no se admite la CLI.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las características de seguridad de Azure Backup](security-overview.md)
