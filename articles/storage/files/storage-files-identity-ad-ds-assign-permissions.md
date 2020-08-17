---
title: Control del acceso a los recursos compartidos de archivos de Azure - Autenticación local de AD DS
description: Obtenga información sobre cómo asignar permisos a una identidad de Active Directory Domain Services que representa la cuenta de almacenamiento. Esto le permite controlar el acceso con la autenticación basada en identidades.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: 16c8058da30821a53a20cf3ea6afdb0e4dbfcb77
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87535100"
---
# <a name="part-two-assign-share-level-permissions-to-an-identity"></a>Parte 2: Asignación de permisos de nivel de recurso compartido a una identidad

Antes de comenzar este artículo, asegúrese de que ha completado el artículo anterior [Habilitación de la autenticación de AD DS para la cuenta](storage-files-identity-ad-ds-enable.md).

Una vez habilitada la autenticación de Active Directory Domain Services (AD DS) en la cuenta de almacenamiento, debe configurar los permisos de nivel de recurso compartido para poder acceder a los recursos compartidos de archivos. La identidad que quiera usar para acceder a los recursos del recurso compartido de archivos de Azure debe ser una identidad híbrida que exista tanto en AD DS como en Azure AD. Por ejemplo, supongamos que tiene un usuario en AD DS que es user1@onprem.contoso.com y que se ha sincronizado con Azure AD como user1@contoso.com mediante la sincronización con Azure AD Connect. Para permitir que este usuario tenga acceso a Azure Files, debe asignar permisos de nivel de recurso compartido a user1@contoso.com. El mismo concepto se aplica a los grupos o entidades de servicio. Por este motivo, debe sincronizar los usuarios y grupos de su instancia de AD DS con Azure AD mediante la sincronización de Azure AD Connect. 

Los permisos de nivel de recurso compartido deben asignarse a la identidad de Azure AD que representa el mismo usuario o grupo en AD DS para admitir la autenticación de AD DS en el recurso compartido de archivos de Azure. La autenticación y autorización con identidades que solo existen en Azure AD, como las identidades administradas de Azure (MSI), no se admiten con la autenticación de AD DS. En este artículo se muestra cómo asignar permisos de nivel de recurso compartido para un recurso compartido de archivos a una identidad.


## <a name="share-level-permissions"></a>Asignación de permisos de nivel de recurso compartido

En general, es recomendable usar permisos de nivel de recurso compartido para la administración del acceso de alto nivel para un grupo de Azure AD que representa un grupo de usuarios e identidades y, después, usar ACL de Windows para el control de acceso pormenorizado en el nivel de directorio o archivo. 

Existen tres roles integrados de Azure para conceder permisos de nivel de recurso compartido a los usuarios:

- El **lector de recursos compartidos de SMB para los datos del archivo de almacenamiento** permite el acceso de lectura en los recursos compartidos de archivos de Azure Storage a través de SMB.
- El **colaborador de recursos compartidos de SMB para los datos del archivo de almacenamiento** permite el acceso de lectura, escritura y eliminación en los recursos compartidos de archivos de Azure Storage a través de SMB.
- El **Colaborador con privilegios elevados de recursos compartidos de SMB de datos de archivos de almacenamiento** permite el acceso de lectura, escritura, eliminación y modificación de ACL de Windows en los recursos compartidos de archivos de Azure Storage mediante SMB.

> [!IMPORTANT]
> El control administrativo total de un recurso compartido de archivos, incluida la capacidad de tomar posición de un archivo, requiere usar la clave de la cuenta de almacenamiento. El control administrativo no puede realizarse con credenciales de Azure AD.

Puede usar Azure Portal, Azure PowerShell o la CLI de Azure para asignar los roles integrados a la identidad de Azure AD de un usuario a fin de conceder permisos de nivel de recurso compartido.

## <a name="assign-an-azure-role"></a>Asignación de un rol de Azure

### <a name="azure-portal"></a>Azure portal

Para asignar un rol de Azure a una identidad de Azure AD mediante [Azure Portal](https://portal.azure.com), siga estos pasos:

1. En Azure Portal, vaya al recurso compartido de archivos o [cree uno](storage-how-to-create-file-share.md).
1. Seleccione **Access Control (IAM)** .
1. Seleccione **Agregar una asignación de roles**.
1. En la hoja **Agregar asignación de roles**, seleccione el rol integrado adecuado (lector o colaborador de recursos compartidos de SMB para datos del archivo de almacenamiento) en la lista desplegable **Rol**. Mantenga la opción **Asignar acceso a** en la configuración predeterminada: **Usuario, grupo o entidad de servicio de Azure AD**. Seleccione la identidad de Azure AD de destino por nombre o dirección de correo electrónico. La identidad de Azure AD seleccionada debe ser una identidad híbrida y no puede ser una identidad solo en la nube. Esto significa que la misma identidad también se representa en AD DS.
1. Seleccione **Guardar** para completar la operación de asignación de roles.

### <a name="powershell"></a>PowerShell

El siguiente ejemplo de PowerShell muestra cómo asignar un rol de Azure a una identidad de Azure AD, según el nombre de inicio de sesión. Para obtener más información sobre la asignación de roles de Azure con PowerShell, consulte [Administración del acceso mediante RBAC y Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

Antes de ejecutar el siguiente script de ejemplo, reemplace los valores de marcador de posición, incluidos los corchetes, por los suyos.

```powershell
#Get the name of the custom role
$FileShareContributorRole = Get-AzRoleDefinition "<role-name>" #Use one of the built-in roles: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
#Constrain the scope to the target file share
$scope = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
#Assign the custom role to the target identity with the specified scope.
New-AzRoleAssignment -SignInName <user-principal-name> -RoleDefinitionName $FileShareContributorRole.Name -Scope $scope
```

### <a name="cli"></a>CLI
  
El siguiente comando de la CLI 2.0 asigna un rol de Azure a una identidad de Azure AD, según el nombre de inicio de sesión. Para obtener más información sobre la asignación de roles de Azure con la CLI de Azure, consulte [Administración del acceso mediante RBAC y la CLI de Azure](../../role-based-access-control/role-assignments-cli.md). 

Antes de ejecutar el siguiente script de ejemplo, no olvide reemplazar los valores de marcador de posición, incluidos los corchetes, por los suyos propios.

```azurecli-interactive
#Assign the built-in role to the target identity: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
az role assignment create --role "<role-name>" --assignee <user-principal-name> --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha asignado los permisos de nivel de recurso compartido, debe configurar los permisos de nivel de archivo y de directorio. Continúe con el artículo siguiente.

[Parte tres: Configuración de permisos de nivel de directorio y de archivo en SMB](storage-files-identity-ad-ds-configure-permissions.md)