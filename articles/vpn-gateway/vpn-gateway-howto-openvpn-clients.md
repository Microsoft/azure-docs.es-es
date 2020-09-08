---
title: Cómo configurar los clientes OpenVPN de Azure VPN Gateway | Microsoft Docs
description: Obtenga información sobre cómo configurar OpenVPN para Azure VPN Gateway para clientes del sistema operativo Windows, Linux y Mac.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 12/12/2019
ms.author: cherylmc
ms.openlocfilehash: 92447a541726c1c11b7b10d6d52cf91cfc07f945
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036868"
---
# <a name="configure-openvpn-clients-for-azure-vpn-gateway"></a>Configuración de los clientes OpenVPN de Azure VPN Gateway

Este artículo le ayuda a configurar clientes del **protocolo &reg;OpenVPN**.

## <a name="before-you-begin"></a>Antes de empezar

Compruebe que ha completado los pasos para configurar OpenVPN para VPN Gateway. Para obtener más información, consulte [Configuración de los clientes OpenVPN de Azure VPN Gateway](vpn-gateway-howto-openvpn.md).

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>Pasos siguientes

Si quiere que los clientes VPN tengan acceso a recursos de otra red virtual, siga las instrucciones del artículo [De red virtual a red virtual](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) para configurar una conexión de red virtual a red virtual. Asegúrese de habilitar BGP en las puertas de enlace y las conexiones; en caso contrario, el tráfico no fluirá.

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
