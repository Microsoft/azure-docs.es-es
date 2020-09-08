---
title: Administración de copias de seguridad de recursos compartidos de archivos de Azure con PowerShell
description: Obtenga información sobre cómo usar PowerShell para administrar y supervisar los recursos compartidos de archivos de Azure de los que el servicio Azure Backup ha realizado una copia de seguridad.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: bb400c25ca6b1deebaef6af6fe4b322951dfcdcc
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826759"
---
# <a name="manage-azure-file-share-backups-with-powershell"></a>Administración de copias de seguridad de recursos compartidos de archivos de Azure con PowerShell

En este artículo se describe el uso de Azure PowerShell para administrar y supervisar los recursos compartidos de archivos de Azure que tienen una copia de seguridad del servicio Azure Backup.

> [!WARNING]
> Asegúrese de que la versión de PowerShell se actualice a la versión mínima de "Az.RecoveryServices 2.6.0" para las copias de seguridad de AFS. Para obtener más información, consulte [la sección](backup-azure-afs-automation.md#important-notice-backup-item-identification) que describe el requisito de este cambio.

## <a name="modify-the-protection-policy"></a>Modificación de la directiva de protección

Para cambiar la directiva empleada para la realizar la copia de seguridad del recurso compartido de archivos de Azure, use [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Especifique el elemento de copia de seguridad pertinente y la nueva directiva de copia de seguridad.

En el siguiente ejemplo se cambia la directiva de protección de **testAzureFS** de **dailyafs** a **monthlyafs**.

```powershell
$monthlyafsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "monthlyafs"
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType AzureFiles -Name "testAzureFS"
Enable-AzRecoveryServicesBackupProtection -Item $afsBkpItem -Policy $monthlyafsPol
```

## <a name="track-backup-and-restore-jobs"></a>Seguimiento de trabajos de copia de seguridad y restauración

Las operaciones de copia de seguridad y restauración a petición devuelven un trabajo con un identificador, como se indica al [ejecutar una copia de seguridad a petición](backup-azure-afs-automation.md#trigger-an-on-demand-backup). Use el cmdlet [Get-AzRecoveryServicesBackupJobDetails](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) para hacer un seguimiento del progreso del trabajo y de los detalles.

```powershell
$job = Get-AzRecoveryServicesBackupJob -JobId 00000000-6c46-496e-980a-3740ccb2ad75 -VaultId $vaultID

 $job | fl


IsCancellable        : False
IsRetriable          : False
ErrorDetails         : {Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureFileShareJobErrorInfo}
ActivityId           : 00000000-5b71-4d73-9465-8a4a91f13a36
JobId                : 00000000-6c46-496e-980a-3740ccb2ad75
Operation            : Restore
Status               : Failed
WorkloadName         : testAFS
StartTime            : 12/10/2018 9:56:38 AM
EndTime              : 12/10/2018 11:03:03 AM
Duration             : 01:06:24.4660027
BackupManagementType : AzureStorage

$job.ErrorDetails

 ErrorCode ErrorMessage                                          Recommendations
 --------- ------------                                          ---------------
1073871825 Microsoft Azure Backup encountered an internal error. Wait for a few minutes and then try the operation again. If the issue persists, please contact Microsoft support.
```

## <a name="stop-protection-on-a-file-share"></a>Detención de la protección en un recurso compartido de archivos

Hay dos maneras de dejar de proteger recursos compartidos de archivos de Azure:

* Detener todos los trabajos futuros de copia de seguridad y *eliminar* todos los puntos de recuperación.
* Detener todos los trabajos futuros de copia de seguridad pero *dejar* los puntos de recuperación.

Puede que dejar los puntos de recuperación en el almacenamiento conlleve un costo asociado, dado que las instantáneas subyacentes creadas por Azure Backup se conservan. Sin embargo, la ventaja de dejarlos es que puede restaurar el recurso compartido de archivos más adelante, si así lo desea. Para más información sobre el costo de dejar los puntos de recuperación, consulte la [información sobre precios](https://azure.microsoft.com/pricing/details/storage/files/). Si opta por eliminar todos los puntos de recuperación, no podrá restaurar el recurso compartido de archivos.

## <a name="stop-protection-and-retain-recovery-points"></a>Detención de la protección y conservación de los puntos de recuperación

Para detener la protección y retener los datos a la vez, puede usar el cmdlet [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection).

En el ejemplo siguiente se detiene la protección del recurso compartido de archivos *afsfileshare*, pero se conservan todos los puntos de recuperación:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "afstesting" -Name "afstest" | select -ExpandProperty ID
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureStorage -WorkloadType AzureFiles -Name "afsfileshare" -VaultId $vaultID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $vaultID
```

```output
WorkloadName     Operation         Status         StartTime                 EndTime                   JobID
------------     ---------         ------         ---------                 -------                   -----
afsfileshare     DisableBackup     Completed      1/26/2020 2:43:59 PM      1/26/2020 2:44:21 PM      98d9f8a1-54f2-4d85-8433-c32eafbd793f
```

El atributo Job ID de la salida se corresponde con el id. del trabajo creado por el servicio de copia de seguridad para la operación de "detención de la protección". Para supervisar el estado de un trabajo, use el cmdlet [Get-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob).

## <a name="stop-protection-without-retaining-recovery-points"></a>Detención de la protección sin conservar los puntos de recuperación

Para detener la protección sin conservar los puntos de recuperación, use el cmdlet [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) y agregue el parámetro **-RemoveRecoveryPoints**.

En el ejemplo siguiente se detiene la protección del recurso compartido de archivos *afsfileshare* sin conservar puntos de recuperación:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "afstesting" -Name "afstest" | select -ExpandProperty ID
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureStorage -WorkloadType AzureFiles -Name "afsfileshare" -VaultId $vaultID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $vaultID -RemoveRecoveryPoints
```

```output
WorkloadName     Operation            Status         StartTime                 EndTime                   JobID
------------     ---------            ------         ---------                 -------                   -----
afsfileshare     DeleteBackupData     Completed      1/26/2020 2:50:57 PM      1/26/2020 2:51:39 PM      b1a61c0b-548a-4687-9d15-9db1cc5bcc85
```

## <a name="next-steps"></a>Pasos siguientes

[Obtenga información acerca de](manage-afs-backup.md) la administración de copias de seguridad de recursos compartidos de archivos de Azure en Azure Portal.
