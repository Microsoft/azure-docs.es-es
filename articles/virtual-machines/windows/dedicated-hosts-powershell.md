---
title: Implementar hosts dedicados de Azure mediante el Azure PowerShell
description: Implemente máquinas virtuales en hosts dedicados mediante Azure PowerShell.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure
ms.date: 08/01/2019
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: 599d13daac2e062c8f71f5f7d7133646a1447123
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87266595"
---
# <a name="deploy-vms-to-dedicated-hosts-using-the-azure-powershell"></a>Implementación de máquinas virtuales en hosts dedicados mediante Azure PowerShell

En este artículo se ofrecen instrucciones para crear un [host dedicado](dedicated-hosts.md) de Azure en el que se pueden hospedar máquinas virtuales (VM). 

Asegúrese de que tiene instalada la versión 2.8.0 de Azure PowerShell, o cualquier versión posterior, y que inicia sesión en una cuenta de Azure con `Connect-AzAccount`. 

## <a name="limitations"></a>Limitaciones

- Actualmente, los conjuntos de escalado de máquinas virtuales no se admiten en los hosts dedicados.
- Los tamaños y tipos de hardware disponibles para hosts dedicados varían según la región. Para más información, consulte la [página de precios](https://aka.ms/ADHPricing) de hosts.

## <a name="create-a-host-group"></a>Creación de un grupo host

Un **grupo host** es un recurso que representa una colección de hosts dedicados. Puede crear un grupo host en una región y una zona de disponibilidad, y agregarle hosts. Al planear la alta disponibilidad, hay otras opciones. Puede usar una o ambas de las dos opciones siguientes con los hosts dedicados: 
- Abarcar varias zonas de disponibilidad. En este caso, es necesario tener un grupo host en cada una de las zonas que quiera usar.
- Abarcar varios dominios de error que se asignan a bastidores físicos. 
 
En cualquier caso, es necesario proporcionar el número de dominios de error del grupo host. Si no quiere abarcar dominios de error en el grupo, use un número de dominios de error de 1. 

También puede usar zonas de disponibilidad y dominios de error a la vez. En este ejemplo se crea en la zona 1 un grupo host con dos dominios de error. 


```azurepowershell-interactive
$rgName = "myDHResourceGroup"
$location = "EastUS"

New-AzResourceGroup -Location $location -Name $rgName
$hostGroup = New-AzHostGroup `
   -Location $location `
   -Name myHostGroup `
   -PlatformFaultDomain 2 `
   -ResourceGroupName $rgName `
   -Zone 1
```

## <a name="create-a-host"></a>Creación de un host

Ahora vamos a crear un host dedicado en el grupo host. Además de un nombre para el host, se le pedirá que proporcione el SKU del host. El SKU del host registra la serie de máquinas virtuales admitidas, así como la generación de hardware del host dedicado.

Para más información sobre los precios y los SKU de host, consulte [Precios de hosts dedicados de Azure](https://aka.ms/ADHPricing).

Si establece un número de dominios de error para el grupo host, se le pedirá que especifique el dominio de error para su host. En este ejemplo, se establece el dominio de error para el host en 1.


```azurepowershell-interactive
$dHost = New-AzHost `
   -HostGroupName $hostGroup.Name `
   -Location $location -Name myHost `
   -ResourceGroupName $rgName `
   -Sku DSv3-Type1 `
   -AutoReplaceOnFailure 1 `
   -PlatformFaultDomain 1
```

## <a name="create-a-vm"></a>Crear una VM

Cree una máquina virtual en el host dedicado. 

Si especificó una zona de disponibilidad al crear el grupo host, debe usar la misma zona al crear la máquina virtual. En este ejemplo, como el grupo host está en la zona 1, es preciso crear la máquina virtual en la zona 1.  


```azurepowershell-interactive
$cred = Get-Credential
New-AzVM `
   -Credential $cred `
   -ResourceGroupName $rgName `
   -Location $location `
   -Name myVM `
   -HostId $dhost.Id `
   -Image Win2016Datacenter `
   -Zone 1 `
   -Size Standard_D4s_v3
```

> [!WARNING]
> Si crea una máquina virtual en un host que no tenga suficientes recursos, la máquina virtual se creará en un estado de error. 

## <a name="check-the-status-of-the-host"></a>Comprobación del estado del host

Mediante el uso de [GetAzHost](/powershell/module/az.compute/get-azhost) con el parámetro `-InstanceView`puede comprobar el estado de mantenimiento del host y el número de máquinas virtuales que aún se pueden implementar.

```azurepowershell-interactive
Get-AzHost `
   -ResourceGroupName $rgName `
   -Name myHost `
   -HostGroupName $hostGroup.Name `
   -InstanceView
```

El resultado será similar al siguiente:

```
ResourceGroupName      : myDHResourceGroup
PlatformFaultDomain    : 1
AutoReplaceOnFailure   : True
HostId                 : 12345678-1234-1234-abcd-abc123456789
ProvisioningTime       : 7/28/2019 5:31:01 PM
ProvisioningState      : Succeeded
InstanceView           : 
  AssetId              : abc45678-abcd-1234-abcd-123456789abc
  AvailableCapacity    : 
    AllocatableVMs[0]  : 
      VmSize           : Standard_D2s_v3
      Count            : 32
    AllocatableVMs[1]  : 
      VmSize           : Standard_D4s_v3
      Count            : 16
    AllocatableVMs[2]  : 
      VmSize           : Standard_D8s_v3
      Count            : 8
    AllocatableVMs[3]  : 
      VmSize           : Standard_D16s_v3
      Count            : 4
    AllocatableVMs[4]  : 
      VmSize           : Standard_D32-8s_v3
      Count            : 2
    AllocatableVMs[5]  : 
      VmSize           : Standard_D32-16s_v3
      Count            : 2
    AllocatableVMs[6]  : 
      VmSize           : Standard_D32s_v3
      Count            : 2
    AllocatableVMs[7]  : 
      VmSize           : Standard_D64-16s_v3
      Count            : 1
    AllocatableVMs[8]  : 
      VmSize           : Standard_D64-32s_v3
      Count            : 1
    AllocatableVMs[9]  : 
      VmSize           : Standard_D64s_v3
      Count            : 1
  Statuses[0]          : 
    Code               : ProvisioningState/succeeded
    Level              : Info
    DisplayStatus      : Provisioning succeeded
    Time               : 7/28/2019 5:31:01 PM
  Statuses[1]          : 
    Code               : HealthState/available
    Level              : Info
    DisplayStatus      : Host available
Sku                    : 
  Name                 : DSv3-Type1
Id                     : /subscriptions/10101010-1010-1010-1010-101010101010/re
sourceGroups/myDHResourceGroup/providers/Microsoft.Compute/hostGroups/myHostGroup/hosts
/myHost
Name                   : myHost
Location               : eastus
Tags                   : {}
```

## <a name="add-an-existing-vm"></a>Incorporación de una máquina virtual existente 

Puede agregar una máquina virtual existente a un host dedicado, pero antes es necesario detenerla o desasignarla. Antes de mover una máquina virtual a un host dedicado, asegúrese de que se admite su configuración:

- El tamaño de la máquina virtual debe estar en la misma familia de tamaños que el host dedicado. Por ejemplo, si el host dedicado es DSv3, el tamaño de la máquina virtual puede ser Standard_D4s_v3, pero no Standard_A4_v2. 
- Es preciso que la máquina virtual se encuentre en la misma región que el host dedicado.
- La máquina virtual no puede formar parte de ningún grupo de ubicación por proximidad. Quite la máquina virtual del grupo de ubicación por proximidad antes de moverla a un host dedicado. Para más información, consulte [Traslado de una VM existente fuera de un grupo de selección de ubicación de proximidad](./proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group)
- La máquina virtual no puede estar en un conjunto de disponibilidad.
- Si la máquina virtual está en una zona de disponibilidad, debe estar en la misma que el grupo host. Los valores de zona de disponibilidad de la máquina virtual y del grupo host deben coincidir.

Reemplace el valor de las variables por su propia información.

```azurepowershell-interactive
$vmRGName = "movetohost"
$vmName = "myVMtoHost"
$dhRGName = "myDHResourceGroup"
$dhGroupName = "myHostGroup"
$dhName = "myHost"

$myDH = Get-AzHost `
   -HostGroupName $dhGroupName `
   -ResourceGroupName $dhRGName `
   -Name $dhName
   
$myVM = Get-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
   
$myVM.Host = New-Object Microsoft.Azure.Management.Compute.Models.SubResource

$myVM.Host.Id = "$myDH.Id"

Stop-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName -Force
   
Update-AzVM `
   -ResourceGroupName $vmRGName `
   -VM $myVM -Debug
   
Start-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
```


## <a name="clean-up"></a>Limpieza

Aunque no se implementen máquinas virtuales, se le cobrará por los hosts dedicados. Elimine los hosts que no use actualmente para ahorrar costos.  

Solo se puede eliminar un host cuando no haya ninguna máquina virtual que lo use. Elimine las máquinas virtuales mediante el comando[Remove-AzVM](/powershell/module/az.compute/remove-azvm).

```azurepowershell-interactive
Remove-AzVM -ResourceGroupName $rgName -Name myVM
```

Después de eliminar las máquinas virtuales, puede eliminar el host mediante [Remove-AzHost](/powershell/module/az.compute/remove-azhost).

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

Una vez que haya eliminado todos los hosts, puede eliminar el grupo host mediante[Remove-AzHostGroup](/powershell/module/az.compute/remove-azhostgroup). 

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

También puede eliminar todo el grupo de recursos con un solo comando mediante [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Se eliminarán todos los recursos creados en el grupo, lo que incluye las máquinas virtuales, los hosts y los grupos host.
 
```azurepowershell-interactive
Remove-AzResourceGroup -Name $rgName
```


## <a name="next-steps"></a>Pasos siguientes

- [Aquí](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md) encontrará una plantilla de ejemplo en la que se usan zonas y dominios de error para obtener la máxima resistencia en una región.

- También se pueden implementar hosts dedicados desde [Azure Portal](dedicated-hosts-portal.md).
