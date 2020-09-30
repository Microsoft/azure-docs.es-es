---
title: 'Escenario: Enrutamiento del tráfico a través de NVA mediante una configuración personalizada'
titleSuffix: Azure Virtual WAN
description: Este escenario le ayuda a enrutar el tráfico a través de las NVA con una NVA diferente para el tráfico vinculado a Internet.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: f233aedf8b51967264994f5a4081f8f4cd99df01
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400014"
---
# <a name="scenario-route-traffic-through-nvas---custom-preview"></a>Escenario: Enrutamiento del tráfico a través de NVA: personalizado (versión preliminar)

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En este escenario de NVA (aplicación virtual de red), el objetivo es enrutar el tráfico a través de una NVA para la comunicación entre redes virtuales y ramas, y usar otra NVA para el tráfico vinculado a Internet. Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Diseño

En este escenario, se usará la convención de nomenclatura:

* "VNet de servicio" para las redes virtuales en las que los usuarios han implementado una NVA (VNet 4 en la **Figura 1**) para inspeccionar el tráfico que no procede de Internet.
* "VNet perimetral" para las redes virtuales en las que los usuarios han implementado una NVA que se usará para inspeccionar el tráfico enlazado a Internet (VNet 5 en la **Figura 1**).
* "Radios de NVA" para las redes virtuales conectadas a una red virtual de NVA (VNet 1, VNet 2 y VNet 3 en la **Figura 1**).
* "Concentradores" para los concentradores de WAN virtual administrados por Microsoft.

En la siguiente matriz de conectividad se resumen los flujos admitidos en este escenario:

**Matriz de conectividad**

| From          | A:|*Radios de NVA*|*VNet de servicio*|*VNet perimetral*|*Ramas estáticas*|
|---|---|---|---|---|---|
| **Radios de NVA**| &#8594;|      X |            X |   Emparejamiento |    estática    |
| **VNet de servicio**| &#8594;|    X |            X |      X    |      X       |
| **VNet perimetral** | &#8594;|       X |            X |      X    |      X       |
| **Ramas** | &#8594;|  estática |            X |      X    |      X       |

En cada una de las celdas de la matriz de conectividad se describe si una conexión de Virtual WAN (el lado "De" del flujo, los encabezados de fila) conoce un prefijo de destino (el lado "A" del flujo, los encabezados de columna en cursiva) de un flujo de tráfico concreto. Una "X" significa que la conectividad se proporciona de forma nativa por Virtual WAN y "estática" significa que la conectividad se proporciona mediante Virtual WAN mediante rutas estáticas. Vamos a profundizar en las diferentes filas:

* Radios de NVA:
  * los radios llegarán a otros radios directamente a través de los concentradores de Virtual WAN.
  * Los radios obtendrán conectividad a las ramas a través de una ruta estática que apunta a la VNet de servicio. No conocerán los prefijos específicos de las ramas (de lo contrario, serían más específicos e invalidarían el resumen).
  * Los radios enviarán el tráfico de Internet a la VNet perimetral mediante un emparejamiento de VNet directo.
* Ramas:
  * las ramas llegarán a los radios a través de un enrutamiento estático que apunta a la VNet de servicio. No conocerán los prefijos específicos de las redes virtuales que invalidan la ruta estática resumida.
* La VNet de servicio será similar a una red virtual de servicios compartidos que debe ser accesible desde todas las redes virtuales y todas las ramas.
* En realidad, la VNet perimetral no necesita tener conectividad a través de Virtual WAN, ya que el único tráfico que admitirá será a través de los emparejamientos de VNet directos. Sin embargo, se usará el mismo modelo de conectividad que para la VNet perimetral con el fin de simplificar la configuración.

Por lo tanto, nuestra matriz de conectividad nos ofrece tres patrones de conectividad distintos, lo que se traduce en tres tablas de rutas. Las asociaciones a las distintas redes virtuales serán las siguientes:

* Radios de NVA:
  * Tabla de enrutamiento asociada: **RT_V2B**
  * Propagación a tablas de enrutamiento: **RT_V2B** y **RT_SHARED**
* Redes virtuales de NVA (internas e Internet):
  * Tabla de enrutamiento asociada: **RT_SHARED**
  * Propagación a tablas de enrutamiento: **RT_SHARED**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **RT_SHARED** y **Predeterminada**

Se necesitan estas rutas estáticas para tener la seguridad de que el tráfico de la red virtual a la rama y de la rama a la red virtual pasa por la NVA en la VNet de servicio (VNet 4):

| Descripción | Tabla de rutas | Ruta estática              |
| ----------- | ----------- | ------------------------- |
| Ramas    | RT_V2B      | 10.2.0.0/16-> vnet4conn  |
| Radios de NVA  | Valor predeterminado     | 10.1.0.0/16 -> vnet4conn  |

Ahora, Virtual WAN sabe a qué conexión enviar los paquetes, pero la conexión debe saber qué hacer al recibir esos paquetes: Aquí es donde se utilizan las tablas de rutas de conexión.

| Descripción | Conexión | Ruta estática            |
| ----------- | ---------- | ----------------------- |
| VNet2Branch | vnet4conn  | 10.2.0.0/16 -> 10.4.0.5 |
| Branch2VNet | vnet4conn  | 10.1.0.0/16 -> 10.4.0.5 |

En este momento, todo debe estar en su lugar.

Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="architecture"></a><a name="architecture"></a>Arquitectura

En la **ilustración 1**, hay un centro **Hub 1**.

* El **Concentrador 1** está conectado directamente a las VNet de NVA **VNet 4** y **VNet 5**.

* Se espera que el tráfico entre las VNet 1, 2 y 3 y las ramas (VPN/ER/P2S) pase por la **NVA VNet 4** 10.4.0.5.

* Se espera que todo el tráfico vinculado a Internet desde las VNet 1, 2 y 3 pase por la **NVA VNet 5** 10.5.0.5.

**Ilustración 1**

:::image type="content" source="./media/routing-scenarios/nva-custom/figure-1.png" alt-text="Ilustración 1":::

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Para configurar el enrutamiento a través de NVA, estos son los pasos que debe tener en cuenta:

1. Para que el tráfico vinculado a Internet pase por la VNet 5, es necesario que las VNet 1, 2 y 3 se conecten directamente a la VNet 5 a través del emparejamiento de VNet. También necesita un UDR configurado en las redes virtuales para 0.0.0.0/0 y el próximo salto 10.5.0.5. Actualmente, Virtual WAN no permite una NVA de próximo salto en el centro virtual para 0.0.0.0/0.

1. En Azure Portal, navegue hasta el centro virtual y cree una tabla de rutas personalizada **RT_Shared** que aprenderá las rutas mediante la propagación desde todas las redes virtuales y las conexiones de rama. En la **ilustración 2**, esto se representa como una tabla de ruta personalizada **RT_Shared** vacía.

   * **Rutas:** No es necesario agregar ninguna ruta estática.

   * **Asociación:** seleccione las VNet 4 y 5, lo que significa que las conexiones de estas redes se asociarán a la tabla de rutas **RT_Shared**.

   * **Propagación:** Como desea que todas las ramas y conexiones de red virtual propaguen sus rutas dinámicamente a esta tabla de rutas, seleccione las ramas y todas las redes virtuales.

1. Cree una tabla de rutas **RT_V2B** personalizada para dirigir el tráfico de las VNet 1, 2 y 3 a las ramas.

   * **Rutas:** Agregue una entrada de ruta estática agregada para las ramas (VPN/ER/P2S) (10.2.0.0/16 en la **Figura 2**) con el próximo salto como conexión de VNet 4. También debe configurar una ruta estática en la conexión de la VNet 4 para los prefijos de rama, e indicar que el próximo salto sea la dirección IP específica de la NVA en la VNet 4.

   * **Asociación:** seleccione las VNet 1, 2 y 3. Esto hará que las conexiones de red virtual 1, 2 y 3 se asociarán a esta tabla de rutas y podrán aprender las rutas (estáticas y dinámicas mediante propagación) en esta tabla de rutas.

   * **Propagación:** Las conexiones propagan las rutas a las tablas de rutas. Al seleccionar las VNet 1, 2 y 3, se habilitará la propagación de rutas desde estas redes virtuales hasta esta tabla de rutas. No es necesario propagar las rutas desde las conexiones de rama a RT_V2B, ya que el tráfico de la red virtual de rama pasa por la NVA en la VNet 4.
  
1. Edite la tabla de rutas **DefaultRouteTable** predeterminada.

   Todas las conexiones VPN, ExpressRoute y VPN de usuario están asociadas a la tabla de rutas predeterminada. Todas las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al mismo conjunto de tablas de rutas.

   * **Rutas:** agregue una entrada de ruta estática agregada para las VNet 1, 2 y 3 (10.1.0.0/16 en la **Figura 2**) con el próximo salto como conexión de VNet 4. También debe configurar una ruta estática en la conexión de VNet 4 para los prefijos agregados de las VNet 1, 2 y 3, e indicar que el próximo salto sea la dirección IP específica de la NVA en la VNet 4.

   * **Asociación:** Asegúrese de que la opción para las ramas (VPN/ER/P2S) esté seleccionada, y asegúrese de que las conexiones de rama locales estén asociadas a *defaultroutetable*.

   * **Propagación:** Asegúrese de que la opción para las ramas (VPN/ER/P2S) esté seleccionada y asegúrese de que las conexiones locales propaguen las rutas a *defaultroutetable*.

**Ilustración 2**

:::image type="content" source="./media/routing-scenarios/nva-custom/figure-2.png" alt-text="Ilustración 2" lightbox="./media/routing-scenarios/nva-custom/figure-2.png":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
