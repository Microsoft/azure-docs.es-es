---
title: 'Ejemplo de script de Azure PowerShell: administrar el tráfico web | Microsoft Docs'
description: 'Ejemplo de script de Azure PowerShell: administrar el tráfico de web con una puerta de enlace de aplicaciones y un conjunto de escalado de máquinas virtuales.'
services: application-gateway
documentationcenter: networking
author: vhorne
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: b0ad5043255c1c8a804245e8e944a8513a958819
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87004824"
---
# <a name="manage-web-traffic-with-azure-powershell"></a>Administración del tráfico web con Azure PowerShell

Este script crea una puerta de enlace de aplicaciones que usa un conjunto de escalado de máquinas virtuales para los servidores back-end. La puerta de enlace de aplicaciones se puede configurar para administrar el tráfico web. Después de ejecutar el script, se puede probar la puerta de enlace de aplicaciones mediante la dirección IP pública.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/application-gateway/create-vmss/create-vmss.ps1 "Create application gateway")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea la configuración de subred. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea la red virtual mediante las configuraciones de la subred. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea la dirección IP pública de la puerta de enlace de aplicaciones. |
| [New-AzApplicationGatewayIPConfiguration](/powershell/module/az.network/new-azapplicationgatewayipconfiguration) | Crea la configuración que asocia una subred con la puerta de enlace de aplicaciones. |
| [New-AzApplicationGatewayFrontendIPConfig](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig) | Crea la configuración que asigna una dirección IP pública a la puerta de enlace de aplicaciones. |
| [New-AzApplicationGatewayFrontendPort](/powershell/module/az.network/new-azapplicationgatewayfrontendport) | Asigna un puerto que se usará para acceder a la puerta de enlace de aplicaciones. |
| [New-AzApplicationGatewayBackendAddressPool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool) | Crea un grupo de back-end para una puerta de enlace de aplicaciones. |
| [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting) | Configura los ajustes para un grupo de back-end. |
| [New-AzApplicationGatewayHttpListener](/powershell/module/az.network/new-azapplicationgatewayhttplistener) | Crea un agente de escucha. |
| [New-AzApplicationGatewayRequestRoutingRule](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule) | Crea una regla de enrutamiento. |
| [New-AzApplicationGatewaySku](/powershell/module/az.network/new-azapplicationgatewaysku) | Especifica el nivel y la capacidad de una puerta de enlace de aplicaciones. |
| [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway) | Crear una puerta de enlace de aplicaciones. |
| [Set-AzVmssStorageProfile](/powershell/module/az.compute/set-azvmssstorageprofile) | Crea un perfil de almacenamiento para el conjunto de escalado. |
| [Set-AzVmssOsProfile](/powershell/module/az.compute/set-azvmssosprofile) | Define el sistema operativo del conjunto de escalado. |
| [Add-AzVmssNetworkInterfaceConfiguration](/powershell/module/az.compute/add-azvmssnetworkinterfaceconfiguration) | Define la interfaz de red del conjunto de escalado. |
| [New-AzVmss](/powershell/module/az.compute/new-azvm) | Crea un conjunto de escalado de máquinas virtuales. |
| [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) | Obtiene la dirección IP pública de una puerta de enlace de aplicaciones. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. | 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Se pueden encontrar más ejemplos de script de PowerShell de la puerta de enlace de aplicaciones en la [documentación de Azure Application Gateway](../powershell-samples.md).
