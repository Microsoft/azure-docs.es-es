---
title: 'Habilitación de la eliminación temporal: recursos compartidos de archivos de Azure'
description: Obtenga información sobre cómo habilitar la eliminación temporal (versión preliminar) en recursos compartidos de archivos de Azure para la recuperación de datos y para evitar la eliminación accidental.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 05/28/2020
ms.author: rogarana
ms.subservice: files
services: storage
ms.openlocfilehash: 2d2a000879a95f86a6cdda3324add5b692476eee
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590122"
---
# <a name="enable-soft-delete-on-azure-file-shares"></a>Habilitación de la eliminación temporal en recursos compartidos de archivos de Azure

Azure Storage ofrece la posibilidad de eliminar temporalmente recursos compartidos de archivos (versión preliminar), con el fin de que pueda recuperar más fácilmente los datos cuando una aplicación u otro usuario de la cuenta de almacenamiento los hayan eliminado por error. Para más información sobre la eliminación temporal, consulte [Cómo evitar la eliminación accidental de recursos compartidos de archivos de Azure](storage-files-prevent-file-share-deletion.md).

En las secciones siguientes se muestra cómo habilitar y usar la eliminación temporal para recursos compartidos de archivos de Azure en una cuenta de almacenamiento existente:

# <a name="portal"></a>[Portal](#tab/azure-portal)

## <a name="getting-started"></a>Introducción

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).
1. Vaya a la cuenta de almacenamiento y seleccione **Eliminación temporal** en **Servicio de archivos**.
1. Seleccione **Habilitada** en **Eliminación temporal de recursos compartidos de archivos**.
1. Seleccione **File share retention period in days** (Período de retención del recurso compartido de archivos en días) y escriba el número que prefiera.
1. Seleccione **Guardar** para confirmar la configuración de retención de datos.

:::image type="content" source="media/storage-how-to-recover-deleted-account/enable-soft-delete-files.png" alt-text="Captura de pantalla del panel de configuración de la eliminación temporal de la cuenta de almacenamiento. Resalte la sección de recursos compartidos de archivos, habilite la alternancia, establezca un período de retención y guárdelo. De esta forma, habilitará la eliminación temporal para todos los recursos compartidos de archivos de la cuenta de almacenamiento.":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="prerequisite"></a>Requisito previo

Los cmdlets de eliminación temporal están disponibles en la versión preliminar [2.1.1](https://www.powershellgallery.com/packages/Az.Storage/2.1.1-preview) y la versión preliminar [2.3.1-](https://www.powershellgallery.com/packages/Az.Storage/2.3.1-preview)del Módulo Az.Storage. 

## <a name="getting-started"></a>Introducción

Para habilitar la eliminación temporal, debe actualizar las propiedades de servicio de un cliente de archivo. En el ejemplo siguiente se habilita la eliminación temporal para todos los recursos compartidos de archivos en una cuenta de almacenamiento:

```azurepowershell-interactive
$rgName = "yourResourceGroupName"
$accountName = "yourStorageAccountName"

Update-AzStorageFileServiceProperty -ResourceGroupName $rgName -StorageAccountName $accountName -EnableShareDeleteRetentionPolicy $true -ShareRetentionDays 7
```

Puede comprobar si la eliminación temporal está habilitada y ver su directiva de retención con el siguiente comando:

```azurepowershell-interactive
Get-AzStorageFileServiceProperty -ResourceGroupName $rgName -StorageAccountName $accountName
```
---

## <a name="restore-soft-deleted-file-share"></a>Restauración del recurso compartido de archivos eliminado temporalmente

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para restaurar un recurso compartido de archivos eliminado temporalmente:

1. Vaya a la cuenta de almacenamiento y seleccione **Recursos compartidos de archivos**.
1. En la hoja del recurso compartido de archivos, habilite **Show deleted shares** (Mostrar recursos compartidos eliminados) para mostrar los recursos compartidos que se han eliminado temporalmente.

    Se mostrarán los recursos compartidos que se encuentren en estado **Eliminado**.

    :::image type="content" source="media/storage-how-to-recover-deleted-account/undelete-file-share.png" alt-text="Si la columna de estado, situada junto a la columna de nombre, se establece en Eliminado, el recurso compartido de archivos se encuentra en un estado de eliminación temporal. Se eliminará de forma permanente después del período de retención especificado.":::

1. Seleccione el recurso compartido y **recuperar**; de esta forma se restaurará el recurso compartido.

    Puede confirmar que el recurso compartido se haya restaurado desde que su estado cambia a **Activo**.

    :::image type="content" source="media/storage-how-to-recover-deleted-account/restored-file-share.png" alt-text="Si la columna de estado, la columna situada junto a la columna de nombre, se establece en Activo, el recurso compartido de archivos se ha restaurado.":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Los cmdlets de eliminación temporal están disponibles en la versión 2.1.1-preview del Módulo Az.Storage. Para restaurar un recurso compartido de archivos eliminado temporalmente, use el siguiente comando:

```azurepowershell-interactive
Restore-AzRmStorageShare -ResourceGroupName $rgname -StorageAccountName $accountName -DeletedShareVersion 01D5E2783BDCDA97
```
---

## <a name="disable-soft-delete"></a>Deshabilitación de la eliminación temporal

Si desea detener la eliminación temporal o eliminar permanentemente un recurso compartido de archivos, siga estas instrucciones:

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la cuenta de almacenamiento y seleccione **Eliminación temporal** en **Configuración**.
1. En **Recursos compartidos de archivos** seleccione **Deshabilitado** en **Soft delete for file shares** (Eliminación temporal para recursos compartidos de archivos).
1. Seleccione **Guardar** para confirmar la configuración de retención de datos.

    :::image type="content" source="media/storage-how-to-recover-deleted-account/disable-soft-delete-files.png" alt-text="Deshabilitar la eliminación temporal le permitirá eliminar de forma inmediata y permanente todos los recursos compartidos de archivos de la cuenta de almacenamiento cuando descanse.":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Los cmdlets de eliminación temporal están disponibles en la versión 2.1.1-preview del Módulo Az.Storage. Puede usar el siguiente comando para deshabilitar la eliminación temporal en la cuenta de almacenamiento:

```azurepowershell-interactive
Update-AzStorageFileServiceProperty -ResourceGroupName $rgName -StorageAccountName $accountName -EnableShareDeleteRetentionPolicy $false
```
---

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre otra forma de protección y recuperación de datos, consulte nuestro artículo [Introducción a las instantáneas de recursos compartidos de Azure Files](storage-snapshots-files.md).
