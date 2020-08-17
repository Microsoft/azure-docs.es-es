---
title: Configuración de una regla de límite de velocidad de WAF para el Azure PowerShell de Front Door
description: Aprenda a configurar una regla de limitación de volumen para un punto de conexión de Front Door existente.
author: vhorne
ms.service: web-application-firewall
ms.topic: article
services: web-application-firewall
ms.date: 02/26/2020
ms.author: victorh
ms.openlocfilehash: 29f50b2cf9523b9266de2f73607b0099f32852e1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005419"
---
# <a name="configure-a-web-application-firewall-rate-limit-rule-using-azure-powershell"></a>Configuración de una regla de límite de velocidad de Firewall de aplicaciones web con Azure PowerShell
La regla de límite de frecuencia del firewall de aplicaciones web (WAF) de Azure para Azure Front Door controla el número de solicitudes permitidas desde clientes durante un minuto.
En este artículo se muestra cómo configurar una regla de límite de frecuencia de WAF que controle el número de solicitudes permitidas desde clientes a una aplicación web que contiene */promo* en la dirección URL mediante Azure PowerShell.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar a configurar una directiva de limitación de velocidad, configure el entorno de PowerShell y cree un perfil de Front Door.
### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones de la página para iniciar sesión con sus credenciales de Azure e instalar el módulo Az de PowerShell.

#### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Conexión a Azure con un cuadro de diálogo interactivo para el inicio de sesión
```
Connect-AzAccount

```
Antes de instalar el módulo Front Door, asegúrese de tener instalada la versión actual de PowerShellGet. Ejecute el siguiente comando y vuelva a abrir PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 

#### <a name="install-azfrontdoor-module"></a>Instalación del módulo Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```
### <a name="create-a-front-door-profile"></a>Creación de un perfil de Front Door
Para crear un perfil de Front Door siga las instrucciones que se describen en [Inicio rápido: Creación de un perfil de Front Door](../../frontdoor/quickstart-create-front-door.md)

## <a name="define-url-match-conditions"></a>Definición de condiciones de coincidencia de URL
Defina una condición de coincidencia de URL (la URL contiene /promo) mediante [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject).
En el ejemplo siguiente se define el elemento */promo* con el valor de la variable *RequestUri*:

```powershell-interactive
   $promoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
     -MatchVariable RequestUri `
     -OperatorProperty Contains `
     -MatchValue "/promo"
```
## <a name="create-a-custom-rate-limit-rule"></a>Creación de una regla de limitación de volumen personalizada
Establezca un límite de volumen mediante [New AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject). En el ejemplo siguiente, el límite se establece en 1000. Las solicitudes desde cualquier cliente a la página de promoción que superan el valor de 1000 durante un minuto se bloquean hasta que se inicia el siguiente minuto.

```powershell-interactive
   $promoRateLimitRule = New-AzFrontDoorWafCustomRuleObject `
     -Name "rateLimitRule" `
     -RuleType RateLimitRule `
     -MatchCondition $promoMatchCondition `
     -RateLimitThreshold 1000 `
     -Action Block -Priority 1
```


## <a name="configure-a-security-policy"></a>Configuración de una directiva de seguridad

Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzureRmResourceGroup`. A continuación, configure una directiva de seguridad con una regla de limitación de volumen personalizada mediante [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) en el grupo de recursos especificado que contiene el perfil de Front Door.

En el ejemplo siguiente se usa el nombre de grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md).

 mediante [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```powershell-interactive
   $ratePolicy = New-AzFrontDoorWafPolicy `
     -Name "RateLimitPolicyExamplePS" `
     -resourceGroupName myResourceGroupFD1 `
     -Customrule $promoRateLimitRule `
     -Mode Prevention `
     -EnabledState Enabled
```
## <a name="link-policy-to-a-front-door-front-end-host"></a>Vinculación de una directiva a un host de front-end de Front Door
Vincule el objeto de directiva de seguridad a un host de front-end de Front Door existente y actualice las propiedades de Front Door. En primer lugar, recupere el objeto de Front Door mediante el comando [Get-AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor).
A continuación, establezca la propiedad *WebApplicationFirewallPolicyLink* de front-end en el valor de *resourceId* del elemento "$ratePolicy" creado en el paso anterior, mediante el uso del comando [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor). 

En el ejemplo siguiente se usa el nombre de grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md). Además, en el ejemplo siguiente, reemplace $frontDoorName por el nombre de su perfil de Front Door. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $ratePolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Solo necesita establecer la propiedad *WebApplicationFirewallPolicyLink* una vez para vincular una directiva de seguridad a un front-end de Front Door. Las posteriores actualizaciones de la directiva se aplican al front-end automáticamente.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Front Door](../../frontdoor/front-door-overview.md). 


