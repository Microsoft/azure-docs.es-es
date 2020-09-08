---
title: 'Creación de una instancia de VPN Gateway basada en rutas: portal'
titleSuffix: Azure VPN Gateway
description: Use Azure Portal para crear rápidamente una instancia de Azure VPN Gateway basada en rutas para una conexión VPN a una red local o para conectar redes virtuales.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 104d911164d0194efba41f1405c17fc240e8906d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393775"
---
# <a name="create-a-route-based-vpn-gateway-using-the-azure-portal"></a>Creación de una instancia de VPN Gateway basada en rutas mediante Azure Portal

En este artículo se explica cómo crear rápidamente una instancia de Azure VPN Gateway basada en rutas mediante Azure Portal.  Una instancia de VPN Gateway se usa al crear una conexión VPN a la red local. También puede utilizar una instancia de VPN Gateway para conectar redes virtuales. 

Los pasos que se describen en este artículo crearán una red virtual, una subred, una subred de puerta de enlace y una instancia de VPN Gateway basada en rutas (puerta de enlace de red virtual). Una vez completada la creación de la puerta de enlace, puede crear las conexiones. Estos pasos requieren una suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-virtual-network"></a><a name="vnet"></a>Creación de una red virtual

[!INCLUDE [Basic Point-to-Site VNet](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="configure-and-create-the-gateway"></a><a name="gwvalues"></a>Configuración y creación de la puerta de enlace

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

[!INCLUDE [Create a gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

>[!NOTE]
>La SKU de puerta de enlace de nivel Básico no admite la autenticación de IKEv2 o RADIUS. Si planea que clientes Mac se conecten a la red virtual, no use la SKU de nivel Básico.

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="view-the-vpn-gateway"></a><a name="viewgw"></a>Visualización de VPN Gateway

1. Después de crea la puerta de enlace, vaya a VNet1 en el portal. La instancia de VPN Gateway aparece en la página Información general como un dispositivo conectado.

   ![Dispositivos conectados](./media/create-routebased-vpn-gateway-portal/view-connected-devices.png "Dispositivos conectados")

2. En la lista de dispositivos, haga clic en **VNet1GW** para ver más información.

   ![Visualización de la puerta de enlace de VPN](./media/create-routebased-vpn-gateway-portal/view-gateway.png "Visualización de la puerta de enlace de VPN")

## <a name="next-steps"></a>Pasos siguientes

Una vez creada la puerta de enlace, puede crear una conexión entre su red virtual y otra red virtual. O bien, cree una conexión entre su red virtual y una ubicación local.

> [!div class="nextstepaction"]
> [Creación de una conexión de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md)<br><br>
> [Creación de una conexión de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md)<br><br>
> [Creación de una conexión a otra red virtual](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
