---
title: Configurar una respuesta personalizada para firewall de aplicaciones web en Azure puerta de entrada
description: Obtenga información sobre cómo configurar un código de respuesta personalizado y un mensaje cuando el firewall de aplicaciones web (WAF) bloquea una solicitud.
services: frontdoor
author: KumudD
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/08/2019
ms.author: tyao;kumud
ms.openlocfilehash: 7a167deb511347798fa609e2aca2a19f8bf12d21
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523715"
---
# <a name="configure-a-custom-response-for-azure-web-application-firewall"></a>Configurar una respuesta personalizada para el firewall de aplicaciones web de Azure

De forma predeterminada, cuando el firewall de aplicaciones web de Azure (WAF) con Azure puerta principal bloquea una solicitud debido a una regla coincidente, devuelve un código de 403 estado con **se bloquea la solicitud** mensaje. Este artículo describe cómo configurar un código de estado de respuesta personalizada y un mensaje de respuesta cuando se bloquea una solicitud con WAF.

## <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones de la página para iniciar sesión con sus credenciales de Azure e instalar el módulo Az.PowerShell.

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

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. En este ejemplo, cree un grupo de recursos mediante el uso de [New AzResourceGroup](/powershell/module/Az.resources/new-Azresourcegroup).

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupWAF
```

## <a name="create-a-new-waf-policy-with-custom-response"></a>Crear una nueva directiva de WAF con respuesta personalizada 

A continuación es un ejemplo de creación de una nueva directiva de WAF con código de estado de respuesta personalizada establecida en 405 y el mensaje para **se le ha bloqueado.** uso de [New AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

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

Modificar el código de respuesta personalizado o configuración de cuerpo de respuesta de una directiva existente de WAF mediante [conjunto AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```azurepowershell
# modify WAF response code
Set-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 403
```

```azurepowershell
# modify WAF response body
Set-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-CustomBlockResponseBody "<html><head><title> Forbidden</title></head><body></body></html>"
```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [puerta delantera](front-door-overview.md)