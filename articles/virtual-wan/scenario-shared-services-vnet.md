---
title: 'Escenario: Enrutamiento a redes virtuales de servicios compartidos'
titleSuffix: Azure Virtual WAN
description: 'Escenarios de enrutamiento: configure rutas para acceder a una red virtual de servicios compartidos con una carga de trabajo a la que desee que accedan todas las redes virtuales y ramas.'
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 08/07/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: c72ff14246f3f96980fe1de2fdef16794bf23e4e
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399997"
---
# <a name="scenario-route-to-shared-services-vnets"></a>Escenario: Enrutamiento a redes virtuales de servicios compartidos

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En este escenario, el objetivo es configurar rutas para acceder a una red virtual **Servicios compartidos** con la carga de trabajo a la que desee que accedan todas las redes virtuales y ramas (VPN/ER/P2S). Algunos ejemplos de estas cargas de trabajo compartidas podrían incluir máquinas virtuales con servicios como controladores de dominio o recursos compartidos de archivos, o servicios de Azure expuestos internamente mediante [puntos de conexión privados de Azure](../private-link/private-endpoint-overview.md).

Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Diseño

Podemos usar una matriz de conectividad para resumir los requisitos de este escenario. En la matriz, cada celda describe si una conexión de Virtual WAN (el lado de origen o "From" del flujo, los encabezados de fila de la tabla) aprende un prefijo de destino (el lado "To" del flujo, los encabezados de columna en cursiva de la tabla) para un flujo de tráfico concreto. Una "X" significa que la conectividad la proporciona Virtual WAN:

**Matriz de conectividad**

| From             | A:   |*Redes virtuales aisladas*|*Red virtual compartida*|*Ramas*|
|---|---|---|---|---|
|**Redes virtuales aisladas**|&#8594;|                |        X        |       X      |
|**Redes virtuales compartidas**  |&#8594;|       X        |        X        |       X      |
|**Ramas**      |&#8594;|       X        |        X        |       X      |

De forma similar al [escenario de redes virtuales aisladas](scenario-isolate-vnets.md), esta matriz de conectividad nos proporciona dos patrones de filas diferentes, que se traducen en dos tablas de rutas (las redes virtuales de servicios compartidos y las ramas tienen los mismos requisitos de conectividad). Virtual WAN ya tiene una tabla de rutas predeterminada, por lo que se necesitará otra tabla de rutas personalizada, a la que se llamará **RT_SHARED** en este ejemplo.

Las redes virtuales se asociarán a la tabla de enrutamiento **RT_SHARED**. Dado que necesitan conectividad a las bifurcaciones y a las redes virtuales de los servicios compartidos, la red virtual y las ramas del servicio compartido deberán propagarse a **RT_SHARED** (de lo contrario, las redes virtuales no aprenderán los prefijos de la rama y de la red virtual compartida). Dado que las ramas siempre están asociadas a la tabla de rutas predeterminada y los requisitos de conectividad son los mismos para las redes virtuales de servicios compartidos, asociaremos también las redes virtuales del servicio compartido a la tabla de rutas predeterminada.

En consecuencia, este es el diseño final:

* Redes virtuales aisladas:
  * Tabla de enrutamiento asociada: **RT_SHARED**
  * Propagación a tablas de enrutamiento: **Valor predeterminado**
* Redes virtuales de servicios compartidos:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **RT_SHARED** y **Predeterminado**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **RT_SHARED** y **Predeterminado**

> [!NOTE]
> Si Virtual WAN está implementada en varias regiones, tendrá que crear la tabla de rutas **RT_SHARED** en cada centro de conectividad y las rutas de cada red virtual de servicios compartidos y la conexión de rama deben propagarse a las tablas de rutas de cada centro de conectividad virtual mediante etiquetas de propagación.

Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Para configurar el escenario, tenga en cuenta los siguientes pasos:

1. Identifique la red virtual **Shared Services** (Servicios compartidos).
2. Cree una tabla de rutas personalizada. En el ejemplo, hacemos referencia a la tabla de rutas como **RT_SHARED**. Para conocer los pasos para crear una tabla de rutas, consulte [Configuración del enrutamiento de centro virtual](how-to-virtual-hub-routing.md). Use los valores siguientes como guía:

   * **Asociación**
     * Para **Redes virtuales, *excepto* la red virtual de servicios compartidos**, seleccione las redes virtuales que desea aislar. Esto hará que todas estas redes virtuales (excepto la red virtual de servicios compartidos) puedan alcanzar el destino en función de las rutas de la tabla de rutas RT_SHARED.

   * **Propagación**
      * En el caso de **Ramas**, propague las rutas a esta tabla de rutas, además de a cualquier otra tabla de rutas que ya haya seleccionado. Debido a este paso, la tabla de rutas RT_SHARED aprenderá las rutas de todas las conexiones de rama (VPN/ER/VPN de usuario).
      * En **Redes virtuales**, seleccione la **red virtual de servicios compartidos**. Debido a este paso, la tabla de rutas RT_SHARED aprenderá las rutas de la conexión de la red virtual de servicios compartidos.

Esto dará lugar a la configuración de enrutamiento que se muestra en la ilustración siguiente:

   :::image type="content" source="./media/routing-scenarios/shared-service-vnet/shared-services.png" alt-text="Red virtual de servicios compartidos" lightbox="./media/routing-scenarios/shared-service-vnet/shared-services.png":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
