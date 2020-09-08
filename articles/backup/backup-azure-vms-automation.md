---
title: Copia de seguridad y recuperación de máquinas virtuales con PowerShell
description: Describe cómo realizar una copia de seguridad y llevar a cabo la recuperación de máquinas virtuales de Azure mediante Azure Backup con PowerShell.
ms.topic: conceptual
ms.date: 09/11/2019
ms.openlocfilehash: f34dc0b5ce4b230b3bc2408bd011180cb855cf17
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892412"
---
# <a name="back-up-and-restore-azure-vms-with-powershell"></a>Copia de seguridad y restauración de máquinas virtuales de con PowerShell

En este artículo se explica cómo realizar la copia de seguridad y la restauración de una máquina virtual de Azure en un almacén de [Azure Backup](backup-overview.md) Recovery Services mediante cmdlets de PowerShell.

En este artículo, aprenderá a:

> [!div class="checklist"]
>
> * Crear un almacén de Recovery Services y a establecer el contexto de este.
> * Definición de una directiva de copia de seguridad
> * Aplicación de la directiva de copia de seguridad para proteger varias máquinas virtuales
> * Desencadenar un trabajo de copia de seguridad a petición para las máquinas virtuales protegidas. Para poder realizar una copia de seguridad de una máquina virtual, o protegerla, es necesario reunir los [requisitos previos](backup-azure-arm-vms-prepare.md) a fin de preparar el entorno para la protección de las máquinas virtuales.

## <a name="before-you-start"></a>Antes de comenzar

* [Más información](backup-azure-recovery-services-vault-overview.md) sobre los almacenes de Recovery Services.
* [Revise](backup-architecture.md#architecture-built-in-azure-vm-backup) la arquitectura de copia de seguridad de la máquina virtual de Azure, [conozca](backup-azure-vms-introduction.md) el proceso de copia de seguridad y [revise](backup-support-matrix-iaas.md) la compatibilidad, las limitaciones y los requisitos previos.
* Revise la jerarquía de objetos de PowerShell para Recovery Services.

## <a name="recovery-services-object-hierarchy"></a>Jerarquía de objetos de Recovery Services

En el diagrama siguiente, se resume la jerarquía de objetos.

![Jerarquía de objetos de Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Revise la [referencia de cmdlet](/powershell/module/az.recoveryservices/) de **Az.RecoveryServices** en la biblioteca de Azure.

## <a name="set-up-and-register"></a>Configuración y registro

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para empezar:

1. [Descargue la versión más reciente de PowerShell](/powershell/azure/install-az-ps).

2. Para buscar los cmdlets de PowerShell de Azure Backup disponibles, escriba el siguiente comando:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

    Aparecen los alias y cmdlets de Azure Backup, Azure Site Recovery y el almacén de Recovery Services. La imagen siguiente es un ejemplo de lo que verá. No es la lista completa de los cmdlets.

    ![Lista de Recovery Services](./media/backup-azure-vms-automation/list-of-recoveryservices-ps.png)

3. Inicie sesión en su cuenta de Azure mediante el cmdlet **Connect-AzAccount**. El cmdlet abrirá una página web que le solicitará las credenciales de la cuenta:

    * Como alternativa, puede incluir sus credenciales de cuenta como un parámetro en el cmdlet **Connect-AzAccount** mediante el parámetro **-Credential**.
    * Si usted es partner CSP que trabaja en nombre de un inquilino, especifique el cliente como inquilino usando su TenantID o su nombre de dominio principal de inquilino. Por ejemplo: **Connect-AzAccount -Tenant "fabrikam.com"**

4. Ya que una cuenta puede tener varias suscripciones, le recomendamos que asocie la suscripción que quiera usar a esa cuenta:

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

5. Si usa Azure Backup por primera vez, debe usar el cmdlet **[Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider)** para registrar el proveedor de Azure Recovery Services con su suscripción.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

6. Puede comprobar que los proveedores se registraron correctamente mediante los siguientes comandos:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

    En la salida del comando, **RegistrationState** se debe cambiar a **Registrado**. En caso contrario, vuelva a ejecutar el cmdlet **[Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider)** .

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Los siguientes pasos le guiarán por el proceso de creación de un almacén de Recovery Services. Un almacén de Recovery Services no es lo mismo que un almacén de copia de seguridad.

1. El almacén de Recovery Services es un recurso de Resource Manager, por lo que deberá colocarlo dentro de un grupo de recursos. Puede usar un grupo de recursos existente o crear uno con el cmdlet **[New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)** . Al crear un grupo de recursos, especifique el nombre y la ubicación.  

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Use el cmdlet [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) para crear el almacén de Recovery Services. Asegúrese de especificar para el almacén la misma ubicación del grupo de recursos.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Especifique el tipo de redundancia de almacenamiento que se usará. Puede usar [almacenamiento con redundancia local (LRS)](../storage/common/storage-redundancy.md) o [almacenamiento con redundancia geográfica (GRS)](../storage/common/storage-redundancy.md). En el ejemplo siguiente se muestra que la opción -BackupStorageRedundancy para testvault está establecida en GeoRedundant.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperty  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

   > [!TIP]
   > Muchos de los cmdlets de Azure Backup requieren el objeto de almacén de Recovery Services como entrada. Por este motivo, es conveniente almacenar el objeto de almacén de Recovery Services de Backup en una variable.
   >
   >

## <a name="view-the-vaults-in-a-subscription"></a>Visualización de los almacenes de una suscripción

Para ver todos los almacenes de la suscripción, use [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault):

```powershell
Get-AzRecoveryServicesVault
```

El resultado es similar al ejemplo siguiente, tenga en cuenta que los valores de ResourceGroupName y Location asociados se proporcionan.

```output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

## <a name="back-up-azure-vms"></a>Copia de seguridad de máquinas virtuales de Azure

Use un almacén de Recovery Services para proteger sus máquinas virtuales. Antes de aplicar la protección, establezca el contexto de almacén (el tipo de datos protegido en el almacén) y compruebe la directiva de protección. La directiva de protección consiste en la programación que indica cuándo se debe ejecutar el trabajo de copia de seguridad y cuánto tiempo se conserva cada instantánea de copia de seguridad.

### <a name="set-vault-context"></a>Establecer el contexto de almacén

Antes de habilitar la protección en una máquina virtual, use [Set-AzRecoveryServicesVaultContext](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext) para establecer el contexto de almacén. Una vez que se haya establecido el contexto de almacén, se aplica a todos los cmdlets posteriores. En el ejemplo siguiente se establece el contexto del almacén *testvault*.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "Contoso-docs-rg" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Recuperación del identificador del almacén

Tenemos previsto dejar de usar la configuración del contexto de almacén según las directrices de Azure PowerShell. En su lugar, puede almacenar o recuperar el identificador del almacén y pasarlo a los comandos pertinentes. Por lo tanto, si no ha establecido el contexto de almacén o quiere especificar el comando que se va a ejecutar para un determinado almacén, pase el Id. de almacén como "-vaultID" a todos los comandos pertinentes de la manera siguiente:

```powershell
$targetVault = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault"
$targetVault.ID
```

Or

```powershell
$targetVaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

### <a name="modifying-storage-replication-settings"></a>Modificación de la configuración de replicación de almacenamiento

Use el comando [Set-AzRecoveryServicesBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) para proporcionar la configuración de replicación de Storage del almacén a LRS/GRS.

```powershell
Set-AzRecoveryServicesBackupProperty -Vault $targetVault -BackupStorageRedundancy GeoRedundant/LocallyRedundant
```

> [!NOTE]
> La redundancia de almacenamiento solo se puede modificar si no hay ningún elemento de copia de seguridad protegido en el almacén.

### <a name="create-a-protection-policy"></a>Creación de una directiva de protección

Cuando se crea un almacén de Recovery Services, este incluye las directivas de retención y protección predeterminadas. La directiva de protección predeterminada activa un trabajo de copia de seguridad cada día a la hora indicada. La directiva de retención predeterminada conserva el punto de recuperación diario durante 30 días. Puede utilizar la directiva predeterminada para proteger rápidamente la máquina virtual y modificar la directiva más adelante con detalles diferentes.

Use **[Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy)** para ver las directivas de protección disponibles en el almacén. Puede usar este cmdlet para obtener una directiva específica o para ver las directivas asociadas a un tipo de carga de trabajo. En el ejemplo siguiente se obtienen las directivas para el tipo de carga de trabajo AzureVM.

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

La salida es similar a la del ejemplo siguiente:

```output
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM
```

> [!NOTE]
> La zona horaria del campo BackupTime en PowerShell es UTC. Sin embargo, cuando el tiempo de copia de seguridad se muestra en Azure Portal, la hora se ajusta a la zona horaria local.
>
>

Una directiva de protección de copia de seguridad está asociada con al menos una directiva de retención. Una directiva de retención define el tiempo que se conserva un punto de recuperación antes de que se elimine.

* Use [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject) para ver la directiva de retención predeterminada.
* Del mismo modo, puede usar [Get-AzRecoveryServicesBackupSchedulePolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject) para obtener la directiva de programación predeterminada.
* El cmdlet [New-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy) crea un objeto de PowerShell que contiene información de la directiva de copia de seguridad.
* Los objetos de directiva de retención y programación se usan como entradas para el cmdlet New-AzRecoveryServicesBackupProtectionPolicy.

De forma predeterminada, se define una hora de inicio en el objeto de directiva de programación. Use el siguiente ejemplo para cambiar la hora de inicio a la hora de inicio deseada. La hora de inicio deseada debe estar también en formato UTC. En el ejemplo siguiente se supone que la hora de inicio deseada es la 1:00 UTC para las copias de seguridad diarias.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Solo tiene que proporcionar la hora de inicio en múltiplos de 30 minutos. En el ejemplo anterior, solo puede ser "01:00:00" o "02:30:00". La hora de inicio no puede ser "01:15:00".

En el ejemplo siguiente se almacenan la directiva de programación y la directiva de retención en variables. En el ejemplo se usan esas variables para definir los parámetros al crear la directiva de protección *NewPolicy*.

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType "AzureVM" -RetentionPolicy $retPol -SchedulePolicy $schPol -VaultId $targetVault.ID
```

La salida es similar a la del ejemplo siguiente:

```output
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM
```

### <a name="enable-protection"></a>Habilitar protección

Una vez que haya definido la directiva de protección, todavía debe habilitar la directiva para un elemento. Use [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) para habilitar la protección. Para habilitar la protección son necesarios dos objetos: el elemento y la directiva. Después de que la directiva se haya asociado con el almacén, el flujo de trabajo de copia de seguridad se desencadena a la hora definida en la programación de la directiva.

> [!IMPORTANT]
> Al usar PowerShell para habilitar la copia de seguridad de varias VM a la vez, asegúrese de que una sola directiva no tenga más de 100 VM asociadas. Este es un [procedimiento recomendado](./backup-azure-vm-backup-faq.md#is-there-a-limit-on-number-of-vms-that-can-beassociated-with-the-same-backup-policy). Actualmente, el cliente de PowerShell no se bloquea explícitamente si hay más de 100 máquinas virtuales, pero está previsto que la comprobación se agregue en el futuro.

En los ejemplos siguientes se habilita la protección para el elemento V2VM mediante la directiva NewPolicy. Los ejemplos varían en función de si la VM está cifrada y del tipo de cifrado.

Para habilitar la protección en **VM de Resource Manager sin cifrar**:

```powershell
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

Para habilitar la protección en VM cifradas (mediante BEK y KEK), debe conceder permiso al servicio Azure Backup para que lea las claves y los secretos del almacén de claves.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

Para habilitar la protección en **VM cifradas (mediante BEK únicamente)** , debe conceder permiso al servicio Azure Backup para que lea los secretos del almacén de claves.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToSecrets backup,get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

> [!NOTE]
> Si está usando la nube de Azure Government, use el valor `ff281ffe-705c-4f53-9f37-a40e6f2c68f3` para el parámetro **ServicePrincipalName** en el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy).
>

## <a name="monitoring-a-backup-job"></a>Supervisión de trabajos de copia de seguridad

Puede supervisar las operaciones de ejecución prolongada, como los trabajos de copia de seguridad, sin usar Azure Portal. Para obtener el estado de un trabajo en curso, use el cmdlet [Get-AzRecoveryservicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob). Este cmdlet obtiene los trabajos de copia de seguridad de un almacén específico, y dicho almacén se especifica en el contexto de almacén. En el ejemplo siguiente se obtiene el estado de un trabajo en curso como una matriz y se almacena el estado en la variable $joblist.

```powershell
$joblist = Get-AzRecoveryservicesBackupJob –Status "InProgress" -VaultId $targetVault.ID
$joblist[0]
```

La salida es similar a la del ejemplo siguiente:

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM             Backup               InProgress            4/23/2016                5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

En lugar de sondear si estos trabajos han finalizado (lo que supone un código adicional innecesario), use el cmdlet [Wait-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob). Este cmdlet detiene la ejecución hasta que el trabajo se complete o se alcance el valor de tiempo de espera especificado.

```powershell
Wait-AzRecoveryServicesBackupJob -Job $joblist[0] -Timeout 43200 -VaultId $targetVault.ID
```

## <a name="manage-azure-vm-backups"></a>Administración de copias de seguridad de máquinas virtuales de Azure

### <a name="modify-a-protection-policy"></a>Modificación de una directiva de protección

Para modificar la directiva de protección, use [Set-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) para modificar los objetos SchedulePolicy o RetentionPolicy.

#### <a name="modifying-scheduled-time"></a>Modificación de la hora programada

Cuando crea una directiva de protección, se asigna una hora de inicio de manera predeterminada. En los ejemplos siguientes se muestra cómo modificar la hora de inicio de una directiva de protección.

````powershell
$SchPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z" (This is the time that you want to start the backup)
$UtcTime = $UtcTime.ToUniversalTime()
$SchPol.ScheduleRunTimes[0] = $UtcTime
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -SchedulePolicy $SchPol -VaultId $targetVault.ID
````

#### <a name="modifying-retention"></a>Modificación de retención

En el ejemplo siguiente se cambia la retención del punto de recuperación a 365 días.

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
$retPol.DailySchedule.DurationCountInDays = 365
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy $RetPol -VaultId $targetVault.ID
```

#### <a name="configuring-instant-restore-snapshot-retention"></a>Configuración de la retención de instantáneas de restauración instantánea

> [!NOTE]
> Desde Azure PowerShell 1.6.0 y versiones posteriores, se puede actualizar el período de retención de instantáneas para la restauración instantánea en la directiva mediante PowerShell.

````powershell
$bkpPol = Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM" -VaultId $targetVault.ID
$bkpPol.SnapshotRetentionInDays=7
Set-AzRecoveryServicesBackupProtectionPolicy -policy $bkpPol -VaultId $targetVault.ID
````

El valor predeterminado será 2. Puede establecer el valor con 1 como mínimo y 5 como máximo. Para las directivas de copia de seguridad semanal, el período se establece en 5 y no se puede cambiar.

#### <a name="creating-azure-backup-resource-group-during-snapshot-retention"></a>Creación de un grupo de recursos de Azure Backup durante la retención de instantáneas

> [!NOTE]
> A partir de la versión 3.7.0 de Azure PowerShell en adelante, se puede crear y editar el grupo de recursos creado para almacenar instantáneas rápidas.

Para obtener más información acerca de las reglas de creación de grupos de recursos y otros detalles relevantes, consulte la documentación [Grupo de recursos de Azure Backup para máquinas virtuales](./backup-during-vm-creation.md#azure-backup-resource-group-for-virtual-machines).

```powershell
$bkpPol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -name "DefaultPolicyForVMs"
$bkpPol.AzureBackupRGName="Contosto_"
$bkpPol.AzureBackupRGNameSuffix="ForVMs"
Set-AzureRmRecoveryServicesBackupProtectionPolicy -policy $bkpPol
```

### <a name="trigger-a-backup"></a>Desencadenar una copia de seguridad

Use [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) para desencadenar un trabajo de copia de seguridad. Si se trata de la copia de seguridad inicial, es una copia de seguridad completa. Las sucesivas copias de seguridad toman una copia incremental. En el ejemplo siguiente se toma una copia de seguridad de máquina virtual que se conservará durante 60 días.

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
$job = Backup-AzRecoveryServicesBackupItem -Item $item -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

La salida es similar a la del ejemplo siguiente:

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM              Backup              InProgress          4/23/2016                  5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

> [!NOTE]
> La zona horaria de los campos de StartTime y EndTime en PowerShell es UTC. Sin embargo, cuando la hora se muestra en Azure Portal, esta se ajusta a la zona horaria local.
>
>

### <a name="change-policy-for-backup-items"></a>Cambio de la directiva para los elementos de copia de seguridad

Puede modificar la directiva existente o cambiar la directiva del elemento de copia de seguridad de Policy1 a Policy2. Para cambiar las directivas para un elemento de copia de seguridad, capture la directiva correspondiente y haga una copia de seguridad del elemento, y use el comando [Enable-AzRecoveryServices](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) con el elemento de copia de seguridad como parámetro.

````powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName> -VaultId $targetVault.ID
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType AzureVM -BackupManagementType AzureVM -Name "<BackupItemName>" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1 -VaultId $targetVault.ID
````

El comando espera hasta que la copia de seguridad de la configuración se complete y devuelve la salida siguiente.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
TestVM           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="stop-protection"></a>Detener protección

#### <a name="retain-data"></a>Conservación de los datos

Si quiere detener la protección, puede usar el cmdlet [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) de PowerShell. De esta forma, se detendrán las copias de seguridad programadas, pero los datos que se hayan incluido en ellas hasta el momento se conservarán.

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
````

#### <a name="delete-backup-data"></a>Eliminación de datos de copia de seguridad

Para quitar completamente los datos de copia de seguridad almacenados en el almacén, agregue el marcador "-RemoveRecoveryPoints" al [comando de protección "disable"](#retain-data).

````powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
````

## <a name="restore-an-azure-vm"></a>Restauración de máquinas virtuales de Azure

Hay una diferencia importante entre restaurar una VM mediante Azure Portal y hacerlo con PowerShell. Con PowerShell, la operación de restauración se completa una vez que se creen los discos y la información de configuración a partir del punto de recuperación. La operación de restauración no crea la máquina virtual. Para crear una máquina virtual desde el disco, consulte la sección sobre la [creación de la máquina virtual a partir de los discos almacenados](backup-azure-vms-automation.md#create-a-vm-from-restored-disks). Si no quiere restaurar toda la VM, sino que quiere restaurar o recuperar algunos archivos desde una copia de seguridad de la VM de Azure, consulte la [sección recuperación de archivos](backup-azure-vms-automation.md#restore-files-from-an-azure-vm-backup).

> [!Tip]
> La operación de restauración no crea la máquina virtual.
>
>

En el siguiente gráfico se muestra la jerarquía de objetos desde RecoveryServicesVault hasta BackupRecoveryPoint.

![Jerarquía de objetos de Recovery Services donde se muestra BackupContainer](./media/backup-azure-vms-arm-automation/backuprecoverypoint-only.png)

Para restaurar los datos de una copia de seguridad, identifique el elemento del que se realizó la copia de seguridad y el punto de recuperación que contiene los datos de un momento específico. Use el cmdlet [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) para restaurar los datos del almacén en la cuenta.

Los pasos básicos para restaurar una máquina virtual de Azure son los siguientes:

* Seleccione la máquina virtual.
* Elija un punto de recuperación.
* Restaure los discos.
* Cree la VM a partir de los discos almacenados.

### <a name="select-the-vm-when-restoring-files"></a>Selección de la máquina virtual (al restaurar archivos)

Para obtener el objeto de PowerShell que identifica el elemento de copia de seguridad correcto, comience en el contenedor del almacén y avance hacia abajo por la jerarquía de objetos. Para seleccionar el contenedor que representa la máquina virtual, use el cmdlet [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) y canalícelo al cmdlet [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

### <a name="choose-a-recovery-point-when-restoring-files"></a>Elección de un punto de recuperación (al restaurar archivos)

Use el cmdlet [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) para enumerar todos los puntos de recuperación del elemento de copia de seguridad. Después, elija el punto de recuperación que se debe restaurar. Si no está seguro de qué punto de recuperación debe usar, se recomienda elegir el punto RecoveryPointType = AppConsistent más reciente de la lista.

En el siguiente script, la variable **$rp**es una matriz de puntos de recuperación para el elemento de copia de seguridad seleccionado de los últimos siete días. La matriz se ordena en orden inverso de tiempo con el punto de recuperación más reciente en el índice 0. Use la indexación de matrices de PowerShell estándar para seleccionar el punto de recuperación. En el ejemplo, $rp[0] selecciona el último punto de recuperación.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
$rp[0]
```

La salida es similar a la del ejemplo siguiente:

```output
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="restore-the-disks"></a>Restauración de los discos

Use el cmdlet [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) para restaurar los datos y la configuración de un elemento de copia de seguridad a un punto de recuperación. Una vez que haya identificado un punto de recuperación, úselo como valor para el parámetro **-RecoveryPoint**. En el ejemplo anterior, **$rp[0]** era el punto de recuperación que se debía usar. En el código de ejemplo siguiente, **$rp[0]** es el punto de recuperación que se va a restaurar en el disco.

Para restaurar los discos y la información de configuración:

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -VaultId $targetVault.ID
$restorejob
```

#### <a name="restore-managed-disks"></a>Restauración de discos administrados

> [!NOTE]
> Si la máquina virtual con respaldo tiene discos administrados y quiere restaurarlos como discos administrados, hemos introducido la funcionalidad del módulo RM para Azure PowerShell v 6.7.0. y versiones posteriores.
>
>

Incluya un parámetro **TargetResourceGroupName** adicional para especificar el grupo de recursos en el que se restaurarán los discos administrados.

> [!IMPORTANT]
> Se recomienda encarecidamente utilizar el parámetro **TargetResourceGroupName** para la restauración de discos administrados, ya que ofrece importantes mejoras de rendimiento. Si no se especifica este parámetro, no puede beneficiarse de la funcionalidad de restauración instantánea y la operación de restauración será más lenta en comparación. Si el propósito es restaurar discos administrados como discos no administrados, no proporcione este parámetro y haga que la intención sea clara proporcionando el parámetro `-RestoreAsUnmanagedDisks`. El parámetro `-RestoreAsUnmanagedDisks` está disponible en Azure PowerShell 3.7.0 y versiones posteriores. En versiones futuras, será obligatorio proporcionar cualquiera de estos parámetros para una experiencia de restauración correcta.
>
>

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID
```

El archivo **VMConfig.JSON** se restaurará en la cuenta de almacenamiento y los discos administrados en el grupo de recursos de destino especificado.

La salida es similar a la del ejemplo siguiente:

```output
WorkloadName     Operation          Status               StartTime                 EndTime            JobID
------------     ---------          ------               ---------                 -------          ----------
V2VM              Restore           InProgress           4/23/2016 5:00:30 PM                        cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

Use el cmdlet [Wait-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) para esperar a que se complete el trabajo de restauración.

```powershell
Wait-AzRecoveryServicesBackupJob -Job $restorejob -Timeout 43200
```

Una vez que se haya completado el trabajo de recuperación, use el cmdlet [Get-AzRecoveryServicesBackupJobDetails](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) para obtener los detalles de la operación de restauración. La propiedad JobDetails tiene la información necesaria para recompilar la máquina virtual.

```powershell
$restorejob = Get-AzRecoveryServicesBackupJob -Job $restorejob -VaultId $targetVault.ID
$details = Get-AzRecoveryServicesBackupJobDetails -Job $restorejob -VaultId $targetVault.ID
```

Una vez que restaure los discos, vaya a la siguiente sección para crear la máquina virtual.

## <a name="replace-disks-in-azure-vm"></a>Reemplazar discos en VM de Azure

Para reemplazar los discos y la información de configuración, realice los pasos siguientes:

* Paso 1: [Restaurar los discos](backup-azure-vms-automation.md#restore-the-disks)
* Paso 2: [Desasociar el disco de datos con PowerShell](../virtual-machines/windows/detach-disk.md#detach-a-data-disk-using-powershell)
* Paso 3: [Conectar un disco a una VM Windows con PowerShell](../virtual-machines/windows/attach-disk-ps.md)

## <a name="create-a-vm-from-restored-disks"></a>Creación de una máquina virtual a partir de discos restaurados

Tras haber restaurado los discos, siga estos pasos para crear y configurar la máquina virtual a partir del disco.

> [!NOTE]
>
> 1. Se necesita el módulo AzureAz 3.0.0 o una versión posterior. <br>
> 2. Para crear máquinas virtuales cifradas a partir de discos restaurados, el rol de Azure debe tener permiso para realizar la acción, **Microsoft.KeyVault/vaults/deploy/action**. Si su rol no tiene este permiso, cree un rol personalizado con esta acción. Para obtener más información, vea [Roles personalizados en RBAC de Azure](../role-based-access-control/custom-roles.md). <br>
> 3. Después de restaurar discos, ahora puede obtener una plantilla de implementación que puede utilizar directamente para crear una nueva máquina virtual. No necesita cmdlets de PowerShell diferentes para crear máquinas virtuales administradas o no administradas que están cifradas o sin cifrar.<br>
> <br>

### <a name="create-a-vm-using-the-deployment-template"></a>Creación de una máquina virtual mediante la plantilla de implementación

Los detalles del trabajo resultante ofrecen la plantilla de URI que se puede consultar e implementar.

```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $templateBlobURI = $properties["Template Blob Uri"]
```

La plantilla no es accesible directamente, ya que está en la cuenta de almacenamiento de un cliente y un contenedor concreto. Necesitamos la dirección URL completa (junto con un token de SAS temporal) para acceder a ella.

1. En primer lugar, extraiga el nombre de la plantilla de templateBlobURI. A continuación se menciona el formato. Puede usar la operación de división en PowerShell para extraer el nombre de la plantilla final de esta dirección URL.

    ```http
    https://<storageAccountName.blob.core.windows.net>/<containerName>/<templateName>
    ```

2. Después, se puede generar la dirección URL completa como se explica [aquí](../azure-resource-manager/templates/secure-template-with-sas-token.md?tabs=azure-powershell#provide-sas-token-during-deployment).

    ```powershell
    Set-AzCurrentStorageAccount -Name $storageAccountName -ResourceGroupName <StorageAccount RG name>
    $templateBlobFullURI = New-AzStorageBlobSASToken -Container $containerName -Blob <templateName> -Permission r -FullUri
    ```

3. Implemente la plantilla para crear una nueva máquina virtual como se explica [aquí](../azure-resource-manager/templates/deploy-powershell.md).

    ```powershell
    New-AzResourceGroupDeployment -Name ExampleDeployment ResourceGroupName ExampleResourceGroup -TemplateUri $templateBlobFullURI -storageAccountType Standard_GRS
    ```

### <a name="create-a-vm-using-the-config-file"></a>Creación de una máquina virtual mediante el archivo de configuración

En la sección siguiente se enumeran los pasos necesarios para crear una máquina virtual mediante el archivo "VMConfig".

> [!NOTE]
> Se recomienda usar la plantilla de implementación detallada antes para crear una máquina virtual. Esta sección (puntos 1 al 6) estará pronto en desuso.

1. Realice una consulta destinada a las propiedades de los discos restaurados para obtener los detalles del trabajo.

   ```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $configBlobName = $properties["Config Blob Name"]
   ```

2. Establezca el contexto de Azure Storage y restaure el archivo de configuración JSON.

   ```powershell
   Set-AzCurrentStorageAccount -Name $storageaccountname -ResourceGroupName "testvault"
   $destination_path = "C:\vmconfig.json"
   Get-AzStorageBlobContent -Container $containerName -Blob $configBlobName -Destination $destination_path
   $obj = ((Get-Content -Path $destination_path -Raw -Encoding Unicode)).TrimEnd([char]0x00) | ConvertFrom-Json
   ```

3. Utilice el archivo de configuración JSON para crear la configuración de la máquina virtual.

   ```powershell
   $vm = New-AzVMConfig -VMSize $obj.'properties.hardwareProfile'.vmSize -VMName "testrestore"
   ```

4. Conecte el disco del sistema operativo y los discos de datos. En este paso se proporcionan ejemplos de diversas configuraciones de VM administrada y cifrada. Use el ejemplo que se adapte a su configuración de VM.

    * **VM no administradas y no cifradas**: Use el ejemplo siguiente para VM no administradas y no cifradas.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
            $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Máquinas virtuales cifradas no administradas con Azure AD (solo mediante BEK)** : En el caso de las máquinas virtuales cifradas no administradas con Azure AD (cifradas solo mediante BEK), debe restaurar el secreto en el almacén de claves para poder asociar discos. Para más información, consulte [Restauración de una máquina virtual de Azure desde un punto de recuperación de Azure Backup](backup-azure-restore-key-secret.md). En el ejemplo siguiente se muestra cómo adjuntar discos del sistema operativo y de datos para máquinas virtuales cifradas. Al establecer el disco del sistema operativo, asegúrese de mencionar el tipo de sistema operativo pertinente.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $dekUrl = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows/Linux
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
        foreach($dd in $obj.'properties.storageProfile'.dataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Máquinas virtuales cifradas no administradas con Azure AD (BEK y KEK)** : En el caso de las máquinas virtuales cifradas no administradas con Azure AD (cifradas mediante BEK y KEK), restaure la clave y el secreto en el almacén de claves para poder asociar los discos. Para más información, consulte [Restauración de una máquina virtual de Azure desde un punto de recuperación de Azure Backup](backup-azure-restore-key-secret.md). En el ejemplo siguiente se muestra cómo adjuntar discos del sistema operativo y de datos para máquinas virtuales cifradas.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $kekUrl = "https://ContosoKeyVault.vault.azure.net:443/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
        foreach($dd in $obj.'properties.storageProfile'.dataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Máquinas virtuales cifradas no administradas sin Azure AD (solo mediante BEK)** : En el caso de las máquinas virtuales cifradas no administradas sin Azure AD (cifradas solo mediante BEK), si **no están disponibles los secretos de keyVault/secret** del origen, restaure los secretos al almacén de claves mediante el procedimiento descrito en [Restauración de una máquina virtual no cifrada desde un punto de recuperación de Azure Backup](backup-azure-restore-key-secret.md). A continuación, ejecute los siguientes scripts para establecer los detalles de cifrado en el blob del sistema operativo restaurado (este paso no es necesario para un blob de datos). $dekurl se puede recuperar del almacén de claves restaurado.

    El siguiente script solo se debe ejecutar cuando no están disponibles los secretos de keyVault/secret del origen.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""}}]}"
        $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
        $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
        $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
        $osBlob.ICloudBlob.SetMetadata()
    ```

    Una vez que **estén disponibles los secretos** y los detalles de cifrado también estén establecidos en el blob del sistema operativo, adjunte los discos mediante el script indicado a continuación.

    Si los secretos de keyVault/secret del origen ya están disponibles, no es necesario ejecutar el script anterior.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Máquinas virtuales cifradas no administradas sin Azure AD (BEK y KEK)** : En el caso de las máquinas virtuales cifradas no administradas sin Azure AD (cifradas mediante BEK y KEK), si **no están disponibles los secretos de keyVault/key/secret** del origen, restaure la clave y los secretos al almacén de claves mediante el procedimiento descrito en [Restauración de una máquina virtual no cifrada desde un punto de recuperación de Azure Backup](backup-azure-restore-key-secret.md). A continuación, ejecute los siguientes scripts para establecer los detalles de cifrado en el blob del sistema operativo restaurado (este paso no es necesario para un blob de datos). $dekurl y $kekurl se pueden recuperar del almacén de claves restaurado.

    El siguiente script solo se debe ejecutar cuando no están disponibles los secretos de keyVault/key/secret del origen.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""},""keyEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""keyUrl"":""$kekUrl""}}]}"
        $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
        $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
        $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
        $osBlob.ICloudBlob.SetMetadata()
    ```

    Una vez que **estén disponibles los secretos de key/secret** y los detalles de cifrado también estén establecidos en el blob del sistema operativo, adjunte los discos mediante el script indicado a continuación.

    Si los secretos de keyVault/key/secret del origen están disponibles, no es necesario ejecutar el script anterior.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Máquinas virtuales administradas y no cifradas**: en estas máquinas virtuales, asocie los discos administrados que ha restaurado. Para obtener información detallada, consulte [Conexión de un disco a una VM con Windows mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Máquinas virtuales administradas y cifradas con Azure AD (solo mediante BEK)** : en estas máquinas virtuales con Azure AD (solo cifradas con BEK), adjunte los discos administrados que ha restaurado. Para obtener información detallada, consulte [Conexión de un disco a una VM con Windows mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Máquinas virtuales administradas y cifradas con Azure AD (BEK y KEK)** : en estas máquinas virtuales con Azure AD (cifradas mediante BEK y KEK), adjunte los discos administrados que ha restaurado. Para obtener información detallada, consulte [Conexión de un disco a una VM con Windows mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Máquinas virtuales cifradas y administradas sin Azure AD (solo mediante BEK)** : en el caso de las máquinas virtuales cifradas y administradas sin Azure AD (cifradas solo mediante BEK), si **no están disponibles los secretos de keyVault/secret** del origen, restaure los secretos al almacén de claves mediante el procedimiento descrito en [Restauración de una máquina virtual no cifrada desde un punto de recuperación de Azure Backup](backup-azure-restore-key-secret.md). A continuación, ejecute los siguientes scripts para establecer los detalles de cifrado en el disco del sistema operativo restaurado (este paso no es necesario para un disco de datos). $dekurl se puede recuperar del almacén de claves restaurado.

    El siguiente script solo se debe ejecutar cuando no están disponibles los secretos de keyVault/secret del origen.  

    ```powershell
    $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
    $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
    $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
    $encryptionSettingsElement = New-Object Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement
    $encryptionSettingsElement.DiskEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.DiskEncryptionKey.SecretUrl = $dekUrl
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings = New-Object System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement]
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings.Add($encryptionSettingsElement)
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettingsVersion = "1.1"
    Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    Una vez que estén disponibles los secretos y los detalles de cifrado también estén establecidos en el disco del sistema operativo, para adjuntar los discos administrados restaurados, consulte [Conexión de un disco a una VM con Windows mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Máquinas virtuales cifradas y administradas sin Azure AD (BEK y KEK)** : en el caso de las máquinas virtuales cifradas y administradas sin Azure AD (cifradas mediante BEK y KEK), si **no están disponibles los secretos de keyVault/key/secret** del origen, restaure la clave y los secretos al almacén de claves mediante el procedimiento descrito en [Restauración de una máquina virtual no cifrada desde un punto de recuperación de Azure Backup](backup-azure-restore-key-secret.md). A continuación, ejecute los siguientes scripts para establecer los detalles de cifrado en el disco del sistema operativo restaurado (este paso no es necesario para discos de datos). $dekurl y $kekurl se pueden recuperar del almacén de claves restaurado.

    El siguiente script solo se debe ejecutar cuando no están disponibles los secretos de keyVault/key/secret del origen.

    ```powershell
    $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
    $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
    $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
    $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
    $encryptionSettingsElement = New-Object Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement
    $encryptionSettingsElement.DiskEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.DiskEncryptionKey.SecretUrl = $dekUrl
    $encryptionSettingsElement.KeyEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndKeyReference
    $encryptionSettingsElement.KeyEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.KeyEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.KeyEncryptionKey.KeyUrl = $kekUrl
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings = New-Object System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement]
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings.Add($encryptionSettingsElement)
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettingsVersion = "1.1"
    Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    Una vez que estén disponibles los secretos de key/secret y los detalles de cifrado también estén establecidos en el disco del sistema operativo, para adjuntar los discos administrados restaurados, consulte [Conexión de un disco a una VM con Windows mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md).

5. Ajuste la configuración de la red.

    ```powershell
    $nicName="p1234"
    $pip = New-AzPublicIpAddress -Name $nicName -ResourceGroupName "test" -Location "WestUS" -AllocationMethod Dynamic
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName "test" -Location "WestUS" -Name "testvNET" -AddressPrefix 10.0.0.0/16
    $virtualNetwork | Set-AzVirtualNetwork
    $vnet = Get-AzVirtualNetwork -Name "testvNET" -ResourceGroupName "test"
    $subnetindex=0
    $nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName "test" -Location "WestUS" -SubnetId $vnet.Subnets[$subnetindex].Id -PublicIpAddressId $pip.Id
    $vm=Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
    ```

6. Cree la máquina virtual.

    ```powershell  
    New-AzVM -ResourceGroupName "test" -Location "WestUS" -VM $vm
    ```

7. Inserte la extensión ADE.
   Si no se insertan las extensiones ADE, los discos de datos se marcarán como no cifrados, por lo que es obligatorio que se ejecuten los pasos siguientes:

   * **Para máquinas virtuales con Azure AD**: Use el comando siguiente para habilitar manualmente el cifrado de los discos de datos.  

     **Solo BEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

     **BEK y KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId  -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

   * **Para máquinas virtuales sin Azure AD**: Use el comando siguiente para habilitar manualmente el cifrado de los discos de datos.

     Si durante la ejecución del comando se solicitara AADClientID, debe actualizar Azure PowerShell.

     **Solo BEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

      **BEK y KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

> [!NOTE]
> Asegúrese de eliminar manualmente los archivos JASON creados como parte del proceso de disco de restauración de máquinas virtuales cifradas.

## <a name="restore-files-from-an-azure-vm-backup"></a>Restauración de archivos desde una copia de seguridad de la máquina virtual de Azure

Además de restaurar discos, también puede restaurar archivos individuales desde una copia de seguridad de la máquina virtual de Azure. La funcionalidad de restauración de archivos proporciona acceso a todos los archivos en un punto de recuperación. Administre los archivos a través del Explorador de archivos como lo haría con los archivos normales.

Los pasos básicos para restaurar un archivo desde una copia de seguridad de la VM de Azure son los siguientes:

* Selección de la máquina virtual
* Elección de un punto de recuperación
* Montaje de los discos del punto de recuperación
* Copia de los archivos necesarios
* Desmontaje de los discos

### <a name="select-the-vm-when-restoring-the-vm"></a>Selección de la máquina virtual (al restaurar la máquina virtual)

Para obtener el objeto de PowerShell que identifica el elemento de copia de seguridad correcto, comience en el contenedor del almacén y avance hacia abajo por la jerarquía de objetos. Para seleccionar el contenedor que representa la máquina virtual, use el cmdlet [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) y canalícelo al cmdlet [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

### <a name="choose-a-recovery-point-when-restoring-the-vm"></a>Elección de un punto de recuperación (al restaurar la máquina virtual)

Use el cmdlet [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) para enumerar todos los puntos de recuperación del elemento de copia de seguridad. Después, elija el punto de recuperación que se debe restaurar. Si no está seguro de qué punto de recuperación debe usar, se recomienda elegir el punto RecoveryPointType = AppConsistent más reciente de la lista.

En el siguiente script, la variable **$rp**es una matriz de puntos de recuperación para el elemento de copia de seguridad seleccionado de los últimos siete días. La matriz se ordena en orden inverso de tiempo con el punto de recuperación más reciente en el índice 0. Use la indexación de matrices de PowerShell estándar para seleccionar el punto de recuperación. En el ejemplo, $rp[0] selecciona el último punto de recuperación.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
$rp[0]
```

La salida es similar a la del ejemplo siguiente:

```output
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="mount-the-disks-of-recovery-point"></a>Montaje de los discos del punto de recuperación

Use el cmdlet [Get-AzRecoveryServicesBackupRPMountScript](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprpmountscript) para obtener el script para montar todos los discos del punto de recuperación.

> [!NOTE]
> Los discos se montan como discos iSCSI conectados a la máquina donde se ejecuta el script. El montaje se produce inmediatamente y no incurre en cargos.
>
>

```powershell
Get-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0] -VaultId $targetVault.ID
```

La salida es similar a la del ejemplo siguiente:

```output
OsType  Password        Filename
------  --------        --------
Windows e3632984e51f496 V2VM_wus2_8287309959960546283_451516692429_cbd6061f7fc543c489f1974d33659fed07a6e0c2e08740.exe
```

Ejecute el script en la máquina en la que desea recuperar los archivos. Para ejecutar el script, debe escribir la contraseña proporcionada. Después de que los discos se conecten, use el Explorador de archivos de Windows para navegar por los nuevos volúmenes y archivos. Para más información, consulte el artículo de Backup, [Recuperación de archivos desde una copia de seguridad de máquina virtual de Azure](backup-azure-restore-files-from-vm.md).

### <a name="unmount-the-disks"></a>Desmontaje de los discos

Una vez que se copien los archivos necesarios, desmonte los discos mediante el cmdlet [Disable-AzRecoveryServicesBackupRPMountScript](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackuprpmountscript). Asegúrese de desmontar los discos de modo que se quite el acceso a los archivos del punto de recuperación.

```powershell
Disable-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0] -VaultId $targetVault.ID
```

## <a name="next-steps"></a>Pasos siguientes

Si prefiere usar PowerShell para interactuar con los recursos de Azure, vea el artículo de PowerShell [Implementación y administración de copia de seguridad para Windows Server](backup-client-automation.md). Si administra copias de seguridad de DPM, vea el artículo [Implementación y administración de copias de seguridad para DPM](backup-dpm-automation.md).
