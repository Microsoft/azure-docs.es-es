---
title: Acerca de Azure VPN Gateway
description: Aprenda qué es VPN Gateway y cómo se usa para conectarse a redes virtuales de sitio a sitio de IPsec/IKE, de red virtual a red virtual y de VPN de punto a sitio.
services: vpn-gateway
author: cherylmc
Customer intent: As someone with a basic network background, but is new to Azure, I want to understand the capabilities of Azure VPN Gateway so that I can securely connect to my Azure virtual networks.
ms.service: vpn-gateway
ms.topic: overview
ms.date: 08/27/2020
ms.author: cherylmc
ms.custom: contperfq1
ms.openlocfilehash: 23d8d28a03217b1359462332da736f852cfaf8ea
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89015397"
---
# <a name="what-is-vpn-gateway"></a>¿Qué es VPN Gateway?

VPN Gateway es un tipo específico de puerta de enlace de red virtual que se usa para enviar tráfico cifrado entre una red virtual de Azure y una ubicación local a través de la red pública de Internet. También puede usar una instancia de VPN Gateway para enviar tráfico cifrado entre las redes virtuales de Azure a través de la red de Microsoft. Cada red virtual solo puede tener una instancia de VPN Gateway. Sin embargo, puede crear varias conexiones a la misma instancia. Al crear varias conexiones a la misma instancia de VPN Gateway, todos los túneles VPN comparten el ancho de banda disponible de la puerta de enlace.

## <a name="what-is-a-virtual-network-gateway"></a><a name="whatis"></a>¿Qué es una puerta de enlace de red virtual?

Una puerta de enlace de red virtual se compone de dos o más máquinas virtuales que se implementan en una subred específica llamada *subred de la puerta de enlace*. Las máquinas virtuales de puerta de enlace de red virtual contienen tablas de enrutamiento y ejecutan servicios específicos de puerta de enlace. Estas máquinas virtuales se crean al generar la puerta de enlace de red virtual. No se pueden configurar directamente las máquinas virtuales que forman parte de la puerta de enlace de red virtual.

Al configurar una puerta de enlace de red virtual, se configura un valor que especifica el tipo de puerta de enlace. El tipo de puerta de enlace especifica cómo se utilizará la puerta de enlace de red virtual y las acciones que realiza la puerta de enlace. El tipo de puerta de enlace "Vpn" especifica que el tipo de puerta de enlace de red virtual creado es una "puerta de enlace de VPN". Esto lo distingue de una puerta de enlace de ExpressRoute, que usa un tipo de puerta de enlace diferente. Una red virtual puede tener dos puertas de enlace de red virtual, una puerta de enlace de VPN y una puerta de enlace de ExpressRoute. Para más información, consulte [Tipos de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwtype).

La creación de una puerta de enlace de red virtual puede tardar en completarse hasta 45 minutos. Al crear una puerta de enlace de red virtual, las máquinas virtuales de puerta de enlace se implementan en la subred de puerta de enlace y se configuran con las opciones que especifique. Después de crear una instancia de VPN Gateway, puede crear una conexión de túnel de VPN de IPsec o IKE entre esa instancia y otra instancia de VPN Gateway (de red virtual a red virtual), o crear una conexión de túnel de VPN de IPsec o IKE con implementaciones locales entre la instancia de VPN Gateway y un dispositivo VPN local (de sitio a sitio). También puede crear una conexión VPN de punto a sitio (VPN a través de OpenVPN, IKEv2 o SSTP) que le permite conectarse a la red virtual desde una ubicación remota como, por ejemplo, una sala de conferencias o desde su casa.

## <a name="configuring-a-vpn-gateway"></a><a name="configuring"></a>Configuración de una instancia de VPN Gateway

Una conexión de puerta de enlace de VPN se basa en varios recursos con una configuración específica. La mayoría de los recursos puede configurarse por separado, aunque en algunos casos es necesario seguir un orden determinado.

### <a name="design"></a><a name="diagrams"></a>Diseño

Es importante saber que hay distintas configuraciones disponibles para las conexiones de VPN Gateway. Es preciso determinar qué configuración es la que mejor se adapta a sus necesidades. Por ejemplo, las conexiones de punto a sitio, de sitio a sitio y de ExpressRoute o de sitio a sitio coexistentes tienen instrucciones y requisitos de configuración diferentes. Para obtener información sobre el diseño y para ver los diagramas de topología de conexión, consulte [Diseño](design.md).

### <a name="planning-table"></a><a name="planningtable"></a>Tabla de planeación

La tabla siguiente puede ayudarle a decidir la mejor opción de conectividad para su solución.

[!INCLUDE [cross-premises](../../includes/vpn-gateway-cross-premises-include.md)]

### <a name="settings"></a><a name="settings"></a>Configuración

La configuración que ha elegido para cada recurso es fundamental para crear una conexión correcta. Para más información sobre los recursos individuales y la configuración de VPN Gateway, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md). El artículo contiene información que le ayudará a entender los tipos de puerta de enlace, de SKU de puerta de enlace, de VPN y de conexión, así como las subredes de puerta de enlace, las puertas de enlace de red local y otras configuraciones de recursos que pueden interesarle.

### <a name="deployment-tools"></a><a name="tools"></a>Herramientas de implementación

Puede empezar a crear y configurar recursos mediante una herramienta de configuración, como el portal de Azure. Más adelante puede decidir cambiar a otra herramienta, como PowerShell, para configurar recursos adicionales o para modificar los existentes cuando sea aplicable. Actualmente, no se pueden configurar todos los recursos ni establecer todas las configuraciones de recurso en Azure Portal. Las instrucciones de los artículos para cada topología de configuración indican cuándo se necesita una herramienta de configuración específica.

## <a name="gateway-skus"></a><a name="gwsku"></a>SKU de puerta de enlace

Al crear una puerta de enlace de red virtual, hay que especificar la SKU de la puerta de enlace que desea usar. Seleccione las SKU que cumplan sus requisitos en función de los tipos de cargas de trabajo, rendimientos, características y Acuerdos de Nivel de Servicio.

* Para más información acerca de las SKU de puerta de enlace, incluidas las características admitidas, los pasos de producción, desarrollo-prueba y configuración, consulte el artículo [Configuración de VPN Gateway: SKU de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku).
* Para más información sobre las SKU heredadas, consulte [Trabajo con SKU heredadas](vpn-gateway-about-skus-legacy.md).

### <a name="gateway-skus-by-tunnel-connection-and-throughput"></a><a name="benchmark"></a>SKU de puerta de enlace por túnel, conexión y rendimiento

[!INCLUDE [Aggregated throughput by SKU](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

## <a name="availability-zones"></a><a name="availability"></a>Zonas de disponibilidad

Las puertas de enlace de VPN se pueden implementar en Azure Availability Zones. Esto aporta una mayor disponibilidad, escalabilidad y resistencia a las puertas de enlace de red virtual. Implementar puertas de enlace en Azure Availability Zones separa de forma física y lógica las puertas de enlace dentro de una región, y protege la conectividad de red local en Azure de errores de nivel de zona. Consulte [Acerca de las puertas de enlace de red virtual con redundancia de zona en Azure Availability Zones](about-zone-redundant-vnet-gateways.md).

## <a name="pricing"></a><a name="pricing"></a>Precios

[!INCLUDE [vpn-gateway-about-pricing-include](../../includes/vpn-gateway-about-pricing-include.md)]

Para más información acerca de las SKU de puerta de enlace para VPN Gateway, consulte [SKU de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

## <a name="faq"></a><a name="faq"></a>P+F

Para conocer las preguntas más frecuentes acerca de VPN Gateway, consulte [Preguntas más frecuentes sobre VPN Gateway](vpn-gateway-vpn-faq.md).

## <a name="whats-new"></a><a name="new"></a>Novedades

Suscríbase a la fuente RSS y vea las actualizaciones más recientes de las características de VPN Gateway en la página [Actualizaciones de Azure](https://azure.microsoft.com/updates/?category=networking&query=VPN%20Gateway).

## <a name="next-steps"></a>Pasos siguientes

- Consulte las [Preguntas más frecuentes sobre VPN Gateway](vpn-gateway-vpn-faq.md) para más información.
- Consulte [Límites del servicio y la suscripción](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).
- Obtenga información sobre las demás [funcionalidades de red](../networking/networking-overview.md) clave de Azure.
