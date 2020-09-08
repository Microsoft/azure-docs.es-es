---
title: Administración de conjuntos de escalado de máquinas virtuales con Azure PowerShell
description: Cmdlets de Azure PowerShell comunes para administrar conjuntos de escalado de máquinas virtuales, por ejemplo, cómo iniciar y detener una instancia, o cambiar la capacidad del conjunto de escalado.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/29/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 45bace9ac174b353bb4662a27e800c0de4eada4b
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89072730"
---
# <a name="manage-a-virtual-machine-scale-set-with-azure-powershell"></a>Administrar un conjunto de escalado de máquinas virtuales con Azure PowerShell

Durante el ciclo de vida de la máquina virtual, es posible que deba ejecutar una o varias tareas de administración. Además, puede crear scripts para automatizar varias tareas de ciclo de vida. En este artículo se detallan algunos de los cmdlets comunes de Azure PowerShell que le permiten realizar estas tareas.

Si necesita crear un conjunto de escalado de máquinas virtuales, puede [crear un conjunto de escalado con Azure PowerShell](quick-create-powershell.md).

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="view-information-about-a-scale-set"></a>Visualización de información acerca de un conjunto de escalado
Para ver información general acerca de un conjunto de escalado, use [Get-AzVmss](/powershell/module/az.compute/get-azvmss). En el ejemplo siguiente se obtiene información sobre el conjunto de escalado denominado *myScaleSet* del grupo de recursos *myResourceGroup*. Escriba sus propios nombres, como se indica a continuación:

```powershell
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```


## <a name="view-vms-in-a-scale-set"></a>Visualización de máquinas virtuales en un conjunto de escalado
Para ver una lista de las instancias de VM en un conjunto de escalado, use [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). En el ejemplo siguiente, se enumeran todas las instancias de máquina virtual en el conjunto de escalado denominado *myScaleSet* y en el grupo de recursos *myResourceGroup*. Proporcione sus propios valores para estos nombres:

```powershell
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

Para ver información adicional acerca de una instancia específica de la VM, agregue el parámetro `-InstanceId` a [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) y especifique una instancia para verla. En el ejemplo siguiente se puede consultar información acerca de la instancia de VM *0* en el conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Escriba sus propios nombres, como se indica a continuación:

```powershell
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="change-the-capacity-of-a-scale-set"></a>Cambio de la capacidad de un conjunto de escalado
Los comandos anteriores muestran información acerca del conjunto de escalado y las instancias de VM. Para aumentar o disminuir el número de instancias en el conjunto de escalado, puede cambiar la capacidad. El conjunto de escalado crea o elimina automáticamente el número necesario de máquinas virtuales, y configura las máquinas virtuales para recibir tráfico de la aplicación.

En primer lugar, cree un objeto de conjunto de escalado con [Get-AzVmss](/powershell/module/az.compute/get-azvmss) y, después, especifique un nuevo valor para `sku.capacity`. Para aplicar el cambio de capacidad, use [Update-AzVmss](/powershell/module/az.compute/update-azvmss). En el ejemplo siguiente se actualiza *myScaleSet* en el grupo de recursos *myResourceGroup* a una capacidad de *5* instancias. Proporcione sus propios valores, como se indica a continuación:

```powershell
# Get current scale set
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 5
Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss
```

La capacidad del conjunto de escalado tarda unos minutos en actualizarse. Si reduce la capacidad de un conjunto de escalado, se quitan primero las máquinas virtuales con los identificadores de instancia más altos.


## <a name="stop-and-start-vms-in-a-scale-set"></a>Detención e inicio de las máquinas virtuales de un conjunto de escalado
Para detener una o varias VM en un conjunto de escalado, use [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se detengan. Si no especifica un identificador de instancia, se detienen todas las máquinas virtuales del conjunto de escalado. Para detener varias máquinas virtuales, separe cada identificador de instancia con una coma.

En el ejemplo siguiente se detiene la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```powershell
Stop-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```

De forma predeterminada, se cancela la asignación de las máquinas virtuales para no acumular cargos de proceso. Si quiere que la máquina virtual permanezca en un estado de aprovisionamiento cuando se detenga, agregue el parámetro `-StayProvisioned` al comando anterior. Las máquinas virtuales detenidas que se mantienen aprovisionadas acumulan cargos regulares de proceso.


### <a name="start-vms-in-a-scale-set"></a>Inicio de máquinas virtuales en un conjunto de escalado
Para iniciar una o varias VM en un conjunto de escalado, use [Start-AzVmss](/powershell/module/az.compute/start-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se inicien. Si no especifica un identificador de instancia, se inician todas las máquinas virtuales del conjunto de escalado. Para iniciar varias máquinas virtuales, separe cada identificador de instancia con una coma.

En el ejemplo siguiente se inicia la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```powershell
Start-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="restart-vms-in-a-scale-set"></a>Reinicio de máquinas virtuales en un conjunto de escalado
Para reiniciar una o varias VM en un conjunto de escalado, use [Restart-AzVmss](/powershell/module/az.compute/restart-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se reinicien. Si no especifica un identificador de instancia, se reinician todas las máquinas virtuales del conjunto de escalado. Para reiniciar varias máquinas virtuales, separe cada identificador de instancia con una coma.

En el ejemplo siguiente se reinicia la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```powershell
Restart-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="remove-vms-from-a-scale-set"></a>Eliminación de las máquinas virtuales de un conjunto de escalado
Para quitar una o varias VM en un conjunto de escalado, use [Remove-AzVmss](/powershell/module/az.compute/remove-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se eliminen. Si no especifica un identificador de instancia, se eliminan todas las máquinas virtuales del conjunto de escalado. Para eliminar varias máquinas virtuales, separe cada identificador de instancia con una coma.

En el ejemplo siguiente se quita la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```powershell
Remove-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="next-steps"></a>Pasos siguientes
Entre otras tareas comunes para conjuntos de escalado se incluye cómo [implementar una aplicación](virtual-machine-scale-sets-deploy-app.md) y cómo [actualizar instancias de máquinas virtuales](virtual-machine-scale-sets-upgrade-scale-set.md). También puede usar Azure PowerShell para [configurar reglas de escalado automático](virtual-machine-scale-sets-autoscale-overview.md).
