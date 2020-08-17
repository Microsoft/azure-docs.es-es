---
title: Protección de la entrega de webhooks con Azure AD en Azure Event Grid
description: Describe cómo enviar eventos a puntos de conexión HTTPS protegidos por Azure Active Directory mediante Azure Event Grid
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 90d06f203bc93177101a87a7a774d816b11b16f6
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460718"
---
# <a name="publish-events-to-azure-active-directory-protected-endpoints"></a>Publicación de eventos en puntos de conexión protegidos por Azure Active Directory

En este artículo se describe cómo aprovechar las ventajas de Azure Active Directory para proteger la conexión entre su suscripción de eventos y el punto de conexión de webhook. Para una introducción a las entidades de servicio y aplicaciones de Azure AD, consulte [Introducción a la Plataforma de identidad de Microsoft (versión 2.0)](../active-directory/develop/v2-overview.md).

En este artículo se usa Azure Portal para la demostración; sin embargo, la característica también se puede habilitar mediante la CLI, PowerShell o los SDK.

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="create-an-azure-ad-application"></a>Creación de una aplicación de Azure AD

Empiece por crear una aplicación de Azure AD para el punto de conexión protegido. Vea https://docs.microsoft.com/azure/active-directory/develop/scenario-protected-web-api-overview.
    - Configure la API protegida para que la llame una aplicación de demonio.
    
## <a name="enable-event-grid-to-use-your-azure-ad-application"></a>Habilitación de Event Grid para que use la aplicación de Azure AD

Use el siguiente script de PowerShell para crear un rol y una entidad de servicio en la aplicación Azure AD. Necesitará el identificador de inquilino y el identificador de objeto de la aplicación Azure AD:

   > [!NOTE]
   > Debe ser miembro del [rol Administrador de aplicaciones de Azure AD](../active-directory/users-groups-roles/directory-assign-admin-roles.md#available-roles) para ejecutar este script.
    
1. Modifique el valor de $myTenantId del script de PowerShell para usar el identificador de inquilino de Azure AD.
1. Modifique el valor de $myAzureADApplicationObjectId del script de PowerShell para usar el identificador de objeto de Azure AD.
1. Ejecute el script modificado.

```PowerShell
# This is your Tenant Id. 
$myTenantId = "<the Tenant Id of your Azure AD Application>"

Connect-AzureAD -TenantId $myTenantId
    
# This is your Azure AD Application's ObjectId. 
$myAzureADApplicationObjectId = "<the Object Id of your Azure AD Application>"
    
# This is the "Azure Event Grid" Azure Active Directory AppId
$eventGridAppId = "4962773b-9cdb-44cf-a8bf-237846a00ab7"
    
# This is the name of the new role we will add to your Azure AD Application
$eventGridRoleName = "AzureEventGridSecureWebhook"
    
# Create an application role of given name and description
Function CreateAppRole([string] $Name, [string] $Description)
{
    $appRole = New-Object Microsoft.Open.AzureAD.Model.AppRole
    $appRole.AllowedMemberTypes = New-Object System.Collections.Generic.List[string]
    $appRole.AllowedMemberTypes.Add("Application");
    $appRole.DisplayName = $Name
    $appRole.Id = New-Guid
    $appRole.IsEnabled = $true
    $appRole.Description = $Description
    $appRole.Value = $Name;
    return $appRole
}
    
# Get my Azure AD Application, it's roles and service principal
$myApp = Get-AzureADApplication -ObjectId $myAzureADApplicationObjectId
$myAppRoles = $myApp.AppRoles
$eventGridSP = Get-AzureADServicePrincipal -Filter ("appId eq '" + $eventGridAppId + "'")

Write-Host "App Roles before addition of new role.."
Write-Host $myAppRoles
    
# Create the role if it doesn't exist
if ($myAppRoles -match $eventGridRoleName)
{
    Write-Host "The Azure Event Grid role is already defined.`n"
}
else
{
    $myServicePrincipal = Get-AzureADServicePrincipal -Filter ("appId eq '" + $myApp.AppId + "'")
    
    # Add our new role to the Azure AD Application
    $newRole = CreateAppRole -Name $eventGridRoleName -Description "Azure Event Grid Role"
    $myAppRoles.Add($newRole)
    Set-AzureADApplication -ObjectId $myApp.ObjectId -AppRoles $myAppRoles
}
    
# Create the service principal if it doesn't exist
if ($eventGridSP -match "Microsoft.EventGrid")
{
    Write-Host "The Service principal is already defined.`n"
}
else
{
    # Create a service principal for the "Azure Event Grid" Azure AD Application and add it to the role
    $eventGridSP = New-AzureADServicePrincipal -AppId $eventGridAppId
}
    
New-AzureADServiceAppRoleAssignment -Id $myApp.AppRoles[0].Id -ResourceId $myServicePrincipal.ObjectId -ObjectId $eventGridSP.ObjectId -PrincipalId $eventGridSP.ObjectId
    
Write-Host "My Azure AD Tenant Id: $myTenantId"
Write-Host "My Azure AD Application Id: $($myApp.AppId)"
Write-Host "My Azure AD Application ObjectId: $($myApp.ObjectId)"
Write-Host "My Azure AD Application's Roles: "
Write-Host $myApp.AppRoles
```
    
## <a name="configure-the-event-subscription"></a>Configuración de la suscripción de eventos

En el flujo de creación de la suscripción de eventos, seleccione el tipo de punto de conexión "Webhook". Una vez proporcionado el identificador URI del punto de conexión, haga clic en la pestaña Características adicionales en la parte superior de la hoja Crear suscripciones de eventos.

![Selección del tipo de punto de conexión webhook](./media/secure-webhook-delivery/select-webhook.png)

En la pestaña Características adicionales, active la casilla "Usar autenticación de AAD" y configure el identificador de inquilino y el identificador de aplicación:

* Copie el identificador de inquilino de Azure AD de la salida del script y escríbalo en el campo Identificador de inquilino de AAD.
* Copie el identificador de aplicación de Azure AD de la salida del script y escríbalo en el campo Identificador de aplicación de AAD.

    ![Acción de webhook seguro](./media/secure-webhook-delivery/aad-configuration.png)

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la supervisión de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para más información sobre la clave de autenticación, vea [Seguridad y autenticación de Event Grid](security-authentication.md).
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
