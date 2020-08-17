---
title: 'Ejemplo de script de Azure PowerShell: creación de reglas personalizadas de un WAF'
description: 'Ejemplo de script de Azure PowerShell: creación de reglas personalizadas de un firewall de aplicaciones web'
author: vhorne
ms.service: application-gateway
ms.topic: sample
ms.date: 6/7/2019
ms.author: victorh
ms.openlocfilehash: 4ffbab992732c39054818b9b4f21871687b05dcb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083147"
---
# <a name="create-web-application-firewall-waf-custom-rules-with-azure-powershell"></a>Creación de reglas personalizadas del firewall de aplicaciones web (WAF) con Azure PowerShell

Este script crea un firewall de aplicaciones web de Application Gateway que usa reglas personalizadas. La regla personalizada bloqueará el tráfico si el encabezado de la solicitud contiene User-Agent *evilbot*.

## <a name="prerequisites"></a>Prerequisites

### <a name="azure-powershell-module"></a>Módulo de Azure PowerShell

Si decide instalar y usar Azure PowerShell localmente, este script requiere la versión 2.1.0 del módulo de Azure PowerShell o cualquier versión posterior.

1. Para encontrar la versión, ejecute `Get-Module -ListAvailable Az`. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).
2. Ejecute `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/application-gateway/waf-rules/waf-custom-rules.ps1 "Custom WAF rules")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name CustomRulesTest
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
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |
|[New-AzApplicationGatewayAutoscaleConfiguration](/powershell/module/az.network/New-AzApplicationGatewayAutoscaleConfiguration)|Crea una configuración de escalabilidad automática para Application Gateway.|
|[New-AzApplicationGatewayFirewallMatchVariable](/powershell/module/az.network/New-AzApplicationGatewayFirewallMatchVariable)|Crea una variable de coincidencia para la condición del firewall.|
|[New-AzApplicationGatewayFirewallCondition](/powershell/module/az.network/New-AzApplicationGatewayFirewallCondition)|Crea una condición de coincidencia para la regla personalizada.|
|[New-AzApplicationGatewayFirewallCustomRule](/powershell/module/az.network/New-AzApplicationGatewayFirewallCustomRule)|Crea una nueva regla personalizada para la directiva de firewall de Application Gateway.|
|[New-AzApplicationGatewayFirewallPolicy](/powershell/module/az.network/New-AzApplicationGatewayFirewallPolicy)|Crea una directiva de firewall de Application Gateway.|
|[New-AzApplicationGatewayWebApplicationFirewallConfiguration](/powershell/module/az.network/New-AzApplicationGatewayWebApplicationFirewallConfiguration)|Crea una configuración de WAF para una puerta de enlace de aplicaciones.|

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de las reglas personalizadas de un firewall de aplicaciones web, consulte [Reglas personalizadas para el firewall de aplicaciones web](../custom-waf-rules-overview.md).
- Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
- Se pueden encontrar más ejemplos de script de PowerShell de la puerta de enlace de aplicaciones en la [documentación de Azure Application Gateway](../powershell-samples.md).
