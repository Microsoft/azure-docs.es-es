---
title: 'Azure VPN Gateway: Configuración de BGP: PowerShell'
description: Este artículo le guiará a la hora de configurar BGP con puertas de enlace de VPN de Azure por medio de Azure Resource Manager y PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5abfa26f5d897ade963253da81927a48cc65d781
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394081"
---
# <a name="how-to-configure-bgp-on-azure-vpn-gateways-using-powershell"></a>Configuración de BGP para Azure VPN Gateway con PowerShell
Este artículo le guiará por los pasos para habilitar BGP en una conexión de VPN de sitio a sitio (S2S) entre locales y una conexión de red virtual a red virtual mediante el modelo de implementación de Resource Manager y PowerShell.



## <a name="about-bgp"></a>Información acerca de BGP
BGP es el protocolo de enrutamiento estándar usado habitualmente en Internet para intercambiar información de enrutamiento y disponibilidad entre dos o más redes. BGP permite que las puertas de enlace de VPN de Azure y los dispositivos VPN locales, denominados vecinos o pares BGP, intercambien "rutas" que comunicarán a ambas puertas de enlace la disponibilidad y la posibilidad de que dichos prefijos pasen a través de las puertas de enlace o los enrutadores implicados. BGP también puede permitir el enrutamiento del tránsito entre varias redes mediante la propagación de las rutas que una puerta de enlace de BGP aprende de un par BGP a todos los demás pares BGP.

Para obtener más información sobre las ventajas de BGP, así como entender los requisitos técnicos y las consideraciones sobre el uso de BGP, consulte [Información general de BGP con Azure VPN Gateway](vpn-gateway-bgp-overview.md).

## <a name="getting-started-with-bgp-on-azure-vpn-gateways"></a>Introducción a BGP en puertas de enlace de VPN de Azure

Este artículo lo guiará a través de los pasos necesarios para realizar las tareas siguientes:

* [Parte 1: Habilitar BGP en la puerta de enlace de VPN de Azure](#enablebgp)
* Parte 2: Establecer una conexión entre locales con BGP
* [Parte 3: Establecer una conexión de red virtual a red virtual con BGP](#v2vbgp)

Cada parte de las instrucciones constituye un bloque de creación básico para habilitar BGP en la conectividad de red. Si completa las tres partes, podrá crear la topología tal como se muestra en el diagrama siguiente:

![Topología de BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

Puede combinar estos elementos juntos para crear una red de tránsito más compleja y de saltos múltiples que satisfaga sus necesidades.

## <a name="part-1---configure-bgp-on-the-azure-vpn-gateway"></a><a name ="enablebgp"></a>Parte 1: Configurar BGP en Azure VPN Gateway
Los siguientes pasos de configuración permiten establecer los parámetros BGP de Azure VPN Gateway como se muestra en el diagrama siguiente:

![Puerta de enlace de BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>Antes de empezar
* Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Instale los cmdlets de PowerShell de Azure Resource Manager. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/) para más información sobre cómo instalar los cmdlets de PowerShell. 

### <a name="step-1---create-and-configure-vnet1"></a>Paso 1: Creación y configuración de VNet1
#### <a name="1-declare-your-variables"></a>1. Declaración de las variables
Para este ejercicio, se empieza por declarar las variables. En este ejemplo se declaran las variables con los valores para este ejercicio. Asegúrese de reemplazar los valores por los suyos propios cuando realice la configuración para el entorno de producción. Puede usar estas variables si está practicando los pasos para familiarizarse con este tipo de configuración. Modifique las variables y después copie y pegue todo en la consola de PowerShell.

```powershell
$Sub1 = "Replace_With_Your_Subscription_Name"
$RG1 = "TestBGPRG1"
$Location1 = "East US"
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
$GWIPName1 = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection12 = "VNet1toVNet2"
$Connection15 = "VNet1toSite5"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2. Conexión a su suscripción y creación de un nuevo grupo de recursos
Para usar los cmdlets de Resource Manager, asegúrese de cambiar al modo de PowerShell. Para obtener más información, consulte [Uso de Windows PowerShell con el Administrador de recursos](../powershell-azure-resource-manager.md).

Abre la consola de PowerShell y conéctate a tu cuenta. Use el siguiente ejemplo para ayudarle a conectarse:

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3. Creación de TestVNet1
En el siguiente ejemplo se crea una red virtual denominada "TestVNet1" y tres subredes: GatewaySubnet, FrontEnd y Backend. Al reemplazar valores, es importante que siempre asigne el nombre GatewaySubnet a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace.

```powershell
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>Paso 2: Creación de la puerta de enlace de VPN para TestVNet1 con parámetros BGP
#### <a name="1-create-the-ip-and-subnet-configurations"></a>1. Cree las configuraciones de IP y de subred
Solicite que se asigne una dirección IP pública a la puerta de enlace que creará para la red virtual. También definirá las configuraciones de subred y de IP necesarias.

```powershell
$gwpip1 = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2. Cree la puerta de enlace de VPN con el número de AS
Cree la puerta de enlace de red virtual para TestVNet1. BGP requiere una VPN Gateway basada en una ruta y también el parámetro de suma, - Asn, con el fin de establecer el ASN (número de AS) para TestVNet1. Si no establece el parámetro ASN, se asignará ASN 65515. Se tardan unos 30 minutos o algo más en crear una puerta de enlace.

```powershell
New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3. Obtenga la dirección IP del par BGP de Azure
Una vez creada la puerta de enlace, debe obtener la dirección IP del par BGP en Azure VPN Gateway. Esta dirección es necesaria para configurar la instancia de Azure VPN Gateway como par BGP para los dispositivos de VPN local.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet1gw.BgpSettingsText
```

El último comando mostrará las configuraciones de BGP correspondientes en Azure VPN Gateway; por ejemplo:

```powershell
$vnet1gw.BgpSettingsText
{
    "Asn": 65010,
    "BgpPeeringAddress": "10.12.255.30",
    "PeerWeight": 0
}
```

Una vez creada la puerta de enlace, puede usar esta puerta de enlace para establecer conexión entre locales o conexión de red virtual a red virtual con BGP. Las siguientes secciones lo guiarán por los pasos necesarios para completar el ejercicio.

## <a name="part-2---establish-a-cross-premises-connection-with-bgp"></a><a name ="crossprembbgp"></a>Parte 2: Establecer una conexión entre locales con BGP

Para establecer una conexión entre locales, debe crear una puerta de enlace de red local para representar el dispositivo VPN local y una conexión para conectarse a VPN Gateway con la puerta de enlace de red local. Aunque hay artículos que lo guiarán a través de estos pasos, este contiene las propiedades adicionales necesarias para especificar los parámetros de configuración de BGP.

![BGP entre locales](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)

Antes de continuar, asegúrese de que ha completado la [parte 1](#enablebgp) de este ejercicio.

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>Paso 1: Cree y configure la puerta de enlace de red local

#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Este ejercicio es continuación del paso de creación de la configuración mostrada en el diagrama. Asegúrese de reemplazar los valores por los que desea usar para su configuración.

```powershell
$RG5 = "TestBGPRG5"
$Location5 = "East US 2"
$LNGName5 = "Site5"
$LNGPrefix50 = "10.52.255.254/32"
$LNGIP5 = "Your_VPN_Device_IP"
$LNGASN5 = 65050
$BGPPeerIP5 = "10.52.255.254"
```

Un par de cosas a tener en cuenta con respecto a los parámetros de la puerta de enlace de red local:

* La puerta de enlace de red local puede estar en la misma ubicación y grupo de recursos que la puerta de enlace de VPN o en otros distintos. Este ejemplo los muestra en distintos grupos de recursos en diferentes ubicaciones.
* El prefijo que debe declarar para la puerta de enlace de red local es la dirección del host de la dirección IP del par BGP en el dispositivo VPN. En este caso, es un prefijo /32 de "10.52.255.254/32".
* Como recordatorio, debe usar diferentes ASN de BGP entre las redes locales y la red virtual de Azure. Si son iguales, tiene que cambiar el ASN de la red virtual si el dispositivo VPN local ya utiliza el ASN para emparejarse con otros vecinos de BGP.

Antes de continuar, asegúrese de que sigue conectado a la suscripción 1.

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2. Cree las puertas de enlace de red local para Site5

Asegúrese de crear el grupo de recursos si no está ya creado, antes de crear la puerta de enlace de red local. Observe los dos parámetros adicionales de la puerta de enlace de red local: Asn y BgpPeerAddress.

```powershell
New-AzResourceGroup -Name $RG5 -Location $Location5

New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>Paso 2: Conecte la puerta de enlace de red virtual y la puerta de enlace de red local

#### <a name="1-get-the-two-gateways"></a>1. Obtenga las dos puertas de enlace

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2. Cree la conexión de TestVNet1 a Site5

En este paso, creará la conexión de TestVNet1 a Site5. Debe especificar "-EnableBGP True" para habilitar BGP para esta conexión. Como se explicó anteriormente, es posible tener conexiones BGP y no BGP para la misma instancia de Azure VPN Gateway. A menos que BGP esté habilitado en la propiedad de conexión, Azure no habilitará BGP para esta conexión, aunque los parámetros BGP ya estén configurados en ambas puertas de enlace.

```powershell
New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

En el ejemplo siguiente se enumeran los parámetros que deberá especificar en la sección de configuración de BGP en el dispositivo VPN local para este ejercicio:

```

- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP    : 10.12.255.30
- Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

La conexión se establece después de unos minutos y se iniciará la sesión de emparejamiento BGP una vez establecida la conexión IPsec.

## <a name="part-3---establish-a-vnet-to-vnet-connection-with-bgp"></a><a name ="v2vbgp"></a>Parte 3: Establecer una conexión de red virtual a red virtual con BGP

En esta sección se agrega una conexión de red virtual a red virtual con BGP, tal como se muestra en el diagrama siguiente:

![BGP para conexiones de red virtual a red virtual](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

Las siguientes instrucciones son continuación de los pasos anteriores. Debe completar la [Parte 1](#enablebgp) para crear y configurar TestVNet1 y VPN Gateway con BGP. 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>Paso 1: cree TestVNet2 y la puerta de enlace de VPN

Es importante asegurarse de que el espacio de direcciones IP de la red virtual nueva, TestVNet2, no se solape con ninguno de sus intervalos de red virtual.

En este ejemplo, las redes virtuales pertenecen a la misma suscripción. Se pueden configurar conexiones de red virtual a red virtual entre distintas suscripciones. Para obtener más información, consulte [Configuración de una conexión de red virtual a red virtual](vpn-gateway-vnet-vnet-rm-ps.md). Asegúrese de agregar "-EnableBgp True" al crear las conexiones para habilitar BGP.

#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Asegúrese de reemplazar los valores por los que desea usar para su configuración.

```powershell
$RG2 = "TestBGPRG2"
$Location2 = "West US"
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
$GWIPName2 = "VNet2GWIP"
$GWIPconfName2 = "gwipconf2"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Cree TestVNet2 en el nuevo grupo de recursos

```powershell
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2
```

#### <a name="3-create-the-vpn-gateway-for-testvnet2-with-bgp-parameters"></a>3. Cree la puerta de enlace de VPN para TestVNet2 con parámetros BGP

Solicite que se asigne una dirección IP pública a la puerta de enlace que creará para la red virtual y defina las configuraciones de IP y subred requeridas.

```powershell
$gwpip2    = New-AzPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2     = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2   = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName2 -Subnet $subnet2 -PublicIpAddress $gwpip2
```

Cree la puerta de enlace de VPN con el número de AS. Debe reemplazar el valor predeterminado del ASN en Azure VPN Gateway. Los ASN para las redes virtuales conectadas deben ser diferentes para habilitar el enrutamiento de tránsito y de BGP.

```powershell
New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gwipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet2ASN
```

### <a name="step-2---connect-the-testvnet1-and-testvnet2-gateways"></a>Paso 2: Conecte las puertas de enlace de TestVNet1 y TestVNet2

En este ejemplo, ambas puertas de enlace están en la misma suscripción. Puede completar este paso en la misma sesión de PowerShell.

#### <a name="1-get-both-gateways"></a>1. Obtenga ambas puertas de enlace

Asegúrese de iniciar sesión y conectarse a la suscripción 1.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
```

#### <a name="2-create-both-connections"></a>2. Cree ambas conexiones

En este paso, creará la conexión de TestVNet1 a TestVNet2 y viceversa.

```powershell
New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> Asegúrese de habilitar BGP para AMBAS conexiones.
> 
> 

Después de completar estos pasos, se establece la conexión después de unos minutos. La sesión de emparejamiento BGP funcionará una vez completada la conexión de red virtual a red virtual.

Si ha completado las tres partes de este ejercicio, habrá establecido la siguiente topología de red:

![BGP para conexiones de red virtual a red virtual](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

## <a name="next-steps"></a>Pasos siguientes

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Creación de una máquina virtual que ejecuta Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md) para ver los pasos.
