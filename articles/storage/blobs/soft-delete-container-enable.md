---
title: Habilitación y administración de la eliminación temporal para contenedores (versión preliminar)
titleSuffix: Azure Storage
description: Habilitación de la eliminación temporal de contenedores (versión preliminar) para recuperar sus datos con mayor facilidad al modificarse o eliminarse estos de forma errónea.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/10/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: c2ce40f7d8a8d08e191fce11c98bd780e0c2fda1
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230082"
---
# <a name="enable-and-manage-soft-delete-for-containers-preview"></a>Habilitación y administración de la eliminación temporal para contenedores (versión preliminar)

La eliminación temporal de contenedores (versión preliminar) evita que los datos se modifiquen o eliminen de forma accidental o errónea. Cuando la eliminación temporal de contenedores está habilitada para una cuenta de almacenamiento, un contenedor y su contenido se pueden recuperar después de eliminarse, dentro de un período de retención especificado.

Si existe la posibilidad de que una aplicación u otro usuario de la cuenta de almacenamiento modifiquen o borren sus datos accidentalmente, Microsoft aconseja activar la eliminación temporal de contenedores. En este artículo se muestra cómo habilitar la eliminación temporal para contenedores. Para más información sobre la eliminación temporal de contenedores, incluida la forma de registrarse para la versión preliminar, vea [Eliminación temporal para contenedores (versión preliminar)](soft-delete-container-overview.md).

Para la protección de datos de un extremo a otro, Microsoft recomienda que también habilite la eliminación temporal para blobs y el control de versiones de blobs. Para información sobre cómo habilitar también la eliminación temporal para blobs, consulte [Habilitación y administración de la eliminación temporal para blobs](soft-delete-blob-enable.md). Para información sobre cómo habilitar el control de versiones de blobs, consulte [Control de versiones de blobs](versioning-overview.md).

## <a name="enable-container-soft-delete"></a>Habilitación de la eliminación temporal de contenedores

Puede habilitar o deshabilitar la eliminación temporal de contenedores para la cuenta de almacenamiento en cualquier momento mediante Azure Portal o una plantilla de Azure Resource Manager.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para habilitar la eliminación temporal de contenedores para la cuenta de almacenamiento mediante Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), vaya a la cuenta de almacenamiento.
1. Vaya a la opción **Protección de datos** en **Blob service**.
1. Establezca la propiedad **Container soft delete** (Eliminación temporal de contenedores) en *Habilitada*.
1. En **Directivas de retención**, especifique cuánto tiempo se conservan los contenedores eliminados temporalmente mediante Azure Storage.
1. Guarde los cambios.

:::image type="content" source="media/soft-delete-container-enable/soft-delete-container-portal-configure.png" alt-text="Captura de pantalla que muestra cómo habilitar la eliminación temporal de contenedores en Azure Portal":::

# <a name="template"></a>[Plantilla](#tab/template)

Para habilitar la eliminación temporal de contenedores con una plantilla Azure Resource Manager, cree una plantilla que establezca la propiedad **containerDeleteRetentionPolicy**. En los siguientes pasos se muestra cómo crear una plantilla en Azure Portal.

1. En Azure Portal, elija **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
1. Elija **Implementación de plantillas**, seleccione **Crear** y, luego, **Cree su propia plantilla en el editor**.
1. En el editor de plantillas, pegue el código JSON siguiente. Reemplace el marcador de posición `<account-name>` por el nombre de la cuenta de almacenamiento.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "variables": {},
      "resources": [
          {
              "type": "Microsoft.Storage/storageAccounts/blobServices",
              "apiVersion": "2019-06-01",
              "name": "<account-name>/default",
              "properties": {
                  "containerDeleteRetentionPolicy": {
                      "enabled": true,
                      "days": 7
                  }
              }
          }
      ]
    }
    ```

---

1. Especifique el período de retención. El valor predeterminado es 7.
1. Guarde la plantilla.
1. Especifique el grupo de recursos de la cuenta y, luego, elija el botón **Revisar y crear** para implementar la plantilla y habilitar la eliminación temporal de contenedores.

## <a name="view-soft-deleted-containers"></a>Visualización de contenedores eliminados temporalmente

Cuando la eliminación temporal está habilitada, puede ver los contenedores eliminados temporalmente en Azure Portal. Los contenedores eliminados temporalmente son visibles durante el período de retención especificado. Una vez expirado el período de retención, un contenedor eliminado temporalmente se elimina permanentemente y deja de estar visible.

Para ver los contenedores eliminados temporalmente en Azure Portal, siga estos pasos:

1. Vaya a su cuenta de almacenamiento en Azure Portal y acceda a la lista de contenedores.
1. Alterne el conmutador Show deleted containers (Mostrar contenedores eliminados) para incluir los contenedores eliminados en la lista.

    :::image type="content" source="media/soft-delete-container-enable/soft-delete-container-portal-list.png" alt-text="Captura de pantalla que muestra cómo ver los contenedores eliminados temporalmente en Azure Portal":::

## <a name="restore-a-soft-deleted-container"></a>Restauración de un contenedor eliminado temporalmente

Puede restaurar un contenedor eliminado temporalmente y su contenido dentro del período de retención. Para restaurar un contenedor eliminado temporalmente en Azure Portal, siga estos pasos:

1. Vaya a su cuenta de almacenamiento en Azure Portal y acceda a la lista de contenedores.
1. Muestre el menú contextual del contenedor que desea eliminar y elija **Recuperar** en el menú.

    :::image type="content" source="media/soft-delete-container-enable/soft-delete-container-portal-restore.png" alt-text="Captura de pantalla que muestra cómo restaurar un contenedor eliminado temporalmente en Azure Portal":::

## <a name="next-steps"></a>Pasos siguientes

- [Eliminación temporal para contenedores (versión preliminar)](soft-delete-container-overview.md)
- [Eliminación temporal para blobs](soft-delete-blob-overview.md)
- [Control de versiones de blobs](versioning-overview.md)