---
title: 'Azure VPN Gateway: Anuncio de rutas personalizadas para clientes VPN de conexión de punto a sitio'
description: Pasos para anunciar rutas personalizadas a sus clientes de punto a sitio
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: a02bd5519b776a063646c11be2a34366fe429f99
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89392398"
---
# <a name="advertise-custom-routes-for-p2s-vpn-clients"></a>Anuncio de rutas personalizadas para clientes VPN de conexión de punto a sitio

Es posible que quiera anunciar rutas personalizadas a todos sus clientes VPN de punto a sitio. Por ejemplo, cuando tiene puntos de conexión de almacenamiento habilitados en su red virtual y quiere que los usuarios remotos puedan tener acceso a estas cuentas de almacenamiento a través de la conexión VPN. Puede anunciar la dirección IP del punto de conexión de almacenamiento a todos sus usuarios remotos de modo que el tráfico a la cuenta de almacenamiento pase por el túnel VPN y no por la red pública de Internet.

![Ejemplo de conexión multisitio de Azure VPN Gateway](./media/vpn-gateway-p2s-advertise-custom-routes/custom-routes.png)

## <a name="to-advertise-custom-routes"></a>Para anunciar rutas personalizadas

Para anunciar rutas personalizadas, use `Set-AzVirtualNetworkGateway cmdlet`. En el siguiente ejemplo se le muestra cómo anunciar la dirección IP para las [tablas de cuentas de almacenamiento de Contoso](https://contoso.table.core.windows.net).

1. Haga ping en *contoso.table.core.windows.net* y tenga en cuenta la dirección IP. Por ejemplo:

    ```cmd
    C:\>ping contoso.table.core.windows.net
    Pinging table.by4prdstr05a.store.core.windows.net [13.88.144.250] with 32 bytes of data:
    ```

2. Ejecute los comandos de PowerShell siguientes:

    ```azurepowershell-interactive
    $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute 13.88.144.250/32
    ```

3. Para agregar varias rutas personalizadas, use una coma y espacios para separar las direcciones. Por ejemplo:

    ```azurepowershell-interactive
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute x.x.x.x/xx , y.y.y.y/yy
    ```
## <a name="to-view-custom-routes"></a>Para ver rutas personalizadas

Use el siguiente ejemplo para ver rutas personalizadas:

  ```azurepowershell-interactive
  $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
  $gw.CustomRoutes | Format-List
  ```
## <a name="to-delete-custom-routes"></a>Para eliminar rutas personalizadas

Use el siguiente ejemplo para eliminar rutas personalizadas:

  ```azurepowershell-interactive
  $gw = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroupName <name of resource group>
  Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -CustomRoute @0
  ```
## <a name="next-steps"></a>Pasos siguientes

Para obtener información de enrutamiento de conexión de punto a sitio adicional, consulte [Acerca del enrutamiento de punto a sitio](vpn-gateway-about-point-to-site-routing.md).
