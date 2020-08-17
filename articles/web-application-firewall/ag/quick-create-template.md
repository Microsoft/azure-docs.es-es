---
title: 'Inicio rápido: Creación de un firewall de aplicaciones web de Azure, versión 2, en Application Gateway: plantilla de Azure Resource Manager'
titleSuffix: Azure Application Gateway
description: Aprenda a usar una plantilla de Azure Resource Manager para crear un firewall de aplicaciones web, versión 2, en Azure Application Gateway.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: quickstart
ms.date: 04/02/2020
ms.author: victorh
ms.openlocfilehash: 081bab0cd930d90ca0d359461e4a41b15ba4911b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075501"
---
# <a name="quickstart-create-an-azure-waf-v2-on-application-gateway-using-an-arm-template"></a>Inicio rápido: Creación de un firewall de aplicaciones web de Azure, versión 2, en Application Gateway mediante una plantilla de Resource Manager

En este inicio rápido, se usa una plantilla de Azure Resource Manager para crear un firewall de aplicaciones web de Azure, versión 2, en Application Gateway.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-wafv2%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea un sencillo firewall de aplicaciones web v2 en Azure Application Gateway Esto incluye una dirección IP de front-end de IP pública, la configuración de HTTP, una regla con un cliente de escucha básico en el puerto 80 y un grupo de back-end. Se crea una directiva de WAF con una regla personalizada para bloquear el tráfico hacia el grupo de back-end basado en un tipo de coincidencia de la dirección IP.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/ag-docs-wafv2/).

:::code language="json" source="~/quickstart-templates/ag-docs-wafv2/azuredeploy.json" range="001-404" highlight="314-358":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/applicationgateways**](/azure/templates/microsoft.network/applicationgateways)
- [**Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies**](/azure/templates/microsoft.network/ApplicationGatewayWebApplicationFirewallPolicies)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses): uno para la puerta de enlace de aplicaciones y dos para las máquina virtuales.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): dos máquinas virtuales
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces): dos para las máquinas virtuales
- [**Microsoft.Compute/virtualMachine/extensions**](/azure/templates/microsoft.compute/virtualmachines/extensions): para configurar las páginas web e IIS

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una puerta de enlace de aplicaciones, la infraestructura de red y dos máquinas virtuales en el grupo de back-end que ejecuta IIS.

   [![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-wafv2%2Fazuredeploy.json)

2. Elija su grupo de recursos o cree uno.
3. Seleccione **Acepto los términos y condiciones indicadas anteriormente** y, después, seleccione **Comprar**. La implementación puede tardar 10 minutos o más en completarse.

## <a name="validate-the-deployment"></a>Validación de la implementación

Aunque no es necesario instalar IIS para crear la puerta de enlace de aplicaciones, se instaló en los servidores de back-end para comprobar si Azure creó correctamente un WAF v2 en la puerta de enlace de aplicaciones.

Use IIS para probar la puerta de enlace de aplicaciones:

1. Busque la dirección IP pública de la puerta de enlace de aplicaciones en la página de **información general**.![ Registre la dirección IP pública de la puerta de enlace de aplicaciones](../../application-gateway/media/application-gateway-create-gateway-portal/application-gateway-record-ag-address.png). O bien, puede seleccionar **Todos los recursos**, escribir *myAGPublicIPAddress* en el cuadro de búsqueda y luego seleccionarlo en los resultados de la búsqueda. Azure muestra la dirección IP pública en la página de **información general**.
2. Copie la dirección IP pública y, luego, péguela en la barra de direcciones del explorador para ir a esa dirección IP.
3. Compruebe la respuesta. Una respuesta **403 Prohibido** comprueba que el WAF se creó correctamente y bloquea las conexiones al grupo de back-end.
4. Cambie la regla personalizada a **Permitir el tráfico**.
  Ejecute el siguiente script de Azure PowerShell, reemplazando el nombre del grupo de recursos:
   ```azurepowershell
   $rg = "<your resource group name>"
   $AppGW = Get-AzApplicationGateway -Name myAppGateway -ResourceGroupName $rg
   $pol = Get-AzApplicationGatewayFirewallPolicy -Name WafPol01 -ResourceGroupName $rg
   $pol[0].customrules[0].action = "allow"
   $rule = $pol.CustomRules
   Set-AzApplicationGatewayFirewallPolicy -Name WafPol01 -ResourceGroupName $rg -CustomRule $rule
   $AppGW.FirewallPolicy = $pol
   Set-AzApplicationGateway -ApplicationGateway $AppGW
   ```

   Actualice el explorador varias veces y debería ver las conexiones a myVM1 y a myVM2.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con la puerta de enlace de aplicaciones, elimine el grupo de recursos. De esta forma, se elimina la puerta de enlace de aplicaciones y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Creación de una puerta de enlace de aplicaciones con un firewall de aplicaciones web mediante Azure Portal](application-gateway-web-application-firewall-portal.md)