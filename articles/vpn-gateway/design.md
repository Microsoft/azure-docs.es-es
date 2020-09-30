---
title: Sobre el diseño de Azure VPN Gateway
description: Conozca cómo puede diseñar una topología de VPN Gateway para conectarse a las redes virtuales de Azure.
services: vpn-gateway
author: cherylmc
Customer intent: As someone with a basic network background, but is new to Azure, I want to understand the capabilities of Azure VPN Gateway so that I can securely connect to my Azure virtual networks.
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: ecfb569f2791b674fe842e1c21cba972a8e9d2a7
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89443014"
---
# <a name="vpn-gateway-design"></a>Diseño de VPN Gateway

Es importante saber que hay distintas configuraciones disponibles para las conexiones de VPN Gateway. Es preciso determinar qué configuración es la que mejor se adapta a sus necesidades. En las secciones siguientes, puede ver diagramas de topología e información de diseño sobre las siguientes conexiones de VPN Gateway. Use los gráficos y las descripciones como ayuda para seleccionar la topología de conexión que mejor se ajuste a sus requisitos. Los diagramas muestran las principales topologías de referencia, pero también se pueden crear configuraciones más complejas con los diagramas como guía.

## <a name="site-to-site-and-multi-site-ipsecike-vpn-tunnel"></a><a name="s2smulti"></a>Sitio a sitio y multisitio (túnel VPN de IPsec/IKE)

### <a name="site-to-site"></a><a name="S2S"></a>De sitio a sitio

Una conexión de puerta de enlace de VPN de sitio a sitio (S2S) es una conexión a través de un túnel VPN IPsec/IKE (IKEv1 o IKEv2). Se pueden usar conexiones S2S para las configuraciones híbridas y entre locales. Una conexión S2S requiere un dispositivo VPN local que tenga una dirección IP pública asignada. Para más información acerca de cómo seleccionar un dispositivo VPN, consulte la sección de [preguntas frecuentes sobre VPN Gateway para dispositivos VPN](vpn-gateway-vpn-faq.md#s2s).

![Ejemplo de conexión de sitio a sitio de Azure VPN Gateway](./media/design/vpngateway-site-to-site-connection-diagram.png)

VPN Gateway se puede configurar en modo activo-espera mediante una dirección IP pública o en modo activo-activo con dos direcciones IP públicas. En modo activo-espera, hay un túnel IPsec activo y el otro túnel está en espera. En esta configuración, el tráfico fluye a través del túnel activo y, si se produce algún problema con este túnel, el tráfico cambia al túnel en espera. Se *recomienda* la configuración de VPN Gateway en modo activo-activo, porque ambos túneles de IPsec están activos simultáneamente, con datos que fluyen a través de los dos al mismo tiempo. Una ventaja adicional del modo activo-activo es que los clientes experimentan un mayor rendimiento.

### <a name="multi-site"></a><a name="Multi"></a>Multisitio

Este tipo de conexión es una variación de la conexión de sitio a sitio. Puede crear más de una conexión VPN desde la puerta de enlace de red virtual, normalmente conectándose a varios sitios locales. Cuando trabaje con varias conexiones, debe usar una VPN de tipo RouteBased (conocida como puerta de enlace dinámica al trabajar con redes virtuales clásicas). Como cada red virtual solo puede tener una puerta de enlace de red virtual, todas las conexiones a través de la puerta de enlace comparten el ancho de banda disponible. Este tipo de conexión se denomina con frecuencia, conexión "multisitio".

![Ejemplo de conexión multisitio de Azure VPN Gateway](./media/design/vpngateway-multisite-connection-diagram.png)

### <a name="deployment-models-and-methods-for-site-to-site-and-multi-site"></a>Modelos de implementación y métodos para conexiones de sitio a sitio y multisitio

[!INCLUDE [site-to-site and multi-site table](../../includes/vpn-gateway-table-site-to-site-include.md)]

## <a name="point-to-site-vpn"></a><a name="P2S"></a>VPN de punto a sitio

Una conexión de puerta de enlace de VPN de punto a sitio (P2S) permite crear una conexión segura a la red virtual desde un equipo cliente individual. Se establece una conexión de punto a sitio al iniciarla desde el equipo cliente. Esta solución resulta útil para los teletrabajadores que deseen conectarse a redes virtuales de Azure desde una ubicación remota, por ejemplo, desde casa o un congreso. La conexión VPN de punto a sitio también es una solución útil en comparación con la conexión VPN de sitio a sitio cuando solo necesitan conectarse a la red virtual algunos clientes.

A diferencia de las conexiones S2S, las conexiones P2S no necesitan una dirección IP pública local ni dispositivos VPN. Se pueden usar conexiones P2S con conexiones S2S a través de la misma instancia de VPN Gateway, siempre que todos los requisitos de configuración para ambas conexiones sean compatibles. Para más información sobre las conexiones de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).

![Ejemplo de conexión de punto a sitio de Azure VPN Gateway](./media/design/point-to-site.png)

### <a name="deployment-models-and-methods-for-p2s"></a>Métodos y modelos de implementación para P2S

[!INCLUDE [vpn-gateway-table-site-to-site](../../includes/vpn-gateway-table-point-to-site-include.md)]

## <a name="vnet-to-vnet-connections-ipsecike-vpn-tunnel"></a><a name="V2V"></a>Conexiones de red virtual a red virtual (túnel VPN de IPsec/IKE)

La conexión de una red virtual a otra es muy parecida a la conexión de una red virtual a una ubicación de un sitio local. Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro con IPsec/IKE. Incluso puede combinar la comunicación de red virtual a red virtual con configuraciones de conexión multisitio. Esto permite establecer topologías de red que combinen la conectividad entre entornos con la conectividad entre redes virtuales.

Las redes virtuales que conecta pueden:

* estar en la misma región o en distintas;
* pertenecer a la misma suscripción o a distintas; 
* usar el mismo modelo de implementación o diferentes.

![Ejemplo de conexión de red virtual a red virtual de Azure VPN Gateway](./media/design/vpngateway-vnet-to-vnet-connection-diagram.png)

### <a name="connections-between-deployment-models"></a>Conexiones entre modelos de implementación

Actualmente, Azure tiene dos modelos de implementación: el clásico y el de Resource Manager. Si lleva un tiempo usando Azure, es probable que tenga máquinas virtuales de Azure y roles de instancia que se ejecuten en una red virtual clásica. Es posible que sus máquinas virtuales e instancias de roles más recientes se estén ejecutando en una red virtual creada en Resource Manager. Puede crear una conexión entre las redes virtuales para permitir que los recursos de una red virtual se comuniquen directamente con los recursos de otra.

### <a name="vnet-peering"></a>Emparejamiento de VNET

Es posible que pueda usar el emparejamiento de VNET para crear la conexión, siempre que la red virtual cumpla determinados requisitos. El emparejamiento de VNET no utiliza una puerta de enlace de red virtual. Para más información, consulte [Emparejamiento de VNET](../virtual-network/virtual-network-peering-overview.md).

### <a name="deployment-models-and-methods-for-vnet-to-vnet"></a>Modelos de implementación y métodos para conexiones de red virtual a red virtual

[!INCLUDE [vpn-gateway-table-vnet-to-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

## <a name="expressroute-private-connection"></a><a name="ExpressRoute"></a>ExpressRoute (conexión privada)

ExpressRoute le permite ampliar sus redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad. Con ExpressRoute, se pueden establecer conexiones con servicios en la nube de Microsoft, como Microsoft Azure, Office 365 y CRM Online. La conectividad puede ser desde una red de conectividad universal (IP VPN), una red Ethernet de punto a punto o una conexión cruzada virtual a través de un proveedor de conectividad en una instalación de ubicación compartida.

Las conexiones ExpressRoute no pasan por la red pública de Internet. Esto permite a las conexiones de ExpressRoute ofrecer más confiabilidad, más velocidad, menor latencia y mayor seguridad que las conexiones normales a través de Internet.

Una conexión de ExpressRoute usa una puerta de enlace de red virtual como parte de su configuración obligatoria. En una conexión de ExpressRoute, se configura una puerta de enlace de red virtual con el tipo de puerta de enlace "ExpressRoute", en lugar de "Vpn". Aunque el tráfico que pasa por un circuito de ExpressRoute no está cifrado de forma predeterminada, es posible crear una solución que le permita enviar tráfico cifrado a través de este. Para más información sobre ExpressRoute, vea la [Información técnica de ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="site-to-site-and-expressroute-coexisting-connections"></a><a name="coexisting"></a>Conexiones de sitio a sitio y de ExpressRoute coexistentes

ExpressRoute es una conexión privada directa desde la WAN (no a través de Internet) a servicios Microsoft, incluido Azure. El tráfico VPN de sitio a sitio viaja cifrado a través de la red pública de Internet. Poder configurar las conexiones VPN de sitio a sitio y ExpressRoute para la misma red virtual tiene varias ventajas.

Puede configurar una VPN de sitio a sitio como una ruta de acceso seguro de conmutación por error para ExpressRoute, o bien usar la VPN de sitio a sitio para conectarse a sitios que no forman parte de su red, pero que están conectados a través de ExpressRoute. Tenga en cuenta que esta configuración requiere dos puertas de enlace de red virtual en la misma red virtual, una con el tipo de puerta de enlace "Vpn" y otra con -el tipo de puerta de enlace "ExpressRoute".

![Ejemplo de conexiones coexistentes en ExpressRoute y VPN Gateway](./media/design/expressroute-vpngateway-coexisting-connections-diagram.png)

### <a name="deployment-models-and-methods-for-s2s-and-expressroute-coexist"></a>Coexisten diversos métodos y modelos de implementación de conexiones S2S y ExpressRoute

[!INCLUDE [vpn-gateway-table-coexist](../../includes/vpn-gateway-table-coexist-include.md)]

## <a name="highly-available-connections"></a><a name="highly-available"></a>Conexiones de alta disponibilidad

Para planear y diseñar conexiones de alta disponibilidad, consulte [Conexiones de alta disponibilidad](vpn-gateway-highlyavailable.md).

## <a name="next-steps"></a>Pasos siguientes

* Consulte las [Preguntas más frecuentes sobre VPN Gateway](vpn-gateway-vpn-faq.md) para más información.

* Obtenga más información sobre la [configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md).

* Para obtener consideraciones sobre BGP de VPN Gateway, consulte [Acerca de BGP](vpn-gateway-bgp-overview.md).

* Consulte [Límites del servicio y la suscripción](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

* Obtenga información sobre las demás [funcionalidades de red](../networking/networking-overview.md) clave de Azure.
