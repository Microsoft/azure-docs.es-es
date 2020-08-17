---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/05/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 9e7386e21442b5a76aae656a36e2858b52ecef65
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87102997"
---
[!INCLUDE [virtual-machines-disks-incremental-snapshots-description](virtual-machines-disks-incremental-snapshots-description.md)]

### <a name="supported-regions"></a>Regiones admitidas
[!INCLUDE [virtual-machines-disks-incremental-snapshots-regions](virtual-machines-disks-incremental-snapshots-regions.md)]

## <a name="restrictions"></a>Restricciones

[!INCLUDE [virtual-machines-disks-incremental-snapshots-restrictions](virtual-machines-disks-incremental-snapshots-restrictions.md)]

## <a name="powershell"></a>PowerShell

Puede usar Azure PowerShell para crear y administrar recursos compartidos de archivos. Necesitará la versión más reciente de Azure PowerShell y el siguiente comando la instalará o actualizará la instalación existente a la más reciente:

```PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser
```

Una vez instalada, inicie sesión en PowerShell con `Connect-AzAccount`.

Para crear una instantánea incremental con Azure PowerShell, establezca la configuración con [New-AzSnapShotConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azsnapshotconfig?view=azps-2.7.0) con el parámetro `-Incremental` y, después, páselo como una variable a [New-AzSnapshot](https://docs.microsoft.com/powershell/module/az.compute/new-azsnapshot?view=azps-2.7.0) con el parámetro `-Snapshot`.

```PowerShell
$diskName = "yourDiskNameHere>"
$resourceGroupName = "yourResourceGroupNameHere"
$snapshotName = "yourDesiredSnapshotNameHere"

# Get the disk that you need to backup by creating an incremental snapshot
$yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName

# Create an incremental snapshot by setting the SourceUri property with the value of the Id property of the disk
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental 
New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig 
```

Puede identificar las instantáneas incrementales desde el mismo disco con las propiedades `SourceResourceId` y `SourceUniqueId` de las instantáneas. `SourceResourceId` es el identificador de recursos de Azure Resource Manager del disco principal. `SourceUniqueId` es el valor heredado de la propiedad `UniqueId` del disco. Si fuera a eliminar un disco y después creara otro con el mismo nombre, el valor de la propiedad `UniqueId` cambiaría.

Puede usar `SourceResourceId` y `SourceUniqueId` para crear una lista de todas las instantáneas asociadas a un disco determinado. Reemplace `<yourResourceGroupNameHere>` por su valor y, a continuación, puede usar el ejemplo siguiente para enumerar las instantáneas incrementales existentes:

```PowerShell
$snapshots = Get-AzSnapshot -ResourceGroupName $resourceGroupName

$incrementalSnapshots = New-Object System.Collections.ArrayList
foreach ($snapshot in $snapshots)
{
    
    if($snapshot.Incremental -and $snapshot.CreationData.SourceResourceId -eq $yourDisk.Id -and $snapshot.CreationData.SourceUniqueId -eq $yourDisk.UniqueId){

        $incrementalSnapshots.Add($snapshot)
    }
}

$incrementalSnapshots
```

## <a name="resource-manager-template"></a>Plantilla de Resource Manager

También puede usar plantillas de Azure Resource Manager para crear una instantánea incremental. Deberá asegurarse de que apiVersion esté establecido en **2019-03-01** y que la propiedad incremental también esté establecida en true. El siguiente fragmento de código es un ejemplo de cómo crear una instantánea incremental con las plantillas de Resource Manager:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "contosodisk1"
    },
  "diskResourceId": {
    "defaultValue": "<your_managed_disk_resource_ID>",
    "type": "String"
  }
  }, 
  "resources": [
  {
    "type": "Microsoft.Compute/snapshots",
    "name": "[concat( parameters('diskName'),'_snapshot1')]",
    "location": "[resourceGroup().location]",
    "apiVersion": "2019-03-01",
    "properties": {
      "creationData": {
        "createOption": "Copy",
        "sourceResourceId": "[parameters('diskResourceId')]"
      },
      "incremental": true
    }
  }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Si quiere ver un código de ejemplo que muestre la funcionalidad diferencial de las instantáneas incrementales con .NET, consulte [Copia de las copias de seguridad de Azure Managed Disks en otra región con una funcionalidad diferencial de instantáneas incrementales](https://github.com/Azure-Samples/managed-disks-dotnet-backup-with-incremental-snapshots).
