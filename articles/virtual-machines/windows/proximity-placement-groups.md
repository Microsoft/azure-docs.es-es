---
title: 'PowerShell: Uso de grupos de selección de ubicación de proximidad'
description: Obtenga información sobre la creación y el uso de grupos de selección de ubicación de proximidad con Azure PowerShell.
services: virtual-machines
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 01/27/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: 9ea986b338d977102d78e9c12bcbe5b2f2c510e7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083453"
---
# <a name="deploy-vms-to-proximity-placement-groups-using-powershell"></a>Implementación de máquinas virtuales en grupos de selección de ubicación de proximidad con PowerShell


Para acercar las máquinas virtuales lo máximo posible con la menor latencia, debe implementarlas dentro de un [grupo de selección de ubicación de proximidad](co-location.md#proximity-placement-groups).

Un grupo de selección de ubicación de proximidad es una agrupación lógica que se usa para asegurarse de que los recursos de proceso de Azure se encuentran físicamente cercanos entre sí. Los grupos de selección de ubicación de proximidad son útiles para las cargas de trabajo en las que la latencia baja es un requisito.


## <a name="create-a-proximity-placement-group"></a>Creación de un grupo de selección de ubicación por proximidad
Cree un grupo de selección de ubicación de proximidad con el cmdlet [New-AzProximityPlacementGroup](/powershell/module/az.compute/new-azproximityplacementgroup). 

```azurepowershell-interactive
$resourceGroup = "myPPGResourceGroup"
$location = "East US"
$ppgName = "myPPG"
New-AzResourceGroup -Name $resourceGroup -Location $location
$ppg = New-AzProximityPlacementGroup `
   -Location $location `
   -Name $ppgName `
   -ResourceGroupName $resourceGroup `
   -ProximityPlacementGroupType Standard
```

## <a name="list-proximity-placement-groups"></a>Enumeración de los grupos de selección de ubicación de proximidad

Puede enumerar todos los grupos de selección de ubicación de proximidad mediante el cmdlet [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
Get-AzProximityPlacementGroup
```


## <a name="create-a-vm"></a>Crear una VM

Cree una máquina virtual en el grupo de selección de ubicación de proximidad con `-ProximityPlacementGroup $ppg.Id` para hacer referencia al identificador de grupo de selección de ubicación de proximidad cuando use [New-AzVM](/powershell/module/az.compute/new-azvm) para crear la máquina virtual.

```azurepowershell-interactive
$vmName = "myVM"

New-AzVm `
  -ResourceGroupName $resourceGroup `
  -Name $vmName `
  -Location $location `
  -OpenPorts 3389 `
  -ProximityPlacementGroup $ppg.Id
```

Puede ver la máquina virtual en el grupo de selección de ubicación mediante [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
Get-AzProximityPlacementGroup -ResourceId $ppg.Id |
    Format-Table -Property VirtualMachines -Wrap
```

### <a name="move-an-existing-vm-into-a-proximity-placement-group"></a>Traslado de una VM existente a un grupo de selección de ubicación de proximidad

Además, puede agregar una VM existente a un grupo de selección de ubicación de proximidad. Primero debe detener o desasignar la VM y, a continuación, actualizarla y reiniciarla.

```azurepowershell-interactive
$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPGResourceGroup -Name myPPG
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Stop-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
Update-AzVM -VM $vm -ResourceGroupName $vm.ResourceGroupName -ProximityPlacementGroupId $ppg.Id
Start-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
```

### <a name="move-an-existing-vm-out-of-a-proximity-placement-group"></a>Traslado de una VM existente fuera de un grupo de selección de ubicación de proximidad

Para quitar una VM de un grupo de selección de ubicación de proximidad, primero debe detener o desasignar la VM y, a continuación, actualizarla y reiniciarla.

```azurepowershell-interactive
$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPGResourceGroup -Name myPPG
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Stop-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
$vm.ProximityPlacementGroup = ""
Update-AzVM -VM $vm -ResourceGroupName $vm.ResourceGroupName 
Start-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
```


## <a name="availability-sets"></a>Conjuntos de disponibilidad
También puede crear un conjunto de disponibilidad en el grupo de selección de ubicación de proximidad. Use el mismo parámetro `-ProximityPlacementGroup` con el cmdlet [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset) para crear un conjunto de disponibilidad, y todas las máquinas virtuales del conjunto de disponibilidad también se crearán en el mismo grupo de selección de ubicación de proximidad.

Para agregar o quitar un conjunto de disponibilidad existente en un grupo de selección de ubicación de proximidad, primero debe detener todas las VM del conjunto de disponibilidad. 

### <a name="move-an-existing-availability-set-into-a-proximity-placement-group"></a>Traslado de un conjunto de disponibilidad existente a un grupo de selección de ubicación de proximidad

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$avSetName = "myAvailabilitySet"
$avSet = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $avSetName
$vmIds = $avSet.VirtualMachinesReferences
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
    } 

$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPG -Name myPPG
Update-AzAvailabilitySet -AvailabilitySet $avSet -ProximityPlacementGroupId $ppg.Id
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName 
    } 
```

### <a name="move-an-existing-availability-set-out-of-a-proximity-placement-group"></a>Traslado de un conjunto de disponibilidad existente fuera de un grupo de selección de ubicación de proximidad

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$avSetName = "myAvailabilitySet"
$avSet = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $avSetName
$vmIds = $avSet.VirtualMachinesReferences
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
    } 

$avSet.ProximityPlacementGroup = ""
Update-AzAvailabilitySet -AvailabilitySet $avSet 
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName 
    } 
```

## <a name="scale-sets"></a>Conjuntos de escalado

También puede crear un conjunto de escalado en el grupo de selección de ubicación de proximidad. Use el mismo parámetro `-ProximityPlacementGroup` con [New-AzVmss](/powershell/module/az.compute/new-azvmss) para crear un conjunto de escalado, y todas las instancias se crearán en el mismo grupo de selección de ubicación de proximidad.


Para agregar o quitar un conjunto de escalado existente en un grupo de selección de ubicación de proximidad, primero debe detener el conjunto de escalado. 

### <a name="move-an-existing-scale-set-into-a-proximity-placement-group"></a>Traslado de un conjunto de escalado existente a un grupo de selección de ubicación de proximidad

```azurepowershell-interactive
$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPG -Name myPPG
$vmss = Get-AzVmss -ResourceGroupName myVMSSResourceGroup -VMScaleSetName myScaleSet
Stop-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
Update-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName -ProximityPlacementGroupId $ppg.Id
Start-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
```

### <a name="move-an-existing-scale-set-out-of-a-proximity-placement-group"></a>Traslado de un conjunto de escalado existente fuera de un grupo de selección de ubicación de proximidad

```azurepowershell-interactive
$vmss = Get-AzVmss -ResourceGroupName myVMSSResourceGroup -VMScaleSetName myScaleSet
Stop-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
$vmss.ProximityPlacementGroup = ""
Update-AzVmss -VirtualMachineScaleSet $vmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName  
Start-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

También puede usar la [CLI de Azure](../linux/proximity-placement-groups.md) para crear grupos de selección de ubicación de proximidad.
