---
title: Circuitos Azure ExpressRoute y emparejamiento
description: En esta página se proporciona información general sobre los circuitos ExpressRoute y los dominios de enrutamiento/emparejamiento.
services: expressroute
author: mialdrid
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: mialdrid
ms.openlocfilehash: e2c730906e8dcb31c3365259088240c74e416855
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86537027"
---
# <a name="expressroute-circuits-and-peering"></a>Circuitos ExpressRoute y emparejamiento

Los circuitos ExpressRoute conectan la infraestructura local a Microsoft a través de un proveedor de conectividad. En este artículo se le ayudará a comprender los circuitos ExpressRoute y los dominios de enrutamiento/emparejamiento. En la ilustración siguiente se muestra una representación lógica de conectividad entre la WAN y Microsoft.

![Diagrama que muestra cómo se conectan los circuitos ExpressRoute a la infraestructura local a Microsoft a través de un proveedor de conectividad.](./media/expressroute-circuit-peerings/expressroute-basic.png)

> [!IMPORTANT]
> El emparejamiento público de Azure está en desuso y no está disponible para nuevos circuitos de ExpressRoute. Los nuevos circuitos admiten el emparejamiento privado y el emparejamiento de Microsoft.  
>

## <a name="expressroute-circuits"></a><a name="circuits"></a>Circuitos ExpressRoute

Un circuito ExpressRoute representa una conexión lógica entre la infraestructura local y los servicios en la nube de Microsoft a través de un proveedor de conectividad. Puede solicitar varios circuitos ExpressRoute. Cada circuito puede estar en la misma región o en diferentes, y puede estar conectado a su entorno local mediante distintos proveedores de conectividad.

Los circuitos ExpressRoute no se asignan a entidades físicas. Un circuito se identifica de forma única mediante un GUID estándar denominado clave de servicio (s-key). La clave de servicio es la única parte de la información que se intercambia entre Microsoft, el proveedor de conectividad y usted. Por motivos de seguridad, la s-key no es un secreto. Hay una asignación 1:1 entre un circuito ExpressRoute y la s-key.

Los circuitos ExpressRoute pueden incluir dos emparejamientos independientes: emparejamiento privado y emparejamiento de Microsoft. No obstante, los circuitos ExpressRoute existentes pueden contener tres emparejamientos: público de Azure, privado de Azure y de Microsoft. Cada emparejamiento es un par de sesiones de BGP independientes configuradas cada una de ellas de forma redundante para una alta disponibilidad. Hay una asignación 1 (1 <= N <= 3) entre un circuito ExpressRoute y los dominios de enrutamiento. Un circuito ExpressRoute puede tener uno, dos o los tres emparejamientos habilitados por circuito ExpressRoute.

Cada circuito tiene un ancho de banda fijo (50 Mbps, 100 Mbps, 200 Mbps, 500 Mbps, 1 Gbps, 10 Gbps) y se asigna a un proveedor de conectividad y una ubicación de emparejamiento. El ancho de banda que seleccione se comparte entre todos los emparejamientos del circuito.

### <a name="quotas-limits-and-limitations"></a><a name="quotas"></a>Cuotas, límites y limitaciones

Se aplican límites y cuotas predeterminados para cada circuito ExpressRoute. Para obtener la información más actualizada sobre las cuotas, consulte [Suscripción de Azure y límites, cuotas y restricciones de servicio](../azure-resource-manager/management/azure-subscription-service-limits.md) .

## <a name="expressroute-peering"></a><a name="routingdomains"></a>Emparejamiento de ExpressRoute

Un circuito ExpressRoute tiene asociados varios dominios de enrutamiento y emparejamiento: Azure público, Azure privado y Microsoft. Cada emparejamiento tiene una configuración idéntica en un par de enrutadores (configuración en activo-activo o de uso compartido de carga) para la alta disponibilidad. Los servicios de Azure se clasifican como *público de Azure* y *privado de Azure* para representar los esquemas de direcciones IP.

![Diagrama que muestra cómo se configuran los emparejamientos públicos de Azure, privados de Azure y de Microsoft en un circuito de ExpressRoute.](./media/expressroute-circuit-peerings/expressroute-peerings.png)

### <a name="azure-private-peering"></a><a name="privatepeering"></a>Configuración entre pares privados de Azure

Los servicios de proceso de Azure, concretamente las máquinas virtuales (IaaS) y los servicios en la nube (PaaS), que se implementan en una red virtual, pueden estar conectados mediante el dominio de emparejamiento privado. El dominio de emparejamiento privado se considera una extensión confiable de la red principal en Microsoft Azure. Puede configurar una conectividad bidireccional entre la red principal y las redes virtuales de Azure (VNet). De esta forma se puede conectar a máquinas virtuales y servicios en la nube directamente en sus direcciones IP privadas.  

Puede conectar más de una red virtual al dominio de emparejamiento privado. Revise la [página de P+G](expressroute-faqs.md) para obtener información sobre los límites y las limitaciones. Para obtener información actualizada sobre los límites, visite la página [Suscripción de Azure y límites, cuotas y restricciones de servicio](../azure-resource-manager/management/azure-subscription-service-limits.md) .  Consulte la página [Enrutamiento](expressroute-routing.md) para obtener información detallada sobre la configuración de enrutamiento.

### <a name="microsoft-peering"></a><a name="microsoftpeering"></a>Emparejamiento de Microsoft

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

La conectividad con los servicios en línea de Microsoft (Office 365 y servicios de PaaS de Azure) se lleva a cabo mediante el emparejamiento de Microsoft. Permitimos la conectividad bidireccional entre la WAN y los servicios en la nube de Microsoft mediante el dominio de enrutamiento de emparejamiento de Microsoft. Solo debe conectarse a los servicios en la nube de Microsoft mediante direcciones IP públicas que pertenezcan a usted o a su proveedor de conectividad, y debe cumplir todas las reglas definidas. Para más información, consulte la página [Requisitos previos de ExpressRoute](expressroute-prerequisites.md).

Para obtener más información sobre los servicios admitidos, los costos y los datos de configuración, consulte la [página de P+F](expressroute-faqs.md) . Para obtener información sobre la lista de proveedores de conectividad que ofrecen soporte técnico de emparejamiento de Microsoft, consulte la página [Ubicaciones de ExpressRoute](expressroute-locations.md) .

## <a name="peering-comparison"></a><a name="peeringcompare"></a>Comparación de emparejamiento

En la tabla siguiente se comparan los tres emparejamientos:

[!INCLUDE [peering comparison](../../includes/expressroute-peering-comparison.md)]

Puede habilitar uno o varios de los dominios de enrutamiento como parte de su circuito ExpressRoute. Puede elegir colocar todos los dominios de enrutamiento en la misma VPN si quiere combinarlos en un único dominio de enrutamiento. También puede colocarlos en diferentes dominios de enrutamiento, como en el diagrama. La configuración recomendada es que el emparejamiento privado esté conectado directamente a la red principal y que los vínculos de emparejamiento público y de Microsoft estén conectados a la red perimetral.

Cada emparejamiento requiere sesiones de BGP independientes (un par para cada tipo de emparejamiento). Los pares de sesión de BGP proporcionan un vínculo de alta disponibilidad. Si se va a conectar mediante proveedores de conectividad de capa 2, tendrá que encargarse de configurar y administrar el enrutamiento. Para obtener más información, revise los [flujos de trabajo](expressroute-workflows.md) para configurar ExpressRoute.

## <a name="expressroute-health"></a><a name="health"></a>Estado de ExpressRoute

Los circuitos de ExpressRoute se pueden supervisar para comprobar la disponibilidad, la conectividad con redes virtuales y la utilización del ancho de banda mediante [ Network Performance Monitor ](https://docs.microsoft.com/azure/networking/network-monitoring-overview) (NPM).

NPM supervisa el estado del emparejamiento privado de Azure y el emparejamiento de Microsoft. Para más información, consulte nuestra [publicación](https://azure.microsoft.com/blog/monitoring-of-azure-expressroute-in-preview/).

## <a name="next-steps"></a>Pasos siguientes

* Busque un proveedor de servicios. Consulte [Ubicaciones y proveedores de servicios de ExpressRoute](expressroute-locations.md).
* Asegúrese de que se cumplen todos los requisitos previos. Consulte [Requisitos previos de ExpressRoute](expressroute-prerequisites.md).
* Configure su conexión ExpressRoute.
  * [Creación y administración de circuitos ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
  * [Configuración del enrutamiento (emparejamiento) para los circuitos ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
