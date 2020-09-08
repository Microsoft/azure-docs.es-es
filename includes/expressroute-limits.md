---
title: archivo de inclusión
description: archivo de inclusión
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 07/25/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 146a257a9b8f31e79283d71ede1e1c5c0355765c
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89147786"
---
| Resource | Límite |
| --- | --- |
| Circuitos ExpressRoute por suscripción |10 |
| Circuitos ExpressRoute por región y suscripción, con Azure Resource Manager |10 |
| Número máximo de rutas anunciadas para emparejamiento privado de Azure con ExpressRoute estándar |4\.000 |
| Número máximo de rutas anunciadas para emparejamiento privado de Azure con complemento de ExpressRoute premium |10 000 |
| Número máximo de rutas anunciadas desde el emparejamiento privado de Azure a partir del espacio de direcciones de red virtual de una conexión ExpressRoute |200 |
| Número máximo de rutas anunciadas para emparejamiento de Microsoft con ExpressRoute estándar |200 |
| Número máximo de rutas anunciadas para emparejamiento de Microsoft con el complemento de ExpressRoute premium |200 |
| Número máximo de circuitos de ExpressRoute vinculados a una puerta de enlace de rendimiento estándar/ErGw1AZ |4 | 
| Número máximo de circuitos de ExpressRoute vinculados a una puerta de enlace de alto rendimiento/ErGw2AZ |8 | 
| Número máximo de circuitos de ExpressRoute vinculados a una puerta de enlace de ultrarrendimiento/ErGw3AZ |16 | 
| Número máximo de circuitos ExpressRoute vinculado a la misma red virtual en distintas ubicaciones de emparejamiento (todas las SKU) |4 |
| Número de vínculos de red virtual permitidos por circuito ExpressRoute |Consulte la tabla [Número de redes virtuales por circuito ExpressRoute](#vnetpercircuit).  |

#### <a name="number-of-virtual-networks-per-expressroute-circuit"></a><a name="vnetpercircuit"></a> Número de redes virtuales por circuito ExpressRoute
| **Tamaño del circuito** | **Número de vínculos de red virtual para estándar** | **Número de vínculos de red virtual con complemento premium** |
| --- | --- | --- |
| 50 Mbps |10 |20 |
| 100 Mbps |10 |25 |
| 200 Mbps |10 |25 |
| 500 Mbps |10 |40 |
| 1 Gbps |10 |50 |
| 2 Gbps |10 |60 |
| 5 Gbps |10 |75 |
| 10 Gbps |10 |100 |
| 40 Gbps* |10 |100 |
| 100 Gbps* |10 |100 |

**Solo ExpressRoute Direct de 100 Gbps*

> [!NOTE]
> Las conexiones de Global Reach cuentan para el límite de conexiones de red virtual por circuito ExpressRoute. Por ejemplo, un circuito Premium de 10 Gbps permitiría 5 conexiones de Global Reach y 95 conexiones a las puertas de enlace de ExpressRoute, o bien 95 conexiones de Global Reach y 5 conexiones a las puertas de enlace de ExpressRoute, o cualquier otra combinación hasta alcanzar el límite de 100 conexiones para el circuito.
