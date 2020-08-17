---
title: 'Acerca de las puertas de enlace de red virtual de ExpressRoute: Azure| Microsoft Docs'
description: Conozca sobre las puertas de enlace de red virtual para ExpressRoute. En este artículo incluye información sobre los tipos y las SKU de puerta de enlace.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 10/14/2019
ms.author: mialdrid
ms.openlocfilehash: e7779e0638ea61c70758394dc212910ba8f1d7f6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081141"
---
# <a name="about-expressroute-virtual-network-gateways"></a>Acerca de las puertas de enlace de red virtual de ExpressRoute

Para conectar una red virtual de Azure y una red local a través de ExpressRoute, primero debe crear una puerta de enlace de red virtual. Una puerta de enlace de red virtual tiene dos propósitos: intercambiar las rutas de IP entre las redes y enrutar el tráfico de red. En este artículo se explican los tipos de puerta de enlace, las SKU de puerta de enlace y el rendimiento previsto por SKU. En este artículo también se explica [FastPath](#fastpath) de ExpressRoute, una característica que permite que el tráfico de red desde la red local omita la puerta de enlace de red virtual para mejorar el rendimiento.

## <a name="gateway-types"></a>Tipos de puerta de enlace

Al crear una puerta de enlace de red virtual, debe especificar varios valores de configuración. Uno de los valores de configuración necesarios, "-GatewayType", especifica si la puerta de enlace se usará para el tráfico VPN o de ExpressRoute. Los dos tipos de puerta de enlace son los siguientes:

* **Vpn**: para enviar tráfico cifrado a través de una conexión a Internet pública, use la puerta de enlace de tipo "Vpn". Este tipo también se conoce como VPN Gateway. Las conexiones de sitio a sitio, de punto a sitio y de red virtual a red virtual utilizan una puerta de enlace de VPN.

* **ExpressRoute**: para enviar tráfico de red en una conexión privada, use la puerta de enlace de tipo "ExpressRoute". Este tipo también se conoce como puerta de enlace de ExpressRoute y es el tipo de puerta de enlace que se utiliza al configurar ExpressRoute.

Cada red virtual tiene una única puerta de enlace de red virtual por cada tipo de puerta de enlace. Por ejemplo, puede tener una puerta de enlace de una red virtual que use -GatewayType Vpn y otra que use -GatewayType ExpressRoute.

## <a name="gateway-skus"></a><a name="gwsku"></a>SKU de puerta de enlace
[!INCLUDE [expressroute-gwsku-include](../../includes/expressroute-gwsku-include.md)]

Si desea actualizar la puerta de enlace a una SKU de puerta de enlace más eficaz, en la mayoría de los casos puede usar el cmdlet de PowerShell "Resize-AzVirtualNetworkGateway". Esto funcionará para las actualizaciones de SKU Standard y HighPerformance. Sin embargo, para actualizar a la SKU UltraPerformance, debe volver a crear la puerta de enlace. Volver a crear una puerta de enlace provoca un tiempo de inactividad.

### <a name="estimated-performances-by-gateway-sku"></a><a name="aggthroughput"></a>Rendimientos estimados por SKU de puerta de enlace
En la tabla siguiente se muestran los tipos de puerta de enlace y los rendimientos estimados. Esta tabla se aplica a los modelos de implementación del Administrador de recursos y clásico.

[!INCLUDE [expressroute-table-aggthroughput](../../includes/expressroute-table-aggtput-include.md)]

> [!IMPORTANT]
> El rendimiento de la aplicación depende de varios factores, como la latencia de extremo a extremo y el número de flujos de tráfico que abre la aplicación. Los números de la tabla representan el límite superior que teóricamente la aplicación puede alcanzar en un entorno ideal.
>
>

## <a name="gateway-subnet"></a><a name="gwsub"></a>Subred de puerta de enlace

Antes de crear una puerta de enlace de ExpressRoute, debe crear una subred de puerta de enlace. La subred de puerta de enlace contiene las direcciones IP que usan los servicios y las máquinas virtuales de la puerta de enlace de red virtual. Al crear la puerta de enlace de red virtual, las máquinas virtuales de puerta de enlace se implementan en la subred de puerta de enlace y se configuran con las opciones necesarias de puerta de enlace de ExpressRoute. Nunca implemente nada más (por ejemplo, máquinas virtuales adicionales) en la subred de puerta de enlace. Para que la subred de puerta de enlace funcione correctamente, su nombre tiene que ser “GatewaySubnet2”. La asignación del nombre "GatewaySubnet" a la subred de puerta de enlace permite a Azure saber que se trata de la subred donde se implementarán las máquinas virtuales y los servicios de la puerta de enlace de red virtual.

>[!NOTE]
>[!INCLUDE [vpn-gateway-gwudr-warning.md](../../includes/vpn-gateway-gwudr-warning.md)]
>

Al crear la subred de puerta de enlace, especifique el número de direcciones IP que contiene la subred. Las direcciones IP de la subred de puerta de enlace se asignan a las máquinas virtuales y los servicios de puerta de enlace. Algunas configuraciones requieren más direcciones IP que otras. 

Cuando planee el tamaño de la subred de puerta de enlace, consulte la documentación de la configuración que piensa crear. Por ejemplo, la configuración de coexistencia de ExpressRoute/VPN Gateway requiere una subred de puerta de enlace mayor que la mayoría de las restantes. Debe asegurarse también de que la subred de puerta de enlace contenga suficientes direcciones IP para acomodar posibles configuraciones adicionales. Aunque es posible crear una subred de puerta de enlace solo de /29, se recomienda que sea de /27, o incluso mayor, (/27, /26, etc.), siempre que se disponga de espacio para hacerlo, ya que puede albergar más configuraciones.

En el ejemplo de PowerShell de Resource Manager siguiente, se muestra una subred de puerta de enlace con el nombre GatewaySubnet. Puede ver que la notación CIDR especifica /27, que permite suficientes direcciones IP para la mayoría de las configuraciones que existen.

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="zone-redundant-gateway-skus"></a><a name="zrgw"></a>SKU de puerta de enlace con redundancia de zona

También puede implementar puertas de enlace de ExpressRoute en Azure Availability Zones. De este modo, se dividen física y lógicamente en distintas Availability Zones, lo que protege la conectividad de red local con Azure de errores de nivel de zona.

![Puerta de enlace de ExpressRoute con redundancia de zona](./media/expressroute-about-virtual-network-gateways/zone-redundant.png)

Las puertas de enlace con redundancia de zona utilizan nuevas SKU de puerta de enlace específicas para la puerta de enlace de ExpressRoute.

* ErGw1AZ
* ErGw2AZ
* ErGw3AZ

Las nuevas SKU de puerta de enlace admiten también otras opciones de implementación que se ajusten mejor sus necesidades. Si crea una puerta de enlace de red virtual con las nuevas SKU de puerta de enlace, tiene además la opción de implementar la puerta de enlace en una zona determinada. Esto se conoce como puerta de enlace zonal. Al implementar una puerta de enlace zonal, todas las instancias de la puerta de enlace se implementan en la misma zona de disponibilidad.

## <a name="fastpath"></a><a name="fastpath"></a>FastPath

La puerta de enlace de red virtual de ExpressRoute está diseñada para intercambiar las rutas de red y enrutar el tráfico de red. FastPath está diseñado para mejorar el rendimiento de las rutas de acceso a los datos entre la red local y una red virtual. Cuando está habilitado, FastPath envía el tráfico de red directamente a las máquinas virtuales que están en la red, omitiendo la puerta de enlace.

Para más información acerca de FastPath, incluidas las limitaciones y los requisitos, consulte [Acerca de FastPath](about-fastpath.md).

## <a name="rest-apis-and-powershell-cmdlets"></a><a name="resources"></a>API de REST y cmdlets de PowerShell
Para más información sobre recursos técnicos y requisitos de sintaxis específicos al usar API de REST y cmdlets de PowerShell para configuraciones de puerta de enlace de red virtual, consulte las páginas siguientes:

| **Clásico** | **Resource Manager** |
| --- | --- |
| [PowerShell](/powershell/module/servicemanagement/azure.service/?view=azuresmps-4.0.0#azure) |[PowerShell](https://docs.microsoft.com/powershell/module/az.network#networking) |
| [REST API](https://msdn.microsoft.com/library/jj154113.aspx) |[REST API](https://msdn.microsoft.com/library/mt163859.aspx) |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre configuraciones de conexión disponibles, consulte [Introducción a ExpressRoute](expressroute-introduction.md).

Para más información sobre cómo crear puertas de enlace de ExpressRoute, consulte [Creación de una puerta de enlace de red virtual para ExpressRoute](expressroute-howto-add-gateway-resource-manager.md).

Para más información acerca de cómo configurar puertas de enlace con redundancia de zona, consulte [Crear una puerta de enlace de red virtual con redundancia de zona](../../articles/vpn-gateway/create-zone-redundant-vnet-gateway.md).

Para más información sobre FastPath, vea [Acerca de FastPath](about-fastpath.md).