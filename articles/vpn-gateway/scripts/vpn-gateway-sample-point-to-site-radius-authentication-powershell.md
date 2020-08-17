---
title: 'Script de ejemplo de Azure PowerShell: configuración de VPN de punto a sitio con autenticación de usuario y contraseña RADIUS | Microsoft Docs'
description: Configure una VPN de punto a sitio con la autenticación de usuario y contraseña RADIUS. En este artículo se usa PowerShell.
services: vpn-gateway
documentationcenter: vpn-gateway
author: kumudD
ms.service: vpn-gateway
ms.devlang: powershell
ms.topic: sample
ms.date: 05/30/2018
ms.author: alzam
ms.openlocfilehash: d546fe2568c0c061c9a08e7fa4a98aa8c5e55c55
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068556"
---
# <a name="create-a-vpn-gateway-and-add-point-to-site-configuration-using-powershell"></a>Creación de instancia de VPN Gateway y adición de una configuración de punto a sitio mediante PowerShell

Este script crea una instancia de VPN Gateway basada en enrutamiento y agrega una configuración de punto a sitio mediante la autenticación de nombre de usuario/contraseña RADIUS.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```azurepowershell-interactive
# Declare variables
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $BESubName = "Backend"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "10.0.0.0/16"
  $FESubPrefix = "10.1.0.0/24"
  $BESubPrefix = "10.1.1.0/24"
  $GWSubPrefix = "10.1.255.0/27"
  $VPNClientAddressPool = "192.168.0.0/24"
  $RG = "TestRG1"
  $Location = "East US"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWIP"
  $GWIPconfName = "gwipconf"
# Create a resource group
New-AzResourceGroup -Name TestRG1 -Location EastUS
# Create a virtual network
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName TestRG1 `
  -Location EastUS `
  -Name VNet1 `
  -AddressPrefix 10.1.0.0/16
# Create a subnet configuration
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Frontend `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork
# Set the subnet configuration for the virtual network
$virtualNetwork | Set-AzVirtualNetwork
# Add a gateway subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName TestRG1 -Name VNet1
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27 -VirtualNetwork $vnet
# Set the subnet configuration for the virtual network
$vnet | Set-AzVirtualNetwork
# Request a public IP address
$gwpip= New-AzPublicIpAddress -Name VNet1GWIP -ResourceGroupName TestRG1 -Location 'East US' `
 -AllocationMethod Dynamic
# Create the gateway IP address configuration
$vnet = Get-AzVirtualNetwork -Name VNet1 -ResourceGroupName TestRG1
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
# Create the VPN gateway
New-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
 -Location 'East US' -IpConfigurations $gwipconfig -GatewayType Vpn `
 -VpnType RouteBased -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
# Create a secure string for the RADIUS secret
$Secure_Secret=Read-Host -AsSecureString -Prompt "RadiusSecret"

# Add the VPN client address pool and the RADIUS server information
$Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
 -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol @( "SSTP", "IkeV2" ) `
 -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, use el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para eliminar el grupo de recursos. Se eliminarán el grupo de recursos y todos los recursos que contiene.

```azurepowershell-interactive
Remove-AzResourceGroup -Name TestRG1
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Agrega una configuración de subred. Esta configuración se utiliza con el proceso de creación de la red virtual. |
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtiene detalles de una red virtual. |
| [Get-AzVirtualNetworkGateway](/powershell/module/az.network/get-azvirtualnetworkgateway) | Obtiene detalles de una puerta de enlace de red virtual. |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtiene detalles de configuración de la subred de red virtual. |
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea una configuración de subred. Esta configuración se utiliza con el proceso de creación de la red virtual. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública. |
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crea una configuración de IP de puerta de enlace. |
| [New-AzVirtualNetworkGateway](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworkgateway) | Crea una puerta de enlace de VPN. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Establece la configuración de subred para la red virtual. |
| [Set-AzVirtualNetworkGateway](/powershell/module/az.network/set-azvirtualnetworkgateway) | Establece la configuración de la puerta de enlace VPN. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
