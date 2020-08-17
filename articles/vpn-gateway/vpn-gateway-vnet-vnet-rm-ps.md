---
title: 'Conexión de una red virtual a otra mediante una conexión entre redes virtuales de Azure VPN Gateway: PowerShell'
description: Conecte redes virtuales entre sí mediante una conexión entre redes virtuales y PowerShell.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/15/2019
ms.author: cherylmc
ms.openlocfilehash: 318656a779aa5ba2279f075fcc44589fe2039682
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87009499"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-powershell"></a>Configuración de una conexión de VPN Gateway de red virtual a red virtual mediante PowerShell

Este artículo le ayuda a conectarse a redes virtuales mediante el tipo de conexión de red virtual a red virtual. Las redes virtuales pueden estar en la misma región o en distintas, así como pertenecer a una única suscripción o a varias. Al conectar redes virtuales de distintas suscripciones, estas no necesitan estar asociadas con el mismo inquilino de Active Directory.

Los pasos descritos en este artículo se aplican al modelo de implementación de Resource Manager y utilizan PowerShell. También se puede crear esta configuración con una herramienta o modelo de implementación distintos, mediante la selección de una opción diferente en la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [CLI de Azure](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Portal de Azure clásico](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Conexión de diferentes modelos de implementación - Azure Portal](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Conexión de diferentes modelos de implementación - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)

## <a name="about-connecting-vnets"></a><a name="about"></a>Acerca de la conexión de redes virtuales

Hay varias formas de conectar redes virtuales. Las siguientes secciones describen distintas formas de conectar redes virtuales.

### <a name="vnet-to-vnet"></a>De red virtual a red virtual

La configuración de una conexión entre redes virtuales es una buena manera de conectar redes virtuales fácilmente. La conexión de una red virtual a otra mediante el tipo de conexión entre redes virtuales (VNet2VNet) es parecida a la creación de una conexión IPsec de sitio a sitio en una ubicación local.  Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro mediante IPsec/IKE y los dos funcionan de la misma forma en lo relativo a la comunicación. La diferencia entre ambos tipos de conexión radica en la manera en que se configura la puerta de enlace de red local. Al crear una conexión entre redes virtuales, no se ve el espacio de direcciones de la puerta de enlace de red local. Se crea y rellena automáticamente. Si actualiza el espacio de direcciones de una de las redes virtuales, la otra sabe automáticamente cómo realizar el enrutamiento al espacio de direcciones actualizado. La creación de una conexión entre redes virtuales suele ser más rápida y sencilla que la creación de una conexión de sitio a sitio entre redes virtuales.

### <a name="site-to-site-ipsec"></a>De sitio a sitio (IPsec)

Si puntualmente trabaja con una configuración de red complicada, puede que prefiera conectar sus redes virtuales mediante los pasos que se indican en [Creación de una red virtual con una conexión VPN de sitio a sitio mediante PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md), en lugar de las instrucciones para la conexión entre redes virtuales. Cuando se usan las instrucciones para la conexión de sitio a sitio, las puertas de enlace de red locales se crean y se configuran manualmente. La puerta de enlace de red local de cada red virtual trata a la otra red virtual como un sitio local. De esta forma, puede especificar más espacio de direcciones para la puerta de enlace de red local a fin de enrutar el tráfico. Si el espacio de direcciones de una red virtual cambia, es preciso que actualice la puerta de enlace de red local correspondiente para reflejar dicho cambio. No se actualiza automáticamente.

### <a name="vnet-peering"></a>Emparejamiento de VNET

Puede que desee considerar conectar sus redes virtuales mediante el emparejamiento de VNET. El emparejamiento de VNET no utiliza una puerta de enlace de VPN y tiene distintas restricciones. Además, [los precios del emparejamiento de VNET](https://azure.microsoft.com/pricing/details/virtual-network) se calculan de forma diferente que los [precios de VPN Gateway entre redes virtuales](https://azure.microsoft.com/pricing/details/vpn-gateway). Para más información, consulte [Emparejamiento de VNET](../virtual-network/virtual-network-peering-overview.md).

## <a name="why-create-a-vnet-to-vnet-connection"></a><a name="why"></a>¿Por qué crear una conexión de red virtual a red virtual?

Puede que desee conectar redes virtuales con una conexión entre redes virtuales por las siguientes razones:

* **Presencia geográfica y redundancia geográfica entre regiones**

  * Puede configurar su propia replicación geográfica o sincronización con conectividad segura sin recurrir a los puntos de conexión a Internet.
  * Con el Equilibrador de carga y el Administrador de tráfico de Azure, puede configurar cargas de trabajo de alta disponibilidad con redundancia geográfica en varias regiones de Azure. Por ejemplo, puede configurar AlwaysOn de SQL con grupos de disponibilidad distribuidos en varias regiones de Azure.
* **Aplicaciones regionales de niveles múltiples con aislamiento o un límite administrativo**

  * Dentro de la misma región, se pueden configurar aplicaciones de niveles múltiples con varias redes virtuales conectadas entre sí para cumplir requisitos administrativos o de aislamiento.

Se puede combinar la comunicación entre redes virtuales con configuraciones de varios sitios. Esto permite establecer topologías de red que combinen la conectividad entre entornos con la conectividad entre redes virtuales.

## <a name="which-vnet-to-vnet-steps-should-i-use"></a><a name="steps"></a>¿Qué instrucciones para la conexión entre redes virtuales debo seguir?

En este artículo, verá dos conjuntos de pasos diferentes. Un conjunto de pasos para las [redes virtuales que residen en la misma suscripción](#samesub) y otro para las [redes virtuales que residen en suscripciones diferentes](#difsub).
La principal diferencia entre ambos conjuntos es que debe utilizar sesiones de PowerShell independientes al configurar las conexiones de redes virtuales que residen en distintas suscripciones. 

Para este ejercicio, puede combinar las configuraciones, o bien elegir con la que desea trabajar. Todas las configuraciones utilizan el tipo de conexión entre redes virtuales. Flujos de tráfico de red entre las redes virtuales que están directamente conectados entre sí. En este ejercicio, el tráfico de TestVNet4 no se enruta a TestVNet5.

* [Redes virtuales que residen en la misma suscripción](#samesub): En los pasos para esta configuración se utilizan TestVNet1 y TestVNet4.

  ![diagrama de v2v](./media/vpn-gateway-vnet-vnet-rm-ps/v2vrmps.png)

* [Redes virtuales que residen en distintas suscripciones](#difsub): En los pasos para esta configuración se utilizan TestVNet1 y TestVNet5.

  ![diagrama de v2v](./media/vpn-gateway-vnet-vnet-rm-ps/v2vdiffsub.png)

## <a name="how-to-connect-vnets-that-are-in-the-same-subscription"></a><a name="samesub"></a>Conexión de redes virtuales que están en la misma suscripción

### <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* Dado que se tarda hasta 45 minutos en crear una puerta de enlace, el tiempo de Azure Cloud Shell expirara periódicamente durante este ejercicio. Para reiniciar Cloud Shell, haga clic en la esquina superior izquierda del terminal. No olvide volver a declarar las variables cuando se reinicie el terminal.

* Si prefiere instalar la versión más reciente del módulo Azure PowerShell localmente, consulte el artículo de [Instalación y configuración de Azure PowerShell](/powershell/azure/).

### <a name="step-1---plan-your-ip-address-ranges"></a><a name="Step1"></a>Paso 1: Planeamiento de los intervalos de direcciones IP

En los pasos siguientes, se crean dos redes virtuales junto con sus subredes de puerta de enlace y configuraciones correspondientes. Luego, se crea una conexión VPN entre las dos redes virtuales. Es importante planear los intervalos de direcciones IP para la configuración de red. Tenga en cuenta que hay que asegurarse de que ninguno de los intervalos de VNet o intervalos de red local se solapen. En estos ejemplos, no se incluye ningún servidor DNS. Si desea disponer de resolución de nombres en las redes virtuales, consulte [Resolución de nombres](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

En los ejemplos usamos los siguientes valores:

**Valores para TestVNet1:**

* Nombre de la red virtual: TestVNet1
* Grupo de recursos: TestRG1
* Ubicación: Este de EE. UU.
* TestVNet1: 10.11.0.0/16 y 10.12.0.0/16
* front-end: 10.11.0.0/24
* BackEnd: 10.12.0.0/24
* GatewaySubnet: 10.12.255.0/27
* GatewayName: VNet1GW
* Public IP: VNet1GWIP
* VPNType: RouteBased
* Connection(1to4): VNet1toVNet4
* Connection(1to5): VNet1toVNet5 (para redes virtuales de suscripciones distintas)
* ConnectionType: VNet2VNet

**Valores para TestVNet4:**

* Nombre de la red virtual: TestVNet4
* TestVNet2: 10.41.0.0/16 y 10.42.0.0/16
* front-end: 10.41.0.0/24
* BackEnd: 10.42.0.0/24
* GatewaySubnet: 10.42.255.0/27
* Grupo de recursos: TestRG4
* Ubicación: Oeste de EE. UU.
* GatewayName: VNet4GW
* Public IP: VNet4GWIP
* VPNType: RouteBased
* Conexión: VNet4toVNet1
* ConnectionType: VNet2VNet


### <a name="step-2---create-and-configure-testvnet1"></a><a name="Step2"></a>Paso 2: Creación y configuración de TestVNet1

1. Compruebe la configuración de la suscripción.

   Si ejecuta Azure PowerShell localmente en el equipo, conéctese a la cuenta. Si usa Azure Cloud Shell, se conectará automáticamente.

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

   Compruebe las suscripciones para la cuenta.

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

   Si tiene varias suscripciones, seleccione la que quera usar.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName nameofsubscription
   ```
2. Declare las variables. En este ejemplo se declaran las variables con los valores para este ejercicio. En la mayoría de los casos, deberá reemplazar los valores por los suyos propios. No obstante, puede usar estas variables si está practicando los pasos para familiarizarse con este tipo de configuración. Si es necesario, modifique las variables y después cópielas y péguelas en la consola de PowerShell.

   ```azurepowershell-interactive
   $RG1 = "TestRG1"
   $Location1 = "East US"
   $VNetName1 = "TestVNet1"
   $FESubName1 = "FrontEnd"
   $BESubName1 = "Backend"
   $VNetPrefix11 = "10.11.0.0/16"
   $VNetPrefix12 = "10.12.0.0/16"
   $FESubPrefix1 = "10.11.0.0/24"
   $BESubPrefix1 = "10.12.0.0/24"
   $GWSubPrefix1 = "10.12.255.0/27"
   $GWName1 = "VNet1GW"
   $GWIPName1 = "VNet1GWIP"
   $GWIPconfName1 = "gwipconf1"
   $Connection14 = "VNet1toVNet4"
   $Connection15 = "VNet1toVNet5"
   ```
3. Cree un grupo de recursos.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG1 -Location $Location1
   ```
4. Cree las configuraciones de subred para TestVNet1. En este ejemplo se crea una red virtual denominada TestVNet1 y tres subredes llamadas: GatewaySubnet, FrontEnd y Backend. Al reemplazar valores, es importante que siempre asigne el nombre GatewaySubnet a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace. Por esta razón, no se asigna a través de la variable siguiente.

   El siguiente ejemplo usa las variables que estableció anteriormente. En este ejemplo, la subred de la puerta de enlace está usando un /27. Aunque es posible crear una subred de puerta de enlace tan pequeña como /29, se recomienda que cree una subred mayor que incluya más direcciones seleccionando al menos /28 o /27. Esto permitirá suficientes direcciones para dar cabida a posibles configuraciones adicionales que desee en el futuro.

   ```azurepowershell-interactive
   $fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
   $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
   $gwsub1 = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix $GWSubPrefix1
   ```
5. Cree TestVNet1.

   ```azurepowershell-interactive
   New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 `
   -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
   ```
6. Solicite que se asigne una dirección IP pública a la puerta de enlace que creará para la red virtual. Observe que AllocationMethod es dinámico. No puede especificar la dirección IP que desea usar. Se asigna dinámicamente a la puerta de enlace. 

   ```azurepowershell-interactive
   $gwpip1 = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 `
   -Location $Location1 -AllocationMethod Dynamic
   ```
7. Establezca la configuración de la puerta de enlace. La configuración de puerta de enlace define la subred y la dirección IP pública. Use el ejemplo para crear la configuración de la puerta de enlace.

   ```azurepowershell-interactive
   $vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
   $subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
   $gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 `
   -Subnet $subnet1 -PublicIpAddress $gwpip1
   ```
8. Cree la puerta de enlace para TestVNet1. En este paso, creará la puerta de enlace de red virtual para TestVNet1. Las configuraciones VNet a VNet requieren un VpnType RouteBased. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 `
   -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
   -VpnType RouteBased -GatewaySku VpnGw1
   ```

Cuando termine con los comandos, esta puerta de enlace tardará hasta 45 minutos en crearse. Si usa Azure Cloud Shell, puede reiniciar la sesión; para ello, haga clic en la esquina superior izquierda del terminal de Cloud Shell y configure TestVNet4. No es necesario esperar hasta que se complete la puerta de enlace de TestVNet1.

### <a name="step-3---create-and-configure-testvnet4"></a>Paso 3: Creación y configuración de TestVNet4

Una vez que haya configurado TestVNet1, cree TestVNet4. Siga los pasos a continuación y reemplace los valores por los suyos propios cuando sea necesario.

1. Conéctese y declare las variables. Asegúrese de reemplazar los valores por los que desea usar para su configuración.

   ```azurepowershell-interactive
   $RG4 = "TestRG4"
   $Location4 = "West US"
   $VnetName4 = "TestVNet4"
   $FESubName4 = "FrontEnd"
   $BESubName4 = "Backend"
   $VnetPrefix41 = "10.41.0.0/16"
   $VnetPrefix42 = "10.42.0.0/16"
   $FESubPrefix4 = "10.41.0.0/24"
   $BESubPrefix4 = "10.42.0.0/24"
   $GWSubPrefix4 = "10.42.255.0/27"
   $GWName4 = "VNet4GW"
   $GWIPName4 = "VNet4GWIP"
   $GWIPconfName4 = "gwipconf4"
   $Connection41 = "VNet4toVNet1"
   ```
2. Cree un grupo de recursos.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG4 -Location $Location4
   ```
3. Cree las configuraciones de subred para TestVNet4.

   ```azurepowershell-interactive
   $fesub4 = New-AzVirtualNetworkSubnetConfig -Name $FESubName4 -AddressPrefix $FESubPrefix4
   $besub4 = New-AzVirtualNetworkSubnetConfig -Name $BESubName4 -AddressPrefix $BESubPrefix4
   $gwsub4 = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix $GWSubPrefix4
   ```
4. Cree TestVNet4.

   ```azurepowershell-interactive
   New-AzVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4 `
   -Location $Location4 -AddressPrefix $VnetPrefix41,$VnetPrefix42 -Subnet $fesub4,$besub4,$gwsub4
   ```
5. Pida una dirección IP pública.

   ```azurepowershell-interactive
   $gwpip4 = New-AzPublicIpAddress -Name $GWIPName4 -ResourceGroupName $RG4 `
   -Location $Location4 -AllocationMethod Dynamic
   ```
6. Establezca la configuración de la puerta de enlace.

   ```azurepowershell-interactive
   $vnet4 = Get-AzVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4
   $subnet4 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet4
   $gwipconf4 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName4 -Subnet $subnet4 -PublicIpAddress $gwpip4
   ```
7. Creación de la puerta de enlace de TestVNet4. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4 `
   -Location $Location4 -IpConfigurations $gwipconf4 -GatewayType Vpn `
   -VpnType RouteBased -GatewaySku VpnGw1
   ```

### <a name="step-4---create-the-connections"></a>Paso 4: Creación de las conexiones

Espere hasta que se hayan completado ambas puertas de enlace. Reinicie la sesión de Azure Cloud Shell, y copie y pegue las variables del principio del paso 2 y el paso 3 en la consola para volver a declarar los valores.

1. Obtenga ambas puertas de enlace de red virtual.

   ```azurepowershell-interactive
   $vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
   $vnet4gw = Get-AzVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4
   ```
2. Cree la conexión de TestVNet1 a TestVNet4. En este paso, creará la conexión de TestVNet1 a TestVNet4. Verá una clave compartida a la que se hace referencia en los ejemplos. Puede utilizar sus propios valores para la clave compartida. Lo importante es que la clave compartida coincida en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGatewayConnection -Name $Connection14 -ResourceGroupName $RG1 `
   -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet4gw -Location $Location1 `
   -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
   ```
3. Cree la conexión de TestVNet4 a TestVNet1. Este paso es similar al anterior, salvo en que se crea la conexión de TestVNet4 a TestVNet1. Asegúrese de que coincidan las claves compartidas. Después de unos minutos, se habrá establecido la conexión.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGatewayConnection -Name $Connection41 -ResourceGroupName $RG4 `
   -VirtualNetworkGateway1 $vnet4gw -VirtualNetworkGateway2 $vnet1gw -Location $Location4 `
   -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
   ```
4. Compruebe la conexión. Consulte la sección [Comprobación de la conexión](#verify).

## <a name="how-to-connect-vnets-that-are-in-different-subscriptions"></a><a name="difsub"></a>Conexión de redes virtuales que están en suscripciones diferentes

En este escenario, se conectan TestVNet1 y TestVNet5. TestVNet1 y TestVNet5 residen en suscripciones diferentes. Las suscripciones no necesitan estar asociadas con el mismo inquilino de Active Directory.

La diferencia entre estos pasos y el conjunto anterior es que parte de los pasos de configuración se deben realizar en una sesión de PowerShell distinta en el contexto de la segunda suscripción. Especialmente cuando las dos suscripciones pertenecen a distintas organizaciones.

Debido al cambio de contexto de la suscripción en este ejercicio, al llegar al paso 8 le resultará más fácil usar PowerShell localmente en el equipo, en lugar de utilizar Azure Cloud Shell.

### <a name="step-5---create-and-configure-testvnet1"></a>Paso 5: Creación y configuración de TestVNet1

Tiene que completar el [paso 1](#Step1) y el [paso 2](#Step2) de la sección anterior para crear y configurar TestVNet1 y VPN Gateway para TestVNet1. Para esta configuración, no se necesita crear TestVNet4 de la sección anterior, aunque, si la creó, no entrará en conflicto con estos pasos. Cuando haya completado el paso 1 y el 2, continúe con el paso 6 para crear TestVNet5.

### <a name="step-6---verify-the-ip-address-ranges"></a>Paso 6: Comprobación de los intervalos de direcciones IP

Es importante asegurarse de que el espacio de direcciones IP de la red virtual nueva, TestVNet5, no se solape con ninguno de los intervalos de red virtual o de puerta de enlace de red local. En este ejemplo, las redes virtuales pueden pertenecer a distintas organizaciones. En este ejercicio, use los siguientes valores para TestVNet5:

**Valores para TestVNet5:**

* Nombre de la red virtual: TestVNet5
* Grupo de recursos: TestRG5
* Ubicación: Japón Oriental
* TestVNet5: 10.51.0.0/16 y 10.52.0.0/16
* front-end: 10.51.0.0/24
* BackEnd: 10.52.0.0/24
* GatewaySubnet: 10.52.255.0.0/27
* GatewayName: VNet5GW
* Public IP: VNet5GWIP
* VPNType: RouteBased
* Conexión: VNet5toVNet1
* ConnectionType: VNet2VNet

### <a name="step-7---create-and-configure-testvnet5"></a>Paso 7: Creación y configuración de TestVNet5

Este paso debe realizarse en el contexto de la nueva suscripción. Es posible que esta parte la realice el administrador de otra organización que posea la suscripción.

1. Declare las variables. Asegúrese de reemplazar los valores por los que desea usar para su configuración.

   ```azurepowershell-interactive
   $Sub5 = "Replace_With_the_New_Subscription_Name"
   $RG5 = "TestRG5"
   $Location5 = "Japan East"
   $VnetName5 = "TestVNet5"
   $FESubName5 = "FrontEnd"
   $BESubName5 = "Backend"
   $GWSubName5 = "GatewaySubnet"
   $VnetPrefix51 = "10.51.0.0/16"
   $VnetPrefix52 = "10.52.0.0/16"
   $FESubPrefix5 = "10.51.0.0/24"
   $BESubPrefix5 = "10.52.0.0/24"
   $GWSubPrefix5 = "10.52.255.0/27"
   $GWName5 = "VNet5GW"
   $GWIPName5 = "VNet5GWIP"
   $GWIPconfName5 = "gwipconf5"
   $Connection51 = "VNet5toVNet1"
   ```
2. Conéctese con la suscripción 5. Abre la consola de PowerShell y conéctate a tu cuenta. Use el siguiente ejemplo para ayudarle a conectarse:

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

   Compruebe las suscripciones para la cuenta.

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

   Especifique la suscripción que desea usar.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName $Sub5
   ```
3. Cree un nuevo grupo de recursos.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG5 -Location $Location5
   ```
4. Cree las configuraciones de subred para TestVNet5.

   ```azurepowershell-interactive
   $fesub5 = New-AzVirtualNetworkSubnetConfig -Name $FESubName5 -AddressPrefix $FESubPrefix5
   $besub5 = New-AzVirtualNetworkSubnetConfig -Name $BESubName5 -AddressPrefix $BESubPrefix5
   $gwsub5 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName5 -AddressPrefix $GWSubPrefix5
   ```
5. Cree TestVNet5.

   ```azurepowershell-interactive
   New-AzVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5 -Location $Location5 `
   -AddressPrefix $VnetPrefix51,$VnetPrefix52 -Subnet $fesub5,$besub5,$gwsub5
   ```
6. Pida una dirección IP pública.

   ```azurepowershell-interactive
   $gwpip5 = New-AzPublicIpAddress -Name $GWIPName5 -ResourceGroupName $RG5 `
   -Location $Location5 -AllocationMethod Dynamic
   ```
7. Establezca la configuración de la puerta de enlace.

   ```azurepowershell-interactive
   $vnet5 = Get-AzVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5
   $subnet5  = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet5
   $gwipconf5 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName5 -Subnet $subnet5 -PublicIpAddress $gwpip5
   ```
8. Cree la puerta de enlace de TestVNet5.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5 -Location $Location5 `
   -IpConfigurations $gwipconf5 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1
   ```

### <a name="step-8---create-the-connections"></a>Paso 8: Creación de las conexiones

En este ejemplo, como las puertas de enlace están en suscripciones diferentes, hemos dividido el paso en dos sesiones de PowerShell marcadas como [Suscripción 1] y [Suscripción 5].

1. **[Suscripción 1]** Obtenga la puerta de enlace de red virtual para la suscripción 1. Inicie sesión y conéctese a la Suscripción 1 antes de ejecutar el siguiente ejemplo:

   ```azurepowershell-interactive
   $vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
   ```

   Copie la salida de los siguientes elementos y envíesela al administrador de la suscripción 5 por correo electrónico u otro método.

   ```azurepowershell-interactive
   $vnet1gw.Name
   $vnet1gw.Id
   ```

   Estos dos elementos tendrán valores similares a la salida de ejemplo siguiente:

   ```
   PS D:\> $vnet1gw.Name
   VNet1GW
   PS D:\> $vnet1gw.Id
   /subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroupsTestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
   ```
2. **[Suscripción 5]** Obtenga la puerta de enlace de red virtual para la suscripción 5. Inicie sesión y conéctese a la Suscripción 5 antes de ejecutar el siguiente ejemplo:

   ```azurepowershell-interactive
   $vnet5gw = Get-AzVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5
   ```

   Copie la salida de los siguientes elementos y envíesela al administrador de la suscripción 1 por correo electrónico u otro método.

   ```azurepowershell-interactive
   $vnet5gw.Name
   $vnet5gw.Id
   ```

   Estos dos elementos tendrán valores similares a la salida de ejemplo siguiente:

   ```
   PS C:\> $vnet5gw.Name
   VNet5GW
   PS C:\> $vnet5gw.Id
   /subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
   ```
3. **[Suscripción 1]** Cree la conexión entre TestVNet1 y TestVNet5. En este paso, creará la conexión de TestVNet1 a TestVNet5. La diferencia en este caso es que no se puede obtener $vnet5gw directamente porque está en una suscripción diferente. Debe crear un nuevo objeto de PowerShell con los valores que se le hayan proporcionado para la suscripción 1 en los pasos anteriores. Use el ejemplo siguiente. Reemplace el nombre (Name), el identificador (Id) y la clave compartida por sus propios valores. Lo importante es que la clave compartida coincida en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión.

   Conéctese a Suscripción 1 antes de ejecutar el ejemplo siguiente:

   ```azurepowershell-interactive
   $vnet5gw = New-Object -TypeName Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
   $vnet5gw.Name = "VNet5GW"
   $vnet5gw.Id   = "/subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW"
   $Connection15 = "VNet1toVNet5"
   New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet5gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
   ```
4. **[Suscripción 5]** Cree la conexión entre TestVNet5 y TestVNet1. Este paso es similar al anterior, salvo en que se crea la conexión de TestVNet5 a TestVNet1. Aquí también se aplica el mismo proceso de creación de un objeto de PowerShell basándose en los valores obtenidos de la suscripción 1. En este paso, asegúrese de que las claves compartidas coincidan.

   Conéctese a Suscripción 5 antes de ejecutar el ejemplo siguiente:

   ```azurepowershell-interactive
   $vnet1gw = New-Object -TypeName Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
   $vnet1gw.Name = "VNet1GW"
   $vnet1gw.Id = "/subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW "
   $Connection51 = "VNet5toVNet1"
   New-AzVirtualNetworkGatewayConnection -Name $Connection51 -ResourceGroupName $RG5 -VirtualNetworkGateway1 $vnet5gw -VirtualNetworkGateway2 $vnet1gw -Location $Location5 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
   ```

## <a name="how-to-verify-a-connection"></a><a name="verify"></a>Comprobación de una conexión

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections powershell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>P+F sobre conexiones de red virtual a red virtual

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte la [documentación sobre máquinas virtuales](https://docs.microsoft.com/azure/) para más información.
* Para más información acerca de BGP, consulte [Información general de BGP](vpn-gateway-bgp-overview.md) y [Configuración de BGP](vpn-gateway-bgp-resource-manager-ps.md).
