---
title: 'Carga de un VHD en Azure o copia de un disco entre regiones: Azure PowerShell'
description: Aprenda a cargar un VHD en un disco administrado de Azure y a copiar un disco administrado entre regiones mediante Azure PowerShell a través de la carga directa.
author: roygara
ms.author: rogarana
ms.date: 06/15/2020
ms.topic: how-to
ms.service: virtual-machines
ms.tgt_pltfrm: linux
ms.subservice: disks
ms.openlocfilehash: ec765e2eb635449140d60ee98762e35aea12fe44
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689616"
---
# <a name="upload-a-vhd-to-azure-or-copy-a-managed-disk-to-another-region---azure-powershell"></a>Carga de un VHD en Azure o copia de un disco administrado en otra región: Azure PowerShell

[!INCLUDE [disks-upload-vhd-to-disk-intro](../../../includes/disks-upload-vhd-to-disk-intro.md)]

## <a name="prerequisites"></a>Requisitos previos

- Descargue la [versión más reciente de AzCopy, v10](../../storage/common/storage-use-azcopy-v10.md#download-and-install-azcopy).
- [Instale el módulo de Azure PowerShell](/powershell/azure/install-Az-ps).
- Si tiene previsto cargar un disco duro virtual desde el entorno local: Un disco duro virtual de tamaño fijo que [se ha preparado para Azure](prepare-for-upload-vhd-image.md), almacenado localmente.
- O bien, un disco administrado en Azure, si desea realizar una acción de copia.

## <a name="getting-started"></a>Introducción

Si prefiere cargar discos a través de una interfaz gráfica de usuario, puede hacerlo mediante el Explorador de Azure Storage. Para obtener más detalles, consulte: [Uso del Explorador de Azure Storage para administrar discos administrados de Azure](../disks-use-storage-explorer-managed-disks.md)

Para cargar un disco duro virtual en Azure, deberá crear un disco administrado vacío que esté configurado para este proceso de carga. Antes de crearlo, hay información adicional que debe saber acerca de estos discos.

Este tipo de disco administrado tiene dos estados únicos:

- ReadToUpload, que significa que el disco está listo para recibir una carga pero no se ha generado ninguna [firma de acceso seguro](../../storage/common/storage-sas-overview.md).
- ActiveUpload, que significa que el disco está listo para recibir una carga y se ha generado la SAS.

> [!NOTE]
> Mientras esté en cualquiera de estos estados, el disco administrado se facturará al [precio de un HDD estándar](https://azure.microsoft.com/pricing/details/managed-disks/), independientemente del tipo real de disco. Por ejemplo, un P10 se facturará como un S10. Esto sucederá hasta que se llame a `revoke-access` en el disco administrado, que es necesario para conectar el disco a una máquina virtual.

## <a name="create-an-empty-managed-disk"></a>Creación de un disco administrado vacío

Antes de que pueda crear un HDD estándar vacío para cargar, necesitará el tamaño de archivo del VHD que quiere cargar, en bytes. El código de ejemplo lo recibirá, pero para hacerlo usted puede usar: `$vhdSizeBytes = (Get-Item "<fullFilePathHere>").length`. Este valor se usa al especificar el parámetro **-UploadSizeInBytes**.

Ahora, en el shell local, cree un HDD estándar vacío para la carga mediante la especificación del valor **Upload** en el parámetro **-CreateOption**, así como el parámetro **-UploadSizeInBytes** del cmdlet [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig?view=azps-1.8.0). Después, llame a [New-AzDisk](/powershell/module/az.compute/new-azdisk?view=azps-1.8.0) para crear el disco.

Reemplace `<yourdiskname>`, `<yourresourcegroupname>` y `<yourregion>`; luego, ejecute los siguientes comandos:

> [!TIP]
> Si va a crear un disco del sistema operativo, agregue -HyperVGeneration "<yourGeneration>" a `New-AzDiskConfig`.

```powershell
$vhdSizeBytes = (Get-Item "<fullFilePathHere>").length

$diskconfig = New-AzDiskConfig -SkuName 'Standard_LRS' -OsType 'Windows' -UploadSizeInBytes $vhdSizeBytes -Location '<yourregion>' -CreateOption 'Upload'

New-AzDisk -ResourceGroupName '<yourresourcegroupname' -DiskName '<yourdiskname>' -Disk $diskconfig
```

Si desea cargar un disco SSD prémium o un disco SSD estándar, reemplace **Standard_LRS** por **Premium_LRS** o **StandardSSD_LRS**. Todavía no se admiten los discos Ultra.

Ahora que ha creado un disco administrado vacío que está configurado para el proceso de carga, puede cargar en él un VHD. Para cargar un VHD en ese disco, necesitará una SAS grabable, con el fin de hacer referencia a ella como destino de la carga.

Para generar una SAS grabable de un disco administrado vacío, reemplace `<yourdiskname>` y `<yourresourcegroupname>`; después, use los siguientes comandos:

```powershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>' -DurationInSecond 86400 -Access 'Write'

$disk = Get-AzDisk -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>'
```

## <a name="upload-a-vhd"></a>Carga de un disco duro virtual

Ahora que tiene una SAS para el disco administrado vacío, puede usarla para establecer el disco administrado como destino del comando de carga.

Use AzCopy V10 para cargar el archivo VHD local en un disco administrado, para lo que se especifica el identificador URI de SAS que ha generado.

Esta carga tiene el mismo rendimiento que el [HDD estándar](disks-types.md#standard-hdd) equivalente. Por ejemplo, si tiene un tamaño que equivale a S4, tendrá un rendimiento de hasta 60 MiB/s. Pero si tiene un tamaño que equivale a S70, tendrá un rendimiento de hasta 500 MiB/s.

```
AzCopy.exe copy "c:\somewhere\mydisk.vhd" $diskSas.AccessSAS --blob-type PageBlob
```

Cuando finalice la carga y ya no necesite escribir más datos en el disco, revoque la SAS. Al revocar la SAS, cambiará el estado del disco administrado y podrá conectar el disco a una máquina virtual.

Reemplace `<yourdiskname>` y `<yourresourcegroupname>`; luego, ejecute el siguiente comando:

```powershell
Revoke-AzDiskAccess -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>'
```

## <a name="copy-a-managed-disk"></a>Copia de un disco administrado

La carga directa también simplifica el proceso de copia de un disco administrado. Puede copiar dentro de la misma región, o bien copiar el disco administrado a otra región.

El script siguiente lo hará automáticamente, el proceso es similar a los pasos descritos anteriormente, con algunas diferencias porque está trabajando con un disco existente.

> [!IMPORTANT]
> Debe agregar un desplazamiento de 512 cuando proporcione el tamaño de disco en bytes de un disco administrado de Azure. Esto se debe a que Azure omite el pie de página al devolver el tamaño del disco. Si no lo hace, se producirá un error en la copia. El siguiente script ya lo hace automáticamente.

Reemplace los `<sourceResourceGroupHere>`, `<sourceDiskNameHere>`, `<targetDiskNameHere>`, `<targetResourceGroupHere>`, `<yourOSTypeHere>` y `<yourTargetLocationHere>` (un ejemplo de un valor de ubicación sería uswest2) con sus valores y, a continuación, ejecute el siguiente script para copiar un disco administrado.

> [!TIP]
> Si va a crear un disco del sistema operativo, agregue -HyperVGeneration "<yourGeneration>" a `New-AzDiskConfig`.

```powershell

$sourceRG = <sourceResourceGroupHere>
$sourceDiskName = <sourceDiskNameHere>
$targetDiskName = <targetDiskNameHere>
$targetRG = <targetResourceGroupHere>
$targetLocate = <yourTargetLocationHere>
#Expected value for OS is either "Windows" or "Linux"
$targetOS = <yourOSTypeHere>

$sourceDisk = Get-AzDisk -ResourceGroupName $sourceRG -DiskName $sourceDiskName

# Adding the sizeInBytes with the 512 offset, and the -Upload flag
$targetDiskconfig = New-AzDiskConfig -SkuName 'Standard_LRS' -osType $targetOS -UploadSizeInBytes $($sourceDisk.DiskSizeBytes+512) -Location $targetLocate -CreateOption 'Upload'

$targetDisk = New-AzDisk -ResourceGroupName $targetRG -DiskName $targetDiskName -Disk $targetDiskconfig

$sourceDiskSas = Grant-AzDiskAccess -ResourceGroupName $sourceRG -DiskName $sourceDiskName -DurationInSecond 86400 -Access 'Read'

$targetDiskSas = Grant-AzDiskAccess -ResourceGroupName $targetRG -DiskName $targetDiskName -DurationInSecond 86400 -Access 'Write'

azcopy copy $sourceDiskSas.AccessSAS $targetDiskSas.AccessSAS --blob-type PageBlob

Revoke-AzDiskAccess -ResourceGroupName $sourceRG -DiskName $sourceDiskName

Revoke-AzDiskAccess -ResourceGroupName $targetRG -DiskName $targetDiskName 
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha cargado correctamente un VHD en un disco administrado, puede conectar el disco a una máquina virtual y empezar a usarlo.

Para aprender a conectar un disco de datos a una máquina virtual, consulte nuestro artículo al respecto: [Conexión de un disco a una VM Windows con PowerShell](attach-disk-ps.md). Para usar el disco como disco del sistema operativo, consulte [Creación de una máquina virtual Windows desde un disco especializado](create-vm-specialized.md#create-the-new-vm).
