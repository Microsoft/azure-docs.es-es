---
title: 'Azure VPN Gateway: Creación de puerta de enlace basada en rutas: PowerShell'
description: Use PowerShell para crear rápidamente una instancia de Azure VPN Gateway basada en rutas para una conexión VPN a una red local o para conectar redes virtuales.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 8b6ab5dcbd38925afbd98381e427426d27110a53
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393520"
---
# <a name="create-a-route-based-vpn-gateway-using-powershell"></a>Creación de una instancia de VPN Gateway basada en rutas mediante PowerShell

En este artículo se explica cómo crear rápidamente una instancia de Azure VPN Gateway basada en rutas mediante PowerShell. Una instancia de VPN Gateway se usa al crear una conexión VPN a la red local. También puede utilizar una instancia de VPN Gateway para conectar redes virtuales.

## <a name="before-you-begin"></a>Antes de empezar

Los pasos que se describen en este artículo crearán una red virtual, una subred, una subred de puerta de enlace y una instancia de VPN Gateway basada en rutas (puerta de enlace de red virtual). Una vez completada la creación de la puerta de enlace, puede crear las conexiones. Estos pasos requieren una suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos. Si ejecuta PowerShell de manera local, abra la consola de PowerShell con privilegios elevados y conéctese a Azure con el comando `Connect-AzAccount`.

```azurepowershell-interactive
New-AzResourceGroup -Name TestRG1 -Location EastUS
```

## <a name="create-a-virtual-network"></a><a name="vnet"></a>Creación de una red virtual

Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el siguiente ejemplo se crea una red virtual denominada **VNet1** en la ubicación **EastUS**:

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName TestRG1 `
  -Location EastUS `
  -Name VNet1 `
  -AddressPrefix 10.1.0.0/16
```

Cree una configuración de subred mediante el cmdlet [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig).

```azurepowershell-interactive
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Frontend `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

Establezca la configuración de la subred para la red virtual mediante el cmdlet [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork).


```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="add-a-gateway-subnet"></a><a name="gwsubnet"></a>Incorporación de una subred de puerta de enlace

La subred de puerta de enlace contiene las direcciones IP reservadas que usan los servicios de puerta de enlace de la red virtual. Utilice los siguientes ejemplos para agregar una subred de puerta de enlace:

Establezca una variable para la red virtual.

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork -ResourceGroupName TestRG1 -Name VNet1
```

Cree la subred de puerta de enlace mediante el cmdlet [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Add-azVirtualNetworkSubnetConfig).

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27 -VirtualNetwork $vnet
```

Establezca la configuración de la subred para la red virtual mediante el cmdlet [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork).

```azurepowershell-interactive
$vnet | Set-AzVirtualNetwork
```

## <a name="request-a-public-ip-address"></a><a name="PublicIP"></a>Solicitud de una dirección IP pública

Una instancia de VPN Gateway debe tener una dirección IP pública asignada de forma dinámica. Cuando crea una conexión a una instancia de VPN Gateway, esta es la dirección IP que especifica. Utilice el siguiente ejemplo para solicitar una dirección IP pública:

```azurepowershell-interactive
$gwpip= New-AzPublicIpAddress -Name VNet1GWIP -ResourceGroupName TestRG1 -Location 'East US' -AllocationMethod Dynamic
```

## <a name="create-the-gateway-ip-address-configuration"></a><a name="GatewayIPConfig"></a>Creación de la configuración de direcciones IP de la puerta de enlace

La configuración de puerta de enlace define la subred y la dirección IP pública. Use el ejemplo siguiente para crear la configuración de la puerta de enlace:

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork -Name VNet1 -ResourceGroupName TestRG1
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
```
## <a name="create-the-vpn-gateway"></a><a name="CreateGateway"></a>Creación de la puerta de enlace de VPN

Una instancia de VPN Gateway puede tardar 45 minutos o más en crearse. Una vez completada la puerta de enlace, puede crear una conexión entre su red virtual y otra red virtual. O bien, cree una conexión entre su red virtual y una ubicación local. Cree una puerta de enlace de VPN mediante el cmdlet [New-AzVirtualNetworkGateway](/powershell/module/az.network/New-azVirtualNetworkGateway).

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'East US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased -GatewaySku VpnGw1
```

## <a name="view-the-vpn-gateway"></a><a name="viewgw"></a>Visualización de VPN Gateway

Puede ver la instancia de VPN Gateway mediante el cmdlet [Get-AzVirtualNetworkGateway](/powershell/module/az.network/Get-azVirtualNetworkGateway).

```azurepowershell-interactive
Get-AzVirtualNetworkGateway -Name Vnet1GW -ResourceGroup TestRG1
```

El resultado será similar al siguiente ejemplo:

```
Name                   : VNet1GW
ResourceGroupName      : TestRG1
Location               : eastus
Id                     : /subscriptions/<subscription ID>/resourceGroups/TestRG1/provide
                         rs/Microsoft.Network/virtualNetworkGateways/VNet1GW
Etag                   : W/"0952d-9da8-4d7d-a8ed-28c8ca0413"
ResourceGuid           : dc6ce1de-2c4494-9d0b-20b03ac595
ProvisioningState      : Succeeded
Tags                   :
IpConfigurations       : [
                           {
                             "PrivateIpAllocationMethod": "Dynamic",
                             "Subnet": {
                               "Id": "/subscriptions/<subscription ID>/resourceGroups/Te
                         stRG1/providers/Microsoft.Network/virtualNetworks/VNet1/subnets/GatewaySubnet"
                             },
                             "PublicIpAddress": {
                               "Id": "/subscriptions/<subscription ID>/resourceGroups/Te
                         stRG1/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP"
                             },
                             "Name": "default",
                             "Etag": "W/\"0952d-9da8-4d7d-a8ed-28c8ca0413\"",
                             "Id": "/subscriptions/<subscription ID>/resourceGroups/Test
                         RG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/de
                         fault"
                           }
                         ]
GatewayType            : Vpn
VpnType                : RouteBased
EnableBgp              : False
ActiveActive           : False
GatewayDefaultSite     : null
Sku                    : {
                           "Capacity": 2,
                           "Name": "VpnGw1",
                           "Tier": "VpnGw1"
                         }
VpnClientConfiguration : null
BgpSettings            : {
     
```

## <a name="view-the-public-ip-address"></a><a name="viewgwpip"></a>Visualización de la dirección IP pública

Para ver la dirección IP pública de su instancia de VPN Gateway, utilice el cmdlet [Get-AzPublicIpAddress](/powershell/module/az.network/Get-azPublicIpAddress).

```azurepowershell-interactive
Get-AzPublicIpAddress -Name VNet1GWIP -ResourceGroupName TestRG1
```

En la respuesta de ejemplo, el valor de IpAddress es la dirección IP pública.

```
Name                     : VNet1GWIP
ResourceGroupName        : TestRG1
Location                 : eastus
Id                       : /subscriptions/<subscription ID>/resourceGroups/TestRG1/provi
                           ders/Microsoft.Network/publicIPAddresses/VNet1GWIP
Etag                     : W/"5001666a-bc2a-484b-bcf5-ad488dabd8ca"
ResourceGuid             : 3c7c481e-9828-4dae-abdc-f95b383
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Dynamic
IpAddress                : 13.90.153.3
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                             "Id": "/subscriptions/<subscription ID>/resourceGroups/Test
                           RG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/
                           default"
                           }
DnsSettings              : null
Zones                    : {}
Sku                      : {
                             "Name": "Basic"
                           }
IpTags                   : {}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, use el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para eliminar el grupo de recursos. Se eliminarán el grupo de recursos y todos los recursos que contiene.

```azurepowershell-interactive
Remove-AzResourceGroup -Name TestRG1
```

## <a name="next-steps"></a>Pasos siguientes

Una vez creada la puerta de enlace, puede crear una conexión entre su red virtual y otra red virtual. O bien, cree una conexión entre su red virtual y una ubicación local.

> [!div class="nextstepaction"]
> [Creación de una conexión de sitio a sitio](vpn-gateway-create-site-to-site-rm-powershell.md)<br><br>
> [Creación de una conexión de punto a sitio](vpn-gateway-howto-point-to-site-rm-ps.md)<br><br>
> [Creación de una conexión a otra red virtual](vpn-gateway-vnet-vnet-rm-ps.md)
