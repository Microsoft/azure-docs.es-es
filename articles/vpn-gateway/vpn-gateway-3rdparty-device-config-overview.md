---
title: Configuraciones de dispositivo VPN asociado para conexión a puertas de enlace de VPN de Azure
description: Este artículo proporciona información general sobre configuraciones de dispositivo VPN asociado para conectarse a puertas de enlace de VPN de Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 01ceb067f589ef04b04ff0bf05e2971ee30f407c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400167"
---
# <a name="overview-of-partner-vpn-device-configurations"></a>Información general sobre las configuraciones de dispositivo VPN asociado
Este artículo proporciona información general sobre configuraciones de dispositivos VPN locales para conectarse a puertas de enlace de VPN de Azure. Se usará una red virtual de Azure de ejemplo y la configuración de la puerta de enlace de VPN para conectarse a distintos dispositivos VPN locales usando los mismos parámetros.



## <a name="device-requirements"></a>Requisitos del dispositivo
Las puertas de enlace de VPN de Azure usan los conjuntos de protocolos IPsec o IKE estándar para túneles VPN de sitio a sitio (S2S). Consulte [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md) para obtener una lista de los parámetros IPsec o IKE y los algoritmos criptográficos para las puertas de enlace de VPN de Azure. También puede especificar los algoritmos exactos y los niveles de clave para una conexión específica, como se describe en [Acerca de los requisitos criptográficos](vpn-gateway-about-compliance-crypto.md).

## <a name="single-vpn-tunnel"></a><a name ="singletunnel"></a>Túnel VPN único
La primera configuración del ejemplo consta de un solo túnel VPN S2S entre una puerta de enlace de VPN de Azure y el dispositivo VPN local. Opcionalmente puede configurar [Border Gateway Protocol (BGP) a través del túnel VPN](#bgp).

![Diagrama de túnel VPN S2S único](./media/vpn-gateway-3rdparty-device-config-overview/singletunnel.png)

Para obtener instrucciones paso a paso para configurar un túnel VPN único, consulte la [configuración de una conexión de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Las siguientes secciones especifican los parámetros de conexión para la configuración de ejemplo y proporcionarán un script de PowerShell para ayudarle a empezar a trabajar.

### <a name="connection-parameters"></a>Parámetros de conexión
Esta sección enumera los parámetros para los ejemplos que se describen en las secciones anteriores.

| **Parámetro**                | **Valor**                    |
| ---                          | ---                          |
| Prefijos de direcciones de red virtual        | 10.11.0.0/16<br>10.12.0.0/16 |
| Dirección IP de la puerta de enlace de VPN de Azure         | Dirección IP de Azure VPN Gateway         |
| Prefijos de direcciones locales | 10.51.0.0/16<br>10.52.0.0/16 |
| Dirección IP del dispositivo VPN local    | Dirección IP del dispositivo VPN local    |
| *Red Virtual BGP ASN                | 65010                        |
| *Dirección IP del par BGP de Azure           | 10.12.255.30                 |
| *ASN de BGP local         | 65050                        |
| *Dirección IP del par BGP local     | 10.52.255.254                |

\* Parámetro opcional solo para BGP.

### <a name="sample-powershell-script"></a>Script de PowerShell de ejemplo
En esta sección se proporciona un script de ejemplo para ayudarle a comenzar. Para instrucciones detalladas consulte [Creación de una red virtual con una conexión VPN S2S de sitio a sitio mediante PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md).

```powershell
# Declare your variables

$Sub1          = "Replace_With_Your_Subscription_Name"
$RG1           = "TestRG1"
$Location1     = "East US 2"
$VNetName1     = "TestVNet1"
$FESubName1    = "FrontEnd"
$BESubName1    = "Backend"
$GWSubName1    = "GatewaySubnet"
$VNetPrefix11  = "10.11.0.0/16"
$VNetPrefix12  = "10.12.0.0/16"
$FESubPrefix1  = "10.11.0.0/24"
$BESubPrefix1  = "10.12.0.0/24"
$GWSubPrefix1  = "10.12.255.0/27"
$VNet1ASN      = 65010
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GWIPName1     = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection15  = "VNet1toSite5"
$LNGName5      = "Site5"
$LNGPrefix50   = "10.52.255.254/32"
$LNGPrefix51   = "10.51.0.0/16"
$LNGPrefix52   = "10.52.0.0/16"
$LNGIP5        = "Your_VPN_Device_IP"
$LNGASN5       = 65050
$BGPPeerIP5    = "10.52.255.254"

# Connect to your subscription and create a new resource group

Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1

# Create virtual network

$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1 $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

# Create VPN gateway

$gwpip1    = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1     = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1   = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1

New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN

# Create local network gateway

New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix51,$LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5

# Create the S2S VPN connection

$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False
```

### <a name="optional-use-custom-ipsecike-policy-with-usepolicybasedtrafficselectors"></a><a name ="policybased"></a>(Opcional) Usar directivas personalizadas de IPsec/IKE con UsePolicyBasedTrafficSelectors
Si los dispositivos VPN no admiten selectores de tráfico universales, tales como configuraciones basadas en enrutamiento o en VTI, cree una directiva personalizada de IPsec/IKE con la opción [UsePolicyBasedTrafficSelectors](vpn-gateway-connect-multiple-policybased-rm-ps.md).

> [!IMPORTANT]
> Tiene que crear una directiva IPsec/IKE para habilitar la opción **UsePolicyBasedTrafficSelectors** en la conexión.


El script de ejemplo siguiente crea una directiva de IPsec o IKE con los algoritmos y parámetros siguientes:
* IKEv2: AES256, SHA384, DHGroup24
* IPsec: AES256, SHA1, PFS24, vigencia de SA 7,200 segundos y 20.480.000 KB (20 GB)

El script aplica la directiva IPsec/IKE y habilita la opción **UsePolicyBasedTrafficSelectors** en la conexión.

```powershell
$ipsecpolicy5 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA1 -PfsGroup PFS24 -SALifeTimeSeconds 7200 -SADataSizeKilobytes 20480000

$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False -IpsecPolicies $ipsecpolicy5 -UsePolicyBasedTrafficSelectors $True
```

### <a name="optional-use-bgp-on-s2s-vpn-connection"></a><a name ="bgp"></a>(Opcional) Usar BGP en una conexión VPN S2S
Al crear la conexión VPN S2S, también tiene la opción de usar [BGP para la puerta de enlace VPN](vpn-gateway-bgp-resource-manager-ps.md). Este enfoque tiene dos diferencias:

* Los prefijos de dirección local pueden ser una dirección de host único. La dirección IP del par BGP local se especifica como sigue:

    ```powershell
    New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
    ```

* Al crear la conexión, tiene que establecer la opción **- EnableBGP** en $True:

    ```powershell
    New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
    ```

## <a name="next-steps"></a>Pasos siguientes
Consulte [Configuración activo-activo de puertas de enlace de VPN para conexiones entre locales y de red virtual a red virtual](vpn-gateway-activeactive-rm-powershell.md) para ver las instrucciones paso a paso para configurar puertas de enlace VPN activo-activo.

