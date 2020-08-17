---
title: Autorización de operaciones de datos
titleSuffix: Azure Storage
description: Más información sobre las distintas maneras de autorizar el acceso al Azure Storage, incluido Azure Active Directory, la autorización con clave compartida o las firmas de acceso compartido (SAS).
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: a56fdf8bc3bfa93c12cce66664e68b223fa97ff2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029542"
---
# <a name="authorizing-access-to-data-in-azure-storage"></a>Autorización del acceso a datos en Azure Storage

Cada vez que accede a datos de la cuenta de almacenamiento, el cliente realiza una solicitud a través de HTTP/HTTPS a Azure Storage. Todas las solicitudes a un recurso seguro deben estar autorizadas para que el servicio garantice que el cliente tiene los permisos necesarios para acceder a los datos.

En la tabla siguiente se describen las opciones que ofrece Azure Storage para autorizar el acceso a los recursos:

| Artefacto de Azure | Clave compartida (clave de cuenta de almacenamiento) | Firma de acceso compartido (SAS) | Azure Active Directory (Azure AD) | Active Directory Domain Services local (versión preliminar) | Acceso de lectura anónimo |
| -------------- | -------------------------------- | ----------------------------- | --------------------------------- | ------------------------------------------------------ | ---------------------------- |
|Azure Blobs     |[Compatible](/rest/api/storageservices/authorize-with-shared-key/)         |[Compatible](storage-sas-overview.md)         |[Compatible](storage-auth-aad.md)         |No compatible|[Compatible](../blobs/storage-manage-access-to-resources.md)         |
|Azure Files (SMB)     |[Compatible](/rest/api/storageservices/authorize-with-shared-key/)         |No compatible         |[Admitido, solo con AAD Domain Services](../files/storage-files-active-directory-overview.md)         |[Admitido, las credenciales deben sincronizarse con Azure AD](../files/storage-files-active-directory-overview.md)|No compatible         |
|Azure Files (REST)     |[Compatible](/rest/api/storageservices/authorize-with-shared-key/)         |[Compatible](storage-sas-overview.md)         |No compatible         |No compatible |No compatibles         |
|Colas de Azure     |[Compatible](/rest/api/storageservices/authorize-with-shared-key/)         |[Compatible](storage-sas-overview.md)         |[Compatible](storage-auth-aad.md)         |No compatible | No compatible         |
|Azure Tables     |[Compatible](/rest/api/storageservices/authorize-with-shared-key/)         |[Compatible](storage-sas-overview.md)         |No compatible         |No compatible| No compatible         |

Cada opción de autorización se describe brevemente a continuación:

- **Integración de Azure Active Directory (Azure AD)** para blobs y colas. Azure AD proporciona control de acceso basado en rol (RBAC) para controlar el acceso de los clientes a los recursos de una cuenta de almacenamiento. Para más información sobre la integración de Azure AD en blobs y colas, consulte [Autorización del acceso a blobs y colas con Azure Active Directory](storage-auth-aad.md).

- **Autenticación de Azure Active Directory Domain Services (Azure DS)** para Azure Files. Azure Files admite la autorización basada en identidad sobre Bloque de mensajes del servidor(SMB) mediante Azure AD DS. Puede usar RBAC para el control específico de acceso de los clientes a los recursos de Azure Files en una cuenta de almacenamiento. Para más información acerca de la autenticación de Azure Files mediante servicios de dominio, consulte la [introducción](../files/storage-files-active-directory-overview.md).

- **Autenticación de Active Directory Domain Services (AD DS) local (AD DS local) (versión preliminar)** para Azure Files. Azure Files admite la autorización basada en identidad sobre SMB mediante AD DS. El entorno de AD DS se puede hospedar en máquinas locales o en VM de Azure. El acceso de SMB a Files se admite mediante el uso de las credenciales de AD DS de las máquinas unidas a un dominio, independientemente de que sea local o en Azure. Puede usar una combinación de RBAC para el control de acceso a nivel de recurso compartido y listas de control de acceso discrecional de NTFS para el cumplimiento de los permisos a nivel de archivo/directorio. Para más información acerca de la autenticación de Azure Files mediante servicios de dominio, consulte la [introducción](../files/storage-files-active-directory-overview.md).

- **Autorización con clave compartida** para blobs, archivos, colas y tablas. Los clientes con clave compartida pasan un encabezado con cada solicitud que está firmado con la clave de acceso de la cuenta de almacenamiento. Para más información, consulte el artículo sobre la [Autorización con clave compartida](/rest/api/storageservices/authorize-with-shared-key/).
- **Firmas de acceso compartido** para blobs, archivos, colas y tablas. Las firmas de acceso compartido (SAS) proporcionan acceso delegado limitado a recursos de una cuenta de almacenamiento. Agregar restricciones al periodo de validez de la firma o a los permisos que concede proporciona flexibilidad para administrar el acceso. Para obtener más información, consulte [Uso de firmas de acceso compartido (SAS)](storage-sas-overview.md).
- **Acceso de lectura público y anónimo** para contenedores y blobs. No se necesita autorización. Para más información, consulte [Administración del acceso de lectura anónimo a contenedores y blobs](../blobs/storage-manage-access-to-resources.md).  

De forma predeterminada, todos los recursos de Azure Storage están protegidos y solo están disponibles para el propietario de la cuenta. Aunque puede usar cualquiera de las estrategias de autorización descritas anteriormente para conceder a los clientes acceso a los recursos de su cuenta de almacenamiento, Microsoft recomienda el uso de Azure AD siempre que sea posible para mayor seguridad y facilidad de uso.

## <a name="next-steps"></a>Pasos siguientes

- [Autorización del acceso a blobs y colas de Azure con Azure Active Directory](storage-auth-aad.md)
- [Autorización con clave compartida](/rest/api/storageservices/authorize-with-shared-key/)
- [Grant limited access to Azure Storage resources using shared access signatures (SAS)](storage-sas-overview.md) (Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido [SAS])
