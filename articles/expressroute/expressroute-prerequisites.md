---
title: 'Azure ExpressRoute: Prerrequisitos'
description: Esta página proporciona una lista de requisitos que deben cumplirse para poder solicitar un circuito ExpressRoute de Azure. Incluye una lista de comprobación.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: df24544a1eb5b857e92e5be05bdbffdfc64a846d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395050"
---
# <a name="expressroute-prerequisites--checklist"></a>Requisitos previos y lista de comprobación de ExpressRoute
Para conectarse a Microsoft Cloud Services con ExpressRoute, debe comprobar que se han cumplido los requisitos enumerados en las secciones siguientes.

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

## <a name="azure-account"></a>Cuenta de Azure
* Una cuenta de Microsoft Azure válida y activa Esta cuenta es necesaria para configurar el circuito ExpressRoute. Los circuitos ExpressRoute son recursos que hay dentro de las suscripciones de Azure. Se necesita una suscripción de Azure, aunque la conectividad esté limitada a servicios en la nube de Microsoft que no sean Azure, como Office 365.
* Una suscripción activa de Office 365 (si usa los servicios de Office 365). Para más información, consulte la sección acerca de los requisitos específicos de Office 365 de este mismo artículo.

## <a name="connectivity-provider"></a>Proveedor de conectividad

* Para conectarse a la nube de Microsoft, puede trabajar con un [asociado de conectividad ExpressRoute](expressroute-locations.md#partners) . Se puede configurar una conexión entre la red local y Microsoft de [tres maneras](expressroute-introduction.md).
* Aunque el proveedor no sea un asociado de conectividad ExpressRoute, puede conectarse a la nube de Microsoft mediante un [proveedor de intercambio de nube](expressroute-locations.md#connectivity-through-exchange-providers).

## <a name="network-requirements"></a>Requisitos de red
* **Redundancia en cada ubicación de emparejamiento**: Microsoft requiere que se configuren sesiones BGP redundantes entre enrutadores de Microsoft y enrutadores de emparejamiento en cada circuito ExpressRoute, aunque solo tenga [una conexión física a un intercambio de nube](expressroute-faqs.md#onep2plink).
* **Redundancia para la recuperación ante desastres**: Microsoft recomienda encarecidamente configurar al menos dos circuitos ExpressRoute en distintas ubicaciones de emparejamiento para evitar un único punto de error.
* **Enrutamiento**: según como se conecte a Microsoft Cloud, usted o su proveedor deben configurar y administrar las sesiones de BGP para los [dominios de enrutamiento](expressroute-circuit-peerings.md). Algunos proveedores de conectividad Ethernet o proveedores de intercambio en la nube pueden ofrecer administración de BGP como un servicio de valor añadido.
* **NAT**: Microsoft solo acepta direcciones IP públicas a través de emparejamiento de Microsoft. Si usa direcciones IP privadas en la red local, usted o su proveedor deben convertir las direcciones IP privadas en direcciones IP públicas [con NAT](expressroute-nat.md).
* **QoS**: Skype Empresarial cuenta con varios servicios (por ejemplo, voz, vídeo o texto) que requieren un tratamiento diferenciado de QoS. Usted y su proveedor deben seguir los [requisitos de QoS](expressroute-qos.md).
* **Seguridad de red**: cuando se conecte a Microsoft Cloud mediante ExpressRoute, tenga en cuenta la [seguridad de red](../best-practices-network-security.md) .

## <a name="office-365"></a>Office 365
Si tiene pensado habilitar Office 365 en ExpressRoute, revise los siguientes documentos para tener más información sobre los requisitos de Office 365.

* [Información general de ExpressRoute para Office 365](https://support.office.com/article/Azure-ExpressRoute-for-Office-365-6d2534a2-c19c-4a99-be5e-33a0cee5d3bd)
* [Enrutamiento de ExpressRoute para Office 365](https://support.office.com/article/Routing-with-ExpressRoute-for-Office-365-e1da26c6-2d39-4379-af6f-4da213218408)
* [Alta disponibilidad y conmutación por error con ExpressRoute](https://aka.ms/erhighavailability)
* [Direcciones URL e intervalos de direcciones IP de Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)
* [Planes de red y ajuste del rendimiento para Office 365](https://support.office.com/article/Network-planning-and-performance-tuning-for-Office-365-e5f1228c-da3c-4654-bf16-d163daee8848)
* [Herramientas y calculadoras de ancho de banda de red](https://support.office.com/article/Network-and-migration-planning-for-Office-365-f5ee6c33-bcd7-4b0b-b0f8-dc1d9fb8d132)
* [Integración de Office 365 con entornos locales](https://support.office.com/article/Office-365-integration-with-on-premises-environments-263faf8d-aa21-428b-aed3-2021837a4b65)
* [Vídeos sobre el aprendizaje avanzado de ExpressRoute en Office 365](https://channel9.msdn.com/series/aer/)

## <a name="next-steps"></a>Pasos siguientes
* Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).
* Encuentre un proveedor de conectividad ExpressRoute. Consulte [Asociados de ExpressRoute de Azure y ubicaciones de emparejamiento](expressroute-locations.md).
* Consulte los requisitos de [enrutamiento](expressroute-routing.md), [NAT](expressroute-nat.md) y [QoS](expressroute-qos.md).
* Configure su conexión ExpressRoute.
  * [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Configuración del enrutamiento](expressroute-howto-routing-arm.md)
  * [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md)
