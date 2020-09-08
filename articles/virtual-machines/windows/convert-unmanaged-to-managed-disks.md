---
title: Conversión de máquina virtual Windows con discos no administrados en discos administrados
description: Conversión de una máquina virtual Windows con discos no administrados a discos administrados mediante PowerShell en el modelo de implementación de Resource Manager
author: roygara
ms.service: virtual-machines-windows
ms.subservice: disks
ms.topic: how-to
ms.date: 07/12/2018
ms.author: rogarana
ms.openlocfilehash: cf601c7f7429d4676ec480ddc016158150da8ce5
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815206"
---
# <a name="convert-a-windows-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>Conversión de máquina virtual Windows con discos no administrados en discos administrados

Si ya dispone de máquinas virtuales Windows que usan discos no administrados, puede convertirlas para usar discos administrados mediante el servicio [Azure Managed Disks](managed-disks-overview.md). Este proceso convierte el disco del SO y los discos de datos conectados.

 

## <a name="before-you-begin"></a>Antes de empezar


* Revise [Planeación de la migración a Managed Disks](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks).

* Revise las [preguntas frecuentes sobre la migración a Managed Disks](../faq-for-disks.md#migrate-to-managed-disks).

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]

* No se eliminan los discos duros virtuales originales ni la cuenta de almacenamiento usada por la máquina virtual antes de la conversión. Seguirán acumulando cargos. Para evitar que se le facture por estos artefactos, elimine los blobs de los discos duros virtuales originales después de comprobar que la conversión esté completa. Si tiene que buscar estos discos no conectados con el fin de eliminarlos, consulte nuestro artículo [Búsqueda y eliminación de discos administrados y no administrados de Azure no conectados](find-unattached-disks.md).


## <a name="convert-single-instance-vms"></a>Conversión de máquinas virtuales de instancia única
En esta sección se explica cómo convertir máquinas virtuales de Azure de instancia única de Unmanaged Disks a Managed Disks. (Si las máquinas virtuales se encuentran en un conjunto de disponibilidad, consulte la sección siguiente). 

1. Desasigne la VM con el cmdlet [Stop-AzVM](/powershell/module/az.compute/stop-azvm). En el ejemplo siguiente se desasigna la VM `myVM` en el grupo de recursos denominado `myResourceGroup`: 

   ```azurepowershell-interactive
   $rgName = "myResourceGroup"
   $vmName = "myVM"
   Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force
   ```

2. Convierta la VM a discos administrados con el cmdlet [ConvertTo-AzVMManagedDisk](/powershell/module/az.compute/convertto-azvmmanageddisk). El proceso siguiente convierte la VM anterior, incluidos el disco del SO y todos los discos de datos, e inicia la máquina virtual:

   ```azurepowershell-interactive
   ConvertTo-AzVMManagedDisk -ResourceGroupName $rgName -VMName $vmName
   ```



## <a name="convert-vms-in-an-availability-set"></a>Conversión de VM de un conjunto de disponibilidad

Si las VM que desea convertir en discos administrados se encuentran en un conjunto de disponibilidad, primero debe convertir el conjunto de disponibilidad en un conjunto de disponibilidad administrado.

1. Convierta el conjunto de disponibilidad con el cmdlet [Update-AzAvailabilitySet](/powershell/module/az.compute/update-azavailabilityset). En el ejemplo siguiente se actualiza el conjunto de disponibilidad denominado `myAvailabilitySet` en el grupo de recursos con nombre `myResourceGroup`:

   ```azurepowershell-interactive
   $rgName = 'myResourceGroup'
   $avSetName = 'myAvailabilitySet'

   $avSet = Get-AzAvailabilitySet -ResourceGroupName $rgName -Name $avSetName
   Update-AzAvailabilitySet -AvailabilitySet $avSet -Sku Aligned 
   ```

   Si la región en que se encuentra el conjunto de disponibilidad solo tiene 2 dominios de error administrados, pero el número de dominios de error no administrados es 3, este comando muestra un error similar a "El número de dominios de error especificado 3 debe estar en el intervalo de 1 a 2". Para resolver el error, actualice el dominio predeterminado a 2 y actualice `Sku` a `Aligned` como sigue:

   ```azurepowershell-interactive
   $avSet.PlatformFaultDomainCount = 2
   Update-AzAvailabilitySet -AvailabilitySet $avSet -Sku Aligned
   ```

2. Desasigne y convierta las máquinas virtuales del conjunto de disponibilidad. El siguiente script desasigna cada VM mediante el cmdlet [Stop-AzVM](/powershell/module/az.compute/stop-azvm), la convierte con [ConvertTo-AzVMManagedDisk](/powershell/module/az.compute/convertto-azvmmanageddisk) y la reinicia automáticamente como parte del proceso de conversión:

   ```azurepowershell-interactive
   $avSet = Get-AzAvailabilitySet -ResourceGroupName $rgName -Name $avSetName

   foreach($vmInfo in $avSet.VirtualMachinesReferences)
   {
     $vm = Get-AzVM -ResourceGroupName $rgName | Where-Object {$_.Id -eq $vmInfo.id}
     Stop-AzVM -ResourceGroupName $rgName -Name $vm.Name -Force
     ConvertTo-AzVMManagedDisk -ResourceGroupName $rgName -VMName $vm.Name
   }
   ```


## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error durante la conversión, o si una máquina virtual presenta un estado de error debido a errores en una conversión anterior, ejecute el cmdlet `ConvertTo-AzVMManagedDisk` de nuevo. Normalmente, un simple reintento desbloquea la situación.
Antes de realizar la conversión, asegúrese de que todas las extensiones de máquina virtual se encuentran en el estado de "Aprovisionamiento realizado correctamente". De lo contrario, se producirá un error de conversión con el código de error 409.

## <a name="convert-using-the-azure-portal"></a>Conversión mediante Azure Portal

Puede convertir discos no administrados en discos administrados mediante Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione la máquina virtual en la lista de máquinas virtuales en el portal.
3. En la hoja de la máquina virtual, seleccione **Discos** en el menú.
4. En la parte superior de la hoja **Discos**, seleccione **Migrar a discos administrados**.
5. Si la máquina virtual está en un conjunto de disponibilidad, habrá una advertencia en la hoja **Migrar a discos administrados** indicándole que debe convertir el conjunto de disponibilidad primero. La advertencia debería tener un vínculo en el que puede hacer clic para convertir el conjunto de disponibilidad. Una vez que se convierta el conjunto de disponibilidad o la máquina virtual no esté en un conjunto de disponibilidad, haga clic en **Migrar** para iniciar el proceso de migración de los discos a discos administrados.

La máquina virtual se detendrá y se reiniciará una vez completada la migración.

## <a name="next-steps"></a>Pasos siguientes

[Conversión de Managed Disks estándar a premium](convert-disk-storage.md)

Realice una copia de solo lectura de una máquina virtual mediante [instantáneas](snapshot-copy-managed-disk.md).
