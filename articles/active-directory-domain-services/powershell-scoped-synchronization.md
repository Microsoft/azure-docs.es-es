---
title: Sincronización con ámbito mediante PowerShell para Azure AD Domain Services | Microsoft Docs
description: Aprenda a usar PowerShell de Azure AD a fin de configurar la sincronización con ámbito entre Azure AD y un dominio administrado de Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/24/2020
ms.author: iainfou
ms.openlocfilehash: 197ae37b0c63b19ebe4dcdf2732169be0f357a07
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293833"
---
# <a name="configure-scoped-synchronization-from-azure-ad-to-azure-active-directory-domain-services-using-azure-ad-powershell"></a>Configuración de la sincronización con ámbito entre Azure AD y Azure Active Directory Domain Services con PowerShell de Azure AD

Para proporcionar servicios de autenticación, Azure Active Directory Domain Services (Azure AD DS) sincroniza los usuarios y grupos de Azure AD. En un entorno híbrido, los usuarios y grupos de un entorno local de Active Directory Domain Services (AD DS) se pueden sincronizar primero con Azure AD mediante Azure AD Connect y, luego, con Azure AD DS.

De forma predeterminada, todos los usuarios y grupos de un directorio de Azure AD se sincronizan con un dominio administrado de Azure AD DS. Si tiene necesidades específicas, puede optar por sincronizar solo un conjunto definido de usuarios.

En este artículo se muestra cómo crear un dominio administrado que use la sincronización con ámbito y, luego, cambiar o deshabilitar el conjunto de usuarios con ámbito mediante PowerShell de Azure AD. También puede [completar estos pasos con Azure Portal][scoped-sync].

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, complete el tutorial para [crear y configurar un dominio administrado de Azure Active Directory Domain Services][tutorial-create-instance].
* Necesita privilegios de *administrador global* en el inquilino de Azure AD para cambiar el ámbito de sincronización de Azure AD DS.

## <a name="scoped-synchronization-overview"></a>Introducción a la sincronización con ámbito

De forma predeterminada, todos los usuarios y grupos de un directorio de Azure AD se sincronizan con un dominio administrado. Si solo unos cuantos usuarios necesitan acceso al dominio administrado, puede sincronizar únicamente esas cuentas de usuario. Esta sincronización son ámbito se basa en grupos. Al configurar la sincronización con ámbito basada en grupos, solo las cuentas de usuario que pertenecen a los grupos que especifique se sincronizan con el dominio administrado. Los grupos anidados no están sincronizados, solo los grupos específicos que seleccione.

Puede cambiar el ámbito de sincronización al crear el dominio administrado o una vez implementado. También puede cambiar el ámbito de sincronización en un dominio administrado existente sin necesidad de volver a crearlo.

Para más información sobre el proceso de sincronización , consulte [Funcionamiento de la sincronización en Azure AD Domain Services][concepts-sync].

> [!WARNING]
> Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Se aplican las siguientes consideraciones:
>
>  * Al cambiar el ámbito de sincronización de un dominio administrado, se produce una resincronización completa.
>  * Los objetos que ya no son necesarios en el dominio administrado se eliminan. Se crean objetos en el dominio administrado.

## <a name="powershell-script-for-scoped-synchronization"></a>Script de PowerShell para la sincronización con ámbito

Para configurar la sincronización de ámbito mediante PowerShell, primero guarde el siguiente script en un archivo llamado `Select-GroupsToSync.ps1`.

Este script configura Azure AD DS para sincronizar los grupos seleccionados de Azure AD. Todas las cuentas de usuario que forman parte de los grupos especificados se sincronizan con el dominio administrado.

Este script se usa en los pasos adicionales de este artículo.

```powershell
param (
    [Parameter(Position = 0)]
    [String[]]$groupsToAdd
)

Connect-AzureAD
$sp = Get-AzureADServicePrincipal -Filter "AppId eq '2565bd9d-da50-47d4-8b85-4c97f669dc36'"
$role = $sp.AppRoles | where-object -FilterScript {$_.DisplayName -eq "User"}

Write-Output "`n****************************************************************************"

Write-Output "Total group-assignments need to be added: $($groupsToAdd.Count)"
$newGroupIds = New-Object 'System.Collections.Generic.HashSet[string]'
foreach ($groupName in $groupsToAdd)
{
    try
    {
        $group = Get-AzureADGroup -Filter "DisplayName eq '$groupName'"
        $newGroupIds.Add($group.ObjectId)

        Write-Output "Group-Name: $groupName, Id: $($group.ObjectId)"
    }
    catch
    {
        Write-Error "Failed to find group: $groupName. Exception: $($_.Exception)."
    }
}

Write-Output "****************************************************************************`n"
Write-Output "`n****************************************************************************"

$currentAssignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId
Write-Output "Total current group-assignments: $($currentAssignments.Count), SP-ObjectId: $($sp.ObjectId)"

$currAssignedObjectIds = New-Object 'System.Collections.Generic.HashSet[string]'
foreach ($assignment in $currentAssignments)
{
    Write-Output "Assignment-ObjectId: $($assignment.PrincipalId)"

    if ($newGroupIds.Contains($assignment.PrincipalId) -eq $false)
    {
        Write-Output "This assignment is not needed anymore. Removing it! Assignment-ObjectId: $($assignment.PrincipalId)"
        Remove-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -AppRoleAssignmentId $assignment.ObjectId
    }
    else
    {
        $currAssignedObjectIds.Add($assignment.PrincipalId)
    }
}

Write-Output "****************************************************************************`n"
Write-Output "`n****************************************************************************"

foreach ($id in $newGroupIds)
{
    try
    {
        if ($currAssignedObjectIds.Contains($id) -eq $false)
        {
            Write-Output "Adding new group-assignment. Role-Id: $($role.Id), Group-Object-Id: $id, ResourceId: $($sp.ObjectId)"
            New-AzureADGroupAppRoleAssignment -Id $role.Id -ObjectId $id -PrincipalId $id -ResourceId $sp.ObjectId
        }
        else
        {
            Write-Output "Group-ObjectId: $id is already assigned."
        }
    }
    catch
    {
        Write-Error "Exception occurred assigning Object-ID: $id. Exception: $($_.Exception)."
    }
}

Write-Output "****************************************************************************`n"
```

## <a name="enable-scoped-synchronization"></a>Habilitación de la sincronización con ámbito

Para habilitar la sincronización con ámbito basada en grupos para un dominio administrado, lleve a cabo los pasos siguientes:

1. En primer lugar, establezca *"filteredSync" = "Enabled"* en el recurso de Azure AD DS y, a continuación, actualice el dominio administrado.

    Cuando se le solicite, especifique las credenciales de un *administrador global* para iniciar sesión en su inquilino de Azure AD con el cmdlet [Connect-AzureAD][Connect-AzureAD]:

    ```powershell
    // Connect to your Azure AD tenant
    Connect-AzureAD

    // Retrieve the Azure AD DS resource.
    $DomainServicesResource = Get-AzResource -ResourceType "Microsoft.AAD/DomainServices"

    // Enable group-based scoped synchronization.
    $enableScopedSync = @{"filteredSync" = "Enabled"}

    // Update the Azure AD DS resource
    Set-AzResource -Id $DomainServicesResource.ResourceId -Properties $enableScopedSync
    ```

1. Ahora, especifique la lista de grupos cuyos usuarios se deben sincronizar con el dominio administrado.

    Ejecute el script `Select-GroupsToSync.ps1` y especifique la lista de grupos que se van a sincronizar. En el ejemplo siguiente, los grupos que se van a sincronizar son *GroupName1* y *GroupName2*.

    > [!WARNING]
    > Debe incluir el grupo *Administradores de controladores de dominio de AAD* en la lista de grupos para la sincronización con ámbito. Si no incluye este grupo, no se puede usar el dominio administrado.

    ```powershell
    .\Select-GroupsToSync.ps1 -groupsToAdd @("AAD DC Administrators", "GroupName1", "GroupName2")
    ```

Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Los objetos que ya no son necesarios en el dominio administrado se eliminan y la resincronización puede tardar mucho tiempo en completarse.

## <a name="modify-scoped-synchronization"></a>Modificación de la sincronización con ámbito

Para modificar la lista de grupos cuyos usuarios se deben sincronizar con el dominio administrado, ejecute el script `Select-GroupsToSync.ps1` y especifique la nueva lista de grupos que se van a sincronizar.

En el ejemplo siguiente, los grupos que se van a sincronizar ya no incluyen *GroupName2*, ahora incluyen *GroupName3*.

> [!WARNING]
> Debe incluir el grupo *Administradores de controladores de dominio de AAD* en la lista de grupos para la sincronización con ámbito. Si no incluye este grupo, no se puede usar el dominio administrado.

Cuando se le solicite, especifique las credenciales de un *administrador global* para iniciar sesión en su inquilino de Azure AD con el cmdlet [Connect-AzureAD][Connect-AzureAD]:

```powershell
.\Select-GroupsToSync.ps1 -groupsToAdd @("AAD DC Administrators", "GroupName1", "GroupName3")
```

Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Los objetos que ya no son necesarios en el dominio administrado se eliminan y la resincronización puede tardar mucho tiempo en completarse.

## <a name="disable-scoped-synchronization"></a>Deshabilitación de la sincronización con ámbito

Para deshabilitar la sincronización con ámbito basada en grupos para un dominio administrado, establezca *"filteredSync" = "Disabled"* en el recurso de Azure AD DS y, luego, actualice el dominio administrado. Cuando finalice, todos los usuarios y grupos están establecidos para sincronizarse desde Azure AD.

Cuando se le solicite, especifique las credenciales de un *administrador global* para iniciar sesión en su inquilino de Azure AD con el cmdlet [Connect-AzureAD][Connect-AzureAD]:

```powershell
// Connect to your Azure AD tenant
Connect-AzureAD

// Retrieve the Azure AD DS resource.
$DomainServicesResource = Get-AzResource -ResourceType "Microsoft.AAD/DomainServices"

// Disable group-based scoped synchronization.
$disableScopedSync = @{"filteredSync" = "Disabled"}

// Update the Azure AD DS resource
Set-AzResource -Id $DomainServicesResource.ResourceId -Properties $disableScopedSync
```

Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Los objetos que ya no son necesarios en el dominio administrado se eliminan y la resincronización puede tardar mucho tiempo en completarse.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el proceso de sincronización , consulte [Funcionamiento de la sincronización en Azure AD Domain Services](synchronization.md).

<!-- INTERNAL LINKS -->
[scoped-sync]: scoped-synchronization.md
[concepts-sync]: synchronization.md
[tutorial-create-instance]: tutorial-create-instance.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md

<!-- EXTERNAL LINKS -->
[Connect-AzureAD]: /powershell/module/azuread/connect-azuread
