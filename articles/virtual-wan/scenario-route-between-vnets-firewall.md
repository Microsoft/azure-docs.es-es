---
title: 'Escenario: Enrutamiento personalizado de Azure Firewall para Virtual WAN'
titleSuffix: Azure Virtual WAN
description: 'Escenarios de enrutamiento: enrutar el tráfico entre redes virtuales directamente, pero usar Azure Firewall para flujos de tráfico de redes virtuales a Internet o rama y de rama a redes virtuales'
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 7a344b9c1383976cfe1b7507c120e19221f3555f
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88589357"
---
# <a name="scenario-azure-firewall---custom"></a>Escenario: Azure Firewall: personalizado

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En este escenario, el objetivo es enrutar el tráfico entre redes virtuales directamente, pero usar Azure Firewall para flujos de tráfico de redes virtuales a Internet o rama y de rama a redes virtuales.

## <a name="design"></a><a name="design"></a>Diseño

Para averiguar el número de tablas de rutas que se necesitarán, puede crear una matriz de conectividad, donde cada celda representa si un origen (fila) puede comunicarse con un destino (columna). La matriz de conectividad de este escenario es trivial, pero aún se puede observar para que sea coherente con otros escenarios.

**Matriz de conectividad**

| From           | A:      | *Redes virtuales*      | *Ramas*    | *Internet*   |
|---             |---       |---           |---            |---           |
| **Redes virtuales**      |   &#8594;|     X        |     AzFW      |     AzFW     |
| **Ramas**   |   &#8594;|    AzFW      |       X       |       X      |

En la tabla anterior, una "X" representa la conectividad directa entre dos conexiones sin que el tráfico atraviese Azure Firewall en Virtual WAN y "AzFW" indica que el flujo pasará por Azure Firewall. Dado que hay dos patrones de conectividad distintos en la matriz, se necesitarán dos tablas de rutas que se configurarán de la siguiente manera:

* Redes virtuales:
  * Tabla de rutas asociada: **RT_VNet**
  * Propagación a tablas de rutas: **RT_VNet**
* Ramas:
  * Tabla de rutas asociada: **Valor predeterminado**
  * Propagación a tablas de rutas: **Valor predeterminado**

> [!NOTE]
> Puede crear una instancia de Virtual WAN independiente con un solo centro virtual protegido en cada región y, a continuación, conectar cada instancia de Virtual WAN entre sí a través de VPN de sitio a sitio.

Para obtener información sobre el enrutamiento de centros virtuales, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

En este escenario, desea enrutar el tráfico a través de Azure Firewall para el tráfico de red virtual a Internet, de red virtual a rama o de rama a red virtual, pero desearía pasar directamente al tráfico de red virtual a red virtual. Si usó Azure Firewall Manager, la configuración de enrutamiento se rellena automáticamente en la **Tabla de enrutamiento predeterminada**. El tráfico privado se aplica a la red virtual y las ramas, el tráfico de Internet se aplica a 0.0.0.0/0.

Las conexiones VPN, ExpressRoute y VPN de usuario se denominan colectivamente ramas y están asociadas a la misma tabla de rutas (predeterminada). Todas las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al mismo conjunto de tablas de rutas. Para poder configurar este escenario, tenga en cuenta los siguientes pasos:

1. Cree una tabla de enrutamiento personalizada **RT_VNet**.
1. Cree una ruta para activar el tráfico de red virtual a Internet y de red virtual a rama: 0.0.0.0/0 con el próximo salto que apunta a Azure Firewall. En la sección Propagación, se asegurará de que las redes virtuales estén seleccionadas, lo que garantizará rutas más específicas y permitirá el flujo de tráfico directo de red virtual a red virtual.

   * En **Asociación:** Seleccione redes virtuales que impliquen que las redes virtuales llegarán al destino según las rutas de esta tabla de enrutamiento.
   * En **Propagación:** Seleccione redes virtuales que impliquen que las redes virtuales se propagan a esta tabla de enrutamiento. En otras palabras, las rutas más específicas se propagarán a esta tabla de enrutamiento, lo que garantiza un flujo de tráfico directo de red virtual a red virtual.

1. Agregue una ruta estática agregada para redes virtuales en la **tabla de enrutamiento predeterminada** para activar el flujo de rama a red virtual a través de Azure Firewall.

   * Recuerde que las ramas están asociadas y se propagan a la tabla de enrutamiento predeterminada.
   * Las ramas no se propagan a la tabla de rutas RT_VNet. Esto garantiza el flujo de tráfico de red virtual a rama a través de Azure Firewall.

Esto hará que la configuración de enrutamiento cambie como se muestra en la **Figura 1**.

**Ilustración 1**

:::image type="content" source="./media/routing-scenarios/between-vnets-firewall/routing.png" alt-text="Ilustración 1":::

> [!NOTE]
> Los centros de Virtual WAN y las redes virtuales conectadas deben estar en la misma región de Azure.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
* Para más información sobre cómo configurar el enrutamiento de centros virtuales, consulte [Configuración del enrutamiento de centro virtual](how-to-virtual-hub-routing.md).
