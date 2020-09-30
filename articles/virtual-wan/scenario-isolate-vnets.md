---
title: 'Escenario: Aislamiento de redes virtuales'
titleSuffix: Azure Virtual WAN
description: Escenarios para enrutamiento - aislamiento de redes virtuales
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: b8a0a8093ab5f4d6c5e528bce592d5c029de30a7
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400048"
---
# <a name="scenario-isolating-vnets"></a>Escenario: Aislamiento de redes virtuales

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En este escenario, el objetivo es evitar que las redes virtuales puedan comunicarse entre sí. Esto se conoce como aislamiento de las redes virtuales. Para obtener información sobre el enrutamiento de centros virtuales, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Diseño

En este escenario, la carga de trabajo dentro de una red virtual concreta permanece aislada y no puede comunicarse con otras redes virtuales. Sin embargo, es necesario que las redes virtuales tengan acceso a todas las ramas (VPN, ER y VPN de usuario). Para averiguar el número de tablas de enrutamiento que va a necesitar, puede crear una matriz de conectividad. En el caso de este escenario, será similar a la tabla siguiente, donde cada celda representa si un origen (fila) puede comunicarse con un destino (columna):

| De |   A |  *Redes virtuales* | *Ramas* |
| -------------- | -------- | ---------- | ---|
| Redes virtuales     | &#8594;|           |     X    |
| Ramas   | &#8594;|    X     |     X    |

Cada una de las celdas de la tabla anterior describe si una conexión de Virtual WAN (el lado "From" del flujo, los encabezados de fila) aprende un prefijo de destino (el lado "To" del flujo, los encabezados de columna en cursiva) para un flujo de tráfico concreto, donde una "X" significa que la conectividad la proporciona Virtual WAN.

Esta matriz de conectividad nos proporciona dos patrones de fila diferentes, que se traducen en dos tablas de enrutamiento. Virtual WAN ya tiene una tabla de enrutamiento predeterminada, por lo que hará falta otra. En este ejemplo, se asignará el nombre **RT_VNET** a la tabla de enrutamiento.

Las redes virtuales se asociarán a la tabla de enrutamiento **RT_VNET**. Como necesitan conectividad con las ramas, estas necesitarán propagarse a **RT_VNET** (de lo contrario, las redes virtuales no aprenderían los prefijos de las ramas). Como las ramas siempre están asociadas a la tabla de enrutamiento predeterminada, las redes virtuales deberán propagarse a la tabla de enrutamiento predeterminada. En consecuencia, este es el diseño final:

* Redes virtuales:
  * Tabla de enrutamiento asociada: **RT_VNET**
  * Propagación a tablas de enrutamiento: **Valor predeterminado**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **RT_VNET** y **Valor predeterminado**

Tenga en cuenta que, puesto que las ramas son los únicos elementos que se propagan a la tabla de enrutamiento **RT_VNET**, serán los únicos prefijos que aprenderán las redes virtuales y no las de otras redes virtuales.

Para obtener información sobre el enrutamiento de centros virtuales, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Para poder configurar este escenario, tenga en cuenta los siguientes pasos:

1. Cree una tabla de enrutamiento personalizada en cada centro. En el ejemplo, la tabla de ruta es **RT_VNet**. Para crear una tabla de rutas, consulte [Configuración del enrutamiento de centro virtual](how-to-virtual-hub-routing.md). Para obtener más información sobre las tablas de rutas, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
2. Cuando cree la tabla de rutas **RT_VNet**, configure las siguientes opciones:

   * **Asociación**: Seleccione las redes virtuales que desea aislar.
   * **Propagación**: Seleccione la opción para las ramas, e indique que las conexiones de rama (VPN/ER/P2S) propagarán las rutas a esta tabla de rutas.

:::image type="content" source="./media/routing-scenarios/isolated/isolated-vnets.png" alt-text="Redes virtuales aisladas":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
