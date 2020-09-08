---
title: 'Introducción a Azure ExpressRoute: Conexión mediante una conexión privada'
description: La introducción técnica sobre ExpressRoute explica cómo funciona una conexión de ExpressRoute a la hora de ampliar la red local a Azure mediante una conexión privada.
services: expressroute
author: mialdrid
ms.service: expressroute
ms.topic: overview
ms.date: 08/25/2020
ms.author: mialdrid
ms.openlocfilehash: 6621a9778025d67046722b7cd60e32a62b574ee5
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852941"
---
# <a name="what-is-azure-expressroute"></a>¿Qué es Azure ExpressRoute?
ExpressRoute le permite ampliar sus redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad. Con ExpressRoute, se pueden establecer conexiones con servicios en la nube de Microsoft, como Microsoft Azure y Office 365.

La conectividad puede ser desde una red de conectividad universal (IP VPN), una red Ethernet de punto a punto, o una conexión cruzada virtual a través de un proveedor de conectividad en una instalación de ubicación compartida. Las conexiones ExpressRoute no pasan por la red pública de Internet. Esto permite a las conexiones de ExpressRoute ofrecer más confiabilidad, más velocidad, latencia coherentes y mayor seguridad que las conexiones normales a través de Internet. Para información sobre cómo conectar la red a Microsoft mediante ExpressRoute, consulte [ExpressRoute connectivity models](expressroute-connectivity-models.md) (Modelos de conectividad de ExpressRoute).

![Introducción a la conexión de ExpressRoute](./media/expressroute-introduction/expressroute-connection-overview.png)

## <a name="key-benefits"></a>Ventajas principales

* Conectividad de nivel 3 entre su red local y Microsoft Cloud a través de un proveedor de conectividad. La conectividad puede ser desde una red de conectividad universal (IP VPN), una red Ethernet de punto a punto, o una conexión cruzada virtual a través de un intercambio de Ethernet.
* Conectividad de servicios en la nube de Microsoft en todas las regiones dentro de la región geopolítica.
* Conectividad global a los servicios de Microsoft en todas las regiones con el complemento ExpressRoute Premium.
* Enrutamiento dinámico entre la red y Microsoft a través de BGP.
* Redundancia integrada en todas las ubicaciones de configuración entre pares para una mayor confiabilidad.
* El tiempo de actividad de conexión [SLA](https://azure.microsoft.com/support/legal/sla/).
* Compatibilidad con QoS de Skype para la empresa.

Para más información, consulte [P+F de ExpressRoute](expressroute-faqs.md).

## <a name="features"></a>Características

### <a name="layer-3-connectivity"></a>Conectividad de nivel 3
Microsoft usa BGP, un protocolo de enrutamiento dinámico estándar del sector, para intercambiar las rutas entre su red local, las instancias de Azure y las direcciones públicas de Microsoft. Establecemos varias sesiones BGP con su red para perfiles de tráfico diferentes. Puede encontrar más información en el artículo [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md) .

### <a name="redundancy"></a>Redundancia
Cada circuito ExpressRoute consta de dos conexiones a dos enrutadores perimetrales de Microsoft Enterprise (MSEE) de una [ubicación de ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-locations#expressroute-locations) desde el proveedor de conectividad o el perímetro de la red. Microsoft requiere una conexión BGP dual desde el proveedor de conectividad o el perímetro de la red, uno a cada MSEE. Si lo desea, tiene la opción de no implementar dispositivos redundantes o circuitos Ethernet en el extremo. Sin embargo, los proveedores de conectividad usan dispositivos redundantes para garantizar que las conexiones se entregan a Microsoft de forma redundante. Una configuración de conectividad redundante de nivel 3 es un requisito para nuestro [SLA](https://azure.microsoft.com/support/legal/sla/) sea válido.

### <a name="connectivity-to-microsoft-cloud-services"></a>Conectividad con los Servicios en la nube de Microsoft
Las conexiones ExpressRoute habilitan el acceso a los servicios siguientes:
* Servicios de Microsoft Azure
* Servicios de Microsoft Office 365

> [!NOTE]
> [!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]
> 

También puede visitar la página [Preguntas más frecuentes de ExpressRoute](expressroute-faqs.md) para obtener una lista detallada de servicios admitidos a través de ExpressRoute.

### <a name="connectivity-to-all-regions-within-a-geopolitical-region"></a>Conectividad a todas las regiones dentro de una región geopolítica
Puede conectarse a Microsoft en una de nuestras [ubicaciones de configuración entre pares](expressroute-locations.md) y acceder a las regiones dentro de la región geopolítica.

Por ejemplo, si se conectó a Microsoft en Ámsterdam mediante ExpressRoute, tendrá acceso a todos los servicios en la nube de Microsoft hospedados en Europa Septentrional y Europa Occidental. Para obtener información general sobre las regiones geopolíticas, las regiones de Microsoft Cloud asociadas y las correspondientes ubicaciones de emparejamiento de ExpressRoute, consulte [Asociados de ExpressRoute y ubicaciones de emparejamiento](expressroute-locations.md).

### <a name="global-connectivity-with-expressroute-premium"></a>Conectividad global con ExpressRoute Premium
Puede habilitar [ExpressRoute Premium](expressroute-faqs.md) para extender la conectividad a través de límites geopolíticos. Por ejemplo, si se conectó a Microsoft en Ámsterdam con ExpressRoute, tendrá acceso a todos los servicios en la nube de Microsoft hospedados en todas las regiones del mundo (excluidas las nubes nacionales). Puede acceder a los servicios implementados en Sudamérica o Australia del mismo modo que accede a las regiones de Europa Septentrional y Oeste de Europa.

### <a name="local-connectivity-with-expressroute-local"></a>Conectividad local con ExpressRoute Local
Para transferir datos de manera rentable, habilite el [SKU local](expressroute-faqs.md) si puede traer los datos a una ubicación de ExpressRoute cerca de la región de Azure deseada. Con Local, la transferencia de datos se incluye en la carga del puerto de ExpressRoute. 

### <a name="across-on-premises-connectivity-with-expressroute-global-reach"></a>Conectividad local con Global Reach de ExpressRoute
Puede permitir que Global Reach de ExpressRoute intercambie datos entre los sitios locales si conecta los diferentes circuitos ExpressRoute. Por ejemplo, si tiene un centro de datos privado en California conectado a ExpressRoute en Silicon Valley, y otro centro de datos privado en Texas conectado a ExpressRoute en Dallas, con Global Reach de ExpressRoute, puede conectar los centros de datos privados entre sí mediante dos circuitos ExpressRoute. El tráfico del centro de datos pasará por la red de Microsoft.

Para obtener más información, consulte [Global Reach de ExpressRoute](expressroute-global-reach.md).
### <a name="rich-connectivity-partner-ecosystem"></a>Ecosistema de socios de conectividad enriquecida
ExpressRoute tiene un ecosistema de proveedores de conectividad y asociados integradores de sistemas en constante crecimiento. Para la información más reciente, consulte [Asociados de ExpressRoute y ubicaciones de emparejamiento](expressroute-locations.md).

### <a name="connectivity-to-national-clouds"></a>Conectividad con nubes nacionales
Microsoft administra entornos de nube aislados para regiones geopolíticas y segmentos de clientes especiales. Consulte la página [Asociados de ExpressRoute y ubicaciones de emparejamiento](expressroute-locations.md) para obtener una lista de nubes y proveedores nacionales.

### <a name="expressroute-direct"></a>ExpressRoute Direct
ExpressRoute Direct ofrece a los clientes la oportunidad de conectarse directamente a la red global de Microsoft en ubicaciones de emparejamiento distribuidas estratégicamente por todo el mundo. ExpressRoute Direct proporciona conectividad dual de 100 Gbps, que es compatible con la conectividad activa/activa a escala.

Algunas de las características clave que ofrece ExpressRoute Direct incluyen:

* Ingesta de datos masivos en servicios como Storage y Cosmos DB
* Aislamiento físico para sectores regulados y que necesitan una conectividad dedicada y aislada, como: banca, gubernamentales y venta al por menor
* Control granular del circuito de distribución en función de la unidad de negocio

Para obtener más información, consulte [Acerca de ExpressRoute Direct](https://go.microsoft.com/fwlink/?linkid=2022973).

### <a name="bandwidth-options"></a>Opciones de ancho de banda
Puede comprar los circuitos ExpressRoute para una amplia gama de anchos de banda. A continuación se enumera la lista de anchos de banda admitidos. Asegúrese de consultar a su proveedor de conectividad para determinar la lista de anchos de banda admitidos que proporcionan.

* 50 Mbps
* 100 Mbps
* 200 Mbps
* 500 Mbps
* 1 Gbps
* 2 Gbps
* 5 Gbps
* 10 Gbps

### <a name="dynamic-scaling-of-bandwidth"></a>Escalado dinámico del ancho de banda
Puede aumentar el ancho de banda del circuito ExpressRoute (dentro de lo posible) sin necesidad de eliminar sus conexiones. Para más información, consulte [Modificación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md#modify).

### <a name="flexible-billing-models"></a>Modelos flexibles de facturación
Puede elegir el modelo de facturación que mejor le convenga. Elija entre los modelos de facturación enumerados a continuación. Para más información, consulte el artículo sobre [Preguntas más frecuentes de ExpressRoute](expressroute-faqs.md).

* **Datos ilimitados**. La facturación se basa en unos honorarios mensuales y todas las transferencias de datos de entrada y de salida están incluidas de forma gratuita.
* **Datos medidos**. La facturación se basa en unos honorarios mensuales y todas las transferencias de datos de entrada están incluidas de forma gratuita. Las transferencias de datos de salida se cobran de acuerdo a un precio por GB de transferencia de datos. Las tasas de transferencia de datos varían según la región.
* **Complemento ExpressRoute Premium**. ExpressRoute Premium es un complemento que se agrega a un circuito ExpressRoute. El complemento ExpressRoute Premium ofrece las siguientes capacidades: 
  * Límites de ruta ampliados para las configuraciones entre pares públicos y privados de Azure, de 4.000 rutas a 10.000 rutas.
  * Conectividad global para los servicios. Un circuito ExpressRoute creado en cualquier región (excepto las nubes nacionales) tendrá acceso a los recurso de cualquier otra región del mundo. Por ejemplo, un circuito ExpressRoute aprovisionado en Silicon Valley puede acceder a una red virtual creada en Oeste de Europa .
  * Número de vínculos de red virtual por circuito ExpressRoute ampliado de 10 a un límite superior, que depende del ancho de banda del circuito.

## <a name="faq"></a>Preguntas más frecuentes
Para ver las preguntas más frecuentes sobre ExpressRoute, consulte [Preguntas más frecuentes de ExpressRoute](expressroute-faqs.md).

## <a name="whats-new"></a><a name="new"></a>Novedades

Suscríbase a la fuente RSS y consulte las actualizaciones más recientes de las características de ExpressRoute en la página [Actualizaciones de Azure](https://azure.microsoft.com/updates/?category=networking&query=ExpressRoute).

## <a name="next-steps"></a>Pasos siguientes
* Información acerca de los [Modelos de conectividad de ExpressRoute](expressroute-connectivity-models.md).
* Obtenga información acerca de las conexiones ExpressRoute y dominios de enrutamiento. Consulte [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md).
* Busque un proveedor de servicios. Consulte [Asociados de ExpressRoute de Azure y ubicaciones de emparejamiento](expressroute-locations.md).
* Asegúrese de que se cumplen todos los requisitos previos. Consulte [Requisitos previos de ExpressRoute](expressroute-prerequisites.md).
* Consulte los requisitos de [enrutamiento](expressroute-routing.md), [NAT](expressroute-nat.md) y [QoS](expressroute-qos.md).
* Configure su conexión ExpressRoute.
  * [Creación y modificación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
  * [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
  * [Conexión de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
* Aprenda sobre las demás [funcionalidades de red](../networking/networking-overview.md) clave de Azure.
