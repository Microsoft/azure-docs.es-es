---
title: Configuración de un túnel VPN Always On
titleSuffix: Azure VPN Gateway
description: Aprenda a usar puertas de enlace con Windows 10 Always On para establecer y configurar túneles de dispositivo persistentes dirigidos a Azure.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: ab951375fdf9c5317e2c3a2f67271666d0987142
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89435854"
---
# <a name="configure-an-always-on-vpn-device-tunnel"></a>Configuración de un túnel de dispositivo VPN para los Grupos de disponibilidad AlwaysOn

[!INCLUDE [intro](../../includes/vpn-gateway-vwan-always-on-intro.md)]

## <a name="configure-the-gateway"></a>Configuración de la puerta de enlace

Configure la puerta de enlace de VPN para usar IKEv2 y la autenticación basada en certificados mediante el artículo sobre la [configuración de una conexión VPN de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

## <a name="configure-the-device-tunnel"></a>Configuración del túnel de dispositivo

[!INCLUDE [device tunnel](../../includes/vpn-gateway-vwan-always-on-device.md)]

## <a name="to-remove-a-profile"></a>Eliminación de un perfil

Para quitar el perfil, ejecute el siguiente comando:

![Limpieza](./media/vpn-gateway-howto-always-on-device-tunnel/cleanup.png)

## <a name="next-steps"></a>Pasos siguientes

Para solucionar problemas, consulte [Problemas de conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
