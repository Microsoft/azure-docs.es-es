---
title: Recuperación ante desastres en máquinas virtuales de Azure con Azure PowerShell y Azure Site Recovery
description: Aprenda a configurar la recuperación ante desastres en máquinas virtuales de Azure con Azure Site Recovery mediante Azure PowerShell.
services: site-recovery
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 3/29/2019
ms.author: sutalasi
ms.openlocfilehash: 182b4f262361db001dcb6d47bf3e8f2aac6bc9b3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091528"
---
# <a name="set-up-disaster-recovery-for-azure-virtual-machines-using-azure-powershell"></a>Configuración de la recuperación ante desastres en máquinas virtuales de Azure mediante Azure PowerShell

En este artículo, aprenderá a configurar y probar la recuperación ante desastres en máquinas virtuales de Azure mediante Azure PowerShell.

Aprenderá a:

> [!div class="checklist"]
> - Cree un almacén de Recovery Services.
> - Establecer el contexto de almacén de la sesión de PowerShell
> - Preparar el almacén para iniciar la replicación de máquinas virtuales de Azure
> - Crear asignaciones de red
> - Cree cuentas de almacenamiento en las que replicar máquinas virtuales.
> - Replicar máquinas virtuales de Azure en una región de recuperación para la recuperación ante desastres
> - Realice una conmutación por error de prueba, una validación y conmutación por error de prueba de limpieza.
> - Realice la conmutación por error a la región de recuperación.

> [!NOTE]
> Puede que no todas las funcionalidades del escenario disponibles a través del portal podrían estén disponibles a través de Azure PowerShell. Algunas de las funcionalidades del escenario que no se admiten actualmente a través de Azure PowerShell son:
> - La capacidad de especificar que todos los discos de una máquina virtual deben replicarse sin tener que especificar explícitamente cada disco de la máquina virtual.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar:
- Asegúrese de entender la [arquitectura y los componentes del escenario](azure-to-azure-architecture.md).
- Revise los [requisitos de compatibilidad](azure-to-azure-support-matrix.md) de todos los componentes.
- Tiene el módulo `Az` de Azure PowerShell. Si necesita instalar o actualizar Azure PowerShell, siga la guía [Cómo instalar y configurar Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Inicio de sesión en la suscripción de Microsoft Azure

Inicie sesión en su suscripción de Azure con el cmdlet `Connect-AzAccount`.

```azurepowershell
Connect-AzAccount
```

Seleccione su suscripción a Azure. Use el cmdlet `Get-AzSubscription` para obtener la lista de suscripciones de Azure a las que tiene acceso. Seleccione la suscripción de Azure con la que quiera trabajar mediante el cmdlet `Set-AzContext`.

```azurepowershell
Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

## <a name="get-details-of-the-virtual-machine-to-be-replicated"></a>Obtención de los detalles de la máquina virtual que se va a replicar

En este artículo, una máquina virtual de la región Este de EE. UU. se replicará y recuperará en la región Oeste de EE. UU. 2. La máquina virtual que se replica es un disco del sistema operativo y un único disco de datos. El nombre de la máquina virtual usada en el ejemplo es `AzureDemoVM`.

```azurepowershell
# Get details of the virtual machine
$VM = Get-AzVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

Write-Output $VM
```

```Output
ResourceGroupName  : A2AdemoRG
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
Name               : AzureDemoVM
Type               : Microsoft.Compute/virtualMachines
Location           : eastus
Tags               : {}
DiagnosticsProfile : {BootDiagnostics}
HardwareProfile    : {VmSize}
NetworkProfile     : {NetworkInterfaces}
OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState  : Succeeded
StorageProfile     : {ImageReference, OsDisk, DataDisks}
```

Obtenga los detalles de los discos de la máquina virtual. Estos detalles se usarán posteriormente al iniciar la replicación de la máquina virtual.

```azurepowershell
$OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
$DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
```

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Cree el grupo de recursos en el que se creará el almacén de Recovery Services.

> [!IMPORTANT]
> * El almacén de Recovery Services, y las máquinas virtuales que se van a proteger, deben estar en diferentes ubicaciones de Azure.
> * El grupo de recursos del almacén de Recovery Services, y las máquinas virtuales que se van a proteger, deben estar en diferentes ubicaciones de Azure.
> * El almacén de Recovery Services, y el grupo de recursos al que pertenece, pueden estar en la misma ubicación de Azure.

En el ejemplo de este artículo, la máquina virtual que se protege se encuentra en la región Este de EE. UU. La región de recuperación seleccionada para la recuperación ante desastres es la región Oeste de EE. UU. 2. El almacén de Recovery Services y el grupo de recursos del almacén están en la región de recuperación, Oeste de EE. UU. 2.

```azurepowershell
#Create a resource group for the recovery services vault in the recovery Azure region
New-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"
```

```Output
ResourceGroupName : a2ademorecoveryrg
Location          : westus2
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg
```

Cree un almacén de Recovery Services. En este ejemplo, se crea un almacén de Recovery Services denominado `a2aDemoRecoveryVault` en la región Oeste de EE. UU. 2.

```azurepowershell
#Create a new Recovery services vault in the recovery region
$vault = New-AzRecoveryServicesVault -Name "a2aDemoRecoveryVault" -ResourceGroupName "a2ademorecoveryrg" -Location "West US 2"

Write-Output $vault
```

```Output
Name              : a2aDemoRecoveryVault
ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoRecoveryVault
Type              : Microsoft.RecoveryServices/vaults
Location          : westus2
ResourceGroupName : a2ademorecoveryrg
SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

## <a name="set-the-vault-context"></a>Establecer el contexto de almacén

Establezca el contexto de almacén que se usará en la sesión de PowerShell. Después de que se establece el contexto del almacén, las operaciones de Azure Site Recovery en la sesión de PowerShell se realizan en el contexto de almacén seleccionado.

```azurepowershell
#Setting the vault context.
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
```

```Output
ResourceName         ResourceGroupName ResourceNamespace          ResourceType
------------         ----------------- -----------------          -----------
a2aDemoRecoveryVault a2ademorecoveryrg Microsoft.RecoveryServices Vaults
```

```azurepowershell
#Delete the downloaded vault settings file
Remove-Item -Path $Vaultsettingsfile.FilePath
```

En el caso de una migración de Azure a Azure, puede establecer el contexto del almacén en el almacén recién creado:

```azurepowershell
#Set the vault context for the PowerShell session.
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
```

## <a name="prepare-the-vault-to-start-replicating-azure-virtual-machines"></a>Preparación del almacén para iniciar la replicación de máquinas virtuales de Azure

### <a name="create-a-site-recovery-fabric-object-to-represent-the-primary-source-region"></a>Creación de un objeto de tejido de Site Recovery para representar la región primaria (origen)

El objeto de tejido en el almacén representa una región de Azure. El objeto de tejido principal se crea para representar la región de Azure a la que pertenecen las máquinas virtuales que se van a proteger. En el ejemplo de este artículo, la máquina virtual que se protege se encuentra en la región Este de EE. UU.

- Solamente se puede crear un objeto de tejido por región.
- Si anteriormente ha habilitado la replicación de Site Recovery para una máquina virtual en Azure Portal, Site Recovery crea automáticamente un objeto de tejido. Si existe un objeto de tejido para una región, no puede crear uno nuevo.

Antes de empezar, tenga claro que las operaciones de Azure Site Recovery se ejecutan de manera asincrónica. Cuando se inicia una operación, se envía un trabajo de Azure Site Recovery y se devuelve un objeto de seguimiento de trabajo. Use el objeto de seguimiento de trabajos para obtener el estado más reciente del trabajo (`Get-AzRecoveryServicesAsrJob`) y para supervisar el estado de la operación.

```azurepowershell
#Create Primary ASR fabric
$TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'East US'  -Name "A2Ademo-EastUS"

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$PrimaryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-EastUS"
```

Si las máquinas virtuales procedentes de varias regiones de Azure se van a proteger en el mismo almacén, cree un objeto de tejido para cada región de Azure de origen.

### <a name="create-a-site-recovery-fabric-object-to-represent-the-recovery-region"></a>Creación de un objeto de tejido de Site Recovery para representar la región de recuperación

El objeto de tejido de recuperación representa la ubicación de Azure de recuperación. Si se produce una conmutación por error, las máquinas virtuales se replicarán y recuperarán en la región de recuperación representada por el tejido de recuperación. La región de Azure de recuperación usada en este ejemplo es Oeste de EE. UU. 2

```azurepowershell
#Create Recovery ASR fabric
$TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'West US 2'  -Name "A2Ademo-WestUS"

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$RecoveryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-WestUS"
```

### <a name="create-a-site-recovery-protection-container-in-the-primary-fabric"></a>Creación de un contenedor de protección de Site Recovery en el tejido principal

El contenedor de protección es un contenedor que se usa para agrupar los elementos replicados en un tejido.

```azurepowershell
#Create a Protection container in the primary Azure region (within the Primary fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $PrimaryFabric -Name "A2AEastUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

Write-Output $TempASRJob.State

$PrimaryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $PrimaryFabric -Name "A2AEastUSProtectionContainer"
```

### <a name="create-a-site-recovery-protection-container-in-the-recovery-fabric"></a>Creación de un contenedor de protección de Site Recovery en el tejido de recuperación

```azurepowershell
#Create a Protection container in the recovery Azure region (within the Recovery fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $RecoveryFabric -Name "A2AWestUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"

Write-Output $TempASRJob.State

$RecoveryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $RecoveryFabric -Name "A2AWestUSProtectionContainer"
```

### <a name="create-a-replication-policy"></a>Creación de una directiva de replicación

```azurepowershell
#Create replication policy
$TempASRJob = New-AzRecoveryServicesAsrPolicy -AzureToAzure -Name "A2APolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$ReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "A2APolicy"
```

### <a name="create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container"></a>Creación de una asignación de contenedor de protección entre el contenedor de protección principal y el de recuperación

Una asignación de contenedor de protección asigna el contenedor de protección principal con un contenedor de protección de recuperación y una directiva de replicación. Cree una asignación por cada directiva de replicación que vaya a usar para replicar máquinas virtuales entre un par de contenedores de protección.

```azurepowershell
#Create Protection container mapping between the Primary and Recovery Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2APrimaryToRecovery" -Policy $ReplicationPolicy -PrimaryProtectionContainer $PrimaryProtContainer -RecoveryProtectionContainer $RecoveryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$EusToWusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $PrimaryProtContainer -Name "A2APrimaryToRecovery"
```

### <a name="create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover"></a>Creación de una asignación de contenedor de protección para la conmutación por recuperación (replicación inversa después de una conmutación por error)

Después de una conmutación por error, cuando esté listo para devolver la máquina virtual conmutada por error a la región de Azure original, realice una conmutación por recuperación. Para ello, la máquina virtual conmutada por error se replica a la inversa desde la región de conmutación por error a la región original. En la replicación inversa, los roles de la región original y la región de recuperación se intercambian. La región original se convierte ahora en la nueva región de recuperación y la que originalmente era la región de recuperación se convierte ahora en la región primaria. La asignación de contenedor de protección para la replicación inversa representa los roles intercambiados de las regiones original y de recuperación.

```azurepowershell
#Create Protection container mapping (for fail back) between the Recovery and Primary Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2ARecoveryToPrimary" -Policy $ReplicationPolicy -PrimaryProtectionContainer $RecoveryProtContainer -RecoveryProtectionContainer $PrimaryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$WusToEusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $RecoveryProtContainer -Name "A2ARecoveryToPrimary"
```

## <a name="create-cache-storage-account-and-target-storage-account"></a>Creación de una cuenta de almacenamiento en caché y una cuenta de almacenamiento de destino

Una cuenta de almacenamiento en caché es una cuenta de almacenamiento estándar en la misma región de Azure que la máquina virtual que se replica. La cuenta de almacenamiento en caché se usa para almacenar temporalmente los cambios de replicación antes de que estos se muevan a la región de Azure de recuperación. Puede elegir, aunque no es necesario, especificar diferentes cuentas de almacenamiento en caché para los distintos discos de una máquina virtual.

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$EastUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestorage" -ResourceGroupName "A2AdemoRG" -Location 'East US' -SkuName Standard_LRS -Kind Storage
```

En máquinas virtuales **que no usan discos administrados**, la cuenta de almacenamiento de destino es la cuenta de almacenamiento de la región de recuperación en la que se replican los discos de la máquina virtual. La cuenta de almacenamiento de destino puede ser una cuenta de almacenamiento estándar o una cuenta de almacenamiento premium. Seleccione el tipo de cuenta de almacenamiento necesario según la velocidad de cambio de los datos (velocidad de escritura de E/S) de los discos y los límites de renovación admitidos de Azure Site Recovery para el tipo de almacenamiento.

```azurepowershell
#Create Target storage account in the recovery region. In this case a Standard Storage account
$WestUSTargetStorageAccount = New-AzStorageAccount -Name "a2atargetstorage" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -SkuName Standard_LRS -Kind Storage
```

## <a name="create-network-mappings"></a>Creación de asignaciones de red

Una asignación de red asigna redes virtuales de la región primaria a las redes virtuales de la región de recuperación. La asignación de red especifica la red virtual de Azure en la región de recuperación a la que debe conmutar por error una máquina virtual de la red virtual principal. Una red virtual de Azure se puede asignar a una sola red virtual de Azure en una región de recuperación.

- Cree una red virtual de Azure en la región de recuperación a la que se conmutará por error:

   ```azurepowershell
    #Create a Recovery Network in the recovery region
    $WestUSRecoveryVnet = New-AzVirtualNetwork -Name "a2arecoveryvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.0.0.0/16"

    Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $WestUSRecoveryVnet -AddressPrefix "10.0.0.0/20" | Set-AzVirtualNetwork

    $WestUSRecoveryNetwork = $WestUSRecoveryVnet.Id
   ```

- Recupere la red virtual principal. La red virtual a la que está conectada la máquina virtual:

   ```azurepowershell
    #Retrieve the virtual network that the virtual machine is connected to

    #Get first network interface card(nic) of the virtual machine
    $SplitNicArmId = $VM.NetworkProfile.NetworkInterfaces[0].Id.split("/")

    #Extract resource group name from the ResourceId of the nic
    $NICRG = $SplitNicArmId[4]

    #Extract resource name from the ResourceId of the nic
    $NICname = $SplitNicArmId[-1]

    #Get network interface details using the extracted resource group name and resource name
    $NIC = Get-AzNetworkInterface -ResourceGroupName $NICRG -Name $NICname

    #Get the subnet ID of the subnet that the nic is connected to
    $PrimarySubnet = $NIC.IpConfigurations[0].Subnet

    # Extract the resource ID of the Azure virtual network the nic is connected to from the subnet ID
    $EastUSPrimaryNetwork = (Split-Path(Split-Path($PrimarySubnet.Id))).Replace("\","/")
   ```

- Cree la asignación de red entre la red virtual principal y la red virtual de recuperación:

   ```azurepowershell
    #Create an ASR network mapping between the primary Azure virtual network and the recovery Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AEusToWusNWMapping" -PrimaryFabric $PrimaryFabric -PrimaryAzureNetworkId $EastUSPrimaryNetwork -RecoveryFabric $RecoveryFabric -RecoveryAzureNetworkId $WestUSRecoveryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
   ```

- Cree la asignación de red para la dirección inversa (conmutación por recuperación):

    ```azurepowershell
    #Create an ASR network mapping for fail back between the recovery Azure virtual network and the primary Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AWusToEusNWMapping" -PrimaryFabric $RecoveryFabric -PrimaryAzureNetworkId $WestUSRecoveryNetwork -RecoveryFabric $PrimaryFabric -RecoveryAzureNetworkId $EastUSPrimaryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
    ```

## <a name="replicate-azure-virtual-machine"></a>Replicación de máquinas virtuales de Azure

Replique la máquina virtual de Azure con **discos administrados**.

```azurepowershell
#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)

#OsDisk
$OSdiskId = $vm.StorageProfile.OsDisk.ManagedDisk.Id
$RecoveryOSDiskAccountType = $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType
$RecoveryReplicaDiskAccountType = $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType

$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $OSdiskId -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

# Data disk
$datadiskId1 = $vm.StorageProfile.DataDisks[0].ManagedDisk.Id
$RecoveryReplicaDiskAccountType = $vm.StorageProfile.DataDisks[0].ManagedDisk.StorageAccountType
$RecoveryTargetDiskAccountType = $vm.StorageProfile.DataDisks[0].ManagedDisk.StorageAccountType

$DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $datadiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.
$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId
```

Replique la máquina virtual de Azure con **discos no administrados**.

```azurepowershell
#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)

#Disk replication configuration for the OS disk
$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -VhdUri $OSDiskVhdURI.Uri -LogStorageAccountId $EastUSCacheStorageAccount.Id -RecoveryAzureStorageAccountId $WestUSTargetStorageAccount.Id

#Disk replication configuration for data disk
$DataDisk1ReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -VhdUri $DataDisk1VhdURI.Uri -LogStorageAccountId $EastUSCacheStorageAccount.Id -RecoveryAzureStorageAccountId $WestUSTargetStorageAccount.Id

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.
$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State
```

Una vez que la operación de replicación inicial se realiza correctamente, se replican los datos de la máquina virtual en la región de recuperación.

El proceso de replicación comienza con la propagación inicial de una copia de los discos de replicación de la máquina virtual en la región de recuperación. Esta fase se conoce como fase de replicación inicial.

Después de completada la replicación inicial, la replicación pasa a la fase de sincronización diferencial. En este momento, la máquina virtual está protegida y puede realizarse en ella una operación de conmutación por error de prueba. El estado de replicación del elemento replicado que representa la máquina virtual pasa al estado **Protegido** una vez finalizada la replicación inicial.

Para supervisar el estado y el mantenimiento de la replicación en la máquina virtual, puede obtener los detalles correspondientes al elemento protegido de la replicación.

```azurepowershell
Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```

```
FriendlyName ProtectionState ReplicationHealth
------------ --------------- -----------------
AzureDemoVM  Protected       Normal
```

## <a name="do-a-test-failover-validate-and-cleanup-test-failover"></a>Realización de una conmutación por error de prueba, una validación y conmutación por error de prueba de limpieza

Después de que la replicación de la máquina virtual ha alcanzado un estado protegido, se puede realizar una operación de conmutación por error de prueba en la máquina virtual (en el elemento protegido de replicación de la máquina virtual).

```azurepowershell
#Create a separate network for test failover (not connected to my DR network)
$TFOVnet = New-AzVirtualNetwork -Name "a2aTFOvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.3.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $TFOVnet -AddressPrefix "10.3.0.0/20" | Set-AzVirtualNetwork

$TFONetwork= $TFOVnet.Id
```

Ejecute una conmutación por error de prueba.

```azurepowershell
$ReplicationProtectedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem -FriendlyName "AzureDemoVM" -ProtectionContainer $PrimaryProtContainer

$TFOJob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -AzureVMNetworkId $TFONetwork -Direction PrimaryToRecovery
```

Espere a que finalice la operación de conmutación por error de prueba.

```azurepowershell
Get-AzRecoveryServicesAsrJob -Job $TFOJob
```

```Output
Name             : 3dcb043e-3c6d-4e0e-a42e-8d4245668547
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/3dcb043e-3c6d-4e0e-a42e-8d4245668547
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : TestFailover
DisplayName      : Test failover
ClientRequestId  : 1ef8515b-b130-4452-a44d-91aaf071931c ActivityId: 907bb2bc-ebe6-4732-8b66-77d0546eaba8
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:29:43 AM
EndTime          : 4/25/2018 4:33:06 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisites check for test failover, Create test virtual machine, Preparing the virtual machine, Start the virtual machine}
Errors           : {}
```

Después de que el trabajo de conmutación por error de prueba se completa correctamente, puede conectarse a la máquina virtual conmutada por error de prueba y validar esta operación.

Después de que la prueba ha finalizado en la máquina virtual conmutada por error de prueba, limpie la copia de prueba mediante una operación de conmutación por error de prueba de limpieza. Esta operación elimina la copia de prueba de la máquina virtual que se creó con la conmutación por error de prueba.

```azurepowershell
$Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $Job_TFOCleanup | Select State
```

```Output
State
-----
Succeeded
```

## <a name="fail-over-to-azure"></a>Conmutación por error a Azure

Conmute por error la máquina virtual a un punto de recuperación concreto.

```azurepowershell
$RecoveryPoints = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $ReplicationProtectedItem

#The list of recovery points returned may not be sorted chronologically and will need to be sorted first, in order to be able to find the oldest or the latest recovery points for the virtual machine.
"{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[-1].RecoveryPointTime
```

```Output
CrashConsistent 4/24/2018 11:10:25 PM
```

```azurepowershell
#Start the fail over job
$Job_Failover = Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[-1]

do {
        $Job_Failover = Get-AzRecoveryServicesAsrJob -Job $Job_Failover;
        sleep 30;
} while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))

$Job_Failover.State
```

```Output
Succeeded
```

Cuando se complete correctamente el trabajo de conmutación por error, puede confirmar la operación de conmutación por error.

```azurepowershell
$CommitFailoverJOb = Start-AzRecoveryServicesAsrCommitFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $CommitFailoverJOb
```

```Output
Name             : 58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : CommitFailover
DisplayName      : Commit
ClientRequestId  : 10a95d6c-359e-4603-b7d9-b7ee3317ce94 ActivityId: 8751ada4-fc42-4238-8de6-a82618408fcf
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:50:58 AM
EndTime          : 4/25/2018 4:51:01 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisite check, Commit}
Errors           : {}
```

## <a name="reprotect-and-fail-back-to-the-source-region"></a>Reprotección y conmutación por recuperación en la región de origen

Después de una conmutación por error, cuando esté listo para volver a la región original, inicie la replicación inversa del elemento protegido de replicación mediante el cmdlet `Update-AzRecoveryServicesAsrProtectionDirection`.

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage
```

```azurepowershell
#Use the recovery protection container, new cache storage account in West US and the source region VM resource group
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure
-ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId
```

Después de que se complete la reprotección, puede conmutar por error en dirección inversa, de Oeste de EE. UU. a Este de EE. UU., y conmutar por recuperación en la región de origen.

## <a name="disable-replication"></a>Deshabilitar replicación

Puede deshabilitar la replicación con el cmdlet `Remove-AzRecoveryServicesAsrReplicationProtectedItem`.

```azurepowershell
Remove-AzRecoveryServicesAsrReplicationProtectedItem -ReplicationProtectedItem $ReplicatedItem
```

## <a name="next-steps"></a>Pasos siguientes

Consulte la [referencia de PowerShell de Azure Site Recovery](/powershell/module/az.RecoveryServices) para obtener información sobre cómo realizar otras tareas, como la creación de planes de recuperación y la prueba de la conmutación por error de los planes de recuperación con PowerShell.
