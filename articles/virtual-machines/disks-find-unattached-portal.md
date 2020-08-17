---
title: 'Identificación de discos de Azure no conectados: Azure Portal'
description: Cómo buscar discos administrados y no administrados (VHD o blobs en páginas) de Azure no conectados mediante la CLI de Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 06/01/2020
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 6632d65fa07788e35b24c2f957e713f824f6b091
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542745"
---
# <a name="find-and-delete-unattached-azure-managed-and-unmanaged-disks---azure-portal"></a>Búsqueda y eliminación de discos administrados y no administrados de Azure no conectados: Azure Portal

Cuando se elimina una máquina virtual (VM) en Azure, de forma predeterminada, no se elimina ningún disco asociado a la máquina virtual. Esto ayuda a evitar la pérdida de datos debido a la eliminación accidental de máquinas virtuales. Después de eliminar una máquina virtual, seguirá pagando por los discos no asociados. En este artículo se muestra cómo buscar y eliminar los discos no conectados mediante Azure Portal y reducir costos innecesarios.

## <a name="managed-disks-find-and-delete-unattached-disks"></a>Discos administrados: Búsqueda y eliminación de discos no conectados

Si tiene discos administrados no conectados y ya no necesita los datos en ellos, el siguiente proceso explica cómo encontrarlos en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Busque y seleccione **Discos**.

    En la hoja **Discos**, se muestra una lista de todos los discos. Cualquier disco que tenga " **-** " en la columna **Propietario** es un disco no conectado.

    [![Captura de pantalla de la hoja de discos administrados, si un disco tiene - en la columna Propietario, se trata de un disco no conectado.](media/disks-find-unattached-portal/managed-disk-unattached-owner.png)](media/disks-find-unattached-portal/managed-disk-owner-unattached.png#lightbox)

1. Seleccione el disco no conectado que desea eliminar; de este modo, se abrirá la hoja del disco.
1. En la hoja del disco, puede confirmar que el disco tiene un estado no conectado y, después, seleccionar **Eliminar**.

    :::image type="content" source="media/disks-find-unattached-portal/delete-managed-disk-unattached.png" alt-text="Captura de pantalla de una hoja de discos administrados individuales. Esta hoja mostrará Sin adjuntar en el estado del disco si no está conectado. Puede eliminar este disco si no necesita conservar los datos más tiempo.":::

## <a name="unmanaged-disks-find-and-delete-unattached-disks"></a>Discos no administrados: Búsqueda y eliminación de discos no conectados

Los discos no administrados son archivos VHD almacenados como [blobs en páginas](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) en [cuentas de Azure Storage](../storage/common/storage-account-overview.md).

Si tiene discos no administrados que no están conectados a una máquina virtual, ya no necesita los datos en ellos y desea eliminarlos, en el siguiente proceso se explica cómo hacerlo desde Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Busque y seleccione **Discos (clásico)** .

    Se le mostrará una lista de todos los discos no administrados. Cualquier disco que tenga " **-** " en la columna **Conectado a** es un disco no conectado.

    :::image type="content" source="media/disks-find-unattached-portal/unmanaged-disk-unattached-attached-to.png" alt-text="Captura de pantalla de la hoja discos no administrados. Los discos de esta hoja que tienen - en la columna Conectado a no están conectados.":::

1. Seleccione el disco no conectado que desea eliminar; se abrirá la hoja del disco.

1. En la hoja del disco, puede confirmar que está desconectado, ya que **Conectado a** seguirá siendo **-** .

    :::image type="content" source="media/disks-find-unattached-portal/unmanaged-disk-unattached-select-blade.png" alt-text="Captura de pantalla de una hoja individual de un disco no administrado. Tendrá - como el valor Conectado a si no está conectado. Si ya no necesita los datos de este disco, puede eliminarlos.":::

1. Seleccione **Eliminar**.

    :::image type="content" source="media/disks-find-unattached-portal/delete-unmanaged-disk-unattached.png" alt-text="Captura de pantalla de una hoja individual de un disco no administrado, con Eliminar resaltado.":::

## <a name="next-steps"></a>Pasos siguientes

Si desea una forma automatizada de buscar y eliminar cuentas de almacenamiento no asociadas, consulte nuestros artículos sobre la [CLI](linux/find-unattached-disks.md) o [PowerShell](windows/find-unattached-disks.md).

Para más información, consulte [Eliminación de una cuenta de almacenamiento](../storage/common/storage-account-create.md#delete-a-storage-account) e [Identificación de discos huérfanos con PowerShell](/archive/blogs/ukplatforms/azure-cost-optimisation-series-identify-orphaned-disks-using-powershell).
