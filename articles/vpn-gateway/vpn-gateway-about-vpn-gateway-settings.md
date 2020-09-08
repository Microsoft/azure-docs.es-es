---
title: 'Azure VPN Gateway: opciones de configuración'
description: Obtenga información sobre los recursos de VPN Gateway y sobre la configuración de una red virtual creada en el modelo de implementación de Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 01/10/2020
ms.author: cherylmc
ms.openlocfilehash: 6f801b94a16ae6fcee6896dd70510469c5e20098
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036919"
---
# <a name="about-vpn-gateway-configuration-settings"></a>Acerca de la configuración de VPN Gateway

Una puerta de enlace de VPN es un tipo de puerta de enlace de red virtual que envía tráfico cifrado entre la red virtual y la ubicación local a través de una conexión pública. También puede utilizar una puerta de enlace de VPN para enviar tráfico entre redes virtuales a través de la red troncal de Azure.

Una conexión de puerta de enlace de VPN se basa en la configuración de varios recursos, cada uno de los cuales contiene valores configurables. Las secciones de este artículo tratan los recursos y la configuración relacionados con una puerta de enlace de VPN para una red virtual creada en el modelo de implementación de Resource Manager. Puede encontrar las descripciones y los diagramas de topología de cada solución de conexión en el artículo [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md).

Los valores de este artículo se aplican a las puertas de enlace de VPN (puertas de enlace de red virtual que usan -GatewayType Vpn). En este artículo no se cubren todos los tipos de puerta de enlace ni puertas de enlace con redundancia de zona.

* Para los valores que se aplican a -GatewayType "ExpressRoute", consulte [Acerca de las puertas de enlace de red virtual para ExpressRoute](../expressroute/expressroute-about-virtual-network-gateways.md).

* Para las puertas de enlace con redundancia de zona, consulte [Acerca de VPN Gateway](about-zone-redundant-vnet-gateways.md).

* Para Virtual WAN, consulte [Acerca de Virtual WAN](../virtual-wan/virtual-wan-about.md).



## <a name="gateway-types"></a><a name="gwtype"></a>Tipos de puerta de enlace

Cada red virtual solo puede tener una puerta de enlace de red de cada tipo. Al crear una puerta de enlace de red virtual, debe asegurarse de que el tipo de puerta de enlace es el correcto para su configuración.

Los valores disponibles para -GatewayType son:

* VPN
* ExpressRoute

Una puerta de enlace de VPN requiere la *red privada virtual* `-GatewayType`.

Ejemplo:

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased
```

## <a name="gateway-skus"></a><a name="gwsku"></a>SKU de puerta de enlace

[!INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

### <a name="configure-a-gateway-sku"></a>Configuración de una SKU de puerta de enlace

#### <a name="azure-portal"></a>Portal de Azure

Si usa Azure Portal para crear una puerta de enlace de red virtual de Resource Manager, puede seleccionar la SKU de la puerta de enlace con el menú desplegable. Las opciones que se presentan corresponden con el tipo de puerta de enlace y tipo de VPN que seleccione.

#### <a name="powershell"></a>PowerShell

En el siguiente ejemplo de PowerShell se especifica `-GatewaySku` como VpnGw1. Al usar PowerShell para crear una puerta de enlace, antes debe crear la configuración de IP y, después, usar una variable para hacer referencia a ella. En este ejemplo, la variable de configuración es $gwipconfig.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'US East' -IpConfigurations $gwipconfig -GatewaySku VpnGw1 `
-GatewayType Vpn -VpnType RouteBased
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWPIP --resource-group TestRG1 --vnet VNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait
```

###  <a name="resizing-or-changing-a-sku"></a><a name="resizechange"></a>Cambio de tamaño o de SKU

Si tiene una puerta de enlace VPN y desea usar una SKU de puerta de enlace distinta, las opciones son o cambiar el tamaño de la SKU de puerta de enlace o cambiar a otra SKU. Al cambiar a otra SKU de puerta de enlace, se elimina completamente la puerta de enlace existente y se crea otra. La creación de una puerta de enlace puede tardar hasta 45 minutos. En cambio, al cambiar el tamaño de la SKU de puerta de enlace, el tiempo de inactividad será corto, ya que no tiene que eliminar y volver crear la puerta de enlace. Si tiene la opción de cambiar el tamaño de la SKU de puerta de enlace, en lugar de cambiarla, aprovéchela. Sin embargo, hay reglas en relación con el cambio de tamaño:

1. A excepción de la SKU básica, puede cambiar el tamaño de una SKU de VPN Gateway a otra SKU de VPN Gateway dentro de la misma generación (Generation1 o Generation2). Por ejemplo, se puede cambiar el tamaño de VpnGw1 de Generation1 a VpnGw2 de Generation1, pero no a VpnGw2 de Generation2.
2. Si trabaja con las SKU de puerta de enlace antiguas, puede cambiar el tamaño entre las SKU Básica, Estándar y HighPerformance.
3. Sin embargo, **no puede** cambiar el tamaño de las SKU de Básica/Estándar/HighPerformance a las SKU de VpnGw. En su lugar, debe [cambiar](#change) a las SKU nuevas.

#### <a name="to-resize-a-gateway"></a><a name="resizegwsku"></a>Cambiar el tamaño de una puerta de enlace

[!INCLUDE [Resize a SKU](../../includes/vpn-gateway-gwsku-resize-include.md)]

####  <a name="to-change-from-an-old-legacy-sku-to-a-new-sku"></a><a name="change"></a>Cambiar de una SKU anterior (heredada) anterior a una nueva

[!INCLUDE [Change a SKU](../../includes/vpn-gateway-gwsku-change-legacy-sku-include.md)]

## <a name="connection-types"></a><a name="connectiontype"></a>Tipos de conexión

En el modelo de implementación de Resource Manager, cada configuración requiere un tipo de conexión de puerta de enlace de red virtual específico. Los valores de PowerShell de Resource Manager para `-ConnectionType` son:

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

En el siguiente ejemplo de PowerShell, vamos a crear una conexión de S2S que requiere el tipo de conexión *IPsec*.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
```

## <a name="vpn-types"></a><a name="vpntype"></a>Tipos de VPN

Al crear la puerta de enlace de red virtual para una configuración de puerta de enlace de VPN, debe especificar un tipo de VPN. El tipo de VPN que elija dependerá de la topología de conexión que desee crear. Por ejemplo, una conexión de P2S requiere un tipo de VPN RouteBased. Un tipo de VPN también puede depender del hardware que se esté usando. Las configuraciones de S2S requieren un dispositivo VPN. Algunos dispositivos VPN solo serán compatibles con un determinado tipo de VPN.

El tipo de VPN que seleccione debe cumplir todos los requisitos de conexión de la solución que desea crear. Por ejemplo, si desea crear una conexión de puerta de enlace de VPN de S2S y una conexión de puerta de enlace de VPN de P2S para la misma red virtual, debería usar el tipo de VPN *RouteBased* , dado que P2S requiere un tipo de VPN RouteBased. También necesitaría comprobar que el dispositivo VPN admite una conexión de VPN RouteBased. 

Una vez que se ha creado una puerta de enlace de red virtual, no puede cambiar el tipo de VPN. Tendrá que eliminar la puerta de enlace de red virtual y crear una nueva. Hay dos tipos de VPN:

[!INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

En el siguiente ejemplo de PowerShell se especifica `-VpnType` como *RouteBased*. Al crear una puerta de enlace, debe asegurarse de que el tipo de VPN es el correcto para su configuración.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig `
-GatewayType Vpn -VpnType RouteBased
```

## <a name="gateway-requirements"></a><a name="requirements"></a>Requisitos de la puerta de enlace

[!INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]

## <a name="gateway-subnet"></a><a name="gwsub"></a>Subred de puerta de enlace

Antes de crear una puerta de enlace de VPN, debe crear una subred de puerta de enlace. La subred de puerta de enlace contiene las direcciones IP que usan los servicios y las máquinas virtuales de la puerta de enlace de red virtual. Al crear la puerta de enlace de red virtual, las máquinas virtuales de puerta de enlace se implementan en la subred de puerta de enlace, y se configuran con las opciones de puerta de enlace de VPN necesarias. Nunca implemente nada más (por ejemplo, máquinas virtuales adicionales) en la subred de puerta de enlace. Para que la subred de puerta de enlace funcione correctamente, su nombre tiene que ser “GatewaySubnet2”. La asignación del nombre "GatewaySubnet" a la subred de puerta de enlace permite a Azure saber que se trata de la subred donde se implementarán las máquinas virtuales y los servicios de la puerta de enlace de red virtual.

>[!NOTE]
>[!INCLUDE [vpn-gateway-gwudr-warning.md](../../includes/vpn-gateway-gwudr-warning.md)]
>

Al crear la subred de puerta de enlace, especifique el número de direcciones IP que contiene la subred. Las direcciones IP de la subred de puerta de enlace se asignan a las máquinas virtuales y los servicios de puerta de enlace. Algunas configuraciones requieren más direcciones IP que otras. 

Cuando planee el tamaño de la subred de puerta de enlace, consulte la documentación de la configuración que piensa crear. Por ejemplo, la configuración de coexistencia de ExpressRoute/VPN Gateway requiere una subred de puerta de enlace mayor que la mayoría de las restantes. Debe asegurarse también de que la subred de puerta de enlace contenga suficientes direcciones IP para acomodar posibles configuraciones adicionales. Aunque es posible crear una subred de puerta de enlace solo de /29, se recomienda que sea de /27, o incluso mayor, (/27, /26, etc.), siempre que se disponga de espacio para hacerlo, ya que puede albergar más configuraciones.

En el ejemplo de PowerShell de Resource Manager siguiente, se muestra una subred de puerta de enlace con el nombre GatewaySubnet. Puede ver que la notación CIDR especifica /27, que permite suficientes direcciones IP para la mayoría de las configuraciones que existen.

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="local-network-gateways"></a><a name="lng"></a>Puertas de enlace de red local

 Una puerta de enlace de red local es diferente a una puerta de enlace de red virtual. Al crear una configuración de puerta de enlace de VPN, la puerta de enlace de red local suele representar la ubicación local. En el modelo de implementación clásica, la puerta de enlace de red local se conoce como un sitio local.

Debe asignar un nombre a la puerta de enlace de red local, así como la dirección IP pública del dispositivo VPN local, y especificar los prefijos de dirección que se encuentran en la ubicación local. Azure examina los prefijos de dirección de destino para el tráfico de red, consulta la configuración que especificó para la puerta de enlace de red local y enruta los paquetes según corresponda. También debe especificar puertas de enlace de red local para configuraciones de red virtual a red virtual local que usan una conexión de puerta de enlace de VPN.

En el ejemplo siguiente de PowerShell, se crea una nueva puerta de enlace de red local:

```azurepowershell-interactive
New-AzLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'
```

A veces es necesario modificar la configuración de la puerta de enlace de red local. Por ejemplo, al agregar o modificar el intervalo de direcciones, o si cambia la dirección IP del dispositivo VPN. Vea [Modificación de la configuración de la puerta de enlace de red local mediante PowerShell](vpn-gateway-modify-local-network-gateway.md).

## <a name="rest-apis-powershell-cmdlets-and-cli"></a><a name="resources"></a>API de REST, cmdlets de PowerShell y CLI

Para más información sobre los recursos técnicos y los requisitos de sintaxis específicos para usar las API de REST y los cmdlets de PowerShell para configuraciones de VPN Gateway, consulte las páginas siguientes:

| **Clásico** | **Resource Manager** |
| --- | --- |
| [PowerShell](/powershell/module/az.network/#networking) |[PowerShell](/powershell/module/az.network#vpn) |
| [REST API](https://msdn.microsoft.com/library/jj154113) |[REST API](/rest/api/network/virtualnetworkgateways) |
| No compatible | [CLI de Azure](/cli/azure/network/vnet-gateway)|

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las configuraciones de conexión disponibles, vea [About VPN Gateway (Acerca de VPN Gateway)](vpn-gateway-about-vpngateways.md).
