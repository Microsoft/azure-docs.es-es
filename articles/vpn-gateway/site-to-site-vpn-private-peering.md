---
title: Conexiones VPN de sitio a sitio a través del emparejamiento privado de ExpressRoute
description: La información de este artículo le ayuda a habilitar VPN de sitio a sitio a través del emparejamiento privado de ExpressRoute con el objetivo de cifrar el tráfico.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/08/2020
ms.author: cherylmc
ms.openlocfilehash: effbe8e771922ea07ad908dd4871f8dcdb7c1d19
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932204"
---
# <a name="configure-a-site-to-site-vpn-connection-over-expressroute-private-peering-preview"></a>Configuración de una conexión VPN de sitio a sitio a través del emparejamiento privado de ExpressRoute (versión preliminar)

Puede configurar una VPN de sitio a sitio para una puerta de enlace de red virtual a través de un emparejamiento privado de ExpressRoute con una dirección IP de RFC 1918. Esta configuración proporciona las siguientes ventajas:

* El tráfico a través del emparejamiento privado está cifrado.

* Los usuarios de punto a sitio que se conectan a una puerta de enlace de red virtual pueden usar ExpressRoute (a través del túnel de sitio a sitio) para acceder a los recursos locales.

>[!NOTE]
>Esta característica solo se admite en puertas de enlace con redundancia de zona. Por ejemplo, VpnGw1AZ, VpnGw2AZ, etc.
>

Para realizar esta configuración, compruebe que cumple los siguientes requisitos previos:

* Tiene un circuito ExpressRoute en funcionamiento que está vinculado a la red virtual donde se ha creado (o creará) la puerta de enlace de VPN.

* Puede conectarse a los recursos mediante la dirección IP de RFC1918 (privada) a través del circuito ExpressRoute.

## <a name="routing"></a><a name="routing"></a>Enrutamiento

En la **Figura 1** se muestra un ejemplo de conectividad de VPN a través del emparejamiento privado de ExpressRoute. En este ejemplo, puede ver una red dentro de la red local conectada a la puerta de enlace de VPN del centro de conectividad de Azure a través del emparejamiento privado de ExpressRoute. Un aspecto importante de esta configuración es el enrutamiento entre las redes locales y Azure a través de las rutas de acceso de ExpressRoute y VPN.

En la Ilustración 1

:::image type="content" source="media/site-to-site-vpn-private-peering/routing.png" alt-text="Ilustración 1":::

El establecimiento de la conectividad es una tarea sencilla:

1. Establezca la conectividad de ExpressRoute con un circuito ExpressRoute y emparejamiento privado.

1. Establezca la conectividad VPN mediante los pasos de este artículo.

### <a name="traffic-from-on-premises-networks-to-azure"></a>Tráfico desde redes locales a Azure

Para el tráfico que procede de redes locales y se dirige a Azure, los prefijos de Azure se anuncian a través del BGP de emparejamiento privado de ExpressRoute y el BGP de VPN. El resultado son dos rutas de red (rutas de acceso) hacia Azure desde las redes locales:

• Una ruta de red a través de la ruta de acceso protegida por IPsec.

• Otra ruta de red directamente a través de ExpressRoute sin protección de IPsec.

Para aplicar el cifrado a la comunicación, debe asegurarse de que para la red conectada a VPN de la **Figura 1** se prefieran las rutas de Azure a través de la puerta de enlace de VPN local en vez de la ruta de acceso directa de ExpressRoute.

### <a name="traffic-from-azure-to-on-premises-networks"></a>Tráfico desde Azure a las redes locales

El mismo requisito se aplica al tráfico de Azure a las redes locales. Para asegurarse de que la ruta de acceso de IPsec se elige antes que la ruta de acceso directa de ExpressRoute (sin IPsec), tiene dos opciones:

• **Anuncie prefijos más específicos en la sesión BGP de VPN para la red conectada a VPN**. Puede anunciar un rango mayor que abarque la red conectada a VPN a través del emparejamiento privado de ExpressRoute y, después, rangos más específicos en la sesión del protocolo de puerta de enlace de borde de VPN. Por ejemplo, anuncie 10.0.0.0/16 a través de ExpressRoute y 10.0.1.0/24 a través de VPN.

• **Anuncie prefijos disjuntos para VPN y ExpressRoute**. Si los rangos de redes conectadas a VPN no están unidos a otras redes conectadas de ExpressRoute, puede anunciar los prefijos en las sesiones de los protocolos de puerta de enlace de borde de ExpressRoute y VPN, respectivamente. Por ejemplo, anuncie 10.0.0.0/24 a través de ExpressRoute y 10.0.1.0/24 a través de VPN.

En ambos ejemplos, Azure enviará tráfico a 10.0.1.0/24 a través de la conexión VPN en lugar de hacerlo directamente a través de ExpressRoute sin protección VPN.

>[!Warning]
>Si anuncia los mismos prefijos a través de las conexiones de ExpressRoute y VPN, Azure usará la ruta de acceso de ExpressRoute directamente sin protección de VPN.
>

## <a name="portal-steps"></a><a name="portal"></a>Pasos del portal

1. Configure una conexión de sitio a sitio. Para conocer los pasos, consulte el artículo [Configuración de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Asegúrese de elegir una SKU de puerta de enlace con redundancia de zona como puerta de enlace. Las SKU con redundancia de zona tienen "AZ" al final. Por ejemplo, VpnGw1AZ.

   :::image type="content" source="media/site-to-site-vpn-private-peering/gateway.png" alt-text="Direcciones IP privadas de puerta de enlace":::
1. Habilite las direcciones IP privadas de la puerta de enlace. Seleccione **Configuración** y, luego, establezca **Gateway Private IPs** (Direcciones IP privadas de puerta de enlace) en **Habilitado**. Haga clic en **Guardar** para guardar los cambios.
1. En la página **Información general**, seleccione **Ver más** para ver la dirección IP privada. Anote esta información, ya que la usará más adelante en los pasos de configuración.

   :::image type="content" source="media/site-to-site-vpn-private-peering/gateway-overview.png" alt-text="Página de información general" lightbox="media/site-to-site-vpn-private-peering/gateway-overview.png":::
1. Para habilitar **Usar la dirección IP privada de Azure** en la conexión, seleccione **Configuración**. Establezca **Usar la dirección IP privada de Azure** en **Habilitado** y, luego. seleccione **Guardar**.

   :::image type="content" source="media/site-to-site-vpn-private-peering/connection.png" alt-text="Gateway Private IPs (Direcciones IP privadas de puerta de enlace): Habilitado":::
1. En el firewall, haga ping a la dirección IP privada que anotó en el paso 3. La dirección IP privada debe ser accesible a través del emparejamiento privado de ExpressRoute.
1. Use esta dirección IP privada como dirección IP remota en el firewall local para establecer el túnel de sitio a sitio a través del emparejamiento privado de ExpressRoute.

## <a name="powershell-steps"></a><a name="powershell"></a>pasos de PowerShell

1. Configure una conexión de sitio a sitio. Para conocer los pasos, consulte el artículo [Configuración de una VPN de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Asegúrese de elegir una SKU de puerta de enlace con redundancia de zona como puerta de enlace. Las SKU con redundancia de zona tienen "AZ" al final. Por ejemplo, VpnGw1AZ.
1. Establezca la marca para usar la dirección IP privada en la puerta de enlace con los siguientes comandos de PowerShell:

   ```azurepowershell-interactive
   $Gateway = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroup <name of resource group>

   Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -EnablePrivateIpAddress $true
   ```

   Verá una dirección IP pública y una privada. Anote la dirección IP que se muestra en la sección "TunnelIpAddresses" de la salida. Usará esta información en un paso posterior.
1. Establezca la conexión para que use la dirección IP privada con el siguiente comando de PowerShell:

   ```azurepowershell-interactive
   $Connection = get-AzVirtualNetworkGatewayConnection -Name <name of the connection> -ResourceGroupName <name of resource group>

   Set-AzVirtualNetworkGatewayConnection --VirtualNetworkGatewayConnection $Connection -UseLocalAzureIpAddress $true
   ```
1. En el firewall, haga ping a la dirección IP privada que anotó en el paso 2. Debe ser accesible a través del emparejamiento privado de ExpressRoute.
1. Use esta dirección IP privada como dirección IP remota en el firewall local para establecer el túnel de sitio a sitio a través del emparejamiento privado de ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre VPN Gateway, consulte [¿Qué es VPN Gateway?](vpn-gateway-about-vpngateways.md)
