---
title: 'Tutorial: Copia de seguridad de varias máquinas virtuales de Azure con PowerShell'
description: En este tutorial se detalla la copia de seguridad de varias máquinas virtuales de Azure en un almacén de Recovery Services mediante Azure PowerShell.
ms.topic: tutorial
ms.date: 03/05/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 33090a0156d147fee5ab362b284ec503039617c4
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89181738"
---
# <a name="back-up-azure-vms-with-powershell"></a>Copia de seguridad de máquinas virtuales de Azure con PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este tutorial se indica cómo implementar un almacén de Recovery Services en [Azure Backup](backup-overview.md) para realizar la copia de seguridad de varias máquinas virtuales de Azure con PowerShell.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear un almacén de Recovery Services y a establecer el contexto de este.
> * Definición de una directiva de copia de seguridad
> * Aplicación de la directiva de copia de seguridad para proteger varias máquinas virtuales
> * Desencadenar un trabajo de copia de seguridad a petición para las máquinas virtuales protegidas. Para poder realizar una copia de seguridad de una máquina virtual, o protegerla, es necesario reunir los [requisitos previos](backup-azure-arm-vms-prepare.md) a fin de preparar el entorno para la protección de las máquinas virtuales.

> [!IMPORTANT]
> En este tutorial se da por supuesto que ya ha creado un grupo de recursos y una máquina virtual de Azure.

## <a name="sign-in-and-register"></a>Inicio de sesión y registro

1. Inicie sesión en la suscripción a Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

    ```powershell
    Connect-AzAccount
    ```

2. La primera vez que use Azure Backup, debe registrar el proveedor de Azure Recovery Services en su suscripción con [Register-AzResourceProvider](/powershell/module/az.Resources/Register-azResourceProvider). Si ya se ha registrado, puede omitir este paso.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Un [almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md) es un contenedor lógico que almacena los datos de copia de seguridad de los recursos protegidos, como las máquinas virtuales de Azure. Cuando se ejecuta un trabajo de copia de seguridad, se crea un punto de recuperación en el almacén de Recovery Services. Posteriormente, se puede usar uno de estos puntos de recuperación para restaurar los datos a un momento dado en el tiempo.

* En este tutorial, puede crear el almacén en el mismo grupo de recursos y ubicación que la máquina virtual de la que desea realizar la copia de seguridad.
* Azure Backup administra automáticamente el almacenamiento de los datos de los que se ha hecho la copia de seguridad. De forma predeterminada, el almacén usa el [almacenamiento con redundancia geográfica (GRS)](../storage/common/storage-redundancy.md). El almacenamiento con redundancia geográfica garantiza que se repliquen los datos de copia de seguridad en una región de Azure secundaria que se encuentra a cientos de kilómetros de distancia de la región primaria.

Cree el almacén como se indica a continuación:

1. Use [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) para crear el almacén. Especifique el nombre del grupo de recursos y la ubicación de la máquina virtual de la que desea realizar la copia de seguridad.

    ```powershell
    New-AzRecoveryServicesVault -Name myRSvault -ResourceGroupName "myResourceGroup" -Location "EastUS"
    ```

2. Muchos de los cmdlets de Azure Backup requieren el objeto de almacén de Recovery Services como entrada. Por este motivo, es conveniente almacenar el objeto de almacén de Recovery Services de Backup en una variable.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault –Name myRSVault
    ```

3. Establezca el contexto con [Set-AzRecoveryServicesVaultContext](/powershell/module/az.RecoveryServices/Set-azRecoveryServicesVaultContext).

   * El contexto de almacén es el tipo de los datos protegidos en el almacén.
   * Una vez que se haya establecido el contexto, se aplica a todos los cmdlets posteriores.

     ```powershell
     Get-AzRecoveryServicesVault -Name "myRSVault" | Set-AzRecoveryServicesVaultContext
     ```

## <a name="back-up-azure-vms"></a>Copia de seguridad de máquinas virtuales de Azure

Las copias de seguridad se ejecutan según la programación especificada en la directiva de copia de seguridad. Cuando se crea un almacén de Recovery Services, este incluye las directivas de retención y protección predeterminadas.

* La directiva de protección predeterminada desencadena un trabajo de copia de seguridad una vez al día a la hora indicada.
* La directiva de retención predeterminada conserva el punto de recuperación diario durante 30 días.

Para habilitar la copia de seguridad de la máquina virtual de Azure en este tutorial, se hace lo siguiente:

1. Especifique un contenedor en el almacén que contenga los datos de copia de seguridad con [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer).
2. Cada máquina virtual de la copia de seguridad es un elemento. Para iniciar un trabajo de copia de seguridad, obtenga información sobre la máquina virtual con [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem).
3. Ejecute una copia de seguridad a petición con [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem).
    * El trabajo de copia de seguridad inicial crea un punto de recuperación completo.
    * Después de la configuración inicial, cada trabajo de copia de seguridad crea puntos de recuperación incrementales.
    * Los puntos de recuperación incremental ahorran tiempo y espacio de almacenamiento, ya que solo transfieren los cambios realizados desde la última copia de seguridad.

Habilite y ejecute la copia de seguridad como sigue:

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered -FriendlyName "V2VM"
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType AzureVM
$job = Backup-AzRecoveryServicesBackupItem -Item $item
```

## <a name="troubleshooting"></a>Solución de problemas

Si se encuentra con problemas mientras realiza la copia de seguridad de la máquina virtual, repase este [artículo de solución de problemas](backup-azure-vms-troubleshoot.md).

### <a name="deleting-a-recovery-services-vault"></a>Eliminación de un almacén de Recovery Services

Si es necesario eliminar un almacén, debe eliminar primero los puntos de recuperación de este y, a continuación, anular el registro del almacén de la siguiente forma:

```powershell
$Cont = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered
$PI = Get-AzRecoveryServicesBackupItem -Container $Cont[0] -WorkloadType AzureVm
Disable-AzRecoveryServicesBackupProtection -RemoveRecoveryPoints $PI[0]
Unregister-AzRecoveryServicesBackupContainer -Container $namedContainer
Remove-AzRecoveryServicesVault -Vault $vault1
```

## <a name="next-steps"></a>Pasos siguientes

* [Repase](backup-azure-vms-automation.md) un tutorial más detallado de copia de seguridad y restauración de máquinas virtuales de Azure con PowerShell.
* [Administración y supervisión de copias de seguridad de máquinas virtuales de Azure](backup-azure-manage-vms.md)
* [Restauración de máquinas virtuales de Azure](backup-azure-arm-restore-vms.md)
