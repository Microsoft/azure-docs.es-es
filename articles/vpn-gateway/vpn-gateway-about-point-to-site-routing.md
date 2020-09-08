---
title: 'Azure VPN Gateway: Acerca del enrutamiento de P2S'
description: Aprenda sobre el enrutamiento de VPN de punto a sitio de Azure para diferentes sistemas operativos, protocolos de acceso remoto y configuraciones de red virtual.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 03/24/2020
ms.author: kumud
ms.openlocfilehash: 54b839170d95918889d1295eb20308688fbfb2a7
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88032380"
---
# <a name="about-point-to-site-vpn-routing"></a>Información sobre el enrutamiento de VPN de punto a sitio

Este artículo lo ayudará a comprender el comportamiento del enrutamiento de VPN de punto a sitio de Azure. El comportamiento del enrutamiento de VPN de punto a sitio depende del SO cliente, el protocolo que se usa para la conexión de VPN y cómo las redes virtuales se conectan entre sí.

Actualmente, Azure admite dos protocolos para el acceso remoto, IKEv2 y SSTP. El protocolo IKEv2 es compatible con muchos sistemas operativos cliente, como Windows, Linux, MacOS, iOS y Android. El protocolo SSTP solo es compatible con Windows. Si realiza un cambio en la topología de la red y tiene clientes VPN de Windows, el paquete de cliente VPN para clientes de Windows se debe descargar e instalar nuevamente para que los cambios se apliquen en el cliente.

> [!NOTE]
> Este artículo solo se aplica al protocolo IKEv2.
>

## <a name="about-the-diagrams"></a><a name="diagrams"></a>Acerca de los diagramas

Hay varios diagramas distintos en este artículo. Cada sección muestra una configuración o topología distinta. En este artículo, las conexiones de sitio a sitio (S2S) y las conexiones de red virtual a red virtual funcionan del mismo modo, debido a que ambas son túneles IPsec. Todas las puertas de enlace de VPN se basan en rutas.

## <a name="one-isolated-vnet"></a><a name="isolatedvnet"></a>Una red virtual aislada

La conexión de puerta de enlace de VPN de punto a sitio en este ejemplo es para una red virtual que no está conectada ni emparejada con ninguna otra red virtual (VNet1). En este ejemplo, los clientes pueden acceder a VNet1.

![enrutamiento de red virtual aislado](./media/vpn-gateway-about-point-to-site-routing/1.jpg "enrutamiento de red virtual aislado")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows pueden acceder a VNet1

* Los clientes no Windows pueden acceder a VNet1

## <a name="multiple-peered-vnets"></a><a name="multipeered"></a>Varias redes virtuales emparejadas

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 está emparejada con VNet2. VNet 2 está emparejada con VNet3. VNet1 está emparejada con VNet4. No hay emparejamiento directo entre VNet1 y VNet3. VNet1 tiene habilitado "Permitir tránsito de puerta de enlace" y VNet2 y VNet4 tienen "Usar puertas de enlace remotas".

Los clientes que usan Windows pueden acceder directamente a redes virtuales emparejadas, pero el cliente VPN se debe descargar de nuevo si se hace algún cambio en el emparejamiento de VNet o en la topología de red. Los clientes no Windows pueden acceder directamente a las redes virtuales emparejadas. El acceso no es transitivo y está limitado solo a las redes virtuales emparejadas directamente.

![varias redes virtuales emparejadas](./media/vpn-gateway-about-point-to-site-routing/2.jpg "varias redes virtuales emparejadas")

### <a name="address-space"></a>Espacio de direcciones:

* VNet1: 10.1.0.0/16

* VNet2: 10.2.0.0/16

* VNet3: 10.3.0.0/16

* VNet4: 10.4.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 10.2.0.0/16, 10.4.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 10.2.0.0/16, 10.4.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows pueden acceder a VNet1, VNet2 y VNet4, pero el cliente de VPN se debe descargar de nuevo para que cualquier cambio en la topología surta efecto.

* Los clientes no Windows pueden acceder a VNet1, VNet2 y VNet4.

## <a name="multiple-vnets-connected-using-an-s2s-vpn"></a><a name="multis2s"></a>Varias redes virtuales conectadas mediante VPN de sitio a sitio

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 se conecta a VNet2 mediante una conexión VPN de sitio a sitio. VNet2 se conecta a VNet3 mediante una conexión VPN de sitio a sitio. No hay emparejamiento directo ni conexión VPN de sitio a sitio entre VNet1 y VNet3. Las conexiones de sitio a sitio no ejecutan BGP para el enrutamiento.

Los clientes que usan Windows o cualquier otro SO compatible solo pueden acceder a VNet1. Para acceder a otras redes virtuales, se debe usar BGP.

![varias redes virtuales y S2S](./media/vpn-gateway-about-point-to-site-routing/3.jpg "varias redes virtuales y S2S")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

* VNet2: 10.2.0.0/16

* VNet3: 10.3.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 10.2.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows solo pueden acceder a VNet1

* Los clientes no Windows solo pueden acceder a VNet1

## <a name="multiple-vnets-connected-using-an-s2s-vpn-bgp"></a><a name="multis2sbgp"></a>Varias redes virtuales conectadas mediante VPN de sitio a sitio (BGP)

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 se conecta a VNet2 mediante una conexión VPN de sitio a sitio. VNet2 se conecta a VNet3 mediante una conexión VPN de sitio a sitio. No hay emparejamiento directo ni conexión VPN de sitio a sitio entre VNet1 y VNet3. Las conexiones de sitio a sitio ejecutan BGP para el enrutamiento.

Los clientes que usan Windows o cualquier otro SO compatible pueden acceder a todas las redes virtuales que están conectadas mediante una conexión VPN de sitio a sitio, pero las rutas a las redes virtuales conectadas se deben agregar manualmente a los clientes Windows.

![varias redes virtuales y S2S (BGP)](./media/vpn-gateway-about-point-to-site-routing/4.jpg "varias redes virtuales y S2S BGP")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

* VNet2: 10.2.0.0/16

* VNet3: 10.3.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 10.2.0.0/16, 10.3.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows pueden acceder a VNet1, VNet2 y VNet3, pero las rutas a VNet2 y VNet3 se deberán agregar manualmente.

* Los clientes no Windows pueden acceder a VNet1, VNet2 y VNet3.

## <a name="one-vnet-and-a-branch-office"></a><a name="vnetbranch"></a>Una red virtual y una sucursal

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 no está conectada ni emparejada con ninguna otra red virtual, pero sí está conectada a un sitio local a través de una conexión VPN de sitio a sitio que no ejecuta BGP.

Los clientes Windows y no Windows solo pueden acceder a VNet1.

![enrutamiento con una red virtual y una sucursal](./media/vpn-gateway-about-point-to-site-routing/5.jpg "enrutamiento con una red virtual y una sucursal")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

* Site1: 10.101.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows solo pueden acceder a VNet1

* Los clientes no Windows solo pueden acceder a VNet1

## <a name="one-vnet-and-a-branch-office-bgp"></a><a name="vnetbranchbgp"></a>Una red virtual y una sucursal (BGP)

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 no está conectada ni emparejada con ninguna otra red virtual, pero sí está conectada a un sitio local (Site1) a través de una conexión VPN de sitio a sitio que ejecuta BGP.

Los clientes Windows pueden acceder a la red virtual y a la sucursal (Site1), pero las rutas a Site1 se deben agregar manualmente al cliente. Los clientes no Windows pueden acceder a la red virtual, así como a la sucursal local.

![una red virtual y una sucursal (BGP)](./media/vpn-gateway-about-point-to-site-routing/6.jpg "una red virtual y una sucursal")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

* Site1: 10.101.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 10.101.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows pueden acceder a VNet1 y a Site1, pero las rutas a Site1 se deberán agregar manualmente.

* Los clientes no Windows pueden acceder a VNet1 y Site1.


## <a name="multiple-vnets-connected-using-s2s-and-a-branch-office"></a><a name="multivnets2sbranch"></a>Varias redes virtual conectadas mediante sitio a sitio y una sucursal

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 se conecta a VNet2 mediante una conexión VPN de sitio a sitio. VNet2 se conecta a VNet3 mediante una conexión VPN de sitio a sitio. No hay emparejamiento directo ni túnel VPN de sitio a sitio entre las redes VNet1 y VNet3. VNet3 se conecta a una sucursal (Site1) mediante una conexión VPN de sitio a sitio. Las conexiones VPN no ejecutan BGP.

Todo los clientes pueden acceder solo a VNet1.

![varias redes virtuales S2S y una sucursal](./media/vpn-gateway-about-point-to-site-routing/7.jpg "varias redes virtuales S2S y una sucursal")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

* VNet2: 10.2.0.0/16

* VNet3: 10.3.0.0/16

* Site1: 10.101.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 10.2.0.0/16, 10.3.0.0/16, 10.101.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows solo pueden acceder a VNet1

* Los clientes no Windows solo pueden acceder a VNet1

## <a name="multiple-vnets-connected-using-s2s-and-a-branch-office-bgp"></a><a name="multivnets2sbranchbgp"></a>Varias redes virtual conectadas mediante sitio a sitio y una sucursal (BGP)

En este ejemplo, la conexión de puerta de enlace de VPN de punto a sitio es para VNet1. VNet1 se conecta a VNet2 mediante una conexión VPN de sitio a sitio. VNet2 se conecta a VNet3 mediante una conexión VPN de sitio a sitio. No hay emparejamiento directo ni túnel VPN de sitio a sitio entre las redes VNet1 y VNet3. VNet3 se conecta a una sucursal (Site1) mediante una conexión VPN de sitio a sitio. Las conexiones VPN ejecutan BGP.

Los clientes que usan Windows pueden acceder a las redes virtuales y los sitios que están conectados mediante una conexión VPN de sitio a sitio, pero las rutas a VNet2, VNet3 y Site1 se deben agregar manualmente al cliente. Los clientes no Windows pueden acceder a las redes virtuales y los sitios que están conectados mediante una conexión VPN de sitio a sitio sin intervención manual de ningún tipo. El acceso es transitivo y los clientes pueden acceder a los recursos de todas las redes virtuales y sitios (locales) conectados.

![varias redes virtuales S2S y una sucursal](./media/vpn-gateway-about-point-to-site-routing/8.jpg "varias redes virtuales S2S y una sucursal")

### <a name="address-space"></a>Espacio de direcciones

* VNet1: 10.1.0.0/16

* VNet2: 10.2.0.0/16

* VNet3: 10.3.0.0/16

* Site1: 10.101.0.0/16

### <a name="routes-added"></a>Rutas agregadas

* Rutas agregadas a clientes Windows: 10.1.0.0/16, 192.168.0.0/24

* Rutas agregadas a clientes no Windows: 10.1.0.0/16, 10.2.0.0/16, 10.3.0.0/16, 10.101.0.0/16, 192.168.0.0/24

### <a name="access"></a>Acceso

* Los clientes Windows pueden acceder a VNet1, VNet2, VNet3 y Site1, pero las rutas a VNet2, VNet3 y Site1 se deben agregar manualmente al cliente.

* Los clientes no Windows pueden acceder a VNet1, VNet2, VNet3 y Site1.

## <a name="next-steps"></a>Pasos siguientes

Consulte el artículo sobre la [creación de una VPN de punto a sitio con Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md) para empezar a crear la VPN de punto a sitio.
