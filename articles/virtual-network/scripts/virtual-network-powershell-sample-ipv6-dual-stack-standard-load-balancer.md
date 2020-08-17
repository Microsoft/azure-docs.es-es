---
title: 'Ejemplo de script de Azure PowerShell: configuración de un front-end IPv6 con Standard Load Balancer (versión preliminar)'
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo configurar un front-end IPv6 en el de script de red virtual de ejemplo con Standard Load Balancer.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.workload: infrastructure-services
ms.date: 07/15/2019
ms.author: kumud
ms.openlocfilehash: 24057c494d76cf9435ed60ecf2a13db4eab36fbf
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291896"
---
# <a name="configure-ipv6-frontend-in-virtual-network-script-sample-with-standard-load-balancerpreview"></a>Configuración de un front-end IPv6 en el de script de red virtual de ejemplo con Standard Load Balancer (versión preliminar)

En este artículo, se explica cómo se implementa en Azure una aplicación de pila doble (IPv4 + IPv6) que contiene una red virtual de pila doble y una subred de pila doble, un equilibrador de carga con configuraciones de front-end duales (IPv4 + IPv6), VM con NIC que tienen una configuración de IP dual, reglas de grupo de seguridad de red dual e IP públicas duales.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/powershell) o desde una instalación de PowerShell local. Si usa PowerShell de forma local, este script requiere la versión 1.0.0 o posterior del módulo Az de Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos
Antes de implementar una aplicación de pila doble en Azure, debe configurar una sola vez la suscripción para esta característica en versión preliminar mediante el siguiente comando de Azure PowerShell:

Regístrese del modo siguiente:
```azurepowershell
Register-AzProviderFeature -FeatureName AllowIPv6VirtualNetwork -ProviderNamespace Microsoft.Network
Register-AzProviderFeature -FeatureName AllowIPv6CAOnStandardLB -ProviderNamespace Microsoft.Network
```
Se tarda hasta 30 minutos en completar el registro de características. Puede comprobar el estado del registro mediante la ejecución del siguiente comando de Azure PowerShell: Compruebe en el registro del modo siguiente:
```azurepowershell
Get-AzProviderFeature -FeatureName AllowIPv6VirtualNetwork -ProviderNamespace Microsoft.Network
Get-AzProviderFeature -FeatureName AllowIPv6CAOnStandardLB -ProviderNamespace Microsoft.Network
```
Una vez completado el registro, ejecute el siguiente comando:

```azurepowershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Network
```

## <a name="sample-script"></a>Script de ejemplo

```azurepowershell
#   Deploys Dual-stack (IPv4+IPv6) Azure virtual network with 2 VMs and Standard Load Balancer with IPv4 and IPv6 public IPs

# Create resource group to contain the deployment
  $rg = New-AzResourceGroup `
  -ResourceGroupName "dsRG1"  `
  -Location "east us"

# Create the public IPs needed for the deployment
  $PublicIP_v4 = New-AzPublicIpAddress `
  -Name "dsPublicIP_v4" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv4 `
  -Sku Standard
  
$PublicIP_v6 = New-AzPublicIpAddress `
  -Name "dsPublicIP_v6" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv6 `
  -Sku Standard

$RdpPublicIP_1 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_1" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -Sku Standard `
  -IpAddressVersion IPv4
  
$RdpPublicIP_2 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_2" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -Sku Standard `
  -IpAddressVersion IPv4

# Create front-end IP address
$frontendIPv4 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v4" `
  -PublicIpAddress $PublicIP_v4

$frontendIPv6 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v6" `
  -PublicIpAddress $PublicIP_v6

# Configure back-end address pool  
$backendPoolv4 = New-AzLoadBalancerBackendAddressPoolConfig `
-Name "dsLbBackEndPool_v4"
$backendPoolv6 = New-AzLoadBalancerBackendAddressPoolConfig `
-Name "dsLbBackEndPool_v6"

# Create load balancer rule
$lbrule_v4 = New-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v4" `
  -FrontendIpConfiguration $frontendIPv4 `
  -BackendAddressPool $backendPoolv4 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80

$lbrule_v6 = New-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v6" `
  -FrontendIpConfiguration $frontendIPv6 `
  -BackendAddressPool $backendPoolv6 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80
  
# Create Standard Load Balancer
$lb = New-AzLoadBalancer `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "MyLoadBalancer" `
-Sku "Standard" `
-FrontendIpConfiguration $frontendIPv4,$frontendIPv6 `
-BackendAddressPool $backendPoolv4,$backendPoolv6 `
-LoadBalancingRule $lbrule_v4,$lbrule_v6

# Create availability set
$avset = New-AzAvailabilitySet `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsAVset" `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2 `
  -Sku aligned
  
# Create network security group and rules
$rule1 = New-AzNetworkSecurityRuleConfig `
-Name 'myNetworkSecurityGroupRuleRDP' `
-Description 'Allow RDP' `
-Access Allow `
-Protocol Tcp `
-Direction Inbound `
-Priority 100 `
-SourceAddressPrefix * `
-SourcePortRange * `
-DestinationAddressPrefix * `
-DestinationPortRange 3389

$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name 'myNetworkSecurityGroupRuleHTTP' `
  -Description 'Allow HTTP' `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange 80 `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80

$nsg = New-AzNetworkSecurityGroup `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "dsNSG1"  `
-SecurityRules $rule1,$rule2

#Create virtual network and subnet
# Create dual stack subnet
$subnet = New-AzVirtualNetworkSubnetConfig `
-Name "dsSubnet" `
-AddressPrefix "10.0.0.0/24","ace:cab:deca:deed::/64"

# Create the virtual network
$vnet = New-AzVirtualNetwork `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "dsVnet" `
-AddressPrefix "10.0.0.0/16","ace:cab:deca::/48"  `
-Subnet $subnet
  
#Create network interfaces (NICs)
$Ip4Config=New-AzNetworkInterfaceIpConfig `
-Name dsIp4Config `
-Subnet $vnet.subnets[0] `
-PrivateIpAddressVersion IPv4 `
-LoadBalancerBackendAddressPool $backendPoolv4 `
-PublicIpAddress  $RdpPublicIP_1
    
$Ip6Config=New-AzNetworkInterfaceIpConfig `
-Name dsIp6Config `
-Subnet $vnet.subnets[0] `
-PrivateIpAddressVersion IPv6 `
-LoadBalancerBackendAddressPool $backendPoolv6
    
$NIC_1 = New-AzNetworkInterface `
-Name "dsNIC1" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-NetworkSecurityGroupId $nsg.Id `
-IpConfiguration $Ip4Config,$Ip6Config 
    
$Ip4Config=New-AzNetworkInterfaceIpConfig `
-Name dsIp4Config `
-Subnet $vnet.subnets[0] `
-PrivateIpAddressVersion IPv4 `
-LoadBalancerBackendAddressPool $backendPoolv4 `
-PublicIpAddress  $RdpPublicIP_2  

$NIC_2 = New-AzNetworkInterface `
-Name "dsNIC2" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-NetworkSecurityGroupId $nsg.Id `
-IpConfiguration $Ip4Config,$Ip6Config 

# Create virtual machines
$cred = get-credential -Message "DUAL STACK VNET SAMPLE:  Please enter the Administrator credential to log into the VMs"

$vmsize = "Standard_A2"
$ImagePublisher = "MicrosoftWindowsServer"
$imageOffer = "WindowsServer"
$imageSKU = "2019-Datacenter"

$vmName= "dsVM1"
$VMconfig1 = New-AzVMConfig -VMName $vmName -VMSize $vmsize -AvailabilitySetId $avset.Id 3> $null | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent 3> $null | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" 3> $null | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption fromImage  3> $null | Add-AzVMNetworkInterface -Id $NIC_1.Id  3> $null 
$VM1 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $VMconfig1 

$vmName= "dsVM2"
$VMconfig2 = New-AzVMConfig -VMName $vmName -VMSize $vmsize -AvailabilitySetId $avset.Id 3> $null | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent 3> $null | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" 3> $null | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption fromImage  3> $null | Add-AzVMNetworkInterface -Id $NIC_2.Id  3> $null 
$VM2 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $VMconfig2

#End Of Script

```
## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```powershell
Remove-AzResourceGroup -Name <resourcegroupname> -Force
```
## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual, un conjunto de disponibilidad, un equilibrador de carga y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea una configuración de subred. Esta configuración se utiliza con el proceso de creación de la red virtual. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual y una subred de Azure. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress)  | Crea una dirección IP pública con una dirección IP estática y un nombre DNS asociado. |
| [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer)  | Crea una instancia de Azure Load Balancer. |
| [New-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) | Crea un sondeo de equilibrador de carga. Un sondeo de equilibrador de carga se usa para supervisar cada máquina virtual del conjunto de equilibrador de carga. Si alguna máquina virtual deja de estar accesible, el tráfico no se enruta a la máquina virtual. |
| [New-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) | Crea una regla de equilibrador de carga. En este ejemplo, se crea una regla para el puerto 80. Según va llegando el tráfico HTTP al equilibrador de carga, se enruta al puerto 80 de una de las máquinas virtuales del conjunto de equilibrador de carga. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea un grupo de seguridad de red (NSG), que es un límite de seguridad entre Internet y la máquina virtual. |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) | Crea una regla de NSG para permitir el tráfico entrante. En este ejemplo, el puerto 22 está abierto al tráfico SSH. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Crea una tarjeta de máquina virtual y la conecta con la red virtual, la subred y el NSG. |
| [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset) | Crea un conjunto de disponibilidad. Los conjuntos de disponibilidad garantizan la disponibilidad de las aplicaciones al repartir las máquinas virtuales entre distintos recursos físicos, de forma que si se produce un error, el conjunto no se verá afectado en su totalidad. |
| [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) | Crea una configuración de máquina virtual. Esta configuración incluye diversa información, como el nombre de la máquina virtual, sistema el operativo y las credenciales administrativas. La configuración se utiliza durante la creación de las máquinas virtuales. |
| [New-AzVM](/powershell/module/az.compute/new-azvm)  | Crea la máquina virtual y la conecta con la tarjeta de red, la red virtual, la subred y el NSG. Este comando también especifica la imagen de máquina virtual que se usará, y las credenciales administrativas.  |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

En la [documentación de la información general de redes de Azure](../powershell-samples.md?toc=%2fazure%2fnetworking%2ftoc.json) puede encontrar ejemplos adicionales de script de PowerShell de redes.
