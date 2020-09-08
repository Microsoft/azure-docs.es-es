---
title: 'Escenario: Enrutamiento del tráfico a través de una aplicación virtual de red (NVA)'
titleSuffix: Azure Virtual WAN
description: Enrutamiento del tráfico a través de una aplicación virtual de red
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 6b62f8c33c73ded978c0c2e3a8c3b7fadea49c96
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852084"
---
# <a name="scenario-route-traffic-through-an-nva"></a>Escenario: Enrutamiento del tráfico a través de una aplicación virtual de red

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En este escenario de NVA, el objetivo consiste en enrutar el tráfico a través de una NVA (aplicación virtual de red) para rama a red virtual y red virtual a rama. Para obtener información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

> [!NOTE]
> Si ya tiene una configuración con rutas anteriores a las nuevas funcionalidades, [Configuración del enrutamiento de centro virtual](how-to-virtual-hub-routing.md) ya está disponible, siga los pasos de estas versiones de los artículos:
>* [Artículo de Azure Portal](virtual-wan-route-table-nva-portal.md)
>* [Artículo de PowerShell](virtual-wan-route-table-nva.md)
>

## <a name="design"></a><a name="design"></a>Diseño

En este escenario, se usará la convención de nomenclatura:

* "VNet de NVA" para redes virtuales en las que los usuarios han implementado una NVA y que han conectado otras redes virtuales como radios (VNet 2 y VNet 4 en la **matriz de conectividad** a continuación).
* "Radios de NVA" para las redes virtuales conectadas a una red virtual de NVA (VNet 5, VNet 6, VNet 7 y VNet 8 en la **matriz de conectividad** a continuación).
* "VNet no de NVA" para redes virtuales conectadas a Virtual WAN que no tienen una NVA u otras redes virtuales emparejadas con ellas (VNet 1 y VNet 3 en la **matriz de conectividad** a continuación).
* "Centros" para los centros de Virtual WAN administrados por Microsoft, a los que se conectan las redes virtuales de NVA. No es necesario que las redes virtuales de radios de NVA estén conectadas a centros de Virtual WAN, solo a redes virtuales de NVA.

En la siguiente matriz de conectividad se resumen los flujos admitidos en este escenario:

**Matriz de conectividad**

| From             | A:|   *Radios de NVA*|*VNet de NVA*|*VNet no de NVA*|*Ramas*|
|---|---|---|---|---|---|
| **Radios de NVA**   | &#8594; | 0/0 UDR  |  Emparejamiento |   0/0 UDR    |  0/0 UDR  |
| **VNet de NVA**    | &#8594; |   estática |      X   |        X     |      X    |
| **VNet no de NVA**| &#8594; |   estática |      X   |        X     |      X    |
| **Ramas**     | &#8594; |   estática |      X   |        X     |      X    |

Cada una de las celdas de la matriz de conectividad describe si una conexión de Virtual WAN (el lado "De" del flujo, los encabezados de fila de la tabla) aprende un prefijo de destino (el lado "A" del flujo, los encabezados de columna en cursiva de la tabla) para un flujo de tráfico concreto. Tenga en cuenta lo siguiente.

* Los radios de NVA no están administrados por Virtual WAN. Como consecuencia, el usuario mantiene los mecanismos con los que se comunicarán con otras redes virtuales o ramas. La conectividad a la red virtual de NVA se proporciona mediante un emparejamiento de VNet, y una ruta predeterminada a 0.0.0.0/0 que apunta a la NVA como próximo salto debe cubrir la conectividad a Internet, a otros radios y a las ramas.
* Las redes virtuales de NVA sabrán sobre sus propios radios de NVA, pero no sobre los radios de NVA conectados a otras redes virtuales de NVA. Por ejemplo, en la tabla 1, VNet 2 sabe sobre VNet 5 y VNet 6, pero no sobre otros radios, como VNet 7 y VNet 8. Se requiere una ruta estática para insertar los prefijos de otros radios en redes virtuales de NVA
* Del mismo modo, las ramas y las redes virtuales que no son de NVA no sabrán sobre ningún radio de NVA, ya que los radios de NVA no están conectados a centros de Virtual WAN. Como consecuencia, aquí también se necesitarán rutas estáticas.

Teniendo en cuenta que los radios de NVA no están administrados por Virtual WAN, todas las demás filas muestran el mismo patrón de conectividad. Por lo tanto, una sola tabla de rutas (la predeterminada) hará lo siguiente:

* Redes virtuales (VNet no de centros y Vnet de usuario-centro):
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **Valor predeterminado**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de enrutamiento: **Valor predeterminado**

Sin embargo, en este escenario debemos pensar en qué rutas estáticas se deben configurar. Cada ruta estática tendrá dos componentes, una parte en el centro de Virtual WAN que indica a los componentes de Virtual WAN qué conexión usar para cada radio, y otra en esa conexión específica que apunta a la dirección IP concreta asignada a la NVA (o a un equilibrador de carga delante de varias NVA), como se muestra en la **Ilustración 1**:

**Ilustración 1**

:::image type="content" source="media/routing-scenarios/nva/nva-static-concept.png" alt-text="Ilustración 1":::

Con esto, las rutas estáticas que necesitamos en la tabla predeterminada para enviar tráfico a los radios de NVA detrás de la red virtual de NVA son las siguientes:

| Descripción | Tabla de rutas | Ruta estática              |
| ----------- | ----------- | ------------------------- |
| Red virtual 2       | Valor predeterminado     | 10.2.0.0/16 -> eastusconn |
| VNet 4       | Valor predeterminado     | 10.4.0.0/16 -> weconn     |

Ahora, Virtual WAN sabe a qué conexión enviar los paquetes, pero la conexión debe saber qué hacer al recibir esos paquetes: Aquí es donde se utilizan las tablas de rutas de conexión. Aquí usaremos los prefijos más cortos (/24 en lugar del más largo /16) para asegurar que estas rutas tengan preferencia sobre las rutas que se importan desde las redes virtuales de NVA (VNet 2 y VNet 4):

| Descripción | Conexión | Ruta estática            |
| ----------- | ---------- | ----------------------- |
| VNet 5       | eastusconn | 10.2.1.0/24 -> 10.2.0.5 |
| VNet 6       | eastusconn | 10.2.2.0/24 -> 10.2.0.5 |
| VNet 7       | weconn     | 10.4.1.0/24 -> 10.4.0.5 |
| VNet 8       | weconn     | 10.4.2.0/24 -> 10.4.0.5 |

Ahora las redes virtuales de NVA, las redes virtuales que no son de NVA y las ramas saben cómo comunicarse con todos los radios de NVA. Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="architecture"></a><a name="architecture"></a>Arquitectura

En la **Ilustración 2**, hay dos centros: **Hub 1** y **Hub 2**.

* **Hub 1** y **Hub 2** están conectados directamente a las redes virtuales de NVA **VNet 2** y **VNet 4**.

* **VNet 5** y **VNet 6** están emparejadas con **VNet 2**.

* **VNet 7** y **VNet 8** están emparejadas con **VNet 4**.

* **VNet 5, 6, 7 y 8** son radios indirectos, ya que no están conectadas directamente a un centro virtual.

**Ilustración 2**

:::image type="content" source="./media/routing-scenarios/nva/nva.png" alt-text="Ilustración 2" lightbox="./media/routing-scenarios/nva/nva.png":::

## <a name="scenario-workflow"></a><a name="workflow"></a>Flujo de trabajo del escenario

Para configurar el enrutamiento a través de NVA, estos son los pasos que debe tener en cuenta:

1. Identifique la conexión de red virtual de radio de NVA. En la **Ilustración 2**, son la **conexión VNet 2 (eastusconn)** y la **conexión VNet 4 (weconn)** .

   Asegúrese de que se han configurado rutas definidas por el usuario:
   * De VNet 5 y VNet 6 a la IP de NVA de VNet 2
   * De VNet 7 y VNet 8 a la IP de NVA de VNet 4 
   
   No es necesario conectar VNet 5, 6, 7 y 8 a los centros virtuales directamente. Asegúrese de que los NSG de VNet 5, 6, 7 y 8 permitan el tráfico para la rama (VPN/ER/P2S) o las redes virtuales conectadas a sus redes virtuales remotas. Por ejemplo, VNet 5 y 6 deben garantizar que los NSG permitan el tráfico para los prefijos de direcciones locales y para VNet 7 y 8, conectadas al Hub 2 remoto.

2. Incorpore una entrada de ruta estática agregada para VNet 2, 5 y 6 a la tabla de rutas predeterminada de Hub 1.

   :::image type="content" source="./media/routing-scenarios/nva/nva-static-expand.png" alt-text="Ejemplo":::

3. Configure una ruta estática para VNet 5 y 6 en la conexión de red virtual de VNet 2. Para establecer la configuración de enrutamiento para una conexión de red virtual, vea [Enrutamiento de centros virtuales](how-to-virtual-hub-routing.md#routing-configuration).

4. Incorpore una entrada de ruta estática agregada para VNet 4, 7 y 8 a la tabla de rutas predeterminada de Hub 1.

5. Repita los pasos 2, 3 y 4 para la tabla de rutas predeterminada de Hub 2.

Esto hará que la configuración de enrutamiento cambie como se muestra en la **Ilustración 3** a continuación.

**Ilustración 3**

   :::image type="content" source="./media/routing-scenarios/nva/nva-result.png" alt-text="Ilustración 3" lightbox="./media/routing-scenarios/nva/nva-result.png":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
