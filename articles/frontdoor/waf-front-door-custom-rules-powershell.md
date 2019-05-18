---
title: 'Configurar una directiva de firewall (WAF) de aplicaciones web con reglas personalizadas y el conjunto de rUtilice predeterminado para la puerta delantera: Azure PowerShell'
description: Obtenga información sobre cómo configurar un WAF directiva constan de reglas personalizadas y administradas para un punto de conexión de puerta de entrada existente.
services: frontdoor
documentationcenter: ''
author: KumudD
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/08/2019
ms.author: kumud;tyao
ms.openlocfilehash: 414869833b894e2688505a91fed8fafe0c912b73
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523733"
---
# <a name="configure-a-web-application-firewall-policy-using-azure-powershell"></a>Configurar una directiva de firewall de aplicaciones web con Azure PowerShell
Directiva de firewall (WAF) de aplicaciones web de Azure define inspecciones necesarias cuando llega una solicitud de acceso principal.
En este artículo se muestra cómo configurar una directiva de WAF que se compone de algunas reglas personalizadas y con administradas por Azure rUtilice conjunto predeterminado habilitado.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar a configurar una directiva de límite de velocidad, configure el entorno de PowerShell y crear un perfil de la puerta delantera.
### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones en la página, inicie sesión con sus credenciales de Azure e instale el módulo de PowerShell de Az.

#### <a name="sign-in-to-azure"></a>Iniciar sesión en Azure
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

## <a name="custom-rule-based-on-http-parameters"></a>Regla personalizada en función de los parámetros de http

El ejemplo siguiente muestra cómo configurar una regla personalizada con dos condiciones de coincidencia con [New AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject). Las solicitudes proceden de un sitio especificado según se define en el origen de referencia y cadena de consulta no contiene "password". 

```powershell-interactive
$referer = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestHeader -OperatorProperty Equal -Selector "Referer" -MatchValue "www.mytrustedsites.com/referpage.html"
$password = New-AzFrontDoorWafMatchConditionObject -MatchVariable QueryString -OperatorProperty Contains -MatchValue "password"
$AllowFromTrustedSites = New-AzFrontDoorCustomRuleObject -Name "AllowFromTrustedSites" -RuleType MatchRule -MatchCondition $referer,$password -Action Allow -Priority 1
```

## <a name="custom-rule-based-on-http-request-method"></a>Regla personalizada basada en método de solicitud http
Crear una regla de bloqueo "PUT" utilizando método [New AzFrontDoorCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject) como sigue:

```powershell-interactive
$put = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestMethod -OperatorProperty Equal -MatchValue PUT
$BlockPUT = New-AzFrontDoorCustomRuleObject -Name "BlockPUT" -RuleType MatchRule -MatchCondition $put -Action Block -Priority 2
```

## <a name="create-a-custom-rule-based-on-size-constraint"></a>Crear una regla personalizada en función de la restricción de tamaño

El ejemplo siguiente crea una regla que bloquea las solicitudes con la dirección Url que tiene más de 100 caracteres con Azure PowerShell:
```powershell-interactive
$url = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestUri -OperatorProperty GreaterThanOrEqual -MatchValue 100
$URLOver100 = New-AzFrontDoorCustomRuleObject -Name "URLOver100" -RuleType MatchRule -MatchCondition $url -Action Block -Priority 3
```
## <a name="add-managed-default-rule-set"></a>Agregar regla administrado predeterminado establecido

El ejemplo siguiente crea un administrado predeterminado conjunto de reglas con Azure PowerShell:
```powershell-interactive
$managedRules = New-AzFrontDoorManagedRuleObject -Type DefaultRuleSet -Version "preview-0.1"
```
## <a name="configure-a-security-policy"></a>Configurar una directiva de seguridad

Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzResourceGroup`. A continuación, configure una directiva de seguridad con las reglas creadas en los pasos anteriores con [New AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) en el grupo de recursos especificado que contiene el perfil de la puerta de entrada.

```powershell-interactive
$myWAFPolicy=New-AzFrontDoorWafPolicy -Name $policyName -ResourceGroupName $resourceGroupName -Customrule $AllowFromTrustedSites,$BlockPUT,$URLOver100 -ManagedRule $managedRules -EnabledState Enabled -Mode Prevention
```

## <a name="link-policy-to-a-front-door-front-end-host"></a>Directiva de vínculo a un host de front-end de puerta de entrada
Vincular el objeto de directiva de seguridad a un host de front-end de puerta principal existente y actualizar las propiedades de la puerta de entrada. En primer lugar, recupere el objeto de la puerta delantera con [Get AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor).
A continuación, establezca el front-end *WebApplicationFirewallPolicyLink* propiedad a la *resourceId* el creado en el paso anterior mediante "$myWAFPolicy $" [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor). 

En el ejemplo siguiente se usa el nombre de grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](quickstart-create-front-door.md). Además, en el ejemplo siguiente, reemplace $frontDoorName con el nombre de su perfil de la puerta de entrada. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $myWAFPolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Deberá establecer *WebApplicationFirewallPolicyLink* propiedad una vez para vincular una directiva de seguridad a una puerta de front-end. Las actualizaciones posteriores de las directivas se aplican automáticamente para el front-end.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [puerta delantera](front-door-overview.md) 
- Obtenga más información sobre [WAF para puerta de entrada](waf-overview.md)
