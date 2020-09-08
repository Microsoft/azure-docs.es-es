---
title: Uso compartido de un servicio Private Link a través Virtual WAN
titleSuffix: Azure Virtual WAN
description: Pasos para configurar Private Link en Virtual WAN
services: virtual-wan
author: erjosito
ms.service: virtual-wan
ms.topic: how-to
ms.date: 08/18/2020
ms.author: jomore
ms.custom: fasttrack-new
ms.openlocfilehash: 98142e3a8904bcbb0352fa768fc72966412dae0b
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590527"
---
# <a name="use-private-link-in-virtual-wan"></a>Uso de Private Link en Virtual WAN

[Azure Private Link](../private-link/private-link-overview.md) es una tecnología que le permite conectar ofertas de plataforma como servicio de Azure mediante la conectividad de direcciones IP privadas exponiendo [puntos de conexión privados](../private-link/private-endpoint-overview.md). Con Azure Virtual WAN, puede implementar un punto de conexión privado en una de las redes virtuales conectadas a cualquier concentrador virtual. Esto proporciona conectividad a cualquier otra red virtual o rama conectada a la misma Virtual WAN.

## <a name="before-you-begin"></a>Antes de empezar

En los pasos de este artículo se da por hecho que ya ha implementado una instancia de Virtual WAN con uno o más concentradores, así como al menos dos redes virtuales conectadas a Virtual WAN.

Para crear una nueva WAN virtual y un nuevo centro, siga los pasos que se describen en los siguientes artículos:

* [Creación de una instancia de Virtual WAN](virtual-wan-site-to-site-portal.md#openvwan)
* [Creación de un centro de conectividad](virtual-wan-site-to-site-portal.md#hub)
* [Conectar una red virtual a un concentrador](virtual-wan-site-to-site-portal.md#hub)

## <a name="create-a-private-link-endpoint"></a><a name="endpoint"></a>Creación de un punto de conexión de vínculo privado

Puede crear un punto de conexión de vínculo privado para muchos servicios diferentes. En este ejemplo, usaremos Azure SQL Database. Puede encontrar más información sobre cómo crear un punto de conexión privado para una instancia de Azure SQL Database en [Inicio rápido: Creación de un punto de conexión privado mediante Azure Portal](../private-link/create-private-endpoint-portal.md). La siguiente imagen muestra la configuración de red de Azure SQL Database:

:::image type="content" source="./media/howto-private-link/create-private-link.png" alt-text="Creación de un vínculo privado" lightbox="./media/howto-private-link/create-private-link.png":::

Después de crear la instancia de Azure SQL Database, puede comprobar que la dirección IP del punto de conexión privado examine los puntos de conexión privados:

:::image type="content" source="./media/howto-private-link/endpoints.png" alt-text="Puntos de conexión privados" lightbox="./media/howto-private-link/endpoints.png":::

Al hacer clic en el punto de conexión privado que hemos creado, debería ver su dirección IP privada, así como su nombre de dominio completo (FQDN). Tenga en cuenta que el punto de conexión privado tiene una dirección IP en el intervalo de la red virtual donde se ha implementado (10.1.3.0/24):

:::image type="content" source="./media/howto-private-link/sql-endpoint.png" alt-text="Punto de conexión de SQL" lightbox="./media/howto-private-link/sql-endpoint.png":::

## <a name="verify-connectivity-from-the-same-vnet"></a><a name="connectivity"></a>Comprobación de la conectividad de la misma red virtual

En este ejemplo, se comprobará la conectividad a la instancia de Azure SQL Database desde una máquina virtual de Ubuntu con las herramientas de MS SQL instaladas. El primer paso consiste en comprobar que la resolución de DNS funciona y que el nombre de dominio completo de Azure SQL Database se resuelve en una dirección IP privada, en la misma red virtual donde se ha implementado el punto de conexión privado (10.1.3.0/24):

```bash
$ nslookup wantest.database.windows.net
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
wantest.database.windows.net    canonical name = wantest.privatelink.database.windows.net.
Name:   wantest.privatelink.database.windows.net
Address: 10.1.3.228
```

Como puede ver en la salida anterior, el nombre de dominio completo `wantest.database.windows.net` está asignado a `wantest.privatelink.database.windows.net` y que la zona DNS privada creada a lo largo del punto de conexión privado se resolverá en la dirección IP privada `10.1.3.228`. Al examinar la zona DNS privada, se confirmará que hay un registro A para el punto de conexión privado asignado a la dirección IP privada:

:::image type="content" source="./media/howto-private-link/dns-zone.png" alt-text="Zona DNS" lightbox="./media/howto-private-link/dns-zone.png":::

Después de comprobar la resolución de DNS correcta, podemos intentar conectar a la base de datos:

```bash
$ query="SELECT CONVERT(char(15), CONNECTIONPROPERTY('client_net_address'));"
$ sqlcmd -S wantest.database.windows.net -U $username -P $password -Q "$query"

10.1.3.75
```

Como puede ver, usamos una consulta SQL especial que nos proporciona la dirección IP de origen que SQL Server ve desde el cliente. En este caso, el servidor ve el cliente con su dirección IP privada (`10.1.3.75`), lo que significa que el tráfico no viaja a través de la red pública de Internet, sino que se dirige directamente al punto de conexión privado.

Tenga en cuenta que debe establecer las variables `username` y `password` para que coincidan con las credenciales definidas en la instancia de Azure SQL Database para que los ejemplos de esta guía funcionen.

## <a name="connect-from-a-different-vnet"></a><a name="vnet"></a>Conexión desde otra red virtual

Ahora que una red virtual de Azure Virtual WAN tiene conectividad con el punto de conexión privado, todas las demás redes virtuales y ramas conectadas a Virtual WAN también pueden tener acceso a ella. Debe proporcionar conectividad a través de cualquiera de los modelos admitidos por la Azure Virtual WAN, como el [escenario universal](scenario-any-to-any.md) o el [escenario de la red virtual de servicios compartidos](scenario-shared-services-vnet.md), por mencionar dos ejemplos.

Cuando tenga conectividad entre la red virtual o la rama en la red virtual donde se ha implementado el punto de conexión privado, deberá configurar la resolución de DNS:

* Si se conecta al punto de conexión privado desde una red virtual, puede usar la misma zona privada que se creó con la instancia de Azure SQL Database.
* Si se conecta al punto de conexión privado desde una rama (VPN de sitio a sitio, VPN de punto a sitio o ExpressRoute), debe usar la resolución DNS local.

En este ejemplo, se va a conectar desde otra red virtual, por lo que primero adjuntaremos la zona DNS privada a la nueva red virtual para que las cargas de trabajo puedan resolver el nombre de dominio completo de Azure SQL Database en la dirección IP privada. Esto se hace a través de la vinculación de la zona DNS privada a la nueva red virtual:

:::image type="content" source="./media/howto-private-link/dns-link.png" alt-text="Vínculo de DNS" lightbox="./media/howto-private-link/dns-link.png":::

Ahora, todas las máquinas virtuales de la red virtual conectada deberían resolver correctamente el nombre de dominio completo de Azure SQL Database en la dirección IP privada del vínculo privado:

```bash
$ nslookup wantest.database.windows.net
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
wantest.database.windows.net    canonical name = wantest.privatelink.database.windows.net.
Name:   wantest.privatelink.database.windows.net
Address: 10.1.3.228
```

Para comprobar de nuevo que esta red virtual (10.1.1.0/24) tiene conectividad con la red virtual original en la que se configuró el punto de conexión privado (10.1.3.0/24), puede comprobar la tabla de rutas efectiva en cualquier máquina virtual de la red virtual:

:::image type="content" source="./media/howto-private-link/effective-routes.png" alt-text="Rutas efectivas" lightbox="./media/howto-private-link/effective-routes.png":::

Como puede ver, hay una ruta que apunta a la red virtual 10.1.3.0/24 inyectada por las puertas de enlace de red virtual de Azure Virtual WAN. Ahora ya podemos probar la conectividad a la base de datos:

```bash
$ query="SELECT CONVERT(char(15), CONNECTIONPROPERTY('client_net_address'));"
$ sqlcmd -S wantest.database.windows.net -U $username -P $password -Q "$query"

10.1.1.75
```

En este ejemplo, hemos visto que la creación de un punto de conexión privado en una de las redes virtuales conectadas a una instancia de Virtual WAN proporciona conectividad al resto de redes virtuales y ramas de dicha instancia.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
