---
title: Información general sobre los servicios de red de Azure
description: Aprenda sobre los servicios de red de Azure, por ejemplo, los servicios de conectividad, protección de aplicaciones, entrega de aplicaciones y supervisión de la red.
services: networking
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.workload: infrastructure-services
ms.date: 03/12/2020
ms.author: kumud
ms.openlocfilehash: 39601da6cc8e10ba8f4a24eeddb97cfe677c031f
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651736"
---
# <a name="azure-networking-services-overview"></a>Información general sobre los servicios de red de Azure

Los servicios de red de Azure proporcionan diversas funcionalidades de red que se pueden usar conjuntamente o por separado. Haga clic en cualquiera de las siguientes funcionalidades principales para obtener más información acerca de ellas:
- [**Servicios de conectividad**](#connect). Conecte recursos de Azure y recursos del entorno local con cualquier combinación de estos servicios de red en Azure: Virtual Network (VNet), Virtual WAN, ExpressRoute, VPN Gateway, NAT Gateway de Virtual Network, Azure DNS, Peering Service y Azure Bastion.
- [**Servicios de protección de aplicaciones**](#protect): Proteja sus aplicaciones con una combinación de estos servicios de red en Azure: Private Link, DDoS Protection, Firewall, grupos de seguridad de red, Firewall de aplicaciones web o puntos de conexión de red virtual.
- [**Servicios de entrega de aplicaciones**](#deliver): Entregue aplicaciones en la red de Azure usando uno o varios de estos servicios de red en Azure: Content Delivery Network (CDN), Azure Front Door Service, Traffic Manager, Application Gateway, Internet Analyzer y Load Balancer.
- [**Supervisión de red**](#monitor): Supervise los recursos de red mediante uno o varios de estos servicios de red en Azure: Network Watcher, Supervisión de ExpressRoute, Azure Monitor o Punto de acceso de terminal de red virtual (TAP).

## <a name="connectivity-services"></a><a name="connect"></a>Servicios de conectividad
 
En esta sección se describen los servicios que proporcionan conectividad entre los recursos de Azure, conectividad desde una red en el entorno local a los recursos de Azure y conectividad de rama a rama en Azure: Virtual Network (VNet), Virtual WAN, ExpressRoute, VPN Gateway, NAT Gateway de Virtual Network, Azure DNS, Azure Peering Service y Azure Bastion.

|Servicio|Motivos para la utilización|Escenarios|
|---|---|---|
|[Red virtual](#vnet)|Permite que varios recursos de Azure se puedan comunicar de forma segura entre ellos, con Internet y con redes en el entorno local.| <p>[Filtrado del tráfico de red](../virtual-network/tutorial-filter-network-traffic.md)</p> <p>[Enrutado del tráfico de red](../virtual-network/tutorial-create-route-table-portal.md)</p> <p>[Restricción del acceso de la red a recursos](../virtual-network/tutorial-restrict-network-access-to-resources.md)</p> <p>[Conexión de redes virtuales](../virtual-network/tutorial-connect-virtual-networks-portal.md)</p>|
|[ExpressRoute](#expressroute)|Amplía las redes en el entorno local en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad.|<p>[Creación y modificación de un circuito ExpressRoute](../expressroute/expressroute-howto-circuit-portal-resource-manager.md)</p> <p>[Creación y modificación del emparejamiento de un circuito ExpressRoute](../expressroute/expressroute-howto-routing-portal-resource-manager.md)</p> <p>[Vinculación de redes virtuales a circuitos ExpressRoute](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)</p> <p>[Configuración y administración de filtros de ruta para circuitos ExpressRoute](../expressroute/how-to-routefilter-portal.md)</p>|
|[VPN Gateway](#vpngateway)|Envía el tráfico cifrado entre una red virtual de Azure y una ubicación local a través de Internet público.|<p>[Conexiones de sitio a sitio](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)</p> <p>[Conexiones de red virtual a red virtual](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)</p> <p>[Conexiones de punto a sitio](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)</p>|
|[Virtual WAN](#virtualwan)|Optimiza y automatiza la conectividad de las ramas a Azure y en ella. Las regiones de Azure sirven como centros que se pueden elegir para conectar las distintas ramas.|<p>[Conexiones de sitio a sitio](../virtual-wan/virtual-wan-site-to-site-portal.md), [conexiones ExpressRoute](../virtual-wan/virtual-wan-expressroute-portal.md)</p>|
|[DNS de Azure](#dns)|Hospeda los dominios DNS que proporcionan la resolución de nombres mediante la infraestructura de Microsoft Azure.|<p>[Hospedaje del dominio en Azure DNS](../dns/dns-delegate-domain-azure-dns.md)</p><p>[Creación de registros DNS para una aplicación web](../dns/dns-web-sites-custom-domain.md)</p> <p>[Creación de un registro de alias para Traffic Manager](../dns/tutorial-alias-tm.md)</p> <p>[Creación de un registro de alias para la dirección IP pública](../dns/tutorial-alias-pip.md)</p> <p>[Creación de un registro de alias para el registro de recursos de zona](../dns/tutorial-alias-rr.md)</p>|
|[Azure Bastion](#bastion)|Configure una conexión RDP/SSH segura e ininterrumpida a las máquinas virtuales directamente en Azure Portal a través de TLS. Cuando se conecta a través de Azure Bastion, las máquinas virtuales no necesitan una dirección IP pública.|<p>[Creación de un host de Azure Bastion](../bastion/bastion-create-host-portal.md)</p><p>[Conexión mediante SSH a una máquina virtual Linux](../bastion/bastion-connect-vm-ssh.md)</p><p>[Conexión mediante RDP a una máquina virtual Windows](../bastion/bastion-connect-vm-rdp.md)</p>|
|[NAT Gateway de Virtual Network](#nat)|Crear una puerta de enlace NAT Gateway para proporcionar conectividad saliente en una máquina virtual.|<p>[Creación de una puerta de enlace NAT Gateway](../virtual-network/quickstart-create-nat-gateway-portal.md)</p>|
|[Azure Peering Service](#azurepeeringservice)|Colaborar con proveedores de servicios para lograr un enrutamiento a la nube de Microsoft óptimo y confiable a través de la red pública.|<p>[Registro de Azure Peering Service](../peering-service/azure-portal.md)</p>|
||||


### <a name="virtual-network"></a><a name="vnet"></a>Virtual Network

Azure Virtual Network (VNet) es el bloque de creación fundamental de una red privada en Azure. Puede usar una red virtual para:
- **Comunicarse entre recursos de Azure**: tanto las máquinas virtuales como otros tipos de recursos de Azure se pueden implementar en una red virtual, como Azure App Service Environment, Azure Kubernetes Service (AKS) y Azure Virtual Machine Scale Sets. Para ver una lista completa de los recursos de Azure que puede implementar en una red virtual, consulte [Integración de red virtual para los servicios de Azure](../virtual-network/virtual-network-for-azure-services.md).
- **Comunicarse entre sí**: Las redes virtuales se pueden conectar entre sí, lo que permite que los recursos de cualquiera de ellas se comuniquen entre sí mediante el emparejamiento de red virtual. Las redes virtuales que conecte pueden estar en la misma región de Azure o en regiones distintas. Para más información, consulte [Emparejamiento de redes virtuales](../virtual-network/virtual-network-peering-overview.md).
- **Comunicarse en Internet**: De manera predeterminada, todos los recursos de una red virtual tienen comunicación de salida hacia Internet. Para comunicarse con un recurso entrante, asígnele una dirección IP pública o un equilibrador de carga público. También puede usar [Direcciones IP públicas](../virtual-network/virtual-network-public-ip-address.md) o [Load Balancer](../load-balancer/load-balancer-overview.md) público para administrar las conexiones salientes.
- **Comunicarse con otras redes en el entorno local**: Puede conectar sus redes y equipos en el entorno local a una red virtual mediante [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../expressroute/expressroute-introduction.md).

Para más información, consulte [¿Qué es Azure Virtual Network?](../virtual-network/virtual-networks-overview.md).

### <a name="expressroute"></a><a name="expressroute"></a>ExpressRoute
ExpressRoute permite ampliar las redes en el entorno local en la nube de Microsoft Azure a través de una conexión privada que facilita un proveedor de conectividad. Esta conexión es privada. El tráfico no pasa por Internet. Con ExpressRoute, se pueden establecer conexiones con servicios en la nube de Microsoft, como Microsoft Azure, Office 365 y Dynamics 365.  Para más información, consulte [¿Qué es ExpressRoute?](../expressroute/expressroute-introduction.md)

![Azure ExpressRoute](./media/networking-overview/expressroute-connection-overview.png)

### <a name="vpn-gateway"></a><a name="vpngateway"></a>VPN Gateway
VPN Gateway ayuda a crear conexiones cifradas entre entornos locales a la red virtual desde ubicaciones locales, o bien a crear conexiones cifradas entre las redes virtuales. Hay distintas configuraciones disponibles para las conexiones VPN Gateway, por ejemplo, de sitio a sitio, de punto a sitio o de red virtual a red virtual.
En el siguiente diagrama se ilustran varias conexiones VPN de sitio a sitio a la misma red virtual.

![Conexiones de Azure VPN Gateway de sitio a sitio](./media/networking-overview/vpngateway-multisite-connection-diagram.png)

Para más información acerca de los distintos tipos de conexiones VPN, consulte [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="virtual-wan"></a><a name="virtualwan"></a>Virtual WAN
Azure Virtual WAN es un servicio de redes que ofrece conectividad entre ramas automatizada y optimizada a y mediante Azure. Las regiones de Azure sirven como centros que se pueden elegir para conectar las distintas ramas. Puede aprovechar la red troncal de Azure para conectar también ramas y disfrutar de la conectividad de rama a red virtual. Azure Virtual WAN reúne muchos servicios de conectividad en la nube de Azure como la VPN de sitio a sitio, ExpressRoute y la VPN de usuario de punto a sitio en una única interfaz operativa. La conectividad con las redes virtuales de Azure se establece mediante el uso de conexiones de red virtual. Para más información, consulte [¿Qué es Azure Virtual WAN?](../virtual-wan/virtual-wan-about.md).

![Diagrama de Virtual WAN](./media/networking-overview/virtualwan1.png)

### <a name="azure-dns"></a><a name="dns"></a>Azure DNS
Azure DNS es un servicio de hospedaje para dominios DNS que ofrece resolución de nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros de DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure. Para más información, consulte [¿Qué es Azure DNS?](../dns/dns-overview.md)

### <a name="azure-bastion"></a><a name="bastion"></a>Azure Bastion
Azure Bastion es un nuevo servicio PaaS totalmente administrado por la plataforma que se aprovisiona en redes virtuales. Proporciona una conectividad RDP/SSH segura e ininterrumpida a las máquinas virtuales directamente en Azure Portal a través de TLS. Cuando se conecta a través de Azure Bastion, las máquinas virtuales no necesitan una dirección IP pública. Para más información, consulte [¿Qué es Azure Bastion?](../bastion/bastion-overview.md).

![Arquitectura de Azure Bastion](./media/networking-overview/architecture.png)

### <a name="virtual-network-nat-gateway"></a><a name="nat"></a>NAT Gateway de Virtual Network
La NAT (traducción de direcciones de red) de Virtual Network simplifica la conectividad a Internet de solo salida para redes virtuales. Cuando se configura en una subred, todas las conexiones salientes usan las direcciones IP públicas estáticas que se hayan especificado. La conectividad saliente es posible sin que el equilibrador de carga ni las direcciones IP públicas estén conectados directamente a máquinas virtuales. Para más información, vea [¿Qué es NAT de Virtual Network?](../virtual-network/nat-overview.md) 

![Puerta de enlace NAT de Virtual Network](./media/networking-overview/flow-map.png)

### <a name="azure-peering-service"></a><a name="azurepeeringservice"></a> Azure Peering Service
Azure Peering Service mejora la conectividad de los clientes a los servicios en la nube de Microsoft, como Office 365, Dynamics 365, software como servicio (SaaS), Azure o cualquier otro servicio de Microsoft accesible a través de la red pública de Internet. Para más información, vea [¿Qué es Azure Peering Service?](../peering-service/about.md)

## <a name="application-protection-services"></a><a name="protect"></a>Servicios de protección de aplicaciones

En esta sección se describen los servicios de red en Azure que ayudan a proteger los recursos de red. Proteja sus aplicaciones con una combinación de estos servicios de red en Azure: Private Link, DDoS Protection, Firewall, grupos de seguridad de red, Firewall de aplicaciones web o puntos de conexión de red virtual.

|Servicio|Motivos para la utilización|Escenario|
|---|---|---|
|[DDoS Protection](#ddosprotection) |Alta disponibilidad para las aplicaciones gracias a la protección frente a cargos excesivos por tráfico IP|[Administración de Azure DDoS Protection](../virtual-network/manage-ddos-protection.md)|
|[Firewall de aplicaciones web](#waf)|<p>[WAF de Azure con Application Gateway](../web-application-firewall/ag/ag-overview.md) proporciona protección regional para las entidades en el espacio de direcciones público y privado</p><p>[WAF de Azure con Front Door ](../web-application-firewall/afds/afds-overview.md) proporciona protección en el perímetro de la red para los puntos de conexión públicos.</p>|<p>[Configuración de las reglas de protección contra bots](../frontdoor/waf-front-door-policy-configure-bot-protection.md)</p> <p>[Configuración de un código de respuesta personalizado](../frontdoor/waf-front-door-configure-custom-response-code.md)</p> <p>[Configuración de la lista de restricción de direcciones IP](../frontdoor/waf-front-door-configure-ip-restriction.md)</p> <p>[Configuración de la regla del límite de velocidad](../frontdoor/waf-front-door-rate-limit-powershell.md)</p> |
|[Azure Firewall](#firewall)|Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones.|<p>[Implementación de Azure Firewall en una red virtual](../firewall/tutorial-firewall-deploy-portal.md)</p> <p>[Implementación de Azure Firewall en una red híbrida](../firewall/tutorial-hybrid-ps.md)</p> <p>[Filtrado del tráfico de entrada con DNAT de Azure Firewall](../firewall/tutorial-firewall-dnat.md)</p>|
|[Grupos de seguridad de red](#nsg)|Control total pormenorizado de los nodos de extremo distribuidos en la máquina virtual o subred para todos los flujos de tráfico de red|[Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/tutorial-filter-network-traffic.md)|
|[Puntos de conexión de servicio de red virtual](#serviceendpoints)|Le permite limitar el acceso de la red a algunos recursos de servicios de Azure para una subred de la red virtual.|[Restringir el acceso de red a los recursos de PaaS](../virtual-network/tutorial-restrict-network-access-to-resources-powershell.md)|
[Private Link](#privatelink)|Permite acceder a los servicios PaaS de Azure (por ejemplo, Azure Storage y SQL Database) y a los servicios hospedados en Azure que son propiedad de los clientes, o a los servicios de asociados, a través de un punto de conexión privado de la red virtual.|<p>[Creación de un punto de conexión privado](../private-link/create-private-endpoint-portal.md)</p><p>[Creación de un servicio Private Link](../private-link/create-private-link-service-portal.md)</p>|
|||
### <a name="ddos-protection"></a><a name="ddosprotection"></a>DDoS Protection 
[Azure DDoS Protection](../virtual-network/manage-ddos-protection.md) ofrece técnicas defensivas contra las amenazas de DDoS más sofisticadas. El servicio proporciona funcionalidades mejoradas de mitigación de DDoS a su aplicación y los recursos implementados en las redes virtuales. Además, los clientes que usan Azure DDoS Protection tienen acceso a la compatibilidad con la respuesta rápida de DDoS para atraer a expertos de DDoS durante un ataque activo.

![Protección contra DDOS](./media/networking-overview/ddos-protection.png)

### <a name="web-application-firewall"></a><a name="waf"></a>Firewall de aplicaciones web

[Azure Web Application Firewall](../web-application-firewall/overview.md) (WAF) de Azure proporciona protección a las aplicaciones web frente a vulnerabilidades de seguridad web comunes, como la inyección de SQL y el scripting entre sitios. WAF de Azure ofrece protección frente a las diez principales vulnerabilidades OWASP a través de reglas administradas. Además, los clientes también pueden configurar reglas personalizadas, que son reglas administradas por el cliente para proporcionar una protección adicional basada en el intervalo IP de origen, y atributos de solicitud como encabezados, cookies, campos de datos de formulario o parámetros de cadena de consulta.

Los clientes pueden optar por implementar [WAF de Azure con Application Gateway](../application-gateway/waf-overview.md), que proporciona protección regional para las entidades en el espacio de direcciones público y privado. Los clientes pueden optar por implementar [WAF de Azure con Front Door](../frontdoor/waf-overview.md), que proporciona protección en el perímetro de la red para los puntos de conexión públicos.

![Firewall de aplicaciones web](./media/networking-overview/waf-overview.png)


### <a name="azure-firewall"></a><a name="firewall"></a>Azure Firewall
Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Con Azure Firewall, puede crear, aplicar y registrar directivas de conectividad de red y de aplicaciones de forma centralizada en suscripciones y redes virtuales. Azure Firewall usa una dirección IP pública estática para los recursos de red virtual, que permite que los firewall externos identifiquen el tráfico procedente de la red virtual. 

Para más información acerca de Azure Firewall, consulte la [documentación de Azure Firewall](../firewall/overview.md).

![Información general de firewalls](./media/networking-overview/firewall-threat.png)

### <a name="network-security-groups"></a><a name="nsg"></a>Grupos de seguridad de red
Puede filtrar el tráfico de red hacia y desde los recursos de Azure en una red virtual de Azure con un grupo de seguridad de red. Para más información, consulte [Introducción a la seguridad](../virtual-network/security-overview.md).

### <a name="service-endpoints"></a><a name="serviceendpoints"></a>Puntos de conexión de servicio
Los puntos de conexión del servicio Virtual Network (red virtual) extienden el espacio de direcciones privadas de la red virtual y la identidad de la red virtual a los servicios de Azure a través de una conexión directa. Los puntos de conexión permiten proteger los recursos de servicio de Azure críticos únicamente para las redes virtuales. El tráfico desde la red virtual al servicio de Azure siempre permanece en la red troncal de Microsoft Azure. Para obtener más información, consulte [puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md).

![Puntos de conexión de servicio de red virtual](./media/networking-overview/vnet-service-endpoints-overview.png)

### <a name="azure-private-link"></a><a name="privatelink"></a>Azure Private Link
[Azure Private Link](../private-link/private-link-overview.md) permite acceder a los servicios PaaS de Azure (por ejemplo, Azure Storage y SQL Database) y a los servicios hospedados en Azure que son propiedad de los clientes, o a los servicios de asociados, a través de un punto de conexión privado de la red virtual.
El tráfico entre la red virtual y el servicio viaja por la red troncal de Microsoft. Ya no es necesario exponer el servicio a la red pública de Internet. Puede crear su propio servicio de vínculo privado en la red virtual y enviarlo a los clientes.

![Introducción al punto de conexión privado](./media/networking-overview/private-endpoint.png)


## <a name="application-delivery-services"></a><a name="deliver"></a>Servicios de entrega de aplicaciones

En esta sección se describen los servicios de red de Azure que ayudan a entregar aplicaciones: Content Delivery Network, Azure Front Door Service, Traffic Manager, Load Balancer y Application Gateway.

|Servicio|Motivos para la utilización|Escenario|
|---|---|---|
|[Content Delivery Network](#cdn)|Proporciona contenido con alto ancho de banda a los usuarios. Las redes CDN guardan el contenido almacenado en caché de los servidores perimetrales en ubicaciones de punto de presencia (POP) que están cerca de los usuarios finales, para minimizar la latencia.|<p>[Tutorial: Incorporación de CDN a una aplicación web](../cdn/cdn-add-to-web-app.md)</p> <p>[Tutorial: Acceso a blobs de almacenamiento mediante un dominio personalizado de Azure CDN a través de HTTPS](..//cdn/cdn-storage-custom-domain-https.md)</p> <p>[Tutorial: Incorporación de un dominio personalizado a un punto de conexión de Azure CDN](../cdn/cdn-map-content-to-custom-domain.md).</p> <p>[Configuración de HTTPS en un dominio personalizado de Azure CDN](../cdn/cdn-custom-ssl.md?tabs=option-1-default-enable-https-with-a-cdn-managed-certificate)</p>|
|[Azure Front Door Service](#frontdoor)|Permite definir, administrar y supervisar el enrutamiento global para el tráfico web gracias a la optimización para obtener el mejor rendimiento y la conmutación por error global instantánea para lograr una alta disponibilidad.|<p>[Tutorial: Incorporación de un dominio personalizado a Azure Front Door Service](../frontdoor/front-door-custom-domain.md)</p> <p>[Tutorial: Configuración de HTTPS en un dominio personalizado de Front Door](../frontdoor/front-door-custom-domain-https.md)</p><p>[Configuración de una directiva de filtrado geográfico para Front Door](../frontdoor/front-door-tutorial-geo-filtering.md)|
|[Traffic Manager](#trafficmanager)|Distribuye el tráfico basado en DNS a los servicios a través de las regiones globales de Azure, al mismo tiempo que ofrece una alta disponibilidad y capacidad de respuesta|<p> [Enrutamiento del tráfico para baja latencia](../traffic-manager/tutorial-traffic-manager-improve-website-response.md)</p><p>[Enrutamiento del tráfico a un punto de conexión prioritario](../traffic-manager/traffic-manager-configure-priority-routing-method.md)</p><p> [Control del tráfico con puntos de conexión ponderados](../traffic-manager/tutorial-traffic-manager-weighted-endpoint-routing.md)</p><p>[Enrutamiento del tráfico según la ubicación geográfica del punto de conexión](../traffic-manager/traffic-manager-configure-geographic-routing-method.md)</p> <p> [Enrutamiento del tráfico basado en la subred del usuario](../traffic-manager/tutorial-traffic-manager-subnet-routing.md)</p>|
|[Equilibrador de carga](#loadbalancer)|Proporciona equilibrio de carga regional mediante el enrutamiento del tráfico entre las zonas de disponibilidad y las redes virtuales. Proporciona equilibrio de carga interno mediante el enrutamiento del tráfico entre los recursos para crear una aplicación regional.|<p> [Equilibrio de carga del tráfico de Internet a las máquinas virtuales](../load-balancer/tutorial-load-balancer-standard-manage-portal.md)</p> <p>[Equilibrio de carga del tráfico a través de máquinas virtuales dentro de una red virtual](../load-balancer/tutorial-load-balancer-standard-internal-portal.md)<p>[Tráfico de reenvío de puertos a un puerto específico en máquinas virtuales concretas](../load-balancer/tutorial-load-balancer-port-forwarding-portal.md)</p><p> [Configuración del equilibrio de carga y las reglas de salida](../load-balancer/configure-load-balancer-outbound-cli.md)</p>|
|[Application Gateway](#applicationgateway)|Azure Application Gateway es un equilibrador de carga de tráfico web que permite administrar el tráfico a las aplicaciones web.|<p>[Inicio rápido: Dirección del tráfico web con Azure Application Gateway: Azure Portal](../application-gateway/quick-create-portal.md)</p><p>[Tutorial: Configuración de una puerta de enlace de aplicaciones con terminación TLS mediante Azure Portal](../application-gateway/create-ssl-portal.md)</p><p>[Crear una puerta de enlace de aplicaciones con redireccionamiento basado en rutas de dirección URL](../application-gateway/create-url-route-portal.md) </p>|
|

### <a name="content-delivery-network"></a><a name="cdn"></a>Content Delivery Network
Azure Content Delivery Network (CDN) ofrece a los desarrolladores una solución global para la entrega rápida de contenido con alto ancho de banda a los usuarios mediante el almacenamiento en caché del contenido en nodos físicos estratégicamente situados en todo el mundo. Para más información acerca de Azure CDN, consulte [Azure Content Delivery Network](../cdn/cdn-overview.md)

![Azure CDN](./media/networking-overview/cdn-overview.png)

### <a name="azure-front-door-service"></a><a name="frontdoor"></a>Azure Front Door Service
Azure Front Door Service permite definir, administrar y supervisar el enrutamiento global para el tráfico web mediante la optimización para obtener el mejor rendimiento y la conmutación por error global instantánea para alta disponibilidad. Con Front Door, las aplicaciones empresariales y de consumidor globales (de varias regiones) se pueden transformar en aplicaciones modernas personalizadas, sólidas y de alto rendimiento, API y contenido que lleguen a un público global mediante Azure. Para más información, consulte [Azure Front Door](../frontdoor/front-door-overview.md).


### <a name="traffic-manager"></a><a name="trafficmanager"></a>Traffic Manager

Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que le permite distribuir el tráfico de forma óptima a servicios de regiones de Azure globales, al tiempo que proporciona una alta disponibilidad y capacidad de respuesta. Traffic Manager proporciona diversos métodos de enrutamiento de tráfico para distribuir el tráfico, como la prioridad, la ponderación, el rendimiento, el método geográfico, el multivalor o la subred. Para más información sobre los métodos de enrutamiento del tráfico, consulte [Métodos de enrutamiento de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md).

En el diagrama siguiente se muestra el enrutamiento basado en la prioridad del punto de conexión con Traffic Manager:

![Método de enrutamiento de tráfico de "prioridad" de Azure Traffic Manager](./media/networking-overview/priority.png)

Para más información sobre Traffic Manager, consulte [¿Qué es Azure Traffic Manager?](../traffic-manager/traffic-manager-overview.md)

### <a name="load-balancer"></a><a name="loadbalancer"></a>Load Balancer
Azure Load Balancer proporciona equilibrio de carga de nivel 4 con latencia baja y rendimiento alto para todos los protocolos UDP y TCP. Administra conexiones entrantes y salientes. Puede configurar los puntos de conexión de carga equilibrada públicos e internos. Puede definir reglas para asignar conexiones entrantes a destinos de grupo de back-end con opciones de sondeo de estado TCP y HTTP para administrar la disponibilidad del servicio. Para más información sobre Load Balancer, consulte el artículo [Introducción a Load Balancer](../load-balancer/load-balancer-overview.md).

La siguiente imagen muestra una aplicación de niveles múltiples con conexión a Internet que utiliza los equilibradores de carga externos e internos:

![Ejemplo de Azure Load Balancer](./media/networking-overview/load-balancer.png)


### <a name="application-gateway"></a><a name="applicationgateway"></a>Application Gateway
Azure Application Gateway es un equilibrador de carga de tráfico web que permite administrar el tráfico a las aplicaciones web. Se trata de un controlador de entrega de aplicaciones (ADC) como servicio que ofrece diversas funcionalidades de equilibrio de carga de capa 7 para las aplicaciones. Para más información, consulte [¿Qué es Azure Application Gateway?](../application-gateway/overview.md).

En el diagrama siguiente se muestra el enrutamiento basado en la ruta de acceso de la dirección URL con Application Gateway.

![Ejemplo de Application Gateway](./media/networking-overview/figure1-720.png)

## <a name="network-monitoring-services"></a><a name="monitor"></a>Servicios de supervisión de red
En esta sección se describen los servicios de red de Azure que ayudan a supervisar los recursos de red: Network Watcher, Supervisión de ExpressRoute, Azure Monitor y TAP de red virtual.

|Servicio|Motivos para la utilización|Escenario|
|---|---|---|
|[Network Watcher](#networkwatcher)|Ayuda a supervisar y solucionar problemas de conectividad; a diagnosticar problemas de VPN, NSG y enrutamiento; y a capturar paquetes en la máquina virtual; y automatiza la activación de herramientas de diagnóstico mediante Azure Functions y Logic Apps|<p>[Inicio rápido: Diagnóstico de problemas al filtrar el tráfico de red de las máquinas virtuales con Azure Portal](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md)</p><p>[Tutorial: Diagnosticar un problema de enrutamiento de red de máquina virtual con Azure Portal](../network-watcher/diagnose-vm-network-routing-problem.md)</p><p>[Tutorial: Supervisión de la comunicación de red entre dos máquinas virtuales mediante Azure Portal](../network-watcher/connection-monitor.md)</p><p>[Diagnosticar problemas de comunicación entre redes](../network-watcher/diagnose-communication-problem-between-networks.md)</p><p>[Registro del tráfico de red hacia una máquina virtual y desde esta](../network-watcher/network-watcher-nsg-flow-logging-portal.md)</p>|
|[Supervisión de ExpressRoute](#expressroutemonitor)|Proporciona supervisión en tiempo real del rendimiento, la disponibilidad y el uso de la red; ayuda con la detección automática de la topología de red; proporciona un aislamiento más rápido de los errores; detecta problemas de red transitorios; ayuda a analizar las características del rendimiento histórico de la red; y admite varias suscripciones|<p>[Configuración de Network Performance Monitor para ExpressRoute](../expressroute/how-to-npm.md)</p><p>[Supervisión, métricas y alertas de ExpressRoute ](../expressroute/expressroute-monitoring-metrics-alerts.md)</p>|
|[Azure Monitor](#azuremonitor)|Ayuda a entender cómo funcionan las aplicaciones e identifica de manera proactiva los problemas que les afectan y los recursos de los que dependen.|<p>[Métricas y alertas de Traffic Manager](../traffic-manager/traffic-manager-metrics-alerts.md)</p><p>[Diagnóstico y métrica de Standard Load Balancer](../load-balancer/load-balancer-standard-diagnostics.md)</p><p>[Tutorial: Supervisión de métricas y registros de Azure Firewall](../firewall/tutorial-diagnostics.md)</p><p>[Supervisión y registro del Firewall de aplicaciones web de Azure](../frontdoor/waf-front-door-monitor.md)</p>|
|[TAP de red virtual](#vnettap)|Proporciona streaming continuo del tráfico de red de la máquina virtual al recopilador de paquetes, habilita las soluciones de administración del rendimiento de las aplicaciones y las redes y las herramientas de análisis de seguridad.|[Creación de un recurso de TAP de red virtual](../virtual-network/tutorial-tap-virtual-network-cli.md)|
|

### <a name="network-watcher"></a><a name="networkwatcher"></a>Network Watcher
Azure Network Watcher proporciona herramientas para supervisar, diagnosticar, ver las métricas y habilitar o deshabilitar registros de recursos en una red virtual de Azure. Para más información, consulte [¿Qué es Azure Network Watcher?](../network-watcher/network-watcher-monitoring-overview.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="expressroute-monitor"></a><a name="expressroutemonitor"></a>Supervisión de ExpressRoute
Para información sobre cómo ver las métricas de los circuitos ExpressRoute, los registros de recursos y las alertas, consulte [Supervisión, métricas y alertas de ExpressRoute](../expressroute/expressroute-monitoring-metrics-alerts.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="azure-monitor"></a><a name="azuremonitor"></a>Azure Monitor
Azure Monitor maximiza la disponibilidad y el rendimiento de las aplicaciones con una completa solución que permite recopilar, analizar y administrar datos telemétricos tanto en la nube como en entornos locales. Esta solución le ayudará a entender cómo funcionan las aplicaciones y le permitirá identificar de manera proactiva los problemas que les afectan y los recursos de los que dependen. Para más información, consulte [Introducción a Azure Monitor](../azure-monitor/overview.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="virtual-network-tap"></a><a name="vnettap"></a>TAP de red virtual
Azure Virtual Network TAP (punto de acceso del terminal) permite transmitir continuamente el tráfico de red de la máquina virtual a un recopilador de paquetes de red o a una herramienta de análisis. Un asociado de la [aplicación virtual de red](https://azure.microsoft.com/solutions/network-appliances/) proporciona el recopilador o la herramienta de análisis de la herramienta. 

En la siguiente imagen se muestra cómo funciona Virtual Network TAP. 

![Cómo funciona Virtual Network TAP](./media/networking-overview/virtual-network-tap-architecture.png)

Para más información, consulte [TAP de red virtual](../virtual-network/virtual-network-tap-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Cree su primera red virtual y conecte algunas máquinas virtuales a ella, siguiendo los pasos descritos en el artículo [Creación de su primera red virtual](../virtual-network/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
- Conecte su equipo a una red virtual siguiendo los pasos descritos en el artículo [Configuración de una conexión de punto a sitio](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
- Equilibre la carga de tráfico de Internet en los servidores públicos siguiendo los pasos descritos en el artículo [Creación de un equilibrador de carga con conexión a Internet](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
 
 
   
