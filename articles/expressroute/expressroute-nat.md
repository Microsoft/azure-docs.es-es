---
title: 'Azure ExpressRoute: Requisitos de NAT para los circuitos'
description: En esta página se proporcionan requisitos detallados para configurar y administrar NAT para circuitos ExpressRoute.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: cherylmc
ms.openlocfilehash: 62effa04fd6130c35d3e2e64a401c124fe383200
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86521928"
---
# <a name="expressroute-nat-requirements"></a>Requisitos de NAT ExpressRoute
Para conectarse a los Servicios en la nube de Microsoft mediante ExpressRoute, necesitará configurar y administrar NAT. Algunos proveedores de conectividad ofrecen la configuración y administración de NAT como un servicio administrado. Consulte a su proveedor de conectividad para saber si ofrece tal servicio. Si no lo hace, debe cumplir los requisitos que se describen a continuación. 

Revise la página [Circuitos y dominios de enrutamiento ExpressRoute](expressroute-circuit-peerings.md) para obtener información general de los distintos dominios de enrutamiento. Para satisfacer los requisitos de dirección IP pública para emparejamiento de Microsoft y público de Azure, se recomienda configurar NAT entre la red y Microsoft. Esta sección proporciona una descripción detallada de la infraestructura NAT que necesita para configurar.

## <a name="nat-requirements-for-microsoft-peering"></a>Requisitos NAT para el emparejamiento de Microsoft
El emparejamiento de Microsoft le permite conectarse a los servicios en la nube de Microsoft que no se admiten a través de la ruta de acceso de emparejamiento público de Azure. La lista de servicios incluye servicios de Office 365, como Exchange Online, SharePoint Online y Skype Empresarial. Microsoft espera poder admitir conectividad bidireccional en el emparejamiento de Microsoft. Al tráfico destinado a los Servicios en la nube de Microsoft se le debe aplicar SNAT a direcciones IPv4 públicas válidas antes de que entre en la red de Microsoft. Se debe aplicar SNAT al tráfico destinado a su red desde Servicios en la nube de Microsoft en el perímetro de Internet para evitar el [enrutamiento asimétrico](expressroute-asymmetric-routing.md). En la ilustración siguiente, se muestra un esquema general acerca de cómo debería configurarse NAT para el emparejamiento de Microsoft.

![Diagrama general de cómo debe configurarse NAT para el emparejamiento de Microsoft.](./media/expressroute-nat/expressroute-nat-microsoft.png) 

### <a name="traffic-originating-from-your-network-destined-to-microsoft"></a>Tráfico procedente de la red destinado a Microsoft
* Debe asegurarse de que el tráfico se introduce en la ruta de acceso de emparejamiento de Microsoft con una dirección IPv4 pública válida. Microsoft debe poder validar al propietario del grupo de direcciones NAT IPv4 en el registro de Internet de enrutamiento regional (RIR) o en un registro de enrutamiento de Internet (IRR). Se realizará una comprobación basándose en el número AS de emparejamiento y las direcciones IP usadas para NAT. Consulte la página [Requisitos de enrutamiento de ExpressRoute](expressroute-routing.md) para obtener información sobre los registros de enrutamiento.
* Las direcciones IP usadas para la configuración de emparejamiento público de Azure y otros circuitos ExpressRoute no se deben anunciar a Microsoft a través de la sesión BGP. No hay ninguna restricción en la longitud del prefijo de la dirección IP NAT anunciado a través de este emparejamiento.
  
  > [!IMPORTANT]
  > El grupo de direcciones IP NAT anunciado a Microsoft no se debe anunciar a Internet. Esto interrumpirá la conectividad con otros servicios de Microsoft.
  > 
  > 

### <a name="traffic-originating-from-microsoft-destined-to-your-network"></a>Tráfico procedente de Microsoft destinado a la red
* Algunos escenarios requieren que Microsoft inicie la conectividad con extremos de servicio hospedados dentro de la red. Un ejemplo típico del escenario sería la conectividad con los servidores ADFS hospedados en la red de Office 365. En tales casos, debe filtrar prefijos adecuados desde la red en el emparejamiento de tráfico de Microsoft. 
* Debe aplicar SNAT al tráfico de Microsoft en el perímetro de Internet para los puntos de conexión de servicio dentro de la red a fin de evitar el [enrutamiento asimétrico](expressroute-asymmetric-routing.md). Las solicitudes **y respuestas** con una dirección IP de destino que coincidan con una ruta recibida a través de ExpressRoute siempre se enviarán a través de ExpressRoute. Se produce el enrutamiento asimétrico si la solicitud se recibe a través de Internet y la respuesta se envía a través de ExpressRoute. La aplicación de SNAT al tráfico de Microsoft entrante en el perímetro de Internet fuerza el tráfico de respuesta hacia el perímetro de Internet, lo que resuelve el problema.

![Enrutamiento asimétrico con ExpressRoute](./media/expressroute-asymmetric-routing/AsymmetricRouting2.png)

## <a name="nat-requirements-for-azure-public-peering"></a>Requisitos de NAT para el emparejamiento público de Azure

> [!NOTE]
> El emparejamiento público de Azure no está disponible para los circuitos nuevos.
> 

La ruta de acceso de emparejamiento público de Azure le permite conectarse a todos los servicios hospedados en Azure a través de sus direcciones IP públicas. Puede tratarse tanto de los servicios que se enumeran en [P+F de ExpressRoute](expressroute-faqs.md) como de los que hospedan los ISV en Microsoft Azure. 

> [!IMPORTANT]
> La conectividad con servicios de Microsoft Azure en el emparejamiento público siempre se inicia desde la red a la red de Microsoft. Por lo tanto, las sesiones no se pueden iniciar desde los servicios de Microsoft Azure a la red a través de ExpressRoute. Si lo intenta, los paquetes enviados a estas direcciones IP anunciadas usarán Internet en lugar de ExpressRoute.
> 

Al tráfico destinado a Microsoft Azure en el emparejamiento público se le debe aplicar SNAT a direcciones IPv4 públicas válidas antes de que entre en la red de Microsoft. La ilustración siguiente proporciona una imagen de alto nivel de cómo NAT podría configurarse para satisfacer el requisito anterior.

![Diagrama general de cómo debe configurarse para aplicar SNAT a direcciones IPv4 públicas válidas antes de que entre en la red de Microsoft.](./media/expressroute-nat/expressroute-nat-azure-public.png) 

### <a name="nat-ip-pool-and-route-advertisements"></a>Anuncios de ruta y grupo de direcciones IP NAT
Debe asegurarse de que el tráfico se introduce en la ruta de acceso de emparejamiento público de Azure con la dirección IPv4 pública válida. Microsoft debe poder validar la propiedad del grupo de direcciones NAT IPv4 en un registro de Internet de enrutamiento regional (RIR) o en un registro de enrutamiento de Internet (IRR). Se realizará una comprobación basándose en el número AS de emparejamiento y las direcciones IP usadas para NAT. Consulte la página [Requisitos de enrutamiento de ExpressRoute](expressroute-routing.md) para obtener información sobre los registros de enrutamiento.

No hay restricciones en la longitud del prefijo de la dirección IP NAT anunciado a través de este emparejamiento. Debe supervisar el grupo NAT y asegurarse de que no tiene necesidad de sesiones NAT.

> [!IMPORTANT]
> El grupo de direcciones IP NAT anunciado a Microsoft no se debe anunciar a Internet. Esto interrumpirá la conectividad con otros servicios de Microsoft.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
* Consulte los requisitos de [enrutamiento](expressroute-routing.md) y [QoS](expressroute-qos.md).
* Para obtener información sobre los flujos de trabajo, consulte [Flujos de trabajo de aprovisionamiento de circuitos y estados de circuito de ExpressRoute](expressroute-workflows.md).
* Configure su conexión ExpressRoute.
  
  * [Creación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
  * [Configuración del enrutamiento](expressroute-howto-routing-portal-resource-manager.md)
  * [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)

