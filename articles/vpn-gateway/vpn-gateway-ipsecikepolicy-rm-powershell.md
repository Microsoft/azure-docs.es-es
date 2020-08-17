---
title: Directiva IPsec o IKE para conexiones de VPN de sitio a sitio o entre redes virtuales
titleSuffix: Azure VPN Gateway
description: Configurar una directiva de IPsec o IKE para conexiones de sitio a sitio o de red virtual a red virtual con instancias de Azure VPN Gateway mediante Azure Resource Manager y PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/14/2018
ms.author: yushwang
ms.openlocfilehash: 21c24fba2cbe03b17a057c09f95d9dd0d3665dc2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87064535"
---
# <a name="configure-ipsecike-policy-for-s2s-vpn-or-vnet-to-vnet-connections"></a>Configurar una directiva de IPsec o IKE para conexiones VPN de sitio a sitio o de red virtual a red virtual

Este artículo le guiará por los pasos para configurar una directiva de IPsec o IKE para conexiones VPN de sitio a sitio o de red virtual a red virtual mediante el modelo de implementación de Resource Manager y PowerShell.



## <a name="about-ipsec-and-ike-policy-parameters-for-azure-vpn-gateways"></a><a name="about"></a>Acerca de los parámetros de la directiva de IPsec e IKE para Azure VPN gateway
El protocolo IPsec e IKE estándar admite una gran variedad de algoritmos criptográficos en diversas combinaciones. En [About cryptographic requirements and Azure VPN gateways](vpn-gateway-about-compliance-crypto.md) (Acerca de los requisitos de cifrado y las puertas de enlace de VPN de Azure) se describe la manera en que esto puede ayudar a garantizar que la conectividad entre locales y de red virtual a red virtual satisface los requisitos de cumplimiento o seguridad.

En este artículo se proporcionan instrucciones para crear y configurar una directiva de IPsec o IKE y aplicarla a una conexión nueva o existente:

* [Parte 1: flujo de trabajo para crear y establecer una directiva de IPsec o IKE](#workflow)
* [Parte 2: algoritmos criptográficos y niveles de clave admitidos](#params)
* [Parte 3: crear una conexión VPN de sitio a sitio con una directiva de IPsec o IKE](#crossprem)
* [Parte 4: crear una conexión de red virtual a red virtual con una directiva de IPsec o IKE](#vnet2vnet)
* [Parte 5: administrar (crear, agregar, quitar) una directiva de IPsec o IKE para una conexión](#managepolicy)

> [!IMPORTANT]
> 1. Tenga en cuenta que la directiva de IPsec/IKE solo funciona en las SKU de puerta de enlace siguiente:
>    * ***VpnGw1, VpnGw2, VpnGw3*** (basadas en enrutamiento)
>    * ***Standard*** y ***HighPerformance*** (basadas en enrutamiento)
> 2. Solo se puede especificar ***una*** combinación de directivas para una conexión dada.
> 3. Es preciso especificar todos los algoritmos y parámetros de IKE (modo principal) e IPsec (modo rápido). No se permite la especificación de una directiva parcial.
> 4. Consulte las especificaciones del proveedor de dispositivos VPN para asegurarse de que los dispositivos VPN locales admiten la directiva. No se pueden establecer conexiones de sitio a sitio o de red virtual a red virtual si las directivas son incompatibles.

## <a name="part-1---workflow-to-create-and-set-ipsecike-policy"></a><a name ="workflow"></a>Parte 1: flujo de trabajo para crear y establecer una directiva de IPsec o IKE
En esta sección se describe el flujo de trabajo para crear y actualizar una directiva de IPsec o IKE en una conexión VPN de sitio a sitio o de red virtual a red virtual:
1. Crear una red virtual y una puerta de enlace de VPN
2. Crear una puerta de enlace de red local para la conexión entre locales, u otra red virtual y puerta de enlace para la conexión de red virtual a red virtual
3. Crear una directiva de IPsec o IKE con los algoritmos y parámetros seleccionados
4. Crear una conexión (IPsec o de red virtual a red virtual) con la directiva de IPsec o IKE
5. Agregar, actualizar o quitar una directiva de IPsec o IKE para una conexión existente

Las instrucciones incluidas en este artículo le ayudan a instalar y configurar directivas de IPsec o IKE como se muestra en el diagrama:

![ipsec-ike-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/ipsecikepolicy.png)

## <a name="part-2---supported-cryptographic-algorithms--key-strengths"></a><a name ="params"></a>Parte 2: algoritmos criptográficos y niveles de clave admitidos

En la tabla siguiente se enumeran los algoritmos criptográficos y los niveles de las claves admitidos que pueden configurar los clientes:

| **IPsec o IKEv2**  | **Opciones**    |
| ---  | --- 
| Cifrado IKEv2 | AES256, AES192, AES128, DES3, DES  
| Integridad de IKEv2  | SHA384, SHA256, SHA1, MD5  |
| Grupo DH         | DHGroup24, ECP384, ECP256, DHGroup14, DHGroup2048, DHGroup2, DHGroup1, No |
| Cifrado IPsec | GCMAES256, GCMAES192, GCMAES128, AES256, AES192, AES128, DES3, DES, No    |
| Integridad de IPsec  | GCMASE256, GCMAES192, GCMAES128, SHA256, SHA1, MD5 |
| Grupo PFS        | PFS24, ECP384, ECP256, PFS2048, PFS2, PFS1, No 
| Vigencia de SA QM   | (**Opcional**: Se usan los valores predeterminados si no se especifica ningún valor)<br>Segundos (entero; **mín. 300**/predeterminado 27000 segundos)<br>KBytes (entero; **mín. 1024**/predeterminado 102400000 KBytes)   |
| Selector de tráfico | UsePolicyBasedTrafficSelectors** ($True/$False; **Opcional**, valor predeterminado $False si no se especifica)    |
|  |  |

> [!IMPORTANT]
> 1. **La configuración de su dispositivo VPN local debe coincidir o contener los siguientes algoritmos y parámetros que se especifican en la directiva de IPsec o IKE de Azure:**
>    * Algoritmo de cifrado de IKE (modo principal/fase 1)
>    * Algoritmo de integridad de IKE (modo principal/fase 1)
>    * Grupo DH (modo principal/fase 1)
>    * Algoritmo de cifrado de IPsec (modo rápido/fase 2)
>    * Algoritmo de integridad de IPsec (modo rápido/fase 2)
>    * Grupo PFS (modo rápido/fase 2)
>    * Selector de tráfico (si se usa UsePolicyBasedTrafficSelectors)
>    * Las vigencias de SA solo son especificaciones locales, no es preciso que coincidan.
>
> 2. **Si se usa GCMAES para el algoritmo de cifrado IPsec, se debe seleccionar el mismo algoritmo GCMAES y longitud de clave para la integridad de IPsec; por ejemplo, el uso de GCMAES128 para ambos**
> 3. En la tabla anterior:
>    * IKEv2 corresponde al modo principal o fase 1
>    * IPsec corresponde al modo rápido o fase 2
>    * El Grupo DH especifica el grupo Diffie-Hellmen utilizado en el modo principal o fase 1
>    * El grupo PFS especifica el grupo Diffie-Hellmen utilizado en el modo rápido o fase 2
> 4. La vigencia de SA del modo principal de IKEv2 se fija en 28 800 segundos en las puertas de enlace de VPN de Azure
> 5. Si establece "UsePolicyBasedTrafficSelectors" en $True en una conexión, configurará la puerta de enlace de VPN de Azure de modo que se conecte al firewall VPN basado en directivas de forma local. Si habilita PolicyBasedTrafficSelectors, debe asegurarse de que el dispositivo VPN tiene los selectores de tráfico coincidentes definidos con todas las combinaciones de sus prefijos de red local (puerta de enlace de red local) a o desde los prefijos de red virtual de Azure, en lugar de cualquiera a cualquiera. Por ejemplo, si sus prefijos de red local son 10.1.0.0/16 y 10.2.0.0/16, y sus prefijos de red virtual son 192.168.0.0/16 y 172.16.0.0/16, debe especificar los siguientes selectores de tráfico:
>    * 10.1.0.0/16 <====> 192.168.0.0/16
>    * 10.1.0.0/16 <====> 172.16.0.0/16
>    * 10.2.0.0/16 <====> 192.168.0.0/16
>    * 10.2.0.0/16 <====> 172.16.0.0/16

Para obtener más información sobre los selectores de tráfico basados en directivas, consulte [Connect multiple on-premises policy-based VPN devices](vpn-gateway-connect-multiple-policybased-rm-ps.md) (Conectar varios dispositivos VPN basados en directivas locales).

En la tabla siguiente se muestran los grupos Diffie-Hellman admitidos en la directiva personalizada:

| **Grupo Diffie-Hellman**  | **Grupo DH**              | **Grupo PFS** | **Longitud de clave** |
| --- | --- | --- | --- |
| 1                         | DHGroup1                 | PFS1         | MODP de 768 bits   |
| 2                         | DHGroup2                 | PFS2         | MODP de 1024 bits  |
| 14                        | DHGroup14<br>DHGroup2048 | PFS2048      | MODP de 2048 bits  |
| 19                        | ECP256                   | ECP256       | ECP de 256 bits    |
| 20                        | ECP384                   | ECP384       | ECP de 384 bits    |
| 24                        | DHGroup24                | PFS24        | MODP de 2048 bits  |

Consulte [RFC3526](https://tools.ietf.org/html/rfc3526) y [RFC5114](https://tools.ietf.org/html/rfc5114) para ver información más detallada.

## <a name="part-3---create-a-new-s2s-vpn-connection-with-ipsecike-policy"></a><a name ="crossprem"></a>Parte 3: crear una conexión VPN de sitio a sitio con una directiva de IPsec o IKE

En esta sección se describen los pasos necesarios para crear una conexión VPN de sitio a sitio con una directiva de IPsec o IKE. Con los pasos siguientes crea la conexión como se muestra en el diagrama:

![s2s-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/s2spolicy.png)

Consulte [Creación de una conexión VPN de sitio a sitio](vpn-gateway-create-site-to-site-rm-powershell.md) para obtener instrucciones detalladas sobre cómo crear una conexión VPN de sitio a sitio.

### <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

* Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Instale los cmdlets de PowerShell de Azure Resource Manager. Vea [Información general sobre Azure PowerShell](/powershell/azure/) para obtener más información sobre la instalación de los cmdlets de PowerShell.

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a><a name="createvnet1"></a>Paso 1: Creación de la red virtual, VPN Gateway y la puerta de enlace de red local

#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Para este ejercicio, se empieza por declarar las variables. Asegúrese de reemplazar los valores por los suyos propios cuando realice la configuración para el entorno de producción.

```powershell
$Sub1          = "<YourSubscriptionName>"
$RG1           = "TestPolicyRG1"
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
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GW1IPName1    = "VNet1GWIP1"
$GW1IPconf1    = "gw1ipconf1"
$Connection16  = "VNet1toSite6"

$LNGName6      = "Site6"
$LNGPrefix61   = "10.61.0.0/16"
$LNGPrefix62   = "10.62.0.0/16"
$LNGIP6        = "131.107.72.22"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2. Conexión a su suscripción y creación de un nuevo grupo de recursos

Asegúrese de cambiar el modo de PowerShell para que use los cmdlets del Administrador de recursos. Para obtener más información, consulte [Uso de Windows PowerShell con el Administrador de recursos](../powershell-azure-resource-manager.md).

Abre la consola de PowerShell y conéctate a tu cuenta. Use el siguiente ejemplo para ayudarle a conectarse:

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>3. Creación de la red virtual, la puerta de enlace de VPN y la puerta de enlace de red local

En el ejemplo siguiente se crea la red virtual, TestVNet1, con tres subredes y VPN Gateway. Al reemplazar valores, es importante que siempre asigne el nombre GatewaySubnet a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace.

```powershell
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

$gw1pip1    = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1      = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1

New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1

New-AzLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
```

### <a name="step-2---create-a-s2s-vpn-connection-with-an-ipsecike-policy"></a><a name="s2sconnection"></a>Paso 2: Creación de una conexión VPN de sitio a sitio con una directiva IPsec/IKE

#### <a name="1-create-an-ipsecike-policy"></a>1. Cree una directiva IPsec/IKE.

El script de ejemplo siguiente crea una directiva de IPsec o IKE con los algoritmos y parámetros siguientes:

* IKEv2: AES256, SHA384 y DHGroup24
* IPsec: AES256, SHA256, sin PFS, 14 400 segundos de vigencia de SA y 102 400 000 KB

```powershell
$ipsecpolicy6 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
```

Si usa GCMAES para IPsec, debe usar el mismo algoritmo GCMAES y longitud de clave para la integridad y el cifrado IPsec. En el ejemplo anterior, los parámetros correspondientes serán "-IpsecEncryption GCMAES256 -IpsecIntegrity GCMAES256" cuando se use GCMAES256.

#### <a name="2-create-the-s2s-vpn-connection-with-the-ipsecike-policy"></a>2. Crear la conexión VPN de sitio a sitio con la directiva de IPsec o IKE

Cree una conexión VPN de sitio a sitio y aplique la directiva de IPsec o IKE creada anteriormente.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng6 = Get-AzLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
```

Opcionalmente, puede agregar "-UsePolicyBasedTrafficSelectors $True" al cmdlet para crear la conexión a fin de permitir que la puerta de enlace de VPN se conecte a dispositivos VPN basados en directivas de forma local, como se ha descrito anteriormente.

> [!IMPORTANT]
> Una vez que se haya especificado una directiva de IPsec o IKE en una conexión, la puerta de enlace de VPN de Azure solo enviará o aceptará la propuesta de IPsec o IKE con los algoritmos criptográficos y los niveles de clave especificados en esa conexión en particular. Asegúrese de que el dispositivo VPN local para la conexión usa o acepta la combinación de directivas exacta. En caso contrario, no se establecerá el túnel VPN de sitio a sitio.


## <a name="part-4---create-a-new-vnet-to-vnet-connection-with-ipsecike-policy"></a><a name ="vnet2vnet"></a>Parte 4: crear una conexión de red virtual a red virtual con una directiva de IPsec o IKE

Los pasos necesarios para crear una conexión de red virtual a red virtual con una directiva de IPsec o IKE son similares a los pasos para crear una conexión VPN de sitio a sitio. Con los scripts de ejemplo siguientes crea la conexión como se muestra en el diagrama:

![v2v-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/v2vpolicy.png)

Consulte [Creación de una conexión de red virtual a red virtual](vpn-gateway-vnet-vnet-rm-ps.md) para conocer los pasos detallados para crear una conexión de red virtual a red virtual. Tiene que completar la [parte 3](#crossprem) para crear y configurar TestVNet1 y VPN Gateway.

### <a name="step-1---create-the-second-virtual-network-and-vpn-gateway"></a><a name="createvnet2"></a>Paso 1: Creación de la segunda red virtual y VPN Gateway

#### <a name="1-declare-your-variables"></a>1. Declaración de las variables

Asegúrese de reemplazar los valores por los que desea usar para su configuración.

```powershell
$RG2          = "TestPolicyRG2"
$Location2    = "East US 2"
$VNetName2    = "TestVNet2"
$FESubName2   = "FrontEnd"
$BESubName2   = "Backend"
$GWSubName2   = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$DNS2         = "8.8.8.8"
$GWName2      = "VNet2GW"
$GW2IPName1   = "VNet2GWIP1"
$GW2IPconf1   = "gw2ipconf1"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-the-second-virtual-network-and-vpn-gateway-in-the-new-resource-group"></a>2. Creación de la segunda red virtual y la puerta de enlace de VPN en el nuevo grupo de recursos

```powershell
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2

$gw2pip1    = New-AzPublicIpAddress -Name $GW2IPName1 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic
$vnet2      = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gw2ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW2IPconf1 -Subnet $subnet2 -PublicIpAddress $gw2pip1

New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gw2ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance
```

### <a name="step-2---create-a-vnet-tovnet-connection-with-the-ipsecike-policy"></a>Paso 2: Creación de una conexión de red virtual a red virtual con la directiva de IPsec o IKE

De forma similar a la conexión VPN de sitio a sitio, cree una directiva de IPsec o IKE y aplíquela a la nueva conexión.

#### <a name="1-create-an-ipsecike-policy"></a>1. Cree una directiva IPsec/IKE.

El script de ejemplo siguiente crea una directiva de IPsec o IKE diferente con los algoritmos y parámetros siguientes:
* IKEv2: AES128, SHA1, DHGroup14
* IPsec: GCMAES128, GCMAES128, PFS14, 14 400 segundos de vigencia de SA y 102 400 000 KB

```powershell
$ipsecpolicy2 = New-AzIpsecPolicy -IkeEncryption AES128 -IkeIntegrity SHA1 -DhGroup DHGroup14 -IpsecEncryption GCMAES128 -IpsecIntegrity GCMAES128 -PfsGroup PFS14 -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
```

#### <a name="2-create-vnet-to-vnet-connections-with-the-ipsecike-policy"></a>2. Crear conexiones de red virtual a red virtual con la directiva de IPsec o IKE

Cree una conexión de red virtual a red virtual y aplique la directiva de IPsec o IKE creada. En este ejemplo, ambas puertas de enlace están en la misma suscripción. Por esta razón, es posible crear y configurar las dos conexiones con la misma directiva de IPsec o IKE en la misma sesión de PowerShell.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2  -ResourceGroupName $RG2

New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -IpsecPolicies $ipsecpolicy2 -SharedKey 'AzureA1b2C3'

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -IpsecPolicies $ipsecpolicy2 -SharedKey 'AzureA1b2C3'
```

> [!IMPORTANT]
> Una vez que se haya especificado una directiva de IPsec o IKE en una conexión, la puerta de enlace de VPN de Azure solo enviará o aceptará la propuesta de IPsec o IKE con los algoritmos criptográficos y los niveles de clave especificados en esa conexión en particular. Asegúrese de que las directivas de IPsec para ambas conexiones son iguales. En caso contrario, no se establecerá la conexión de red virtual a red virtual.

Después de completar estos pasos, la conexión se establecerá al cabo de unos minutos y tendrá la topología de red siguiente, tal como se mostró al principio:

![ipsec-ike-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/ipsecikepolicy.png)


## <a name="part-5---update-ipsecike-policy-for-a-connection"></a><a name ="managepolicy"></a>Parte 5: actualizar una directiva de IPsec o IKE para una conexión

En la última sección se muestra cómo administrar la directiva de IPsec o IKE para una conexión existente de sitio a sitio o de red virtual a red virtual. En el ejercicio siguiente se explica cómo se realizan las siguientes operaciones en una conexión:

1. Mostrar la directiva de IPsec o IKE de una conexión
2. Agregar o actualizar la directiva de IPsec o IKE para una conexión
3. Eliminar la directiva de IPsec o IKE de una conexión

Los mismos pasos se aplican a las conexiones de sitio a sitio y de red virtual a red virtual.

> [!IMPORTANT]
> La directiva de IPsec o IKE solo se admite en las puertas de enlace de VPN basadas en rutas *Standard* y *HighPerformance*. No funciona en la SKU de puerta de enlace básica o la puerta de enlace de VPN basada en directivas.

#### <a name="1-show-the-ipsecike-policy-of-a-connection"></a>1. Mostrar la directiva de IPsec o IKE de una conexión

En el ejemplo siguiente se muestra cómo obtener la directiva de IPsec o IKE configurada en una conexión. Los scripts son una continuación de los ejercicios anteriores.

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
$connection6.IpsecPolicies
```

El último comando muestra la directiva de IPsec o IKE actual configurada en la conexión, si existe. La siguiente es una salida de ejemplo para la conexión:

```powershell
SALifeTimeSeconds   : 14400
SADataSizeKilobytes : 102400000
IpsecEncryption     : AES256
IpsecIntegrity      : SHA256
IkeEncryption       : AES256
IkeIntegrity        : SHA384
DhGroup             : DHGroup24
PfsGroup            : PFS24
```

Si no hay ninguna directiva de IPsec o IKE configurada, el comando (PS> $connection6.IpsecPolicies) obtiene un valor devuelto vacío. Esto no significa que IPsec o IKE no esté configurado en la conexión, sino que no hay ninguna directiva de IPsec o IKE personalizada. La conexión real usa la directiva predeterminada que se negocia entre el dispositivo VPN local y Azure VPN Gateway.

#### <a name="2-add-or-update-an-ipsecike-policy-for-a-connection"></a>2. Agregar o actualizar una directiva de IPsec o IKE para una conexión

Los pasos para agregar una nueva directiva o actualizar una directiva existente en una conexión son los mismos: crear una directiva y, después, aplicar la nueva directiva a la conexión.

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

$newpolicy6   = New-AzIpsecPolicy -IkeEncryption AES128 -IkeIntegrity SHA1 -DhGroup DHGroup14 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000

Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -IpsecPolicies $newpolicy6
```

Para habilitar "UsePolicyBasedTrafficSelectors" al conectarse a un dispositivo VPN local basado en directivas, agregue el parámetro "-UsePolicyBaseTrafficSelectors" al cmdlet o establézcalo en $False para deshabilitar la opción:

```powershell
Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -IpsecPolicies $newpolicy6 -UsePolicyBasedTrafficSelectors $True
```

Puede obtener la conexión de nuevo para comprobar si la directiva está actualizada.

```powershell
$connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
$connection6.IpsecPolicies
```

Debería ver la salida de la última línea tal como se muestra en el ejemplo siguiente:

```powershell
SALifeTimeSeconds   : 14400
SADataSizeKilobytes : 102400000
IpsecEncryption     : AES256
IpsecIntegrity      : SHA256
IkeEncryption       : AES128
IkeIntegrity        : SHA1
DhGroup             : DHGroup14
PfsGroup            : None
```

#### <a name="3-remove-an-ipsecike-policy-from-a-connection"></a>3. Eliminar una directiva de IPsec o IKE de una conexión

Una vez que se quite la directiva personalizada de una conexión, Azure VPN Gateway vuelve a la [lista predeterminada de propuestas de IPsec o IKE](vpn-gateway-about-vpn-devices.md) y vuelve a negociar con el dispositivo VPN local.

```powershell
$RG1           = "TestPolicyRG1"
$Connection16  = "VNet1toSite6"
$connection6   = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

$currentpolicy = $connection6.IpsecPolicies[0]
$connection6.IpsecPolicies.Remove($currentpolicy)

Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6
```

Puede usar el mismo script para comprobar si la directiva se ha quitado de la conexión.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre los selectores de tráfico basados en directivas, vea [Connect multiple on-premises policy-based VPN devices](vpn-gateway-connect-multiple-policybased-rm-ps.md) (Conectar varios dispositivos VPN basados en directivas locales).

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Creación de una máquina virtual que ejecuta Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md) para ver los pasos.
