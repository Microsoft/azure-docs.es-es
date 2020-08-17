---
title: 'Vista de las rutas eficaces de un centro de conectividad virtual: Azure Virtual WAN | Microsoft Docs'
description: Visualización de las rutas eficaces de un concentrador virtual en Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 06/29/2020
ms.author: cherylmc
ms.openlocfilehash: a7e42ddeb4abacd8707dda4cd558933b0d7a34f4
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513713"
---
# <a name="view-virtual-hub-effective-routes"></a>Visualización de rutas efectivas de centro virtual

Puede ver todas las rutas del centro virtual WAN en el Azure Portal. Este artículo lo guiará por los pasos necesarios para ver las rutas eficaces. Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

> [!NOTE]
> En Azure Portal, es posible que algunas de estas características se sigan implementando y no estén disponibles hasta la semana del 17 de agosto. 
>

## <a name="select-connections-or-route-tables"></a><a name="routing"></a>Seleccionar conexiones o tablas de rutas

1. Navegue hasta el centro virtual y seleccione **Enrutamiento**. En la página Enrutamiento, seleccione **Rutas eficaces**.
1. En la lista desplegable, puede seleccionar **Tipo de conexión** o una**Tabla de rutas**. Si no ve una opción de Tabla de rutas, significa que no tiene configurada una tabla de rutas personalizada o predeterminada en este centro virtual.
1. En la lista desplegable para **las conexiones y las tablas de rutas**, puede seleccionar entre los siguientes elementos:

   * Conexión de Virtual Network
   * Conexión de sitio VPN
   * Conexión ExpressRoute
   * Conexión de punto a sitio
   * Tabla de rutas

   :::image type="content" source="./media/effective-routes-virtual-hub/routing.png" alt-text="Enrutamiento":::

## <a name="view-output"></a><a name="output"></a>Ver salida

En el resultado de la página se muestran los siguientes campos:

* **Prefijo**: Prefijo de dirección conocido para la entidad actual.
* **Tipo de próximo salto**: Puede ser Conexión Virtual Network, VPN_S2S_Gateway, ExpressRouteGateway, Concentrador remoto o Azure Firewall.
* **Próximo salto**: Esta es la dirección IP o simplemente muestra en el vínculo para implicar al centro actual.
* **Origin** (Origen): Identificador de recurso del origen de enrutamiento.
* **Ruta de acceso AS**: La ruta de acceso del atributo BGP como (sistema autónomo) muestra todos los números AS que deben atravesarse para llegar a la ubicación a la que se publica el prefijo al que está asociada la ruta de acceso.

### <a name="example"></a><a name="example"></a>Ejemplo

Los valores de la siguiente tabla de ejemplo implican que la conexión del concentrador virtual o la tabla de rutas ha aprendido la ruta de 10.2.0.0/24 (un prefijo de rama). Ha aprendido la ruta debido al **tipo de próximo salto VPN** VPN_S2S_Gateway con el identificador de recurso del **próximo salto** VPN Gateway. **Origen de ruta** apunta al identificador de recurso de la puerta de enlace de VPN o la tabla o conexión de enrutamiento de origen. **La ruta de acceso AS** indica la ruta de acceso AS para la rama.

Use la barra de desplazamiento de la parte inferior de la tabla para ver la "ruta de acceso del sistema autónomo (AS)".

| **Prefijo** |  **Tipo del próximo salto** | **Próximo salto** |  **Origen de la ruta** |**Ruta de acceso del sistema autónomo (AS)** |
| ---        | ---                | ---          | ---               | ---         |
| 10.2.0.0/24| VPN_S2S_Gateway |10.1.0.6, 10.1.0.7|/subscriptions/`<sub id>`/resourceGroups/`<resource group name>`/providers/Microsoft.Network/vpnGateways/vpngw| 20 000|

**Consideraciones:**

* Si ve 0.0.0.0/0 en la salida de **Obtener rutas eficaces**, significa que la ruta existe en una de las tablas de rutas. Pero si esta ruta se configuró para internet, se requiere una marca adicional **"enableInternetSecurity": true** en la conexión. La ruta efectiva de la NIC de la máquina virtual no mostrará la ruta si la marca "enableInternetSecurity" en la conexión es "false".

* El campo **Propagar ruta predeterminada** se ve en el portal Azure Virtual WAN cuando edita una conexión de red virtual, una conexión VPN o una conexión ExpressRoute. Este campo indica la marca **enableInternetSecurity**, que siempre es "false" de forma predeterminada para las conexiones ExpressRoute y VPN, pero "true" para las conexiones de red virtual.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte la [Introducción a Virtual WAN](virtual-wan-about.md).
* Para más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
