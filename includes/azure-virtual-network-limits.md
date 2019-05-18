---
title: archivo de inclusión
description: archivo de inclusión
services: networking
author: anavinahar
ms.service: networking
ms.topic: include
ms.date: 05/15/2019
ms.author: anavin
ms.custom: include file
ms.openlocfilehash: 8cee74696d9636a8ac1d6fcbab493f18184355e1
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787639"
---
<a name="virtual-networking-limits-classic"></a>Los límites siguientes se aplican solo para los recursos administrados a través de redes la **clásico** modelo de implementación por suscripción. Aprenda a [ver el uso de recursos actual comparado con los límites de su suscripción](../articles/networking/check-usage-against-limits.md).

| Recurso | Límite predeterminado | Límite máximo |
| --- | --- | --- |
| Redes virtuales |50 |100 |
| Sitios de red local |20 |Póngase en contacto con el servicio de soporte técnico. |
| Servidores DNS por red virtual |20 |20 |
| Direcciones IP privadas por red virtual |4 096 |4 096 |
| Conexiones TCP concurrentes o flujo UDP por NIC de una máquina virtual o instancia de rol |500.000, hasta 1 000 000 de dos o varias NIC. |500.000, hasta 1 000 000 de dos o varias NIC. |
| Grupos de seguridad de red (NSG) |200 |200 |
| Reglas de NSG por NSG |1000 |1000 |
| Tablas de rutas definidas por el usuario |200 |200 |
| Rutas definidas por el usuario por tabla de rutas |400 |400 |
| Direcciones IP públicas (dinámicas) |5 |Ponerse en contacto con soporte técnico |
| Direcciones IP públicas reservadas |20 |Ponerse en contacto con soporte técnico |
| VIP pública por implementación |5 |Ponerse en contacto con soporte técnico |
| VIP privada (equilibrio de carga interno) por implementación |1 |1 |
| Listas de control de acceso de punto de conexión (ACL) |50 |50 |

#### <a name="azure-resource-manager-virtual-networking-limits"></a>Límites de redes - Azure Resource Manager
Los límites siguientes se aplican solo para los recursos administrados a través de redes **Azure Resource Manager** por región y suscripción. Aprenda a [ver el uso de recursos actual comparado con los límites de su suscripción](../articles/networking/check-usage-against-limits.md).

> [!NOTE]
> Recientemente hemos aumentado todos los límites predeterminados a sus límites máximos. Si no hay ninguna columna de límite máximo, el recurso no tiene límites ajustables. Si se hubiera estos límites incrementando el soporte técnico en el pasado y no ve los límites de actualizado en las tablas siguientes, [abrir una solicitud de soporte técnico al cliente en línea sin cargo](../articles/azure-resource-manager/resource-manager-quota-errors.md)

| Recurso | Límite predeterminado y máximo | 
| --- | --- |
| Redes virtuales |1000 |
| Subredes por red virtual |3000 |
| Emparejamientos de redes virtuales por red virtual |500 |
| Servidores DNS por red virtual |20 |
| Direcciones IP privadas por red virtual |65,536 |
| Direcciones IP privadas por interfaz de red |256 |
| Direcciones IP privadas por cada máquina virtual |256 |
| Conexiones TCP concurrentes o flujo UDP por NIC de una máquina virtual o instancia de rol |500,000 |
| Tarjetas de interfaz de red |65,536 |
| Grupos de seguridad de red |5.000 |
| Reglas de NSG por NSG |1000 |
| Direcciones IP y rangos especificados para el origen o destino en un grupo de seguridad |4.000 |
| Grupos de seguridad de aplicación |3000 |
| Grupos de seguridad de aplicaciones por configuración de IP, por NIC |20 |
| Configuraciones de IP por grupo de seguridad de aplicaciones |4.000 |
| Grupos de seguridad de aplicaciones que se pueden especificar en todas las reglas de seguridad de un grupo de seguridad de red |100 |
| Tablas de rutas definidas por el usuario |200 |
| Rutas definidas por el usuario por tabla de rutas |400 |
| Certificados raíz de punto a sitio por puerta de enlace de VPN de Azure |20 |
| TAP de red virtual |100 |
| Configuraciones de TAP de la interfaz de red por TAP de red virtual |100 |

#### <a name="publicip-address"></a>Límites de dirección IP pública
| Recurso | Límite predeterminado | Límite máximo |
| --- | --- | --- |
| Direcciones IP públicas - dinámicas | 1000 por Basic. |Póngase en contacto con el servicio de soporte técnico. |
| Direcciones IP públicas - estáticas | 1000 por Basic. |Póngase en contacto con el servicio de soporte técnico. |
| Direcciones IP públicas - estáticas | 200 estándar.|Póngase en contacto con el servicio de soporte técnico. |
| Tamaño de prefijo IP pública (versión preliminar) | /28 | Póngase en contacto con el servicio de soporte técnico. |

#### <a name="load-balancer"></a>Límites del equilibrador de carga
Los límites siguientes solo se aplican a los recursos de redes administrados a través de Azure Resource Manager por región y por suscripción. Aprenda a [ver el uso de recursos actual comparado con los límites de su suscripción](../articles/networking/check-usage-against-limits.md).

| Recurso | Límite predeterminado y máximo |
| --- | --- |
| Equilibradores de carga | 1000 | 
| Reglas por recurso, básico | 250 |
| Reglas por recurso, estándar | 1500 | 
| Reglas por configuración de dirección IP | 299 |
| Reglas por NIC | 500 |
| Configuraciones de IP front-end, básico | 200 |
| Configuraciones de IP front-end, estándar | 600 |
| Grupo de back-end, básico | 100, conjunto de disponibilidad único |
| Grupo de back-end, estándar | 1000, red virtual única |
| Recursos de back-end por cada equilibrador de carga estándar<sup>1</sup> | 150 |
| Puertos de alta disponibilidad, estándar | 1 por interno front-end |

<sup>1</sup>el límite es de hasta 150 recursos, en cualquier combinación de recursos de máquina virtual independiente, el conjunto de disponibilidad de recursos y recursos de conjunto de escalado de máquina virtual.

