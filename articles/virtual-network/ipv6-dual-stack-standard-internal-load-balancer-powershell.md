---
title: 'Implementación de una aplicación de doble pila IPv6 con equilibrador de carga interno estándar en Azure: PowerShell'
titlesuffix: Azure Virtual Network
description: En este artículo se muestra cómo implementar una aplicación de doble pila IPv6 con implementador de carga interno estándar en una red virtual de Azure mediante Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/14/2019
ms.author: kumud
ms.openlocfilehash: c224332eec31b343bdc53564ef4075a0620ac340
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289578"
---
# <a name="deploy-an-ipv6-dual-stack-application-using-standard-internal-load-balancer-in-azure---powershell-preview"></a>Implementación de una aplicación de doble pila IPv6 con equilibrador de carga interno estándar en Azure: PowerShell (versión preliminar)

En este artículo se explica cómo se implementa en Azure una aplicación de doble pila (IPv4 + IPv6) que contiene una red virtual de pila doble y una subred, un equilibrador de carga interno estándar con configuraciones de front-end dobles (IPv4 + IPv6), máquinas virtuales con NIC que tienen una configuración de IP doble, un grupo de seguridad de red e IP públicas.

El procedimiento para crear un equilibrador de carga interno compatible con IPv6 es casi idéntico al del equilibrador de carga compatible con IPv6 orientado a Internet que se describe [aquí](virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). Las únicas diferencias en la creación de un equilibrador de carga interno se encuentran en la configuración de front-end, tal como se muestra en el ejemplo de PowerShell siguiente:

```azurepowershell
 $frontendIPv6 = New-AzLoadBalancerFrontendIpConfig `
 -Name "dsLbFrontEnd_v6" `
 -PrivateIpAddress "ace:cab:deca:deed::100" `
 -PrivateIpAddressVersion "IPv6" `
 -Subnet $DsSubnet
```

Los cambios que convierten la anterior en una configuración de front-end de equilibrador de carga interno son:
- El `PrivateIpAddressVersion` se especifica como "IPv6"
- El argumento `-PublicIpAddress` se ha omitido o se ha reemplazado por `-PrivateIpAddress`. Tenga en cuenta que la dirección privada debe estar en el intervalo del espacio IP de la subred en el que se implemente el equilibrador de carga interno. Si se omite una `-PrivateIpAddress` estática, se seleccionará la siguiente dirección IPv6 libre de la subred en la que se haya implementado el equilibrador de carga interno.
- La subred de doble pila en la que se implementará el equilibrador de carga interno se especifica con un argumento `-Subnet` o `-SubnetId`.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, la versión del módulo de Azure PowerShell que necesita este artículo es la 6.9.0 u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="prerequisites"></a>Prerrequisitos
Para poder implementar una aplicación de pila doble en Azure, primero debe configurar la suscripción de esta característica en vista previa (GB) utilizando el siguiente procedimiento de Azure PowerShell:

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

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para poder generar la red virtual de doble pila, debe crear primero un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente se crea un grupo de recursos llamado *dsStd_ILB_RG* en la ubicación *Este de EE. UU.* :

```azurepowershell
$rg = New-AzResourceGroup `
  -ResourceGroupName "dsStd_ILB_RG"  `
  -Location "east us"
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses"></a>Creación de direcciones IP públicas IPv4 e IPv6
Para acceder a las máquinas virtuales desde Internet, necesita direcciones IP públicas IPv4 e IPv6 para las máquinas virtuales. Cree las direcciones IP públicas con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). En el ejemplo siguiente se crean unas direcciones IP públicas IPv4 e IPv6 denominadas *RdpPublicIP_1* y *RdpPublicIP_2* en el grupo de recursos *dsStd_ILB_RG*:

```azurepowershell
$RdpPublicIP_1 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_1" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv4  `
  -sku Standard
  
$RdpPublicIP_2 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_2" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv4  `
  -sku Standard
```
## <a name="create-the-virtual-network-and-the-subnet"></a>Creación de la red virtual y la subred

Cree una red virtual mediante [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) con doble pila y una configuración de subred mediante [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). En el ejemplo siguiente se crea una red virtual denominada *dsVnet* con *dsSubnet*.

```azurepowershell
# Create dual stack subnet config
$DsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name "dsSubnet" `
  -AddressPrefix "10.0.0.0/24","ace:cab:deca:deed::/64"

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsVnet" `
  -AddressPrefix "10.0.0.0/16","ace:cab:deca::/48"  `
  -Subnet $DsSubnet

#Refresh the fully populated subnet for use in load balancer frontend configuration
$DsSubnet = get-AzVirtualNetworkSubnetconfig -name dsSubnet -VirtualNetwork $vnet
```
## <a name="create-standard-load-balancer"></a>Creación de Standard Load Balancer

En esta sección, configurará la dirección IP de front-end doble (IPv4 e IPv6) y el grupo de direcciones de back-end del equilibrador de carga, además de crear una instancia de Standard Load Balancer.

### <a name="create-front-end-ip"></a>Creación de una dirección IP de front-end

Cree una dirección IP de front-end con [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig). En el ejemplo siguiente, se crean unas configuraciones de IP de front-end IPv4 e IPv6 llamadas *dsLbFrontEnd_v4* y *dsLbFrontEnd_v6*:

```azurepowershell
$frontendIPv4 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v4" `
  -PrivateIpAddress "10.0.0.100"  `
  -PrivateIpAddressVersion "IPv4"   `
  -Subnet $DsSubnet

$frontendIPv6 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v6" `
  -PrivateIpAddress "ace:cab:deca:deed::100"  `
  -PrivateIpAddressVersion "IPv6"   `
  -Subnet $DsSubnet

```

### <a name="configure-back-end-address-pool"></a>Configuración del grupo de direcciones de back-end

Cree un grupo de direcciones de back-end con [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig). En el resto de los pasos, las máquinas virtuales se conectan a este grupo back-end. En el ejemplo siguiente, se crean los grupos de direcciones de back-end *dsLbBackEndPool_v4* y *dsLbBackEndPool_v6*, que van a incluir máquinas virtuales con configuraciones de NIC IPV4 e IPv6:

```azurepowershell
$backendPoolv4 = New-AzLoadBalancerBackendAddressPoolConfig -Name "dsLbBackEndPool_v4"

$backendPoolv6 = New-AzLoadBalancerBackendAddressPoolConfig -Name "dsLbBackEndPool_v6"
```

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración de la IP de front-end para el tráfico entrante y el grupo de IP de back-end para el tráfico entrante, junto con los puertos de origen y destino requeridos. Para tener la seguridad de que solo reciben tráfico las máquinas virtuales correctas, también puede definir un sondeo de estado. Los equilibradores de carga básicos usan un sondeo de IPv4 para evaluar el estado de los puntos de conexión IPv4 e IPv6 de las máquinas virtuales. Los equilibradores de carga estándar permiten realizar explícitamente sondeos de estado de IPv6.

Cree una regla del equilibrador de carga con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig). En el ejemplo siguiente, se crean reglas del equilibrador de carga llamadas *dsLBrule_v4* y *dsLBrule_v6*, y se equilibra el tráfico del puerto *TCP* *80* dirigido a las configuraciones de IP de front-end IPv4 e IPv6:

```azurepowershell
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
```

### <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Cree una instancia de Standard Load Balancer con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). En el ejemplo siguiente se crea un equilibrador de carga público estándar denominado *myInternalLoadBalancer* mediante las configuraciones de direcciones IP de front-end IPv4 e IPv6, grupos de servidores back-end y las reglas de equilibrio de carga que creó en los pasos anteriores:

```azurepowershell
$lb = New-AzLoadBalancer  `
  -ResourceGroupName $rg.ResourceGroupName  `
  -Location $rg.Location  `
  -Name  "MyInternalLoadBalancer"  `
  -Sku "Standard"  `
  -FrontendIpConfiguration  $frontendIPv4,$frontendIPv6  `
  -BackendAddressPool  $backendPoolv4,$backendPoolv6  `
  -LoadBalancingRule  $lbrule_v4,$lbrule_v6

```

## <a name="create-network-resources"></a>Crear recursos de red
Para poder implementar algunas máquinas virtuales y probar el equilibrador, debe crear unos recursos de red que lo permitan: un conjunto de disponibilidad, un grupo de seguridad de red y varias NIC virtuales. 

### <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad
Para mejorar la alta disponibilidad de la aplicación, coloque las máquinas virtuales en un conjunto de disponibilidad.

Cree un conjunto de disponibilidad con [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). En el ejemplo siguiente se crea un conjunto de disponibilidad denominado *dsAVset*:

```azurepowershell
$avset = New-AzAvailabilitySet `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsAVset" `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2 `
  -Sku aligned
```

### <a name="create-network-security-group"></a>Creación de un grupo de seguridad de red

Cree un grupo de seguridad de red para las reglas que van a controlar la comunicación entrante y saliente de la red virtual.

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>Creación de una regla de grupo de seguridad de red para el puerto 3389

Cree una regla de grupo de seguridad de red para permitir conexiones RDP en el puerto 3389 con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell
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
```
#### <a name="create-a-network-security-group-rule-for-port-80"></a>Creación de una regla de grupo de seguridad de red para el puerto 80

Cree una regla de grupo de seguridad de red para permitir conexiones de Internet a través del puerto 80 con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell
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
```
#### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsNSG1"  `
  -SecurityRules $rule1,$rule2
```
### <a name="create-nics"></a>Creación de tarjetas NIC

Cree NIC virtuales con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). En el ejemplo siguiente, se crean dos NIC virtuales, ambas con las configuraciones IPv4 e IPv6. (Una NIC virtual para cada máquina virtual que cree para la aplicación en los pasos siguientes).

```azurepowershell

# Create the IPv4 configuration for NIC 1
$Ip4Config=New-AzNetworkInterfaceIpConfig `
  -Name dsIp4Config `
  -Subnet $vnet.subnets[0] `
  -PrivateIpAddressVersion IPv4 `
  -LoadBalancerBackendAddressPool $backendPoolv4 `
  -PublicIpAddress  $RdpPublicIP_1

# Create the IPv6 configuration
$Ip6Config=New-AzNetworkInterfaceIpConfig `
  -Name dsIp6Config `
  -Subnet $vnet.subnets[0] `
  -PrivateIpAddressVersion IPv6 `
  -LoadBalancerBackendAddressPool $backendPoolv6

# Create NIC 1
$NIC_1 = New-AzNetworkInterface `
  -Name "dsNIC1" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -NetworkSecurityGroupId $nsg.Id `
  -IpConfiguration $Ip4Config,$Ip6Config

# Create the IPv4 configuration for NIC 2
$Ip4Config=New-AzNetworkInterfaceIpConfig `
  -Name dsIp4Config `
  -Subnet $vnet.subnets[0] `
  -PrivateIpAddressVersion IPv4 `
  -LoadBalancerBackendAddressPool $backendPoolv4 `
  -PublicIpAddress  $RdpPublicIP_2

# Create NIC 2 reusing the IPv6 configuration from NIC 1
$NIC_2 = New-AzNetworkInterface `
  -Name "dsNIC2" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -NetworkSecurityGroupId $nsg.Id `
  -IpConfiguration $Ip4Config,$Ip6Config

```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Establezca un nombre de usuario de administrador y una contraseña para las máquinas virtuales con [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential):

```azurepowershell
$cred = get-credential -Message "DUAL STACK VNET SAMPLE:  Please enter the Administrator credential to log into the VM's"
```

Ahora puede crear las máquinas virtuales con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente, se crean dos máquinas virtuales y los componentes de red virtual necesarios, si aún no existen.

```azurepowershell
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
```
## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Visualización de la red virtual de doble pila IPv6 en Azure Portal
Para ver la red virtual de doble pila IPv6 en Azure Portal, siga estos pasos:
1. En la barra de búsqueda del portal, escriba *dsVnet*.
2. Seleccione **dsVnet** cuando aparezca en los resultados de búsqueda. De este modo, se abrirá la **página de información general** de la red virtual de doble pila llamada *dsVnet*. En la red virtual de doble pila, se muestran las dos NIC con las configuraciones IPv4 e IPv6 en una subred de pila doble denominada *dsSubnet*.

![Red virtual de doble pila IPv6 con equilibrador de carga interno estándar](./media/ipv6-dual-stack-standard-internal-load-balancer-powershell/ipv6-dual-stack-virtual-network.png)


> [!NOTE]
> En esta versión preliminar, la dirección IPv6 de la red virtual de Azure está disponible en Azure Portal en modo de solo lectura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurepowershell
Remove-AzResourceGroup -Name dsStd_ILB_RG
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado una instancia de Standard Load Balancer con una configuración IP de front-end doble (IPv4 e IPv6). También ha creado dos máquinas virtuales que contienen NIC con configuraciones de IP duales (IPV4 + IPv6) que se agregaron al grupo de back-end del equilibrador de carga. Para más información sobre la compatibilidad de IPv6 en las redes virtuales de Azure, consulte [¿Qué es IPv6 para Azure Virtual Network?](ipv6-overview.md)
