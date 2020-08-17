---
title: 'Conexión de una red virtual a una red virtual mediante una conexión de red virtual a red virtual: Azure CLI'
description: Conecte redes virtuales entre sí mediante una conexión entre redes virtuales y la CLI de Azure.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/14/2018
ms.author: cherylmc
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6cb1c8ad41ccd5f16c4a68b0b8ef4ace768bde8b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495648"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-azure-cli"></a>Configuración de una conexión de puerta de enlace de VPN de red virtual a red virtual mediante la CLI de Azure

Este artículo le ayuda a conectarse a redes virtuales mediante el tipo de conexión de red virtual a red virtual. Las redes virtuales pueden estar en la misma región o en distintas, así como pertenecer a una única suscripción o a varias. Al conectar redes virtuales de distintas suscripciones, estas no necesitan estar asociadas con el mismo inquilino de Active Directory.

Los pasos descritos en este artículo se aplican al modelo de implementación de Resource Manager y usan la CLI de Azure. También se puede crear esta configuración con una herramienta o modelo de implementación distintos, mediante la selección de una opción diferente en la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [CLI de Azure](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Portal de Azure clásico](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Conexión de diferentes modelos de implementación - Azure Portal](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Conexión de diferentes modelos de implementación - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about-connecting-vnets"></a><a name="about"></a>Acerca de la conexión de redes virtuales

Hay varias formas de conectar redes virtuales. Las siguientes secciones describen distintas formas de conectar redes virtuales.

### <a name="vnet-to-vnet"></a>De red virtual a red virtual

La configuración de una conexión entre redes virtuales es una buena manera de conectar redes virtuales fácilmente. La conexión de una red virtual a otra mediante el tipo de conexión entre redes virtuales es parecida a la creación de una conexión IPsec de sitio a sitio en una ubicación local. Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro mediante IPsec/IKE y los dos funcionan de la misma forma en lo relativo a la comunicación. La diferencia entre ambos tipos de conexión radica en la manera en que se configura la puerta de enlace de red local. Al crear una conexión entre redes virtuales, no se ve el espacio de direcciones de la puerta de enlace de red local. Se crea y rellena automáticamente. Si actualiza el espacio de direcciones de una de las redes virtuales, la otra sabe automáticamente cómo realizar el enrutamiento al espacio de direcciones actualizado. La creación de una conexión entre redes virtuales suele ser más rápida y sencilla que la creación de una conexión de sitio a sitio entre redes virtuales.

### <a name="connecting-vnets-using-site-to-site-ipsec-steps"></a>Conexión de redes virtuales mediante los pasos de sitio a sitio (IPsec)

Si puntualmente trabaja con una configuración de red complicada, puede que prefiera conectar sus redes virtuales mediante los pasos que se indican en [Creación de una red virtual con una conexión VPN de sitio a sitio mediante PowerShell](vpn-gateway-howto-site-to-site-resource-manager-cli.md), en lugar de los pasos entre redes virtuales. Cuando se usan las instrucciones para la conexión de sitio a sitio, las puertas de enlace de red locales se crean y se configuran manualmente. La puerta de enlace de red local de cada red virtual trata a la otra red virtual como un sitio local. De esta forma, puede especificar más espacio de direcciones para la puerta de enlace de red local a fin de enrutar el tráfico. Si el espacio de direcciones de una red virtual cambia, es preciso que actualice manualmente la puerta de enlace de red local correspondiente para reflejar dicho cambio. No se actualiza automáticamente.

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

En este artículo, verá dos conjuntos de pasos diferentes de la conexión entre redes virtuales. Un conjunto de pasos para las [redes virtuales que residen en la misma suscripción](#samesub) y otro para las [redes virtuales que residen en suscripciones diferentes](#difsub). 

Para este ejercicio, puede combinar las configuraciones, o bien elegir con la que desea trabajar. Todas las configuraciones utilizan el tipo de conexión entre redes virtuales. Flujos de tráfico de red entre las redes virtuales que están directamente conectados entre sí. En este ejercicio, el tráfico de TestVNet4 no se enruta a TestVNet5.

* [Redes virtuales que residen en la misma suscripción:](#samesub) En los pasos para esta configuración se utilizan TestVNet1 y TestVNet4.

  ![diagrama de v2v](./media/vpn-gateway-howto-vnet-vnet-cli/v2vrmps.png)

* [Redes virtuales que residen en distintas suscripciones:](#difsub) En los pasos para esta configuración se utilizan TestVNet1 y TestVNet5.

  ![diagrama de v2v](./media/vpn-gateway-howto-vnet-vnet-cli/v2vdiffsub.png)


## <a name="connect-vnets-that-are-in-the-same-subscription"></a><a name="samesub"></a>Conexión de redes virtuales que están en la misma suscripción

### <a name="before-you-begin"></a>Antes de empezar

Antes de empezar, instale la versión más reciente de los comandos de la CLI (2.0 o posteriores). Para obtener más información sobre la instalación de los comandos de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="plan-your-ip-address-ranges"></a><a name="Plan"></a>Planeamiento de los intervalos de direcciones IP

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

### <a name="step-1---connect-to-your-subscription"></a><a name="Connect"></a>Paso 1: Conexión a la suscripción

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-numbers-include.md)]

### <a name="step-2---create-and-configure-testvnet1"></a><a name="TestVNet1"></a>Paso 2: Creación y configuración de TestVNet1

1. Cree un grupo de recursos.

   ```azurecli
   az group create -n TestRG1  -l eastus
   ```
2. Cree TestVNet1 y las subredes para TestVNet1. En este ejemplo se crea una red virtual denominada TestVNet1 y una subred llamada FrontEnd.

   ```azurecli
   az network vnet create -n TestVNet1 -g TestRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24
   ```
3. Cree otro espacio de direcciones para la subred de back-end. Tenga en cuenta que, en este paso, se especifican tanto el espacio de direcciones que se creó antes como el adicional que se va a agregar. Esto se debe a que el comando [az network vnet update](https://docs.microsoft.com/cli/azure/network/vnet) sobrescribe la configuración anterior. Asegúrese de especificar todos los prefijos de direcciones cuando use este comando.

   ```azurecli
   az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestRG1
   ```
4. Cree la subred de back-end.
  
   ```azurecli
   az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestRG1 --address-prefix 10.12.0.0/24 
   ```
5. Cree la subred de la puerta de enlace. Asegúrese de que la subred de puerta de enlace se llame "GatewaySubnet". Este nombre es obligatorio. En este ejemplo, la subred de la puerta de enlace está usando un /27. Aunque es posible crear una subred de puerta de enlace tan pequeña como /29, se recomienda que cree una subred mayor que incluya más direcciones seleccionando al menos /28 o /27. Esto permitirá suficientes direcciones para dar cabida a posibles configuraciones adicionales que desee en el futuro.

   ```azurecli 
   az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestRG1 --address-prefix 10.12.255.0/27
   ```
6. Solicite que se asigne una dirección IP pública a la puerta de enlace que creará para la red virtual. Observe que AllocationMethod es dinámico. No puede especificar la dirección IP que desea usar. Se asigna dinámicamente a la puerta de enlace.

   ```azurecli
   az network public-ip create -n VNet1GWIP -g TestRG1 --allocation-method Dynamic
   ```
7. Cree la puerta de enlace de red virtual para TestVNet1. Las configuraciones VNet a VNet requieren un VpnType RouteBased. Si este comando se ejecuta con el parámetro '--no-wait', no se verán los comentarios o resultados. El parámetro "--no-wait" permite que la puerta de enlace se cree en segundo plano. No significa que se termine de crear la puerta de enlace de VPN de inmediato. Se suelen tardar 45 minutos o más en crear una puerta de enlace, según la SKU de puerta de enlace que se use.

   ```azurecli
   az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address VNet1GWIP -g TestRG1 --vnet TestVNet1 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-3---create-and-configure-testvnet4"></a><a name="TestVNet4"></a>Paso 3: Creación y configuración de TestVNet4

1. Cree un grupo de recursos.

   ```azurecli
   az group create -n TestRG4  -l westus
   ```
2. Cree TestVNet4.

   ```azurecli
   az network vnet create -n TestVNet4 -g TestRG4 --address-prefix 10.41.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.41.0.0/24
   ```

3. Cree subredes adicionales para TestVNet4.

   ```azurecli
   az network vnet update -n TestVNet4 --address-prefixes 10.41.0.0/16 10.42.0.0/16 -g TestRG4 
   az network vnet subnet create --vnet-name TestVNet4 -n BackEnd -g TestRG4 --address-prefix 10.42.0.0/24 
   ```
4. Cree la subred de la puerta de enlace.

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet4 -n GatewaySubnet -g TestRG4 --address-prefix 10.42.255.0/27
   ```
5. Solicite una dirección IP pública.

   ```azurecli
   az network public-ip create -n VNet4GWIP -g TestRG4 --allocation-method Dynamic
   ```
6. Cree la puerta de enlace de la red virtual TestVNet4.

   ```azurecli
   az network vnet-gateway create -n VNet4GW -l westus --public-ip-address VNet4GWIP -g TestRG4 --vnet TestVNet4 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-4---create-the-connections"></a><a name="createconnect"></a>Paso 4: Creación de las conexiones

Ahora tiene dos redes virtuales con puertas de enlace de VPN. El siguiente paso consiste en crear conexiones de puerta de enlace de VPN entre las puertas de enlace de red virtual. Si usó los ejemplos anteriores, las puertas de enlace de red virtual se encuentran en grupos de recursos distintos. Cuando las puertas de enlace están en grupos de recursos distintos, debe identificar y especificar los identificadores de recursos para cada puerta de enlace al establecer una conexión. Si sus redes virtuales están en el mismo grupo de recursos, puede usar el [segundo conjunto de instrucciones](#samerg) porque no es necesario especificar los identificadores de recursos.

### <a name="to-connect-vnets-that-reside-in-different-resource-groups"></a><a name="diffrg"></a>Conexión de redes virtuales que residen en grupos de recursos distintos

1. Obtenga el id. de recurso de VNet1GW de la salida del comando siguiente:

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   En la salida, busque la línea "id:". Los valores entrecomillados se necesitan para crear la conexión en la sección siguiente. Copie estos valores en un editor de texto, como el Bloc de notas, para que pueda pegarlos fácilmente al crear la conexión.

   Salida de ejemplo:

   ```
   "activeActive": false, 
   "bgpSettings": { 
    "asn": 65515, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
   }, 
   "enableBgp": false, 
   "etag": "W/\"ecb42bc5-c176-44e1-802f-b0ce2962ac04\"", 
   "gatewayDefaultSite": null, 
   "gatewayType": "Vpn", 
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW", 
   "ipConfigurations":
   ```

   Copie los valores después de **"id":** dentro de las comillas.

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. Obtenga el id. de recurso de VNet4GW y copie los valores en un editor de texto.

   ```azurecli
   az network vnet-gateway show -n VNet4GW -g TestRG4
   ```

3. Cree la conexión de TestVNet1 a TestVNet4. En este paso, creará la conexión de TestVNet1 a TestVNet4. Se hace referencia a una clave compartida en los ejemplos. Puede utilizar sus propios valores para la clave compartida. Lo importante es que la clave compartida coincida en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW 
   ```
4. Cree la conexión de TestVNet4 a TestVNet1. Este paso es similar al anterior, salvo en que se crea la conexión de TestVNet4 a TestVNet1. Asegúrese de que coincidan las claves compartidas. Se tarda unos minutos en establecer la conexión.

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG4 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW -l westus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1G
   ```
5. Compruebe las conexiones. Consulte [Comprobación de las conexiones](#verify).

### <a name="to-connect-vnets-that-reside-in-the-same-resource-group"></a><a name="samerg"></a>Para conectar redes virtuales que residen en el mismo grupo de recursos

1. Cree la conexión de TestVNet1 a TestVNet4. En este paso, creará la conexión de TestVNet1 a TestVNet4. Tenga en cuenta que los grupos de recursos son iguales en los ejemplos. También verá una clave compartida a la que se hace referencia en los ejemplos. Puede usar sus propios valores para la clave compartida, pero esta debe coincidir en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet4GW
   ```
2. Cree la conexión de TestVNet4 a TestVNet1. Este paso es similar al anterior, salvo en que se crea la conexión de TestVNet4 a TestVNet1. Asegúrese de que coincidan las claves compartidas. Se tarda unos minutos en establecer la conexión.

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG1 --vnet-gateway1 VNet4GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet1GW
   ```
3. Compruebe las conexiones. Consulte [Comprobación de las conexiones](#verify).

## <a name="connect-vnets-that-are-in-different-subscriptions"></a><a name="difsub"></a>Conexión de redes virtuales que están en suscripciones diferentes

En este escenario, se conectan TestVNet1 y TestVNet5. Las redes virtuales residen en suscripciones distintas. Las suscripciones no necesitan estar asociadas con el mismo inquilino de Active Directory. Los pasos para esta configuración permiten agregar una conexión de red virtual a red virtual adicional para poder conectar TestVNet1 a TestVNet5.

### <a name="step-5---create-and-configure-testvnet1"></a><a name="TestVNet1diff"></a>Paso 5: Creación y configuración de TestVNet1

Estas instrucciones son una continuación de los pasos descritos en las secciones anteriores. Tiene que completar el [paso 1](#Connect) y el [paso 2](#TestVNet1) para crear y configurar TestVNet1 y VPN Gateway para TestVNet1. Para esta configuración, no se necesita crear TestVNet4 de la sección anterior, aunque, si la creó, no entrará en conflicto con estos pasos. Cuando haya completado el paso 1 y el 2, continúe con el paso 6 (a continuación).

### <a name="step-6---verify-the-ip-address-ranges"></a><a name="verifyranges"></a>Paso 6: Comprobación de los intervalos de direcciones IP

Cuando cree conexiones adicionales, es importante asegurarse de que el espacio de direcciones IP de la red virtual nueva no se solape con ninguno de los demás intervalos de redes virtuales o de puertas de enlace de red local. En este ejercicio, use los siguientes valores para TestVNet5:

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

### <a name="step-7---create-and-configure-testvnet5"></a><a name="TestVNet5"></a>Paso 7: Creación y configuración de TestVNet5

Este paso debe realizarse en el contexto de la nueva suscripción, Suscripción 5. Es posible que esta parte la realice el administrador de otra organización que posea la suscripción. Para cambiar entre suscripciones, use `az account list --all` para obtener una lista de las suscripciones disponibles para su cuenta y después use `az account set --subscription <subscriptionID>` para cambiar a la suscripción que desea usar.

1. Asegúrese de que está conectados a Suscripción 5 y cree un grupo de recursos.

   ```azurecli
   az group create -n TestRG5  -l japaneast
   ```
2. Cree TestVNet5.

   ```azurecli
   az network vnet create -n TestVNet5 -g TestRG5 --address-prefix 10.51.0.0/16 -l japaneast --subnet-name FrontEnd --subnet-prefix 10.51.0.0/24
   ```

3. Agregue subredes.

   ```azurecli
   az network vnet update -n TestVNet5 --address-prefixes 10.51.0.0/16 10.52.0.0/16 -g TestRG5
   az network vnet subnet create --vnet-name TestVNet5 -n BackEnd -g TestRG5 --address-prefix 10.52.0.0/24
   ```

4. Agregue la subred de puerta de enlace.

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet5 -n GatewaySubnet -g TestRG5 --address-prefix 10.52.255.0/27
   ```

5. Pida una dirección IP pública.

   ```azurecli
   az network public-ip create -n VNet5GWIP -g TestRG5 --allocation-method Dynamic
   ```
6. Creación de la puerta de enlace de TestVNet5

   ```azurecli
   az network vnet-gateway create -n VNet5GW -l japaneast --public-ip-address VNet5GWIP -g TestRG5 --vnet TestVNet5 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-8---create-the-connections"></a><a name="connections5"></a>Paso 8: Creación de las conexiones

Este paso se divide en dos sesiones de la CLI marcadas como **[Suscripción 1]** y **[Suscripción 5]** , ya que las puertas de enlace están en suscripciones diferentes. Para cambiar entre suscripciones, use `az account list --all` para obtener una lista de las suscripciones disponibles para su cuenta y después use `az account set --subscription <subscriptionID>` para cambiar a la suscripción que desea usar.

1. **[Suscripción 1]** Inicie sesión y conéctese a Suscripción 1. Ejecute el siguiente comando para obtener el nombre y el identificador de la puerta de enlace de la salida:

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   Copie la salida para "id:". Envíe el identificador y el nombre de la puerta de enlace de red virtual (VNet1GW) al administrador de Suscripción 5 por correo electrónico u otro método.

   Salida de ejemplo:

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. **[Suscripción 5]** Inicie sesión y conéctese a Suscripción 5. Ejecute el siguiente comando para obtener el nombre y el identificador de la puerta de enlace de la salida:

   ```azurecli
   az network vnet-gateway show -n VNet5GW -g TestRG5
   ```

   Copie la salida para "id:". Envíe el identificador y el nombre de la puerta de enlace de red virtual (VNet5GW) al administrador de Suscripción 1 por correo electrónico u otro método.

3. **[Suscripción 1]** En este paso, va a crear la conexión de TestVNet1 a TestVNet5. Puede usar sus propios valores para la clave compartida, pero esta debe coincidir en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión. Asegúrese de conectarse a la suscripción 1.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet5 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
   ```

4. **[Suscripción 5]** Este paso es similar al anterior, salvo en que se crea la conexión de TestVNet5 a TestVNet1. Asegúrese de que las claves compartidas coincidan y que se conecta a Suscripción 5.

   ```azurecli
   az network vpn-connection create -n VNet5ToVNet1 -g TestRG5 --vnet-gateway1 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW -l japaneast --shared-key "eeffgg" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
   ```

## <a name="verify-the-connections"></a><a name="verify"></a>Comprobación de las conexiones
[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>P+F sobre conexiones de red virtual a red virtual
[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Para más información, consulte la [documentación sobre máquinas virtuales](https://docs.microsoft.com/azure/).
* Para más información acerca de BGP, consulte [Información general de BGP](vpn-gateway-bgp-overview.md) y [Configuración de BGP](vpn-gateway-bgp-resource-manager-ps.md).
