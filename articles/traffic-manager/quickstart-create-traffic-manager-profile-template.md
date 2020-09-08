---
title: 'Inicio rápido: Creación de una instancia de Traffic Manager mediante una plantilla de Azure Resource Manager'
description: En este inicio rápido se describe cómo crear un perfil de Azure Traffic Manager mediante una plantilla de Azure Resource Manager.
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: duau
ms.date: 09/01/2020
ms.openlocfilehash: dbdb6a255fdf0214103a0011f25b0a6d25014e69
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299157"
---
# <a name="quickstart-create-a-traffic-manager-profile-using-an-arm-template"></a>Inicio rápido: Creación de un perfil de Traffic Manager mediante una plantilla de Resource Manager

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para crear un perfil de Traffic Manager con puntos de conexión externos mediante el método de enrutamiento del rendimiento.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-traffic-manager-external-endpoint%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-traffic-manager-external-endpoint).

:::code language="json" source="~/quickstart-templates/101-traffic-manager-external-endpoint/azuredeploy.json":::

En la plantilla, se define un recurso de Azure:

* [**Microsoft.Network/trafficManagerProfiles**](/azure/templates/microsoft.network/trafficmanagerprofiles)

Para encontrar más plantillas relacionadas con Azure Traffic Manager, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione **Try It** (Probarlo) en el bloque de código siguiente para abrir Azure Cloud Shell y siga las instrucciones para iniciar sesión en Azure. 

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-traffic-manager-external-endpoint/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Espere hasta que vea el aviso de la consola.

1. Seleccione **Copiar** en el bloque de código anterior para copiar el script de PowerShell.

1. Haga clic con el botón derecho en el panel de consola del shell y, a continuación, seleccione **Pegar**.

1. Escriba los valores.

    La implementación de la plantilla crea un perfil con dos puntos de conexión externos. **Endpoint1** usa un punto de conexión de destino de *w<span>ww.microsoft</span>.com* con la ubicación en **Norte de Europa**. **Endpoint2** usa un punto de conexión de destino de *d<span>ocs.microsoft</span>.com* con la ubicación en **Centro-sur de EE. UU.** . 

    El nombre del grupo de recursos es el nombre del proyecto con **rg** anexado.

    > [!NOTE]
    > **uniqueDNSname** debe ser un nombre globalmente único para que la plantilla se implemente correctamente. Si se produce un error en la implementación, vuelva al paso 1.

    La plantilla tarda unos minutos en implementarse. Al finalizar, la salida es parecida a esta:

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-arm-powershell-output.png" alt-text="Salida de la implementación de PowerShell de la plantilla de Resource Manager de Azure Traffic Manager":::

Azure PowerShell se usa para implementar la plantilla. Además de Azure PowerShell, también puede usar Azure Portal, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Validación de la implementación

1. Determine el nombre DNS del perfil de Traffic Manager mediante [Get-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/get-aztrafficmanagerprofile).

    ```azurepowershell-interactive
    Get-AzTrafficManagerProfile -Name ExternalEndpointExample -ResourceGroupName $resourceGroupName | Select RelativeDnsName
    ```

    Copie el valor **RelativeDnsName**. El nombre DNS del perfil de Traffic Manager es *<* relativednsname *>.trafficmanager.net*. 

1. En una instancia local de PowerShell, ejecute el siguiente comando, pero reemplace la variable **{relativeDNSname}** por *<* relativednsname *>.trafficmanager.net*.

    ```powershell
    Resolve-DnsName -Name {relativeDNSname} | Select-Object NameHost | Select -First 1
    ```
    El valor de NameHost que debería obtener es *w<span>ww.microsoft</span>.com* o *d<span>ocs.microsoft</span>.com* en función de la región que esté más próxima.

1. Para comprobar si puede resolverlo en el otro punto de conexión, deshabilite el punto de conexión para el destino que recibió en el último paso. Reemplace **{endpointName}** por **endpoint1** o **endpoint2** para deshabilitar el destino de *w<span>ww.microsoft</span>.com* o *d<span>ocs.microsoft</span>.com*, respectivamente.

    ```azurepowershell-interactive
    Disable-AzTrafficManagerEndpoint -Name {endpointName} -Type ExternalEndpoints -ProfileName ExternalEndpointExample -ResourceGroupName $resourceGroupName -Force
    ```
1. Vuelva a ejecutar el comando del paso 2 en una instancia de PowerShell local. Esta vez debe obtener el otro valor de NameHost para el otro punto de conexión. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite el perfil de Traffic Manager, elimine el grupo de recursos. Esta acción quita el perfil de Traffic Manager y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:
* Perfil de Traffic Manager

Para más información sobre el enrutamiento del tráfico, continúe con los tutoriales de Traffic Manager.

> [!div class="nextstepaction"]
> [Tutoriales de Traffic Manager](tutorial-traffic-manager-improve-website-response.md)
