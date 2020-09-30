---
title: 'Creación y configuración de directivas IPsec personalizadas para punto a sitio: PowerShell'
titleSuffix: Azure VPN Gateway
description: Este artículo le ayuda a crear y establecer directivas IPSec personalizadas para configuraciones P2S de VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/09/2020
ms.author: cherylmc
ms.openlocfilehash: 852ff5e6ad847ff33f8e32b4c3d9f8b325b80716
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89663545"
---
# <a name="create-and-set-custom-ipsec-policies-for-point-to-site"></a>Creación y configuración de directivas IPsec personalizadas para punto a sitio

Si su entorno requiere una directiva IPsec personalizada para el cifrado, puede configurar fácilmente un objeto de directiva con los valores necesarios. Este artículo le ayuda a crear un objeto de directiva personalizado y, a continuación, a configurarlo mediante PowerShell.

## <a name="before-you-begin"></a>Antes de empezar

### <a name="prerequisites"></a>Requisitos previos

Compruebe que el entorno cumpla los siguientes requisitos previos:

* Ya tiene configurada una VPN de punto a sitio que funciona. Si no es así, configure una con los pasos del artículo**Creación de una VPN de punto a sitio** con [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md) o [Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

## <a name="1-set-variables"></a><a name="signin"></a>1. Configuración de variables

Declare las variables que desea utilizar. Use el ejemplo siguiente y reemplace los valores por los suyos propios cuando sea necesario. Si cierra la sesión de PowerShell/Cloud Shell en cualquier momento durante el ejercicio, simplemente copie y pegue los valores de nuevo para volver a declarar las variables.

  ```azurepowershell-interactive
  $RG = "TestRG"
  $GWName = "VNet1GW"
  ```

## <a name="2-create-policy-object"></a><a name="create"></a>2. Creación de un objeto de directiva

Cree un objeto de directiva IPsec personalizada. Puede ajustar los valores para que cumplan los criterios que necesite.

```azurepowershell-interactive
$vpnclientipsecpolicy = New-AzVpnClientIpsecPolicy -IpsecEncryption AES256 -IpsecIntegrity SHA256 -SALifeTime 86471 -SADataSize 429496 -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup2 -PfsGroup PFS2
```

## <a name="3-update-gateway-and-set-policy"></a><a name="update"></a>3. Actualización de la puerta de enlace y establecimiento de la directiva

En este paso, actualice la puerta de enlace de VPN P2S existente y establezca la directiva IPsec.

```azurepowershell-interactive
$gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gateway -VpnClientIpsecPolicy $vpnclientipsecpolicy
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las configuraciones P2S, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).