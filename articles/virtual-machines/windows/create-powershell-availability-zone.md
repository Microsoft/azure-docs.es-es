---
title: Creación de una máquina virtual Windows en la zona mediante Azure PowerShell
description: Creación de una máquina virtual Windows en una zona de disponibilidad con Azure PowerShell
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: ''
ms.openlocfilehash: 95d97605f00a3a80c097eb16695c9d296e17c33d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825249"
---
# <a name="create-a-windows-virtual-machine-in-an-availability-zone-with-powershell"></a>Creación de una máquina virtual Windows en una zona de disponibilidad con PowerShell

En este artículo se detallan el uso de Azure PowerShell para crear una máquina virtual de Azure que ejecute Windows Server 2016 en una zona de disponibilidad de Azure. Una [zona de disponibilidad](../../availability-zones/az-overview.md) es una zona separada físicamente en una región de Azure. Use zonas de disponibilidad para proteger sus datos y aplicaciones de la improbable pérdida o error de todo un centro de datos.

Para usar una zona de disponibilidad, cree la máquina virtual [en una región de Azure compatible](../../availability-zones/az-region.md).

 

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en la suscripción a Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

```powershell
Connect-AzAccount
```

## <a name="check-vm-sku-availability"></a>Comprobación de la disponibilidad del SKU de la máquina virtual
La disponibilidad de tamaños de máquinas virtuales o SKU puede variar según la región y la zona. Para ayudarle a planear el uso de las Zonas de disponibilidad, puede enumerar las SKU de la máquina virtual disponibles por región de Azure y zona. Con esta funcionalidad se asegura de elegir un tamaño apropiado de máquina virtual y obtener la resistencia deseada en las distintas zonas. Para más información sobre los diferentes tipos y tamaños de máquina virtual, consulte [Introducción a los tamaños de máquina virtual](../sizes.md).

Puede ver las SKU de máquina virtual disponibles con el comando [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku). En el ejemplo siguiente se enumeran las SKU de máquina virtual disponibles en la región *eastus2*:

```powershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("eastus2")};
```

La salida es similar al siguiente ejemplo reducido, en el que se muestran las Zonas de disponibilidad en las que está disponible cada tamaño de máquina virtual:

```powershell
ResourceType                Name  Location      Zones   [...]
------------                ----  --------      -----
virtualMachines  Standard_DS1_v2   eastus2  {1, 2, 3}
virtualMachines  Standard_DS2_v2   eastus2  {1, 2, 3}
[...]
virtualMachines     Standard_F1s   eastus2  {1, 2, 3}
virtualMachines     Standard_F2s   eastus2  {1, 2, 3}
[...]
virtualMachines  Standard_D2s_v3   eastus2  {1, 2, 3}
virtualMachines  Standard_D4s_v3   eastus2  {1, 2, 3}
[...]
virtualMachines   Standard_E2_v3   eastus2  {1, 2, 3}
virtualMachines   Standard_E4_v3   eastus2  {1, 2, 3}
```


## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En este ejemplo se crea un grupo de recursos denominado *myResourceGroup* en la región *eastus2*. 

```powershell
New-AzResourceGroup -Name myResourceGroup -Location EastUS2
```

## <a name="create-networking-resources"></a>Creación de los recursos de red principales

### <a name="create-a-virtual-network-subnet-and-a-public-ip-address"></a>Cree una red virtual, una subred y una dirección IP pública 
Estos recursos se utilizan para proporcionar conectividad de red a la máquina virtual y conectarse a Internet. Cree la dirección IP en una zona de disponibilidad, en este ejemplo, *2*. En un paso posterior, puede crear la máquina virtual en la misma zona que se usó para crear la dirección IP.

```powershell
# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName myResourceGroup -Location eastus2 `
    -Name myVNet -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address in an availability zone and specify a DNS name
$pip = New-AzPublicIpAddress -ResourceGroupName myResourceGroup -Location eastus2 -Zone 2 `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)" -Sku Standard
```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>Creación de un grupo de seguridad de red y una regla de grupo de seguridad de red 
El grupo de seguridad de red protege la máquina virtual con reglas entrantes y salientes. En este caso, se crea una regla de entrada para el puerto 3389, que permite las conexiones entrantes al Escritorio remoto. También queremos crear una regla de entrada para el puerto 80, que permita el tráfico web entrante.

```powershell
# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
    -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location eastus2 `
    -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-virtual-machine"></a>Creación de una tarjeta de red para la máquina virtual 
Cree una tarjeta de red con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) para la máquina virtual. Esta conecta la máquina virtual a una subred, un grupo de seguridad de red y una dirección IP pública.

```powershell
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location eastus2 `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-virtual-machine"></a>Crear máquina virtual

Cree una configuración de máquina virtual. Esta configuración incluye los ajustes que se usan al implementar la máquina virtual como una imagen de máquina virtual, el tamaño y la configuración de autenticación. El tamaño *Standard_DS1_v2* de este ejemplo se admite en las zonas de disponibilidad. Esta configuración también especifica la zona de disponibilidad que se establece al crear la dirección IP. Cuando se realiza este paso, se le solicitará las credenciales. Los valores que especifique se configuran como el nombre de usuario y la contraseña de la máquina virtual.

```powershell
# Define a credential object
$cred = Get-Credential

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName myVM -VMSize Standard_DS1_v2 -Zone 2 | `
    Set-AzVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzVMNetworkInterface -Id $nic.Id
```

Cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm).

```powershell
New-AzVM -ResourceGroupName myResourceGroup -Location eastus2 -VM $vmConfig
```

## <a name="confirm-zone-for-managed-disk"></a>Confirmación de zona para disco administrado

Ha creado el recurso de dirección IP de la máquina virtual en la misma zona de disponibilidad que la máquina virtual. También se crea el recurso de disco administrado para la máquina virtual en la misma zona de disponibilidad. Puede comprobarlo con [Get-AzDisk](/powershell/module/az.compute/get-azdisk):

```powershell
Get-AzDisk -ResourceGroupName myResourceGroup
```

La salida muestra que el disco administrado está en la misma zona de disponibilidad que la máquina virtual:

```powershell
ResourceGroupName  : myResourceGroup
AccountType        : PremiumLRS
OwnerId            : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/virtualMachines/myVM
ManagedBy          : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx//resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/virtualMachines/myVM
Sku                : Microsoft.Azure.Management.Compute.Models.DiskSku
Zones              : {2}
TimeCreated        : 9/7/2017 6:57:26 PM
OsType             : Windows
CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
DiskSizeGB         : 127
EncryptionSettings :
ProvisioningState  : Succeeded
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/disks/myVM_OsDisk_1_bd921920bb0a4650becfc2d830000000
Name               : myVM_OsDisk_1_bd921920bb0a4650becfc2d830000000
Type               : Microsoft.Compute/disks
Location           : eastus2
Tags               : {}
```


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear una máquina virtual en una zona de disponibilidad. Aprenda más sobre la [disponibilidad](../availability.md) de las máquinas virtuales de Azure.