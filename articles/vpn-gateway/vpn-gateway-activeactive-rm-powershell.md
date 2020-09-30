---
title: Configuración de conexiones S2S activo-activo de Azure VPN Gateway
description: Este artículo le guiará a la hora de configurar conexiones activo-activo con Azure VPN Gateway usando Azure Resource Manager y PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.reviewer: cherylmc
ms.openlocfilehash: 022ccaab0b210cd2d656b69f505791d1a2aa963f
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440786"
---
# <a name="configure-active-active-s2s-vpn-connections-with-azure-vpn-gateways"></a>Configuración de conexiones VPN S2S activo-activo con Azure VPN Gateway

Este artículo le guiará por los pasos para crear conexiones activo-activo entre entornos locales y de red virtual a red virtual mediante el modelo de implementación de Resource Manager y PowerShell. También puede configurar una puerta de enlace activo-activo en Azure Portal.

## <a name="about-highly-available-cross-premises-connections"></a>Acerca de las conexiones entre entornos locales de alta disponibilidad
Para lograr una alta disponibilidad en la conectividad de red virtual a red virtual y entre entornos locales, debe implementar varias puertas de enlace VPN y establecer varias conexiones en paralelo entre sus redes y Azure. Consulte [Conectividad de alta disponibilidad entre locales y de red virtual a red virtual](vpn-gateway-highlyavailable.md) para obtener información general de las opciones de conectividad y la topología.

En este artículo se proporcionan instrucciones para configurar una conexión VPN activo-activo entre entornos locales, así como una conexión activo-activo entre dos redes virtuales.

* [Parte 1: Creación y configuración de Azure VPN Gateway en el modo activo-activo](#aagateway)
* [Parte 2: Establecimiento de conexiones activo-activo entre entornos locales](#aacrossprem)
* [Parte 3: Establecimiento de conexiones activo-activo de red virtual a red virtual](#aav2v)

Si ya tiene una instancia de VPN Gateway, puede hacer lo siguiente:
* [Actualizar una instancia de VPN Gateway existente de activo-en espera a activo-activo o viceversa](#aaupdate)

Puede combinar todos estos elementos para crear una red más compleja de alta disponibilidad que satisfaga sus necesidades.

> [!IMPORTANT]
> El modo activo-activo está disponible para todas las SKU, excepto Basic.

## <a name="part-1---create-and-configure-active-active-vpn-gateways"></a><a name ="aagateway"></a>Parte 1: Creación y configuración de puertas de enlace VPN activo-activo
Los pasos a continuación configurarán Azure VPN Gateway en modos activo-activo. Las diferencias clave entre las puertas de enlace activo-activo y activo-en espera:

* Tiene que crear dos configuraciones de IP de puerta de enlace con dos direcciones IP públicas
* Tiene que establecer la marca EnableActiveActiveFeature
* La SKU de puerta de enlace debe ser VpnGw1, VpnGw2, VpnGw3 o HighPerformance (SKU heredada).

Las demás propiedades son las mismas que las de las puertas de enlace que no son activo-activo. 

### <a name="before-you-begin"></a>Antes de empezar
* Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Tendrá que instalar los cmdlets de PowerShell de Azure Resource Manager si no quiere usar CloudShell en el explorador. Vea [Información general sobre Azure PowerShell](/powershell/azure/) para obtener más información sobre la instalación de los cmdlets de PowerShell.

### <a name="step-1---create-and-configure-vnet1"></a>Paso 1: Creación y configuración de VNet1
#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Para este ejercicio, comenzaremos declarando las variables. Si utiliza la opción "Probar" de Cloud Shell, se conectará automáticamente a su cuenta. Si usa PowerShell localmente, utilice el siguiente ejemplo para ayudarle a conectarse:

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
```

En el ejemplo siguiente, se declaran las variables con los valores para este ejercicio. Asegúrese de reemplazar los valores por los suyos propios cuando realice la configuración para el entorno de producción. Puede usar estas variables si está practicando los pasos para familiarizarse con este tipo de configuración. Modifique las variables y después copie y pegue todo en la consola de PowerShell.

```azurepowershell-interactive
$Sub1 = "Ross"
$RG1 = "TestAARG1"
$Location1 = "West US"
$VNetName1 = "TestVNet1"
$FESubName1 = "FrontEnd"
$BESubName1 = "Backend"
$GWSubName1 = "GatewaySubnet"
$VNetPrefix11 = "10.11.0.0/16"
$VNetPrefix12 = "10.12.0.0/16"
$FESubPrefix1 = "10.11.0.0/24"
$BESubPrefix1 = "10.12.0.0/24"
$GWSubPrefix1 = "10.12.255.0/27"
$VNet1ASN = 65010
$DNS1 = "8.8.8.8"
$GWName1 = "VNet1GW"
$GW1IPName1 = "VNet1GWIP1"
$GW1IPName2 = "VNet1GWIP2"
$GW1IPconf1 = "gw1ipconf1"
$GW1IPconf2 = "gw1ipconf2"
$Connection12 = "VNet1toVNet2"
$Connection151 = "VNet1toSite5_1"
$Connection152 = "VNet1toSite5_2"
```

#### <a name="2-create-a-new-resource-group"></a>2. Creación de un nuevo grupo de recursos

Use el ejemplo siguiente para crear un grupo de recursos:

```azurepowershell-interactive
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3. Creación de TestVNet1
En el ejemplo siguiente se crea una red virtual denominada TestVNet1 y tres subredes llamadas: GatewaySubnet, FrontEnd y Backend. Al reemplazar valores, es importante que siempre asigne el nombre GatewaySubnet a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace.

```azurepowershell-interactive
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-active-active-mode"></a>Paso 2: Creación de la puerta de enlace de VPN para TestVNet1 con modo activo-activo
#### <a name="1-create-the-public-ip-addresses-and-gateway-ip-configurations"></a>1. Creación de las configuraciones de direcciones de IP públicas y la IP de puerta de enlace
Solicite que se asignen dos direcciones IP públicas a la puerta de enlace que creará para la red virtual. También definirá las configuraciones de subred y de IP necesarias.

```azurepowershell-interactive
$gw1pip1 = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$gw1pip2 = New-AzPublicIpAddress -Name $GW1IPName2 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1
$gw1ipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf2 -Subnet $subnet1 -PublicIpAddress $gw1pip2
```

#### <a name="2-create-the-vpn-gateway-with-active-active-configuration"></a>2. Creación de la puerta de enlace VPN con una configuración activo-activo
Cree la puerta de enlace de red virtual para TestVNet1. Tenga en cuenta que hay dos entradas de GatewayIpConfig y la marca EnableActiveActiveFeature está establecida. Se tardan unos 45 minutos o algo más en crear una puerta de enlace.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1,$gw1ipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN -EnableActiveActiveFeature -Debug
```

#### <a name="3-obtain-the-gateway-public-ip-addresses-and-the-bgp-peer-ip-address"></a>3. Obtención de las direcciones IP públicas de puerta de enlace y la dirección IP del par BGP
Una vez creada la puerta de enlace, debe obtener la dirección IP del par BGP en Azure VPN Gateway. Esta dirección es necesaria para configurar la instancia de Azure VPN Gateway como par BGP para los dispositivos de VPN local.

```azurepowershell-interactive
$gw1pip1 = Get-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1
$gw1pip2 = Get-AzPublicIpAddress -Name $GW1IPName2 -ResourceGroupName $RG1
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
```

Use los siguientes cmdlets para mostrar las dos direcciones IP públicas asignadas a la puerta de enlace VPN y sus correspondientes direcciones IP del par BGP para cada instancia de puerta de enlace:

```azurepowershell-interactive
PS D:\> $gw1pip1.IpAddress
40.112.190.5

PS D:\> $gw1pip2.IpAddress
138.91.156.129

PS D:\> $vnet1gw.BgpSettingsText
{
  "Asn": 65010,
  "BgpPeeringAddress": "10.12.255.4,10.12.255.5",
  "PeerWeight": 0
}
```

El orden de las direcciones IP públicas para las instancias de puerta de enlace y las direcciones de emparejamiento BGP correspondiente es el mismo. En este ejemplo, la máquina virtual de la puerta de enlace con la IP pública 40.112.190.5 utilizará 10.12.255.4 como su dirección de emparejamiento BGP y la puerta de enlace con 138.91.156.129 usará 10.12.255.5. Esta información es necesaria cuando configura los dispositivos VPN locales en conexión con la puerta de enlace activo-activo. La puerta de enlace se muestra en el diagrama siguiente con todas las direcciones:

![puerta de enlace activo-activo](./media/vpn-gateway-activeactive-rm-powershell/active-active-gw.png)

Una vez creada la puerta de enlace, puede usarla para establecer conexión activo-activo entre entornos locales o de red virtual a red virtual. Las siguientes secciones lo guiarán por los pasos necesarios para completar el ejercicio.

## <a name="part-2---establish-an-active-active-cross-premises-connection"></a><a name ="aacrossprem"></a>Parte 2: Establecimiento de una conexión activo-activo entre entornos locales
Para establecer una conexión entre locales, debe crear una puerta de enlace de red local para representar el dispositivo VPN local y una conexión para conectarse a la puerta de enlace de VPN de Azure con la puerta de enlace de red local. En este ejemplo, la instancia de Azure VPN Gateway está en modo activo-activo. Como resultado, aunque hay solo un dispositivo VPN local (puerta de enlace de red local) y un recurso de conexión, ambas instancias de Azure VPN Gateway establecerán túneles VPN de S2S con el dispositivo local.

Antes de continuar, asegúrese de que ha completado la [parte 1](#aagateway) de este ejercicio.

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>Paso 1: Cree y configure la puerta de enlace de red local
#### <a name="1-declare-your-variables"></a>1. Declaración de las variables
Este ejercicio continuará generando la configuración mostrada en el diagrama. Asegúrese de reemplazar los valores por los que desea usar para su configuración.

```azurepowershell-interactive
$RG5 = "TestAARG5"
$Location5 = "West US"
$LNGName51 = "Site5_1"
$LNGPrefix51 = "10.52.255.253/32"
$LNGIP51 = "131.107.72.22"
$LNGASN5 = 65050
$BGPPeerIP51 = "10.52.255.253"
```

Un par de cosas a tener en cuenta con respecto a los parámetros de la puerta de enlace de red local:

* La puerta de enlace de red local puede estar en la misma ubicación y grupo de recursos que la puerta de enlace de VPN o en otros distintos. Este ejemplo los muestra en distintos grupos de recursos pero en la misma ubicación de Azure.
* Si hay un único dispositivo VPN local como se muestra anteriormente, la conexión activo-activo puede trabajar con o sin el protocolo BGP. Este ejemplo utiliza BGP para la conexión entre redes locales.
* Si BGP está habilitado el prefijo que debe declarar para la puerta de enlace de red local es la dirección del host de la dirección IP del par BGP en el dispositivo VPN. En este caso, es un prefijo /32 de "10.52.255.253/32".
* Como recordatorio, debe usar diferentes ASN de BGP entre las redes locales y la red virtual de Azure. Si son iguales, tiene que cambiar el ASN de la red virtual si el dispositivo VPN local ya utiliza el ASN para emparejarse con otros vecinos de BGP.

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2. Cree las puertas de enlace de red local para Site5
Antes de continuar, asegúrese de que sigue conectado a la suscripción 1. Cree el grupo de recursos si aún no se ha creado.

```azurepowershell-interactive
New-AzResourceGroup -Name $RG5 -Location $Location5
New-AzLocalNetworkGateway -Name $LNGName51 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP51 -AddressPrefix $LNGPrefix51 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP51
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>Paso 2: Conecte la puerta de enlace de red virtual y la puerta de enlace de red local
#### <a name="1-get-the-two-gateways"></a>1. Obtenga las dos puertas de enlace

```azurepowershell-interactive
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw1 = Get-AzLocalNetworkGateway  -Name $LNGName51 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2. Cree la conexión de TestVNet1 a Site5
En este paso, crea la conexión de TestVNet1 a Site5_1 con "EnableBGP" establecido como $True.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name $Connection151 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw1 -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

#### <a name="3-vpn-and-bgp-parameters-for-your-on-premises-vpn-device"></a>3. Parámetros VPN y BGP para el dispositivo VPN local
En el ejemplo siguiente se enumeran los parámetros que deberá especificar en la sección de configuración de BGP en el dispositivo VPN local para este ejercicio:

```
- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.253
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP 1  : 10.12.255.4 for tunnel to 40.112.190.5
- Azure VNet BGP IP 2  : 10.12.255.5 for tunnel to 138.91.156.129
- Static routes        : Destination 10.12.255.4/32, nexthop the VPN tunnel interface to 40.112.190.5
                         Destination 10.12.255.5/32, nexthop the VPN tunnel interface to 138.91.156.129
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

La conexión debe establecerse después de unos minutos y se iniciará la sesión de emparejamiento BGP una vez establecida la conexión IPsec. En este ejemplo hasta ahora se ha configurado un solo dispositivo VPN local, con el resultado que se muestra en el diagrama a continuación:

![activo-activo-entre entornos locales](./media/vpn-gateway-activeactive-rm-powershell/active-active.png)

### <a name="step-3---connect-two-on-premises-vpn-devices-to-the-active-active-vpn-gateway"></a>Paso 3: Conexión de dos dispositivos VPN locales con la puerta de enlace VPN activo-activo
Si tiene dos dispositivos VPN en la misma red local, puede lograr redundancia dual conectando Azure VPN Gateway al segundo dispositivo VPN.

#### <a name="1-create-the-second-local-network-gateway-for-site5"></a>1. Creación de la puertas de enlace de red local para Site5
La dirección IP de puerta de enlace, el prefijo de dirección y la dirección de emparejamiento de BGP para la segunda puerta de enlace de red local no pueden superponerse con la puerta de enlace de red local anterior en la misma red local.

```azurepowershell-interactive
$LNGName52 = "Site5_2"
$LNGPrefix52 = "10.52.255.254/32"
$LNGIP52 = "131.107.72.23"
$BGPPeerIP52 = "10.52.255.254"
```

```azurepowershell-interactive
New-AzLocalNetworkGateway -Name $LNGName52 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP52 -AddressPrefix $LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP52
```

#### <a name="2-connect-the-vnet-gateway-and-the-second-local-network-gateway"></a>2. Paso 2: Conexión de la puerta de enlace de red virtual y la segunda puerta de enlace de red local
Cree la conexión de TestVNet1 a Site5_2 con "EnableBGP" establecido como $True

```azurepowershell-interactive
$lng5gw2 = Get-AzLocalNetworkGateway -Name $LNGName52 -ResourceGroupName $RG5
```

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name $Connection152 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw2 -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

#### <a name="3-vpn-and-bgp-parameters-for-your-second-on-premises-vpn-device"></a>3. Los parámetros VPN y BGP para el segundo dispositivo VPN local
De forma similar, a continuación se indican los parámetros que tendrá que especificar en el segundo dispositivo VPN:

```
- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP 1  : 10.12.255.4 for tunnel to 40.112.190.5
- Azure VNet BGP IP 2  : 10.12.255.5 for tunnel to 138.91.156.129
- Static routes        : Destination 10.12.255.4/32, nexthop the VPN tunnel interface to 40.112.190.5
                         Destination 10.12.255.5/32, nexthop the VPN tunnel interface to 138.91.156.129
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

Una vez que se establece la conexión (túneles), tendrá dos dispositivos VPN redundantes y túneles que conectan la red local y Azure:

![redundancia-dual-entre entornos locales](./media/vpn-gateway-activeactive-rm-powershell/dual-redundancy.png)

## <a name="part-3---establish-an-active-active-vnet-to-vnet-connection"></a><a name ="aav2v"></a>Parte 3: Establecimiento de una conexión activo-activo de red virtual a red virtual
En esta sección se crea una conexión de red virtual a red virtual activo-activo con BGP. 

Las instrucciones que siguen son continuación de los pasos anteriores ya explicados. Tiene que completar la [Parte 1](#aagateway) para crear y configurar TestVNet1 y VPN Gateway con BGP. 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>Paso 1: cree TestVNet2 y la puerta de enlace de VPN
Es importante asegurarse de que el espacio de direcciones IP de la red virtual nueva, TestVNet2, no se solape con ninguno de sus intervalos de red virtual.

En este ejemplo, las redes virtuales pertenecen a la misma suscripción. Puede configurar las conexiones de red virtual a red virtual entre distintas suscripciones, para ello consulte [Configuración de una conexión de red virtual a red virtual](vpn-gateway-vnet-vnet-rm-ps.md) para más información. Asegúrese de agregar "-EnableBgp True" al crear las conexiones para habilitar BGP.

#### <a name="1-declare-your-variables"></a>1. Declaración de las variables
Asegúrese de reemplazar los valores por los que desea usar para su configuración.

```azurepowershell-interactive
$RG2 = "TestAARG2"
$Location2 = "East US"
$VNetName2 = "TestVNet2"
$FESubName2 = "FrontEnd"
$BESubName2 = "Backend"
$GWSubName2 = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$VNet2ASN = 65020
$DNS2 = "8.8.8.8"
$GWName2 = "VNet2GW"
$GW2IPName1 = "VNet2GWIP1"
$GW2IPconf1 = "gw2ipconf1"
$GW2IPName2 = "VNet2GWIP2"
$GW2IPconf2 = "gw2ipconf2"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Cree TestVNet2 en el nuevo grupo de recursos

```azurepowershell-interactive
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2
```

#### <a name="3-create-the-active-active-vpn-gateway-for-testvnet2"></a>3. Creación de la puerta de enlace VPN activo-activo para TestVNet2
Solicite que se asignen dos direcciones IP públicas a la puerta de enlace que creará para la red virtual. También definirá las configuraciones de subred y de IP necesarias.

```azurepowershell-interactive
$gw2pip1 = New-AzPublicIpAddress -Name $GW2IPName1 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic
$gw2pip2 = New-AzPublicIpAddress -Name $GW2IPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2 = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gw2ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW2IPconf1 -Subnet $subnet2 -PublicIpAddress $gw2pip1
$gw2ipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GW2IPconf2 -Subnet $subnet2 -PublicIpAddress $gw2pip2
```

Cree la puerta de enlace VPN con el número de AS y la marca "EnableActiveActiveFeature". Tenga en cuenta que debe reemplazar el valor predeterminado del ASN en las puertas de enlace de VPN de Azure. Los ASN para las redes virtuales conectadas deben ser diferentes para habilitar el enrutamiento de tránsito y de BGP.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gw2ipconf1,$gw2ipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet2ASN -EnableActiveActiveFeature
```

### <a name="step-2---connect-the-testvnet1-and-testvnet2-gateways"></a>Paso 2: Conecte las puertas de enlace de TestVNet1 y TestVNet2
En este ejemplo, ambas puertas de enlace están en la misma suscripción. Puede completar este paso en la misma sesión de PowerShell.

#### <a name="1-get-both-gateways"></a>1. Obtenga ambas puertas de enlace
Asegúrese de iniciar sesión y conectarse a la suscripción 1.

```azurepowershell-interactive
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
```

#### <a name="2-create-both-connections"></a>2. Cree ambas conexiones
En este paso, creará la conexión de TestVNet1 a TestVNet2 y viceversa.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> Asegúrese de habilitar BGP para AMBAS conexiones.
> 
> 

Después de completar estos pasos, la conexión se establecerá en unos minutos y la sesión de emparejamiento de BGP se iniciará una vez se complete la conexión de red virtual a red virtual con redundancia dual:

![activo-activo-v2v](./media/vpn-gateway-activeactive-rm-powershell/vnet-to-vnet.png)

## <a name="update-an-existing-vpn-gateway"></a><a name ="aaupdate"></a>Actualización de una instancia de VPN Gateway existente

Al cambiar una puerta de enlace de activo-en espera a activo-activo, crea otra dirección IP pública y después agrega una segunda configuración IP de puerta de enlace. En esta sección, le ayudamos a cambiar una instancia de Azure VPN Gateway existente del modo activo-en espera al modo activo-activo o viceversa mediante PowerShell. También puede cambiar una puerta de enlace en Azure Portal en la página **Configuración** de la puerta de enlace de red virtual.

### <a name="change-an-active-standby-gateway-to-an-active-active-gateway"></a>Cambio de una puerta de enlace de activo-en espera a una puerta de enlace activo-activo

En el ejemplo siguiente se convierte una puerta de enlace de activo-en espera en puerta de enlace activo-activo. 

#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Reemplace los siguientes parámetros utilizados para los ejemplos con los valores que necesita para su propia configuración y, a continuación, declare estas variables.

```azurepowershell-interactive
$GWName = "TestVNetAA1GW"
$VNetName = "TestVNetAA1"
$RG = "TestVPNActiveActive01"
$GWIPName2 = "gwpip2"
$GWIPconf2 = "gw1ipconf2"
```

Después de declarar las variables, puede copiar y pegar este ejemplo en la consola de PowerShell.

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$location = $gw.Location
```

#### <a name="2-create-the-public-ip-address-then-add-the-second-gateway-ip-configuration"></a>2. Cree la dirección IP pública, y agregue una segunda configuración IP de puerta de enlace

```azurepowershell-interactive
$gwpip2 = New-AzPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG -Location $location -AllocationMethod Dynamic
Add-AzVirtualNetworkGatewayIpConfig -VirtualNetworkGateway $gw -Name $GWIPconf2 -Subnet $subnet -PublicIpAddress $gwpip2
```

#### <a name="3-enable-active-active-mode-and-update-the-gateway"></a>3. Habilite el modo activo-activo y actualice la puerta de enlace

En este paso, habilite el modo activo-activo y actualice la puerta de enlace. En el ejemplo, la instancia de VPN Gateway usa actualmente una SKU estándar heredada. Sin embargo, el modo activo-activo no admite la SKU estándar. Para cambiar el tamaño de la SKU heredada a uno que sea compatible (en este caso, HighPerformance), solo tiene que especificar la SKU heredada compatible que quiere usar.

* No se puede cambiar una SKU heredada a una de las SKU nuevas con este paso. Solo puede cambiar el tamaño de una SKU heredada a otra SKU heredada compatible. Por ejemplo, no puede cambiar la SKU de estándar a VpnGw1 (aunque VpnGw1 se admite para el modo activo-activo) porque el nivel estándar incluye una SKU heredada y VpnGw1 incluye una SKU actual. Para obtener más información acerca de cómo cambiar el tamaño y migrar las SKU, consulte [SKU de Gateway](vpn-gateway-about-vpngateways.md#gwsku).

* Si desea cambiar el tamaño de una SKU actual, por ejemplo, VpnGw1 a VpnGw3, puede hacerlo mediante este paso porque las SKU se encuentran en la misma familia. Para ello, debe usar el valor: ```-GatewaySku VpnGw3```.

Al usar este valor en su entorno, si no es necesario cambiar el tamaño de la puerta de enlace, no tendrá que especificar -GatewaySku. Tenga en cuenta que en este paso debe establecer el objeto de puerta de enlace en PowerShell para desencadenar la actualización real. Esta actualización puede tardar entre 30 y 45 minutos, incluso si no va a cambiar el tamaño de la puerta de enlace.

```azurepowershell-interactive
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -EnableActiveActiveFeature -GatewaySku HighPerformance
```

### <a name="change-an-active-active-gateway-to-an-active-standby-gateway"></a>Cambio de una puerta de enlace de activo-activo a una puerta de enlace activo-en espera
#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Reemplace los siguientes parámetros utilizados para los ejemplos con los valores que necesita para su propia configuración y, a continuación, declare estas variables.

```azurepowershell-interactive
$GWName = "TestVNetAA1GW"
$RG = "TestVPNActiveActive01"
```

Tras declarar las variables, obtenga el nombre de la configuración IP que desea quitar.

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$ipconfname = $gw.IpConfigurations[1].Name
```

#### <a name="2-remove-the-gateway-ip-configuration-and-disable-the-active-active-mode"></a>2. Quite la configuración de IP de puerta de enlace y deshabilite el modo activo-activo

Use este ejemplo para quitar la configuración IP de la puerta de enlace y deshabilitar el modo activo-activo. Tenga en cuenta que debe establecer el objeto de puerta de enlace en PowerShell para desencadenar la actualización real.

```azurepowershell-interactive
Remove-AzVirtualNetworkGatewayIpConfig -Name $ipconfname -VirtualNetworkGateway $gw
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -DisableActiveActiveFeature
```

Esta actualización puede tardar hasta 30 o 45 minutos.

## <a name="next-steps"></a>Pasos siguientes
Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Creación de una máquina virtual que ejecuta Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) para ver los pasos.
