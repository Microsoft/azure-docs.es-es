---
title: Configuración de la recuperación ante desastres de Hyper-V (con VMM) en un sitio secundario con Azure Site Recovery o PowerShell
description: Describe cómo configurar la recuperación ante desastres de máquinas virtuales de Hyper-V en nubes VMM en un sitio VMM secundario con Azure Site Recovery y PowerShell.
services: site-recovery
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 1/10/2020
ms.author: sutalasi
ms.openlocfilehash: a4f7d330db9a4a0d9b435ebe7527e55e37c254e2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086224"
---
# <a name="set-up-disaster-recovery-of-hyper-v-vms-to-a-secondary-site-by-using-powershell-resource-manager"></a>Configuración de la recuperación ante desastres de máquinas virtuales de Hyper-V en un sitio secundario con PowerShell (Resource Manager)

En este artículo se explica cómo automatizar los pasos necesarios para la replicación de máquinas virtuales de Hyper-V en nubes de System Center Virtual Machine Manager a una nube de Virtual Machine Manager en un sitio local secundario con [Azure Site Recovery](site-recovery-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

- Revise la [arquitectura del escenario y sus componentes](hyper-v-vmm-architecture.md).
- Revise los [requisitos de compatibilidad](./vmware-physical-secondary-support-matrix.md) de todos los componentes.
- Asegúrese de que los servidores de Virtual Machine Manager y los hosts de Hyper-V cumplan los [requisitos de compatibilidad](./vmware-physical-secondary-support-matrix.md).
- Compruebe que las máquinas virtuales que desea replicar cumplen con la [compatibilidad con máquinas replicadas](./vmware-physical-secondary-support-matrix.md).

## <a name="prepare-for-network-mapping"></a>Preparar la asignación de red

La [asignación de red](hyper-v-vmm-network-mapping.md) funciona entre redes de máquinas virtuales de Virtual Machine Manager locales en nubes de origen y nubes de destino. La asignación hace lo siguiente:

- Conecta las máquinas virtuales a las redes de máquinas virtuales de destino después de la conmutación por error.
- Ubica de manera óptima las máquinas virtuales de réplica en servidores de host Hyper-V de destino.
- Si no configura la asignación de red, las máquinas virtuales de réplica no se conectarán a ninguna red de máquina virtual después de la conmutación por error.

Prepare Virtual Machine Manager como sigue:

- Asegúrese de que las [redes lógicas de Virtual Machine Manager](/system-center/vmm/network-logical) estén en los servidores de Virtual Machine Manager de origen y de destino:
  - La red lógica del servidor de origen se debe asociar con la nube de origen en la que se encuentran los hosts Hyper-V.
  - La red lógica del servidor de destino se debe asociar con la nube de destino.
- Asegúrese de que las [redes de VM](/system-center/vmm/network-virtual) estén en los servidores de Virtual Machine Manager de origen y de destino. Las redes de máquina virtual se deben vincular a la red lógica de cada ubicación.
- Conecte máquinas virtuales en los host Hyper-V de origen a la red de máquina virtual de origen.

## <a name="prepare-for-powershell"></a>Preparación para PowerShell

Asegúrese de que tiene Azure PowerShell listo para usar:

- Si ya usa PowerShell, actualice a la versión 0.8.10 o posterior. [Más información](/powershell/azure/) sobre cómo configurar PowerShell.
- Una vez haya instalado y configurado PowerShell, revise los [cmdlets del servicio](/powershell/azure/).
- Para obtener más información sobre cómo usar los valores de parámetros, las entradas y las salidas de PowerShell, lea la guía de [introducción](/powershell/azure/get-started-azureps).

## <a name="set-up-a-subscription"></a>Configuración de una suscripción

1. En PowerShell, inicie sesión en la cuenta de Azure.

   ```azurepowershell
   $UserName = "<user@live.com>"
   $Password = "<password>"
   $SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
   $Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
   Connect-AzAccount #-Credential $Cred
   ```

1. Recupere una lista de sus suscripciones con los identificadores de suscripción. Tome nota del identificador de la suscripción en la que desea crear el almacén de Recovery Services.

   ```azurepowershell
   Get-AzSubscription
   ```

1. Establezca la suscripción para el almacén.

   ```azurepowershell
   Set-AzContext –SubscriptionID <subscriptionId>
   ```

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

1. Cree un grupo de recursos de Azure Resource Manager si no dispone de uno.

   ```azurepowershell
   New-AzResourceGroup -Name #ResourceGroupName -Location #location
   ```

1. Cree un almacén de Recovery Services. Guarde el objeto de almacén en una variable para usarlo más adelante.

   ```azurepowershell
   $vault = New-AzRecoveryServicesVault -Name #vaultname -ResourceGroupName #ResourceGroupName -Location #location
   ```

   Después de crearlo, puede recuperar el objeto de almacén mediante el cmdlet `Get-AzRecoveryServicesVault`.

## <a name="set-the-vault-context"></a>Establecer el contexto de almacén

1. Recupere un almacén existente.

   ```azurepowershell
   $vault = Get-AzRecoveryServicesVault -Name #vaultname
   ```

1. Establezca el contexto de almacén.

   ```azurepowershell
   Set-AzRecoveryServicesAsrVaultContext -Vault $vault
   ```

## <a name="install-the-site-recovery-provider"></a>Instalación del proveedor de Site Recovery

1. En el equipo de Virtual Machine Manager, cree un directorio mediante la ejecución del comando siguiente:

   ```azurepowershell
   New-Item -Path C:\ASR -ItemType Directory
   ```

1. Extraiga los archivos mediante el archivo de instalación del proveedor descargado.

   ```console
   pushd C:\ASR\
   .\AzureSiteRecoveryProvider.exe /x:. /q
   ```

1. Instale el proveedor y espere a que finalice la instalación.

   ```console
   .\SetupDr.exe /i
   $installationRegPath = "HKLM:\Software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
   do
   {
     $isNotInstalled = $true;
     if(Test-Path $installationRegPath)
     {
       $isNotInstalled = $false;
     }
   }While($isNotInstalled)
   ```

1. Registre el servidor en el almacén.

   ```console
   $BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
   pushd $BinPath
   $encryptionFilePath = "C:\temp\".\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice
   ```

## <a name="create-and-associate-a-replication-policy"></a>Creación y asociación de una directiva de replicación

1. Cree una directiva de replicación (en este caso para Hyper-V 2012 R2) como se indica a continuación:

   ```azurepowershell
   $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
   $PolicyName = “replicapolicy”
   $RepProvider = HyperVReplica2012R2
   $Recoverypoints = 24                    #specify the number of hours to retain recovery points
   $AppConsistentSnapshotFrequency = 4 #specify the frequency (in hours) at which app consistent snapshots are taken
   $AuthMode = "Kerberos"  #options are "Kerberos" or "Certificate"
   $AuthPort = "8083"  #specify the port number that will be used for replication traffic on Hyper-V hosts
   $InitialRepMethod = "Online" #options are "Online" or "Offline"

   $policyresult = New-AzRecoveryServicesAsrPolicy -Name $policyname -ReplicationProvider $RepProvider -ReplicationFrequencyInSeconds $Replicationfrequencyinseconds -NumberOfRecoveryPointsToRetain $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours $AppConsistentSnapshotFrequency -Authentication $AuthMode -ReplicationPort $AuthPort -ReplicationMethod $InitialRepMethod
   ```

   > [!NOTE]
   > La nube de Virtual Machine Manager puede contener hosts de Hyper-V que ejecuten versiones distintas de Windows Server, pero la directiva de replicación es para una versión específica de un sistema operativo. Si tiene diferentes hosts que ejecutan diferentes sistemas operativos, cree directivas de replicación independientes para cada sistema. Por ejemplo, si tiene cinco hosts que ejecutan Windows Server 2012 y tres que ejecutan Windows Server 2012 R2, cree dos directivas de replicación; una para cada tipo de sistema operativo.

1. Obtenga el contenedor de protección principal (nube de Virtual Machine Manager principal) y el de recuperación (nube de Virtual Machine Manager de recuperación).

   ```azurepowershell
   $PrimaryCloud = "testprimarycloud"
   $primaryprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $PrimaryCloud;

   $RecoveryCloud = "testrecoverycloud"
   $recoveryprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $RecoveryCloud;
   ```

1. Recupere la directiva de replicación que creó, utilizando el nombre descriptivo.

   ```azurepowershell
   $policy = Get-AzRecoveryServicesAsrPolicy -FriendlyName $policyname
   ```

1. Inicie la asociación del contenedor de protección (nube de Virtual Machine Manager) con la directiva de replicación.

   ```azurepowershell
   $associationJob  = New-AzRecoveryServicesAsrProtectionContainerMapping -Policy $Policy -PrimaryProtectionContainer $primaryprotectionContainer -RecoveryProtectionContainer $recoveryprotectionContainer
   ```

1. Espere a que finalice el trabajo de asociación de directivas. Para comprobar si el trabajo ha terminado, use el siguiente fragmento de código de PowerShell:

   ```azurepowershell
   $job = Get-AzRecoveryServicesAsrJob -Job $associationJob

   if($job -eq $null -or $job.StateDescription -ne "Completed")
   {
     $isJobLeftForProcessing = $true;
   }
   ```

1. Cuando haya finalizado el procesamiento, ejecute el siguiente comando:

   ```azurepowershell
   if($isJobLeftForProcessing)
   {
     Start-Sleep -Seconds 60
   }
   While($isJobLeftForProcessing)
   ```

Para comprobar la finalización de la operación, siga los pasos descritos en [Supervisión de la actividad](#monitor-activity).

##  <a name="configure-network-mapping"></a>Configurar asignación de red

1. Utilice este comando para recuperar los servidores del almacén actual. El comando almacena los servidores de Site Recovery en la variable de matriz `$Servers`.

   ```azurepowershell
   $Servers = Get-AzRecoveryServicesAsrFabric
   ```

1. Ejecute este comando para recuperar las redes para los servidores de origen y destino de Virtual Machine Manager.

   ```azurepowershell
   $PrimaryNetworks = Get-AzRecoveryServicesAsrNetwork -Fabric $Servers[0]

   $RecoveryNetworks = Get-AzRecoveryServicesAsrNetwork -Fabric $Servers[1]
   ```

   > [!NOTE]
   > El servidor de Virtual Machine Manager de origen puede ser el primero o el segundo en la matriz de servidores. Compruebe los nombres de servidor de Virtual Machine Manager y recupere las redes correspondientes.

1. Este cmdlet crea una asignación entre la red principal y la de recuperación. Especifica la red principal como primer elemento de `$PrimaryNetworks`. Especifica la red de recuperación como primer elemento de `$RecoveryNetworks`.

   ```azurepowershell
   New-AzRecoveryServicesAsrNetworkMapping -PrimaryNetwork $PrimaryNetworks[0] -RecoveryNetwork $RecoveryNetworks[0]
   ```

## <a name="enable-protection-for-vms"></a>Habilitación de la protección de las máquinas virtuales

Una vez configurados correctamente los servidores, las nubes y las redes, habilite la protección de las máquinas virtuales en la nube.

1. Para habilitar la protección, ejecute el siguiente comando para recuperar el contenedor de protección:

   ```azurepowershell
   $PrimaryProtectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $PrimaryCloudName
   ```

1. Obtenga la entidad de protección (máquina virtual) de la siguiente manera:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -FriendlyName $VMName -ProtectionContainer $PrimaryProtectionContainer
   ```

1. Habilite la replicación de la máquina virtual.

   ```azurepowershell
   $jobResult = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $protectionentity -ProtectionContainerMapping $policy -VmmToVmm
   ```

> [!NOTE]
> Si quiere realizar la replicación en discos administrados habilitados para CMK en Azure, siga estos pasos con Az PowerShell 3.3.0 en adelante:
>
> 1. Habilitar la conmutación por error a discos administrados mediante la actualización de las propiedades de máquina virtual
> 1. Usar el cmdlet `Get-AzRecoveryServicesAsrReplicationProtectedItem` para capturar el identificador de disco de cada disco del elemento protegido.
> 1. Crear un objeto de diccionario con el cmdlet `New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"` para que contenga la asignación de un identificador de disco a un conjunto de cifrado de disco. Estos conjuntos de cifrado de disco se crearán previamente de forma automática en la región de destino.
> 1. Actualizar las propiedades de la máquina virtual mediante el cmdlet `Set-AzRecoveryServicesAsrReplicationProtectedItem` al pasar el objeto de diccionario en el parámetro **DiskIdToDiskEncryptionSetMap**.

## <a name="run-a-test-failover"></a>Ejecución de una conmutación por error de prueba

Para probar la implementación, ejecute una conmutación por error de prueba para una única máquina virtual. También puede crear un plan de recuperación que contenga varias máquinas virtuales y ejecutar una conmutación por error de prueba del plan. La conmutación por error de prueba simula su mecanismo de conmutación por error y recuperación en una red aislada.

1. Recupere la máquina virtual en la que conmutarán por error las máquinas virtuales.

   ```azurepowershell
   $Servers = Get-AzRecoveryServicesASRFabric
   $RecoveryNetworks = Get-AzRecoveryServicesAsrNetwork -Name $Servers[1]
   ```

1. Realización de una conmutación por error de prueba.

   Para una sola máquina virtual:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -FriendlyName $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrTestFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity -VMNetwork $RecoveryNetworks[1]
   ```

   Para un plan de recuperación:

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrTestFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan -VMNetwork $RecoveryNetworks[1]
   ```

Para comprobar la finalización de la operación, siga los pasos descritos en [Supervisión de la actividad](#monitor-activity).

## <a name="run-planned-and-unplanned-failovers"></a>Ejecución de conmutaciones por error planeadas y no planeadas

1. Realización de una conmutación por error planeada.

   Para una sola máquina virtual:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrPlannedFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity
   ```

   Para un plan de recuperación:

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrPlannedFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan
   ```

1. Realización de una conmutación por error no planeada.

   Para una sola máquina virtual:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrUnplannedFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity
   ```

   Para un plan de recuperación:

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrUnplannedFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan
   ```

## <a name="monitor-activity"></a>Supervisión de la actividad

Utilice los comandos siguientes para supervisar la actividad de conmutación por error. Espere a que el procesamiento finalice entre los trabajos.

```azurepowershell
Do
{
    $job = Get-AzRecoveryServicesAsrJob -TargetObjectId $associationJob.JobId;
    Write-Host "Job State:{0}, StateDescription:{1}" -f Job.State, $job.StateDescription;
    if($job -eq $null -or $job.StateDescription -ne "Completed")
    {
        $isJobLeftForProcessing = $true;
    }

if($isJobLeftForProcessing)
    {
        Start-Sleep -Seconds 60
    }
}While($isJobLeftForProcessing)
```

## <a name="next-steps"></a>Pasos siguientes

[Más información](/powershell/module/az.recoveryservices) sobre Site Recovery con los cmdlets de PowerShell de Resource Manager.
