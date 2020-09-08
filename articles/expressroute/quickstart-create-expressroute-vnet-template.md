---
title: Creación de un circuito de ExpressRoute mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un circuito de ExpressRoute mediante una plantilla de Azure Resource Manager.
services: expressroute
author: duongau
mnager: kumud
ms.service: expressroute
ms.topic: quickstart
ms.custom: subject-armsq
ms.date: 08/31/2020
ms.author: duau
ms.openlocfilehash: 2bc4c2a3ee7f1a1dbd85d46e49ec27e5276de159
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237152"
---
# <a name="quickstart-create-an-expressroute-circuit-with-private-peering-using-an-arm-template"></a>Inicio rápido: Creación de un circuito ExpressRoute con emparejamiento privado mediante una plantilla de Resource Manager

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para crear un circuito de ExpressRoute con emparejamiento privado.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-expressroute-private-peering-vnet%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-expressroute-private-peering-vnet).

En este inicio rápido va a crear un circuito de ExpressRoute con *Equinix* como proveedor de servicios. El circuito usará una *SKU Premium* con un ancho de banda de *50 Mbps*y la ubicación de emparejamiento de *Washington DC*. El emparejamiento privado se habilitará con una subred principal y secundaria de *192.168.10.16/30* y *192.168.10.20/30*respectivamente. También se creará una red virtual,junto con una puerta de enlace de *ExpressRoute de alto rendimiento*.

:::code language="json" source="~/quickstart-templates/101-expressroute-private-peering-vnet/azuredeploy.json" range="001-351" highlight="183-219":::

En la plantilla se han definido varios recursos de Azure:

* [**Microsoft.Network/expressRouteCircuits**](/azure/templates/microsoft.network/expressRouteCircuits)
* [**Microsoft.Network/expressRouteCircuits/Peers**](/azure/templates/microsoft.network/expressRouteCircuits/peerings) (se usa para habilitar el emparejamiento privado en el circuito)
* [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networkSecurityGroups) (el grupo de seguridad de red se aplica a las subredes de la red virtual)
* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks) 
* [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicIPAddresses) (la dirección IP pública que usa la puerta de enlace de ExpressRoute)
* [**Microsoft.Network/virtualNetworkGateways**](/azure/templates/microsoft.network/virtualNetworkGateways) (puerta de enlace de ExpressRoute se usa para vincular la red virtual al circuito)

Para encontrar más plantillas relacionadas con ExpressRoute, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione **Try It** (Probarlo) en el bloque de código siguiente para abrir Azure Cloud Shell y siga las instrucciones para iniciar sesión en Azure. 

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-private-peering-vnet/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Espere hasta que vea el aviso de la consola.

1. Seleccione **Copiar** en el bloque de código anterior para copiar el script de PowerShell.

1. Haga clic con el botón derecho en el panel de consola del shell y, a continuación, seleccione **Pegar**.

1. Escriba los valores.

    El nombre del grupo de recursos es el nombre del proyecto con **rg** anexado.

La plantilla tarda aproximadamente 20 en implementarse. Al finalizar, la salida es parecida a esta:

:::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-powershell-output.png" alt-text="Salida de la implementación de PowerShell de la plantilla de Resource Manager de ExpressRoute":::

Azure PowerShell se usa para implementar la plantilla. Además de Azure PowerShell, también puede usar Azure Portal, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Validación de la implementación

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Grupos de recursos** en el panel izquierdo.

1. Seleccione el grupo de recursos que creó en la sección anterior. El nombre del grupo de recursos predeterminado es el nombre del proyecto con **rg** anexado.

1. El grupo de recursos debe contener los recursos que se ven aquí:

     :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-resource-group.png" alt-text="Grupo de recursos de implementación de ExpressRoute":::

1. Seleccione el circuito ExpressRoute **er-ck01** para comprobar que su estado es **Habilitado**, que el estado del proveedor es **No aprovisionado** y que el emparejamiento privado tiene el estado **Aprovisionado**.

    :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-circuit.png" alt-text="Circuito de implementación de ExpressRoute":::

> [!NOTE]
> Deberá llamar al proveedor para completar el proceso de aprovisionamiento antes de poder vincular la red virtual al circuito.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el circuito de ExpressRoute, elimine el grupo de recursos. Esta acción quita el circuito de ExpressRoute y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:
* Circuito ExpressRoute
* Virtual Network
* VPN Gateway
* Dirección IP pública
* grupos de seguridad de red

Para más información sobre la vinculación de una red virtual a un circuito, vaya a los tutoriales de ExpressRoute.

> [!div class="nextstepaction"]
> [Tutoriales de ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)

* Para más información sobre los flujos de trabajo de ExpressRoute, vea [Flujos de trabajo de ExpressRoute](expressroute-workflows.md).
* Para más información sobre el emparejamiento de circuitos, vea [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md).
* Para más información sobre redes virtuales, vea [Información general sobre redes virtuales](../virtual-network/virtual-networks-overview.md).