---
title: 'Azure ExpressRoute: Diseño para lograr alta disponibilidad'
description: Esta página proporciona recomendaciones de arquitectura para una alta disponibilidad al usar Azure ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 06/28/2019
ms.author: duau
ms.openlocfilehash: d6e63ad5781b0b8d6f46aee1cb88373721bb9dfc
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397906"
---
# <a name="designing-for-high-availability-with-expressroute"></a>Diseño de alta disponibilidad con ExpressRoute

ExpressRoute está diseñado para ofrecer una alta disponibilidad y así poder proporcionar conectividad de red privada a nivel de operador a los recursos de Microsoft. En otras palabras, no hay un único punto de error en la ruta de ExpressRoute dentro de la red de Microsoft. Para maximizar la disponibilidad, el segmento del cliente y del proveedor de servicios de su circuito de ExpressRoute también debe ser diseñado para ofrecer una alta disponibilidad. En este artículo, primero veremos las opciones a tener en cuenta de la arquitectura de red para crear una conectividad de red robusta mediante ExpressRoute; a continuación, veremos las características de ajuste preciso que le ayudarán a mejorar la alta disponibilidad del circuito de ExpressRoute.

>[!NOTE]
>Los conceptos descritos en este artículo se aplican de la misma forma cuando se crea un circuito ExpressRoute en una red WAN virtual o fuera de él.
>

## <a name="architecture-considerations"></a>Consideraciones sobre la arquitectura

En la siguiente ilustración se muestra el método recomendado de conectarse mediante un circuito de ExpressRoute para maximizar la disponibilidad de un circuito de ExpressRoute.

 [![1]][1]

Para obtener una alta disponibilidad, es esencial mantener la redundancia del circuito de ExpressRoute en toda la red de un extremo a otro. En otras palabras, debe mantener la redundancia de la red local y no debe comprometer la redundancia de la red de su proveedor de servicios. Mantener la redundancia al mínimo implica evitar puntos únicos de errores en la red. Tener energía y refrigeración redundantes para los dispositivos de red mejorará aún más la alta disponibilidad.

### <a name="first-mile-physical-layer-design-considerations"></a>Consideraciones de diseño del nivel físico de la primera milla

 Si finaliza las conexiones principales y secundarias de un circuito de ExpressRoute en el mismo equipo de las instalaciones del cliente (CPE), está comprometiendo la alta disponibilidad de su red local. Además, si configura las conexiones principales y secundarias a través del mismo puerto de un CPE (ya sea terminando las dos conexiones en diferentes subinterfaces o combinando las dos conexiones dentro de la red del asociado), está obligando a ese asociado a comprometer la alta disponibilidad de su segmento de red. Este compromiso se ilustra en la siguiente imagen.

[![2]][2]

Por otro lado, si finaliza las conexiones principales y secundarias de un circuito de ExpressRoute en diferentes ubicaciones geográficas, podría estar comprometiendo el rendimiento de la red de la conectividad. Si el tráfico equilibra la carga activamente a través de las conexiones principales y secundarias que finalizan en diferentes ubicaciones geográficas, una diferencia sustancial potencial en la latencia de la red entre las dos rutas daría como resultado un rendimiento de la red subóptimo. 

En cuanto a las opciones de diseño con redundancia geográfica, consulte [Designing for disaster recovery with ExpressRoute][DR] (Diseño para recuperación ante desastres con ExpressRoute).

### <a name="active-active-connections"></a>Conexiones activas-activas

La red de Microsoft está configurada para administrar las conexiones principales y secundarias de los circuitos de ExpressRoute en modo activo-activo. Sin embargo, a través de los anuncios de ruta, puede forzar las conexiones redundantes de un circuito de ExpressRoute para poder trabajar en modo activo-pasivo. La publicidad de rutas más específicas y la preparación de rutas de acceso BGP AS son técnicas comunes que se usan para hacer que una ruta de acceso se prefiera sobre otra.

Para mejorar la alta disponibilidad, se recomienda trabajar con las conexiones de un circuito de ExpressRoute en modo activo-activo. Si deja que las conexiones funcionen en modo activo-activo, la red de Microsoft equilibrará la carga del tráfico en todas las conexiones en función del flujo.

Al ejecutar las conexiones principales y secundarias de un circuito de ExpressRoute en modo activo-pasivo, se corre el riesgo de que ambas conexiones produzcan un error que derivará en otro error en la ruta de acceso activa. Las causas comunes de errores en la conmutación son la falta de administración activa de la conexión pasiva y la conexión pasiva que anuncia las rutas obsoletas.

Como alternativa, ejecutar las conexiones principales y secundarias de un circuito de ExpressRoute en modo activo-activo, solo produce la mitad de los flujos que crean un error y se vuelven a enrutar; a continuación, se produce un error en la conexión de ExpressRoute. Por lo tanto, el modo activo-activo ayudará significativamente a mejorar el tiempo medio de recuperación (MTTR).

### <a name="nat-for-microsoft-peering"></a>NAT para el emparejamiento de Microsoft 

El emparejamiento de Microsoft está diseñado para la comunicación entre los puntos de conexión públicos. Por lo general, los puntos de conexión privados en el entorno local son la traducción de direcciones de red (NAT) con IP pública en la red del cliente o asociado, antes de que estos se comuniquen mediante el emparejamiento de Microsoft. Suponiendo que usa las conexiones principal y secundaria en el modo activo-activo, se decidirá el impacto que tendrá NAT (dónde y cómo) en la rapidez con la que se recupera después de un error en una de las conexiones de ExpressRoute. Dos opciones de NAT diferentes se ilustran en la siguiente imagen:

[![3]][3]

En la opción 1, NAT se aplica después de dividir el tráfico entre las conexiones principal y secundaria de ExpressRoute. Para cumplir con los requisitos de estado de NAT, se usan grupos de NAT independientes entre los dispositivos principal y secundario para que el tráfico de retorno llegue al mismo dispositivo de borde a través del cual salió el flujo.

En la opción 2, se usa un conjunto común de NAT antes de dividir el tráfico entre las conexiones principal y secundaria de ExpressRoute. Es importante distinguir el conjunto de NAT común antes de dividir el tráfico; esto no significa que debe agregar un único punto de error, puesto que comprometería la alta disponibilidad.

Con la opción 1, después de un error de conexión de ExpressRoute, se interrumpe la capacidad para alcanzar el grupo de NAT correspondiente. Por lo tanto, todos los flujos interrumpidos deben restablecerse mediante TCP o el nivel de aplicación después de que pase el tiempo de espera de la ventana correspondiente. Si cualquiera de los grupos de NAT se usa en la interfaz de cualquiera de los servidores locales y si se produjera un error en la conectividad correspondiente, no se puede obtener acceso a los servidores locales desde Azure hasta que se solucione el error de conectividad.

Así pues, con la opción 2, se puede obtener acceso a NAT incluso después de un error en la conexión principal o secundaria. Por lo tanto, la propia capa de red puede redireccionar los paquetes y permitir una recuperación más rápida después del error. 

> [!NOTE]
> Si usa la opción 1 de NAT (grupos de NAT independientes para conexiones de ExpressRoute principales y secundarias) y asigna un puerto de una dirección IP desde uno de los grupos de NAT a un servidor local, no se podrá acceder al servidor a través del circuito de ExpressRoute cuando se produzca un error en la conexión correspondiente.
> 

## <a name="fine-tuning-features-for-private-peering"></a>Características de ajuste preciso para el emparejamiento privado

En esta sección, revisaremos las características opcionales (según la implementación de Azure y su sensibilidad a MTTR) que le ayudarán a mejorar la alta disponibilidad de su circuito de ExpressRoute. Específicamente, revisaremos la implementación de las puertas de enlace de red virtuales de ExpressRoute y la detección de reenvío bidireccional (BFD).

### <a name="availability-zone-aware-expressroute-virtual-network-gateways"></a>Zona de disponibilidad compatible con las puertas de enlace de red virtual de ExpressRoute

Una zona de disponibilidad de una región de Azure es una combinación de un dominio de error y un dominio de actualización. Si opta por implementar IaaS de Azure con redundancia de zona, es posible que también quiera configurar las puertas de enlace de la red virtual con redundancia de zona que finalizan el emparejamiento privado de ExpressRoute. Para más información, consulte [Acerca de las puertas de enlace de red virtual con redundancia de zona en Azure Availability Zones][zone redundant vgw]. Para configurar la puerta de enlace de red virtual con redundancia de zona, consulte [Crear una puerta de enlace de red virtual con redundancia de zona en Azure Availability Zones][conf zone redundant vgw].

### <a name="improving-failure-detection-time"></a>Mejorar el tiempo de detección de errores

ExpressRoute admite BFD a través del emparejamiento privado. BFD reduce el tiempo de detección de errores en la red denominada "Capa 2" entre Microsoft Enterprise Edge (MSEE) y sus vecinos BGP en el entorno local de 3 minutos (predeterminado) hasta menos de un segundo. Gracias a esta rapidez en el tiempo de detección de errores, podrá acelerar la recuperación ante errores. Para obtener más información, consulte [Configuración de BFD a través de ExpressRoute][BFD].

## <a name="next-steps"></a>Pasos siguientes

En este artículo, discutiremos cómo diseñar la alta disponibilidad de una conectividad de circuito de ExpressRoute. Como un punto de emparejamiento del circuito de ExpressRoute se ancla a una ubicación geográfica, podría verse afectado por un error grave que afecte a toda la ubicación. 

En cuanto a las opciones de diseño para compilar una conectividad de red con redundancia geográfica en la red troncal de Microsoft que pueda soportar errores graves (que afectan a toda la región), consulte [Designing for disaster recovery with ExpressRoute private peering][DR] (Diseñar para la recuperación ante desastres con el emparejamiento privado de ExpressRoute).

<!--Image References-->
[1]: ./media/designing-for-high-availability-with-expressroute/exr-reco.png "Recomendación para conectarse con ExpressRoute "
[2]: ./media/designing-for-high-availability-with-expressroute/suboptimal-lastmile-connectivity.png "conectividad de última milla subóptima"
[3]: ./media/designing-for-high-availability-with-expressroute/nat-options.png "opciones de NAT"


<!--Link References-->
[zone redundant vgw]: https://docs.microsoft.com/azure/vpn-gateway/about-zone-redundant-vnet-gateways
[conf zone redundant vgw]: https://docs.microsoft.com/azure/vpn-gateway/create-zone-redundant-vnet-gateway
[Configure Global Reach]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-set-global-reach
[BFD]: https://docs.microsoft.com/azure/expressroute/expressroute-bfd
[DR]: https://docs.microsoft.com/azure/expressroute/designing-for-disaster-recovery-with-expressroute-privatepeering




