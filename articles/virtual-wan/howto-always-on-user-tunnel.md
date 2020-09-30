---
title: Configuración de un túnel de usuario de VPN Always On
titleSuffix: Azure Virtual WAN
description: En este artículo se describe cómo configurar un túnel de usuario de VPN AlwaysOn en Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/10/2020
ms.author: cherylmc
ms.openlocfilehash: 14790d4be91ffd0463436b6ca1d5c8794c102697
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90016374"
---
# <a name="configure-an-always-on-vpn-user-tunnel-for-virtual-wan"></a>Configuración de un túnel de usuario de VPN Always On en Virtual WAN

[!INCLUDE [intro](../../includes/vpn-gateway-vwan-always-on-intro.md)]

## <a name="prerequisites"></a>Requisitos previos

Debe crear una configuración de punto a sitio y editar la asignación del concentrador virtual. Consulte las secciones siguientes para ver instrucciones:

* [Creación de una configuración de P2S](virtual-wan-point-to-site-portal.md#p2sconfig)
* [Creación de un centro de conectividad con puerta de enlace P2S](virtual-wan-point-to-site-portal.md#hub)

## <a name="configure-a-user-tunnel"></a>Configuración de un túnel de usuario

[!INCLUDE [user tunnel](../../includes/vpn-gateway-vwan-always-on-user.md)]

## <a name="to-remove-a-profile"></a>Eliminación de un perfil

Para quitar un perfil, use estos pasos:

1. Ejecute el siguiente comando:

   ```powershell
   C:\> Remove-VpnConnection UserTest  
   ```

1. Desconecte la conexión y desactive la opción **Conectar automáticamente**.

   ![Limpieza](./media/howto-always-on-user-tunnel/disconnect.jpg)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
