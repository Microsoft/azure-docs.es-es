---
title: 'Enumeración de asignaciones de roles de Azure mediante Azure PowerShell: RBAC de Azure'
description: Obtenga información sobre cómo determinar a qué recursos tienen acceso los usuarios, grupos, entidades de servicio e identidades administradas mediante Azure PowerShell y el control de acceso basado en roles (RBAC) de Azure (RBAC de Azure).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 9e225dba-9044-4b13-b573-2f30d77925a9
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/28/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: e27fe0589498de13f5eb6e17f8869bb9d7352a09
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372083"
---
# <a name="list-azure-role-assignments-using-azure-powershell"></a>Enumeración de asignaciones de roles de Azure mediante Azure PowerShell

[!INCLUDE [Azure RBAC definition list access](../../includes/role-based-access-control-definition-list.md)] En este artículo se describe cómo enumerar las asignaciones de roles mediante Azure PowerShell.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

> [!NOTE]
> Si su organización ha externalizado funciones de administración a un proveedor de servicios que usa la [administración de recursos delegados de Azure](../lighthouse/concepts/azure-delegated-resource-management.md), las asignaciones de roles autorizadas por ese proveedor de servicios no se mostrarán aquí.

## <a name="prerequisites"></a>Prerrequisitos

- [PowerShell en Azure Cloud Shell](/azure/cloud-shell/overview) o [Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="list-role-assignments-for-the-current-subscription"></a>Enumeración de asignaciones de roles de la subscripción actual

La forma más fácil de obtener una lista de todas las asignaciones de roles de la suscripción actual (incluidas las asignaciones de roles heredadas de los grupos raíz y de administración) es usar [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) sin ningún parámetro.

```azurepowershell
Get-AzRoleAssignment
```

```Example
PS C:\> Get-AzRoleAssignment

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
DisplayName        : Alain
SignInName         : alain@example.com
RoleDefinitionName : Storage Blob Data Reader
RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Marketing
SignInName         :
RoleDefinitionName : Contributor
RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
ObjectId           : 22222222-2222-2222-2222-222222222222
ObjectType         : Group
CanDelegate        : False

...
```

## <a name="list-role-assignments-for-a-subscription"></a>Enumeración de asignaciones de roles de una subscripción

Para enumerar todas las asignaciones de roles en un ámbito de suscripción, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment). Para obtener el identificador de suscripción, lo puede encontrar en la hoja **Suscripciones** de Azure Portal o bien puede usar [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription).

```azurepowershell
Get-AzRoleAssignment -Scope /subscriptions/<subscription_id>
```

```Example
PS C:\> Get-AzRoleAssignment -Scope /subscriptions/00000000-0000-0000-0000-000000000000
```

## <a name="list-role-assignments-for-a-user"></a>Lista de las asignaciones de rol de un usuario

Para enumerar todos los roles que están asignados a un usuario específico, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -SignInName <email_or_userprincipalname>
```

```Example
PS C:\> Get-AzRoleAssignment -SignInName isabella@example.com | FL DisplayName, RoleDefinitionName, Scope

DisplayName        : Isabella Simonsen
RoleDefinitionName : BizTalk Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
```

Para enumerar todos los roles asignados a un usuario especificado y los roles que se asignan a los grupos a los que pertenece el usuario, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -SignInName <email_or_userprincipalname> -ExpandPrincipalGroups
```

```Example
Get-AzRoleAssignment -SignInName isabella@example.com -ExpandPrincipalGroups | FL DisplayName, RoleDefinitionName, Scope
```

## <a name="list-role-assignments-for-a-resource-group"></a>Lista de las asignaciones de roles de un grupo de recursos

Para enumerar todas las asignaciones de roles en un ámbito de grupo de recursos, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -ResourceGroupName <resource_group_name>
```

```Example
PS C:\> Get-AzRoleAssignment -ResourceGroupName pharma-sales | FL DisplayName, RoleDefinitionName, Scope

DisplayName        : Alain Charon
RoleDefinitionName : Backup Operator
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales

DisplayName        : Isabella Simonsen
RoleDefinitionName : BizTalk Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales

DisplayName        : Alain Charon
RoleDefinitionName : Virtual Machine Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
```

## <a name="list-role-assignments-for-a-management-group"></a>Lista de asignaciones de rol para un grupo de administración

Para enumerar todas las asignaciones de roles en un ámbito de grupo de administración, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment). Para obtener el identificador del grupo de administración, puede encontrarlo en la hoja **Grupos de administración** de Azure Portal o usar [Get-AzManagementGroup](/powershell/module/az.resources/get-azmanagementgroup).

```azurepowershell
Get-AzRoleAssignment -Scope /providers/Microsoft.Management/managementGroups/<group_id>
```

```Example
PS C:\> Get-AzRoleAssignment -Scope /providers/Microsoft.Management/managementGroups/marketing-group
```

## <a name="list-role-assignments-for-a-resource"></a>Lista de las asignaciones de roles de un recurso

Para enumerar las asignaciones de roles de un recurso específico, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) y el parámetro `-Scope`. El ámbito será diferente según el recurso. Para obtener el ámbito, puede ejecutar `Get-AzRoleAssignment` sin parámetros para mostrar todas las asignaciones de roles y, a continuación, buscar el ámbito que quiere enumerar.

```azurepowershell
Get-AzRoleAssignment -Scope "/subscriptions/<subscription_id>/resourcegroups/<resource_group_name>/providers/<provider_name>/<resource_type>/<resource>
```

En el ejemplo siguiente se muestra cómo enumerar las asignaciones de roles para una cuenta de almacenamiento. Tenga en cuenta que este comando también enumera las asignaciones de roles en ámbitos más altos, como los grupos de recursos y las suscripciones, que se aplican a esta cuenta de almacenamiento.

```Example
PS C:\> Get-AzRoleAssignment -Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/storage-test-rg/providers/Microsoft.Storage/storageAccounts/storagetest0122"
```

Si quiere simplemente mostrar las asignaciones de roles que se asignan directamente en un recurso, puede usar el comando [Where-Object](/powershell/module/microsoft.powershell.core/where-object) para filtrar la lista.

```Example
PS C:\> Get-AzRoleAssignment | Where-Object {$_.Scope -eq "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/storage-test-rg/providers/Microsoft.Storage/storageAccounts/storagetest0122"}
```

## <a name="list-role-assignments-for-classic-service-administrator-and-co-administrators"></a>Enumeración de asignaciones de roles para el administrador y los coadministradores del servicio clásico

Para enumerar las asignaciones de roles para el administrador y los coadministradores de una suscripción clásica, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment -IncludeClassicAdministrators
```

## <a name="list-role-assignments-for-a-managed-identity"></a>Lista de asignaciones de roles para una identidad administrada

1. Obtenga el identificador de objeto de la identidad administrada asignada por el usuario o el sistema. 

    Para obtener el identificador de objeto de una identidad administrada asignada por el usuario, puede usar [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal).

    ```azurepowershell
    Get-AzADServicePrincipal -DisplayNameBeginsWith "<name> or <vmname>"
    ```

1. Para enumerar las asignaciones de roles, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

    ```azurepowershell
    Get-AzRoleAssignment -ObjectId <objectid>
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Incorporación o eliminación de asignaciones de roles de Azure con Azure PowerShell](role-assignments-powershell.md)
