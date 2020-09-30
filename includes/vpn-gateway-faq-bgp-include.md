---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 09/17/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 32e4658af48a0ae3bde08de18cf1d8204878d671
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "91025143"
---
### <a name="is-bgp-supported-on-all-azure-vpn-gateway-skus"></a>¿Se admite BGP en todas las SKU de Azure VPN Gateway?
El protocolo de puerta de enlace de borde se admite en todas las SKU de Azure VPN Gateway, excepto en la SKU básica.

### <a name="can-i-use-bgp-with-azure-policy-based-vpn-gateways"></a>¿Puedo usar BGP con puertas de enlace de VPN basadas en directivas de Azure?
No, BGP solo es compatible con puertas de enlace de VPN basadas en enrutamiento.

### <a name="what-asn-autonomous-system-numbers-can-i-use"></a>¿Qué ASN (números de sistema autónomo) puedo usar?
Puede usar sus propios ASN públicos o privados tanto en sus redes locales como en las redes virtuales de Azure, **excepto** los rangos reservados por Azure o IANA:

> [!IMPORTANT]
>
> Los siguientes ASN están reservados por Azure o IANA:
> * ASN reservados por Azure:
>    * ASN públicos: 8074, 8075, 12076
>    * ASN privados: 65515, 65517, 65518, 65519, 65520
> * ASN [reservados por IANA](http://www.iana.org/assignments/iana-as-numbers-special-registry/iana-as-numbers-special-registry.xhtml)
>    * 23456, 64496-64511, 65535-65551 y 429496729
>
> Estos ASN no se pueden especificar para sus los dispositivos VPN locales al conectarse con las puertas de enlace de VPN.
>

### <a name="can-i-use-32-bit-4-byte-asns-autonomous-system-numbers"></a>¿Puedo usar ASN (números de sistema autónomo) de 32 bits (4 bytes)?
Sí, Azure VPN Gateway ya admite ASN de 32 bits (4 bytes). Use PowerShell, la CLI o el SDK para realizar la configuración mediante el uso de ASN en formato decimal.

### <a name="what-private-asns-can-i-use"></a>¿Qué ASN privados puedo usar?
El intervalo de ASN privados disponible que se puede usar es:

* 64512-65514, 65521-65534

Estos ASN no están reservados por IANA ni por Azure para su uso y, por tanto, se pueden utilizar para asignarlos a su instancia de Azure VPN Gateway.

### <a name="what-address-does-azure-vpn-gateway-use-for-bgp-peer-ip"></a>¿Qué dirección utiliza la puerta de enlace de VPN de Azure para la dirección IP del par BGP?

* De forma predeterminada, la puerta de enlace de VPN de Azure asignará una única dirección IP del rango de GatewaySubnet a las puertas de enlace de VPN en modo activo/en espera, o bien dos direcciones IP para puertas de enlace de VPN en modo activo/activo. Estas direcciones se asignan automáticamente al crear la puerta de enlace de VPN. Puede obtener las direcciones IP de BGP reales asignadas mediante PowerShell (Get-AzVirtualNetworkGateway, busque la propiedad "bgpPeeringAddress"), o en Azure Portal (en la propiedad "Configurar ASN BGP" de la página Configuración de puerta de enlace).

* Si los enrutadores locales de la red privada virtual utilizan las direcciones IP (169.254.x.x) de **APIPA** como direcciones IP del protocolo de puerta de enlace de borde, es preciso especificar una **dirección IP de BGP de APIPA de Azure** adicional en la puerta de enlace de VPN de Azure. La puerta de enlace de VPN seleccionará la dirección de APIPA que se va a usar con el par BGP de APIPA especificado de la puerta de enlace de red local, o bien una dirección IP privada, en el caso del par BGP local que no sea APIPA. Para más información, consulte el artículo sobre la [configuración del protocolo de puerta de enlace de borde](../articles/vpn-gateway/bgp-howto.md).

### <a name="what-are-the-requirements-for-the-bgp-peer-ip-addresses-on-my-vpn-device"></a>¿Cuáles son los requisitos para las direcciones IP del par BGP en mi dispositivo VPN?
La dirección del par BGP local **NO DEBE** coincidir con la dirección IP pública del dispositivo VPN ni con el espacio de direcciones de red virtual de VPN Gateway. Utilice otra dirección IP en el dispositivo VPN para la dirección IP del par BGP. Puede ser una dirección asignada a la interfaz de bucle invertido en el dispositivo, y puede ser una dirección IP normal o una dirección de APIPA (169.254.x.x). Si el dispositivo usa una dirección de APIPA para el protocolo de puerta de enlace de borde, es preciso especificar una dirección IP para BGP de APIPA BGP en la puerta de enlace de VPN de Azure VPN, como se describe en el artículo sobre la [configuración del protocolo de puerta de enlace de borde](../articles/vpn-gateway/bgp-howto.md). Especifique esta dirección en la puerta de enlace de red local correspondiente que representa la ubicación.

### <a name="what-should-i-specify-as-my-address-prefixes-for-the-local-network-gateway-when-i-use-bgp"></a>¿Qué debo especificar como mis prefijos de dirección para la puerta de enlace de red local al utilizar BGP?

> [!IMPORTANT]
>
>Aquí se ha producido un cambio, con respecto al requisito que estaba documentado. Si se usa el protocolo de puerta de enlace de borde para una conexión, es preciso dejar el campo **Espacio de direcciones** ***vacío*** del recurso de la puerta de enlace de red local correspondiente. La puerta de enlace de VPN de Azure agregará internamente una ruta de host a la IP del par BGP local a través del túnel de IPsec. **NO** agregue la ruta /32 en el campo Espacio de direcciones. Es redundante y si usa una dirección de APIPA como la IP del protocolo de puerta de enlace de borde del dispositivo VPN local, no se puede agregar a este campo. Si agrega otros prefijos en el campo Espacio de direcciones, se agregarán como **rutas estáticas** en la puerta de enlace de VPN de Azure, además de las rutas aprendidas a través del protocolo de puerta de enlace de borde.
>

### <a name="can-i-use-the-same-asn-for-both-on-premises-vpn-networks-and-azure-vnets"></a>¿Puedo usar el mismo ASN para redes VPN locales y redes virtuales de Azure?
No, debe asignar ASN diferentes entre las redes locales y sus redes virtuales de Azure si las está conectando con BGP. Azure VPN Gateway tiene un ASN de 65515 predeterminado asignado, independientemente de que BGP esté habilitado, o no, para la conectividad entre locales. Para invalidar este valor predeterminado, asigne otro ASN al crear la puerta de enlace de VPN o cambie el ASN después de crearla. Debe asignar los ASN locales a las puertas de enlace de red local de Azure correspondientes.

### <a name="what-address-prefixes-will-azure-vpn-gateways-advertise-to-me"></a>¿Qué prefijos de direcciones de puertas de enlace de VPN de Azure se me anunciarán?
La puerta de enlace de VPN de Azure anunciará las siguientes rutas a sus dispositivos BGP locales:

* Sus prefijos de dirección de red virtual
* Los prefijos de dirección de todas las puertas de enlace de red local conectadas a la puerta de enlace de VPN de Azure
* Las rutas aprendidas en otras sesión de intercambio de tráfico (peering) de BGP conectadas a la puerta de enlace de VPN de Azure, **excepto la ruta predeterminada o las rutas superpuestas con cualquier prefijo de red virtual**.

### <a name="how-many-prefixes-can-i-advertise-to-azure-vpn-gateway"></a>¿Cuántos prefijos se pueden anunciar en Azure VPN Gateway?
Se admiten hasta 4000 prefijos. Si el número de prefijos supera el límite, se elimina la sesión BGP.

### <a name="can-i-advertise-default-route-00000-to-azure-vpn-gateways"></a>¿Puedo anunciar la ruta predeterminada (0.0.0.0/0) para puertas de enlace de VPN de Azure?
Sí.

Tenga en cuenta que esto obligará a todo el tráfico de salida de la red virtual a dirigirse a su sitio local e impedirá que las máquinas virtuales de la red virtual acepten comunicaciones públicas desde Internet directamente como, por ejemplo, RDP o SSH desde Internet a las máquinas virtuales.

### <a name="can-i-advertise-the-exact-prefixes-as-my-virtual-network-prefixes"></a>¿Puedo anunciar los mismos prefijos que los de mis prefijos de red virtual?

No, la plataforma de Azure bloqueará o filtrará el anuncio de los mismos prefijos que los de cualquiera de sus otros prefijos de dirección de red virtual. Sin embargo, puede anunciar un prefijo que sea un superconjunto de lo que contenga su red virtual. 

Por ejemplo, si una red virtual usara el espacio de direcciones 10.0.0.0/16, se podría anunciar 10.0.0.0/8. Sin embargo, no se pueden anunciar 10.0.0.0/16 o 10.0.0.0/24.

### <a name="can-i-use-bgp-with-my-vnet-to-vnet-connections"></a>¿Puedo usar BGP con mis conexiones de red virtual a red virtual?
Sí, puede utilizar BGP para conexiones entre locales y conexiones de red virtual a red virtual.

### <a name="can-i-mix-bgp-with-non-bgp-connections-for-my-azure-vpn-gateways"></a>¿Puedo combinar BGP con conexiones no de BGP para mi puertas de enlace de VPN de Azure?
Sí, puede mezclar conexiones de BGP y no de BGP para la misma puerta de enlace de VPN de Azure.

### <a name="does-azure-vpn-gateway-support-bgp-transit-routing"></a>¿Azure VPN Gateway admite el enrutamiento del tránsito de BGP?
Sí, se admite el enrutamiento del tránsito de BGP, con la excepción de que las puertas de enlace de VPN de Azure **NO** anunciarán las rutas predeterminadas a otros pares de BGP. Para habilitar el enrutamiento del tránsito a través de varias puertas de enlace de VPN de Azure, debe habilitar BGP en todas las conexiones de red virtual a red virtual intermedias. Para más información, consulte [Acerca de BGP](../articles/vpn-gateway/vpn-gateway-bgp-overview.md).

### <a name="can-i-have-more-than-one-tunnel-between-azure-vpn-gateway-and-my-on-premises-network"></a>¿Puedo tener más de un túnel entre mi red local y Azure VPN Gateway?
Sí, puede establecer más de un túnel VPN S2S entre Azure VPN Gateway y la red local. Tenga en cuenta que todos estos túneles se incluirán en el recuento total de túneles de las puertas de enlace de la VPN de Azure y el protocolo de puerta de enlace de borde debe habilitarse en los dos túneles.

Por ejemplo, si tiene dos túneles redundantes entre una puerta de enlace de VPN de Azure y una de las redes locales, consumirán 2 túneles de la cuota total para la puerta de enlace de VPN de Azure.

### <a name="can-i-have-multiple-tunnels-between-two-azure-vnets-with-bgp"></a>¿Puedo tener varios túneles entre dos redes virtuales de Azure con BGP?
Sí, pero al menos una de las puertas de enlace de la red virtual debe estar en una configuración activa-activa.

### <a name="can-i-use-bgp-for-s2s-vpn-in-an-expressroutes2s-vpn-coexistence-configuration"></a>¿Se puede usar BGP para VPN de sitio a sitio en una configuración en que coexisten VPN de sitio a sitio y ExpressRoute?
Sí. 

### <a name="what-should-i-add-to-my-on-premises-vpn-device-for-the-bgp-peering-session"></a>¿Qué debo agregar a mi dispositivo VPN local para la sesión de emparejamiento BGP?
Debe agregar una ruta de host de la dirección IP del par BGP de Azure en el dispositivo VPN que apunta al túnel VPN S2S de IPsec. Por ejemplo, si la dirección IP del par VPN de Azure es "10.12.255.30", debe agregar una ruta de host para "10.12.255.30" con una interfaz de próximo salto de la interfaz de túnel IPsec coincidente en el dispositivo VPN.
