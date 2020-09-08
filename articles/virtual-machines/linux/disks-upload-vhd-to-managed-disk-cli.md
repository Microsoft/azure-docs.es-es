---
title: 'Carga de un VHD en Azure o copia de un disco entre regiones: CLI de Azure'
description: Obtenga información sobre cómo cargar un disco duro virtual en un disco administrado de Azure y copiar un disco administrado en todas las regiones, mediante la CLI de Azure, a través de una carga directa.
services: virtual-machines,storage
author: roygara
ms.author: rogarana
ms.date: 06/15/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: c7eb50caa4e7f0505809da64dd0309c6e0b8709f
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691350"
---
# <a name="upload-a-vhd-to-azure-or-copy-a-managed-disk-to-another-region---azure-cli"></a>Carga de un VHD en Azure o copia de un disco administrado en otra región: CLI de Azure

[!INCLUDE [disks-upload-vhd-to-disk-intro](../../../includes/disks-upload-vhd-to-disk-intro.md)]

## <a name="prerequisites"></a>Requisitos previos

- Descargue la [versión más reciente de AzCopy, v10](../../storage/common/storage-use-azcopy-v10.md#download-and-install-azcopy).
- [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
- Si tiene previsto cargar un disco duro virtual desde el entorno local: Un disco duro virtual de tamaño fijo que [se ha preparado para Azure](../windows/prepare-for-upload-vhd-image.md), almacenado localmente.
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

Antes de que pueda crear un HDD estándar vacío para cargar, necesitará el tamaño de archivo del VHD que quiere cargar, en bytes. Para ello, puede usar `wc -c <yourFileName>.vhd` o `ls -al <yourFileName>.vhd`. Este valor se usa al especificar el parámetro **- upload-size-bytes**.

Cree un HDD estándar vacío para la carga mediante la especificación tanto del parámetro **-–for-upload** como del parámetro **--upload-size-bytes** en un cmdlet [disk create](/cli/azure/disk#az-disk-create):

Reemplace `<yourdiskname>`, `<yourresourcegroupname>`, `<yourregion>` por los valores de su elección. El parámetro `--upload-size-bytes` contiene un valor de ejemplo de `34359738880`, reemplácelo por un valor adecuado.

> [!TIP]
> Si va a crear un disco del sistema operativo, agregue--hyper-v-generation <yourGeneration> a `az disk create`.

```azurecli
az disk create -n <yourdiskname> -g <yourresourcegroupname> -l <yourregion> --for-upload --upload-size-bytes 34359738880 --sku standard_lrs
```

Si desea cargar un disco SSD prémium o un disco SSD estándar, reemplace **standard_lrs** por **premium_LRS** o **standardssd_lrs**. Los discos Ultra no se admiten por ahora.

Ahora que ha creado un disco administrado vacío que está configurado para el proceso de carga, puede cargar en él un disco duro virtual. Para cargar un disco duro virtual en ese disco, necesitará una SAS grabable, con el fin de que pueda hacer referencia a ella como el destino de la carga.

Para generar una SAS grabable de un disco administrado vacío, reemplace `<yourdiskname>` y `<yourresourcegroupname>`; después, use el siguiente comando:

```azurecli
az disk grant-access -n <yourdiskname> -g <yourresourcegroupname> --access-level Write --duration-in-seconds 86400
```

Valor devuelto de ejemplo:

```output
{
  "accessSas": "https://md-impexp-t0rdsfgsdfg4.blob.core.windows.net/w2c3mj0ksfgl/abcd?sv=2017-04-17&sr=b&si=600a9281-d39e-4cc3-91d2-923c4a696537&sig=xXaT6mFgf139ycT87CADyFxb%2BnPXBElYirYRlbnJZbs%3D"
}
```

## <a name="upload-a-vhd"></a>Carga de un disco duro virtual

Ahora que tiene una SAS para el disco administrado vacío, puede usarla para establecer el disco administrado como destino del comando de carga.

Use AzCopy V10 para cargar el archivo VHD local en un disco administrado, para lo que se especifica el identificador URI de SAS que ha generado.

Esta carga tiene el mismo rendimiento que el [HDD estándar](disks-types.md#standard-hdd) equivalente. Por ejemplo, si tiene un tamaño que equivale a S4, tendrá un rendimiento de hasta 60 MiB/s. Pero si tiene un tamaño que equivale a S70, tendrá un rendimiento de hasta 500 MiB/s.

```bash
AzCopy.exe copy "c:\somewhere\mydisk.vhd" "sas-URI" --blob-type PageBlob
```

Cuando finalice la carga y ya no necesite escribir más datos en el disco, revoque la SAS. Al revocar la SAS, cambiará el estado del disco administrado y podrá conectar el disco a una máquina virtual.

Reemplace `<yourdiskname>` y `<yourresourcegroupname>` y, a continuación, use el siguiente comando para que se pueda usar el disco:

```azurecli
az disk revoke-access -n <yourdiskname> -g <yourresourcegroupname>
```

## <a name="copy-a-managed-disk"></a>Copia de un disco administrado

La carga directa también simplifica el proceso de copia de un disco administrado. Puede copiar dentro de la misma región o entre regiones (a otra región).

El script siguiente lo hará automáticamente, el proceso es similar a los pasos descritos anteriormente, con algunas diferencias porque está trabajando con un disco existente.

> [!IMPORTANT]
> Debe agregar un desplazamiento de 512 cuando proporcione el tamaño de disco en bytes de un disco administrado de Azure. Esto se debe a que Azure omite el pie de página al devolver el tamaño del disco. Si no lo hace, se producirá un error en la copia. El siguiente script ya lo hace automáticamente.

Reemplace los `<sourceResourceGroupHere>`, `<sourceDiskNameHere>`, `<targetDiskNameHere>`, `<targetResourceGroupHere>` y `<yourTargetLocationHere>` (un ejemplo de un valor de ubicación sería uswest2) con sus valores y, a continuación, ejecute el siguiente script para copiar un disco administrado.

> [!TIP]
> Si va a crear un disco del sistema operativo, agregue--hyper-v-generation <yourGeneration> a `az disk create`.

```azurecli
sourceDiskName = <sourceDiskNameHere>
sourceRG = <sourceResourceGroupHere>
targetDiskName = <targetDiskNameHere>
targetRG = <targetResourceGroupHere>
targetLocale = <yourTargetLocationHere>

sourceDiskSizeBytes= $(az disk show -g $sourceRG -n $sourceDiskName --query '[diskSizeBytes]' -o tsv)

az disk create -g $targetRG -n $targetDiskName -l $targetLocale --for-upload --upload-size-bytes $(($sourceDiskSizeBytes+512)) --sku standard_lrs

targetSASURI = $(az disk grant-access -n $targetDiskName -g $targetRG  --access-level Write --duration-in-seconds 86400 -o tsv)

sourceSASURI=$(az disk grant-access -n $sourceDiskName -g $sourceRG --duration-in-seconds 86400 --query [accessSas] -o tsv)

.\azcopy copy $sourceSASURI $targetSASURI --blob-type PageBlob

az disk revoke-access -n $sourceDiskName -g $sourceRG

az disk revoke-access -n $targetDiskName -g $targetRG
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha cargado correctamente un disco duro virtual en un disco administrado, puede conectar el disco como [disco de datos a una máquina virtual ya existente](add-disk.md) o [conectarlo como disco del sistema operativo](upload-vhd.md#create-the-vm) para crear una nueva máquina virtual. 
