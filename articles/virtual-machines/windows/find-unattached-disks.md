---
title: Búsqueda y eliminación de discos administrados y no administrados de Azure no conectados
description: Cómo buscar y eliminar discos administrados y no administrados (VHD o blobs en páginas) de Azure no asociados mediante Azure PowerShell.
author: roygara
ms.service: virtual-machines-windows
ms.topic: how-to
ms.date: 02/22/2019
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: d1e7c90e558a6834a169b528d2e8c2f96af377b0
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705705"
---
# <a name="find-and-delete-unattached-azure-managed-and-unmanaged-disks"></a>Búsqueda y eliminación de discos administrados y no administrados de Azure no conectados

Cuando se elimina una máquina virtual (VM) en Azure, de forma predeterminada, no se elimina ningún disco asociado a la máquina virtual. Esta característica ayuda a evitar la pérdida de datos debido a la eliminación accidental de máquinas virtuales. Después de eliminar una máquina virtual, seguirá pagando por los discos no asociados. En este artículo se muestra cómo buscar y eliminar los discos no asociados y reducir costos innecesarios.

## <a name="managed-disks-find-and-delete-unattached-disks"></a>Discos administrados: Búsqueda y eliminación de discos no conectados

El siguiente script busca los [discos administrados](managed-disks-overview.md) no asociados examinando el valor de la propiedad **ManagedBy**. Cuando un disco administrado está asociado a una máquina virtual, la propiedad **ManagedBy** contiene el identificador de recurso de la máquina virtual. Cuando se desasocia, le propiedad **ManagedBy** tiene un valor null. El script examina todos los discos administrados de una suscripción de Azure. Cuando el script localiza un disco administrad con la propiedad **ManagedBy** establecida en null, se determina que el disco no está asociado.

>[!IMPORTANT]
>Primero, ejecute el script y establezca la variable **deleteUnattachedDisks** en 0. Esta acción le permite buscar y ver todos los discos administrados no asociados.
>
>Después de revisar todos los discos no asociados, vuelva a ejecutar el script y establezca la variable **deleteUnattachedDisks** en 1. Esta acción le permite eliminar todos los discos administrados no asociados.

```azurepowershell-interactive
# Set deleteUnattachedDisks=1 if you want to delete unattached Managed Disks
# Set deleteUnattachedDisks=0 if you want to see the Id of the unattached Managed Disks
$deleteUnattachedDisks=0
$managedDisks = Get-AzDisk
foreach ($md in $managedDisks) {
    # ManagedBy property stores the Id of the VM to which Managed Disk is attached to
    # If ManagedBy property is $null then it means that the Managed Disk is not attached to a VM
    if($md.ManagedBy -eq $null){
        if($deleteUnattachedDisks -eq 1){
            Write-Host "Deleting unattached Managed Disk with Id: $($md.Id)"
            $md | Remove-AzDisk -Force
            Write-Host "Deleted unattached Managed Disk with Id: $($md.Id) "
        }else{
            $md.Id
        }
    }
 }
```

## <a name="unmanaged-disks-find-and-delete-unattached-disks"></a>Discos no administrados: Búsqueda y eliminación de discos no conectados

Los discos no administrados son archivos VHD almacenados como [blobs en páginas](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) en [cuentas de Azure Storage](../../storage/common/storage-account-overview.md). El siguiente script busca discos no administrados no asociados (blobs en páginas) examinando el valor de la propiedad **LeaseStatus**. Cuando un disco no administrado está asociado a una máquina virtual, la propiedad **LeaseStatus** se establece en **Bloqueado**. Cuando un disco no administrado está desasociado, la propiedad **LeaseStatus** se establece en **Desbloqueado**. El script examina todos los discos no administrados de todas las cuentas de almacenamiento de Azure de una suscripción de Azure. Cuando el script localiza un disco no administrado con la propiedad **LeaseStatus** establecida en **Desbloqueado**, se determina que el disco no está asociado.

>[!IMPORTANT]
>Primero, ejecute el script y establezca la variable **deleteUnattachedVHDs** en `$false`. Esta acción le permite buscar y ver todos los discos duros virtuales no administrados y no asociados.
>
>Después de revisar todos los discos no asociados, vuelva a ejecutar el script y establezca la variable **deleteUnattachedVHDs** en `$true`. Esta acción le permite eliminar todos los discos duros virtuales no administrados y no asociados.

```azurepowershell-interactive
# Set deleteUnattachedVHDs=$true if you want to delete unattached VHDs
# Set deleteUnattachedVHDs=$false if you want to see the Uri of the unattached VHDs
$deleteUnattachedVHDs=$false
$storageAccounts = Get-AzStorageAccount
foreach($storageAccount in $storageAccounts){
    $storageKey = (Get-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName -Name $storageAccount.StorageAccountName)[0].Value
    $context = New-AzStorageContext -StorageAccountName $storageAccount.StorageAccountName -StorageAccountKey $storageKey
    $containers = Get-AzStorageContainer -Context $context
    foreach($container in $containers){
        $blobs = Get-AzStorageBlob -Container $container.Name -Context $context
        #Fetch all the Page blobs with extension .vhd as only Page blobs can be attached as disk to Azure VMs
        $blobs | Where-Object {$_.BlobType -eq 'PageBlob' -and $_.Name.EndsWith('.vhd')} | ForEach-Object { 
            #If a Page blob is not attached as disk then LeaseStatus will be unlocked
            if($_.ICloudBlob.Properties.LeaseStatus -eq 'Unlocked'){
                    if($deleteUnattachedVHDs){
                        Write-Host "Deleting unattached VHD with Uri: $($_.ICloudBlob.Uri.AbsoluteUri)"
                        $_ | Remove-AzStorageBlob -Force
                        Write-Host "Deleted unattached VHD with Uri: $($_.ICloudBlob.Uri.AbsoluteUri)"
                    }
                    else{
                        $_.ICloudBlob.Uri.AbsoluteUri
                    }
            }
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte [Eliminación de una cuenta de almacenamiento](../../storage/common/storage-account-create.md#delete-a-storage-account) e [Identificación de discos huérfanos con PowerShell](/archive/blogs/ukplatforms/azure-cost-optimisation-series-identify-orphaned-disks-using-powershell).
