---
title: 'Configuración de conexiones coexistentes de VPN S2S y ExpressRoute: Azure PowerShell'
description: Configure conexiones ExpressRoute y VPN de sitio a sitio que puedan coexistir en el modelo de implementación de Resource Manager con PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/11/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: cc53aa9cab765f16fdf7a517793f8f36ca30daac
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89396189"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-using-powershell"></a>Configuración de conexiones ExpressRoute y de sitio a sitio coexistentes con PowerShell
> [!div class="op_single_selector"]
> * [PowerShell: administrador de recursos](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell: clásico](expressroute-howto-coexist-classic.md)
> 
> 

En este artículo se le ayuda a configurar conexiones ExpressRoute y VPN de sitio a sitio que coexisten. Tener la posibilidad de configurar VPN de sitio a sitio y ExpressRoute tiene varias ventajas. Puede configurar una VPN de sitio a sitio como una ruta de acceso seguro de conmutación por error para ExpressRoute, o bien usar las VPN de sitio a sitio para conectarse a sitios que no están conectados mediante ExpressRoute. En este artículo trataremos los pasos para configurar ambos escenarios. Este artículo se aplica al modelo de implementación de Resource Manager.

La configuración de las conexiones coexistentes de VPN de sitio a sitio y ExpressRoute tiene varias ventajas:

* Puede configurar una VPN de sitio a sitio como una ruta de conmutación por error segura para ExpressRoute. 
* Si lo desea, también puede usar redes VPN de sitio a sitio para conectarse a los sitios que no están conectados mediante ExpressRoute. 

En este artículo, se explican los pasos para configurar ambos escenarios. Este artículo se aplica al modelo de implementación de Resource Manager y utiliza PowerShell. También puede configurar estos escenarios con Azure Portal, aunque la documentación aún no está disponible. Puede configurar cualquier puerta de enlace en primer lugar. Normalmente, no se incurrirá en ningún tiempo de inactividad cuando se agrega una nueva puerta de enlace o una conexión de puerta de enlace.

>[!NOTE]
>Si desea crear una VPN de sitio a sitio a través de un circuito de ExpressRoute, consulte [este artículo](site-to-site-vpn-over-microsoft-peering.md).
>

## <a name="limits-and-limitations"></a>Límites y limitaciones
* **No se admite el enrutamiento transitorio.** No se puede realizar un enrutamiento (a través de Azure) entre una red local conectada a través de una VPN sitio a sitio y una red local conectada a través de ExpressRoute.
* **No se admite la puerta de enlace de la SKU de nivel Básico.** Debe utilizar una puerta de enlace de la SKU que no sea de nivel Básico tanto para la [puerta de enlace de ExpressRoute](expressroute-about-virtual-network-gateways.md) como para la [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* **Solo se admite la VPN Gateway basada en rutas.** Debe usar una [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md) basada en rutas. También puede usar una VPN Gateway basada en rutas con una conexión VPN configurada para "selectores de tráfico basados en directivas", tal y como se describe en [Conexión a varios dispositivos VPN basados en directivas](../vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps.md).
* **Se debe configurar una ruta estática para VPN Gateway.** Si la red local está conectada tanto a ExpressRoute como a una VPN de sitio a sitio, debe tener configurada una ruta estática en la red local para enrutar la conexión VPN de sitio a sitio a la red pública de Internet.
* **VPN Gateway se configura de manera predeterminada en ASN 65515 si no se especifica.** Azure VPN Gateway admite el protocolo de enrutamiento de BGP. Para especificar el ASN (número AS) de la red virtual, agregue el modificador - Asn. Si no se especifica este parámetro, el número AS predeterminado es 65515. Puede usar cualquier ASN para la configuración, pero si selecciona un valor distinto de 65515, debe restablecer la puerta de enlace para que la configuración surta efecto.
* **La subred de la puerta de enlace debe ser /27 o un prefijo más corto**, (como /26, /25). De lo contrario, recibirá un mensaje de error al agregar la puerta de enlace de red virtual de ExpressRoute.

## <a name="configuration-designs"></a>Diseños de configuración
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>Configuración de una VPN de sitio a sitio como una ruta de acceso de conmutación por error para ExpressRoute
Puede configurar una conexión VPN de sitio a sitio como una copia de seguridad para ExpressRoute. Esta conexión se aplica únicamente a las redes virtuales vinculadas a la ruta de acceso de emparejamiento privado de Azure. No hay ninguna solución de conmutación por error basada en VPN para servicios que sea accesible mediante el emparejamiento de Microsoft Azure. El circuito ExpressRoute siempre es el vínculo principal. Los datos fluyen a través de la ruta de acceso de la VPN de sitio a sitio solo si se produce un error en el circuito ExpressRoute. Para evitar el enrutamiento asimétrico, la configuración de red local también debería preferir el circuito ExpressRoute a través de la VPN de sitio a sitio. Si prefiere utilizar la ruta de ExpressRoute, dé mayor preferencia local a las rutas que reciben ExpressRoute. 

> [!NOTE]
> Si bien se prefiere el circuito ExpressRoute a la VPN de sitio a sitio cuando ambas rutas son las mismas, Azure utilizará la coincidencia de prefijo más larga para elegir la ruta hacia el destino del paquete.
> 
> 

![Coexistencia](media/expressroute-howto-coexist-resource-manager/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>Configuración de una VPN de sitio a sitio para conectarse a sitios no conectados mediante ExpressRoute
Puede configurar la red para sitios que se conectan directamente a Azure mediante una VPN de sitio a sitio y otros que se conectan a través de ExpressRoute. 

![Coexistencia](media/expressroute-howto-coexist-resource-manager/scenario2.jpg)

> [!NOTE]
> No se puede configurar una red virtual como un enrutador de tránsito.
> 
> 

## <a name="selecting-the-steps-to-use"></a>Selección de los pasos a seguir
Hay dos conjuntos diferentes de procedimientos que puede elegir. El procedimiento de configuración que seleccione depende de si ya tiene una red virtual existente a la que quiere conectarse o si desea crear una nueva.

* No tengo una red virtual y necesito crear una.
  
    Si aún no tiene una red virtual, este procedimiento le guía en la creación de una nueva red virtual mediante el modelo de implementación de Resource Manager y la creación de nuevas conexiones VPN de sitio a sitio y ExpressRoute. Para configurar una red virtual, siga los pasos que se describen en [Creación de una nueva red virtual y conexiones coexistentes](#new).
* Ya tengo una red virtual del modelo de implementación de Resource Manager.
  
    Puede que ya tenga una red virtual con una conexión VPN de sitio a sitio o una conexión ExpressRoute existentes. En este escenario, si la máscara de subred de la puerta de enlace es /28, o menor (/28, /29, etc.), tendrá que eliminar la puerta de enlace existente. La sección [Configuración de conexiones coexistentes para una red virtual existente](#add) le guía en la eliminación de la puerta de enlace y la creación de nuevas conexiones VPN de sitio a sitio y ExpressRoute.
  
    Si elimina y crea de nuevo la puerta de enlace, se producirá un tiempo de inactividad en las conexiones entre locales. Sin embargo, si las máquinas virtuales y los servicios están configurados para ello, podrán seguir comunicándose con el exterior a través del equilibrador de carga mientras configura la puerta de enlace.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [working with cloud shell](../../includes/expressroute-cloudshell-powershell-about.md)]


## <a name="to-create-a-new-virtual-network-and-coexisting-connections"></a><a name="new"></a>Creación de una nueva red virtual y conexiones coexistentes
Este procedimiento le guía en la creación de una red virtual y conexiones de sitio a sitio y ExpressRoute que coexisten. Los cmdlets que se usan en esta configuración pueden ser ligeramente diferentes de aquellos con los que podría estar familiarizado. Asegúrese de usar los cmdlets especificados en estas instrucciones.

1. Inicie sesión y seleccione su suscripción.

   [!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]
2. Configure las variables.

   ```azurepowershell-interactive
   $location = "Central US"
   $resgrp = New-AzResourceGroup -Name "ErVpnCoex" -Location $location
   $VNetASN = 65515
   ```
3. Cree una red virtual, incluida la subred de la puerta de enlace. Para más información acerca de cómo crear una red virtual en Azure, consulte [Create a virtual network](../virtual-network/manage-virtual-network.md#create-a-virtual-network) (Creación de una red virtual). Para más información acerca de cómo crear subredes, consulte [Incorporación de una subred](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet)
   
   > [!IMPORTANT]
   > La subred de puerta de enlace debe ser /27 o un prefijo más corto (por ejemplo, /26 o /25).
   > 
   > 
   
    Cree una nueva red virtual.

   ```azurepowershell-interactive
   $vnet = New-AzVirtualNetwork -Name "CoexVnet" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AddressPrefix "10.200.0.0/16"
   ```
   
    Agregue subredes.

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name "App" -VirtualNetwork $vnet -AddressPrefix "10.200.1.0/24"
   Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.0/24"
   ```
   
    Guarde la configuración de red virtual.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
4. <a name="vpngw"></a>A continuación, cree la puerta de enlace de la VPN de sitio a sitio. Para más información sobre la configuración de VPN Gateway, consulte [Creación de una red virtual con una conexión de sitio a sitio mediante PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md). GatewaySku solamente es compatible con las puertas de enlace de VPN *VpnGw1*, *VpnGw2*, *VpnGw3*, *Standard* y *HighPerformance*. Las configuraciones de la coexistencia de ExpressRoute-VPN Gateway no se admiten en la SKU de nivel Básico. El valor de VpnType debe ser *RouteBased*.

   ```azurepowershell-interactive
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   $gwIP = New-AzPublicIpAddress -Name "VPNGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
   $gwConfig = New-AzVirtualNetworkGatewayIpConfig -Name "VPNGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
   New-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "Vpn" -VpnType "RouteBased" -GatewaySku "VpnGw1"
   ```
   
    Azure VPN Gateway admite el protocolo de enrutamiento de BGP. Para especificar el ASN (número AS) de la red virtual, agregue el modificador - Asn al siguiente comando. Si no se especifica ese parámetro, el valor predeterminado del número AS será 65515.

   ```azurepowershell-interactive
   $azureVpn = New-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "Vpn" -VpnType "RouteBased" -GatewaySku "VpnGw1" -Asn $VNetASN
   ```
   
    Puede buscar el IP de emparejamiento BGP y el número de AS que Azure usa para VPN Gateway en $azureVpn.BgpSettings.BgpPeeringAddress y $azureVpn.BgpSettings.Asn de BGP. Para más información, consulte [Configuración de BGP](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md) para Azure VPN Gateway.
5. Cree una entidad de puerta de enlace de VPN de sitio local. Este comando no configura la puerta de enlace de VPN local. En su lugar, permite proporcionar la configuración de puerta de enlace local, como la dirección IP pública y el espacio de direcciones local, para que la puerta de enlace de VPN de Azure pueda conectarse a ella.
   
    Si el dispositivo VPN local solo admite enrutamiento estático, las rutas estáticas se pueden configurar como se indica a continuación:

   ```azurepowershell-interactive
   $MyLocalNetworkAddress = @("10.100.0.0/16","10.101.0.0/16","10.102.0.0/16")
   $localVpn = New-AzLocalNetworkGateway -Name "LocalVPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -GatewayIpAddress *<Public IP>* -AddressPrefix $MyLocalNetworkAddress
   ```
   
    Si el dispositivo VPN local admite el BGP y desea habilitar el enrutamiento dinámico, es preciso que conozca la IP de emparejamiento BGP y el número de AS que usa el dispositivo VPN local.

   ```azurepowershell-interactive
   $localVPNPublicIP = "<Public IP>"
   $localBGPPeeringIP = "<Private IP for the BGP session>"
   $localBGPASN = "<ASN>"
   $localAddressPrefix = $localBGPPeeringIP + "/32"
   $localVpn = New-AzLocalNetworkGateway -Name "LocalVPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -GatewayIpAddress $localVPNPublicIP -AddressPrefix $localAddressPrefix -BgpPeeringAddress $localBGPPeeringIP -Asn $localBGPASN
   ```
6. Configure el dispositivo VPN local para que se conecte a la nueva puerta de enlace de VPN de Azure. Para obtener más información sobre la configuración del dispositivo VPN, vea [Configuración de dispositivos VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md).

7. Vincule la puerta de enlace de VPN sitio a sitio en Azure a la puerta de enlace local.

   ```azurepowershell-interactive
   $azureVpn = Get-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName
   New-AzVirtualNetworkGatewayConnection -Name "VPNConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $azureVpn -LocalNetworkGateway2 $localVpn -ConnectionType IPsec -SharedKey <yourkey>
   ```
 

8. Si está conectado a un circuito ExpressRoute existente, omita los pasos 8 y 9 y pase directamente al paso 10. Configure los circuitos ExpressRoute. Para más información acerca de cómo configurar los circuitos ExpressRoute, consulte [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md).


9. Configure el emparejamiento privado de Azure a través del circuito ExpressRoute. Para más información acerca de cómo configurar el emparejamiento privado de Azure a través del circuito ExpressRoute, consulte [Configuración del emparejamiento](expressroute-howto-routing-arm.md).

10. <a name="gw"></a>Cree una puerta de enlace de ExpressRoute. Para más información sobre la configuración de la puerta de enlace de ExpressRoute, consulte [Adición de una puerta de enlace de VPN a una red virtual de Resource Manager para una configuración de ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). El valor de GatewaySKU debe ser *Standard*, *HighPerformance* o *UltraPerformance*.

    ```azurepowershell-interactive
    $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
    $gwIP = New-AzPublicIpAddress -Name "ERGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
    $gwConfig = New-AzVirtualNetworkGatewayIpConfig -Name "ERGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
    $gw = New-AzVirtualNetworkGateway -Name "ERGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "ExpressRoute" -GatewaySku Standard
    ```
11. Vincule la puerta de enlace de ExpressRoute al circuito ExpressRoute. Una vez completado este paso, se ha establecido la conexión entre su red local y Azure a través de ExpressRoute. Para más información sobre la operación de vinculación, consulte [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md).

    ```azurepowershell-interactive
    $ckt = Get-AzExpressRouteCircuit -Name "YourCircuit" -ResourceGroupName "YourCircuitResourceGroup"
    New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $gw -PeerId $ckt.Id -ConnectionType ExpressRoute
    ```

## <a name="to-configure-coexisting-connections-for-an-already-existing-vnet"></a><a name="add"></a>Para configurar conexiones coexistentes para una red virtual ya existente
Si tiene una red virtual con una sola una puerta de enlace de red virtual (por ejemplo, una puerta de enlace de VPN de sitio a sitio) y desea agregar una segunda puerta de enlace de otro tipo (por ejemplo, una puerta de enlace de ExpressRoute), compruebe el tamaño de la subred de la puerta de enlace. Si la subred de la puerta de enlace es/27, o mayor, puede omitir los pasos siguientes y seguir los de la sección anterior para agregar una puerta de enlace VPN de sitio a sitio o una puerta de enlace de ExpressRoute. Si la subred de la puerta de enlace es /28 o /29, primero debe eliminar la puerta de enlace de red virtual y aumentar el tamaño de la subred de la puerta de enlace. Los pasos de esta sección le muestran cómo hacerlo.

Los cmdlets que se usan en esta configuración pueden ser ligeramente diferentes de aquellos con los que podría estar familiarizado. Asegúrese de usar los cmdlets especificados en estas instrucciones.

1. Elimine la puerta de enlace de la VPN de ExpressRoute o de sitio a sitio.

   ```azurepowershell-interactive 
   Remove-AzVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
   ```
2. Elimine la subred de puerta de enlace.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name <yourvnetname> -ResourceGroupName <yourresourcegroup> Remove-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
   ```
3. Agregue una subred de puerta de enlace que sea /27 o superior.
   
   > [!NOTE]
   > Si no tiene suficientes direcciones IP en la red virtual para aumentar el tamaño de la subred de puerta de enlace, debe agregar más espacio de direcciones IP.
   > 
   > 

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
   Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.0/24"
   ```
   
    Guarde la configuración de red virtual.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
4. Ya tiene una red virtual sin puertas de enlace. Para crear nuevas puertas de enlace y configurar las conexiones, utilice los siguientes ejemplos:

   Establezca las variables.

    ```azurepowershell-interactive
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   $gwIP = New-AzPublicIpAddress -Name "ERGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
   $gwConfig = New-AzVirtualNetworkGatewayIpConfig -Name "ERGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
   ```

   Cree la puerta de enlace.

   ```azurepowershell-interactive
   $gw = New-AzVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup> -Location <yourlocation> -IpConfigurations $gwConfig -GatewayType "ExpressRoute" -GatewaySku Standard
   ```

   Cree la conexión.

   ```azurepowershell-interactive
   $ckt = Get-AzExpressRouteCircuit -Name "YourCircuit" -ResourceGroupName "YourCircuitResourceGroup"
   New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $gw -PeerId $ckt.Id -ConnectionType ExpressRoute
   ```

## <a name="to-add-point-to-site-configuration-to-the-vpn-gateway"></a>Incorporación de la configuración de punto a sitio a la puerta de enlace de VPN

Para agregar una configuración de punto a sitio a la puerta de enlace de VPN en una configuración de coexistencia, puede seguir los pasos que se indican a continuación. Para cargar el certificado raíz VPN, debe instalar PowerShell localmente en el equipo, o utilizar Azure Portal.

1. Agregue el grupo de direcciones de clientes de VPN.

   ```azurepowershell-interactive
   $azureVpn = Get-AzVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName
   Set-AzVirtualNetworkGatewayVpnClientConfig -VirtualNetworkGateway $azureVpn -VpnClientAddressPool "10.251.251.0/24"
   ```
2. En Azure, cargue el certificado raíz de VPN de la puerta de enlace de VPN. En este ejemplo, se supone que el certificado raíz se almacena en la máquina local donde se ejecutan los siguientes cmdlets de PowerShell y que el usuario está ejecutando PowerShell localmente. También puede cargar el certificado con Azure Portal.

   ```powershell
   $p2sCertFullName = "RootErVpnCoexP2S.cer" 
   $p2sCertMatchName = "RootErVpnCoexP2S" 
   $p2sCertToUpload=get-childitem Cert:\CurrentUser\My | Where-Object {$_.Subject -match $p2sCertMatchName} 
   if ($p2sCertToUpload.count -eq 1){write-host "cert found"} else {write-host "cert not found" exit} 
   $p2sCertData = [System.Convert]::ToBase64String($p2sCertToUpload.RawData) 
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $p2sCertFullName -VirtualNetworkGatewayname $azureVpn.Name -ResourceGroupName $resgrp.ResourceGroupName -PublicCertData $p2sCertData
   ```

Para más información sobre la VPN de punto a sitio, consulte [Configuración de una conexión punto a sitio a una red virtual mediante PowerShell](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md).

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).
