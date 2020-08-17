---
title: Creación de una instantánea de una unidad de disco duro virtual en Azure
description: Aprenda a crear una copia de una máquina virtual de Azure como copia de seguridad o para solucionar problemas.
author: roygara
manager: twooley
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 10/08/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: e5ecb99c7f64d81d57c5d6d2cb25967913a752b4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074138"
---
# <a name="create-a-snapshot"></a>Crear una instantánea

Una instantánea es una copia completa de solo lectura de un disco duro virtual (VHD). Tome una instantánea de un disco duro virtual de datos o del sistema operativo para realizar una copia de seguridad o para solucionar problemas de la máquina virtual.

Si va a usar la instantánea para crear una máquina virtual, se recomienda un cierre limpio de la máquina virtual antes de tomar una instantánea para así limpiar cualquier proceso que esté en curso.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal 

Para crear una instantánea, complete los siguientes pasos: 
1.  En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso**.
2. Busque y seleccione **Snapshot**.
3. En la ventana **Instantánea**, seleccione **Crear**. Aparece la ventana **Crear instantánea**.
4. Escriba un **nombre** para la instantánea.
5. Seleccione un [Grupo de recursos](../../azure-resource-manager/management/overview.md#resource-groups) existente o escriba el nombre de uno nuevo. 
6. Seleccione una **ubicación** del centro de datos de Azure.  
7. Como **disco de origen**, seleccione el disco administrado para la instantánea.
8. Seleccione el **tipo de cuenta** que se usará para almacenar la instantánea. Use **Standard_HDD**, a menos que necesite que la instantánea se almacene en un disco de alto rendimiento.
9. Seleccione **Crear**.

## <a name="use-powershell"></a>Uso de PowerShell

En los pasos siguientes se muestra cómo copiar el disco duro virtual y crear la configuración de la instantánea. Después, puede tomar una instantánea del disco mediante el cmdlet [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot). 

 

1. Configure algunos parámetros: 

   ```azurepowershell-interactive
   $resourceGroupName = 'myResourceGroup' 
   $location = 'eastus' 
   $vmName = 'myVM'
   $snapshotName = 'mySnapshot'  
   ```

2. Obtenga la máquina virtual:

   ```azurepowershell-interactive
   $vm = Get-AzVM `
       -ResourceGroupName $resourceGroupName `
       -Name $vmName
   ```

3. Cree la configuración de la instantánea. En este ejemplo, es la instantánea del disco del sistema operativo:

   ```azurepowershell-interactive
   $snapshot =  New-AzSnapshotConfig `
       -SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
       -Location $location `
       -CreateOption copy
   ```
   
   > [!NOTE]
   > Si desea almacenar la instantánea en un almacenamiento resistente a zonas, debe crearla en una región que admita [zonas de disponibilidad ](../../availability-zones/az-overview.md) e incluir el parámetro `-SkuName Standard_ZRS`.   
   
4. Tome la instantánea:

   ```azurepowershell-interactive
   New-AzSnapshot `
       -Snapshot $snapshot `
       -SnapshotName $snapshotName `
       -ResourceGroupName $resourceGroupName 
   ```


## <a name="next-steps"></a>Pasos siguientes

Cree una máquina virtual a partir de una instantánea; para ello, cree primero un disco administrado con la instantánea y conéctelo como disco del sistema operativo. Para más información, consulte el ejemplo de [Creación de una máquina virtual a partir de una instantánea con PowerShell](./../scripts/virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md?toc=%2fpowershell%2fmodule%2ftoc.json).
