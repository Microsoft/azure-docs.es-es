---
title: Guía de control de versiones de blobs del Explorador de Azure Storage | Microsoft Docs
description: Guía de control de versiones de blobs del Explorador de Azure Storage
services: storage
author: chuye
ms.service: storage
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: chuye
ms.openlocfilehash: d318983cec1365b71d14731395e71528621659d6
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051879"
---
# <a name="azure-storage-explorer-blob-versioning-guide"></a>Guía de control de versiones de blobs del Explorador de Azure Storage

El Explorador de Microsoft Azure Storage facilita el acceso y la administración de las versiones de los blobs. Esta guía le ayudará a comprender cómo funciona el control de versiones de blobs en el Explorador de Storage. Antes de continuar, se recomienda que lea más sobre el [control de versiones de blobs](https://docs.microsoft.com/azure/storage/blobs/versioning-overview).

## <a name="terminology"></a>Terminología

En esta sección se proporcionan algunas definiciones para ayudarle a comprender su uso en este artículo.

- Eliminación temporal: característica de protección de datos automática alternativa. Puede obtener más información sobre la eliminación temporal [aquí](https://docs.microsoft.com/azure/storage/blobs/soft-delete-blob-overview).
- Blob activo: un blob o versión de blob se crea con un estado activo. Solo puede trabajar en los blobs o versiones de blob que están en estado activo.
- Blob eliminado temporalmente: una versión de blob o un blob marcado como eliminado temporalmente. Los blobs eliminados temporalmente solo se conservan durante su período de retención.
- Versión de blob: un blob que se ha creado con el control de versiones de blobs habilitado. Cada versión de blob está asociada a un id. de versión.
- Versión actual: una versión de blob marcada como la versión actual.
- Versión anterior: una versión de blob que no es la versión actual.
- Blob sin control de versiones: un blob que se ha creado con el control de versiones de blobs deshabilitado. un blob sin control de versiones no tiene un id. de versión.

## <a name="view-blob-versions"></a>Visualización de las versiones de blob

El Explorador de Storage admite cuatro vistas diferentes para ver los blobs.

| Ver | Blobs sin control de versiones activos | Blobs sin control de versiones eliminados temporalmente | Versiones de blob |
| ---- | :----------: | :-----------: | :------------------: |
| Blobs activos | Sí | No | Solo la versión actual |
| Blobs activos y blobs eliminados temporalmente | Sí | Sí | Solo la versión actual |
| Blobs activos y blobs sin una versión actual | Sí | No | Versión actual o última versión activa |
| Todos los blobs y blobs sin una versión actual | Sí | Sí | Versión actual o última versión |

### <a name="active-blobs"></a>Blobs activos

En esta vista, el Explorador de Azure Storage muestra:

- Blobs sin control de versiones activos
- Versiones actuales

### <a name="active-blobs-and-soft-deleted-blobs"></a>Blobs activos y blobs eliminados temporalmente

En esta vista, el Explorador de Azure Storage muestra:

- Blobs sin control de versiones activos
- Blobs sin control de versiones eliminados temporalmente
- Versiones actuales

### <a name="active-blobs-and-blobs-without-current-version"></a>Blobs activos y blobs sin una versión actual

En esta vista, el Explorador de Azure Storage muestra:

- Blobs sin control de versiones activos
- Versiones actuales
- Versiones anteriores activas más recientes 

En el caso de los blobs que no tienen una versión actual, pero sí una versión anterior activa, el Explorador de Azure Storage muestra la versión anterior activa más reciente como una representación de ese blob.

### <a name="all-blobs-and-blobs-without-current-version"></a>Todos los blobs y blobs sin una versión actual

En esta vista, el Explorador de Azure Storage muestra:

- Blobs sin control de versiones activos
- Blobs sin control de versiones eliminados temporalmente
- Versiones actuales
- Versiones anteriores más recientes 

En el caso de los blobs que no tienen una versión actual, el Explorador de Azure Storage muestra la versión anterior más reciente como una representación de ese blob.

> [!Note]
> Debido a una limitación del servicio, el Explorador de Azure Storage necesita realizar un procesamiento adicional para obtener una vista jerárquica de los directorios virtuales al enumerar las versiones de blobs. Tomará más tiempo enumerar los blobs en las siguientes vistas:
> 
> - Blobs activos y blobs sin una versión actual
> - Todos los blobs y blobs sin una versión actual

## <a name="manage-blob-versions"></a>Administración de las versiones de blob

### <a name="view-versions-of-a-blob"></a>Visualización de las versiones de un blob

El Explorador de Azure Storage proporciona un comando para **Administrar versiones** para ver todas las versiones de un blob. Para ver las versiones de un blob, seleccione el blob cuyas versiones quiere ver y seleccione **Administrar el historial &rarr; Administrar versiones** en la barra de herramientas o en el menú contextual.

### <a name="download-blob-versions"></a>Descarga de versiones de blob

Para descargar una o más versiones de un blob, seleccione las versiones del blob que quiere descargar y seleccione **Descargar** en la barra de herramientas o en el menú contextual.

Si va a descargar varias versiones de un blob, los archivos descargados tendrán el id. de versión al principio del nombres de archivo.

### <a name="delete-blob-versions"></a>Eliminación de las versiones del blob

Para eliminar una o más versiones de un blob, seleccione las versiones del blob que quiere eliminar y seleccione **Eliminar** en la barra de herramientas o en el menú contextual.

Las versiones de blob están sujetas a la directiva de eliminación temporal. Si está habilitada la eliminación temporal, las versiones de blob se eliminarán de manera temporal. La eliminación de una versión actual es un caso especial. Al eliminar una versión actual, esta se convertirá automáticamente en una versión anterior activa.

### <a name="promote-blob-version"></a>Promoción de una versión de blob

Puede restaurar el contenido de un blob al promover una versión anterior para que se convierta en la versión actual. Seleccione la versión de blob que quiera promocionar y seleccione **Promote Version** (Promover versión) en la barra de herramientas o en el menú contextual.

La versión de blob promovida sobrescribirá a los blobs sin control de versiones. Asegúrese de que ya no necesita los datos o realice una copia de seguridad de los mismos antes de confirmar la operación. Las versiones actuales se convierten automáticamente en versiones anteriores, por lo que el Explorador de Azure Storage no le pedirá su confirmación.

### <a name="undelete-blob-version"></a>Recuperación de las versiones de blob

Las versiones de blob no se pueden recuperar individualmente. Deben recuperarse todas a la vez. Para recuperar todas las versiones de blob de un blob, seleccione cualquiera de ellas y seleccione **Undelete Selected** (Recuperar selección) en la barra de herramientas o en el menú contextual.

### <a name="change-access-tier-of-blob-versions"></a>Cambio de nivel de acceso en las versiones de blob

Cada versión de blob tiene su propio nivel de acceso. Para cambiar el nivel de acceso de las versiones de blob, seleccione aquellas cuyo nivel de acceso quiera cambiar y seleccione **Cambiar nivel de acceso...** en el menú contextual.

## <a name="see-also"></a>Consulte también

* [Control de versiones de blobs](https://docs.microsoft.com/azure/storage/blobs/versioning-overview)
* [Eliminación temporal para blobs](https://docs.microsoft.com/azure/storage/blobs/soft-delete-blob-overview)
