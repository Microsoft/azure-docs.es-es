---
title: 'Actualización de una aplicación IPv4 a IPv6 en Azure Virtual Network: PowerShell'
titlesuffix: Azure Virtual Network
description: En este artículo, se explica cómo se implementan direcciones IPv6 en una aplicación existente en Azure Virtual Network utilizando Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: f2ff81c1f1989e28fa48e4307a13433a7b98e915
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051056"
---
# <a name="upgrade-an-ipv4-application-to-ipv6-in-azure-virtual-network---powershell"></a>Actualización de una aplicación IPv4 a IPv6 en Azure Virtual Network: PowerShell

En este artículo se muestra cómo agregar conectividad IPv6 a una aplicación IPv4 existente en una instancia de Azure Virtual Network con una instancia de Standard Load Balancer e IP pública. La actualización local incluye:
- Espacio de direcciones IPv6 para la red virtual y la subred
- Standard Load Balancer con configuraciones de front-end IPv4 e IPV6
- Máquinas virtuales con NIC que tienen una configuración IPv4 + IPv6
- IP pública de IPv6 para que el equilibrador de carga tenga conectividad IPv6 accesible desde Internet



[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, la versión del módulo de Azure PowerShell que necesita este artículo es la 6.9.0 u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por supuesto que implementó una instancia de Standard Load Balancer como se describe en [Inicio rápido: Creación de una instancia de Standard Load Balancer mediante Azure PowerShell](../load-balancer/quickstart-load-balancer-standard-public-powershell.md).

## <a name="retrieve-the-resource-group"></a>Recuperación del grupo de recursos

Para poder generar la red virtual de doble pila, debe recuperar el grupo de recursos con [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup).

```azurepowershell
 $rg = Get-AzResourceGroup  -ResourceGroupName "myResourceGroupSLB"
```

## <a name="create-an-ipv6-ip-addresses"></a>Creación de direcciones IP IPv6

Cree una dirección IPv6 pública con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) para su instancia de Standard Load Balancer. En el ejemplo siguiente, se crea una dirección IP pública IPv6 denominada *PublicIP_v6* en el grupo de recursos *myResourceGroupSLB*:

```azurepowershell
  
  $PublicIP_v6 = New-AzPublicIpAddress `
  -Name "PublicIP_v6" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Sku Standard  `
  -AllocationMethod Static `
  -IpAddressVersion IPv6
```

## <a name="configure-load-balancer-frontend"></a>Configuración del front-end del equilibrador de carga

Recupere la configuración del equilibrador de carga existente y, a continuación, agregue la nueva dirección IP IPv6 mediante [Add-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/Add-AzLoadBalancerFrontendIpConfig) como se indica a continuación:

```azurepowershell
# Retrieve the load balancer configuration
$lb = Get-AzLoadBalancer -ResourceGroupName $rg.ResourceGroupName -Name "MyLoadBalancer"
# Add IPv6 components to the local copy of the load balancer configuration
$lb | Add-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v6" `
  -PublicIpAddress $PublicIP_v6
#Update the running load balancer with the new frontend
$lb | Set-AzLoadBalancer
```

## <a name="configure-load-balancer-backend-pool"></a>Configuración del grupo back-end de equilibradores de carga

Cree el grupo back-end en la copia local de la configuración del equilibrador de carga y actualice el equilibrador de carga en ejecución con la nueva configuración del grupo back-end como se indica a continuación:

```azurepowershell
$lb | Add-AzLoadBalancerBackendAddressPoolConfig -Name "LbBackEndPool_v6"
# Update the running load balancer with the new backend pool
$lb | Set-AzLoadBalancer
```

## <a name="configure-load-balancer-rules"></a>Creación de reglas de equilibrador de carga
Recupere la configuración existente del grupo back-end y front-end de equilibradores de carga y, a continuación, agregue nuevas reglas de equilibrio de carga mediante [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/Add-AzLoadBalancerRuleConfig).

```azurepowershell
# Retrieve the updated (live) versions of the frontend and backend pool
$frontendIPv6 = Get-AzLoadBalancerFrontendIpConfig -Name "dsLbFrontEnd_v6" -LoadBalancer $lb
$backendPoolv6 = Get-AzLoadBalancerBackendAddressPoolConfig -Name "LbBackEndPool_v6" -LoadBalancer $lb
# Create new LB rule with the frontend and backend
$lb | Add-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v6" `
  -FrontendIpConfiguration $frontendIPv6 `
  -BackendAddressPool $backendPoolv6 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80
#Finalize all the load balancer updates on the running load balancer
$lb | Set-AzLoadBalancer
```
## <a name="add-ipv6-address-ranges"></a>Incorporación de intervalos de direcciones IPv6

Agregue intervalos de direcciones IPv6 a la red virtual y a la subred que hospeda las máquinas virtuales como se indica a continuación:

```azurepowershell
#Add IPv6 ranges to the VNET and subnet
#Retreive the VNET object
$vnet = Get-AzVirtualNetwork  -ResourceGroupName $rg.ResourceGroupName -Name "myVnet" 
#Add IPv6 prefix to the VNET
$vnet.addressspace.addressprefixes.add("ace:cab:deca::/48")
#Update the running VNET
$vnet |  Set-AzVirtualNetwork

#Retrieve the subnet object from the local copy of the VNET
$subnet= $vnet.subnets[0]
#Add IPv6 prefix to the Subnet (subnet of the VNET prefix, of course)
$subnet.addressprefix.add("ace:cab:deca::/64")
#Update the running VNET with the new subnet configuration
$vnet |  Set-AzVirtualNetwork

```
## <a name="add-ipv6-configuration-to-nic"></a>Incorporación de la configuración IPv6 a NIC

Configuración de todas las NIC de máquina virtual con una dirección IPv6 mediante [Add-AzNetworkInterfaceIpConfig](/powershell/module/az.network/Add-AzNetworkInterfaceIpConfig) como se indica a continuación:

```azurepowershell

#Retrieve the NIC objects
$NIC_1 = Get-AzNetworkInterface -Name "myNic1" -ResourceGroupName $rg.ResourceGroupName
$NIC_2 = Get-AzNetworkInterface -Name "myNic2" -ResourceGroupName $rg.ResourceGroupName
$NIC_3 = Get-AzNetworkInterface -Name "myNic3" -ResourceGroupName $rg.ResourceGroupName
#Add an IPv6 IPconfig to NIC_1 and update the NIC on the running VM
$NIC_1 | Add-AzNetworkInterfaceIpConfig -Name MyIPv6Config -Subnet $vnet.Subnets[0]  -PrivateIpAddressVersion IPv6 -LoadBalancerBackendAddressPool $backendPoolv6 
$NIC_1 | Set-AzNetworkInterface
#Add an IPv6 IPconfig to NIC_2 and update the NIC on the running VM
$NIC_2 | Add-AzNetworkInterfaceIpConfig -Name MyIPv6Config -Subnet $vnet.Subnets[0]  -PrivateIpAddressVersion IPv6 -LoadBalancerBackendAddressPool $backendPoolv6 
$NIC_2 | Set-AzNetworkInterface
#Add an IPv6 IPconfig to NIC_3 and update the NIC on the running VM
$NIC_3 | Add-AzNetworkInterfaceIpConfig -Name MyIPv6Config -Subnet $vnet.Subnets[0]  -PrivateIpAddressVersion IPv6 -LoadBalancerBackendAddressPool $backendPoolv6 
$NIC_3 | Set-AzNetworkInterface

```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Visualización de la red virtual de doble pila IPv6 en Azure Portal
Para ver la red virtual de doble pila IPv6 en Azure Portal, siga estos pasos:
1. En la barra de búsqueda del portal, escriba *myVnet*.
2. Seleccione **myVnet** cuando aparezca en los resultados de búsqueda. De este modo, se abrirá la **página de información general** de la red virtual de doble pila llamada *myVnet*. En la red virtual de doble pila, se muestran las tres NIC con las configuraciones IPv4 e IPv6 en una subred de pila doble denominada *mySubnet*.

  ![Red virtual de doble pila IPv6 de Azure](./media/ipv6-add-to-existing-vnet-powershell/ipv6-dual-stack-vnet.png)



## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyAzureResourceGroupSLB
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha actualizado una instancia de Standard Load Balancer existente con una configuración de direcciones IP de front-end IPv4 a una configuración de doble pila (IPv4 e IPv6). También ha agregado configuraciones IPv6 a las NIC de las máquinas virtuales en el grupo back-end y a la instancia de Virtual Network que las hospeda. Para más información sobre la compatibilidad de IPv6 en las redes virtuales de Azure, consulte [¿Qué es IPv6 para Azure Virtual Network?](ipv6-overview.md)
