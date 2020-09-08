---
title: Información general Azure Virtual WAN | Microsoft Docs
description: Obtenga información acerca de la conectividad entre ramas escalable y automatizada de rama a rama de Virtual WAN, las regiones en que está disponible y sus asociados.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: overview
ms.date: 08/18/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to understand what Virtual WAN is and if it is the right choice for my Azure network.
ms.openlocfilehash: 2d8ac3775021c574d3c03e44b06d6b5e689070e7
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852661"
---
# <a name="what-is-azure-virtual-wan"></a>¿Qué es Azure Virtual WAN?

Azure Virtual WAN es un servicio de red que aporta muchas funciones de red, seguridad y enrutamiento para proporcionar una única interfaz operativa. Estas funcionalidades incluyen la conectividad de ramas (mediante la automatización de la conectividad desde dispositivos de asociados de Virtual WAN, como SD-WAN o VPN CPE), la conectividad VPN de sitio a sitio, la conectividad VPN de usuarios remotos (de punto a sitio), la conectividad privada (ExpressRoute), la conectividad interna de la nube (conectividad transitiva para redes virtuales), la interconectividad de VPN ExpressRoute, el enrutamiento, Azure Firewall y el cifrado para la conectividad privada. No es necesario disponer de todos estos casos de uso para empezar a usar Virtual WAN. Puede empezar a trabajar con un solo caso de uso y, posteriormente, ajustar la red a medida que crezca.

La arquitectura de Virtual WAN es del tipo hub-and-spoke y cuenta con escalado y rendimiento integrados en las ramas (dispositivos VPN y SD-WAN), los usuarios (clientes de VPN de Azure, OpenVPN e IKEv2), los circuitos de ExpressRoute y las redes virtuales. Permite la [arquitectura de red de tránsito global](virtual-wan-global-transit-network-architecture.md) donde el "centro de conectividad" de la red que se hospeda en la nube permite la conectividad transitiva entre puntos de conexión que pueden estar distribuidos en distintos tipos de "radios".

Las regiones de Azure sirven como concentradores que se pueden elegir para conectar. Todos los centros de conectividad están conectados en una malla completa en una red WAN virtual estándar, lo que facilita al usuario el uso de la red troncal de Microsoft para la conectividad de cualquier tipo (cualquier radio). En el caso de la topología radial con dispositivos SD-WAN y VPN, los usuarios pueden configurarla manualmente en Azure Virtual WAN, o bien usar la solución de asociado CPE de Virtual WAN (SD-WAN/VPN) para configurar la conectividad con Azure. Tenemos una lista de asociados que admiten la automatización de la conectividad (la capacidad de exportar la información del dispositivo a Azure, descargar la configuración de Azure y establecer conectividad) con Azure Virtual WAN. Para más información, consulte el artículo sobre [los asociados y las ubicaciones de Virtual WAN](virtual-wan-locations-partners.md).

![Diagrama de Virtual WAN](./media/virtual-wan-about/virtualwan1.png)

En este artículo se proporciona una vista rápida de la conectividad de red en Azure Virtual WAN. Virtual WAN ofrece las siguientes ventajas:

* **Soluciones de conectividad integrada en una topología en estrella tipo hub-and-stroke:** Configuración de sitio a sitio automatizada y conectividad entre sitios locales y un centro de Azure.
* **Instalación y configuración automatizadas de radio:** conecte sin problemas las cargas de trabajo y las redes virtuales al centro de Azure.
* **Solución intuitiva de problemas:** puede ver el flujo de un extremo a otro dentro de Azure y usar esta información para realizar las acciones necesarias.

## <a name="basic-and-standard-virtual-wans"></a><a name="basicstandard"></a>Redes WAN virtuales de tipo Básico y Estándar

Existen dos tipos de WAN virtuales: Básico y Estándar. En la tabla siguiente se muestran las configuraciones disponibles para cada tipo.

[!INCLUDE [Basic and Standard SKUs](../../includes/virtual-wan-standard-basic-include.md)]

Para conocer los pasos para actualizar una WAN virtual, consulte [Actualización de una WAN virtual de plan Básico a Estándar](upgrade-virtual-wan.md).

## <a name="architecture"></a><a name="architecture"></a>Arquitectura

Para obtener información acerca de la arquitectura de Virtual WAN y de cómo migrar a Virtual WAN, consulte los siguientes artículos:

* [Arquitectura de Virtual WAN](migrate-from-hub-spoke-topology.md)
* [Arquitectura de red de tránsito global](virtual-wan-global-transit-network-architecture.md)

## <a name="virtual-wan-resources"></a><a name="resources"></a>Recursos de Virtual WAN

Para configurar una red Virtual WAN de extremo a otro, debe crear los siguientes recursos:

* **virtualWAN:** El recurso virtualWAN representa una superposición virtual de la red de Azure y es una colección de varios recursos. Contiene vínculos a todos los concentradores virtuales que le gustaría tener dentro de Virtual WAN. Los recursos de Virtual WAN están aislados entre sí y no pueden contener un concentrador común. Los centros de conectividad virtuales mediante Virtual WAN no se comunican entre sí.

* **Centro:** Un concentrador virtual es una red virtual administrada por Microsoft. El concentrador contiene varios puntos de conexión de servicio para habilitar la conectividad. Desde la red local (vpnsite), puede conectarse a una instancia de VPN Gateway dentro del centro de conectividad virtual, conectar circuitos ExpressRoute a un centro de conectividad virtual o incluso conectar usuarios móviles a una puerta de enlace de punto a sitio en el centro de conectividad virtual. El concentrador es el núcleo de la red en una región. Solo puede haber un único concentrador por cada región de Azure.

  Una puerta de enlace de concentrador no es lo mismo que la puerta de enlace de red virtual que usó para ExpressRoute y VPN Gateway. Por ejemplo, cuando se usa Virtual WAN, no se crea una conexión de sitio a sitio desde un sitio local directamente a una red virtual. En su lugar, se crea una conexión de sitio a sitio en el centro. El tráfico siempre pasa a través de la puerta de enlace del concentrador. Esto significa que las redes virtuales no necesitan su propia puerta de enlace de red virtual. Virtual WAN permite que las redes virtuales obtengan provecho del escalado fácilmente mediante el concentrador virtual y su puerta de enlace.

* **Conexión de red virtual del centro:** el recurso de conexión de red virtual de centro se usa para conectar el centro sin problemas a una red virtual.

* **Conexión centro a centro:** los centros están conectados entre sí en una WAN virtual. Esto implica que una rama, un usuario o una red virtual conectados a un concentrador local pueden comunicarse con otra rama o red virtual mediante la arquitectura de malla completa de los concentradores conectados. También puede conectar redes virtuales dentro de un centro de conectividad virtual en tránsito mediante el concentrador virtual, así como redes virtuales a través del concentrador, con el marco conectado de concentrador a concentrador.

* **Tabla de rutas de centro:** puede crear una ruta del centro virtual y aplicarla a la tabla de rutas del centro virtual. Puede aplicar varias rutas a la tabla de rutas del concentrador virtual.

**Recursos adicionales de Virtual WAN**

* **Sitio:** este recurso se usa solo para las conexiones de sitio a sitio. El recurso del sitio es **vpnsite**. Representa el dispositivo VPN local y su configuración. Al trabajar con un asociado de Virtual WAN, tiene una solución integrada para exportar automáticamente esta información a Azure.

## <a name="types-of-connectivity"></a><a name="connectivity"></a>Tipos de conectividad

Virtual WAN permite dos tipos de conectividad: VPN de sitio a sitio, VPN de usuario (de punto a sitio) y ExpressRoute.

### <a name="site-to-site-vpn-connections"></a><a name="s2s"></a>Conexión VPN de sitio a sitio

Puede conectarse a los recursos de Azure por medio de una conexión IPsec/IKE (IKEv2) de sitio a sitio. Para más información, consulte [Creación de una conexión de sitio a sitio mediante Virtual WAN](virtual-wan-site-to-site-portal.md). 

Este tipo de conexión requiere un dispositivo VPN o un dispositivo de asociado de Virtual WAN. Los asociados de Virtual WAN proporcionan automatización de la conectividad, que es la capacidad de exportar la información del dispositivo a Azure, descargar la configuración de Azure y establecer conectividad con el centro de conectividad de Azure Virtual WAN. Para ver una lista de los asociados y las ubicaciones disponibles, consulte el artículo acerca [asociados y ubicaciones de Virtual WAN](virtual-wan-locations-partners.md). Si el proveedor de dispositivos VPN/SD-WAN no aparece en el vínculo mencionado, puede simplificar la operación mediante el uso de las instrucciones detalladas que se proporcionan en [Creación de una conexión de sitio a sitio mediante Azure Virtual WAN](virtual-wan-site-to-site-portal.md) para configurar la conexión.

### <a name="user-vpn-point-to-site-connections"></a><a name="uservpn"></a>Conexiones de VPN de usuario (de punto a sitio)

Puede conectarse a los recursos en Azure por medio de una conexión de IPsec/IKE (IKEv2) u OpenVPN. Este tipo de conexión requiere que se configure un cliente VPN en el equipo cliente. Para más información, consulte [Creación de una conexión de punto a sitio](virtual-wan-point-to-site-portal.md).

### <a name="expressroute-connections"></a><a name="er"></a>Conexiones de ExpressRoute
ExpressRoute permite conectar una red local a Azure mediante una conexión privada. Para crear la conexión, consulte [Create an ExpressRoute connection using Virtual WAN](virtual-wan-expressroute-portal.md) (Creación de una conexión de ExpressRoute mediante Virtual WAN).

### <a name="hub-to-vnet-connections"></a><a name="hub"></a>Conexiones de centro de conectividad a red virtual

Conexión de una red virtual de Azure a un centro de conectividad virtual. Para más información, consulte [Conexión de una red virtual a un centro de conectividad](virtual-wan-site-to-site-portal.md#vnet).

### <a name="transit-connectivity"></a><a name="transit"></a>Conectividad de tránsito

#### <a name="transit-connectivity-between-vnets"></a><a name="transit-vnet"></a>Conectividad de tránsito entre redes virtuales

Virtual WAN permite la conectividad de tránsito entre redes virtuales. Las redes virtuales se conectan a un centro virtual a través de una conexión de red virtual. La conectividad de tránsito entre las redes virtuales de **Virtual WAN Estándar** se habilita debido a la presencia de un enrutador en cada centro virtual. Se crea una instancia de este enrutador la primera vez que se crea el centro virtual.

El enrutador puede tener cuatro estados de enrutamiento: Provisioned, Provisioning, Failed o None. El **estado del enrutamiento** se encuentra en Azure Portal. Solo hay que ir a la página del centro virtual.

* El estado **None** indica que el centro virtual no ha aprovisionado el enrutador. Esto puede ocurrir si la instancia de Virtual WAN es del tipo *Básico* o si el centro virtual se ha implementado antes de que el servicio estuviera disponible.
* El estado **Failed** indica un error durante la creación de instancias. Para crear una instancia del enrutador o restablecerlo, puede encontrar la opción **Reset Router** (Restablecer enrutador) en la página de información general del centro virtual en Azure Portal.

Todos los enrutadores del centro virtual admiten un rendimiento agregado de hasta 50 Gbps. La conectividad entre las conexiones de red virtual asume un carga de trabajo total de 2000 máquinas virtuales en todas las redes virtuales conectadas a un único centro virtual.

#### <a name="transit-connectivity-between-vpn-and-expressroute"></a><a name="transit-er"></a>Conectividad de tránsito entre VPN y ExpressRoute

Virtual WAN permite la conectividad de tránsito entre VPN y ExpressRoute. Esto implica que los sitios conectados mediante VPN o los usuarios remotos pueden comunicarse con sitios conectados mediante ExpressRoute. También hay una asunción implícita de que la **marca rama a rama** está habilitada y que el protocolo BGP se admite en las conexiones de VPN y ExpressRoute. Esta marca se puede encontrar en la configuración de Azure Virtual WAN en Azure Portal. La administración de todas las rutas la proporciona el enrutador del centro virtual, que también permite la conectividad de tránsito entre las redes virtuales.

### <a name="custom-routing"></a><a name="routing"></a>Enrutamiento personalizado

Virtual WAN proporciona mejoras de enrutamiento avanzadas. La capacidad para configurar tablas de rutas personalizadas, optimizar el enrutamiento de redes virtuales con la asociación y propagación de rutas, agrupar lógicamente tablas de rutas con etiquetas y simplificar numerosos escenarios de enrutamiento de dispositivos virtuales de red (NVA) o de servicios compartidos.

### <a name="global-vnet-peering"></a><a name="global"></a>Emparejamiento de red virtual global

El emparejamiento de red virtual global proporciona un mecanismo para conectar dos redes virtuales de diferentes regiones. En Virtual WAN, las conexiones de red virtual conectan redes virtuales a centros virtuales. El usuario no necesita configurar el emparejamiento global de redes virtuales de forma explícita. Las redes virtuales conectadas a un centro virtual en la misma región incurren en cargos de emparejamiento de red virtual. Las redes virtuales conectadas a un centro virtual de otra región incurren en cargos de emparejamiento de red virtual global.

### <a name="expressroute-traffic-encryption"></a><a name="encryption"></a>Cifrado del tráfico de ExpressRoute

Azure Virtual WAN proporciona la capacidad de cifrar el tráfico de ExpressRoute. La técnica proporciona un tránsito cifrado entre las redes locales y las redes virtuales de Azure a través de ExpressRoute, sin necesidad de pasar por la red pública de Internet ni utilizar direcciones IP públicas. Para más información, consulte [IPsec sobre ExpressRoute para Virtual WAN](vpn-over-expressroute.md).

## <a name="locations"></a><a name="locations"></a>Ubicaciones

Para obtener información de las ubicaciones, consulte el de ubicación, consulte el artículo acerca de los [asociados y ubicaciones de Virtual WAN](virtual-wan-locations-partners.md).

## <a name="route-tables-for-basic-and-standard-virtual-wans"></a><a name="route"></a>Tablas de rutas para las versiones Básico y Estándar de Virtual WAN

Las tablas de rutas ahora tienen características para la asociación y propagación. Una tabla de rutas preexistente es una tabla de enrutamiento que no tiene estas características. Si tiene rutas preexistentes en el enrutamiento del centro y desea usar las nuevas funcionalidades, tenga en cuenta lo siguiente:

* **Clientes de la versión Estándar de Virtual WAN con rutas preexistentes en un centro virtual**: Si tiene rutas preexistentes en la sección Enrutamiento del centro de Azure Portal, primero debe eliminarlas y, luego, intentar crear tablas de rutas nuevas (disponibles en la sección Tablas de rutas del centro de Azure Portal).

* **Clientes de la versión Básica de Virtual WAN con rutas preexistentes en un centro virtual**: Si tiene rutas preexistentes en la sección Enrutamiento del centro de Azure Portal, primero debe eliminarlas y, después, **actualizar** la versión Básica de Virtual WAN a la versión Estándar. Consulte [Actualización de una virtual WAN de Básica a Estándar](upgrade-virtual-wan.md).

## <a name="faq"></a><a name="faq"></a>P+F

[!INCLUDE [Virtual WAN FAQ](../../includes/virtual-wan-faq-include.md)]

## <a name="whats-new"></a><a name="new"></a>Novedades

Suscríbase a la fuente RSS y vea las actualizaciones más recientes de las características de Virtual WAN en la página [Actualizaciones de Azure](https://azure.microsoft.com/updates/?category=networking&query=VIRTUAL%20WAN).

## <a name="next-steps"></a>Pasos siguientes

[Creación de una conexión de sitio a sitio mediante Virtual WAN](virtual-wan-site-to-site-portal.md)
