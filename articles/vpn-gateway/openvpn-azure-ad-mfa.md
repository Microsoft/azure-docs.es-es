---
title: 'Habilitar MFA para usuarios de VPN: Autenticación de Azure AD'
description: Habilitar la autenticación multifactor para los usuarios de VPN
services: vpn-gateway
author: kumudD
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/14/2020
ms.author: alzam
ms.openlocfilehash: e6d3ff32d2daaf35ff00bd0ee1d8f42669683827
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86998602"
---
# <a name="enable-azure-multi-factor-authentication-mfa-for-vpn-users"></a>Habilitar Azure Multi-Factor Authentication (MFA) para los usuarios de VPN

[!INCLUDE [overview](../../includes/vpn-gateway-vwan-openvpn-enable-mfa-overview.md)]

## <a name="enable-authentication"></a><a name="enableauth"></a>Habilitar la autenticación

[!INCLUDE [enable authentication](../../includes/vpn-gateway-vwan-openvpn-enable-auth.md)]

## <a name="configure-sign-in-settings"></a><a name="enablesign"></a>Configuración de las opciones de inicio de sesión

[!INCLUDE [sign in](../../includes/vpn-gateway-vwan-openvpn-sign-in.md)]

## <a name="option-1---per-user-access"></a><a name="peruser"></a>Opción 1: Acceso por usuario

[!INCLUDE [per user](../../includes/vpn-gateway-vwan-openvpn-per-user.md)]

## <a name="option-2---conditional-access"></a><a name="conditional"></a>Opción 2: Acceso condicional

[!INCLUDE [conditional access](../../includes/vpn-gateway-vwan-openvpn-conditional.md)]

## <a name="next-steps"></a>Pasos siguientes

Para conectarse a la red virtual, debe crear y configurar un perfil de cliente VPN. Consulte [Configurar un cliente VPN para conexiones P2S VPN](openvpn-azure-ad-client.md).