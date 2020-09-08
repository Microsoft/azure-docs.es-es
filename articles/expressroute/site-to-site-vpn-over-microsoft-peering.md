---
title: 'Azure ExpressRoute: Configuración de VPN de sitio a sitio en el emparejamiento de Microsoft'
description: Configure la conectividad de IPsec/IKE a Azure a través de un circuito de emparejamiento de Microsoft de ExpressRoute mediante una puerta de enlace VPN de sitio a sitio.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 366f27a0e2a22e9aa10dda20e105bf644255bdd4
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393146"
---
# <a name="configure-a-site-to-site-vpn-over-expressroute-microsoft-peering"></a>Configuración de una VPN de sitio a sitio a través del emparejamiento de Microsoft de ExpressRoute

Este artículo le ayudará a configurar una conexión cifrada segura entre su red local y sus redes virtuales (VNET) de Azure a través de una conexión privada ExpressRoute. Puede usar el emparejamiento de Microsoft para establecer un túnel VPN de IPsec/IKE de sitio a sitio entre las redes locales seleccionadas y las redes virtuales de Azure. La configuración de un túnel seguro a través de ExpressRoute permite el intercambio de datos con confidencialidad, antireproducción, autenticidad e integridad.

>[!NOTE]
>Al configurar una VPN de sitio a sitio a través del emparejamiento de Microsoft, se le aplican cargos por VPN Gateway y la salida de VPN. Para obtener más información, vea [Precios de VPN Gateway](https://azure.microsoft.com/pricing/details/vpn-gateway).
>
>

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="architecture"></a><a name="architecture"></a>Arquitectura


  ![información general sobre la conectividad](./media/site-to-site-vpn-over-microsoft-peering/IPsecER_Overview.png)


Para obtener altos niveles de disponibilidad y redundancia, puede configurar varios túneles a través de los dos pares de MSEE-PE de un circuito ExpressRoute y habilitar el equilibrio de carga entre los túneles.

  ![opciones de alta disponibilidad](./media/site-to-site-vpn-over-microsoft-peering/HighAvailability.png)

Los túneles VPN a través del emparejamiento de Microsoft se pueden finalizar mediante VPN Gateway, o bien mediante una aplicación virtual de red (NVA) disponible a través de Azure Marketplace. Puede intercambiar rutas de forma estática o dinámica a través de los túneles cifrados sin exponer el intercambio de ruta al emparejamiento de Microsoft subyacente. En los ejemplos de este artículo, se usa BGP (diferente de la sesión BGP utilizada para crear el emparejamiento de Microsoft) para intercambiar prefijos de forma dinámica a través de los túneles cifrados.

>[!IMPORTANT]
>Para el lado local, el emparejamiento de Microsoft suele finalizarse en la red perimetral y el emparejamiento privado, en la zona de la red principal. Las dos zonas deben separarse mediante firewalls. Si configura el emparejamiento de Microsoft exclusivamente para habilitar la tunelización segura a través de ExpressRoute, recuerde realizar el filtrado solo a través de las direcciones IP públicas de interés que se anuncian a través del emparejamiento de Microsoft.
>
>

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

1. Configure el emparejamiento de Microsoft para su circuito ExpressRoute.
2. Anuncie los prefijos públicos regionales de Azure seleccionados en su red local a través del emparejamiento de Microsoft.
3. Configure una instancia de VPN Gateway y establezca los túneles IPsec.
4. Configure el dispositivo VPN local.
5. Cree la conexión IPsec/IKE de sitio a sitio.
6. (Opcional) Configure los firewalls o el filtrado en el dispositivo VPN local.
7. Pruebe y valide la comunicación IPsec a través del circuito ExpressRoute.

## <a name="1-configure-microsoft-peering"></a><a name="peering"></a>1. Configuración del emparejamiento de Microsoft

Para configurar una conexión VPN de sitio a sitio a través de ExpressRoute, debe aprovechar el emparejamiento de Microsoft de ExpressRoute.

* Para configurar un nuevo circuito ExpressRoute, comience con el artículo [Requisitos previos de ExpressRoute](expressroute-prerequisites.md) y, a continuación, consulte [Creación y modificación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md).

* Si ya tiene un circuito ExpressRoute, pero no tiene configurado el emparejamiento de Microsoft, consulte el artículo [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-arm.md#msft) para configurarlo.

Una vez configurado el circuito y el emparejamiento de Microsoft, se puede ver fácilmente a través de la página **Información general** de Azure Portal.

![circuito](./media/site-to-site-vpn-over-microsoft-peering/ExpressRouteCkt.png)

## <a name="2-configure-route-filters"></a><a name="routefilter"></a>2. Configuración de filtros de ruta

Un filtro de ruta permite identificar los servicios que desea consumir mediante el emparejamiento de Microsoft del circuito ExpressRoute. Se trata básicamente de una lista de todos los valores permitidos de la comunidad de BGP. 

![filtro de ruta](./media/site-to-site-vpn-over-microsoft-peering/route-filter.png)

En este ejemplo, la implementación se realiza solo en la región *Oeste de EE. UU. 2 de Azure*. Se agrega una regla de filtro de ruta para permitir solo el anuncio de prefijos regionales de Oeste de EE. UU. 2 de Azure, cuyo valor de comunidad de BGP es *12076:51026*. Para especificar los prefijos regionales que quiere permitir seleccione **Administrar regla**.

En el filtro de ruta, también debe elegir los circuitos ExpressRoute a los que se aplicará el filtro de ruta. Para elegir los circuitos ExpressRoute puede seleccionar **Agregar circuito**. En la ilustración anterior, el filtro de ruta está asociado al circuito ExpressRoute de ejemplo.

### <a name="21-configure-the-route-filter"></a><a name="configfilter"></a>2.1 Configuración del filtro de ruta

Configure un filtro de ruta. Para consultar los pasos, vea [Configuración de filtros de ruta para el emparejamiento de Microsoft](how-to-routefilter-portal.md).

### <a name="22-verify-bgp-routes"></a><a name="verifybgp"></a>2.2 Comprobación de las rutas BGP

Después de crear correctamente el emparejamiento de Microsoft a través del circuito ExpressRoute y de asociar un filtro de ruta al circuito, puede comprobar las rutas BGP procedentes de los MSEE en los dispositivos PE que se emparejan con los MSEE. El comando de comprobación varía según el sistema operativo de los dispositivos PE.

#### <a name="cisco-examples"></a>Ejemplos de Cisco

En este ejemplo se utiliza un comando Cisco IOS-XE. En el ejemplo, se utiliza una instancia de reenvío y enrutamiento virtual (VRF) para aislar el tráfico de emparejamiento.

```
show ip bgp vpnv4 vrf 10 summary
```

La siguiente salida parcial muestra que se han recibido 68 prefijos del vecino \*.243.229.34 con ASN 12076 (MSEE):

```
...

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
X.243.229.34    4        12076   17671   17650    25228    0    0 1w4d           68
```

Para ver la lista de prefijos procedentes del vecino, use el ejemplo siguiente:

```
sh ip bgp vpnv4 vrf 10 neighbors X.243.229.34 received-routes
```

Para confirmar que recibe el conjunto correcto de prefijos, puede realizar una comparación. La salida de comando de Azure PowerShell siguiente enumera los prefijos anunciados a través del emparejamiento de Microsoft para cada uno de los servicios y para cada una de las regiones de Azure:

```azurepowershell-interactive
Get-AzBgpServiceCommunity
```

## <a name="3-configure-the-vpn-gateway-and-ipsec-tunnels"></a><a name="vpngateway"></a>3. Configuración de VPN Gateway y de los túneles IPsec

En esta sección, se crean túneles VPN IPsec entre Azure VPN Gateway y el dispositivo VPN local. En los ejemplos se utilizan dispositivos VPN Cisco Cloud Service Router (CSR1000).

El siguiente diagrama muestra los túneles VPN IPsec establecidos entre el dispositivo VPN local 1 y el par de instancias de Azure VPN Gateway. Los dos túneles VPN IPsec establecidos entre el dispositivo VPN local 2 y el par de instancias de Azure VPN Gateway no se muestra en el diagrama y los detalles de configuración no se enumeran. Sin embargo, tener túneles VPN adicionales mejora la alta disponibilidad.

  ![Túneles VPN](./media/site-to-site-vpn-over-microsoft-peering/EstablishTunnels.png)

A través del par de túneles IPsec, una sesión eBGP está establecida para intercambiar rutas de redes privadas. El siguiente diagrama muestra la sesión BGP establecida a través del par de túneles IPsec:

  ![Sesiones BGP a través de un par de túneles](./media/site-to-site-vpn-over-microsoft-peering/TunnelBGP.png)

El diagrama siguiente muestra la información general resumida de la red de ejemplo:

  ![red de ejemplo](./media/site-to-site-vpn-over-microsoft-peering/OverviewRef.png)

### <a name="about-the-azure-resource-manager-template-examples"></a>Acerca de los ejemplos de plantillas de Azure Resource Manager

En los ejemplos, las terminaciones de túneles IPsec y VPN Gateway se configuran mediante una plantilla de Azure Resource Manager. Si no está familiarizado con el uso de plantillas de Resource Manager o con los conceptos básicos de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md). La plantilla de esta sección crea un entorno de Azure virgen (VNet). Sin embargo, si tiene una red virtual existente, puede hacer referencia a esta en la plantilla. Si no está familiarizado con las configuraciones IPsec/IKE de sitio a sitio de VPN Gateway, consulte el tema sobre la [creación de una conexión de sitio a sitio](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md).

>[!NOTE]
>No es necesario usar plantillas de Azure Resource Manager para crear esta configuración. Puede crear esta configuración mediante Azure Portal o PowerShell.
>
>

### <a name="31-declare-the-variables"></a><a name="variables3"></a>3.1 Declaración de las variables

En este ejemplo, las declaraciones de variables corresponden a la red de ejemplo. Al declarar variables, modifique esta sección para reflejar su entorno.

* La variable **localAddressPrefix** es una matriz de direcciones IP locales para finalizar los túneles IPsec.
* La clase **gatewaySku** determina el rendimiento VPN. Para más información sobre las clases gatewaySku y vpnType, consulte [Acerca de la configuración de VPN Gateway](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsku). Para conocer los precios, consulte [Precios de VPN Gateway](https://azure.microsoft.com/pricing/details/vpn-gateway).
* Establezca la clase **vpnType** en **RouteBased**.

```json
"variables": {
  "virtualNetworkName": "SecureVNet",       // Name of the Azure VNet
  "azureVNetAddressPrefix": "10.2.0.0/24",  // Address space assigned to the VNet
  "subnetName": "Tenant",                   // subnet name in which tenants exists
  "subnetPrefix": "10.2.0.0/25",            // address space of the tenant subnet
  "gatewaySubnetPrefix": "10.2.0.224/27",   // address space of the gateway subnet
  "localGatewayName": "localGW1",           // name of remote gateway (on-premises)
  "localGatewayIpAddress": "X.243.229.110", // public IP address of the on-premises VPN device
  "localAddressPrefix": [
    "172.16.0.1/32",                        // termination of IPsec tunnel-1 on-premises 
    "172.16.0.2/32"                         // termination of IPsec tunnel-2 on-premises 
  ],
  "gatewayPublicIPName1": "vpnGwVIP1",    // Public address name of the first VPN gateway instance
  "gatewayPublicIPName2": "vpnGwVIP2",    // Public address name of the second VPN gateway instance 
  "gatewayName": "vpnGw",                 // Name of the Azure VPN gateway
  "gatewaySku": "VpnGw1",                 // Azure VPN gateway SKU
  "vpnType": "RouteBased",                // type of VPN gateway
  "sharedKey": "string",                  // shared secret needs to match with on-premises configuration
  "asnVpnGateway": 65000,                 // BGP Autonomous System number assigned to the VPN Gateway 
  "asnRemote": 65010,                     // BGP Autonmous Syste number assigned to the on-premises device
  "bgpPeeringAddress": "172.16.0.3",      // IP address of the remote BGP peer on-premises
  "connectionName": "vpn2local1",
  "vnetID": "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
  "gatewaySubnetRef": "[concat(variables('vnetID'),'/subnets/','GatewaySubnet')]",
  "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables('subnetName'))]",
  "api-version": "2017-06-01"
},
```

### <a name="32-create-virtual-network-vnet"></a><a name="vnet"></a>3.2 Creación de una red virtual (VNet)

Si está asociando una red virtual existente con túneles VPN, puede omitir este paso.

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/virtualNetworks",
  "name": "[variables('virtualNetworkName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "[variables('azureVNetAddressPrefix')]"
      ]
    },
    "subnets": [
      {
        "name": "[variables('subnetName')]",
        "properties": {
          "addressPrefix": "[variables('subnetPrefix')]"
        }
      },
      {
        "name": "GatewaySubnet",
        "properties": {
          "addressPrefix": "[variables('gatewaySubnetPrefix')]"
        }
      }
    ]
  },
  "comments": "Create a Virtual Network with Subnet1 and Gatewaysubnet"
},
```

### <a name="33-assign-public-ip-addresses-to-vpn-gateway-instances"></a><a name="ip"></a>3.3 Asignación de direcciones IP públicas a instancias de VPN Gateway
 
Asigne una dirección IP pública para cada instancia de VPN Gateway.

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/publicIPAddresses",
    "name": "[variables('gatewayPublicIPName1')]",
    "location": "[resourceGroup().location]",
    "properties": {
      "publicIPAllocationMethod": "Dynamic"
    },
    "comments": "Public IP for the first instance of the VPN gateway"
  },
  {
    "apiVersion": "[variables('api-version')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[variables('gatewayPublicIPName2')]",
    "location": "[resourceGroup().location]",
    "properties": {
      "publicIPAllocationMethod": "Dynamic"
    },
    "comments": "Public IP for the second instance of the VPN gateway"
  },
```

### <a name="34-specify-the-on-premises-vpn-tunnel-termination-local-network-gateway"></a><a name="termination"></a>3.4 Especificación de la terminación de túnel VPN local (puerta de enlace de red local)

Los dispositivos VPN locales se conocen como **puerta de enlace de red local**. El siguiente fragmento de código JSON también especifica detalles del par BGP remoto:

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/localNetworkGateways",
  "name": "[variables('localGatewayName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "localNetworkAddressSpace": {
      "addressPrefixes": "[variables('localAddressPrefix')]"
    },
    "gatewayIpAddress": "[variables('localGatewayIpAddress')]",
    "bgpSettings": {
      "asn": "[variables('asnRemote')]",
      "bgpPeeringAddress": "[variables('bgpPeeringAddress')]",
      "peerWeight": 0
    }
  },
  "comments": "Local Network Gateway (referred to your on-premises location) with IP address of remote tunnel peering and IP address of remote BGP peer"
},
```

### <a name="35-create-the-vpn-gateway"></a><a name="creategw"></a>3.5 Creación de la instancia de VPN Gateway

En esta sección de la plantilla se configura la instancia de VPN Gateway con la configuración necesaria para una configuración activa/activa. Tenga en cuenta los siguientes requisitos:

* Cree la instancia de VPN Gateway con una clase VpnType **"RouteBased"** . Esta configuración es obligatoria si quiere habilitar el enrutamiento BGP entre VPN Gateway y la VPN local.
* Para establecer túneles VPN entre las dos instancias de VPN Gateway y un dispositivo local determinado en el modo activo/activo, el parámetro **"activeActive"** está establecido en **true** en la plantilla de Resource Manager. Para obtener más información sobre las instancias de VPN Gateway de alta disponibilidad, consulte el tema sobre la [conectividad de VPN Gateway de alta disponibilidad](../vpn-gateway/vpn-gateway-highlyavailable.md).
* Para configurar sesiones eBGP entre los túneles VPN, debe especificar dos ASN diferentes en cada lado. Es preferible especificar números ASN privados. Para obtener más información, consulte el tema de [información general de BGP y Azure VPN Gateway](../vpn-gateway/vpn-gateway-bgp-overview.md).

```json
{
"apiVersion": "[variables('api-version')]",
"type": "Microsoft.Network/virtualNetworkGateways",
"name": "[variables('gatewayName')]",
"location": "[resourceGroup().location]",
"dependsOn": [
  "[concat('Microsoft.Network/publicIPAddresses/', variables('gatewayPublicIPName1'))]",
  "[concat('Microsoft.Network/publicIPAddresses/', variables('gatewayPublicIPName2'))]",
  "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
],
"properties": {
  "ipConfigurations": [
    {
      "properties": {
        "privateIPAllocationMethod": "Dynamic",
        "subnet": {
          "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
          "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('gatewayPublicIPName1'))]"
        }
      },
      "name": "vnetGtwConfig1"
    },
    {
      "properties": {
        "privateIPAllocationMethod": "Dynamic",
        "subnet": {
          "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
          "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('gatewayPublicIPName2'))]"
        }
      },
          "name": "vnetGtwConfig2"
        }
      ],
      "sku": {
        "name": "[variables('gatewaySku')]",
        "tier": "[variables('gatewaySku')]"
      },
      "gatewayType": "Vpn",
      "vpnType": "[variables('vpnType')]",
      "enableBgp": true,
      "activeActive": true,
      "bgpSettings": {
        "asn": "[variables('asnVpnGateway')]"
      }
    },
    "comments": "VPN Gateway in active-active configuration with BGP support"
  },
```

### <a name="36-establish-the-ipsec-tunnels"></a><a name="ipsectunnel"></a>3.6 Establecimiento de los túneles IPsec

La acción final del script crea túneles IPsec entre la Azure VPN Gateway y el dispositivo VPN local.

```json
{
  "apiVersion": "[variables('api-version')]",
  "name": "[variables('connectionName')]",
  "type": "Microsoft.Network/connections",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Network/virtualNetworkGateways/', variables('gatewayName'))]",
    "[concat('Microsoft.Network/localNetworkGateways/', variables('localGatewayName'))]"
  ],
  "properties": {
    "virtualNetworkGateway1": {
      "id": "[resourceId('Microsoft.Network/virtualNetworkGateways', variables('gatewayName'))]"
    },
    "localNetworkGateway2": {
      "id": "[resourceId('Microsoft.Network/localNetworkGateways', variables('localGatewayName'))]"
    },
    "connectionType": "IPsec",
    "routingWeight": 0,
    "sharedKey": "[variables('sharedKey')]",
    "enableBGP": "true"
  },
  "comments": "Create a Connection type site-to-site (IPsec) between the Azure VPN Gateway and the VPN device on-premises"
  }
```

## <a name="4-configure-the-on-premises-vpn-device"></a><a name="device"></a>4. Configuración del dispositivo VPN local

Azure VPN Gateway es compatible con muchos dispositivos VPN de diferentes proveedores. Para obtener información sobre la configuración y los dispositivos que se han validado para funcionar con VPN Gateway, consulte [Acerca de los dispositivos VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md).

Para configurar el dispositivo VPN, necesita los elementos siguientes:

* Una clave compartida. Se trata de la misma clave compartida que se especifica al crear la conexión VPN de sitio a sitio. En los ejemplos se utiliza una clave compartida básica. Se recomienda que genere y utilice una clave más compleja.
* Se trata de la dirección IP pública de VPN Gateway. Puede ver la dirección IP pública mediante Azure Portal, PowerShell o la CLI. Para buscar la dirección IP pública de VPN Gateway mediante Azure Portal, vaya a Puertas de enlace de red virtual y haga clic en el nombre de la puerta de enlace.

Normalmente los pares eBGP están conectados directamente (a menudo a través de una conexión WAN). Sin embargo, cuando se configura eBGP a través de túneles VPN IPsec mediante el emparejamiento de Microsoft de ExpressRoute, existen varios dominios de enrutamiento entre los pares BGP. Use el comando **ebgp-multihop** para establecer la relación de vecino eBGP entre los dos pares no conectados directamente. El entero que sigue al comando ebgp-multihop especifica el valor de TTL en los paquetes BGP. El comando **maximum-paths eibgp 2** habilita el equilibrio de carga de tráfico entre las dos rutas BGP.

### <a name="cisco-csr1000-example"></a><a name="cisco1"></a>Ejemplo de CSR1000 de Cisco

En el ejemplo siguiente se muestra la configuración de CSR1000 de Cisco en una máquina virtual de Hyper-V como el dispositivo VPN local:

```
!
crypto ikev2 proposal az-PROPOSAL
 encryption aes-cbc-256 aes-cbc-128 3des
 integrity sha1
 group 2
!
crypto ikev2 policy az-POLICY
 proposal az-PROPOSAL
!
crypto ikev2 keyring key-peer1
 peer azvpn1
  address 52.175.253.112
  pre-shared-key secret*1234
 !
!
crypto ikev2 keyring key-peer2
 peer azvpn2
  address 52.175.250.191
  pre-shared-key secret*1234
 !
!
!
crypto ikev2 profile az-PROFILE1
 match address local interface GigabitEthernet1
 match identity remote address 52.175.253.112 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local key-peer1
!
crypto ikev2 profile az-PROFILE2
 match address local interface GigabitEthernet1
 match identity remote address 52.175.250.191 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local key-peer2
!
crypto ikev2 dpd 10 2 on-demand
!
!
crypto ipsec transform-set az-IPSEC-PROPOSAL-SET esp-aes 256 esp-sha-hmac
 mode tunnel
!
crypto ipsec profile az-VTI1
 set transform-set az-IPSEC-PROPOSAL-SET
 set ikev2-profile az-PROFILE1
!
crypto ipsec profile az-VTI2
 set transform-set az-IPSEC-PROPOSAL-SET
 set ikev2-profile az-PROFILE2
!
!
interface Loopback0
 ip address 172.16.0.3 255.255.255.255
!
interface Tunnel0
 ip address 172.16.0.1 255.255.255.255
 ip tcp adjust-mss 1350
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 52.175.253.112
 tunnel protection ipsec profile az-VTI1
!
interface Tunnel1
 ip address 172.16.0.2 255.255.255.255
 ip tcp adjust-mss 1350
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 52.175.250.191
 tunnel protection ipsec profile az-VTI2
!
interface GigabitEthernet1
 description External interface
 ip address x.243.229.110 255.255.255.252
 negotiation auto
 no mop enabled
 no mop sysid
!
interface GigabitEthernet2
 ip address 10.0.0.1 255.255.255.0
 negotiation auto
 no mop enabled
 no mop sysid
!
router bgp 65010
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 network 10.0.0.0 mask 255.255.255.0
 network 10.1.10.0 mask 255.255.255.128
 neighbor 10.2.0.228 remote-as 65000
 neighbor 10.2.0.228 ebgp-multihop 5
 neighbor 10.2.0.228 update-source Loopback0
 neighbor 10.2.0.228 soft-reconfiguration inbound
 neighbor 10.2.0.228 filter-list 10 out
 neighbor 10.2.0.229 remote-as 65000    
 neighbor 10.2.0.229 ebgp-multihop 5
 neighbor 10.2.0.229 update-source Loopback0
 neighbor 10.2.0.229 soft-reconfiguration inbound
 maximum-paths eibgp 2
!
ip route 0.0.0.0 0.0.0.0 10.1.10.1
ip route 10.2.0.228 255.255.255.255 Tunnel0
ip route 10.2.0.229 255.255.255.255 Tunnel1
!
```

## <a name="5-configure-vpn-device-filtering-and-firewalls-optional"></a><a name="firewalls"></a>5. Configuración del filtrado de dispositivos VPN y firewalls (opcional)

Configure el firewall y el filtrado de acuerdo con sus requisitos.

## <a name="6-test-and-validate-the-ipsec-tunnel"></a><a name="testipsec"></a>6. Prueba y validación del túnel IPsec

El estado de los túneles IPsec se puede comprobar en Azure VPN Gateway mediante los comandos de Powershell:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayConnection -Name vpn2local1 -ResourceGroupName myRG | Select-Object  ConnectionStatus,EgressBytesTransferred,IngressBytesTransferred | fl
```

Salida de ejemplo:

```azurepowershell
ConnectionStatus        : Connected
EgressBytesTransferred  : 17734660
IngressBytesTransferred : 10538211
```

Para comprobar el estado de los túneles en las instancias de Azure VPN Gateway de forma independiente, use el ejemplo siguiente:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayConnection -Name vpn2local1 -ResourceGroupName myRG | Select-Object -ExpandProperty TunnelConnectionStatus
```

Salida de ejemplo:

```azurepowershell
Tunnel                           : vpn2local1_52.175.250.191
ConnectionStatus                 : Connected
IngressBytesTransferred          : 4877438
EgressBytesTransferred           : 8754071
LastConnectionEstablishedUtcTime : 11/04/2017 17:03:30

Tunnel                           : vpn2local1_52.175.253.112
ConnectionStatus                 : Connected
IngressBytesTransferred          : 5660773
EgressBytesTransferred           : 8980589
LastConnectionEstablishedUtcTime : 11/04/2017 17:03:13
```

También puede comprobar el estado del túnel en el dispositivo VPN local.

Ejemplo de CSR1000 de Cisco:

```
show crypto session detail
show crypto ikev2 sa
show crypto ikev2 session detail
show crypto ipsec sa
```

Salida de ejemplo:

```
csr1#show crypto session detail

Crypto session current status

Code: C - IKE Configuration mode, D - Dead Peer Detection
K - Keepalives, N - NAT-traversal, T - cTCP encapsulation
X - IKE Extended Authentication, F - IKE Fragmentation
R - IKE Auto Reconnect

Interface: Tunnel1
Profile: az-PROFILE2
Uptime: 00:52:46
Session status: UP-ACTIVE
Peer: 52.175.250.191 port 4500 fvrf: (none) ivrf: (none)
      Phase1_id: 52.175.250.191
      Desc: (none)
  Session ID: 3
  IKEv2 SA: local 10.1.10.50/4500 remote 52.175.250.191/4500 Active
          Capabilities:DN connid:3 lifetime:23:07:14
  IPSEC FLOW: permit ip 0.0.0.0/0.0.0.0 0.0.0.0/0.0.0.0
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 279 drop 0 life (KB/Sec) 4607976/433
        Outbound: #pkts enc'ed 164 drop 0 life (KB/Sec) 4607992/433

Interface: Tunnel0
Profile: az-PROFILE1
Uptime: 00:52:43
Session status: UP-ACTIVE
Peer: 52.175.253.112 port 4500 fvrf: (none) ivrf: (none)
      Phase1_id: 52.175.253.112
      Desc: (none)
  Session ID: 2
  IKEv2 SA: local 10.1.10.50/4500 remote 52.175.253.112/4500 Active
          Capabilities:DN connid:2 lifetime:23:07:17
  IPSEC FLOW: permit ip 0.0.0.0/0.0.0.0 0.0.0.0/0.0.0.0
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 668 drop 0 life (KB/Sec) 4607926/437
        Outbound: #pkts enc'ed 477 drop 0 life (KB/Sec) 4607953/437
```

El protocolo de línea en la interfaz de túnel virtual (VTI) no cambia a "activo" hasta que se completa la fase 2 de IKE. El comando siguiente comprueba la asociación de seguridad:

```
csr1#show crypto ikev2 sa

IPv4 Crypto IKEv2  SA

Tunnel-id Local                 Remote                fvrf/ivrf            Status
2         10.1.10.50/4500       52.175.253.112/4500   none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3277 sec

Tunnel-id Local                 Remote                fvrf/ivrf            Status
3         10.1.10.50/4500       52.175.250.191/4500   none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3280 sec

IPv6 Crypto IKEv2  SA

csr1#show crypto ipsec sa | inc encaps|decaps
    #pkts encaps: 177, #pkts encrypt: 177, #pkts digest: 177
    #pkts decaps: 296, #pkts decrypt: 296, #pkts verify: 296
    #pkts encaps: 554, #pkts encrypt: 554, #pkts digest: 554
    #pkts decaps: 746, #pkts decrypt: 746, #pkts verify: 746
```

### <a name="verify-end-to-end-connectivity-between-the-inside-network-on-premises-and-the-azure-vnet"></a><a name="verifye2e"></a>Comprobación de la conectividad de extremo a extremo entre la red interior local y la red virtual de Azure

Si los túneles IPsec están activos y las rutas estáticas están establecidas correctamente, debe poder hacer ping a la dirección IP del par BGP remoto:

```
csr1#ping 10.2.0.228
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.0.228, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/5 ms

#ping 10.2.0.229
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.0.229, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/6 ms
```

### <a name="verify-the-bgp-sessions-over-ipsec"></a><a name="verifybgp"></a>Comprobación de las sesiones BGP a través de IPsec

En Azure VPN Gateway, compruebe el estado del par BGP:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayBGPPeerStatus -VirtualNetworkGatewayName vpnGtw -ResourceGroupName SEA-C1-VPN-ER | ft
```

Salida de ejemplo:

```azurepowershell
  Asn ConnectedDuration LocalAddress MessagesReceived MessagesSent Neighbor    RoutesReceived State    
  --- ----------------- ------------ ---------------- ------------ --------    -------------- -----    
65010 00:57:19.9003584  10.2.0.228               68           72   172.16.0.10              2 Connected
65000                   10.2.0.228                0            0   10.2.0.228               0 Unknown  
65000 07:13:51.0109601  10.2.0.228              507          500   10.2.0.229               6 Connected
```

Para comprobar la lista de prefijos de red recibidos a través de eBGP desde el concentrador VPN local, puede filtrar por el atributo "Origin":

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayLearnedRoute -VirtualNetworkGatewayName vpnGtw -ResourceGroupName myRG  | Where-Object Origin -eq "EBgp" |ft
```

En la salida del ejemplo, ASN 65010 es el número de sistema autónomo de BGP en la VPN local.

```azurepowershell
AsPath LocalAddress Network      NextHop     Origin SourcePeer  Weight
------ ------------ -------      -------     ------ ----------  ------
65010  10.2.0.228   10.1.10.0/25 172.16.0.10 EBgp   172.16.0.10  32768
65010  10.2.0.228   10.0.0.0/24  172.16.0.10 EBgp   172.16.0.10  32768
```

Para ver la lista de rutas anunciadas:

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName vpnGtw -ResourceGroupName myRG -Peer 10.2.0.228 | ft
```

Salida de ejemplo:

```azurepowershell
AsPath LocalAddress Network        NextHop    Origin SourcePeer Weight
------ ------------ -------        -------    ------ ---------- ------
       10.2.0.229   10.2.0.0/24    10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.10/32 10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.5/32  10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.1/32  10.2.0.229 Igp                  0
65010  10.2.0.229   10.1.10.0/25   10.2.0.229 Igp                  0
65010  10.2.0.229   10.0.0.0/24    10.2.0.229 Igp                  0
```

Ejemplo para la red CSR1000 local de Cisco:

```
csr1#show ip bgp neighbors 10.2.0.228 routes
BGP table version is 7, local router ID is 172.16.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   10.2.0.0/24      10.2.0.228                             0 65000 i
 r>   172.16.0.1/32    10.2.0.228                             0 65000 i
 r>   172.16.0.2/32    10.2.0.228                             0 65000 i
 r>   172.16.0.3/32   10.2.0.228                             0 65000 i

Total number of prefixes 4
```

La lista de redes anunciada desde la red CSR1000 local de Cisco en Azure VPN Gateway se puede mostrar con el comando siguiente:

```
csr1#show ip bgp neighbors 10.2.0.228 advertised-routes
BGP table version is 7, local router ID is 172.16.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   10.0.0.0/24      0.0.0.0                  0         32768 i
 *>   10.1.10.0/25     0.0.0.0                  0         32768 i

Total number of prefixes 2
```

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de Network Performance Monitor para ExpressRoute](how-to-npm.md)

* [Agregar una conexión de sitio a sitio a una red virtual con una conexión de VPN Gateway existente](../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
