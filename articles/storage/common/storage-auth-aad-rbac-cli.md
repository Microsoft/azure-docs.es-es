---
title: Uso de la CLI de Azure para asignar un rol de RBAC para el acceso a datos
titleSuffix: Azure Storage
description: Aprenda a usar la CLI de Azure para asignar permisos a una entidad de seguridad de Azure Active Directory con el control de acceso basado en rol (RBAC). Azure Storage admite roles integrados y personalizados de Azure para la autenticación mediante Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/16/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: afa48c07e628eaeb013c24fa38fcda1adea2555a
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87368986"
---
# <a name="use-azure-cli-to-assign-an-rbac-role-for-access-to-blob-and-queue-data"></a>Uso de la CLI de Azure para asignar un rol de RBAC para el acceso a datos de blobs y colas

Azure Active Directory (Azure AD) autoriza derechos de acceso a los recursos protegidos mediante el [control de acceso basado en rol (RBAC)](../../role-based-access-control/overview.md). Azure Storage define un conjunto de roles integrados de Azure que engloban los conjuntos comunes de permisos que se usan para acceder a los datos de los blobs o de las colas.

Cuando un rol RBAC se asigna a una entidad de seguridad de Azure AD, Azure concede acceso a esos recursos a esa entidad de seguridad. El acceso se puede limitar al nivel de la suscripción, el grupo de recursos, la cuenta de almacenamiento o un contenedor individual o una cola. Una entidad de seguridad de Azure AD puede ser un usuario, un grupo, una entidad de servicio de aplicación o una [identidad de servicio administrada para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

En este artículo se describe cómo usar la CLI de Azure para enumerar los roles integrados de Azure y asignarlos a los usuarios. Para obtener más información sobre cómo usar la CLI de Azure, vea [Interfaz de la línea de comandos (CLI) de Azure](/cli/azure).

## <a name="rbac-roles-for-blobs-and-queues"></a>Roles RBAC para blobs y colas

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>Determinar el ámbito de recursos

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="list-available-rbac-roles"></a>Mostrar los roles RBAC disponibles

Para enumerar los roles integrados de Azure con la CLI de Azure, use el comando [az role definition list](/cli/azure/role/definition#az-role-definition-list):

```azurecli-interactive
az role definition list --out table
```

Aparecerá una lista con los roles de datos de Azure Storage integrados, así como otros roles integrados de Azure:

```Example
Storage Blob Data Contributor             Allows for read, write and delete access to Azure Storage blob containers and data
Storage Blob Data Owner                   Allows for full access to Azure Storage blob containers and data, including assigning POSIX access control.
Storage Blob Data Reader                  Allows for read access to Azure Storage blob containers and data
Storage Queue Data Contributor            Allows for read, write, and delete access to Azure Storage queues and queue messages
Storage Queue Data Message Processor      Allows for peek, receive, and delete access to Azure Storage queue messages
Storage Queue Data Message Sender         Allows for sending of Azure Storage queue messages
Storage Queue Data Reader                 Allows for read access to Azure Storage queues and queue messages
```

## <a name="assign-an-rbac-role-to-a-security-principal"></a>Asignación de un rol RBAC a una entidad de seguridad

Para asignar un rol RBAC a una entidad de seguridad, use el comando [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create). El formato del comando puede variar en función del ámbito de la asignación. Los ejemplos siguientes muestran cómo asignar un rol a un usuario en varios ámbitos, pero puede usar el mismo comando para asignar un rol a cualquier entidad de seguridad.

### <a name="container-scope"></a>Ámbito de contenedor

Para asignar un rol cuyo ámbito es un contenedor, especifique una cadena que contenga el ámbito del contenedor en el parámetro `--scope`. El ámbito de un contenedor tiene este formato:

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>
```

En el siguiente ejemplo se asigna el rol **Colaborador de datos de Storage Blob** a un usuario y el ámbito se establece en un nivel de contenedor. Asegúrese de reemplazar los valores de ejemplo y los valores de marcador de posición entre corchetes angulares por los suyos propios:

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>"
```

### <a name="queue-scope"></a>Ámbito de cola

Para asignar un rol cuyo ámbito es una cola, especifique una cadena que contenga el ámbito de la cola en el parámetro `--scope`. El ámbito de una cola tiene este formato:

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue>
```

En el siguiente ejemplo se asigna el rol **Colaborador de datos de Storage Queue** a un usuario y el ámbito se establece en un nivel de cola. Asegúrese de reemplazar los valores de ejemplo y los valores de marcador de posición entre corchetes angulares por los suyos propios:

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue>"
```

### <a name="storage-account-scope"></a>Ámbito de cuenta de almacenamiento

Para asignar un rol cuyo ámbito es la cuenta de almacenamiento, especifique el ámbito del recurso de la cuenta de almacenamiento en el parámetro `--scope`. El ámbito de una cuenta de almacenamiento tiene este formato:

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

En el siguiente ejemplo se muestra cómo asignar el rol **Lector de datos de Storage Blob** a un usuario en el nivel de la cuenta de almacenamiento. Asegúrese de reemplazar los valores de ejemplo por sus propios valores: \

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

### <a name="resource-group-scope"></a>Ámbito de grupo de recursos

Para asignar un rol cuyo ámbito es el grupo de recursos, especifique el identificador o el nombre del grupo de recursos en el parámetro `--resource-group`. En el siguiente ejemplo se asigna el rol **Lector de datos de la cola de Storage Blob** a un usuario en el nivel del grupo de recursos. Asegúrese de reemplazar los valores de ejemplo y los valores de marcador de posición entre corchetes angulares por los suyos propios:

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Reader" \
    --assignee <email> \
    --resource-group <resource-group>
```

### <a name="subscription-scope"></a>Ámbito de suscripción

Para asignar un rol cuyo ámbito es la suscripción, especifique el ámbito de la suscripción en el parámetro `--scope`. El ámbito de una suscripción tiene este formato:

```
/subscriptions/<subscription>
```

En el siguiente ejemplo se muestra cómo asignar el rol **Lector de datos de Storage Blob** a un usuario en el nivel de la cuenta de almacenamiento. Asegúrese de reemplazar los valores de ejemplo por sus propios valores: 

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>"
```

## <a name="next-steps"></a>Pasos siguientes

- [Administración del acceso a los recursos de Azure mediante RBAC y Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Conceder acceso a datos de blob y de cola de Azure con RBAC mediante Azure PowerShell](storage-auth-aad-rbac-powershell.md)
- [Conceder acceso a datos de blob y cola de Azure con RBAC en Azure Portal](storage-auth-aad-rbac-portal.md)
