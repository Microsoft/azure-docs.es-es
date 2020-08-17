---
title: Conexión de Azure a nubes públicas | Microsoft Docs
description: Descripción de varias formas de conectar Azure a otras nubes públicas
services: expressroute
author: osamazia
ms.service: expressroute
ms.topic: article
ms.date: 07/24/2019
ms.author: osamaz
ms.openlocfilehash: 80863b56334b0d2d76cdf505dcd15c5cc4c14c52
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081107"
---
# <a name="connecting-azure-with-public-clouds"></a>Conexión de Azure a nubes públicas

Muchas empresas siguen una estrategia multinube, por sus objetivos empresariales y técnicos. Esto incluye el costo, la flexibilidad, la disponibilidad de características, la redundancia, la soberanía de los datos, etc. Esta estrategia les ayuda a aprovechar mejor las dos nubes. 

Este enfoque también plantea desafíos para la empresa en cuanto a la arquitectura de red y de aplicación. Algunos de estos desafíos son la latencia y el rendimiento de los datos. Para abordar estos desafíos, los clientes buscan conectarse directamente a varias nubes. Algunos proveedores de servicios proporcionan una solución para que los clientes se conecten a varios proveedores de la nube. En otros casos, el cliente puede implementar su propio enrutador para conectarse a varias nubes públicas.
## <a name="connectivity-via-expressroute"></a>Conectividad a través de ExpressRoute
ExpressRoute permite a los clientes ampliar las redes locales en la nube de Microsoft mediante una conexión privada que facilita un proveedor de conectividad. Con ExpressRoute, los clientes pueden establecer conexiones con los servicios en la nube de Microsoft.

Hay tres formas de conectarse mediante ExpressRoute.

1. Proveedor Layer3
2. Proveedor Layer2
3. Conexión directa

### <a name="layer3-provider"></a>Proveedor Layer3

Los proveedores Layer3 se suelen conocer como proveedores de VPN de IP o VPN de MPLS. Los clientes aprovechan estos proveedores para la conectividad multipunto entre sus centros de datos, ramas y la nube. Los clientes se conectan al proveedor L3 mediante BGP o la ruta predeterminada estática. El proveedor de servicios anuncia las rutas en los sitios de los clientes, los centros de datos y la nube pública. 
 
Al conectarse a través del proveedor Layer3, Microsoft anuncia las rutas de la red virtual del cliente al proveedor de servicios mediante BGP. El proveedor puede tener dos implementaciones diferentes.

![Diagrama que muestra un proveedor Layer3.](media/expressroute-connect-azure-to-public-cloud/azure-to-public-clouds-l3.png)

Es posible que el proveedor haga que cada proveedor de servicios en la nube aterrice en un VRF independiente, por ejemplo, si el tráfico de todos los proveedores de servicios en la nube va a llegar al enrutador del cliente. Si el cliente ejecuta BGP con el proveedor de servicios, estas rutas se volverán a anunciar a otros proveedores de servicios en la nube de forma predeterminada. 

Si el proveedor de servicios hace que todos los proveedores en la nube aterricen en el mismo VRF, las rutas se anunciarán a otros proveedores de servicios en la nube directamente desde el proveedor de servicios. Esto supone una operación BGP estándar en la que las rutas eBGP se anuncian a otros eBGP colindantes de forma predeterminada.

Cada nube pública tiene un límite de prefijo diferente, por lo que al distribuir la rutas, el proveedor de servicios debe tomar precauciones.

### <a name="layer2-provider-and-direct-connection"></a>Proveedor Layer2 y conexión directa

Aunque la conectividad física de ambos modelos es diferente, BGP en Layer3 se establece directamente entre MSEE y el enrutador del cliente. Para ExpressRoute, el cliente directo se conecta a MSEE directamente. Mientras que en el caso de Layer2, el proveedor de servicios amplía la VLAN de las instalaciones del cliente a la nube. Los clientes ejecutan BGP a partir de la red de Layer2 para conectar sus controladores de red a la nube.
![Diagrama que muestra un proveedor Layer2 y conexión directa.](media/expressroute-connect-azure-to-public-cloud/azure-to-public-clouds-l2.png)
En ambos casos, el cliente tendrá conexiones de punto a punto a cada nube pública. El cliente establecerá una conexión BGP independiente a cada nube pública. De forma predeterminada, las rutas que recibe un proveedor de servicios en la nube se anuncian a otro. Cada proveedor de servicios en la nube tiene un límite de prefijo diferente, por lo que al anunciar las rutas, el cliente debe prestarles atención. El cliente puede usar los botones de BGP habituales con Microsoft mientras anuncia rutas de otras nubes públicas.

## <a name="direct-connection-with-expressroute"></a>Conexión directa con ExpressRoute

Los clientes pueden elegir conectar ExpressRoute directamente a la oferta de conectividad directa del proveedor de servicios de nube. Dos proveedores de servicios en la nube se conectarán entre ellos y la conexión BGP se establecerá directamente entre sus enrutadores. Actualmente, este tipo de conexión está disponible con Oracle.

## <a name="site-to-site-vpn"></a>VPN de sitio a sitio

Los clientes pueden aprovechar Internet para conectar sus instancias en Azure con otras nubes públicas. Casi todos los proveedores de servicios en la nube ofrecen funcionalidades de VPN de sitio a sitio. Sin embargo, podrían existir incompatibilidades por la falta de ciertas variantes. Por ejemplo, algunos proveedores de servicios en la nube solo admiten IKEv1, por lo que n esa nube se necesita un punto de conexión de terminación de VPN. Para aquellos proveedores de servicios en la nube que admiten IKEv2, se puede establecer un túnel directo entre las puertas de enlace de VPN en ambos proveedores de servicios en la nube.

La conexión VPN de sitio a sitio no se considera una solución de baja latencia y alto rendimiento. Sin embargo, se puede usar como copia de seguridad de la conectividad física.

## <a name="next-steps"></a>Pasos siguientes
Consulte [P+F de ExpressRoute][ER-FAQ] para resolver las dudas adicionales sobre la conectividad de redes virtuales y ExpressRoute.

Consulte [Configuración de una conexión directa entre Azure y Oracle Cloud][ER-OCI] para conocer detalles sobre la conectividad entre Azure y Oracle.

<!--Link References-->
[ER-FAQ]: https://docs.microsoft.com/azure/expressroute/expressroute-faqs
[ER-OCI]: https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/configure-azure-oci-networking



