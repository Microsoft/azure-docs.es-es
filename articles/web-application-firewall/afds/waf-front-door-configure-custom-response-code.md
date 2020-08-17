---
title: Configuración de respuestas personalizadas para el Firewall de aplicaciones web (WAF) en Azure Front Door
description: Obtenga información sobre cómo configurar un mensaje y un código de respuesta personalizados cuando el Firewall WAF bloquee una solicitud.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 06/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: a995460793686d8293d77965e74e2cbf916925a0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005606"
---
# <a name="configure-a-custom-response-for-azure-web-application-firewall-waf"></a>Configuración de una respuesta personalizada para el Firewall de aplicaciones web de Azure (WAF)

De forma predeterminada, cuando WAF bloquea una solicitud debido a una regla coincidente, devuelve un código de estado 403 con el mensaje **La solicitud está bloqueada**. El mensaje predeterminado también incluye la cadena de referencia de seguimiento que se puede usar para vincular [entradas de registro](https://docs.microsoft.com/azure/web-application-firewall/afds/waf-front-door-monitor) para la solicitud.  Puede configurar un código de estado de respuesta personalizada y un mensaje personalizado con una cadena de referencia para el caso de uso. En este artículo se describe cómo configurar una página de respuesta personalizada cuando WAF bloquea una solicitud.

## <a name="configure-custom-response-status-code-and-message-use-portal"></a>Configuración del código de estado de respuesta personalizada y el portal de uso de mensajes

Puede configurar un cuerpo y un código de estado de respuesta personalizados en "Configuración de directiva", en el portal de WAF.

:::image type="content" source="../media/waf-front-door-configure-custom-response-code/custom-response-settings.png" alt-text="Configuración de directiva de WAF":::

En el ejemplo anterior, conservamos el código de respuesta como 403 y configuramos un breve mensaje "Póngase en contacto con nosotros" como se muestra en la siguiente imagen:

:::image type="content" source="../media/waf-front-door-configure-custom-response-code/custom-response.png" alt-text="Ejemplo de respuesta de personalizada":::

"{{azure-ref}}" inserta la cadena de referencia única en el cuerpo de la respuesta. El valor coincide con el campo TrackingReference de los registros `FrontdoorAccessLog` y `FrontdoorWebApplicationFirewallLog`.

## <a name="configure-custom-response-status-code-and-message-use-powershell"></a>Configuración del código de estado de respuesta personalizada y el PowerShell de uso de mensajes

### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell

Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones de la página para iniciar sesión con sus credenciales de Azure e instalar el módulo Az.PowerShell.

### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Conexión a Azure con un cuadro de diálogo interactivo para el inicio de sesión

```
Connect-AzAccount
Install-Module -Name Az

```
Asegúrese de tener instalada la versión actual de PowerShellGet. Ejecute el comando siguiente y vuelva a abrir PowerShell.
```
Install-Module PowerShellGet -Force -AllowClobber
``` 
### <a name="install-azfrontdoor-module"></a>Instalación del módulo Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Aquí creamos un grupo de recursos con [New-AzResourceGroup](/powershell/module/Az.resources/new-Azresourcegroup).

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupWAF
```

### <a name="create-a-new-waf-policy-with-custom-response"></a>Creación de una directiva de WAF con respuesta personalizada 

A continuación se muestra un ejemplo de creación de una directiva de WAF con el código de estado de respuesta personalizado establecido en 405 y el mensaje **You are blocked** (Se le ha bloqueado), mediante [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy)

```azurepowershell
# WAF policy setting
New-AzFrontDoorWafPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 405 `
-CustomBlockResponseBody "<html><head><title>You are blocked.</title></head><body></body></html>"
```

Modifique la configuración del cuerpo de la respuesta o el código de respuesta personalizados de una directiva de WAF existente mediante el comando [Update-AzFrontDoorFireWallPolicy](/powershell/module/az.frontdoor/Update-AzFrontDoorWafPolicy).

```azurepowershell
# modify WAF response code
Update-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 403
```

```azurepowershell
# modify WAF response body
Update-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-CustomBlockResponseBody "<html><head><title>Forbidden</title></head><body>{{azure-ref}}</body></html>"
```

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre [Firewall de aplicaciones web con Azure Front Door](../afds/afds-overview.md)