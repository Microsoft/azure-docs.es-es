---
title: archivo de inclusión
description: archivo de inclusión
services: networking
author: anavinahar
ms.service: networking
ms.topic: include
ms.date: 01/14/2020
ms.author: anavin
ms.custom: include file
ms.openlocfilehash: b963cc459094e823499e962736550dcea2345ba2
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854106"
---
### <a name="networking-limits---azure-resource-manager"></a><a name="azure-resource-manager-virtual-networking-limits"></a>Límites de redes - Azure Resource Manager
Los límites siguientes solo se aplican a los recursos de redes administrados mediante **Azure Resource Manager** por región y por suscripción. Aprenda a [ver el uso de recursos actual comparado con los límites de su suscripción](../articles/networking/check-usage-against-limits.md).

> [!NOTE]
> Recientemente hemos aumentado todos los límites predeterminados a sus límites máximos. Si no hay ninguna columna de límite máximo, el recurso no tiene límites ajustables. Si el soporte técnico ha aumentado estos límites en el pasado y no ve los límites actualizados en las tablas siguientes, [abra una solicitud de soporte técnico al cliente en línea sin cargo](../articles/azure-resource-manager/templates/error-resource-quota.md)

| Resource | Límite | 
| --- | --- |
| Redes virtuales |1,000 |
| Subredes por red virtual |3000 |
| Emparejamientos de redes virtuales por red virtual |500 |
| [Puertas de enlace de red virtual (puertas de enlace VPN) por red virtual](../articles/vpn-gateway/vpn-gateway-about-vpngateways.md#gwsku) |1 |
| [Puertas de enlace de red virtual (puertas de enlace ExpressRoute) por red virtual](../articles/expressroute/expressroute-about-virtual-network-gateways.md#gwsku) |1 |
| Servidores DNS por red virtual |20 |
| Direcciones IP privadas por red virtual |65 536 |
| Direcciones IP privadas por interfaz de red |256 |
| Direcciones IP privadas por máquina virtual |256 |
| Direcciones IP públicas por interfaz de red |256 |
| Direcciones IP públicas por máquina virtual |256 |
| [Conexiones TCP concurrentes o flujo UDP por NIC de una máquina virtual o instancia de rol](../articles/virtual-network/virtual-machine-network-throughput.md#flow-limits-and-recommendations) |500.000 |
| Tarjetas adaptadoras de red |65 536 |
| Grupos de seguridad de red |5\.000 |
| Reglas de NSG por NSG |1,000 |
| Direcciones IP y rangos especificados para el origen o destino en un grupo de seguridad |4\.000 |
| Grupos de seguridad de aplicaciones |3000 |
| Grupos de seguridad de aplicaciones por configuración de IP, por NIC |20 |
| Configuraciones de IP por grupo de seguridad de aplicaciones |4\.000 |
| Grupos de seguridad de aplicaciones que se pueden especificar en todas las reglas de seguridad de un grupo de seguridad de red |100 |
| Tablas de rutas definidas por el usuario |200 |
| Rutas definidas por el usuario por tabla de ruta |400 |
| Certificados raíz de punto a sitio por Azure VPN Gateway |20 |
| TAP de red virtual |100 |
| Configuraciones de TAP de la interfaz de red por TAP de red virtual |100 |

#### <a name="public-ip-address-limits"></a><a name="publicip-address"></a>Límites de dirección IP pública
| Resource | Límite predeterminado | Límite máximo |
| --- | --- | --- |
| Direcciones IP públicas<sup>1</sup> | 10 para Basic. | Póngase en contacto con el servicio de soporte técnico. |
| Direcciones IP públicas estáticas<sup>1</sup> | 10 para Basic. | Póngase en contacto con el servicio de soporte técnico. |
| Direcciones IP públicas estándar<sup>1</sup> | 10 | Póngase en contacto con el servicio de soporte técnico. |
| Prefijos de dirección IP pública | Limitado por el número de direcciones IP públicas estándar en una suscripción | Póngase en contacto con el servicio de soporte técnico. |
| Longitud del prefijo de IP pública | /28 | Póngase en contacto con el servicio de soporte técnico. |

<sup>1</sup>Los límites predeterminados para las direcciones IP públicas varían según el tipo de categoría de la oferta, por ejemplo, versión de prueba gratuita, Pago por uso o CSP. Por ejemplo, el valor predeterminado para las suscripciones Contrato Enterprise es 1000.

#### <a name="load-balancer-limits"></a><a name="load-balancer"></a>Límites del equilibrador de carga
Los límites siguientes solo se aplican a los recursos de redes administrados a través de Azure Resource Manager por región y por suscripción. Aprenda a [ver el uso de recursos actual comparado con los límites de su suscripción](../articles/networking/check-usage-against-limits.md).

**Standard Load Balancer**

| Resource                                | Límite         |
|-----------------------------------------|-------------------------------|
| Equilibradores de carga                          | 1,000                         |
| Reglas por recurso                      | 1500                         |
| Reglas por NIC (en todas las direcciones IP de una NIC) | 300                           |
| Configuraciones de direcciones IP de front-end              | 600                           |
| Tamaño de grupo de back-end                       | 1000 configuraciones de IP, una sola red virtual |
| Recursos de back-end por Load Balancer <sup>1<sup> | 150                   |
| Puertos de alta disponibilidad                 | 1 por front-end interno       |
| Reglas de salida por Load Balancer        | 600                           |
| [Tiempo de espera de inactividad de TCP](https://docs.microsoft.com/azure/load-balancer/load-balancer-tcp-idle-timeout#tcp-idle-timeout) | 4 minutos/30 minutos          |

<sup>1</sup>El límite es de hasta 150 recursos, en cualquier combinación de recursos de máquinas virtuales independientes, recursos de conjuntos de disponibilidad y grupos de ubicación de conjuntos de escalado de máquinas virtuales.

**Load Balancer básico**

| Resource                                | Límite        |
|-----------------------------------------|------------------------------|
| Equilibradores de carga                          | 1,000                        |
| Reglas por recurso                      | 250                          |
| Reglas por NIC (en todas las direcciones IP de una NIC) | 300                          |
| Configuraciones de direcciones IP de front-end              | 200                          |
| Tamaño de grupo de back-end                       | 300 configuraciones de IP, un solo conjunto de disponibilidad |
| Conjuntos de disponibilidad por Load Balancer     | 150                          |

<a name="virtual-networking-limits-classic"></a>Los límites siguientes se aplican solo a los recursos de redes administrados a través del modelo de implementación **clásico** por suscripción. Aprenda a [ver el uso de recursos actual comparado con los límites de su suscripción](../articles/networking/check-usage-against-limits.md).

| Resource | Límite predeterminado | Límite máximo |
| --- | --- | --- |
| Redes virtuales |100 |100 |
| Sitios de red local |20 |50 |
| Servidores DNS por red virtual |20 |20 |
| Direcciones IP privadas por red virtual |4 096 |4 096 |
| Conexiones TCP concurrentes o flujo UDP por NIC de una máquina virtual o instancia de rol |500 000, hasta 1 000 000 de dos o varias NIC. |500 000, hasta 1 000 000 de dos o varias NIC. |
| Grupos de seguridad de red (NSG) |200 |200 |
| Reglas de NSG por NSG |200 |1,000 |
| Tablas de rutas definidas por el usuario |200 |200 |
| Rutas definidas por el usuario por tabla de ruta |400 |400 |
| Direcciones IP públicas (dinámicas) |500 |500 |
| Direcciones IP públicas reservadas |500 |500 |
| VIP pública por implementación |5 |Ponerse en contacto con soporte técnico |
| VIP privada (equilibrio de carga interno) por implementación |1 |1 |
| Listas de control de acceso (ACL) de punto de conexión |50 |50 |