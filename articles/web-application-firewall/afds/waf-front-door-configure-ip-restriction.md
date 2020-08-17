---
title: Configuración de la regla de WAF de restricción de IP para Azure Front Door
description: Aprenda a configurar una regla de firewall de aplicaciones web para restringir las direcciones IP de un punto de conexión existente de Azure Front Door.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 03/26/2020
ms.author: tyao
ms.openlocfilehash: f41dc688996b2431060a3cde209ca1ed4a21fe8c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005623"
---
# <a name="configure-an-ip-restriction-rule-with-a-web-application-firewall-for-azure-front-door"></a>Configuración de una regla de restricción de IP con un firewall de aplicaciones web para Azure Front Door

Este artículo muestra cómo configurar las reglas de restricción de IP en un firewall de aplicaciones web (WAF) para Azure Front Door con Azure Portal, la CLI de Azure, Azure PowerShell o una plantilla de Azure Resource Manager.

Una regla de control de acceso basado en la dirección IP es una regla de WAF personalizada que permite controlar el acceso a las aplicaciones web. Para ello, especifica una lista de direcciones IP o rangos de direcciones IP en formato de Enrutamiento entre dominios sin clase (CIDR).

De forma predeterminada, se puede acceder a la aplicación web desde Internet. Si desea limitar el acceso a los clientes de una lista de direcciones IP conocidas o intervalos de direcciones IP, puede crear una regla de coincidencia de IP que contenga la lista de direcciones IP como valores coincidentes y establece el operador en "Not" (la negación es verdadera) y la acción en **Bloquear**. Después de que se aplique una regla de restricción de IP, las solicitudes que provengan de direcciones que no se encuentren en esta lita de permitidos reciben una respuesta 403 Prohibido.

## <a name="configure-a-waf-policy-with-the-azure-portal"></a>Configuración de una directiva de WAF con Azure Portal

### <a name="prerequisites"></a>Prerrequisitos

Para crear un perfil de Azure Front Door, siga las instrucciones que se describen en [Inicio rápido: Creación de una instancia de Front Door para una aplicación web global de alta disponibilidad](../../frontdoor/quickstart-create-front-door.md).

### <a name="create-a-waf-policy"></a>Creación de una directiva WAF

1. En Azure Portal, seleccione **Crear un recurso**, escriba **Web application firewall** en el cuadro de búsqueda y, a continuación, seleccione **Web Application Firewall (WAF)** .
2. Seleccione **Crear**.
3. En la página **Crear una directiva WAF**, use los valores siguientes para completar la pestaña **Aspectos básicos**:
   
   |Configuración  |Value  |
   |---------|---------|
   |Directiva de     |WAF global (Front Door)|
   |Subscription     |Seleccione su suscripción.|
   |Resource group     |Seleccione el grupo de recursos en que se encuentra Front Door.|
   |Nombre de la directiva     |Escriba un nombre para la directiva.|
   |Estado de directiva     |habilitado|

   Seleccione **Siguiente: Configuración de directivas**

1. En la pestaña **Configuración de directiva**, seleccione **Prevención**. En **Bloquear cuerpo de respuesta**, escriba *You've been blocked!* para poder ver que la regla personalizada está en vigor.
2. Seleccione **Siguiente: Reglas administradas**.
3. Seleccione **Siguiente: Reglas personalizadas**.
4. Seleccione **Agregar regla personalizada**.
5. En la página **Agregar regla personalizada**, use los siguientes valores de prueba para crear una regla personalizada:

   |Configuración  |Value  |
   |---------|---------|
   |Nombre de la regla personalizada     |FdWafCustRule|
   |Status     |habilitado|
   |Tipo de regla     |Match|
   |Priority    |100|
   |Tipo de coincidencia     |Dirección IP|
   |Variable de coincidencia|RemoteAddr|
   |Operación|No contiene|
   |Dirección o intervalo de direcciones IP|10.10.10.0/24|
   |Entonces|Deny traffic|

   :::image type="content" source="../media/waf-front-door-configure-ip-restriction/custom-rule.png" alt-text="Regla personalizada":::

   Seleccione **Agregar**.
6. Seleccione **Siguiente: Asociación**.
7. Seleccione **Agregar un host de front-end**.
8. En **Host de front-end**, seleccione el host de front-end y, a continuación, **Agregar**.
9. Seleccione **Revisar + crear**.
10. Una vez pasada la validación de la directiva, seleccione **Crear**.

### <a name="test-your-waf-policy"></a>Prueba de la directiva WAF

1. Una vez finalizada la implementación de la directiva WAF, busque el nombre del host de front-end de Front Door.
2. Debería ver el mensaje de bloque personalizado.

   :::image type="content" source="../media/waf-front-door-configure-ip-restriction/waf-rule-test.png" alt-text="Prueba de regla de WAF":::

   > [!NOTE]
   > Se usó una dirección IP privada deliberadamente en la regla personalizada para garantizar que la regla se desencadene. En una implementación real, cree las reglas *allow* y *deny* mediante direcciones IP para su situación concreta.

## <a name="configure-a-waf-policy-with-the-azure-cli"></a>Configuración de una directiva de WAF con la CLI de Azure

### <a name="prerequisites"></a>Prerrequisitos
Antes de empezar a configurar una directiva de restricción de IP, configure el entorno de CLI y cree un perfil de Azure Front Door.

#### <a name="set-up-the-azure-cli-environment"></a>Configuración del entorno de la CLI de Azure
1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) o use Azure Cloud Shell. Azure Cloud Shell es un shell de Bash gratuito que puede ejecutarse directamente en Azure Portal. Tiene la CLI de Azure preinstalada y configurada para utilizarla con la cuenta. Seleccione el botón **Probar** en los siguientes comandos de la CLI y, después, inicie sesión con su cuenta de Azure en la sesión de Cloud Shell que se abre. Una vez iniciada la sesión, escriba `az extension add --name front-door` para agregar la extensión de Azure Front Door.
 2. Si usa la CLI localmente en Bash, inicie sesión en Azure mediante `az login`.

#### <a name="create-an-azure-front-door-profile"></a>Creación de un perfil de Azure Front Door
Para crear un perfil de Azure Front Door, siga las instrucciones que se describen en [Inicio rápido: Creación de una instancia de Front Door para una aplicación web global de alta disponibilidad](../../frontdoor/quickstart-create-front-door.md).

### <a name="create-a-waf-policy"></a>Creación de una directiva WAF

Cree una directiva WAF mediante el comando [az network front-door waf-policy create](/cli/azure/ext/front-door/network/front-door/waf-policy?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-create). En el ejemplo que se indica a continuación, reemplace el nombre de la directiva *IPAllowPolicyExampleCLI* por un nombre de directiva único.

```azurecli-interactive 
az network front-door waf-policy create \
  --resource-group <resource-group-name> \
  --subscription <subscription ID> \
  --name IPAllowPolicyExampleCLI
  ```
### <a name="add-a-custom-ip-access-control-rule"></a>Adición de una regla de control de acceso por IP personalizada

Use el comando [az network front-door waf-policy custom-rule create](/cli/azure/ext/front-door/network/front-door/waf-policy/rule?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-rule-create) para agregar una regla de control de acceso por IP personalizada para la directiva WAF que acaba de crear.

En los siguientes ejemplos:
-  Reemplace *IPAllowPolicyExampleCLI* por la directiva única que creó anteriormente.
-  Reemplace *ip-address-range-1*, *ip-address-range-2* por su propio rango.

En primer lugar, cree una regla de permiso de IP para la directiva creada en el paso anterior. 
> [!NOTE]
> **--defer** es necesario porque una regla debe tener una condición de coincidencia para agregarse en el paso siguiente.

```azurecli
az network front-door waf-policy rule create \
  --name IPAllowListRule \
  --priority 1 \
  --rule-type MatchRule \
  --action Block \
  --resource-group <resource-group-name> \
  --policy-name IPAllowPolicyExampleCLI --defer
```
A continuación, agregue la condición de coincidencia a la regla:

```azurecli
az network front-door waf-policy rule match-condition add \
--match-variable RemoteAddr \
--operator IPMatch \
--values "ip-address-range-1" "ip-address-range-2" \
--negate true \
--name IPAllowListRule \
  --resource-group <resource-group-name> \
  --policy-name IPAllowPolicyExampleCLI 
  ```
                                                   
### <a name="find-the-id-of-a-waf-policy"></a>Búsqueda del identificador de una directiva de WAF 
Para buscar el identificador de una directiva WAF, utilice el comando [az network front-door waf-policy show](/cli/azure/ext/front-door/network/front-door/waf-policy?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-show). Reemplace *IPAllowPolicyExampleCLI* en el ejemplo siguiente por la directiva que ha creado anteriormente.

   ```azurecli
   az network front-door  waf-policy show \
     --resource-group <resource-group-name> \
     --name IPAllowPolicyExampleCLI
   ```

### <a name="link-a-waf-policy-to-an-azure-front-door-front-end-host"></a>Vinculación de una directiva de WAF a un host de front-end de Azure Front Door

Establezca el identificador *WebApplicationFirewallPolicyLink* de Azure Front Door en el identificador de la directiva mediante el uso del comando [az network front-door update](/cli/azure/ext/front-door/network/front-door?view=azure-cli-latest#ext-front-door-az-network-front-door-update). Reemplace *IPAllowPolicyExampleCLI* por la directiva única que creó anteriormente.

   ```azurecli
   az network front-door update \
     --set FrontendEndpoints[0].WebApplicationFirewallPolicyLink.id=/subscriptions/<subscription ID>/resourcegroups/resource-group-name/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/IPAllowPolicyExampleCLI \
     --name <frontdoor-name>
     --resource-group <resource-group-name>
   ```
En este ejemplo, la directiva de WAF se aplica a **FrontendEndpoints [0]** . Puede vincular la directiva de WAF a cualquiera de los front-ends.
> [!Note]
> Solo es necesario establecer la propiedad **WebApplicationFirewallPolicyLink** una vez para vincular una directiva de WAF a un front-end de Azure Front Door. Las posteriores actualizaciones de la directiva se aplican al front-end.

## <a name="configure-a-waf-policy-with-azure-powershell"></a>Configuración de una directiva de WAF con Azure PowerShell

### <a name="prerequisites"></a>Prerrequisitos
Antes de empezar a configurar una directiva de restricción de IP, configure el entorno de PowerShell y cree un perfil de Azure Front Door.

#### <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell
Azure PowerShell proporciona un conjunto de cmdlets que usan el modelo de [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para administrar los recursos de Azure.

Puede instalar [Azure PowerShell](https://docs.microsoft.com/powershell/azure/) en el equipo local y usarlo en cualquier sesión de PowerShell. Siga las instrucciones de la página para iniciar sesión en PowerShell con las credenciales de Azure y, después, instale el módulo Az.

1. Utilice el siguiente comando para conectarse a Azure y, después, use un cuadro de diálogo interactivo para iniciar sesión.
    ```
    Connect-AzAccount
    ```
 2. Antes de instalar un módulo de Azure Front Door asegúrese de que tiene instalada la versión actual del módulo PowerShellGet. Ejecute el siguiente comando y vuelva a abrir PowerShell.

    ```
    Install-Module PowerShellGet -Force -AllowClobber
    ``` 

3. Instale el módulo Az.FrontDoor con el siguiente comando. 
    
    ```
    Install-Module -Name Az.FrontDoor
    ```
### <a name="create-an-azure-front-door-profile"></a>Creación de un perfil de Azure Front Door
Para crear un perfil de Azure Front Door, siga las instrucciones que se describen en [Inicio rápido: Creación de una instancia de Front Door para una aplicación web global de alta disponibilidad](../../frontdoor/quickstart-create-front-door.md).

### <a name="define-an-ip-match-condition"></a>Definición de una condición de coincidencia IP
Use el comando [New AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) para definir una condición de coincidencia de IP.
En el siguiente ejemplo, reemplace *ip-address-range-1*, *ip-address-range-2* por su propio rango.    
```powershell
$IPMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable  RemoteAddr `
-OperatorProperty IPMatch `
-MatchValue "ip-address-range-1", "ip-address-range-2"
-NegateCondition 1
```
     
### <a name="create-a-custom-ip-allow-rule"></a>Creación de una regla de permiso de IP personalizada

Use el comando [New-AzFrontDoorWafCustomRuleObject](/powershell/module/Az.FrontDoor/New-azfrontdoorwafcustomruleobject) para definir una acción y establecer una prioridad. En el ejemplo siguiente, se bloquearán las solicitudes que no procedan de IP de clientes que coincidan con la lista.

```azurepowershell
$IPAllowRule = New-AzFrontDoorWafCustomRuleObject `
-Name "IPAllowRule" `
-RuleType MatchRule `
-MatchCondition $IPMatchCondition `
-Action Block -Priority 1
```

### <a name="configure-a-waf-policy"></a>Configuración de una directiva de WAF
Busque el nombre del grupo de recursos que contiene el perfil de Azure Front Door. Para ello, debe usar `Get-AzResourceGroup`. A continuación, configure una directiva WAF con la regla IP, para lo que debe usar [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```azurepowershell
  $IPAllowPolicyExamplePS = New-AzFrontDoorWafPolicy `
    -Name "IPRestrictionExamplePS" `
    -resourceGroupName <resource-group-name> `
    -Customrule $IPAllowRule`
    -Mode Prevention `
    -EnabledState Enabled
   ```

### <a name="link-a-waf-policy-to-an-azure-front-door-front-end-host"></a>Vinculación de una directiva de WAF a un host de front-end de Azure Front Door

Vincule un objeto de directiva de WAF a un host de front-end existente y actualice las propiedades de Azure Front Door. En primer lugar, recupere el objeto de Azure Front Door. Para hacerlo, use [Get-AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor). A continuación, establezca la propiedad **WebApplicationFirewallPolicyLink** en el identificador de recurso de *$IPAllowPolicyExamplePS*, que se creó en el paso anterior, mediante el uso del comando [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor).

```azurepowershell
  $FrontDoorObjectExample = Get-AzFrontDoor `
    -ResourceGroupName <resource-group-name> `
    -Name $frontDoorName
  $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $IPBlockPolicy.Id
  Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
```

> [!NOTE]
> En este ejemplo, la directiva de WAF se aplica a **FrontendEndpoints [0]** . Una directiva de WAF se puede vincular a cualquiera de los front-ends. Solo es necesario establecer la propiedad **WebApplicationFirewallPolicyLink** una vez para vincular una directiva de WAF a un front-end de Azure Front Door. Las posteriores actualizaciones de la directiva se aplican al front-end.


## <a name="configure-a-waf-policy-with-a-resource-manager-template"></a>Configuración de una directiva de WAF con una plantilla de Resource Manager
Para ver la plantilla que crea una directiva de Azure Front Door y una directiva de WAF con reglas de restricción de IP personalizadas, vaya a [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-clientip).


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un perfil de Azure Front Door](../../frontdoor/quickstart-create-front-door.md).
