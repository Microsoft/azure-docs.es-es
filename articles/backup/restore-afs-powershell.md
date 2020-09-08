---
title: Restauración de Azure Files con PowerShell
description: En este artículo se aprende a restaurar Azure Files mediante el servicio Azure Backup y PowerShell.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: 60c9848e12de80bcafe4553a9e8f3e27e8876d41
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826504"
---
# <a name="restore-azure-files-with-powershell"></a>Restauración de Azure Files con PowerShell

En este artículo se explica cómo restaurar un recurso compartido de archivos completo, o archivos específicos, desde un punto de restauración creado por el servicio [Azure Backup](backup-overview.md) mediante Azure PowerShell.

Puede restaurar un recurso compartido de archivos al completo o archivos específicos de dicho recurso compartido. La restauración puede llevarse a cabo en la ubicación original o en una ubicación alternativa.

> [!WARNING]
> Asegúrese de que la versión de PowerShell se actualice a la versión mínima de "Az.RecoveryServices 2.6.0" para las copias de seguridad de AFS. Para más información, consulte [la sección](backup-azure-afs-automation.md#important-notice-backup-item-identification) que describe el requisito de este cambio.

>[!NOTE]
>Azure Backup admite ahora la restauración de varios archivos o carpetas en la ubicación original o alternativa mediante PowerShell. Consulte [esta sección](#restore-multiple-files-or-folders-to-original-or-alternate-location) del documento para más información sobre cómo hacerlo.

## <a name="fetch-recovery-points"></a>Captura de puntos de recuperación

Use [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) para enumerar todos los puntos de recuperación del elemento del que se ha realizado la copia de seguridad.

En el script siguiente:

* La variable **$rp**es una matriz de puntos de recuperación para el elemento de copia de seguridad seleccionado de los últimos siete días.
* La matriz se ordena en orden inverso de tiempo con el punto de recuperación más reciente en el índice **0**.
* Use la indexación de matrices de PowerShell estándar para seleccionar el punto de recuperación.
* En el ejemplo, **$rp[0]** selecciona el último punto de recuperación.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"
$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID
$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0] | fl
```

La salida será similar a la siguiente.

```powershell
FileShareSnapshotUri : https://testStorageAcct.file.core.windows.net/testAzureFS?sharesnapshot=2018-11-20T00:31:04.00000
                       00Z
RecoveryPointType    : FileSystemConsistent
RecoveryPointTime    : 11/20/2018 12:31:05 AM
RecoveryPointId      : 86593702401459
ItemName             : testAzureFS
Id                   : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Micros                      oft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;teststorageRG;testStorageAcct/protectedItems/AzureFileShare;testAzureFS/recoveryPoints/86593702401462
WorkloadType         : AzureFiles
ContainerName        : storage;teststorageRG;testStorageAcct
ContainerType        : AzureStorage
BackupManagementType : AzureStorage
```

Una vez seleccionado el punto de recuperación adecuado, restaure el recurso compartido de archivos o el archivo en la ubicación original o en una ubicación alternativa.

## <a name="restore-an-azure-file-share-to-an-alternate-location"></a>Restauración de un recurso compartido de archivos de Azure en una ubicación alternativa

Use [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) para restaurar en el punto de recuperación seleccionado. Especifique estos parámetros para identificar la ubicación alternativa:

* **TargetStorageAccountName**: La cuenta de almacenamiento en la que se restaura el contenido de la copia de seguridad. La cuenta de almacenamiento de destino debe estar en la misma ubicación que el almacén.
* **TargetFileShareName**: Los recursos compartidos de archivos de la cuenta de almacenamiento de destino en la que se restaura el contenido de la copia de seguridad.
* **TargetFolder**: La carpeta del recurso compartido de archivos en la que se restauran los datos. Si el contenido de la copia de seguridad debe restaurarse en una carpeta raíz, asigne los valores de la carpeta de destino como una cadena vacía.
* **ResolveConflict**: Instrucción en caso de conflicto con los datos restaurados. Acepta **Overwrite** o **Skip**.

Ejecute el cmdlet con los parámetros de la manera siguiente:

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -ResolveConflict Overwrite
```

El comando devuelve un trabajo con un identificador del cual se puede realizar un seguimiento, como se indica en el ejemplo siguiente.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS        Restore              InProgress           12/10/2018 9:56:38 AM                               9fd34525-6c46-496e-980a-3740ccb2ad75
```

## <a name="restore-an-azure-file-to-an-alternate-location"></a>Restauración de un archivo de Azure en una ubicación alternativa

Use [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) para restaurar en el punto de recuperación seleccionado. Especifique estos parámetros para identificar la ubicación alternativa y para identificar de forma única el archivo que quiere restaurar.

* **TargetStorageAccountName**: La cuenta de almacenamiento en la que se restaura el contenido de la copia de seguridad. La cuenta de almacenamiento de destino debe estar en la misma ubicación que el almacén.
* **TargetFileShareName**: Los recursos compartidos de archivos de la cuenta de almacenamiento de destino en la que se restaura el contenido de la copia de seguridad.
* **TargetFolder**: La carpeta del recurso compartido de archivos en la que se restauran los datos. Si el contenido de la copia de seguridad debe restaurarse en una carpeta raíz, asigne los valores de la carpeta de destino como una cadena vacía.
* **SourceFilePath**: La ruta de acceso absoluta del archivo que se va a restaurar en el recurso compartido de archivos, como una cadena. Esta ruta de acceso es la misma ruta utilizada en el cmdlet de PowerShell **Get-AzStorageFile**.
* **SourceFileType**: Indica si se ha seleccionado un directorio o un archivo. Admite **Directory** o **File**.
* **ResolveConflict**: Instrucción en caso de conflicto con los datos restaurados. Acepta **Overwrite** o **Skip**.

Los parámetros adicionales (SourceFilePath y SourceFileType) solo están relacionadas con el archivo individual que quiere restaurar.

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

Este comando devuelve un trabajo con un identificador del cual se puede realizar un seguimiento, como se indica en la sección anterior.

## <a name="restore-azure-file-shares-and-files-to-the-original-location"></a>Restauración de archivos y recursos compartidos de archivos de Azure en la ubicación original

Al restaurar en una ubicación original, no es necesario especificar parámetros relacionados con el destino. Solo debe proporcionarse **ResolveConflict**.

### <a name="overwrite-an-azure-file-share"></a>Sobrescribir un recurso compartido de archivos de Azure

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -ResolveConflict Overwrite
```

### <a name="overwrite-an-azure-file"></a>Sobrescribir un archivo de Azure

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

## <a name="restore-multiple-files-or-folders-to-original-or-alternate-location"></a>Restauración de varios archivos o carpetas en una ubicación original o alternativa

Utilice el comando [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) pasando la ruta de acceso de todos los archivos o carpetas que desea restaurar como valor para el parámetro **MultipleSourceFilePath**.

### <a name="restore-multiple-files"></a>Restauración de varios archivos

En el siguiente script, se intenta restaurar los archivos *FileSharePage.png* y *MyTestFile.txt*.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"

$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID

$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"

$RP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID

$files = ("FileSharePage.png", "MyTestFile.txt")

Restore-AzRecoveryServicesBackupItem -RecoveryPoint $RP[0] -MultipleSourceFilePath $files -SourceFileType File -ResolveConflict Overwrite -VaultId $vault.ID -VaultLocation $vault.Location
```

### <a name="restore-multiple-directories"></a>Restauración de varios directorios

En el siguiente script, se intenta restaurar los directorios *zrs1_restore* y *Restore*.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"

$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID

$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"

$RP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID

$files = ("Restore","zrs1_restore")

Restore-AzRecoveryServicesBackupItem -RecoveryPoint $RP[0] -MultipleSourceFilePath $files -SourceFileType Directory -ResolveConflict Overwrite -VaultId $vault.ID -VaultLocation $vault.Location
```

El resultado será similar al siguiente:

```output
WorkloadName         Operation         Status          StartTime                EndTime       JobID
------------         ---------         ------          ---------                -------       -----
azurefiles           Restore           InProgress      4/5/2020 8:01:24 AM                    cd36abc3-0242-44b1-9964-0a9102b74d57
```

Si desea restaurar varios archivos o carpetas en una ubicación alternativa, utilice los scripts anteriores especificando los valores de parámetros relacionados con la ubicación de destino, tal como se ha explicado anteriormente en [Restauración de un archivo de Azure en una ubicación alternativa](#restore-an-azure-file-to-an-alternate-location).

## <a name="next-steps"></a>Pasos siguientes

[Aprenda a](restore-afs.md) restaurar Azure Files en Azure Portal.
