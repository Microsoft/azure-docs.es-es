---
title: Actualización de los destinos de almacenamiento en Azure HPC Cache
description: Procedimientos para editar los destinos de almacenamiento en Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/02/2020
ms.author: v-erkel
ms.openlocfilehash: f11e12c4f30977514e04b09c7e1c3012eb7888a7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092463"
---
# <a name="edit-storage-targets"></a>Edición de los destinos de almacenamiento

Puede quitar o modificar un destino de almacenamiento desde la página **Destinos de almacenamiento** del portal o mediante la CLI de Azure.

> [!TIP]
> En el [vídeo de administración de Azure HPC Cache](https://azure.microsoft.com/resources/videos/managing-hpc-cache/) se muestra cómo editar un destino de almacenamiento en Azure Portal.

## <a name="remove-a-storage-target"></a>Eliminación de un destino de almacenamiento

### <a name="portal"></a>[Portal](#tab/azure-portal)

Para quitar un destino de almacenamiento, selecciónelo en la lista y haga clic en el botón **Eliminar**.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

Use [az hpc-cache storage-target remove](/cli/azure/ext/hpc-cache/hpc-cache/storage-target#ext-hpc-cache-az-hpc-cache-storage-target-remove) para eliminar un destino de almacenamiento de la memoria caché.

```azurecli
$ az hpc-cache storage-target remove --resource-group cache-rg --cache-name doc-cache0629 --name blob1

{- Finished ..
  "endTime": "2020-07-09T21:45:06.1631571+00:00",
  "name": "2f95eac1-aded-4860-b19c-3f089531a7ec",
  "startTime": "2020-07-09T21:43:38.5461495+00:00",
  "status": "Succeeded"
}
```

---

Esta acción quita la asociación del destino de almacenamiento con este sistema de Azure HPC Cache, pero no cambia el sistema de almacenamiento de back-end. Por ejemplo, si ha usado un contenedor Azure Blob Storage, el contenedor y su contenido siguen existiendo después de eliminarlo de la memoria caché. Puede agregar el contenedor a otra instancia de Azure HPC Cache, volver a agregarlo a esta caché o eliminarlo con Azure Portal.

Los cambios de archivo almacenados en la caché se escriben en el sistema de almacenamiento de back-end antes de que se quite el destino de almacenamiento. Este proceso puede tardar una hora o más si hay muchos datos cambiados en la caché.

## <a name="update-storage-targets"></a>Actualización de destinos de almacenamiento

Se pueden editar los destinos de almacenamiento para modificar algunas de sus propiedades. Para cada tipo de almacenamiento se pueden modificar distintas propiedades:

* En el caso de los destinos de almacenamiento de blob, se puede cambiar la ruta de acceso del espacio de nombres.

* En el caso de los destinos de almacenamiento de NFS, se pueden cambiar estas propiedades:

  * Ruta de acceso del espacio de nombres
  * Modelo de uso
  * Exportación
  * Subdirectorio de exportación

No se puede editar el nombre, el tipo o el sistema de almacenamiento de back-end de un destino de almacenamiento (contenedor de blobs, o dirección IP o nombre de host de NFS). Si tiene que cambiar estas propiedades, elimine el destino de almacenamiento y cree un reemplazo con el valor nuevo.

En Azure Portal, puede ver qué campos se pueden editar si hace clic en el nombre del destino de almacenamiento y abre la página de detalles. También puede modificar los destinos de almacenamiento con la CLI de Azure.

![captura de pantalla de la página de edición de un destino de almacenamiento de NFS](media/hpc-cache-edit-storage-nfs.png)

## <a name="update-an-nfs-storage-target"></a>Actualización de un destino de almacenamiento de NFS

En el caso de un destino de almacenamiento de NFS, se pueden actualizar varias propiedades. (Consulte la captura de pantalla anterior para obtener una página de edición de ejemplo).

* **Modelo de uso**: el modelo de uso influye en el modo en que la memoria caché conserva los datos. Lea [Elección de un modelo de uso](hpc-cache-add-storage.md#choose-a-usage-model) para obtener más información.
* **Ruta de acceso del espacio de nombres virtual**: ruta de acceso que los clientes usan para montar este destino de almacenamiento. Lea [Planeamiento del espacio de nombres agregado](hpc-cache-namespace.md) para obtener más detalles.
* **Ruta de acceso de exportación de NFS**: exportación del sistema de almacenamiento que se va a usar para esta ruta de acceso del espacio de nombres.
* **Ruta de acceso del subdirectorio**: subdirectorio (en la exportación) que se va a asociar a esta ruta de acceso del espacio de nombres. Deje este campo en blanco si no necesita especificar un subdirectorio.

Cada ruta de acceso del espacio de nombres necesita una combinación única de exportación y subdirectorio. Es decir, no se pueden crear dos rutas de acceso de cliente distintas al mismo directorio en el sistema de almacenamiento de back-end.

### <a name="portal"></a>[Portal](#tab/azure-portal)

Después de realizar los cambios, haga clic en **Aceptar** para actualizar el destino de almacenamiento o en **Cancelar** para descartar los cambios.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

Use el comando [az nfs-storage-target](/cli/azure/ext/hpc-cache/hpc-cache/nfs-storage-target) para cambiar el modelo de uso, la ruta de acceso del espacio de nombres virtual, y los valores de exportación o subdirectorio de NFS para un destino de almacenamiento.

* Para cambiar el modelo de uso, use la opción ``--nfs3-usage-model``. Ejemplo: ``--nfs3-usage-model WRITE_WORKLOAD_15``

* Para cambiar la ruta de acceso del espacio de nombres, exportar o exportar un subdirectorio, use la opción ``--junction``.

  El parámetro ``--junction`` usa estos valores:

  * ``namespace-path``: Ruta de acceso del archivo virtual orientado al cliente
  * ``nfs-export``: Exportación del sistema de almacenamiento que se va a asociar a la ruta de acceso orientada al cliente
  * ``target-path`` (opcional): Subdirectorio de la exportación, si es necesario

  Ejemplo: ``--junction namespace-path="/nas-1" nfs-export="/datadisk1" target-path="/test"``

El nombre de la memoria caché, el nombre del destino de almacenamiento y el grupo de recursos son obligatorios en todos los comandos de actualización.

Ejemplo de comando: <!-- having problem testing this -->

```azurecli
az hpc-cache nfs-storage-target update --cache-name mycache \
    --name rivernfs0 --resource-group doc-rg0619 \
    --nfs3-usage-model READ_HEAVY_INFREQ
```

Si la memoria caché se ha detenido o no tiene un estado correcto, la actualización se aplica después de que la memoria caché vuelve al estado correcto.

---

## <a name="update-an-azure-blob-storage-target"></a>Actualización de un destino de almacenamiento de blobs de Azure

En el caso de un destino de almacenamiento de blobs, puede modificar la ruta de acceso del espacio de nombres virtual.

### <a name="portal"></a>[Portal](#tab/azure-portal)

La página de detalles de un destino de almacenamiento de blob permite modificar la ruta de acceso del espacio de nombres virtual.

![captura de pantalla de la página de edición de un destino de almacenamiento de blob](media/hpc-cache-edit-storage-blob.png)

Cuando se haya terminado, haga clic en **Aceptar** para actualizar el destino de almacenamiento o en **Cancelar** para descartar los cambios.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

Use [az hpc-cache blob-storage-target update](/cli/azure/ext/hpc-cache/hpc-cache/blob-storage-target#ext-hpc-cache-az-hpc-cache-blob-storage-target-update) para actualizar la ruta de acceso del espacio de nombres de un destino.

```azurecli
az hpc-cache blob-storage-target update --cache-name cache-name --name target-name \
    --resource-group rg --storage-account "/subscriptions/<subscription_ID>/resourceGroups/erinazcli/providers/Microsoft.Storage/storageAccounts/rg"  \
    --container-name "container-name" --virtual-namespace-path "/new-path"
```

Si la memoria caché se ha detenido o no tiene un estado correcto, la actualización se aplicará después de que la memoria caché vuelva al estado correcto.

---

## <a name="next-steps"></a>Pasos siguientes

* Lea [Incorporación de destinos de almacenamiento](hpc-cache-add-storage.md) para obtener más información sobre estas opciones.
* Lea [Planeamiento del espacio de nombres agregado](hpc-cache-namespace.md) para obtener más recomendaciones sobre el uso de rutas de acceso virtuales.
