---
title: Eliminación de servidores y deshabilitación de la protección | Microsoft Docs
description: En este artículo se describe cómo anular el registro de servidores desde un almacén de Site Recovery y deshabilitar la protección para máquinas virtuales y servidores físicos.
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 06/18/2019
ms.author: sharrai
ms.openlocfilehash: a4f6c318a7521e1fbc03ff3a47e34e992cce44df
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89424793"
---
# <a name="remove-servers-and-disable-protection"></a>Quitar servidores y deshabilitar la protección

Este artículo describe cómo anular el registro de servidores desde un almacén de Recovery Services y cómo deshabilitar la protección para las máquinas protegidas por Site Recovery.


## <a name="unregister-a--configuration-server"></a>Anulación del registro de un servidor de configuración

Si replica máquinas virtuales de VMware o servidores físicos de Windows/Linux en Azure, puede anular el registro de un servidor de configuración no conectado desde un almacén de la forma siguiente:

1. [Deshabilite la protección para las máquinas virtuales](#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure).
2. [Desasocie o elimine](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) las directivas de replicación.
3. [Elimine el servidor de configuración](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server).

## <a name="unregister-a-vmm-server"></a>Anulación del registro de un servidor VMM

1. Detenga la replicación de máquinas virtuales en nubes en el servidor VMM que desea quitar.
2. Elimine cualquier asignación de red utilizada por las nubes en el servidor VMM que desea eliminar. En **Site Recovery Infrastructure** > **For System Center VMM** > **Network Mapping** (Infraestructura de Site Recovery > Para System Center VMM > Asignación de red), haga clic con el botón derecho en la asignación de red > **Eliminar**.
3. Escriba el ID del servidor VMM.
4. Disocie las directivas de replicación de las nubes en el servidor VMM que desea quitar.  En **Site Recovery Infrastructure** > **For System Center VMM** >  **Replication Policies**, (Infraestructura de Site Recovery > Para System Center VMM > Directivas de replicación) haga doble clic en la directiva asociada. Haga clic con el botón derecho en la nube > **Disassociate** (Desasociar).
5. Elimine el servidor VMM o el nodo activo. En **Site Recovery Infrastructure** > **For System Center VMM** > **VMM Servers** (Infraestructura de Site Recovery > Para System Center VMM > Servidores VMM) haga clic con el botón derecho en el servidor > **Eliminar**.
6. Si el servidor de VMM estaba en estado desconectado, descargue y ejecute el [script de limpieza](https://aka.ms/asr-cleanup-script-vmm) en el servidor VMM. Abra PowerShell con la opción **Ejecutar como administrador** para cambiar la directiva de ejecución para el ámbito predeterminado (LocalMachine). En el script, especifique el ID del servidor VMM que desea quitar. El script quita el registro y la información de emparejamiento de nube del servidor.
5. Ejecute el script de limpieza en cualquier servidor VMM secundario.
6. Ejecute el script de limpieza en tos los demás nodos de clúster VMM pasivos que tengan instalado el proveedor.
7. Desinstale el proveedor manualmente en el servidor VMM. Si tiene un clúster, quítelo de todos los nodos.
8. Si estaban replicando máquinas virtuales en Azure, tiene que desinstalar el agente de Microsoft Recovery Services de los hosts de Hyper-V en las nubes eliminadas.

## <a name="unregister-a-hyper-v-host-in-a-hyper-v-site"></a>Anulación del registro de un host Hyper-V en un sitio de Hyper-V

Los hosts Hyper-V que no están administrados por VMM se reúnen en un sitio de Hyper-V. Puede eliminar un host en un sitio Hyper-V de la forma siguiente:

1. Deshabilite la replicación para máquinas virtuales de Hyper-V que se encuentran en el host.
2. Disocie las directivas para el sitio de Hyper-V. En **Site Recovery Infrastructure** > **For Hyper-V Sites** >  **Replication Policies**, (Infraestructura de Site Recovery > Para sitios Hyper-V > Directivas de replicación) haga doble clic en la directiva asociada. Haga clic con el botón derecho en el sitio > **Disassociate** (Desasociar).
3. Elimine los hosts de Hyper-V. En **Site Recovery Infrastructure** > **For Hyper V Sites** > **Hyper-V Hosts** (Infraestructura de Site Recovery > Para sitios de Hyper V > Hosts Hyper-V), haga clic con el botón derecho en el servidor > **Eliminar**.
4. Elimine el sitio de Hyper-V una vez que se hayan quitado todos los hosts del mismo. En **Site Recovery Infrastructure** > **For Hyper V Sites** > **Hyper-V Sites** (Infraestructura de Site Recovery > Para sitios de Hyper V > Sitios de Hyper-V), haga clic con el botón derecho en el servidor > **Eliminar**.
5. Si el host de Hyper-V se encontraba en estado **Desconectado**, ejecute el siguiente script en cada host de Hyper-V que se haya quitado. El script limpia la configuración en el servidor y anula su registro en el almacén.


```powershell
        pushd .
        try
        {
            $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
            $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
            $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
            $isAdmin=$principal.IsInRole($administrators)
            if (!$isAdmin)
            {
                "Please run the script as an administrator in elevated mode."
                $choice = Read-Host
                return;
            }

            $error.Clear()
            "This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
            $choice =  Read-Host

            if (!($choice -eq 'Y' -or $choice -eq 'y'))
            {
            "Stopping cleanup."
            return;
            }

            $serviceName = "dra"
            $service = Get-Service -Name $serviceName
            if ($service.Status -eq "Running")
            {
                "Stopping the Azure Site Recovery service..."
                net stop $serviceName
            }

            $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
            $registrationPath = $asrHivePath + '\Registration'
            $proxySettingsPath = $asrHivePath + '\ProxySettings'
            $draIdvalue = 'DraID'
            $idMgmtCloudContainerId='IdMgmtCloudContainerId'


            if (Test-Path $asrHivePath)
            {
                if (Test-Path $registrationPath)
                {
                    "Removing registration related registry keys."
                    Remove-Item -Recurse -Path $registrationPath
                }

                if (Test-Path $proxySettingsPath)
                {
                    "Removing proxy settings"
                    Remove-Item -Recurse -Path $proxySettingsPath
                }

                $regNode = Get-ItemProperty -Path $asrHivePath
                if($regNode.DraID -ne $null)
                {
                    "Removing DraId"
                    Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
                }
                if($regNode.IdMgmtCloudContainerId -ne $null)
                {
                    "Removing IdMgmtCloudContainerId"
                    Remove-ItemProperty -Path $asrHivePath -Name $idMgmtCloudContainerId
                }
                "Registry keys removed."
            }

            # First retrieve all the certificates to be deleted
            $ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
            # Open a cert store object
            $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
            $store.Open('ReadWrite')
            # Delete the certs
            "Removing all related certificates"
            foreach ($cert in $ASRcerts)
            {
                $store.Remove($cert)
            }
        }catch
        {
            [system.exception]
            Write-Host "Error occurred" -ForegroundColor "Red"
            $error[0]
            Write-Host "FAILED" -ForegroundColor "Red"
        }
        popd
```


## <a name="disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure"></a>Deshabilitación de la protección de una máquina virtual de VMware o un servidor físico (WMware a Azure)

1. En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina > **Deshabilitar replicación**.
2. En la página **Deshabilitar replicación**, seleccione una de estas opciones:
    - **Deshabilitar la replicación y quitar (recomendado)** : esta opción quita el elemento replicado de Azure Site Recovery y se detiene la replicación de la máquina. Se limpia la configuración de replicación en el servidor de configuración y se detiene la facturación de Site Recovery para este servidor protegido. Tenga en cuenta que esta opción solo se puede usar cuando el servidor de configuración está en estado conectado.
    - **Quitar**: esta opción debe utilizarse solo si el entorno de origen se ha eliminado o no es accesible (no conectado). Esto quita el elemento replicado de Azure Site Recovery (se detiene la facturación). La configuración de replicación en el servidor de configuración **no** se va a limpiar. 

> [!NOTE]
> En ambas opciones no se desinstalará el servicio de movilidad de los servidores protegidos; hay que desinstalarlo manualmente. Si va a proteger el servidor de nuevo con el mismo servidor de configuración, puede omitir la desinstalación del servicio de movilidad.

> [!NOTE]
> Si ya conmutó por error una VM y esta se ejecuta en Azure, tenga en cuenta que deshabilitar la protección no elimina ni afecta su estado de conmutación por error.
## <a name="disable-protection-for-a-azure-vm-azure-to-azure"></a>Deshabilitación de la protección de una máquina virtual de Azure (Azure a Azure)

-  En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina > **Deshabilitar replicación**.
> [!NOTE]
> No se desinstalará el servicio de movilidad de los servidores protegidos; tendrá que desinstalarlo manualmente. Si va a proteger el servidor de nuevo, puede omitir la desinstalación del servicio de movilidad.

## <a name="disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure"></a>Deshabilitación de la protección para una máquina virtual de Hyper-V (Hyper-V a Azure)

> [!NOTE]
> Utilice este procedimiento si está replicando máquinas virtuales de Hyper-V a Azure sin un servidor VMM. Si va a replicar las máquinas virtuales mediante el escenario **System Center VMM a Azure**, siga las instrucciones de la sección Deshabilitación de la protección para una máquina virtual de Hyper-V que se replica mediante el escenario de System Center VMM a Azure.

1. En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina > **Deshabilitar replicación**.
2. En **Deshabilitar replicación**, puede seleccionar una de estas opciones:
   - **Deshabilitar la replicación y quitar (recomendado)** : esta opción quita el elemento replicado de Azure Site Recovery y se detiene la replicación de la máquina. Se limpiará la configuración de replicación en la máquina virtual local y se detiene facturación de Site Recovery para este servidor protegido.
   - **Quitar**: esta opción debe utilizarse solo si el entorno de origen se ha eliminado o no es accesible (no conectado). Esto quita el elemento replicado de Azure Site Recovery (se detiene la facturación). La configuración de replicación de la máquina virtual local **no** se va a limpiar. 

    > [!NOTE]
    > Si elige la opción **Quitar**, ejecute el siguiente conjunto de scripts para limpiar la configuración de replicación en Hyper-V Server local.

    > [!NOTE]
    > Si ya conmutó por error una VM y esta se ejecuta en Azure, tenga en cuenta que deshabilitar la protección no elimina ni afecta su estado de conmutación por error.

1. En el servidor host de Hyper-V de origen, para quitar la replicación de la máquina virtual. Reemplace SQLVM1 por el nombre de la máquina virtual y ejecute el script desde un PowerShell administrativo.

```powershell
    $vmName = "SQLVM1"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario"></a>Deshabilitación de la protección para una máquina virtual de Hyper-V que se replica mediante el escenario de System Center VMM a Azure

1. En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina > **Deshabilitar replicación**.
2. En la página **Deshabilitar replicación**, seleccione una de estas opciones:

   - **Deshabilitar la replicación y quitar (recomendado)** : esta opción quita el elemento replicado de Azure Site Recovery y se detiene la replicación de la máquina. Se limpia la configuración de replicación en la máquina virtual local y se detiene facturación de Site Recovery para este servidor protegido.
   - **Quitar**: esta opción debe utilizarse solo si el entorno de origen se ha eliminado o no es accesible (no conectado). Esto quita el elemento replicado de Azure Site Recovery (se detiene la facturación). La configuración de replicación de la máquina virtual local **no** se va a limpiar. 

     > [!NOTE]
     > Si elige la opción **Quitar**, ejecute los scripts siguientes para limpiar la configuración de replicación en VMM Server local.
3. Ejecute este script en el servidor VMM de origen mediante PowerShell (se requieren privilegios de administrador) en la consola VMM. Reemplace el marcador de posición **SQLVM1** por el nombre de la máquina virtual.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Set-SCVirtualMachine -VM $vm -ClearDRProtection
    ```

4. Los pasos anteriores borrarán la configuración de replicación en el servidor VMM. Para detener la replicación de la máquina virtual que se ejecuta en el servidor host de Hyper-V, ejecute este script. Reemplace SQLVM1 con el nombre de la máquina virtual y host01.contoso.com con el nombre del servidor host de Hyper-V.

```powershell
    $vmName = "SQLVM1"
    $hostName  = "host01.contoso.com"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-secondary-vmm-server-using-the-system-center-vmm-to-vmm-scenario"></a>Deshabilitación de la protección para una máquina virtual de Hyper-V que se replica a un servidor VMM secundario con el escenario de System Center VMM a VMM

1. En **Elementos protegidos** > **Elementos replicados**, haga clic con el botón derecho en la máquina > **Deshabilitar replicación**.
2. En la página **Deshabilitar replicación**, seleccione una de estas opciones:

   - **Deshabilitar la replicación y quitar (recomendado)** : esta opción quita el elemento replicado de Azure Site Recovery y se detiene la replicación de la máquina. Se limpia la configuración de replicación en la máquina virtual local y se detiene facturación de Site Recovery para este servidor protegido.
   - **Quitar**: esta opción debe utilizarse solo si el entorno de origen se ha eliminado o no es accesible (no conectado). Esto quita el elemento replicado de Azure Site Recovery (se detiene la facturación). La configuración de replicación de la máquina virtual local **no** se va a limpiar. Ejecute el siguiente conjunto de scripts para limpiar la configuración de replicación de las máquinas virtuales locales.
     > [!NOTE]
     > Si elige la opción **Quitar**, ejecute los scripts siguientes para limpiar la configuración de replicación en VMM Server local.

3. Ejecute este script en el servidor VMM de origen mediante PowerShell (se requieren privilegios de administrador) en la consola VMM. Reemplace el marcador de posición **SQLVM1** por el nombre de la máquina virtual.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Set-SCVirtualMachine -VM $vm -ClearDRProtection
    ```

4. En el servidor VMM secundario, ejecute este script para limpiar la configuración de la máquina virtual secundaria:

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Remove-SCVirtualMachine -VM $vm -Force
    ```

5. En el servidor VMM secundario, actualice las máquinas virtuales en el servidor host de Hyper-V para que se vuelva a detectar la máquina virtual secundaria en la consola VMM.
6. Los pasos anteriores borrarán la configuración de replicación en el servidor VMM. Si desea detener la replicación de la máquina virtual, ejecute el siguiente script en las máquinas virtuales principales y secundarias. Reemplace SQLVM1 por el nombre de la máquina virtual.

    ```powershell
    Remove-VMReplication –VMName "SQLVM1"
    ```
