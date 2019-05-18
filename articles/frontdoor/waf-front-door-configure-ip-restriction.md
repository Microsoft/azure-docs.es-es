---
title: Configurar una regla de restricción de IP con la regla de firewall de aplicaciones web para el acceso de principal de Azure
description: Obtenga información sobre cómo configurar una regla de WAF de restricción de direcciones IP para un punto de conexión de puerta de entrada existente.
services: frontdoor
documentationcenter: ''
author: KumudD
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: kumud;tyao
ms.openlocfilehash: b129579916330a34a2a78d98f2c7653f129d3319
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523711"
---
# <a name="configure-an-ip-restriction-rule-with-web-application-firewall-for-azure-front-door-preview"></a>Configurar una regla de restricción de IP con firewall de aplicaciones web para Azure puerta de entrada (versión preliminar)
 Este artículo muestra cómo configurar las reglas de restricción de IP en firewall de aplicaciones web de Azure (WAF) para la puerta de entrada mediante el uso de la plantilla de Azure Resource Manager, Azure PowerShell o CLI de Azure.

Una regla de control de acceso basados en direcciones IP es una regla de WAF personalizada que le permite controlar el acceso a las aplicaciones web mediante la especificación de una lista de direcciones IP o intervalos de direcciones IP en formato de enrutamiento entre dominios sin clase (CIDR).

De forma predeterminada, la aplicación web es accesible desde internet. Si desea limitar el acceso a sus aplicaciones web solo a los clientes de una lista de direcciones IP conocidas o intervalos de direcciones IP, deberá crear dos reglas de coincidencia de IP. Primera regla de coincidencia de IP contiene la lista de direcciones IP como valores coincidentes y establezca la acción para "Permitir". El segundo con una prioridad más baja, es bloquear todas las demás direcciones IP mediante el operador "All" y establezca la acción para "Bloque". Una vez que se aplique una regla de restricción de IP, las solicitudes procedentes de direcciones que están fuera de esta lista de permitidos recibe una respuesta 403 (prohibido).  

> [!IMPORTANT]
> La característica de restricción de IP de WAF para Azure puerta de entrada está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="configure-waf-policy-with-azure-cli"></a>Configurar la directiva de WAF con CLI de Azure

### <a name="prerequisites"></a>Requisitos previos
Antes de empezar a configurar una directiva de restricción de IP, configure el entorno de CLI y crear un perfil de la puerta delantera.

#### <a name="set-up-azure-cli-environment"></a>Configurar el entorno de CLI de Azure
1. Instalar el [CLI de Azure](/cli/azure/install-azure-cli), o usar Azure Cloud Shell. Azure Cloud Shell es un shell de Bash gratuito que se puede ejecutar directamente en Azure Portal. Tiene la CLI de Azure preinstalada y configurada para utilizar con su cuenta. Seleccione el **Pruébelo** botón en la CLI de los comandos siguientes. Seleccionar **Pruébelo** invoca un Cloud Shell que pueda iniciar sesión con su cuenta de Azure. Una vez que se inicia una sesión de cloud shell, escriba `az extension add --name front-door` para agregar la extensión de la puerta principal.
 2. Si usa la CLI localmente en Bash, inicie sesión en Azure con `az login`.

#### <a name="create-front-door-profile"></a>Crear perfil de la puerta de entrada
Para crear un perfil de Front Door siga las instrucciones que se describen en [Inicio rápido: Crear un perfil de la puerta delantera](quickstart-create-front-door.md)

### <a name="create-a-waf-policy"></a>Crear una directiva de WAF

Cree una directiva de WAF con el [waf-directiva de red az crear](/cli/azure/ext/front-door/network/waf-policy?view=azure-cli-latest#ext-front-door-az-network-waf-policy-create) comando. En el ejemplo siguiente, reemplace el nombre de la directiva *IPAllowPolicyExampleCLI* con un nombre de directiva único.

```azurecli-interactive 
az network waf-policy create \
  --resource-group <resource-group-name> \
  --subscription <subscription ID> \
  --name IPAllowPolicyExampleCLI
  ```
### <a name="add-custom-ip-access-control-rule"></a>Agregar regla de control de acceso IP personalizado

Agregar una regla de control de acceso IP personalizada a la directiva de WAF que se creó en el paso anterior con el [crear az red waf-policy regla personalizada](/cli/azure/ext/front-door/network/waf-policy/custom-rule?view=azure-cli-latest#ext-front-door-az-network-waf-policy-custom-rule-create) comando. 

En el ejemplo siguiente:
-  Reemplace *IPAllowPolicyExampleCLI* con la directiva única que creó anteriormente.
-  Reemplace *ip-direcciones-intervalo de-1*, *ip-direcciones-intervalo de-2* con su propio intervalo.

En primer lugar, cree la dirección IP que la regla para las direcciones especificadas.

```azurecli
az network waf-policy custom-rule create \
  --name IPAllowListRule \
  --priority 1 \
  --rule-type MatchRule \
  --match-condition RemoteAddr IPMatch ["<ip-address-range-1>","<ip-address-range-2>"] \
  --action Allow \
  --resource-group <resource-group-name> \
  --policy-name IPAllowPolicyExampleCLI
```
A continuación, cree un bloque de todas las reglas de IP con una prioridad menor que la regla de permiso de la dirección IP anterior. Reemplace el *IPAllowPolicyExampleCLI* con la directiva única que creó anteriormente.

```azurecli
az network waf-policy custom-rule create \
  --name IPDenyAllRule\
  --priority 2 \
  --rule-type MatchRule \
  --match-condition RemoteAddr Any
  --action Block \
  --resource-group <resource-group-name> \
  --policy-name IPAllowPolicyExampleCLI
 ```

### <a name="find-waf-policy-id"></a>Buscar Id. de directiva de WAF
Buscar el identificador de una directiva de WAF con el [show de waf-directiva de red az](/cli/azure/ext/front-door/network/waf-policy?view=azure-cli-latest#ext-front-door-az-network-waf-policy-show) comando. Reemplace el *IPAllowPolicyExampleCLI* con la directiva única que creó anteriormente.

   ```azurecli
   az network waf-policy show \
     --resource-group <resource-group-name> \
     --name IPAllowPolicyExampleCLI
   ```

### <a name="link-waf-policy-to-a-front-door-front-end-host"></a>Directiva de WAF de vínculo a un host de front-end de puerta de entrada

Establecer la puerta delantera *WebApplicationFirewallPolicyLink* identificador para el identificador de directiva con el [actualización de puerta principal de red az](/cli/azure/ext/front-door/network/front-door?view=azure-cli-latest#ext-front-door-az-network-front-door-update) comando. Reemplace el *IPAllowPolicyExampleCLI* con la directiva única que creó anteriormente.

   ```azurecli
   az network front-door update \
     --set FrontendEndpoints[0].WebApplicationFirewallPolicyLink.id=/subscriptions/<subscription ID>/resourcegroups/<resource- name>/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/IPAllowPolicyExampleCLI \
     --name <frontdoor-name>
     --resource-group <resource-group-name>
   ```
En este ejemplo, la directiva de WAF se aplica a FrontendEndpoints [0]. Puede vincular directivas WAF a cualquiera de los servidores front-end.
> [!Note]
> Deberá establecer el **WebApplicationFirewallPolicyLink** propiedad una vez que se va a vincular una directiva de WAF a una puerta de front-end. Las actualizaciones posteriores de las directivas se aplican automáticamente para el front-end.

## <a name="configure-waf-policy-with-azure-powershell"></a>Configurar la directiva de WAF con Azure PowerShell

### <a name="prerequisites"></a>Requisitos previos
Antes de empezar a configurar una directiva de restricción de IP, configure el entorno de PowerShell y crear un perfil de la puerta de entrada.

#### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell ofrece un conjunto de cmdlets que usan el modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure. 

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones en la página, inicie sesión con sus credenciales de Azure e instale el módulo de PowerShell de Az.

##### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Conectarse a Azure con un diálogo de inicio de sesión interactivo
```
Connect-AzAccount

```
Antes de instalar el módulo de puerta de entrada, asegúrese de que tiene la versión actual de PowerShellGet instalado. Ejecute el siguiente comando y vuelva a abrir PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 

##### <a name="install-azfrontdoor-module"></a>Instalación del módulo Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```
### <a name="create-a-front-door-profile"></a>Creación de un perfil de Front Door
Para crear un perfil de Front Door siga las instrucciones que se describen en [Inicio rápido: Crear un perfil de la puerta delantera](quickstart-create-front-door.md)

### <a name="define-ip-match-condition"></a>Definir condición de coincidencia IP
Use la [New AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) comando para definir una condición de coincidencia IP. En el ejemplo siguiente, reemplace *ip-direcciones-intervalo de-1*, *ip-direcciones-intervalo de-2* con su propio intervalo.

```powershell
  $IPMatchCondition = New-AzFrontDoorWafMatchConditionObject `
    -MatchVariable  RemoteAddr `
    -OperatorProperty IPMatch `
    -MatchValue ["ip-address-range-1", "ip-address-range-2"]
```
Crear a una coincidencia de IP de todas las reglas de condición
```powershell
  $IPMatchALlCondition = New-AzFrontDoorWafMatchConditionObject `
    -MatchVariable  RemoteAddr `
    -OperatorProperty Any
    
```

### <a name="create-a-custom-ip-allow-rule"></a>Crear un personalizado direcciones IP permitidas de regla
   Use la [New AzFrontDoorCustomRuleObject](/powershell/module/Az.FrontDoor/New-azfrontdoorwafcustomruleobject) comando para definir una acción y establecer una prioridad. En el ejemplo siguiente, se permitirán las solicitudes de direcciones IP de cliente que coincide con la lista. 

```powershell
  $IPAllowRule = New-AzFrontDoorCustomRuleObject `
    -Name "IPAllowRule" `
    -RuleType MatchRule `
    -MatchCondition $IPMatchCondition `
    -Action Allow -Priority 1
```
Cree un bloque en todas las reglas de IP con una prioridad menor que la regla de permiso de la dirección IP anterior.

```powershell
  $IPBlockAll = New-AzFrontDoorCustomRuleObject `
    -Name "IPDenyAll" `
    -RuleType MatchRule `
    -MatchCondition $IPMatchALlCondition `
    -Action Block `
    -Priority 2
   ```

### <a name="configure-waf-policy"></a>Configurar la directiva de WAF
Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzResourceGroup`. A continuación, configure una directiva de WAF con la regla de bloque IP mediante [New AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```powershell
  $IPAllowPolicyExamplePS = New-AzFrontDoorWafPolicy `
    -Name "IPRestrictionExamplePS" `
    -resourceGroupName <resource-group-name> `
    -Customrule $IPAllowRule $IPBlockAll `
    -Mode Prevention `
    -EnabledState Enabled
   ```

### <a name="link-waf-policy-to-a-front-door-front-end-host"></a>Directiva de WAF de vínculo a un host de front-end de puerta de entrada

Vincule el objeto de directiva de WAF en un host de front-end de puerta principal existente y actualizar las propiedades de la puerta de entrada. Primero recupere el objeto de la puerta delantera con [Get AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor). A continuación, establezca el front-end *WebApplicationFirewallPolicyLink* propiedad en el valor de resourceId de la *$IPAllowPolicyExamplePS* creado en el paso anterior con el [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor) comando.

```powershell
  $FrontDoorObjectExample = Get-AzFrontDoor `
    -ResourceGroupName <resource-group-name> `
    -Name $frontDoorName
  $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $IPBlockPolicy.Id
  Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
```

> [!NOTE]
> En este ejemplo, la directiva de WAF se aplica a FrontendEndpoints [0]. Puede vincular directivas WAF a cualquiera de los servidores front-end. Deberá establecer *WebApplicationFirewallPolicyLink* propiedad una vez que se va a vincular una directiva de WAF a una puerta de front-end. Las actualizaciones posteriores de las directivas se aplican automáticamente para el front-end.


## <a name="configure-waf-policy-with-resource-manager-template"></a>Configurar la directiva de WAF con plantilla de Resource Manager
Ver la plantilla que crea una puerta de entrada y una directiva de WAF con reglas de restricción de IP personalizadas [aquí](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-clientip).


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [crear un perfil de la puerta delantera](quickstart-create-front-door.md).
