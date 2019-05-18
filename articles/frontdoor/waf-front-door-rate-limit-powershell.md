---
title: 'Configurar una regla de límite de tasa de web application firewall para la puerta delantera: Azure PowerShell'
description: Obtenga información sobre cómo configurar una regla de límite de velocidad para un punto de conexión de puerta de entrada existente.
services: frontdoor
documentationcenter: ''
author: KumudD
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/16/2019
ms.author: kumud;tyao
ms.openlocfilehash: 3701a69ab72abf20a4f1608a1cee56c9cea38aca
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523634"
---
# <a name="configure-a-web-application-firewall-rate-limit-rule-using-azure-powershell"></a>Configurar una regla de límite web application firewall frecuencia mediante Azure PowerShell
La regla de límites de velocidad de web de Azure aplicación firewall (WAF) para el acceso de principal de Azure controla el número de solicitudes permitidas desde una dirección IP de cliente único durante una duración de un minuto.
En este artículo se muestra cómo configurar una regla de límite de tasa de WAF que controla el número de solicitudes permitidas desde un solo cliente para una aplicación web que contenga */promo* en la dirección URL con Azure PowerShell.

> [!IMPORTANT]
> La característica de regla de límite de velocidad de WAF para Azure puerta de entrada está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar a configurar una directiva de límite de velocidad, configure el entorno de PowerShell y crear un perfil de la puerta delantera.
### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones en la página, inicie sesión con sus credenciales de Azure e instale el módulo de PowerShell de Az.

#### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Conexión a Azure con un cuadro de diálogo interactivo para el inicio de sesión
```
Connect-AzAccount

```
Antes de instalar el módulo de Front Door asegúrese de que tiene instalada la última versión de PowerShellGet. Ejecute el comando siguiente y vuelva a abrir PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 

#### <a name="install-azfrontdoor-module"></a>Instalación del módulo Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```
### <a name="create-a-front-door-profile"></a>Creación de un perfil de Front Door
Para crear un perfil de Front Door siga las instrucciones que se describen en [Inicio rápido: Crear un perfil de la puerta delantera](quickstart-create-front-door.md)

## <a name="define-url-match-conditions"></a>Definir condiciones de coincidencia url
Definir una condición de coincidencia de dirección URL (dirección URL contiene /promo) mediante [New AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject).
En el ejemplo siguiente se buscan */promo* como el valor de la *RequestUri* variable:

```powershell-interactive
   $promoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
     -MatchVariable RequestUri `
     -OperatorProperty Contains `
     -MatchValue "/promo"
```
## <a name="create-a-custom-rate-limit-rule"></a>Crear una regla de límite de velocidad personalizada
Establecer un límite de velocidad mediante [New AzFrontDoorCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject). En el ejemplo siguiente, el límite se establece en 1000. Las solicitudes desde cualquier cliente a la página de promoción superior a 1000 durante un minuto se bloquean hasta que se inicie el siguiente minuto.

```powershell-interactive
   $promoRateLimitRule = New-AzFrontDoorCustomRuleObject `
     -Name "rateLimitRule" `
     -RuleType RateLimitRule `
     -MatchCondition $promoMatchCondition `
     -RateLimitThreshold 1000 `
     -Action Block -Priority 1
```


## <a name="configure-a-security-policy"></a>Configurar una directiva de seguridad

Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzureRmResourceGroup`. A continuación, configure una directiva de seguridad con una regla de límite de velocidad personalizada mediante [New AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) en el grupo de recursos especificado que contiene el perfil de la puerta de entrada.

En el ejemplo siguiente se usa el nombre de grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](quickstart-create-front-door.md).

 uso de [New AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```powershell-interactive
   $ratePolicy = New-AzFrontDoorWafPolicy `
     -Name "RateLimitPolicyExamplePS" `
     -resourceGroupName myResourceGroupFD1 `
     -Customrule $promoRateLimitRule `
     -Mode Prevention `
     -EnabledState Enabled
```
## <a name="link-policy-to-a-front-door-front-end-host"></a>Directiva de vínculo a un host de front-end de puerta de entrada
Vincular el objeto de directiva de seguridad a un host de front-end de puerta principal existente y actualizar las propiedades de la puerta de entrada. Primero recupere el objeto de la puerta delantera con [Get AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor) comando.
A continuación, establezca el front-end *WebApplicationFirewallPolicyLink* propiedad a la *resourceId* de creado en el paso anterior mediante "$ratePolicy" [conjunto AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor) comando. 

En el ejemplo siguiente se usa el nombre de grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](quickstart-create-front-door.md). Además, en el ejemplo siguiente, reemplace $frontDoorName con el nombre de su perfil de la puerta de entrada. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $ratePolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Deberá establecer *WebApplicationFirewallPolicyLink* propiedad una vez para vincular una directiva de seguridad a una puerta de front-end. Las actualizaciones posteriores de las directivas se aplican automáticamente para el front-end.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [puerta delantera](front-door-overview.md) 


