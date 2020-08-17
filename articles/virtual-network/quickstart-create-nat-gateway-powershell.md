---
title: 'Creación de una puerta de enlace de NAT: Azure PowerShell'
titlesuffix: Azure Virtual Network NAT
description: En este inicio rápido se muestra cómo crear una puerta de enlace de NAT mediante Azure PowerShell.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a NAT gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/18/2020
ms.author: allensu
ms.openlocfilehash: de4e32d79cf4dfb3a5f54544c65544297a2c0232
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88054177"
---
# <a name="tutorial-create-a-nat-gateway-using-azure-powershell"></a>Tutorial: Creación de una puerta de enlace de NAT mediante Azure PowerShell

En este tutorial se muestra cómo usar el servicio Azure Virtual Network NAT. Creará una puerta de enlace de NAT para proporcionar conectividad saliente para una máquina virtual en Azure. 


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Puede realizar este tutorial con Azure Cloud Shell o ejecutar los comandos de forma local.  Si no ha usado Azure Cloud Shell, [inicie sesión ahora](https://shell.azure.com).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=latest). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente se crea un grupo de recursos llamado **myResourceGroupNAT** en la ubicación **eastus2**:

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
  
New-AzResourceGroup -Name $rsg -Location $loc
```

## <a name="create-the-nat-gateway"></a>Creación de la puerta de enlace de NAT

Las opciones de IP pública de la puerta de enlace de NAT son:

* **Direcciones IP públicas**
* **Prefijos de dirección IP pública**

Ambos se pueden usar con la puerta de enlace de NAT.

Para demostrarlo, se va a agregar una dirección IP pública y un prefijo de dirección IP pública a este escenario.

### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Para acceder a Internet, necesita una o varias direcciones IP públicas para la puerta de enlace de NAT. Use [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress?view=latest) para crear el recurso de dirección IP pública llamado **myPublicIP** en **myResourceGroupNAT**. El resultado de este comando se almacenará en una variable **$publicIP** para su uso posterior.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$sku = 'Standard'
$pbnm = 'myPublicIP'
  
$publicIP = 
New-AzPublicIpAddress -Name $pbnm -ResourceGroupName $rsg -AllocationMethod Static -Location $loc -Sku $sku
```

### <a name="create-a-public-ip-prefix"></a>Creación de un prefijo de dirección IP pública

Use [New-AzPublicIpPrefix](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipprefix?view=latest) para crear un recurso de prefijo de dirección IP pública llamado **myPublicIPprefix** en **myResourceGroupNAT**.  El resultado de este comando se almacenará en una variable llamada **$publicIPPrefix** para su uso posterior.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$pxnm = 'myPublicIPprefix'

$publicIPPrefix = 
New-AzPublicIpPrefix -Name $pxnm -ResourceGroupName $rsg -Location $loc -PrefixLength 31
```

### <a name="create-a-nat-gateway-resource"></a>Creación de un recurso de puerta de enlace de NAT

En esta sección se detalla cómo crear y configurar los componentes siguientes del servicio NAT mediante el recurso de puerta de enlace de NAT:
  - Un grupo de direcciones IP públicas y un prefijo de dirección IP pública que se usarán en los flujos salientes traducidos por el recurso de puerta de enlace de NAT.
  - Cambie el tiempo de espera de inactividad de 4 (el valor predeterminado) a 10 minutos.

Cree una puerta de enlace de NAT de Azure global con [New-AzNatGateway](https://docs.microsoft.com/powershell/module/az.network/new-aznatgateway). El resultado de este comando creará un recurso de puerta de enlace llamado **myNATgateway** que usa la dirección IP pública **myPublicIP** y el prefijo de dirección IP pública **myPublicIPprefix**. El tiempo de espera de inactividad se establece en 10 minutos.  El resultado de este comando se almacenará en una variable llamada **$natGateway** para su uso posterior.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$sku = 'Standard'
$gnm = 'myNATgateway'

$natGateway = 
New-AzNatGateway -Name $gnm -ResourceGroupName $rsg -PublicIpAddress $publicIP -PublicIpPrefix $publicIPPrefix -Location $loc -Sku $sku -IdleTimeoutInMinutes 10
  ```

En este momento, la puerta de enlace de NAT es funcional y todo lo que queda es configurar qué subredes de una red virtual deben usarla.

## <a name="configure-virtual-network"></a>Configurar la red virtual

Cree la red virtual y asocie la subred a la puerta de enlace.

Cree una red virtual denominada **myVnet** con una subred llamada **mySubnet** con [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig?view=latest) en **myResourceGroup** usando [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork?view=latest). El espacio de direcciones IP de la red virtual es **192.168.0.0/16**. La subred de la red virtual es **192.168.0.0/24**.  El resultado de los comandos se almacenará en las variables llamadas **$subnet** y **$Vnet** para su uso posterior.

```azurepowershell-interactive
$sbnm = 'mySubnet'
$vnnm = 'myVnet'
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$pfxsub = '192.168.0.0/24'
$pfxvn = '192.168.0.0/16'

$subnet = 
New-AzVirtualNetworkSubnetConfig -Name $sbnm -AddressPrefix $pfxsub -NatGateway $natGateway

$vnet = 
New-AzVirtualNetwork -Name $vnnm -ResourceGroupName $rsg -Location $loc -AddressPrefix $pfxvn -Subnet $subnet
```

Todo el tráfico saliente a destinos de Internet usa ahora el servicio NAT.  No es necesario configurar una UDR.

## <a name="create-a-vm-to-use-the-nat-service"></a>Creación de una máquina virtual para usar el servicio NAT

Ahora se va a crear una máquina virtual para usar el servicio NAT.  Esta máquina virtual tiene una dirección IP pública en el nivel de instancia que le permite el acceso a la máquina virtual.  El servicio NAT tiene en cuenta la dirección del flujo y reemplazará el destino predeterminado de Internet en la subred. La dirección IP pública de la máquina virtual no se usará para las conexiones salientes.

### <a name="create-public-ip-for-source-vm"></a>Creación de una dirección IP pública para la máquina virtual de origen

Se va a crear una dirección IP pública que se usará para acceder a la máquina virtual.  Use [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress?view=latest) para crear el recurso de dirección IP pública llamado **myPublicIPVM** en **myResourceGroupNAT**.  El resultado de este comando se almacenará en una variable llamada **$publicIpVM** para su uso posterior.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$ipnm = 'myPublicIPVM'
$sku = 'Standard'

$publicIpVM = 
New-AzPublicIpAddress -Name $ipnm -ResourceGroupName $rsg -AllocationMethod Static -Location $loc -Sku $sku
```

### <a name="create-an-nsg-and-expose-ssh-endpoint-for-vm"></a>Creación de un grupo de seguridad de red y exposición del punto de conexión de SSH para la máquina virtual

Las direcciones IP públicas estándar son "seguras de forma predeterminada", por lo que deberá crear un grupo de seguridad de red para permitir el acceso de entrada mediante SSH. Use [New-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecuritygroup?view=latest) para crear un recurso de grupo de seguridad de red llamado **myNSG**. Use [New-AzNetworkSecurityRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecurityruleconfig?view=latest) para crear una regla de grupo de seguridad de red para el acceso mediante SSH llamada **ssh** en **myResourceGroupNAT**.  El resultado de este comando se almacenará en una variable llamada **$nsg** para su uso posterior.

```azurepowershell-interactive
$rnm = 'ssh'
$rdesc = 'SSH access'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '100'
$prt = '22'
$rsg = 'myResourceGroupNAT'
$rnm = 'myNSG'
$loc = 'eastus2'

$sshrule = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $rdesc -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange $prt

$nsg = 
New-AzNetworkSecurityGroup -ResourceGroupName $rsg -Name $rnm -Location $loc -SecurityRules $sshrule 
```

### <a name="create-nic-for-vm"></a>Creación de una NIC para una máquina virtual

Cree una interfaz de red con [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface?view=azps-2.8.0) llamada **myNic**. Este comando asocia la dirección IP pública y el grupo de seguridad de red. El resultado de este comando se almacenará en una variable llamada **$nic** para su uso posterior.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$nmn = 'myNic'
$loc = 'eastus2'

$nic = 
New-AzNetworkInterface -ResourceGroupName $rsg -Name $nmn -NetworkSecurityGroupID $nsg.Id -PublicIPAddressID $publicIPVM.Id -SubnetID $vnet.Subnets[0].Id -Location $loc
```

### <a name="create-vm"></a>Creación de una máquina virtual

#### <a name="create-ssh-key-pair"></a>Creación del par de claves SSH

Necesita un par de claves SSH para completar esta guía de inicio rápido. Si ya tiene un par de claves SSH, puede omitir este paso.

Use ssh-keygen para crear un par de claves SSH.

```azurepowershell-interactive
ssh-keygen -t rsa -b 2048
```
Para más información sobre cómo crear pares de claves SSH, incluido el uso de PuTTy, consulte [Uso de claves SSH con Windows](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows).

Si crea el par de claves SSH mediante Cloud Shell, se almacenará en una imagen de contenedor. Esta [cuenta de almacenamiento se crea automáticamente](https://docs.microsoft.com/azure/cloud-shell/persisting-shell-storage). No elimine la cuenta de almacenamiento, ni el recurso compartido de archivos que contiene, hasta que haya recuperado las claves.

#### <a name="create-vm-configuration"></a>Creación de una configuración de máquina virtual

Para crear una máquina virtual en PowerShell, cree una configuración que tenga valores para la imagen que se va a usar y las opciones de tamaño y autenticación. La configuración se usa para crear la máquina virtual.

Defina las credenciales de SSH, la información del sistema operativo y el tamaño de máquina virtual. En este ejemplo, la clave SSH se almacena en ~/.ssh/id_rsa.pub.

```azurepowershell-interactive
#Define a credential object

$securePassword = 
ConvertTo-SecureString ' ' -AsPlainText -Force

$cred = 
New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration

$vnm = 'myVM'
$vsz = 'Standard_D1'
$pub = 'Canonical'
$off = 'UbuntuServer'
$sku = '18.04-LTS'
$ver = 'latest'

$vmConfig = 
New-AzVMConfig -VMName $vnm -VMSize $vsz

Set-AzVMOperatingSystem -VM $vmConfig -Linux -ComputerName $vnm -Credential $cred -DisablePasswordAuthentication

Set-AzVMSourceImage -VM $vmConfig -PublisherName $pub -Offer $off -Skus $sku -Version $ver

Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Configure the SSH key

$sshPublicKey = cat ~/.ssh/id_rsa.pub

Add-AzVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

```
Combine las definiciones de configuración para crear una máquina virtual llamada **myVM** con [New-AzVM](/powershell/module/az.compute/new-azvm?view=azps-2.8.0) en **myResourceGroupNAT**.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'

New-AzVM -ResourceGroupName $rsg -Location $loc -VM $vmconfig
```

Espere a que la máquina virtual se prepare para la implementación y continúe con el resto de los pasos.

## <a name="discover-the-ip-address-of-the-vm"></a>Detección de la dirección IP de la máquina virtual

En primer lugar, es necesario detectar la dirección IP de la máquina virtual que ha creado. Para obtener la dirección IP pública de la máquina virtual, use [Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress?view=latest). 

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$nmn = 'myPublicIPVM'

Get-AzPublicIpAddress -ResourceGroupName $rsg -Name $nmn | select IpAddress
``` 

>[!IMPORTANT]
>Copie la dirección IP pública y, luego, péguela en un bloc de notas para que pueda usarla para acceder a la máquina virtual.

### <a name="sign-in-to-vm"></a>Inicio de sesión en la máquina virtual

Las credenciales de SSH deben almacenarse en la instancia de Cloud Shell de la operación anterior.  Abra [Azure Cloud Shell](https://shell.azure.com) en el explorador. Use la dirección IP recuperada en el paso anterior para conectarse a la máquina virtual mediante SSH.

```bash
ssh azureuser@<ip-address-destination>
```

Ahora está listo para usar el servicio NAT.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=latest) para quitar el grupo de recursos y todos los recursos que contiene.

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroupNAT
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una puerta de enlace de NAT y una máquina virtual para usarla. 

Revise las métricas de Azure Monitor para ver el funcionamiento del servicio NAT. Diagnostique problemas como el agotamiento de recursos de los puertos SNAT disponibles.  El agotamiento de recursos de los puertos SNAT se soluciona agregando recursos de dirección IP pública o recursos de prefijo de dirección IP pública adicionales (o ambos).


- Obtenga más información sobre [Azure Virtual Network NAT](./nat-overview.md).
- Más información sobre [recursos de puerta de enlace de NAT](./nat-gateway-resource.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con la CLI de Azure](./quickstart-create-nat-gateway-cli.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con Azure Portal](./quickstart-create-nat-gateway-portal.md).
> [!div class="nextstepaction"]


