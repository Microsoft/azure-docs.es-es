---
title: 'Tutorial: Creación y administración de una puerta de enlace mediante Azure VPN Gateway'
description: Siga este tutorial para aprender a crear, implementar y administrar Azure VPN Gateway mediante PowerShell.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: tutorial
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 34aa805e6af68933326325587c0a442fe5e3652c
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89420662"
---
# <a name="tutorial-create-and-manage-a-vpn-gateway-using-powershell"></a>Tutorial: Creación y administración de una puerta de enlace de VPN con PowerShell

Las puertas de enlace de VPN de Azure proporcionan conectividad entre locales entre el entorno local del cliente y Azure. En este tutorial se tratan los elementos básicos de implementación de Azure VPN Gateway, como crear y administrar una puerta de enlace de VPN. Aprenderá a:

> [!div class="checklist"]
> * Creación de una puerta de enlace de VPN
> * Visualización de la dirección IP pública
> * Cambio de tamaño de una puerta de enlace de VPN
> * Restablecimiento de una instancia de VPN Gateway

En el siguiente diagrama se muestran la red virtual y la puerta de enlace de VPN creadas como parte de este tutorial.

![VNet y VPN Gateway](./media/vpn-gateway-tutorial-create-gateway-powershell/vnet1-gateway.png)

### <a name="working-with-azure-cloud-shell-and-azure-powershell"></a>Trabajo con Azure Cloud Shell y Azure PowerShell

[!INCLUDE [working with cloud shell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

## <a name="common-network-parameter-values"></a>Valores de parámetros de red comunes

A continuación se muestran los valores de los parámetros utilizados en este tutorial. En los ejemplos, las variables se traducen a lo siguiente:

```
#$RG1         = The name of the resource group
#$VNet1       = The name of the virtual network
#$Location1   = The location region
#$FESubnet1   = The name of the first subnet
#$BESubnet1   = The name of the second subnet
#$VNet1Prefix = The address range for the virtual network
#$FEPrefix1   = Addresses for the first subnet
#$BEPrefix1   = Addresses for the second subnet
#$GwPrefix1   = Addresses for the GatewaySubnet
#$VNet1ASN    = ASN for the virtual network
#$DNS1        = The IP address of the DNS server you want to use for name resolution
#$Gw1         = The name of the virtual network gateway
#$GwIP1       = The public IP address for the virtual network gateway
#$GwIPConf1   = The name of the IP configuration
```

Cambie los valores siguientes, según la configuración de su entorno y red y, luego, copie y pegue las variables correspondientes a este tutorial para definirlas. Si la sesión de Cloud Shell agota el tiempo de espera o debe usar una ventana de PowerShell diferente, copie y pegue las variables en la nueva sesión y siga con el tutorial.

```azurepowershell-interactive
$RG1         = "TestRG1"
$VNet1       = "VNet1"
$Location1   = "East US"
$FESubnet1   = "FrontEnd"
$BESubnet1   = "Backend"
$VNet1Prefix = "10.1.0.0/16"
$FEPrefix1   = "10.1.0.0/24"
$BEPrefix1   = "10.1.1.0/24"
$GwPrefix1   = "10.1.255.0/27"
$VNet1ASN    = 65010
$DNS1        = "8.8.8.8"
$Gw1         = "VNet1GW"
$GwIP1       = "VNet1GWIP"
$GwIPConf1   = "gwipconf1"
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Primero se debe crear un grupo de recursos. En el ejemplo siguiente, se crea un grupo de recursos llamado *TestRG1* en la región *Este de EE. UU.* :

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName $RG1 -Location $Location1
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Azure VPN Gateway proporciona conectividad entre locales y funcionalidad de servidor VPN P2S para la red virtual. Agregue la puerta de enlace VPN a una red virtual existente o cree una nueva red virtual y la puerta de enlace. Observe que el ejemplo especifica el nombre de la subred de la puerta de enlace. Debe especificar siempre el nombre de la subred de puerta de enlace como "GatewaySubnet" para que funcione correctamente. En este ejemplo se crea una red virtual con tres subredes: Frontend, Backend y GatewaySubnet mediante [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) y [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork):

```azurepowershell-interactive
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubnet1 -AddressPrefix $FEPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubnet1 -AddressPrefix $BEPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -AddressPrefix $GwPrefix1
$vnet   = New-AzVirtualNetwork `
            -Name $VNet1 `
            -ResourceGroupName $RG1 `
            -Location $Location1 `
            -AddressPrefix $VNet1Prefix `
            -Subnet $fesub1,$besub1,$gwsub1
```

## <a name="request-a-public-ip-address-for-the-vpn-gateway"></a>Solicitud de una dirección IP pública para la puerta de enlace de VPN

Las puertas de enlace de VPN de Azure se comunican con los dispositivos VPN locales a través de Internet para realizar la negociación IKE (Intercambio de claves por red) y establecer los túneles IPsec. Cree y asigne una dirección IP pública a su puerta de enlace de VPN, como se muestra en el ejemplo siguiente con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) y [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig):

> [!IMPORTANT]
> Actualmente, solo puede usar una dirección IP pública dinámica para la puerta de enlace. No se admite la dirección IP estática en puertas de enlace de VPN de Azure.

```azurepowershell-interactive
$gwpip    = New-AzPublicIpAddress -Name $GwIP1 -ResourceGroupName $RG1 `
              -Location $Location1 -AllocationMethod Dynamic
$subnet   = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' `
              -VirtualNetwork $vnet
$gwipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GwIPConf1 `
              -Subnet $subnet -PublicIpAddress $gwpip
```

## <a name="create-a-vpn-gateway"></a>Creación de una puerta de enlace de VPN

Una instancia de VPN Gateway puede tardar 45 minutos o más en crearse. Una vez completada la creación de la puerta de enlace, puede crear una conexión entre su red virtual y otra red virtual. O bien, puede crear una conexión entre su red virtual y una ubicación local. Cree una puerta de enlace de VPN mediante el cmdlet [New-AzVirtualNetworkGateway](/powershell/module/az.network/New-azVirtualNetworkGateway).

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $Gw1 -ResourceGroupName $RG1 `
  -Location $Location1 -IpConfigurations $gwipconf -GatewayType Vpn `
  -VpnType RouteBased -GatewaySku VpnGw1
```

Valores de parámetros de clave:
* GatewayType: use **Vpn** para las conexiones de sitio a sitio y entre redes virtuales
* VpnType: use **RouteBased** para interactuar con una amplia variedad de dispositivos VPN y más características de enrutamiento
* GatewaySku: **VpnGw1** es el valor predeterminado; cámbielo por otra SKU de VpnGw, si necesita mayor rendimiento o más conexiones. Consulte [SKU de puertas de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku) para más información.

Si usa el TryIt, puede que se agote el tiempo de espera de la sesión. No pasa nada. La puerta de enlace sigue creándose.

Cuando haya completado la creación de la puerta de enlace, puede crear una conexión entre su red virtual y otra red virtual, o bien crear una conexión entre la red virtual y una ubicación local. También puede configurar una conexión P2S a la red virtual desde un equipo cliente.

## <a name="view-the-gateway-public-ip-address"></a>Visualización de la dirección IP pública de la puerta de enlace

Si conoce el nombre de la dirección IP pública, use [Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress) para mostrar la dirección IP pública asignada a la puerta de enlace.

Si la sesión ha agotado el tiempo de espera, copie los parámetros de red comunes que aparecen al principio de este tutorial en la nueva sesión y continúe.

```azurepowershell-interactive
$myGwIp = Get-AzPublicIpAddress -Name $GwIP1 -ResourceGroup $RG1
$myGwIp.IpAddress
```

## <a name="resize-a-gateway"></a>Cambio del tamaño de una puerta de enlace

Puede cambiar la SKU de la puerta de enlace de VPN después de crear la puerta de enlace. Las distintas SKU de puerta de enlace admiten distintas especificaciones, como capacidades de proceso, número de conexiones, etc. En el ejemplo siguiente se usa [Resize-AzVirtualNetworkGateway](/powershell/module/az.network/Resize-azVirtualNetworkGateway) para cambiar el tamaño de la puerta de enlace de VpnGw1 a VpnGw2. Consulte [SKU de puertas de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku) para más información.

```azurepowershell-interactive
$gateway = Get-AzVirtualNetworkGateway -Name $Gw1 -ResourceGroup $RG1
Resize-AzVirtualNetworkGateway -GatewaySku VpnGw2 -VirtualNetworkGateway $gateway
```

Cambiar el tamaño de una puerta de enlace de VPN también tarda entre 30 y 45 minutos, aunque esta operación **no** interrumpe ni quita las conexiones y configuraciones existentes.

## <a name="reset-a-gateway"></a>Restablecimiento de una puerta de enlace

Como parte de los pasos de solución de problemas, puede restablecer la puerta de enlace de VPN de Azure para forzar a la puerta de enlace de VPN a reiniciar las configuraciones de túnel de IPsec/IKE. Use [Reset-AzVirtualNetworkGateway](/powershell/module/az.network/Reset-azVirtualNetworkGateway) para restablecer la puerta de enlace.

```azurepowershell-interactive
$gateway = Get-AzVirtualNetworkGateway -Name $Gw1 -ResourceGroup $RG1
Reset-AzVirtualNetworkGateway -VirtualNetworkGateway $gateway
```

Para más información, consulte [Restablecimiento de una puerta de enlace](vpn-gateway-resetgw-classic.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a avanzar al [siguiente tutorial](vpn-gateway-tutorial-vpnconnection-powershell.md), querrá mantener estos recursos porque son los requisitos previos.

Sin embargo, si la puerta de enlace es parte de la implementación de un prototipo, una evaluación o una prueba de concepto, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la puerta de enlace de VPN y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $RG1
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido conceptos básicos sobre la creación y administración de la puerta de enlace de VPN. Por ejemplo:

> [!div class="checklist"]
> * Creación de una puerta de enlace de VPN
> * Visualización de la dirección IP pública
> * Cambio de tamaño de una puerta de enlace de VPN
> * Restablecimiento de una instancia de VPN Gateway

Avance a los siguientes tutoriales para aprender sobre las conexiones S2S, de red virtual a red virtual y P2S.

> [!div class="nextstepaction"]
> * [Creación de conexiones S2S](vpn-gateway-tutorial-vpnconnection-powershell.md)
> * [Creación de conexiones de red virtual a red virtual](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [Creación de conexiones P2S](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
