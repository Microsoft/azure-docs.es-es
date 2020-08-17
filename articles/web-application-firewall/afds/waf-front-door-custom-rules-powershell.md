---
title: Configuración de reglas personalizadas de WAF y conjunto de reglas predeterminado para Azure Front Door
description: Obtenga información sobre cómo configurar un directiva del WAF con reglas personalizadas y administradas para un punto de conexión de Front Door existente.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 09/05/2019
ms.author: victorh
ms.openlocfilehash: 40f0195ced375da09891eb577bd81f36b2f37cba
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005555"
---
# <a name="configure-a-web-application-firewall-policy-using-azure-powershell"></a>Configuración de una directiva del firewall de aplicaciones web con Azure PowerShell

Una directiva del firewall de aplicaciones web (WAF) de Azure define las inspecciones necesarias cuando llega una solicitud a Front Door.
En este artículo se muestra cómo configurar una directiva del WAF que se compone de algunas reglas personalizadas y con el conjunto de reglas predeterminado administrado por Azure habilitado.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar a configurar una directiva de limitación de velocidad, configure el entorno de PowerShell y cree un perfil de Front Door.

### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell

Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones de la página para iniciar sesión con sus credenciales de Azure e instalar el módulo Az de PowerShell.

#### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

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

Para crear un perfil de Front Door siga las instrucciones que se describen en [Inicio rápido: Creación de un perfil de Front Door](../../frontdoor/quickstart-create-front-door.md)

## <a name="custom-rule-based-on-http-parameters"></a>Regla personalizada basada en parámetros HTTP

El ejemplo siguiente muestra cómo configurar una regla personalizada con dos condiciones de coincidencia con [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject). Las solicitudes provienen de un sitio especificado, según se define en el origen de referencia y la cadena de consulta no contiene "password". 

```powershell-interactive
$referer = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestHeader -OperatorProperty Equal -Selector "Referer" -MatchValue "www.mytrustedsites.com/referpage.html"
$password = New-AzFrontDoorWafMatchConditionObject -MatchVariable QueryString -OperatorProperty Contains -MatchValue "password"
$AllowFromTrustedSites = New-AzFrontDoorWafCustomRuleObject -Name "AllowFromTrustedSites" -RuleType MatchRule -MatchCondition $referer,$password -Action Allow -Priority 1
```

## <a name="custom-rule-based-on-http-request-method"></a>Regla personalizada basada en el método de solicitud HTTP

Puede crear una regla de bloqueo del método "PUT" con [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject) como sigue:

```powershell-interactive
$put = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestMethod -OperatorProperty Equal -MatchValue PUT
$BlockPUT = New-AzFrontDoorWafCustomRuleObject -Name "BlockPUT" -RuleType MatchRule -MatchCondition $put -Action Block -Priority 2
```

## <a name="create-a-custom-rule-based-on-size-constraint"></a>Creación de una regla personalizada basada en una restricción de tamaño

El ejemplo siguiente crea una regla que bloquea las solicitudes con una dirección URL de más de 100 caracteres con Azure PowerShell:
```powershell-interactive
$url = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestUri -OperatorProperty GreaterThanOrEqual -MatchValue 100
$URLOver100 = New-AzFrontDoorWafCustomRuleObject -Name "URLOver100" -RuleType MatchRule -MatchCondition $url -Action Block -Priority 3
```
## <a name="add-managed-default-rule-set"></a>Adición del conjunto de reglas predeterminado administrado

El ejemplo siguiente crea un conjunto de reglas predeterminado administrado con Azure PowerShell:
```powershell-interactive
$managedRules =  New-AzFrontDoorWafManagedRuleObject -Type DefaultRuleSet -Version 1.0
```
## <a name="configure-a-security-policy"></a>Configuración de una directiva de seguridad

Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzResourceGroup`. A continuación, configure una directiva de seguridad con las reglas creadas en los pasos anteriores mediante [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) en el grupo de recursos especificado que contiene el perfil de Front Door.

```powershell-interactive
$myWAFPolicy=New-AzFrontDoorWafPolicy -Name $policyName -ResourceGroupName $resourceGroupName -Customrule $AllowFromTrustedSites,$BlockPUT,$URLOver100 -ManagedRule $managedRules -EnabledState Enabled -Mode Prevention
```

## <a name="link-policy-to-a-front-door-front-end-host"></a>Vinculación de una directiva a un host de front-end de Front Door

Vincule el objeto de directiva de seguridad a un host de front-end de Front Door existente y actualice las propiedades de Front Door. En primer lugar, recupere el objeto de Front Door mediante [Get-AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor).
A continuación, establezca la propiedad *WebApplicationFirewallPolicyLink* de front-end en el valor de *resourceId* del elemento "$myWAFPolicy$" creado en el paso anterior con [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor). 

En el ejemplo siguiente se usa el nombre de grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md). Además, en el ejemplo siguiente, reemplace $frontDoorName por el nombre de su perfil de Front Door. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $myWAFPolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Solo necesita establecer la propiedad *WebApplicationFirewallPolicyLink* una vez para vincular una directiva de seguridad a un front-end de Front Door. Las posteriores actualizaciones de la directiva se aplican al front-end automáticamente.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Front Door](../../frontdoor/front-door-overview.md) 
- Más información sobre [WAF con Front Door](afds-overview.md)
