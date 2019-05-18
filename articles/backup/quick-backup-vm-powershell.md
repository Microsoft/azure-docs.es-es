---
title: 'Guía de inicio rápido de Azure: Copia de seguridad de una máquina virtual con PowerShell'
description: Aprenda a realizar copias de seguridad de sus máquinas virtuales con Azure PowerShell
services: backup
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 04/16/2019
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 05432f5a38c3d907afa95ac7b1b3adfe9c5515fe
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236341"
---
# <a name="back-up-a-virtual-machine-in-azure-with-powershell"></a>Copia de seguridad de una máquina virtual en Azure con PowerShell

El módulo [Azure PowerShell AZ](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-1.4.0) se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. 

[Azure Backup](backup-overview.md) puede hacer copias de seguridad de máquinas locales, aplicaciones y máquinas virtuales de Azure. En este artículo se muestra cómo realizar una copia de seguridad de una máquina virtual de Azure con el módulo AZ. Como alternativa, puede realizar una copia de seguridad de una máquina virtual mediante la [CLI de Azure](quick-backup-vm-cli.md), o en [Azure Portal](quick-backup-vm-portal.md).

Esta guía de inicio rápido permite realizar copias de seguridad en una máquina virtual de Azure existente. Si necesita crear una máquina virtual, puede [crearla con Azure PowerShell](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fpowershell%2fmodule%2ftoc.json).

Para realizar los pasos de esta guía, se requiere la versión 1.0.0 del módulo Azure PowerShell AZ, o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita instalarla o actualizarla, consulte el artículo sobre [cómo instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="sign-in-and-register"></a>Inicio de sesión y registro

1. Inicie sesión en la suscripción de Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

    ```powershell
    Connect-AzAccount
    ```
2. La primera vez que use Azure Backup, debe registrar el proveedor de Azure Recovery Services en su suscripción con [Register-AzResourceProvider](/powershell/module/az.Resources/Register-azResourceProvider) de la siguiente manera:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```


## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Un [almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md) es un contenedor lógico que almacena los datos de copia de seguridad de los recursos protegidos, como las máquinas virtuales de Azure. Cuando se ejecuta un trabajo de copia de seguridad, se crea un punto de recuperación en el almacén de Recovery Services. Posteriormente, se puede usar uno de estos puntos de recuperación para restaurar los datos a un momento dado en el tiempo.

Al crear el almacén:

- Como grupo de recursos y ubicación, especifique el grupo de recursos y la ubicación de la máquina virtual de la que desea realizar la copia de seguridad.
- Si ha usado el [script de ejemplo](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fpowershell%2fmodule%2ftoc.json) para crear la máquina virtual, el grupo de recursos se denomina **myResourceGroup**, la máquina virtual es ***myVM** y los recursos están en la región **WestEurope**.
- Azure Backup administra automáticamente el almacenamiento de los datos de los que se ha hecho la copia de seguridad. De forma predeterminada, el almacén usa el [almacenamiento con redundancia geográfica (GRS)](../storage/common/storage-redundancy-grs.md). El almacenamiento con redundancia geográfica garantiza que se repliquen los datos de copia de seguridad en una región de Azure secundaria que se encuentra a cientos de kilómetros de distancia de la región primaria.

Ahora, cree un almacén:


1. Use [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) para crear el almacén:

    ```powershell
    New-AzRecoveryServicesVault `
        -ResourceGroupName "myResourceGroup" `
        -Name "myRecoveryServicesVault" `
    -Location "WestEurope"
    ```

2. Establezca el contexto con [Set-AzRecoveryServicesVaultContext](/powershell/module/az.RecoveryServices/Set-azRecoveryServicesVaultContext) de la siguiente manera:

    ```powershell
    Get-AzRecoveryServicesVault `
        -Name "myRecoveryServicesVault" | Set-AzRecoveryServicesVaultContext
    ```

3. Cambie la configuración de redundancia de almacenamiento (LRS y GRS) del almacén con [Set-AzRecoveryServicesBackupProperty](https://docs.microsoft.com/powershell/module/az.recoveryservices/Set-AzRecoveryServicesBackupProperty), tal como se muestra a continuación:
    
    ```powershell
    Get-AzRecoveryServicesVault `
        -Name "myRecoveryServicesVault" | Set-AzRecoveryServicesBackupProperty -BackupStorageRedundancy LocallyRedundant/GeoRedundant
    ```
    > [!NOTE]
    > La redundancia de almacenamiento solo se puede modificar si no hay ningún elemento de copia de seguridad protegido en el almacén.

## <a name="enable-backup-for-an-azure-vm"></a>Habilitación de la copia de seguridad de una máquina virtual de Azure

Habilite la copia de seguridad para una máquina virtual de Azure y especifique una directiva de copia de seguridad.

- La directiva define cuándo debe ejecutarse la copia de seguridad y cuánto tiempo se deben retener los puntos de recuperación creados por las copias de seguridad.
- La directiva de protección predeterminada ejecuta una copia de seguridad de la máquina virtual una vez al día y conserva los puntos de recuperación creados durante 30 días. Puede usar esta directiva predeterminada para proteger rápidamente la máquina virtual. 

Habilite la copia de seguridad de la manera siguiente:

1. En primer lugar, establezca la directiva predeterminada con [Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy):

    ```powershell
    $policy = Get-AzRecoveryServicesBackupProtectionPolicy     -Name "DefaultPolicy"
    ```

2. Habilite la copia de seguridad de la máquina virtual con [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Especifique la directiva, el grupo de recursos y el nombre de la máquina virtual.

    ```powershell
    Enable-AzRecoveryServicesBackupProtection `
        -ResourceGroupName "myResourceGroup" `
        -Name "myVM" `
        -Policy $policy
    ```


## <a name="start-a-backup-job"></a>Inicio de un trabajo de copia de seguridad

Las copias de seguridad se ejecutan según la programación especificada en la directiva de copia de seguridad. También puede ejecutar una copia de seguridad ad hoc:

- El trabajo de copia de seguridad inicial crea un punto de recuperación completo.
- Después de la configuración inicial, cada trabajo de copia de seguridad crea puntos de recuperación incrementales.
- Los puntos de recuperación incremental ahorran tiempo y espacio de almacenamiento, ya que solo transfieren los cambios realizados desde la última copia de seguridad.

Para ejecutar una copia de seguridad ad hoc, utilice [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem). 
- Puede especificar un contenedor en el almacén que contenga los datos de copia de seguridad con [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer).
- Cada máquina virtual de la que se va a realizar una copia se trata como un elemento. Para iniciar un trabajo de copia de seguridad, obtenga información sobre la máquina virtual con [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

Ejecute un trabajo de copia de seguridad ad hoc de la siguiente manera:

1. Especifique el contenedor, obtenga información de la máquina virtual y ejecute la copia de seguridad.

    ```powershell
    $backupcontainer = Get-AzRecoveryServicesBackupContainer `
        -ContainerType "AzureVM" `
        -FriendlyName "myVM"

    $item = Get-AzRecoveryServicesBackupItem `
        -Container $backupcontainer `
        -WorkloadType "AzureVM"

    Backup-AzRecoveryServicesBackupItem -Item $item
    ```

2. Puede que tarde un máximo de 20 minutos ya que el primer trabajo de copia de seguridad crea un punto de recuperación completa. Supervise el trabajo como se describe en el procedimiento siguiente.


## <a name="monitor-the-backup-job"></a>Supervisión del trabajo de copia de seguridad

1. Ejecute [Get-AzRecoveryservicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) para supervisar el estado del trabajo.

    ```powershell
    Get-AzRecoveryservicesBackupJob
    ```
    La salida es similar al ejemplo siguiente, que muestra que el estado del trabajo es **InProgress**:

    ```
    WorkloadName   Operation         Status       StartTime              EndTime                JobID
    ------------   ---------         ------       ---------              -------                -----
    myvm           Backup            InProgress   9/18/2017 9:38:02 PM                          9f9e8f14
    myvm           ConfigureBackup   Completed    9/18/2017 9:33:18 PM   9/18/2017 9:33:51 PM   fe79c739
    ```

2. Si el estado del trabajo es **Completed**, la máquina virtual está protegida y se ha almacenado un punto de recuperación completa.


## <a name="clean-up-the-deployment"></a>Limpieza de la implementación

Si ya no necesita realizar copias de seguridad de la máquina virtual, puede limpiarla.
- Si desea restaurar la máquina virtual, omita el paso de limpieza.
- Si ha usado una máquina virtual existente, puede omitir el último cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para dejar tanto el grupo de recursos como la máquina virtual en su lugar.

Deshabilite la protección, elimine los puntos de restauración y el almacén. A continuación, elimine el grupo de recursos y los recursos de máquina virtual asociados como se indica a continuación:

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```


## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un almacén de Recovery Services, ha habilitado la protección en una máquina virtual y ha creado el punto de recuperación inicial. 

- [Aprenda](tutorial-backup-vm-at-scale.md) a realizar una copia de seguridad de máquinas virtuales en Azure Portal.
- [Aprenda a](tutorial-restore-disk.md) restaurar rápidamente una máquina virtual.
