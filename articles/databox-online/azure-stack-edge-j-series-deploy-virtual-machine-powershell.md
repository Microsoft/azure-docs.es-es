---
title: Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU a través de Azure PowerShell
description: En este artículo se explica cómo crear y administrar máquinas virtuales en un dispositivo Azure Stack Edge Pro con GPU por medio de Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: aa492acdedc2d131d28c894031de2181e87a2f3e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90890694"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-azure-powershell"></a>Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU a través de Azure PowerShell

<!--[!INCLUDE [azure-stack-edge-gateway-deploy-vm-overview](../../includes/azure-stack-edge-gateway-deploy-virtual-machine-overview.md)]-->

En este tutorial se describe cómo crear y administrar una máquina virtual en un dispositivo Azure Stack Edge Pro por medio de Azure PowerShell.

## <a name="vm-deployment-workflow"></a>Flujo de trabajo de implementación de una VM

En el diagrama siguiente se muestra el flujo de trabajo de implementación.

![Flujo de trabajo de implementación de una VM](media/azure-stack-edge-j-series-deploy-virtual-machine-powershell/vm-workflow_r.svg)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [azure-stack-edge-gateway-deploy-vm-prerequisites](../../includes/azure-stack-edge-gateway-deploy-virtual-machine-prerequisites.md)]



## <a name="query-for-built-in-subscription-on-the-device"></a>Consulta de la suscripción integrada en el dispositivo

Por Azure Resource Manager, solo se admite una única suscripción fija visible para el usuario. Esta suscripción es única por dispositivo, y este identificador de suscripción o nombre de suscripción no se pueden cambiar.

Esta suscripción contiene todos los recursos necesarios para la creación de VM. 

> [!IMPORTANT]
> Esta suscripción no está conectada ni relacionada con la suscripción a Azure y reside localmente en el dispositivo.

Esta suscripción se usará para implementar las VM.

1.  Para mostrar esta suscripción, escriba:

    ```powershell
    Get-AzureRmSubscription
    ```
    
    A continuación se muestra una salida de ejemplo.

    ```powershell
    PS C:\windows\system32> Get-AzureRmSubscription
    
    Name                 Id                 TenantId          State
    ----                 --                --------           -----
    Default Provider Subscription A4257FDE-B946-4E01-ADE7-674760B8D1A3 c0257de7-538f-415c-993a-1b87a031879d Enabled
    
    PS C:\windows\system32>
    ```
        
3.  Obtenga la lista de los proveedores de recursos registrados que se ejecutan en el dispositivo. Normalmente, esta lista incluye Proceso, Red y Almacenamiento.

    ```powershell
    Get-AzureRMResourceProvider
    ```

    > [!NOTE]
    > Los proveedores de recursos están previamente registrados y no se pueden modificar ni cambiar.
    
    A continuación se muestra una salida de ejemplo:

    ```powershell
    Get-AzureRmResourceProvider
    ProviderNamespace : Microsoft.Compute
    RegistrationState : Registered
    ResourceTypes     : {virtualMachines, virtualMachines/extensions, locations, operations...}
    Locations         : {DBELocal}
    ZoneMappings      :
    
    ProviderNamespace : Microsoft.Network
    RegistrationState : Registered
    ResourceTypes     : {operations, locations, locations/operations, locations/usages...}
    Locations         : {DBELocal}
    ZoneMappings      :
    
    ProviderNamespace : Microsoft.Resources
    RegistrationState : Registered
    ResourceTypes     : {tenants, locations, providers, checkresourcename...}
    Locations         : {DBELocal}
    ZoneMappings      :
    
    ProviderNamespace : Microsoft.Storage
    RegistrationState : Registered
    ResourceTypes     : {storageaccounts, storageAccounts/blobServices, storageAccounts/tableServices,
                        storageAccounts/queueServices...}
    Locations         : {DBELocal}
    ZoneMappings      :
    ```
    
## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y administran los recursos de Azure, como la cuenta de almacenamiento, el disco y el disco administrado.

> [!IMPORTANT]
> Todos los recursos se crean en la misma ubicación que el dispositivo y la ubicación se establece en **DBELocal**.

```powershell
New-AzureRmResourceGroup -Name <Resource group name> -Location DBELocal
```

A continuación se muestra una salida de ejemplo.

```powershell
New-AzureRmResourceGroup -Name rg191113014333 -Location DBELocal 
Successfully created Resource Group:rg191113014333
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Cree una cuenta de almacenamiento en el grupo de recursos creado en el paso anterior. Se trata de una **cuenta de almacenamiento local** que se usará para cargar la imagen de disco virtual de la VM.

```powershell
New-AzureRmStorageAccount -Name <Storage account name> -ResourceGroupName <Resource group name> -Location DBELocal -SkuName Standard_LRS
```

> [!NOTE]
> Con Azure Resource Manager solo se pueden crear cuentas de almacenamiento local, como las cuentas de almacenamiento con redundancia local (Standard_LRS o Premium_LRS). Para crear cuentas de almacenamiento en capas, vea los pasos descritos en el tema sobre [incorporación y conexión a cuentas de almacenamiento en el dispositivo Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-add-storage-accounts.md).

A continuación se muestra una salida de ejemplo.

```powershell
New-AzureRmStorageAccount -Name sa191113014333  -ResourceGroupName rg191113014333 -SkuName Standard_LRS -Location DBELocal

ResourceGroupName      : rg191113014333
StorageAccountName     : sa191113014333
Id                     : /subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/rg191113014333/providers/Microsoft.Storage/storageaccounts/sa191113014333
Location               : DBELocal
Sku                    : Microsoft.Azure.Management.Storage.Models.Sku
Kind                   : Storage
Encryption             : Microsoft.Azure.Management.Storage.Models.Encryption
AccessTier             :
CreationTime           : 11/13/2019 9:43:49 PM
CustomDomain           :
Identity               :
LastGeoFailoverTime    :
PrimaryEndpoints       : Microsoft.Azure.Management.Storage.Models.Endpoints
PrimaryLocation        : DBELocal
ProvisioningState      : Succeeded
SecondaryEndpoints     :
SecondaryLocation      :
StatusOfPrimary        : Available
StatusOfSecondary      :
Tags                   :
EnableHttpsTrafficOnly : False
NetworkRuleSet         :
Context                : Microsoft.WindowsAzure.Commands.Common.Storage.LazyAzureStorageContext
ExtendedProperties     : {}
```

Para obtener la clave de la cuenta de almacenamiento, ejecute el comando `Get-AzureRmStorageAccountKey`. A continuación se muestra un resultado de ejemplo del comando.

```powershell
PS C:\Users\Administrator> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: my-resource-ase
Name:myasestoracct

KeyName Value
------- -----
key1 /IjVJN+sSf7FMKiiPLlDm8mc9P4wtcmhhbnCa7...
key2 gd34TcaDzDgsY9JtDNMUgLDOItUU0Qur3CBo6Q...
```

## <a name="add-blob-uri-to-hosts-file"></a>Adición del URI del blob al archivo de hosts

Ya ha agregado el URI del blob en el archivo de hosts para el cliente que está usando para conectarse al almacenamiento de blobs en la sección [Modificación del archivo de host para la resolución de nombres de punto de conexión](azure-stack-edge-j-series-connect-resource-manager.md#step-5-modify-host-file-for-endpoint-name-resolution). Esta era la entrada para el URI del blob:

\<Azure consistent network services VIP \> \<storage name\>.blob.\<appliance name\>.\<dnsdomain\>


## <a name="install-certificates"></a>Instalación de certificados

Si usa *https*, deberá instalar los certificados adecuados en el dispositivo. En este caso, instale el certificado del punto de conexión de blob. Para obtener más información, consulte cómo crear y cargar certificados en [Administración de certificados](azure-stack-edge-j-series-manage-certificates.md).

## <a name="upload-a-vhd"></a>Carga de un disco duro virtual

Copie las imágenes de disco que se van a usar en los blobs en páginas en la cuenta de almacenamiento local que creó en los pasos anteriores. Puede usar una herramienta como [AzCopy](../storage/common/storage-use-azcopy-v10.md) para cargar el disco duro virtual en la cuenta de almacenamiento que creó en los pasos anteriores. 

Antes de utilizar AzCopy, asegúrese de que [AzCopy esté configurado correctamente](#configure-azcopy) para usarlo con la versión de la API REST del almacenamiento de blobs que emplea con el dispositivo Azure Stack Edge Pro.

```powershell
AzCopy /Source:<sourceDirectoryForVHD> /Dest:<blobContainerUri> /DestKey:<storageAccountKey> /Y /S /V /NC:32  /BlobType:page /destType:blob 
```

> [!NOTE]
> Establezca `BlobType` en page para crear un disco administrado fuera del VHD. Establezca `BlobType` en block al escribir en cuentas de almacenamiento en capas mediante AzCopy.

Puede descargar las imágenes de disco del marketplace. Para obtener pasos detallados, vaya a [Obtención de la imagen de disco virtual de Azure Marketplace](azure-stack-edge-j-series-create-virtual-machine-image.md).

A continuación se muestra una salida de ejemplo que usa AzCopy 7.3. Para obtener más información sobre este comando, vaya a [Carga de archivos VHD en la cuenta de almacenamiento mediante AzCopy](../devtest-labs/devtest-lab-upload-vhd-using-azcopy.md).


```powershell
AzCopy /Source:\\hcsfs\scratch\vm_vhds\linux\ /Dest:http://sa191113014333.blob.dbe-1dcmhq2.microsoftdatabox.com/vmimages /DestKey:gJKoyX2Amg0Zytd1ogA1kQ2xqudMHn7ljcDtkJRHwMZbMK== /Y /S /V /NC:32 /BlobType:page /destType:blob /z:2e7d7d27-c983-410c-b4aa-b0aa668af0c6
```


## <a name="create-managed-disks-from-the-vhd"></a>Creación de discos administrados a partir del VHD

Cree un disco administrado desde el VHD cargado.

```powershell
$DiskConfig = New-AzureRmDiskConfig -Location DBELocal -CreateOption Import -SourceUri "Source URL for your VHD"
```
A continuación se muestra una salida de ejemplo: 
<code>
$DiskConfig = New-AzureRmDiskConfig -Location DBELocal -CreateOption Import –SourceUri http://</code><code>sa191113014333.blob.dbe-1dcmhq2.microsoftdatabox.com/vmimages/ubuntu13.vhd</code> 

```powershell
New-AzureRMDisk -ResourceGroupName <Resource group name> -DiskName <Disk name> -Disk $DiskConfig
```

A continuación se muestra una salida de ejemplo. Para obtener más información sobre este cmdlet, vaya a [New-AzureRmDisk](https://docs.microsoft.com/powershell/module/azurerm.compute/new-azurermdisk?view=azurermps-6.13.0).

```powershell
Tags               :
New-AzureRmDisk -ResourceGroupName rg191113014333 -DiskName ld191113014333 -Disk $DiskConfig

ResourceGroupName  : rg191113014333
ManagedBy          :
Sku                : Microsoft.Azure.Management.Compute.Models.DiskSku
Zones              :
TimeCreated        : 11/13/2019 1:49:07 PM
OsType             :
CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
DiskSizeGB         : 30
EncryptionSettings :
ProvisioningState  : Succeeded
Id                 : /subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/rg191113014333/providers/Micros
                     oft.Compute/disks/ld191113014333
Name               : ld191113014333
Type               : Microsoft.Compute/disks
Location           : DBELocal
Tags               : {}
```

## <a name="create-a-vm-image-from-the-image-managed-disk"></a>Creación de una imagen de VM a partir de la imagen del disco administrado

Use el siguiente comando para crear una imagen de VM a partir del disco administrado. Reemplace los valores de \< \> por los nombres que elija.

```powershell
$imageConfig = New-AzureRmImageConfig -Location DBELocal
$ManagedDiskId = (Get-AzureRmDisk -Name <Disk name> -ResourceGroupName <Resource group name>).Id
Set-AzureRmImageOsDisk -Image $imageConfig -OsType '<OS type>' -OsState 'Generalized' -DiskSizeGB <Disk size> -ManagedDiskId $ManagedDiskId 

The supported OS types are Linux and Windows.

For OS Type=Linux, for example:
Set-AzureRmImageOsDisk -Image $imageConfig -OsType 'Linux' -OsState 'Generalized' -DiskSizeGB <Disk size> -ManagedDiskId $ManagedDiskId
New-AzureRmImage -Image $imageConfig -ImageName <Image name>  -ResourceGroupName <Resource group name>
```

A continuación se muestra una salida de ejemplo. Para obtener más información sobre este cmdlet, vaya a [New-AzureRmImage](https://docs.microsoft.com/powershell/module/azurerm.compute/new-azurermimage?view=azurermps-6.13.0).

```powershell
New-AzureRmImage -Image Microsoft.Azure.Commands.Compute.Automation.Models.PSImage -ImageName ig191113014333  -ResourceGroupName rg191113014333
ResourceGroupName    : rg191113014333
SourceVirtualMachine :
StorageProfile       : Microsoft.Azure.Management.Compute.Models.ImageStorageProfile
ProvisioningState    : Succeeded
Id                   : /subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/rg191113014333/providers/Micr
                       osoft.Compute/images/ig191113014333
Name                 : ig191113014333
Type                 : Microsoft.Compute/images
Location             : dbelocal
Tags                 : {}
```

## <a name="create-vm-with-previously-created-resources"></a>Creación de una VM con recursos creados anteriormente

Debe crear una red virtual y asociar una interfaz de red virtual antes de crear e implementar la VM.

> [!IMPORTANT]
> Al crear una red virtual y una interfaz de red virtual, se aplican las siguientes reglas:
> - Solo se puede crear una red virtual (incluso entre grupos de recursos) y debe coincidir exactamente con la red lógica en cuanto al espacio de direcciones.
> -   Solo se permitirá una subred en la red virtual. La subred debe tener exactamente el mismo espacio de direcciones que la red virtual.
> -   Solo se permitirá el método de asignación estática durante la creación de VNIC, y el usuario debe proporcionar una dirección IP privada.

 
**Creación de una red virtual**

```powershell
$subNetId=New-AzureRmVirtualNetworkSubnetConfig -Name <Subnet name> -AddressPrefix <Address Prefix>
$aRmVN = New-AzureRmVirtualNetwork -ResourceGroupName <Resource group name> -Name <Vnet name> -Location DBELocal -AddressPrefix <Address prefix> -Subnet $subNetId
```

**Creación de una VNIC con el identificador de subred de la red virtual**

```powershell
$ipConfig = New-AzureRmNetworkInterfaceIpConfig -Name <IP config Name> -SubnetId $aRmVN.Subnets[0].Id -PrivateIpAddress <Private IP>
$Nic = New-AzureRmNetworkInterface -Name <Nic name> -ResourceGroupName <Resource group name> -Location DBELocal -IpConfiguration $ipConfig
```

A continuación se muestra el resultado de ejemplo de estos comandos:

```powershell
PS C:\Users\Administrator> $subNetId=New-AzureRmVirtualNetworkSubnetConfig -Name my-ase-subnet -AddressPrefix "5.5.0.0/16"

PS C:\Users\Administrator> $aRmVN = New-AzureRmVirtualNetwork -ResourceGroupName Resource-my-ase -Name my-ase-virtualnetwork -Location DBELocal -AddressPrefix "5.5.0.0/16" -Subnet $subNetId
WARNING: The output object type of this cmdlet will be modified in a future release.
PS C:\Users\Administrator> $ipConfig = New-AzureRmNetworkInterfaceIpConfig -Name my-ase-ip -SubnetId $aRmVN.Subnets[0].Id
PS C:\Users\Administrator> $Nic = New-AzureRmNetworkInterface -Name my-ase-nic -ResourceGroupName Resource-my-ase -Location DBELocal -IpConfiguration $ipConfig
WARNING: The output object type of this cmdlet will be modified in a future release.

PS C:\Users\Administrator> $Nic

PS C:\Users\Administrator> (Get-AzureRmNetworkInterface)[0]

Name                        : nic200108020444
ResourceGroupName           : rg200108020444
Location                    : dbelocal
Id                          : /subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/rg200108020444/providers/Microsoft.Network/networ
                              kInterfaces/nic200108020444
Etag                        : W/"f9d1759d-4d49-42fa-8826-e218e0b1d355"
ResourceGuid                : 3851ae62-c13e-4416-9386-e21d9a2fef0f
ProvisioningState           : Succeeded
Tags                        :
VirtualMachine              : {
                                "Id": "/subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/rg200108020444/providers/Microsoft.Compu
                              te/virtualMachines/VM200108020444"
                              }
IpConfigurations            : [
                                {
                                  "Name": "ip200108020444",
                                  "Etag": "W/\"f9d1759d-4d49-42fa-8826-e218e0b1d355\"",
                                  "Id": "/subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/rg200108020444/providers/Microsoft.Net
                              work/networkInterfaces/nic200108020444/ipConfigurations/ip200108020444",
                                  "PrivateIpAddress": "5.5.166.65",
                                  "PrivateIpAllocationMethod": "Static",
                                  "Subnet": {
                                    "Id": "/subscriptions/a4257fde-b946-4e01-ade7-674760b8d1a3/resourceGroups/DbeSystemRG/providers/Microsoft.Netw
                              ork/virtualNetworks/vSwitch1/subnets/subnet123",
                                    "ResourceNavigationLinks": [],
                                    "ServiceEndpoints": []
                                  },
                                  "ProvisioningState": "Succeeded",
                                  "PrivateIpAddressVersion": "IPv4",
                                  "LoadBalancerBackendAddressPools": [],
                                  "LoadBalancerInboundNatRules": [],
                                  "Primary": true,
                                  "ApplicationGatewayBackendAddressPools": [],
                                  "ApplicationSecurityGroups": []
                                }
                              ]
DnsSettings                 : {
                                "DnsServers": [],
                                "AppliedDnsServers": []
                              }
EnableIPForwarding          : False
EnableAcceleratedNetworking : False
NetworkSecurityGroup        : null
Primary                     : True
MacAddress                  : 00155D18E432                :
```

Opcionalmente, al crear un VNIC para una VM, puede pasar la dirección IP pública. En este caso, la dirección IP pública devolverá la dirección IP privada. 

```powershell
New-AzureRmPublicIPAddress -Name <Public IP> -ResourceGroupName <ResourceGroupName> -AllocationMethod Static -Location DBELocal
$publicIP = (Get-AzureRmPublicIPAddress -Name <Public IP> -ResourceGroupName <Resource group name>).Id
$ipConfig = New-AzureRmNetworkInterfaceIpConfig -Name <ConfigName> -PublicIpAddressId $publicIP -SubnetId $subNetId
```


**Crear una máquina virtual**

Ahora puede usar la imagen de la VM para crear una VM y conectarla a la red virtual que creó anteriormente.

```powershell
$pass = ConvertTo-SecureString "<Password>" -AsPlainText -Force;
$cred = New-Object System.Management.Automation.PSCredential("<Enter username>", $pass)

You will use this username, password to login to the VM, once it is created and powered up.

$VirtualMachine = New-AzureRmVMConfig -VMName <VM name> -VMSize "Standard_D1_v2"

$VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -<OS type> -ComputerName <Your computer Name> -Credential $cred

$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name <OS Disk Name> -Caching "ReadWrite" -CreateOption "FromImage" -Linux -StorageAccountType StandardLRS

$nicID = (Get-AzureRmNetworkInterface -Name <nic name> -ResourceGroupName <Resource Group Name>).Id

$VirtualMachine = Add-AzureRmVMNetworkInterface -VM $VirtualMachine -Id $nicID

$image = (Get-AzureRmImage -ResourceGroupName <Resource Group Name> -ImageName $ImageName).Id

$VirtualMachine = Set-AzureRmVMSourceImage -VM $VirtualMachine -Id $image

New-AzureRmVM -ResourceGroupName <Resource Group Name> -Location DBELocal -VM $VirtualMachine -Verbose
```

## <a name="connect-to-a-vm"></a>Conexión a una máquina virtual

Los pasos para conectarse pueden ser diferentes dependiendo de si se ha creado una máquina virtual Windows o Linux.

### <a name="connect-to-linux-vm"></a>Conexión a una máquina virtual Linux

Siga estos pasos para conectarse a una máquina virtual Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]

### <a name="connect-to-windows-vm"></a>Conexión a una máquina virtual Windows

Siga estos pasos para conectarse a una máquina virtual Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]


<!--Connect to the VM using the private IP that you passed during the VM creation.

Open an SSH session to connect with the IP address.

`ssh -l <username> <ip address>`

When prompted, provide the password that you used when creating the VM.

If you need to provide the SSH key, use this command.

ssh -i c:/users/Administrator/.ssh/id_rsa Administrator@5.5.41.236

If you used a public IP address during VM creation, you can use that IP to connect to the VM. To get the public IP: 

```powershell
$publicIp = Get-AzureRmPublicIpAddress -Name <Public IP> -ResourceGroupName <Resource group name>
```
The public IP in this case will be the same as the private IP that you passed during virtual network interface creation.-->


## <a name="manage-vm"></a>Administración de la VM

En la siguiente sección se describen algunas de las operaciones comunes en torno a la máquina virtual que se creará en el dispositivo Azure Stack Edge Pro.

### <a name="list-vms-running-on-the-device"></a>Enumeración de VM que se ejecutan en el dispositivo

Ejecute el siguiente comando para devolver una lista de todas las máquinas virtuales que se ejecutan en el dispositivo Azure Stack Edge Pro.


`Get-AzureRmVM -ResourceGroupName <String> -Name <String>`
 

### <a name="turn-on-the-vm"></a>Activación de la VM

Ejecute el siguiente cmdlet para activar una máquina virtual que se ejecute en el dispositivo:


`Start-AzureRmVM [-Name] <String> [-ResourceGroupName] <String>`


Para obtener más información sobre este cmdlet, vaya a [Start-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/start-azurermvm?view=azurermps-6.13.0).

### <a name="suspend-or-shut-down-the-vm"></a>Suspensión o apagado de la VM

Ejecute el siguiente cmdlet para detener o apagar una máquina virtual que se esté ejecutando en el dispositivo:


```powershell
Stop-AzureRmVM [-Name] <String> [-StayProvisioned] [-ResourceGroupName] <String>
```


Para obtener más información sobre este cmdlet, vaya a [Stop-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/stop-azurermvm?view=azurermps-6.13.0).

### <a name="add-a-data-disk"></a>Agregar un disco de datos

Si los requisitos de carga de trabajo en la VM aumentan, es posible que tenga que agregar un disco de datos.

```powershell
Add-AzureRmVMDataDisk -VM $VirtualMachine -Name "disk1" -VhdUri "https://contoso.blob.core.windows.net/vhds/diskstandard03.vhd" -LUN 0 -Caching ReadOnly -DiskSizeinGB 1 -CreateOption Empty 
 
Update-AzureRmVM -ResourceGroupName "<Resource Group Name string>" -VM $VirtualMachine
```

### <a name="delete-the-vm"></a>Eliminación de la máquina virtual

Ejecute el siguiente cmdlet para quitar una máquina virtual del dispositivo:

```powershell
Remove-AzureRmVM [-Name] <String> [-ResourceGroupName] <String>
```

Para obtener más información sobre este cmdlet, vaya a [Remove-AzureRmVm](https://docs.microsoft.com/powershell/module/azurerm.compute/remove-azurermvm?view=azurermps-6.13.0).


## <a name="supported-vm-sizes"></a>Tamaños de máquinas virtuales que se admiten

El tamaño de la máquina virtual determina la cantidad de recursos de proceso, como memoria, CPU y GPU, que están disponibles para la máquina virtual. Las máquinas virtuales deben crearse con un tamaño de máquina virtual adecuado para la carga de trabajo. Aunque todas las máquinas se ejecutarán en el mismo hardware, los tamaños de máquina tienen límites diferentes para el acceso al disco, lo que puede ayudarle a administrar el acceso total al disco en todas las VM. Si esta aumenta, también se puede cambiar el tamaño de las máquinas virtuales existentes.

Se pueden usar las siguientes máquinas virtuales de la serie Standard Dv2 para crear un dispositivo Azure Stack Edge Pro.

### <a name="dv2-series"></a>Serie Dv2
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|-------------------|----|----|-----|----|------|------------|---------|
|**Standard_D1_v2** |1   |3,5 |50   |500 |3000  |4 / 4x500   |2 |
|**Standard_D2_v2** |2   |7   |100  |500 |6000  |8 / 8x500   |2 |
|**Standard_D3_v2** |4   |14  |200  |500 |12000 |16 / 16x500 |4 |
|**Standard_D4_v2** |8   |28  |400  |500 |24000 |32 / 32x500 |8 |
|**Standard_D5_v2** |16  |56  |800  |500 |48000 |64 / 64x500 |8 |

### <a name="dsv2-series"></a>DSv2-series
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|--------------------|----|----|----|-----|------|-------------|---------|
|**Standard_DS1_v2** |1   |3,5 |7   |1000 |4000  |4 / 4x2300   |2 |
|**Standard_DS2_v2** |2   |7   |14  |1000 |8000  |8 / 8x2300   |2 |
|**Standard_DS3_v2** |4   |14  |28  |1000 |16000 |16 / 16x2300 |4 |
|**Standard_DS4_v2** |8   |28  |56  |1000 |32000 |32 / 32x2300 |8 |
|**Standard_DS5_v2** |16  |56  |112 |1000 |64000 |64 / 64x2300 |8 |

### <a name="dv2-series"></a>Serie Dv2
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|--------------------|----|----|-----|----|-------|-------------|---------|
|**Standard_D11_v2** |2   |14  |100  |500 |6000   |8 / 8x500    |2 |
|**Standard_D12_v2** |4   |28  |200  |500 |12000  |16 / 16x500  |4 |
|**Standard_D13_v2** |8   |56  |400  |500 |24000  |32 / 32x500  |8 |
|**Standard_D14_v2** |16  |112 |800  |500 |48000  |64 / 64x500  |8 |


### <a name="dsv2-series"></a>DSv2-series
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|---------------------|----|----|-----|-----|-------|--------------|---------|
|**Standard_DS11_v2** |2   |14  |28   |1000 |8000   |4 / 4x2300    |2 |
|**Standard_DS12_v2** |4   |28  |56   |1000 |16000  |8 / 8x2300    |4 |
|**Standard_DS13_v2** |8   |56  |112  |1000 |32000  |16 / 16x2300  |8 |
|**Standard_DS14_v2** |16  |112 |224  |1000 |64000  |32 / 32x2300  |8 |

Para obtener más información, vaya a los [tamaños de VM de uso general de la serie Dv2](../virtual-machines/dv2-dsv2-series.md#dv2-series).

## <a name="unsupported-vm-operations-and-cmdlets"></a>Operaciones y cmdlets de VM que no se admiten

No se admiten las extensiones, los conjuntos de escalado, los conjuntos de disponibilidad ni las instantáneas.

## <a name="configure-azcopy"></a>Configuración de AzCopy

Al instalar la versión más reciente de AzCopy, tendrá que configurar AzCopy para asegurarse de que coincide con la versión de la API REST del almacenamiento de blobs del dispositivo Azure Stack Edge Pro.

En el cliente usado para acceder al dispositivo Azure Stack Edge Pro, configure una variable global para que coincida con la versión de la API REST del almacenamiento de blobs.

### <a name="on-windows-client"></a>En un cliente Windows 

`$Env:AZCOPY_DEFAULT_SERVICE_API_VERSION = "2017-11-09"`

### <a name="on-linux-client"></a>En un cliente Linux

`export AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09`

Para comprobar si la variable de entorno de AzCopy se estableció correctamente, siga estos pasos:

1. Ejecute "azcopy env".
2. Busque el parámetro `AZCOPY_DEFAULT_SERVICE_API_VERSION`. Debe tener el valor establecido en los pasos anteriores.


## <a name="next-steps"></a>Pasos siguientes

[Cmdlets de Azure Resource Manager](https://docs.microsoft.com/powershell/module/azurerm.resources/?view=azurermps-6.13.0)
