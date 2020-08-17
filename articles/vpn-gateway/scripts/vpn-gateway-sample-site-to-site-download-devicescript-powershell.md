---
title: 'Ejemplo de script de Azure PowerShell: descarga de la plantilla de configuración de dispositivos | Microsoft Docs'
description: Descarga de la plantilla de configuración de dispositivos
services: vpn-gateway
documentationcenter: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.devlang: powershell
ms.topic: sample
ms.date: 01/09/2020
ms.author: yushwang
ms.openlocfilehash: 8c0be71f5ffe82f6b6e1cb607baf070c499bd46f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077236"
---
# <a name="download-vpn-device-template-using-powershell"></a>Descarga de la plantilla de dispositivos VPN mediante PowerShell

Este script descarga la plantilla de dispositivos VPN para una conexión

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```azurepowershell-interactive
# Declare variables
$RG          = "TestRG1"
$GWName      = "VNet1GW"
$Connection  = "VNet1toSite1"

# List the available VPN device models and versions
Get-AzVirtualNetworkGatewaySupportedVpnDevice -Name $GWName -ResourceGroupName $RG

# Download the configuration template for the connection
Get-AzVirtualNetworkGatewayConnectionVpnDeviceConfigScript -Name $Connection -ResourceGroupName $RG `
-DeviceVendor Juniper -DeviceFamily Juniper_SRX_GA -FirmwareVersion Juniper_SRX_12.x_GA
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, use el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para eliminar el grupo de recursos. Se eliminarán el grupo de recursos y todos los recursos que contiene.

```azurepowershell-interactive
Remove-AzResourceGroup -Name TestRG1
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzVirtualNetworkGatewaySupportedVpnDevice](/powershell/module/az.network/Get-azVirtualNetworkGatewaySupportedVpnDevice) | Muestra todos los modelos y las versiones disponibles de dispositivo VPN. |
| [Get-AzVirtualNetworkGatewayConnectionVpnDeviceConfigScript](/powershell/module/az.network/Get-azVirtualNetworkGatewayConnectionVpnDeviceConfigScript) | Descarga la plantilla de configuración para la conexión. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
