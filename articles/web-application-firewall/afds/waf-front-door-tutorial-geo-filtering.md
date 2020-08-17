---
title: Configuración de la directiva de Firewall de aplicaciones web de filtrado geográfico para Azure Front Door Service
description: En este tutorial, aprenderá a crear una directiva de filtrado geográfico y a asociar la directiva con el host Front Door frontend existente
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: fd38d900285f6cda0c98dd1f49076ea04180b18c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068321"
---
# <a name="set-up-a-geo-filtering-waf-policy-for-your-front-door"></a>Configure una directiva de WAF de filtrado geográfico para Front Door

En este tutorial se muestra cómo usar Azure PowerShell para crear una directiva de filtrado geográfico de ejemplo y a asociarla con el host de front-end de Front Door existente. Esta directiva de filtrado geográfico de ejemplo bloqueará las solicitudes de todos los países o regiones, salvo Estados Unidos.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar a configurar una directiva de filtrado geográfico, configure el entorno de PowerShell y crear un perfil de Front Door.
### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones de la página para iniciar sesión con sus credenciales de Azure e instalar el módulo Az.PowerShell.

#### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Conexión a Azure con un cuadro de diálogo interactivo para el inicio de sesión

```
Install-Module -Name Az
Connect-AzAccount
```
Asegúrese de tener instalada la versión actual de PowerShellGet. Ejecute el comando siguiente y vuelva a abrir PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 
#### <a name="install-azfrontdoor-module"></a>Instalación del módulo Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```

### <a name="create-a-front-door-profile"></a>Creación de un perfil de Front Door

Para crear un perfil de Front Door siga las instrucciones que se describen en [Inicio rápido: Creación de un perfil de Front Door](../../frontdoor/quickstart-create-front-door.md).

## <a name="define-geo-filtering-match-condition"></a>Definición de la condición de coincidencia del filtrado geográfico

Cree una condición de coincidencia de ejemplo que selecciona las solicitudes que no proceden de "EE.UU.", utilizando [New AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) en los parámetros al crear una condición de coincidencia. Se proporcionan códigos de país o región de dos letras para la asignación de país o región en [¿Qué es el filtrado geográfico en un dominio para Azure Front Door?](waf-front-door-geo-filtering.md).

```azurepowershell-interactive
$nonUSGeoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable RemoteAddr `
-OperatorProperty GeoMatch `
-NegateCondition $true `
-MatchValue "US"
```
 
## <a name="add-geo-filtering-match-condition-to-a-rule-with-action-and-priority"></a>Adición de una condición de coincidencia de filtrado geográfico a una regla con Action y Priority

Cree un objeto CustomRule `nonUSBlockRule` en función de la condición de coincidencia, un valor de Action y un valor de Priority, para lo que debe usar [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject).  Un objeto CustomRule puede tener varios MatchCondition.  En este ejemplo, el valor de Action es Block y el de Priority es 1, la prioridad más alta.

```
$nonUSBlockRule = New-AzFrontDoorWafCustomRuleObject `
-Name "geoFilterRule" `
-RuleType MatchRule `
-MatchCondition $nonUSGeoMatchCondition `
-Action Block `
-Priority 1
```

## <a name="add-rules-to-a-policy"></a>Adición de reglas a una directiva

Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzResourceGroup`. A continuación, cree un objeto de directiva `geoPolicy` que contenga `nonUSBlockRule` para lo que debe usar [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) en el grupo de recursos especificado que contiene el perfil de Front Door. Debe especificar un nombre único para la directiva de replicación geográfica. 

En el ejemplo siguiente se usa el nombre del grupo de recursos *myResourceGroupFD1* con la suposición de que ha creado el perfil de Front Door con las instrucciones proporcionadas en el [Inicio rápido: Creación de una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md). En el ejemplo siguiente, reemplace el nombre de la directiva *geoPolicyAllowUSOnly* con un nombre de directiva único.

```
$geoPolicy = New-AzFrontDoorWafPolicy `
-Name "geoPolicyAllowUSOnly" `
-resourceGroupName myResourceGroupFD1 `
-Customrule $nonUSBlockRule  `
-Mode Prevention `
-EnabledState Enabled
```

## <a name="link-waf-policy-to-a-front-door-frontend-host"></a>Vinculación de una directiva de WAF a un host de front-end de Front Door

Vincule el objeto de directiva de WAF al host de front-end de Front Door existente y actualice las propiedades de Front Door. 

Para ello, en primer lugar recupere el objeto Front Door mediante [Get-AzFrontDoor](/powershell/module/az.frontdoor/get-azfrontdoor). 

```
$geoFrontDoorObjectExample = Get-AzFrontDoor -ResourceGroupName myResourceGroupFD1
$geoFrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $geoPolicy.Id
```

Luego, establezca la propiedad WebApplicationFirewallPolicyLink de front-end en el valor de resourceId de `geoPolicy`mediante [Set-AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```
Set-AzFrontDoor -InputObject $geoFrontDoorObjectExample[0]
```

> [!NOTE] 
> La propiedad WebApplicationFirewallPolicyLink solo es preciso establecerse una vez para vincular una directiva de WAF a un host de front-end de Front Door. Las posteriores actualizaciones de la directiva se aplican al host del front-end.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [firewall de aplicaciones web de Azure](../overview.md).
- Aprenda a [crear una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md).
