---
title: Comandos comunes de PowerShell para Azure Virtual Network
description: Comandos de red comunes de PowerShell para empezar a crear una red virtual y sus recursos asociados para máquinas virtuales.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/17/2017
ms.author: cynthn
ms.openlocfilehash: b4d6b20e63c42616aad0f8776fae159a0f2aa455
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088383"
---
# <a name="common-powershell-commands-for-azure-virtual-networks"></a>Comandos comunes de PowerShell para Azure Virtual Network

Si desea crear una máquina virtual, debe crear una [red virtual](../../virtual-network/virtual-networks-overview.md) o conocer una red virtual existente en la que pueda agregar la máquina virtual. Normalmente, cuando se crea una máquina virtual, también hay que plantearse crear los recursos que se describen en este artículo.

Consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para obtener más información sobre cómo instalar la versión más reciente de Azure PowerShell, seleccionar la suscripción que quiere usar e iniciar sesión en su cuenta.

Algunas de las variables podrían serle útiles si ejecuta más de uno de los comandos de este artículo:

- $location: la ubicación de los recursos de red. Puede usar [Get-AzLocation](/powershell/module/az.resources/get-azlocation) para buscar una [región geográfica](https://azure.microsoft.com/regions/) que se adapte a sus necesidades.
- $myResourceGroup: el nombre del grupo de recursos donde se encuentran los recursos de red.

## <a name="create-network-resources"></a>Crear recursos de red

| Tarea | Get-Help |
| ---- | ------- |
| Crear configuraciones de subred |$subnet1 = [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) -Name "mySubnet1" -AddressPrefix XX.X.X.X/XX<BR>$subnet2 = New-AzVirtualNetworkSubnetConfig -Name "mySubnet2" -AddressPrefix XX.X.X.X/XX<BR><BR>Una red típica podría tener una subred para un [equilibrador de carga accesible desde Internet](../../load-balancer/load-balancer-overview.md) y una subred independiente para un [equilibrador de carga interno](../../load-balancer/load-balancer-overview.md). |
| Creación de una red virtual |$vnet = [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup -Location $location -AddressPrefix XX.X.X.X/XX -Subnet $subnet1, $subnet2 |
| Prueba para un nombre de dominio único |[Test-AzDnsAvailability](/powershell/module/az.network/test-azdnsavailability) -DomainNameLabel "myDNS" -Location $location<BR><BR>Puede especificar un nombre de dominio DNS para un [recurso de dirección IP pública](../../virtual-network/public-ip-addresses.md), lo que crea una asignación para nombrededominio.ubicación.cloudapp.azure.com en la dirección IP pública de los servidores DNS que administra Azure. El nombre solo puede contener letras, números y guiones. El primer y último carácter debe ser una letra o un número y el nombre de dominio debe ser único dentro de su ubicación de Azure. Si se devuelve **True** , significa que el nombre propuesto es único globalmente. |
| Crear una dirección IP pública |$pip = [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) -Name "myPublicIp" -ResourceGroupName $myResourceGroup -DomainNameLabel "myDNS" -Location $location -AllocationMethod Dynamic<BR><BR>La dirección IP pública utiliza el nombre de dominio que probó anteriormente y la emplea la configuración de front-end del equilibrador de carga. |
| Crear una configuración de direcciones IP de front-end |$frontendIP = [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) -Name "myFrontendIP" -PublicIpAddress $pip<BR><BR>La configuración de front-end incluye la dirección IP pública que creó anteriormente para el tráfico de red entrante. |
| Creación de un grupo de direcciones de back-end |$beAddressPool = [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) -Name "myBackendAddressPool"<BR><BR>Proporciona direcciones internas para el back-end del equilibrador de carga al que se accede mediante una interfaz de red. |
| Elaboración de un sondeo |$healthProbe = [New-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) -Name "myProbe" -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>Contiene las sondas de estado utilizadas para comprobar la disponibilidad de las instancias de máquina virtual del grupo de direcciones de back-end. |
| Crear una regla de equilibrio de carga |$lbRule = [New-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>Contiene reglas que asignan un puerto público del equilibrador de carga a un puerto del grupo de direcciones de back-end. |
| Crear una regla NAT entrante |$inboundNATRule = [New-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) -Name "myInboundRule1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>Contiene reglas de asignación de un puerto público del equilibrador de carga a un puerto de una máquina virtual específica del grupo de direcciones de back-end. |
| Creación de un equilibrador de carga |$loadBalancer = [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) -ResourceGroupName $myResourceGroup -Name "myLoadBalancer" -Location $location -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe |
| Crear una interfaz de red |$nic1= [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) -ResourceGroupName $myResourceGroup -Name "myNIC" -Location $location -PrivateIpAddress XX.X.X.X -Subnet $subnet2 -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>Cree una interfaz de red empleando la dirección IP pública y la subred de red virtual que creó anteriormente. |

## <a name="get-information-about-network-resources"></a>Obtención de información sobre recursos de red

| Tarea | Get-Help |
| ---- | ------- |
| Enumerar redes virtuales |[Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) -ResourceGroupName $myResourceGroup<BR><BR>Enumera todas las redes virtuales del grupo de recursos. |
| Obtener información sobre una red virtual |Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup |
| Enumerar subredes de una red virtual |Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup &#124; Select Subnets |
| Obtener información sobre una subred |[Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>Obtiene información sobre la subred de la red virtual especificada. El valor $vnet representa el objeto devuelto por Get-AzVirtualNetwork. |
| Enumerar direcciones IP |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) -ResourceGroupName $myResourceGroup<BR><BR>Enumera las direcciones IP públicas del grupo de recursos. |
| Enumerar equilibradores de carga |[Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) -ResourceGroupName $myResourceGroup<BR><BR>Enumera todos los equilibradores de carga del grupo de recursos. |
| Enumerar interfaces de red |[Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) -ResourceGroupName $myResourceGroup<BR><BR>Enumera todas las interfaces de red del grupo de recursos. |
| Obtener información sobre una interfaz de red |Get-AzNetworkInterface -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>Obtiene información sobre una interfaz de red específica. |
| Obtener la configuración de dirección IP de una interfaz de red |[Get-AzNetworkInterfaceIPConfig](/powershell/module/az.network/get-aznetworkinterfaceipconfig) -Name "myNICIP" -NetworkInterface $nic<BR><BR>Obtiene información sobre la configuración de dirección IP de la interfaz de red especificada. El valor $nic representa el objeto devuelto por Get-AzNetworkInterface. |

## <a name="manage-network-resources"></a>Administración de recursos de red

| Tarea | Get-Help |
| ---- | ------- |
| Agregar una red a una red virtual |[Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) -AddressPrefix XX.X.X.X/XX -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>Agrega una subred a una red virtual existente. El valor $vnet representa el objeto devuelto por Get-AzVirtualNetwork. |
| Eliminar una red virtual |[Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup<BR><BR>Quita la red virtual especificada del grupo de recursos. |
| Eliminar una interfaz de red |[Remove-AzNetworkInterface](/powershell/module/az.network/remove-aznetworkinterface) -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>Quita la interfaz de red especificada del grupo de recursos. |
| Eliminar un equilibrador de carga |[Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer) -Name "myLoadBalancer" -ResourceGroupName $myResourceGroup<BR><BR>Quita el equilibrador de carga especificado del grupo de recursos. |
| Eliminar una dirección IP pública |[Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress)-Name "myIPAddress" -ResourceGroupName $myResourceGroup<BR><BR>Quita la dirección IP pública especificada del grupo de recursos. |

## <a name="next-steps"></a>Pasos siguientes
Utilice la interfaz de red que acaba de crear cuando [cree una VM](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json).
