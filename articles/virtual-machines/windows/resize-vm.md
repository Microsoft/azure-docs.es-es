---
title: Cambio de tamaño de una máquina virtual Windows en Azure
description: Cambie el tamaño que se usa para una máquina virtual de Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: sizes
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/13/2020
ms.author: cynthn
ms.openlocfilehash: 4860422fefb5a95fe41912b4898f02867f7b3aeb
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832253"
---
# <a name="resize-a-windows-vm"></a>Cambio de tamaño de una máquina virtual Windows

En este artículo se muestra cómo mover una máquina virtual a otro [tamaño de máquina virtual](../sizes.md).

Después de crear una máquina virtual, puede escalarla o reducirla verticalmente cambiando su tamaño. En algunos casos, hay que desasignarla antes. Esto puede suceder si el nuevo tamaño no está disponible en el clúster de hardware que hospeda la actualmente la máquina virtual.

Si la máquina virtual usa Premium Storage, asegúrese de elegir una versión **s** del tamaño para obtener compatibilidad con este nivel de almacenamiento. Por ejemplo, elija Standard_E4**s**_v3 en lugar de Standard_E4_v3.

## <a name="use-the-portal"></a>Uso del portal

1. Abra [Azure Portal](https://portal.azure.com).
1. Abra la página de la máquina virtual.
1. En el menú de la izquierda, seleccione **Tamaño**.
1. Elija un tamaño nuevo en la lista de tamaños disponibles y, después, seleccione **Cambiar tamaño**.


Si la máquina virtual está en ejecución, el cambio de tamaño hará que se reinicie. Detener la máquina virtual puede revelar tamaños adicionales.

## <a name="use-powershell-to-resize-a-vm-not-in-an-availability-set"></a>Uso de PowerShell para cambiar el tamaño de una máquina virtual que no está en un conjunto de disponibilidad

Establezca algunas variables. Reemplace los valores por su propia información.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

Muestre los tamaños de máquina virtual que están disponibles en el clúster de hardware donde se hospeda la máquina virtual. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

Si se muestra el tamaño deseado, ejecute los comandos siguientes para cambiar el tamaño de la máquina virtual. Si el tamaño deseado no aparece, vaya al paso 3.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMsize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

Si el tamaño deseado no se muestra, ejecute los siguientes comandos para desasignar la máquina virtual, cambiar su tamaño y reiniciarla. Sustituya **\<newVMsize>** por el tamaño que desee.
   
```powershell
Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName
```

> [!WARNING]
> Al desasignar la máquina virtual, se liberan todas las direcciones IP dinámicas asignadas a ella. Esto no afecta a los discos del SO y de datos. 
> 
> 

## <a name="use-powershell-to-resize-a-vm-in-an-availability-set"></a>Uso de PowerShell para cambiar el tamaño de una máquina virtual en un conjunto de disponibilidad

Si el nuevo tamaño de una máquina virtual de un conjunto de disponibilidad no está disponible en el clúster de hardware que hospeda la máquina virtual, habrá que desasignar todas las máquinas virtuales del conjunto de disponibilidad para cambiar el tamaño de la máquina virtual. También tendrá que actualizar el tamaño de otras máquinas virtuales del conjunto de disponibilidad después de cambiar el tamaño de una máquina virtual. Para cambiar el tamaño de una máquina virtual de un conjunto de disponibilidad, siga estos pasos.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

Muestre los tamaños de máquina virtual que están disponibles en el clúster de hardware donde se hospeda la máquina virtual. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

Si se muestra el tamaño deseado, ejecute el comando siguiente para cambiar el tamaño de la máquina virtual. Si no aparece, vaya a la sección siguiente.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName 
$vm.HardwareProfile.VmSize = "<newVmSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
    
Si el tamaño deseado no aparece, continúe con los pasos siguientes para desasignar todas las máquinas virtuales del conjunto de disponibilidad, cambiar su tamaño y reiniciarlas.

Detenga todas las máquinas virtuales del conjunto de disponibilidad.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines |  Stop-AzVM -Force -NoWait  
```

Cambie el tamaño de todas las máquinas virtuales del conjunto de disponibilidad y reinícielas.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$newSize = "<newVmSize>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines | Foreach-Object { $_.HardwareProfile.VmSize = $newSize }
$virtualMachines | Update-AzVM
$virtualMachines | Start-AzVM
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener una mayor escalabilidad, ejecute varias instancias de VM y escálelas horizontalmente. Para obtener más información, consulte el artículo sobre [escalado automático de máquinas Linux en un conjunto de escalado de máquinas virtuales](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md).