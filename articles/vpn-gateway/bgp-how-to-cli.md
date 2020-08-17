---
title: 'Configuración de BGP en instancias de Azure VPN Gateway: CLI'
description: Este artículo le guiará a la hora de configurar BGP con Azure VPN Gateway con Azure Resource Manager y la CLI.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/25/2018
ms.author: yushwang
ms.openlocfilehash: f3a6dd70bf57cabe8014c49e2cd3ee38c8de72b4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87064787"
---
# <a name="how-to-configure-bgp-on-an-azure-vpn-gateway-by-using-cli"></a>Configuración de BGP en Azure VPN Gateway con la CLI

Este artículo le ayudará a habilitar BGP en una conexión de VPN de sitio a sitio (S2S) entre locales y una conexión de red virtual a red virtual (es decir, una conexión entre redes virtuales) mediante el modelo de implementación de Azure Resource Manager y la CLI de Azure.

## <a name="about-bgp"></a>Información acerca de BGP

BGP es el protocolo de enrutamiento estándar usado habitualmente en Internet para intercambiar información de enrutamiento y disponibilidad entre dos o más redes. BGP permite a Azure VPN Gateway y a los dispositivos locales de VPN, denominados vecinos o pares BGP, intercambiar rutas. Las rutas informan a ambas puertas de enlace sobre la disponibilidad y accesibilidad de los prefijos para pasar por las puertas de enlace o los enrutadores pertinentes. BGP también permite el enrutamiento del tráfico entre varias redes mediante la propagación de las rutas de aprendizaje de la puerta de enlace de BGP desde un par BGP a todos los demás pares BGP.

Para obtener más información sobre las ventajas de BGP y conocer los requisitos técnicos y las consideraciones sobre el uso de BGP, consulte [Información general de BGP con Azure VPN Gateway](vpn-gateway-bgp-overview.md).

Este artículo resulta útil para las siguientes tareas:

* [Habilitar BGP para VPN Gateway](#enablebgp) (obligatorio)

  Después, puede completar cualquiera de las siguientes secciones, o ambas:

* [Establecer una conexión entre locales con BGP](#crossprembgp)
* [Establecer una conexión de red virtual a red virtual con BGP](#v2vbgp)

Cada una de estas tres secciones constituye un bloque de creación básico para habilitar BGP en la conectividad de red. Si completa las tres secciones, podrá crear la topología tal como se muestra en el diagrama siguiente:

![Topología de BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

Puede combinar estas secciones para crear una red de tránsito más compleja y de saltos múltiples que satisfaga sus necesidades.

## <a name="enable-bgp-for-your-vpn-gateway"></a><a name ="enablebgp"></a>Habilitar BGP para VPN Gateway

Esta sección es necesaria antes de realizar alguno de los pasos de las otras dos secciones de configuración. Los siguientes pasos de configuración permiten establecer los parámetros BGP de Azure VPN Gateway como se muestra en el diagrama siguiente:

![Puerta de enlace de BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>Antes de empezar

Instale la versión más reciente de los comandos de la CLI (2.0 o posteriores). Para más información acerca de la instalación de los comandos de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) y [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli).

### <a name="step-1-create-and-configure-testvnet1"></a>Paso 1: Creación y configuración de TestVNet1

#### <a name="1-connect-to-your-subscription"></a><a name="Login"></a>1. su suscripción

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

#### <a name="2-create-a-resource-group"></a>2. Crear un grupo de recursos

En el ejemplo siguiente se crea un grupo de recursos con el nombre "TestRG1" en la ubicación "eastus". Si ya tiene un grupo de recursos en la región en la que desea crear la red virtual, puede utilizar ese.

```azurecli
az group create --name TestBGPRG1 --location eastus
```

#### <a name="3-create-testvnet1"></a>3. Creación de TestVNet1

El siguiente ejemplo crea una red virtual denominada TestVNet1 y tres subredes: GatewaySubnet, FrontEnd y BackEnd. Al reemplazar valores, es importante que siempre asigne el nombre "GatewaySubnet" a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace.

El primer comando crea el espacio de direcciones de front-end y la subred de front-end. El segundo comando crea otro espacio de direcciones para la subred de back-end. Los comandos terceros y cuarto crean la subred Backend y GatewaySubnet.

```azurecli
az network vnet create -n TestVNet1 -g TestBGPRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24 
 
az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestBGPRG1 
 
az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestBGPRG1 --address-prefix 10.12.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestBGPRG1 --address-prefix 10.12.255.0/27 
```

### <a name="step-2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>Paso 2: Creación de VPN Gateway para TestVNet1 con parámetros BGP

#### <a name="1-create-the-public-ip-address"></a>1. Crear la dirección IP pública

Pida una dirección IP pública. La dirección IP pública se asignará a la instancia de VPN Gateway creada para la red virtual.

```azurecli
az network public-ip create -n GWPubIP -g TestBGPRG1 --allocation-method Dynamic 
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2. Cree la puerta de enlace de VPN con el número de AS

Cree la puerta de enlace de red virtual para TestVNet1. BGP requiere una instancia de VPN Gateway basada en rutas. También necesita el parámetro adicional `-Asn` para establecer el número de sistema autónomo (ASN) para TestVNet1. Se tardan unos 45 minutos o algo más en crear una puerta de enlace. 

Si este comando se ejecuta con el parámetro `--no-wait`, no se verán los comentarios o resultados. Este parámetro `--no-wait` permite que la puerta de enlace se cree en segundo plano. No significa que la puerta de enlace de VPN se cree de inmediato.

```azurecli
az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address GWPubIP -g TestBGPRG1 --vnet TestVNet1 --gateway-type Vpn --sku HighPerformance --vpn-type RouteBased --asn 65010 --no-wait
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3. Obtención de la dirección IP del par BGP de Azure

Después de crear la puerta de enlace, debe obtener la dirección IP del par BGP en Azure VPN Gateway. Esta dirección es necesaria para configurar VPN Gateway como par BGP para los dispositivos de VPN local.

Ejecute el comando siguiente y consulte la sección de `bgpSettings` en la parte superior de la salida:

```azurecli
az network vnet-gateway list -g TestBGPRG1 
 
  
"bgpSettings": { 
      "asn": 65010, 
      "bgpPeeringAddress": "10.12.255.30", 
      "peerWeight": 0 
    }
```

Después de crear la puerta de enlace, puede usarla para establecer una conexión entre locales o una conexión de red virtual a red virtual con BGP.

## <a name="establish-a-cross-premises-connection-with-bgp"></a><a name ="crossprembgp"></a>Establecer una conexión entre locales con BGP

Para establecer una conexión entre locales, debe crear una puerta de enlace de red local para representar el dispositivo VPN local. A continuación, conecte Azure VPN Gateway con la puerta de enlace de red local. Aunque estos pasos son similares a la creación de otras conexiones, incluyen propiedades adicionales para especificar los parámetros de configuración de BGP.

![BGP entre locales](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)


### <a name="step-1-create-and-configure-the-local-network-gateway"></a>Paso 1: Cree y configure la puerta de enlace de red local

Este ejercicio es continuación del paso de creación de la configuración mostrada en el diagrama. Asegúrese de reemplazar los valores por los que desea usar para su configuración. Cuando trabaje con puertas de enlace de red local, tenga en cuenta los siguientes aspectos:

* La puerta de enlace de red local puede estar en la misma ubicación y grupo de recursos que la puerta de enlace de VPN o en una ubicación y grupo de recurso distintos. Este ejemplo los muestra las puertas de enlace en distintos grupos de recursos en diferentes ubicaciones.
* El prefijo mínimo que debe declarar para la puerta de enlace de red local es la dirección del host de la dirección IP del par BGP en el dispositivo VPN. En este caso, es un prefijo /32 de 10.51.255.254/32.
* Como recordatorio, debe usar diferentes ASN de BGP entre las redes locales y la red virtual de Azure. Si son iguales, debe cambiar el ASN de la red virtual si los dispositivos VPN locales ya utilizan el ASN para estar al mismo nivel que otros vecinos BGP.

Antes de continuar, asegúrese de haber completado la sección [Habilitar BGP para VPN Gateway](#enablebgp) de este ejercicio y de que aún está conectado a la suscripción 1. Tenga en cuenta que, en este ejemplo, se crea un grupo de recursos. Además, observe los dos parámetros adicionales para la puerta de enlace de red local: `Asn` y `BgpPeerAddress`.

```azurecli
az group create -n TestBGPRG5 -l eastus2 
 
az network local-gateway create --gateway-ip-address 23.99.221.164 -n Site5 -g TestBGPRG5 --local-address-prefixes 10.51.255.254/32 --asn 65050 --bgp-peering-address 10.51.255.254
```

### <a name="step-2-connect-the-vnet-gateway-and-local-network-gateway"></a>Paso 2: Conecte la puerta de enlace de red virtual y la puerta de enlace de red local

En este paso, creará la conexión de TestVNet1 a Site5. Debe especificar el parámetro `--enable-bgp` para habilitar BGP para esta conexión. 

En este ejemplo, la puerta de enlace de red virtual y la puerta de enlace de red local están en distintos grupos de recursos. Cuando las puertas de enlace están en distintos grupos de recursos, debe especificar el identificador de recurso completo de las dos puertas de enlace para establecer una conexión entre las redes virtuales.

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1. Obtención del id. de recurso de VNet1GW

Use la salida del siguiente comando para obtener el id. de recurso de VNet1GW:

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

En la salida, busque la línea `"id":`. Necesita los valores entrecomillados para crear la conexión en la sección siguiente.

Salida de ejemplo:

```
{ 
  "activeActive": false, 
  "bgpSettings": { 
    "asn": 65010, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
  }, 
  "enableBgp": true, 
  "etag": "W/\"<your etag number>\"", 
  "gatewayDefaultSite": null, 
  "gatewayType": "Vpn", 
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
```

Copie estos valores después de `"id":` en un editor de texto, como el Bloc de notas, para que pueda pegarlos fácilmente al crear la conexión. 

```
"id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
```

#### <a name="2-get-the-resource-id-of-site5"></a>2. Obtención del id. de recurso de Site5

Use el siguiente comando para obtener el id. de recurso de Site5 de la salida:

```azurecli
az network local-gateway show -n Site5 -g TestBGPRG5
```

#### <a name="3-create-the-testvnet1-to-site5-connection"></a>3. Creación de la conexión de TestVNet1 a Site5

En este paso, creará la conexión de TestVNet1 a Site5. Como se explicó anteriormente, es posible tener conexiones BGP y no BGP para la misma instancia de Azure VPN Gateway. A menos que BGP esté habilitado en la propiedad de conexión, Azure no habilitará BGP para esta conexión, aunque los parámetros BGP ya estén configurados en ambas puertas de enlace. Reemplace los identificadores de suscripción por los suyos propios.

```azurecli
az network vpn-connection create -n VNet1ToSite5 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "abc123" --local-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG5/providers/Microsoft.Network/localNetworkGateways/Site5 --no-wait
```

Para este ejercicio, en el ejemplo siguiente se enumeran los parámetros que deberá especificar en la sección de configuración de BGP en el dispositivo VPN local:

```
Site5 ASN            : 65050
Site5 BGP IP         : 10.52.255.254
Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
Azure VNet ASN       : 65010
Azure VNet BGP IP    : 10.12.255.30
Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

Después de unos minutos, la conexión debería haberse establecido. Se inicia la sesión de emparejamiento de BGP después de establecer la conexión IPsec.

## <a name="establish-a-vnet-to-vnet-connection-with-bgp"></a><a name ="v2vbgp"></a>Establecer una conexión de red virtual a red virtual con BGP

En esta sección se agrega una conexión de red virtual a red virtual con BGP, tal como se muestra en el diagrama siguiente: 

![BGP para conexiones de red virtual a red virtual](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

Las siguientes instrucciones son la continuación de los pasos de las secciones previas. Debe completar la sección [Habilitar BGP para VPN Gateway](#enablebgp) para crear y configurar TestVNet1 y VPN Gateway con BGP.

### <a name="step-1-create-testvnet2-and-the-vpn-gateway"></a>Paso 1: Cree TestVNet2 y VPN Gateway

Es importante asegurarse de que el espacio de direcciones IP de la red virtual nueva, TestVNet2, no se solape con ninguno de sus intervalos de red virtual.

En este ejemplo, las redes virtuales pertenecen a la misma suscripción. Se pueden configurar conexiones de red virtual a red virtual entre distintas suscripciones. Para más información, vea [Configuración de una conexión de red virtual a red virtual](vpn-gateway-howto-vnet-vnet-cli.md). Asegúrese de agregar `-EnableBgp $True` al crear las conexiones para habilitar BGP.

#### <a name="1-create-a-new-resource-group"></a>1. Creación de un nuevo grupo de recursos

```azurecli
az group create -n TestBGPRG2 -l westus
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Cree TestVNet2 en el nuevo grupo de recursos

El primer comando crea el espacio de direcciones de front-end y la subred de front-end. El segundo comando crea otro espacio de direcciones para la subred de back-end. Los comandos terceros y cuarto crean la subred Backend y GatewaySubnet.

```azurecli
az network vnet create -n TestVNet2 -g TestBGPRG2 --address-prefix 10.21.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.21.0.0/24 
 
az network vnet update -n TestVNet2 --address-prefixes 10.21.0.0/16 10.22.0.0/16 -g TestBGPRG2 
 
az network vnet subnet create --vnet-name TestVNet2 -n BackEnd -g TestBGPRG2 --address-prefix 10.22.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet2 -n GatewaySubnet -g TestBGPRG2 --address-prefix 10.22.255.0/27
```

#### <a name="3-create-the-public-ip-address"></a>3. Crear la dirección IP pública

Pida una dirección IP pública. La dirección IP pública se asignará a la instancia de VPN Gateway creada para la red virtual.

```azurecli
az network public-ip create -n GWPubIP2 -g TestBGPRG2 --allocation-method Dynamic
```

#### <a name="4-create-the-vpn-gateway-with-the-as-number"></a>4. Cree la puerta de enlace de VPN con el número de AS

Cree la puerta de enlace de red virtual para TestVNet2. Debe reemplazar el valor predeterminado del ASN en Azure VPN Gateway. Los ASN para las redes virtuales conectadas deben ser diferentes para habilitar el enrutamiento de tránsito y de BGP.
 
```azurecli
az network vnet-gateway create -n VNet2GW -l westus --public-ip-address GWPubIP2 -g TestBGPRG2 --vnet TestVNet2 --gateway-type Vpn --sku Standard --vpn-type RouteBased --asn 65020 --no-wait
```

### <a name="step-2-connect-the-testvnet1-and-testvnet2-gateways"></a>Paso 2: Conecte las puertas de enlace de TestVNet1 y TestVNet2

En este paso, creará la conexión de TestVNet1 a Site5. Debe especificar el parámetro `--enable-bgp` para habilitar BGP para esta conexión.

En el ejemplo siguiente, la puerta de enlace de red virtual y la puerta de enlace de red local están en distintos grupos de recursos. Cuando las puertas de enlace están en distintos grupos de recursos, debe especificar el identificador de recurso completo de las dos puertas de enlace para establecer una conexión entre las redes virtuales. 

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1. Obtención del id. de recurso de VNet1GW 

Obtenga el id. de recurso de VNet1GW de la salida del comando siguiente:

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

#### <a name="2-get-the-resource-id-of-vnet2gw"></a>2. Obtención del id. de recurso de VNet2GW

Obtenga el id. de recurso de VNet2GW de la salida del comando siguiente:

```azurecli
az network vnet-gateway show -n VNet2GW -g TestBGPRG2
```

#### <a name="3-create-the-connections"></a>3. Crear las conexiones

Cree la conexión de TestVNet1 a TestVNet2 y viceversa. Reemplace los identificadores de suscripción por los suyos propios.

```azurecli
az network vpn-connection create -n VNet1ToVNet2 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW
```

```azurecli
az network vpn-connection create -n VNet2ToVNet1 -g TestBGPRG2 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW --enable-bgp -l westus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
```

> [!IMPORTANT]
> Habilite BGP para *ambas* conexiones.
> 
> 

Después de completar estos pasos, se establecerá la conexión en cuestión de minutos. La sesión de emparejamiento BGP funcionará después de completar la conexión de red virtual a red virtual.

## <a name="next-steps"></a>Pasos siguientes

Después de completar la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Creación de una máquina virtual que ejecuta Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md) para ver los pasos.
