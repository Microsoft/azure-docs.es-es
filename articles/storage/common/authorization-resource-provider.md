---
title: Uso del proveedor de recursos de Azure Storage para acceder a los recursos de administración
description: El proveedor de recursos de Azure Storage es un servicio que proporciona acceso a los recursos de administración de Azure Storage. Se puede usar para crear, actualizar, administrar y eliminar recursos como cuentas de almacenamiento, puntos de conexión privados y claves de acceso de cuentas.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 12/12/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 3d0ef8a8641c3814fa7c9964786a7f24f5e54a01
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534947"
---
# <a name="use-the-azure-storage-resource-provider-to-access-management-resources"></a>Uso del proveedor de recursos de Azure Storage para acceder a los recursos de administración

Azure Resource Manager es el servicio de implementación y administración para Azure. El proveedor de recursos de Azure Storage es un servicio que se basa en Azure Resource Manager y que proporciona acceso a los recursos de administración de Azure Storage. Se puede usar para crear, actualizar, administrar y eliminar recursos como cuentas de almacenamiento, puntos de conexión privados y claves de acceso de cuentas. Para más información sobre Azure Resource Manager, consulte [Introducción a Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview).

El proveedor de recursos de Azure Storage se puede usar para realizar acciones tales como crear o eliminar una cuenta de almacenamiento u obtener una lista de cuentas de almacenamiento en una suscripción. Para autorizar las solicitudes que se realicen en el proveedor de recursos de Azure Storage, utilice Azure Active Directory (Azure AD). En este artículo se describe cómo asignar permisos a los recursos de administración y se proporcionan ejemplos en los que se muestran cómo realizar solicitudes en el proveedor de recursos de Azure Storage.

## <a name="management-resources-versus-data-resources"></a>Recursos de administración frente a recursos de datos

Microsoft proporciona dos API REST para trabajar con recursos de Azure Storage. Estas API forman la base de todas las acciones que se pueden realizar en Azure Storage. La API REST de Azure Storage permite trabajar con los datos de una cuenta de almacenamiento, lo que incluye datos de blobs, colas, archivos y tablas. La API REST del proveedor de recursos de Azure Storage permite trabajar tanto con la cuenta de almacenamiento como con los recursos relacionados.

Una solicitud que lee o escribe datos de blobs requiere permisos diferentes que los de una que realiza una operación de administración. RBAC proporciona un control específico de los permisos de ambos tipos de recursos. Cuando asigne un rol de Azure a una entidad de seguridad, asegúrese de que sabe qué permisos va a conceder dicha entidad. Para obtener una referencia detallada que describa las acciones asociadas a cada rol integrado de Azure, consulte [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

Azure Storage admite el uso de Azure AD para autorizar solicitudes en Blob Storage y Queue Storage. Para más información sobre los roles de Azure para las operaciones de datos de blobs y colas, consulte [Autorización del acceso a blobs y colas mediante Active Directory](storage-auth-aad.md).

## <a name="assign-management-permissions-with-role-based-access-control-rbac"></a>Asignación de permisos de administración con el control de acceso basado en rol (RBAC)

Cada suscripción de Azure tiene una instancia de Azure Active Directory asociada que administra usuarios, grupos y aplicaciones. A los usuarios, grupos o aplicaciones también se los denomina como entidad de seguridad en el contexto de la [plataforma de identidad de Microsoft](/azure/active-directory/develop/). Mediante el control de acceso basado en rol (RBAC) es posible conceder a los recursos de una suscripción acceso a una entidad de seguridad que se defina en Active Directory.

Cuando se asigna un rol de Azure a una entidad de seguridad, también se indica el ámbito en el que están en vigor los permisos que concede el rol. En el caso de las operaciones de administración, puede asignar un rol en el nivel de la suscripción, el grupo de recursos o la cuenta de almacenamiento. Para asignar un rol de Azure a una entidad de seguridad, utilice [Azure Portal](https://portal.azure.com/), las [herramientas de la CLI de Azure](../../cli-install-nodejs.md), [PowerShell](/powershell/azure/) o la [API REST del proveedor de recursos de Azure Storage](/rest/api/storagerp).

Para obtener más información, consulte [¿Qué es el control de acceso basado en roles (RBAC) de Azure?](../../role-based-access-control/overview.md) y [Roles de administrador de suscripciones clásico de Azure y roles de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

### <a name="built-in-roles-for-management-operations"></a>Roles integrados para las operaciones de administración

Azure proporciona roles integrados que conceden permisos para llamar a las operaciones de administración. Azure Storage también proporciona roles integrados que se usan específicamente con el proveedor de recursos de Azure Storage.

Los roles integrados que conceden permisos para llamar a las operaciones de administración del almacenamiento incluyen los que se describen en la siguiente tabla:

|    Rol de Azure    |    Descripción    |    ¿Incluye acceso a las claves de las cuentas?    |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| **Propietario** | Puede administrar todos los recursos de almacenamiento y acceder a los recursos.  | Si, proporciona permisos para ver y volver a generar las claves de la cuenta de almacenamiento. |
| **Colaborador**  | Puede administrar todos los recursos de almacenamiento, pero no puede administrar la asignación a los recursos. | Si, proporciona permisos para ver y volver a generar las claves de la cuenta de almacenamiento. |
| **Lector** | Puede ver información acerca de la cuenta de almacenamiento, pero no puede ver las claves de la cuenta. | No. |
| **Colaborador de la cuenta de almacenamiento** | Puede administrar la cuenta de almacenamiento, obtener información acerca de los grupos de recursos y recursos de la suscripción, y crear y administrar implementaciones de los grupos de recursos de la suscripción. | Si, proporciona permisos para ver y volver a generar las claves de la cuenta de almacenamiento. |
| **Administrador de acceso de usuario** | Puede administrar el acceso a la cuenta de almacenamiento.   | Sí, permite a una entidad de seguridad asignar cualquier permiso tanto a ella misma como a otras. |
| **Colaborador de la máquina virtual** | Puede administrar máquinas virtuales, pero no la cuenta de almacenamiento a la que están conectadas.   | Si, proporciona permisos para ver y volver a generar las claves de la cuenta de almacenamiento. |

La tercera columna de la tabla indica si el rol integrado admite **Microsoft.Storage/storageAccounts/listkeys/action**. Esta acción concede los permisos necesarios para leer y volver a generar las claves de la cuenta de almacenamiento. Los permisos para acceder a los recursos de administración de Azure Storage no incluyen los permisos para acceder a los datos. Sin embargo, si un usuario tiene acceso a las claves de la cuenta, puede usarlas para acceder a los datos de Azure Storage mediante la autorización de claves compartidas.

### <a name="custom-roles-for-management-operations"></a>Roles personalizados para las operaciones de administración

Azure también admite la definición de roles personalizados de Azure para el acceso a los recursos de administración. Para más información sobre los roles personalizados, consulte [Roles personalizados de Azure](../../role-based-access-control/custom-roles.md).

## <a name="code-samples"></a>Ejemplos de código

A continuación, encontrará ejemplos de código en los que se muestra cómo autorizar y llamar operaciones de administración desde las bibliotecas de administración de Azure Storage:

- [.NET](https://github.com/Azure-Samples/storage-dotnet-resource-provider-getting-started)
- [Java](https://github.com/Azure-Samples/storage-java-manage-storage-accounts)
- [Node.js](https://github.com/Azure-Samples/storage-node-resource-provider-getting-started)
- [Python](https://github.com/Azure-Samples/storage-python-manage)

## <a name="azure-resource-manager-versus-classic-deployments"></a>Implementaciones clásicas frentes a las realizadas mediante Azure Resource Manager

El modelo de implementación de Resource Manager y el modelo de implementación clásica representan dos formas diferentes de implementar y administrar las soluciones de Azure. Microsoft recomienda usar el modelo de implementación de Azure Resource Manager a la hora de crear cuentas de almacenamiento. Si es posible, también recomienda volver a crear con el modelo de Resource Manager aquellas cuentas de almacenamiento existentes en las que se usó el modelo de implementación clásica. Aunque se pueden crear cuentas de almacenamiento mediante el modelo de implementación clásica, este es menos flexible y tarde o temprano estará en desuso.

Para más información sobre los modelos de implementación de Azure, consulte el artículo acerca de la [implementación clásica y con Resource Manager](../../azure-resource-manager/management/deployment-models.md).

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview)
- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md)
- [Objetivos de escalabilidad para el proveedor de recursos de Azure Storage](scalability-targets-resource-provider.md)
