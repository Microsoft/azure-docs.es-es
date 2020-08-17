---
title: Modificación de un conjunto de escalado de máquinas virtuales de Azure
description: Aprenda a modificar y actualizar un conjunto de escalado de máquinas virtuales de Azure con las API REST, Azure PowerShell y la CLI de Azure
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 03/10/2020
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: f7a61ed039a3d8ed643e3b1b3d79384e35847986
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029304"
---
# <a name="modify-a-virtual-machine-scale-set"></a>Modificación de un conjunto de escalado de máquinas virtuales

A lo largo del ciclo de vida de las aplicaciones, puede que tenga la necesidad de modificar o actualizar el conjunto de escalado de máquinas virtuales. Estas actualizaciones pueden incluir cómo actualizar la configuración del conjunto de escalado o cambiar la configuración de la aplicación. En este artículo se describe cómo modificar un conjunto de escalado existente con las API REST, Azure PowerShell o la CLI de Azure.

## <a name="fundamental-concepts"></a>Conceptos básicos

### <a name="the-scale-set-model"></a>El modelo del conjunto de escalado
Un conjunto de escalado tiene un "modelo de conjunto de escalado" que captura el estado *deseado* del conjunto de escalado como un todo. Para consultar el modelo de un conjunto de escalado, puede usar: 

- API REST con [compute/virtualmachinescalesets/get](/rest/api/compute/virtualmachinescalesets/get) como se indica a continuación:

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Azure PowerShell con [Get-AzVmss](/powershell/module/az.compute/get-azvmss):

    ```powershell
    Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
    ```

- CLI de Azure con [az vmss show](/cli/azure/vmss):

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet
    ```

- También puede usar [resources.azure.com](https://resources.azure.com) o los [SDK de Azure](https://azure.microsoft.com/downloads/) específicos del lenguaje.

La presentación exacta de la salida depende de las opciones proporcionadas al comando. En el ejemplo siguiente se muestra una salida de ejemplo condensada de la CLI de Azure:

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "overprovision": true,
  "plan": null,
  "singlePlacementGroup": true,
  "sku": {
    "additionalProperties": {},
    "capacity": 1,
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

Estas propiedades se aplican al conjunto de escalado como un todo.


### <a name="the-scale-set-instance-view"></a>La vista de la instancia del conjunto de escalado
Un conjunto de escalado tiene también una "vista de instancia del conjunto de escalado" que captura el estado de *tiempo de ejecución* actual del conjunto de escalado como un todo. Para consultar la vista de instancia de un conjunto de escalado, puede usar:

- API REST con [compute/virtualmachinescalesets/getinstanceview](/rest/api/compute/virtualmachinescalesets/getinstanceview) de la manera siguiente:

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/instanceView?api-version={apiVersion}
    ```

- Azure PowerShell con [Get-AzVmss](/powershell/module/az.compute/get-azvmss):

    ```powershell
    Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceView
    ```

- CLI de Azure con [az vmss get-instance-view](/cli/azure/vmss):

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
    ```

- También puede usar [resources.azure.com](https://resources.azure.com) o los [SDK de Azure](https://azure.microsoft.com/downloads/) específicos del lenguaje.

La presentación exacta de la salida depende de las opciones proporcionadas al comando. En el ejemplo siguiente se muestra una salida de ejemplo condensada de la CLI de Azure:

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
{
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": null,
      "time": "{time}"
    }
  ],
  "virtualMachine": {
    "additionalProperties": {},
    "statusesSummary": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "count": 1
      }
    ]
  }
}
```

Estas propiedades proporcionan un resumen del estado de tiempo de ejecución actual de las máquinas virtuales del conjunto de escalado, incluido el estado de las extensiones que aplicadas al conjunto de escalado.


### <a name="the-scale-set-vm-model-view"></a>La vista del modelo de máquina virtual del conjunto de escalado
Al igual que un conjunto de escalado tiene una vista de modelo, cada instancia de máquina virtual de este tiene su propia vista de modelo. Para consultar la vista de modelo de una instancia de máquina virtual de un conjunto de escalado puede usar:

- API REST con [compute/virtualmachinescalesetvms/get](/rest/api/compute/virtualmachinescalesetvms/get) de la manera siguiente:

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId?api-version={apiVersion}
    ```

- Azure PowerShell con [Get-AzVmssVm](/powershell/module/az.compute/get-azvmssvm):

    ```powershell
    Get-AzVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- CLI de Azure con [az vmss show](/cli/azure/vmss):

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- También puede usar [resources.azure.com](https://resources.azure.com) o los [SDK de Azure](https://azure.microsoft.com/downloads/).

La presentación exacta de la salida depende de las opciones proporcionadas al comando. En el ejemplo siguiente se muestra una salida de ejemplo condensada de la CLI de Azure:

```azurecli
$ az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "name": "{name}",
  "sku": {
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

Estas propiedades describen la configuración de una instancia de máquina virtual en un conjunto de escalado, no la del conjunto de escalado como un todo. Por ejemplo, el modelo de conjunto de escalado tiene la propiedad `overprovision`, mientras que el modelo de una instancia de máquina virtual de un conjunto de escalado no. Esta diferencia se debe a que el exceso de aprovisionamiento es una propiedad del conjunto de escalado como un todo, y no de las instancias de máquinas virtuales individuales del conjunto de escalado. Para más información sobre el aprovisionamiento en exceso, consulte [Consideraciones de diseño para conjuntos de escalado](virtual-machine-scale-sets-design-overview.md#overprovisioning).


### <a name="the-scale-set-vm-instance-view"></a>La vista de la instancia de máquina virtual del conjunto de escalado
Al igual que un conjunto de escalado tiene una vista de instancia, cada instancia de máquina virtual de este tiene su propia vista de instancia. Para consultar la vista de instancia de una instancia de máquina virtual concreta de un conjunto de escalado puede usar:

- API REST con [compute/virtualmachinescalesetvms/getinstanceview](/rest/api/compute/virtualmachinescalesetvms/getinstanceview) de la manera siguiente:

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId/instanceView?api-version={apiVersion}
    ```

- Azure PowerShell con [Get-AzVmssVm](/powershell/module/az.compute/get-azvmssvm):

    ```powershell
    Get-AzVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -InstanceView
    ```

- CLI de Azure con [az vmss get-instance-view](/cli/azure/vmss)

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- También puede usar [resources.azure.com](https://resources.azure.com) o los [SDK de Azure](https://azure.microsoft.com/downloads/).

La presentación exacta de la salida depende de las opciones proporcionadas al comando. En el ejemplo siguiente se muestra una salida de ejemplo condensada de la CLI de Azure:

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
{
  "additionalProperties": {
    "osName": "ubuntu",
    "osVersion": "16.04"
  },
  "disks": [
    {
      "name": "{name}",
      "statuses": [
        {
          "additionalProperties": {},
          "code": "ProvisioningState/succeeded",
          "displayStatus": "Provisioning succeeded",
          "time": "{time}"
        }
      ]
    }
  ],
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "time": "{time}"
    },
    {
      "additionalProperties": {},
      "code": "PowerState/running",
      "displayStatus": "VM running"
    }
  ],
  "vmAgent": {
    "statuses": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Ready",
        "level": "Info",
        "message": "Guest Agent is running",
        "time": "{time}"
      }
    ],
    "vmAgentVersion": "{version}"
  },
}
```

Estas propiedades describen el estado actual del entorno de tiempo de ejecución de una instancia de máquina virtual en un conjunto de escalado, que incluye cualquier extensión que se aplica al conjunto de escalado.


## <a name="how-to-update-global-scale-set-properties"></a>Actualización de las propiedades globales del conjunto de escalado
Para actualizar una propiedad global del conjunto de escalado, debe actualizar la propiedad en el modelo de conjunto de escalado. Puede usar para ello:

- API REST con [compute/virtualmachinescalesets/createorupdate](/rest/api/compute/virtualmachinescalesets/createorupdate) de la manera siguiente:

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Puede implementar una plantilla de Resource Manager con las propiedades de la API REST para actualizar las propiedades globales del conjunto de escalado.

- Azure PowerShell con [Update-AzVmss](/powershell/module/az.compute/update-azvmss):

    ```powershell
    Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -VirtualMachineScaleSet {scaleSetConfigPowershellObject}
    ```

- CLI de Azure con [az vmss update](/cli/azure/vmss):
    - Para modificar una propiedad:

        ```azurecli
        az vmss update --set {propertyPath}={value}
        ```

    - Para añadir un objeto a una propiedad de la lista de un conjunto de escalado: 

        ```azurecli
        az vmss update --add {propertyPath} {JSONObjectToAdd}
        ```

    - Para quitar un objeto de una propiedad de la lista de un conjunto de escalado: 

        ```azurecli
        az vmss update --remove {propertyPath} {indexToRemove}
        ```

    - Si ha implementado previamente el conjunto de escalado con el `az vmss create`comando, puede ejecutarlo de nuevo `az vmss create`para actualizar el conjunto de escalado. Asegúrese de que todas las propiedades del comando `az vmss create` son las mismas que antes, excepto las propiedades que desea modificar.

- También puede usar [resources.azure.com](https://resources.azure.com) o los [SDK de Azure](https://azure.microsoft.com/downloads/).

Una vez que se actualiza el modelo de conjunto de escalado, la nueva configuración se aplica a las nuevas máquinas virtuales creadas en el conjunto de escalado. Sin embargo, los modelos de las máquinas virtuales existentes del conjunto de escalado se deben seguir actualizando con el modelo de conjunto de escalado general más reciente. En el modelo de cada máquina virtual hay una propiedad booleana llamada `latestModelApplied` que indica si la máquina virtual está o no actualizada con el modelo de conjunto de escalado general más reciente (`true` significa que sí lo está).


## <a name="how-to-bring-vms-up-to-date-with-the-latest-scale-set-model"></a>Actualización de las máquinas virtuales con el modelo de conjunto de escalado más reciente
Los conjuntos de escalado tienen una "directiva de actualización" que determina cómo se actualizan las máquinas virtuales con el modelo de conjunto de escalado más reciente. Los tres modos de la directiva de actualización son:

- **Automático**: en este modo, el conjunto de escalado no ofrece ninguna garantía sobre el orden en el que se desactivan las máquinas virtuales. El conjunto de escalado puede desactivarlas todas al mismo tiempo. 
- **Gradual**: en este modo, el conjunto de escalado implementa la actualización por lotes con un tiempo de pausa opcional entre ellos.
- **Manual**: en este modo, cuando se actualiza el modelo de conjunto de escalado, no ocurre nada en las máquinas virtuales existentes.
 
Para actualizar las máquinas virtuales existentes, debe realizar una "actualización manual" de cada una de ellas. Puede realizar esta actualización manual mediante:

- API REST con [compute/virtualmachinescalesets/updateinstances](/rest/api/compute/virtualmachinescalesets/updateinstances) de la manera siguiente:

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/manualupgrade?api-version={apiVersion}
    ```

- Azure PowerShell con [Update-AzVmssInstance](/powershell/module/az.compute/update-azvmssinstance):
    
    ```powershell
    Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- CLI de Azure con [az vmss update-instances](/cli/azure/vmss)

    ```azurecli
    az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
    ```

- También puede usar los [SDK de Azure](https://azure.microsoft.com/downloads/) específicos del lenguaje.

>[!NOTE]
> Los clústeres de Service Fabric solo pueden usar el modo *automático*, pero la actualización se administra de forma diferente. Para más información, consulte [Actualización de la aplicación de Service Fabric](../service-fabric/service-fabric-application-upgrade.md).

Hay un tipo de modificación en las propiedades globales del conjunto de escalado que no sigue la directiva de actualización. Los cambios en el perfil del disco de datos y del sistema operativo del conjunto de escalado (como nombre de usuario administrador y contraseña) solo se pueden realizar en la versión de API *2017-12-01* o posterior. Estos cambios solo se aplican a las máquinas virtuales creadas después del cambio en el modelo de conjunto de escalado. Para actualizar las máquinas virtuales existentes, debe realizar un "restablecimiento de la imagen inicial" de cada una de ellas. Para ello, puede usar:

- API REST con [compute/virtualmachinescalesets/reimage](/rest/api/compute/virtualmachinescalesets/reimage) de la manera siguiente:

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/reimage?api-version={apiVersion}
    ```

- Azure PowerShell con [Set-AzVmssVm](/powershell/module/az.compute/set-azvmssvm):

    ```powershell
    Set-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -Reimage
    ```

- CLI de Azure con [az vmss reimage](/cli/azure/vmss):

    ```azurecli
    az vmss reimage --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- También puede usar los [SDK de Azure](https://azure.microsoft.com/downloads/) específicos del lenguaje.


## <a name="properties-with-restrictions-on-modification"></a>Propiedades con restricciones en la modificación

### <a name="create-time-properties"></a>Propiedades de tiempo de creación
Algunas propiedades solo pueden establecerse al crear el conjunto de escalado. Estas propiedades incluyen:

- Zonas de disponibilidad
- Image reference publisher
- Image reference offer
- Tipo de cuenta de almacenamiento de disco de sistema operativo administrado

### <a name="properties-that-can-only-be-changed-based-on-the-current-value"></a>Propiedades que solo se pueden cambiar en función del valor actual
Algunas propiedades pueden cambiarse, con excepciones dependiendo del valor actual. Estas propiedades incluyen:

- **singlePlacementGroup**: si singlePlacementGroup es true, se puede cambiar a false. Sin embargo, si singlePlacementGroup es false, **no se puede** cambiar a true.
- **subnet**: la subred de un conjunto de escalado se puede modificar siempre y cuando la subred original y la nueva subred estén en la misma red virtual.

### <a name="properties-that-require-deallocation-to-change"></a>Propiedades que requieren desasignación para poder cambiar
Algunas propiedades solo pueden cambiar a ciertos valores si se desasignan las máquinas virtuales del conjunto de escalado. Estas propiedades incluyen:

- **Nombre de SKU**: si el nombre de la nueva SKU de máquina virtual no se admite en el hardware en el que se encuentra actualmente el conjunto de escalado, debe desasignar las máquinas virtuales del conjunto de escalado antes de modificar el nombre de SKU. Para más información, consulte [cómo cambiar el tamaño de una máquina virtual de Azure](../virtual-machines/windows/resize-vm.md).


## <a name="vm-specific-updates"></a>Actualizaciones específicas de la máquina virtual
Algunas modificaciones pueden aplicarse a máquinas virtuales específicas, y no a las propiedades globales del conjunto de escalado. Actualmente, la única actualización específica de la máquina virtual que se admite es asociar/desasociar discos de datos en las máquinas virtuales del conjunto de escalado. Esta característica se encuentra en su versión preliminar. Para más información, consulte la [documentación de versión preliminar](https://github.com/Azure/vm-scale-sets/tree/master/preview/disk).


## <a name="scenarios"></a>Escenarios

### <a name="application-updates"></a>Actualizaciones de aplicaciones
Si una aplicación se implementa en un conjunto de escalado mediante extensiones, al actualizar la configuración de extensiones la aplicación se actualiza en consonancia con la directiva de actualización. Por ejemplo, si tiene una nueva versión de un script que se va a ejecutar en una extensión de script personalizado, podría actualizar la propiedad *fileUris* para que apunte al nuevo script. En algunos casos, puede que desee forzar una actualización aunque no se haya modificado la configuración de extensiones (por ejemplo, ha actualizado el script sin cambiar su URI). En estos casos, puede modificar la propiedad *forceUpdateTag* para forzar una actualización. La plataforma Azure no interpreta esta propiedad. Si cambia el valor, no hay ningún efecto sobre cómo se ejecuta la extensión. Solo fuerza a que la extensión se vuelva a ejecutar. Para más información sobre *forceUpdateTag*, consulte la [documentación de API REST de las extensiones](/rest/api/compute/virtualmachineextensions/createorupdate). Tenga en cuenta que *forceUpdateTag* se puede utilizar con todas las extensiones, no solo con la extensión de script personalizado.

También es habitual que las aplicaciones se implementen mediante una imagen personalizada. Este escenario se explica en la siguiente sección.

### <a name="os-updates"></a>Actualizaciones de SO
Si usa imágenes de plataforma de Azure, puede actualizar la imagen mediante la modificación de la propiedad *imageReference* (para más información, consulte la [documentación de API REST](/rest/api/compute/virtualmachinescalesets/createorupdate)).

>[!NOTE]
> Con imágenes de plataforma, es habitual especificar la versión más reciente de la referencia de imagen. Al realizar procedimientos de creación, escalado horizontal y restablecimiento de imagen inicial, las máquinas virtuales se crean con la última versión disponible. Sin embargo, **no** significa que la imagen del sistema operativo se actualice automáticamente con el tiempo a medida que se lanzan nuevas versiones de imágenes. Existe actualmente una característica distinta en versión preliminar que proporciona actualizaciones automáticas del sistema operativo. Para más información, consulte la [documentación de actualizaciones automáticas del sistema operativo](virtual-machine-scale-sets-automatic-upgrade.md).

Si usa imágenes personalizadas, puede actualizar la imagen mediante la actualización del identificador de *imageReference* (para más información, consulte la [documentación de API REST](/rest/api/compute/virtualmachinescalesets/createorupdate)).

## <a name="examples"></a>Ejemplos

### <a name="update-the-os-image-for-your-scale-set"></a>Actualización de la imagen del sistema operativo en el conjunto de escalado
Puede que tenga un conjunto de escalado que ejecuta una versión anterior de Ubuntu LTS 16.04 y quiere actualizar a una versión más reciente de Ubuntu LTS 16.04, como *16.04.201801090*. La propiedad de versión de referencia de imagen no es parte de una lista, por lo que puede modificar directamente estas propiedades con uno de los siguientes comandos:

- Azure PowerShell con [Update-AzVmss](/powershell/module/az.compute/update-azvmss), como sigue:

    ```powershell
    Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -ImageReferenceVersion 16.04.201801090
    ```

- CLI de Azure con [az vmss update](/cli/azure/vmss):

    ```azurecli
    az vmss update --resource-group myResourceGroup --name myScaleSet --set virtualMachineProfile.storageProfile.imageReference.version=16.04.201801090
    ```

Como alternativa, puede cambiar la imagen que se usa el conjunto de escalado. Por ejemplo, es posible que desee actualizar o cambiar una imagen personalizada usada por el conjunto de escalado. Puede cambiar la imagen que usa el conjunto de escalado mediante la actualización de la propiedad de identificador de la referencia de imagen. La propiedad de identificador de referencia de imagen no es parte de una lista, por lo que puede modificarla directamente con uno de los siguientes comandos:

- Azure PowerShell con [Update-AzVmss](/powershell/module/az.compute/update-azvmss), como sigue:

    ```powershell
    Update-AzVmss `
        -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -ImageReferenceId /subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myNewImage
    ```

- CLI de Azure con [az vmss update](/cli/azure/vmss):

    ```azurecli
    az vmss update \
        --resource-group myResourceGroup \
        --name myScaleSet \
        --set virtualMachineProfile.storageProfile.imageReference.id=/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myNewImage
    ```


### <a name="update-the-load-balancer-for-your-scale-set"></a>Actualización del equilibrador de carga en el conjunto de escalado
Supongamos que tiene un conjunto de escalado con una instancia de Azure Load Balancer y quiere reemplazar Azure Load Balancer por una instancia de Azure Application Gateway. Las propiedades de Load Balancer y Application Gateway de un conjunto de escalado son parte de una lista, así que puede usar los comandos para quitar o agregar elementos de la lista en lugar de modificar las propiedades directamente:

- Azure PowerShell:

    ```powershell
    # Get the current model of the scale set and store it in a local PowerShell object named $vmss
    $vmss=Get-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
    
    # Create a local PowerShell object for the new desired IP configuration, which includes the reference to the application gateway
    $ipconf = New-AzVmssIPConfig "myNic" -ApplicationGatewayBackendAddressPoolsId /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendAddressPoolName} -SubnetId $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Subnet.Id –Name $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Name
    
    # Replace the existing IP configuration in the local PowerShell object (which contains the references to the current Azure Load Balancer) with the new IP configuration
    $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0] = $ipconf
    
    # Update the model of the scale set with the new configuration in the local PowerShell object
    Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -virtualMachineScaleSet $vmss
    ```

- CLI de Azure:

    ```azurecli
    # Remove the load balancer backend pool from the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools 0
    
    # Remove the load balancer backend pool from the scale set model; only necessary if you have NAT pools configured on the scale set
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools 0
    
    # Add the application gateway backend pool to the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].ApplicationGatewayBackendAddressPools '{"id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendPoolName}"}'
    ```

>[!NOTE]
> Estos comandos dan por hecho que solo hay una configuración IP y un equilibrador de carga en el conjunto de escalado. Si hay varios, debe usar un índice de lista distinto de *0*.


## <a name="next-steps"></a>Pasos siguientes
También puede realizar tareas de administración comunes en conjuntos de escalado con la [CLI de Azure](virtual-machine-scale-sets-manage-cli.md) o [Azure PowerShell](virtual-machine-scale-sets-manage-powershell.md).
