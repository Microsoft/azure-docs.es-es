---
title: 'Escenario: universal'
titleSuffix: Azure Virtual WAN
description: 'Escenarios de enrutamiento: universal'
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 95fa7a8c6abd0ad65b367cacef15b8faa16da640
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553437"
---
# <a name="scenario-any-to-any"></a>Escenario: Universal

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En un escenario universal, cualquier radio puede comunicarse con otro radio. Cuando existen varios centros, el enrutamiento de centro a centro (también conocido como hub-to-hub o interhub) está habilitado de forma predeterminada en Virtual WAN Estándar. Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Diseño

Para averiguar el número de tablas de rutas que se necesitarán en un escenario de WAN virtual, puede crear una matriz de conectividad, donde cada celda representa si un origen (fila) puede comunicarse con un destino (columna). La matriz de conectividad de este escenario es trivial, pero se incluye para que sea coherente con otros escenarios.

| De |   A |  *Redes virtuales* | *Ramas* |
| -------------- | -------- | ---------- | ---|
| Redes virtuales     | &#8594;|      X     |     X    |
| Ramas   | &#8594;|    X     |     X    |

Cada una de las celdas de la tabla anterior describe si una conexión de Virtual WAN (el lado "From" del flujo, los encabezados de fila de la tabla) aprende un prefijo de destino (el lado "To" del flujo, los encabezados de columna en cursiva de la tabla) para un flujo de tráfico concreto.

Dado que todas las conexiones de redes virtuales y ramas (VPN, ExpressRoute y VPN de usuario) tienen los mismos requisitos de conectividad, se requiere una sola tabla de rutas. Como resultado, todas las conexiones se asociarán y se propagarán a la misma tabla de rutas, la tabla de rutas predeterminada:

* Redes virtuales:
  * Tabla de rutas asociada: **Valor predeterminado**
  * Propagación a tablas de rutas: **Valor predeterminado**
* Ramas:
  * Tabla de rutas asociada: **Valor predeterminado**
  * Propagación a tablas de rutas: **Valor predeterminado**

Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="architecture"></a><a name="architecture"></a>Arquitectura

En la **Ilustración 1**, todas las redes virtuales y ramas (VPN, ExpressRoute, P2S) pueden comunicarse entre sí. En un centro virtual, las conexiones funcionan de la siguiente manera:

* Una conexión VPN conecta un sitio VPN a una puerta de enlace de VPN.
* La conexión de red virtual conecta una red virtual a un centro virtual. El enrutador del centro virtual proporciona la funcionalidad de tránsito entre las redes virtuales.
* Una conexión ExpressRoute conecta un circuito de ExpressRoute a una puerta de enlace de ExpressRoute.

Estas conexiones (de forma predeterminada en la creación) están asociadas a la tabla de rutas predeterminada, a menos que defina la configuración de enrutamiento de la conexión en **Ninguno** o en una tabla de rutas personalizada. De forma predeterminada, estas conexiones también se propagan a la tabla de rutas predeterminada. Esto es lo que permite un escenario universal, en el que cualquier radio (red virtual, VPN, ER, P2S) puede comunicarse con cualquier otro.

**Ilustración 1**

:::image type="content" source="./media/routing-scenarios/any-any/figure-1.png" alt-text="ilustración 1":::

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Este escenario está habilitado de forma predeterminada para Virtual WAN Estándar. Si la configuración de rama a rama está deshabilitada en la configuración de WAN, no se permitirá la conectividad entre los radios de las ramas. VPN/ExpressRoute/VPN de usuario se consideran radios de ramas en Virtual WAN.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
