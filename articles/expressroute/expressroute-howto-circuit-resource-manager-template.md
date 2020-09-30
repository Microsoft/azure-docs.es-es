---
title: 'Plantilla de Azure ExpressRoute: Creación de un circuito ExpressRoute'
description: Aprenda a crear un circuito de Azure ExpressRoute mediante la implementación de una plantilla de Azure Resource Manager con Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/13/2019
ms.author: duau
ms.openlocfilehash: 3428f092c4b2356e9ea6afa54d1a03fc8b4f9f05
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89566184"
---
# <a name="create-an-expressroute-circuit-by-using-azure-resource-manager-template"></a>Creación de un circuito de ExpressRoute mediante una plantilla de Azure Resource Manager

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [CLI de Azure](howto-circuit-cli.md)
> * [Plantilla de Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-circuit-classic.md)
>

Aprenda a crear un circuito de ExpressRoute mediante la implementación de una plantilla de Azure Resource Manager con Azure PowerShell. Para más información sobre el desarrollo de plantillas de Resource Manager, consulte la [documentación de Resource Manager](/azure/azure-resource-manager/) y la [referencia de plantilla](/azure/templates/microsoft.network/expressroutecircuits).

## <a name="before-you-begin"></a>Antes de empezar

* Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.
* Asegúrese de que tiene permisos para crear recursos de red. Si no tiene los permisos adecuados, póngase en contacto con el administrador de cuenta.
* Puede [ver un vídeo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit) antes de comenzar para entender mejor los pasos.

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Crear y aprovisionar un circuito ExpressRoute

[Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) tiene una buena colección de plantillas de Resource Manager. Use alguna de las [plantillas existentes](https://azure.microsoft.com/resources/templates/101-expressroute-circuit-create/) para crear un circuito de ExpressRoute.

[!code-json[create-azure-expressroute-circuit](~/quickstart-templates/101-expressroute-circuit-create/azuredeploy.json)]

Para ver más plantillas relacionadas, seleccione [aquí](https://azure.microsoft.com/resources/templates/?term=expressroute).

Para crear un circuito de ExpressRoute mediante la implementación de una plantilla:

1. Seleccione **Probar** en el bloque de código siguiente y luego siga las instrucciones para iniciar sesión en Azure Cloud Shell.

    ```azurepowershell-interactive
    $circuitName = Read-Host -Prompt "Enter a circuit name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${circuitName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-circuit-create/azuredeploy.json"

    $serviceProviderName = "Equinix"
    $peeringLocation = "Silicon Valley"
    $bandwidthInMbps = 500
    $sku_tier = "Premium"
    $sku_family = "MeteredData"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -circuitName $circuitName -serviceProviderName $serviceProviderName -peeringLocation $peeringLocation -bandwidthInMbps $bandwidthInMbps -sku_tier $sku_tier -sku_family $sku_family

    Write-Host "Press [ENTER] to continue ..."
    ```

   * El **nivel de SKU** determina si un circuito ExpressRoute es [Local](expressroute-faqs.md#expressroute-local), Estándar o [Premium](expressroute-faqs.md#expressroute-premium). Puede especificar *Local*, *Estándar o *Premium*. No se puede cambiar la SKU de *Estándar o Premium* a *Local*.
   * La **familia de SKU** determina el tipo de facturación. Puede seleccionar *Metereddata* para el plan de datos limitado y *Unlimiteddata* para el plan de datos ilimitado. Puede cambiar el tipo de facturación de *Metereddata* a *Unlimiteddata*, pero no se puede cambiar el tipo de *Unlimiteddata* a *Metereddata*. Un circuito *Local* solo es *Unlimiteddata*.
   * La **ubicación de emparejamiento** es la ubicación física de emparejamiento con Microsoft.

     > [!IMPORTANT]
     > La ubicación de emparejamiento indica la [ubicación física](expressroute-locations.md) de emparejamiento con Microsoft. **No** está vinculada a la propiedad Location, que hace referencia a la ubicación geográfica donde se encuentra el proveedor de recursos de red de Azure. Aunque no están relacionadas, se recomienda elegir un proveedor de recursos de red geográficamente cerca de la ubicación de emparejamiento del circuito.

    El nombre del grupo de recursos es el nombre del espacio de nombres de Service Bus con **rg** anexado.

2. Seleccione **Copiar** para copiar el script de PowerShell.
3. Haga clic con el botón derecho en la consola del shell y seleccione **Pegar**.

Lleva algunos minutos crear un centro de eventos.

En este tutorial se usa Azure PowerShell para implementar la plantilla. Para otros métodos de implementación de plantillas, consulte:

* [Mediante Azure Portal](../azure-resource-manager/templates/deploy-portal.md).
* [Mediante la CLI de Azure](../azure-resource-manager/templates/deploy-cli.md).
* [Mediante la API REST](../azure-resource-manager/templates/deploy-rest.md).

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a><a name="delete"></a>Desaprovisionar y eliminar un circuito ExpressRoute

Puede eliminar el circuito ExpressRoute seleccionando el icono **Eliminar** . Tenga en cuenta la información siguiente:

* Tiene que desvincular todas las redes virtuales del circuito ExpressRoute. Si se produce un error en esta operación, compruebe si hay alguna red virtual vinculada al circuito.
* Si el estado de aprovisionamiento del proveedor de servicios del circuito ExpressRoute es **Aprovisionando** o **Aprovisionado**, debe colaborar con su proveedor de servicios para que desaprovisionen el circuito. Se le siguen reservando recursos y facturándole por ello hasta que el proveedor de servicios complete el desaprovisionamiento del circuito y nos lo notifique.
* Si el proveedor de servicios ha desaprovisionado el circuito (el estado de aprovisionamiento del proveedor de servicios está establecido en **No aprovisionado**), puede eliminar el circuito. Esto detiene la facturación del circuito.

Puede eliminar el circuito de ExpressRoute si ejecuta el siguiente comando de PowerShell:

```azurepowershell-interactive
$circuitName = Read-Host -Prompt "Enter the same circuit name that you used earlier"
$resourceGroupName = "${circuitName}rg"

Remove-AzExpressRouteCircuit -ResourceGroupName $resourceGroupName -Name $circuitName
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear el circuito, lleve a cabo los pasos siguientes:

* [Crear y modificar el enrutamiento para el circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Vincular la red virtual a su circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
