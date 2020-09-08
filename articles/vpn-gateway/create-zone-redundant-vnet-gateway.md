---
title: Crear una puerta de enlace de red virtual con redundancia de zona en Azure Availability Zones
description: Obtenga información sobre cómo implementar puertas de enlace de VPN con redundancia de zona y puertas de enlace de ExpressRoute en Azure Availability Zones.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/10/2020
ms.author: cherylmc
ms.openlocfilehash: 1404f5201ff5f543e34c6a4c8e5713cab3a4c51b
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88033485"
---
# <a name="create-a-zone-redundant-virtual-network-gateway-in-azure-availability-zones"></a>Crear una puerta de enlace de red virtual con redundancia de zona en Azure Availability Zones

Puede implementar puertas de enlace de VPN Gateway y ExpressRoute en Azure Availability Zones. Esto aporta una mayor disponibilidad, escalabilidad y resistencia a las puertas de enlace de red virtual. Implementar puertas de enlace en Azure Availability Zones separa de forma física y lógica las puertas de enlace dentro de una región, y protege la conectividad de red local en Azure de errores de nivel de zona. Para obtener más información, consulte [About zone-redundant virtual network gateways](about-zone-redundant-vnet-gateways.md) (Acerca de las puertas de enlace de red virtual con redundancia de zona) y [About Azure Availability Zones](../availability-zones/az-overview.md) (Acerca de Azure Availability Zones).

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="1-declare-your-variables"></a><a name="variables"></a>1. Declaración de las variables

Declare las variables que desea utilizar. Use el ejemplo siguiente y sustituya los valores por los suyos propios cuando sea necesario. Si cierra la sesión de PowerShell/Cloud Shell en cualquier momento durante el ejercicio, copie y pegue los valores de nuevo para volver a declarar las variables. Cuando especifique la ubicación, compruebe que se admite la región que especifique. Para más información, consulte las [preguntas más frecuentes](#faq).

```azurepowershell-interactive
$RG1         = "TestRG1"
$VNet1       = "VNet1"
$Location1   = "CentralUS"
$FESubnet1   = "FrontEnd"
$BESubnet1   = "Backend"
$GwSubnet1   = "GatewaySubnet"
$VNet1Prefix = "10.1.0.0/16"
$FEPrefix1   = "10.1.0.0/24"
$BEPrefix1   = "10.1.1.0/24"
$GwPrefix1   = "10.1.255.0/27"
$Gw1         = "VNet1GW"
$GwIP1       = "VNet1GWIP"
$GwIPConf1   = "gwipconf1"
```

## <a name="2-create-the-virtual-network"></a><a name="configure"></a>2. Crear la red virtual

Cree un grupo de recursos.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName $RG1 -Location $Location1
```

Cree una red virtual.

```azurepowershell-interactive
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubnet1 -AddressPrefix $FEPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubnet1 -AddressPrefix $BEPrefix1
$vnet = New-AzVirtualNetwork -Name $VNet1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNet1Prefix -Subnet $fesub1,$besub1
```

## <a name="3-add-the-gateway-subnet"></a><a name="gwsub"></a>3. Adición de la subred de puerta de enlace

La subred de puerta de enlace contiene las direcciones IP reservadas que usan los servicios de puerta de enlace de la red virtual. Utilice los siguientes ejemplos para agregar y establecer una subred de puerta de enlace:

Agregue la subred de puerta de enlace.

```azurepowershell-interactive
$getvnet = Get-AzVirtualNetwork -ResourceGroupName $RG1 -Name VNet1
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27 -VirtualNetwork $getvnet
```

Establezca la configuración de subred de puerta de enlace para la red virtual.

```azurepowershell-interactive
$getvnet | Set-AzVirtualNetwork
```
## <a name="4-request-a-public-ip-address"></a><a name="publicip"></a>4. Solicitar una dirección IP pública
 
En este paso, seleccione las instrucciones que se aplican a la puerta de enlace que desea crear. La selección de las zonas para la implementación de las puertas de enlace depende de las zonas especificadas para la dirección IP pública.

### <a name="for-zone-redundant-gateways"></a><a name="ipzoneredundant"></a>Para las puertas de enlace con redundancia de zona

Solicite una dirección IP pública con una SKU de IP pública **Estándar** y no especifique ninguna zona. En este caso, la dirección IP pública Estándar creada será una IP pública con redundancia de zona.   

```azurepowershell-interactive
$pip1 = New-AzPublicIpAddress -ResourceGroup $RG1 -Location $Location1 -Name $GwIP1 -AllocationMethod Static -Sku Standard
```

### <a name="for-zonal-gateways"></a><a name="ipzonalgw"></a>Para las puertas de enlace zonales

Solicite una dirección IP pública con una SKU de IP pública **Estándar**. Especifique la zona (1, 2 ó 3). Todas las instancias de puerta de enlace se implementarán en esta zona.

```azurepowershell-interactive
$pip1 = New-AzPublicIpAddress -ResourceGroup $RG1 -Location $Location1 -Name $GwIP1 -AllocationMethod Static -Sku Standard -Zone 1
```

### <a name="for-regional-gateways"></a><a name="ipregionalgw"></a>Para las puertas de enlace regionales

Solicite una dirección IP pública con una SKU de IP pública **Básica**. En este caso, la puerta de enlace se implementa como una puerta de enlace regional y no se agrega redundancia de zona a la puerta de enlace. Las instancias de puerta de enlace se crean en las zonas respectivas.

```azurepowershell-interactive
$pip1 = New-AzPublicIpAddress -ResourceGroup $RG1 -Location $Location1 -Name $GwIP1 -AllocationMethod Dynamic -Sku Basic
```
## <a name="5-create-the-ip-configuration"></a><a name="gwipconfig"></a>5. Creación de la configuración de IP

```azurepowershell-interactive
$getvnet = Get-AzVirtualNetwork -ResourceGroupName $RG1 -Name $VNet1
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $GwSubnet1 -VirtualNetwork $getvnet
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GwIPConf1 -Subnet $subnet -PublicIpAddress $pip1
```

## <a name="6-create-the-gateway"></a><a name="gwconfig"></a>6. Creación de la puerta de enlace

Cree la puerta de enlace de red virtual.

### <a name="for-expressroute"></a>Para ExpressRoute

```azurepowershell-interactive
New-AzVirtualNetworkGateway -ResourceGroup $RG1 -Location $Location1 -Name $Gw1 -IpConfigurations $GwIPConf1 -GatewayType ExpressRoute -GatewaySku ErGw1AZ
```

### <a name="for-vpn-gateway"></a>Para VPN Gateway

```azurepowershell-interactive
New-AzVirtualNetworkGateway -ResourceGroup $RG1 -Location $Location1 -Name $Gw1 -IpConfigurations $GwIPConf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1AZ
```

## <a name="faq"></a><a name="faq"></a>P+F

### <a name="what-will-change-when-i-deploy-these-new-skus"></a>¿Qué cambiará cuando proceda a implementar estas SKU nuevas?

Desde su perspectiva, puede implementar puertas de enlace con redundancia de zona. Esto significa que todas las instancias de las puertas de enlace se implementarán en distintas zonas de disponibilidad de Azure, y cada zona de disponibilidad es un dominio de error y actualización diferente. Por ello, las puertas de enlace son más confiables, disponibles y resistentes a los errores de la zona.

### <a name="can-i-use-the-azure-portal"></a>¿Puedo usar Azure Portal?

Sí, puede usar Azure Portal para implementar las nuevas SKU. Sin embargo, verá estas nuevas SKU solo en las regiones de Azure que tengan Azure Availability Zones.

### <a name="what-regions-are-available-for-me-to-use-the-new-skus"></a>¿Qué regiones están disponibles para usar las nuevas SKU?

Consulte [Zonas de disponibilidad](../availability-zones/az-region.md) para obtener la lista más reciente de las regiones disponibles.

### <a name="can-i-changemigrateupgrade-my-existing-virtual-network-gateways-to-zone-redundant-or-zonal-gateways"></a>¿Puedo convertir/migrar/actualizar mis puertas de enlace de red virtual existentes en puertas de enlace zonales o con redundancia de zona?

Actualmente no se permite migrar las puertas de enlace de red virtual existentes a puertas de enlace zonales o con redundancia de zona. Sin embargo, puede eliminar la puerta de enlace existente y volver a crear una puerta de enlace zonal o con redundancia de zona.

### <a name="can-i-deploy-both-vpn-and-express-route-gateways-in-same-virtual-network"></a>¿Puedo implementar puertas de enlace de VPN y ExpressRoute en la misma red virtual?

Se admite la coexistencia de puertas de enlace de VPN y ExpressRoute en la misma red virtual. Sin embargo, le recomendamos que reserve un intervalo de direcciones IP /27 para la subred de puerta de enlace.
