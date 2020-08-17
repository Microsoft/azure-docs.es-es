---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/06/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: bc5d4e5a8a9607f74eb8733855a216b83520ff11
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503722"
---
Azure proporciona los siguientes roles integrados para autorizar el acceso a datos de blob y cola con Azure AD y OAuth:

- [Propietario de datos de Storage Blob](../articles/role-based-access-control/built-in-roles.md#storage-blob-data-owner): se usa para establecer la propiedad y administrar el control de acceso POSIX en Azure Data Lake Storage Gen2. Para más información, consulte [Control de acceso en Azure Data Lake Storage Gen2](../articles/storage/blobs/data-lake-storage-access-control.md).
- [Colaborador de datos de Storage Blob](../articles/role-based-access-control/built-in-roles.md#storage-blob-data-contributor): se usa para conceder permisos de lectura, escritura y eliminación a los recursos de almacenamiento de blobs.
- [Lector de datos de Storage Blob](../articles/role-based-access-control/built-in-roles.md#storage-blob-data-reader): se usa para conceder permisos de solo lectura a los recursos de almacenamiento de blobs.
- [Delegador de Blob Storage](../articles/role-based-access-control/built-in-roles.md#storage-blob-delegator): obtiene una clave de delegación de usuarios que se usa para crear una firma de acceso compartido que se firma con credenciales de Azure AD para un contenedor o un blob.
- [Colaborador de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-contributor): se usa para conceder permisos de lectura, escritura y eliminación a las colas de Azure.
- [Lector de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-reader): se usa para conceder permisos de solo lectura a las colas de Azure.
- [Procesador de mensajes de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-message-processor): se usa para conceder permisos de inspección, recuperación y eliminación a los mensajes de las colas de Azure Storage.
- [Emisor de mensajes de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-message-sender): se usa para conceder permisos de adición a los mensajes de las colas de Azure Storage.

Solo los roles definidos explícitamente para el acceso a datos permiten a una entidad de seguridad acceder a los datos de blobs o colas. Los roles integrados, como **Propietario**, **Colaborador** y **Colaborador de la cuenta de almacenamiento**, permiten que una entidad de seguridad administre una cuenta de almacenamiento, pero no proporcionan acceso a los datos de blobs o colas dentro de esa cuenta a través de Azure AD. Sin embargo, si un rol incluye **Microsoft.Storage/storageAccounts/listKeys/action**, el usuario al que se haya asignado ese rol podrá acceder a los datos de la cuenta de almacenamiento mediante la autorización de clave compartida con las claves de acceso de la cuenta. Para más información, consulte [Usar Azure Portal para tener acceso a datos de blob o de cola](../articles/storage/common/storage-access-blobs-queues-portal.md).

Para obtener información detallada sobre los roles integrados de Azure para Azure Storage para los servicios de datos y el servicio de administración, consulte la sección **Almacenamiento** en [Roles integrados de Azure para RBAC de Azure](../articles/role-based-access-control/built-in-roles.md#storage). Además, para obtener información sobre los diferentes tipos de roles que proporcionan permisos en Azure, consulte [Roles de administrador de suscripciones clásico, roles de Azure y roles de Azure AD](../articles/role-based-access-control/rbac-and-directory-admin-roles.md).

> [!IMPORTANT]
> Las asignaciones de roles de Azure pueden tardar hasta cinco minutos en propagarse.
