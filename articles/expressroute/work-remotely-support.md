---
title: Uso de Azure ExpressRoute para admitir usuarios remotos
description: En esta página se describe cómo puede aprovechar Azure ExpressRoute para habilitar el trabajo remoto debido a la pandemia de COVID-19.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/22/2020
ms.author: duau
ms.openlocfilehash: e783e80a1883f5aea7a266d3ec670ce2a898e735
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89392963"
---
# <a name="using-azure-expressroute-to-create-hybrid-connectivity-to-support-remote-users"></a>Uso de Azure ExpressRoute para crear una conectividad híbrida para admitir usuarios remotos

En este artículo se describe cómo puede aprovechar ExpressRoute, Azure, la red de Microsoft y el ecosistema de asociados de Azure para trabajar de forma remota.

## <a name="connecting-to-azure-services-with-expressroute"></a>Conexión a los servicios de Azure con ExpressRoute

>[!NOTE]
>En este artículo se describe cómo puede aprovechar ExpressRoute, Azure, la red de Microsoft y el ecosistema de asociados de Azure para trabajar de forma remota y mitigar los problemas de red que está experimentando debido a la crisis de COVID-19.
>

[ExpressRoute](expressroute-introduction.md) es un servicio de Azure que permite la conectividad privada entre los centros de datos de Microsoft y la infraestructura local o de una instalación de coubicación. Las conexiones ExpressRoute no pasan por la red pública de Internet. Ofrecen conectividad segura, confiabilidad, seguridad y velocidad con una latencia menor y más coherente que las conexiones a Internet típicas.

## <a name="useful-links"></a>Vínculos útiles

* [Aumento del ancho de banda de un circuito ExpressRoute existente](expressroute-howto-circuit-portal-resource-manager.md#modify)
* [Supervisión, métricas y alertas de ExpressRoute ](expressroute-monitoring-metrics-alerts.md#expressroute-gateway-connections-in-bitsseconds)
* [Optimización de rutas con ExpressRoute](expressroute-optimize-routing.md)
* [Azure ExpressRoute para O365](https://docs.microsoft.com/office365/enterprise/azure-expressroute?redirectSourcePath=%252farticle%252f6d2534a2-c19c-4a99-be5e-33a0cee5d3bd)
* [Consideraciones sobre el enrutamiento asimétrico](expressroute-asymmetric-routing.md)
* [Abrir una solicitud de soporte técnico en Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

## <a name="troubleshoot"></a>Solución de problemas

* [Comprobación de la conectividad de ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* Obtención de la tabla de ARP en [Resource Manager](expressroute-troubleshooting-arp-resource-manager.md) y [el modelo de implementación clásico](expressroute-troubleshooting-arp-classic.md)
* [Restablecimiento de un circuito con errores](reset-circuit.md)
* [Solución de problemas de rendimiento de la red](expressroute-troubleshooting-network-performance.md)

## <a name="next-steps"></a>Pasos siguientes

* Información acerca de los [Modelos de conectividad de ExpressRoute](expressroute-connectivity-models.md)
* Obtenga información acerca de las conexiones ExpressRoute y dominios de enrutamiento. Consulte [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md)
* Busque un proveedor de servicios. Consulte [Asociados de ExpressRoute de Azure y ubicaciones de emparejamiento](expressroute-locations.md)
* Asegúrese de que se cumplen todos los requisitos previos. Consulte [Requisitos previos de ExpressRoute](expressroute-prerequisites.md).
* [Creación y modificación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
* [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Conexión de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
