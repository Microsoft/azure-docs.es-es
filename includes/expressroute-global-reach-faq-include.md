---
title: archivo de inclusión
description: archivo de inclusión
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 03/19/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 317a480c13c5c6e00653fd61878a379df3f65ac4
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2019
ms.locfileid: "65753937"
---
### <a name="what-is-expressroute-global-reach"></a>¿Qué es Global Reach de ExpressRoute?

Global Reach de ExpressRoute es un servicio de Azure que conecta las redes locales con el servicio ExpressRoute a través de la red global de Microsoft. Por ejemplo, si tiene un centro de datos privado en California conectado a ExpressRoute en Silicon Valley y otro centro de datos privado en Texas conectado a ExpressRoute en Dallas, con Global Reach de ExpressRoute puede conectar los centros de datos privados entre sí a través de las dos conexiones de ExpressRoute y el tráfico entre sus centros de datos pasará por la red troncal de Microsoft.

### <a name="how-do-i-enable-or-disable-expressroute-global-reach"></a>¿Cómo habilitar o deshabilitar Global Reach de ExpressRoute?

Al conectar los circuitos de ExpressRoute, se habilita Global Reach de ExpressRoute. La característica se deshabilita al desconectar los circuitos. Consulte la [configuración](../articles/expressroute/expressroute-howto-set-global-reach.md).

### <a name="do-i-need-expressroute-premium-for-expressroute-global-reach"></a>¿Es necesario ExpressRoute Premium para Global Reach de ExpressRoute?

Si los circuitos de ExpressRoute están en la misma región geopolítica, no es necesario ExpressRoute Premium para conectarlos entre sí. Si los dos circuitos de ExpressRoute están en distintas regiones geopolíticas, necesita ExpressRoute Premium en los dos circuitos para habilitar la conectividad entre ellos. 

### <a name="how-will-i-be-charged-for-expressroute-global-reach"></a>¿Cómo se cobrará Global Reach de ExpressRoute?

ExpressRoute permite la conectividad de la red local con los servicios en la nube de Microsoft. Global Reach de ExpressRoute permite la conectividad entre sus propias redes locales a través de los circuitos de ExpressRoute existentes, aprovechando la red global de Microsoft. Global Reach de ExpressRoute se factura por separado del servicio de ExpressRoute existente. Hay una cuota complementaria para habilitar esta característica en cada circuito de ExpressRoute. El tráfico entre las redes locales habilitado por Global Reach de ExpressRoute se facturará con una tasa de salida en el origen y una tasa de entrada en el destino. Las tarifas se basan en la zona en la que se encuentran los circuitos.

### <a name="where-is-expressroute-global-reach-supported"></a>¿Dónde se admite Global Reach de ExpressRoute?

Alcance Global de ExpressRoute se admite en [seleccionar países o regiones o lugares](../articles/expressroute/expressroute-global-reach.md). Los circuitos de ExpressRoute se deben crear en las ubicaciones de emparejamiento en dichos países o regiones o en lugares.

### <a name="i-have-more-than-two-on-premises-networks-each-connected-to-an-expressroute-circuit-can-i-enable-expressroute-global-reach-to-connect-all-of-my-on-premises-networks-together"></a>Tengo más de dos redes locales, cada una conectada a un circuito de ExpressRoute. ¿Puedo habilitar Global Reach de ExpressRoute para conectar todas mis redes locales entre sí?

Sí, puede hacerlo, siempre y cuando los circuitos se encuentran en los países o regiones admitidos. Debe conectar dos circuitos de ExpressRoute cada vez. Para crear una red totalmente integrada, debe enumerar todos los pares de circuitos y repetir la configuración. 

### <a name="can-i-enable-expressroute-global-reach-between-two-expressroute-circuits-at-the-same-peering-location"></a>¿Puedo habilitar Global Reach de ExpressRoute entre dos circuitos de ExpressRoute en la misma ubicación de emparejamiento?

No. Los dos circuitos deben estar en distintas ubicaciones de emparejamiento. Si un metro en un país o región admitidas tiene más de una ubicación de emparejamiento de ExpressRoute, se pueden conectar los circuitos ExpressRoute creados en distintas ubicaciones de emparejamiento en ese metro. 

### <a name="if-expressroute-global-reach-is-enabled-between-circuit-x-and-circuit-y-and-between-circuit-y-and-circuit-z-will-my-on-premises-networks-connected-to-circuit-x-and-circuit-z-talk-to-each-other-via-microsofts-network"></a>Si Global Reach de ExpressRoute está habilitado entre el circuito X y el circuito Y y también entre el circuito Y y el circuito Z, ¿se comunicarán entre sí las redes locales conectadas al circuito X y al circuito Z a través de la red de Microsoft?

No. Para habilitar la conectividad entre dos de las redes locales, se deben conectar explícitamente los circuitos de ExpressRoute correspondientes. En el ejemplo anterior, debe conectar el circuito X y el circuito Z. 

### <a name="what-is-the-network-throughput-i-can-expect-between-my-on-premises-networks-after-i-enable-expressroute-global-reach"></a>¿Cuál es el rendimiento de red que puedo esperar entre mis redes locales después de habilitar Global Reach de ExpressRoute?

El rendimiento de red entre las redes locales habilitado por Global Reach de ExpressRoute está limitado por el menor de los dos circuitos de ExpressRoute. El tráfico local a Azure y tráfico local a local comparten el mismo circuito y están sujetos al mismo extremo de ancho de banda. 

### <a name="with-expressroute-global-reach-what-are-the-limits-on-the-number-of-routes-i-can-advertise-and-the-number-of-routes-i-will-receive"></a>Con ExpressRoute Global Reach, ¿cuáles son los límites en el número de rutas que puedo anunciar y el número de rutas que recibiré?

El número de rutas que puede anunciar a Microsoft en el emparejamiento privado de Azure sigue siendo 4000 en un circuito estándar o 10 000 en un circuito premium. El número de rutas que recibirá de Microsoft en el emparejamiento privado de Azure será la suma de las rutas de las redes virtuales de Azure y las rutas de las otras redes locales conectadas a través de ExpressRoute Global Reach. Asegúrese de establecer un límite de prefijo máximo aproximado en el enrutador local. 

### <a name="what-is-the-sla-for-expressroute-global-reach"></a>¿Cuál es el Acuerdo de Nivel de Servicio de ExpressRoute Global Reach?

ExpressRoute Global Reach proporcionará el mismo [Acuerdo de Nivel de Servicio de disponibilidad](https://azure.microsoft.com/support/legal/sla/expressroute/v1_3/) que el servicio ExpressRoute habitual.
