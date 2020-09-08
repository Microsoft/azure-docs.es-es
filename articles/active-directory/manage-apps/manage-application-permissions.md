---
title: 'Administración de permisos de usuario y de administrador: Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo revisar y administrar los permisos de una aplicación en Azure AD. Por ejemplo, revoque todos los permisos concedidos a una aplicación.
services: active-directory
author: mimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 7/10/2020
ms.author: mimart
ms.reviewer: luleonpla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 95e13cedc0cdbaedc8c00b9d855057da7e631c19
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510885"
---
# <a name="take-action-on-overprivileged-or-suspicious-applications-in-azure-active-directory"></a>Adopción de medidas ante una aplicación con privilegios excesivos o sospechosa en Azure Active Directory

Obtenga información sobre cómo revisar y administrar los permisos de una aplicación. En este artículo se proporcionarán diferentes acciones que puede llevar a cabo para proteger una aplicación en función del escenario. Estas acciones se pueden realizar en todas las aplicaciones que se han agregado a su inquilino de Azure Active Directory (Azure AD) mediante el consentimiento del usuario o del administrador.

Para más información acerca de cómo dar consentimiento a las aplicaciones, consulte [Marco de consentimiento de Azure Active Directory](../develop/consent-framework.md).

## <a name="prerequisites"></a>Requisitos previos

Para realizar las siguientes acciones, debe iniciar sesión como administrador global, administrador de la aplicación o administrador de aplicaciones en la nube.

Para restringir el acceso a las aplicaciones, debe requerir la asignación de usuarios y, luego, asignar usuarios o grupos a la aplicación.  Para más información, consulte [Métodos para asignar usuarios y grupos](methods-for-assigning-users-and-groups.md).

Puede acceder al portal de Azure AD para obtener scripts de PowerShell contextuales para realizar las acciones.
 
1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
3. Seleccione la aplicación a la que desea restringir el acceso.
4. Seleccione **Permisos**. En la barra de comandos, seleccione **Revisar permisos**.

![Captura de pantalla de la ventana de revisión de permisos.](./media/manage-application-permissions/review-permissions.png)


## <a name="control-access-to-an-application"></a>Control del acceso a una aplicación

Para restringir el acceso a esta aplicación, se recomienda activar la opción **Asignación de usuario**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
3. Seleccione la aplicación a la que desea restringir el acceso.
4. Seleccione **Propiedades** y, después, establezca **User requirement required** (Requisito de usuario obligatorio) en **Sí**.
5. Seleccione **Usuarios y grupos** y, a continuación, quite los usuarios no deseados asignados a la aplicación.
6. Asigne usuarios o grupos a la aplicación.

De manera opcional, puede quitar todos los usuarios asignados a la aplicación mediante PowerShell.

## <a name="revoke-all-permissions-for-an-application"></a>Revocación de todos los permisos de una aplicación

Mediante el script de PowerShell, puede revocar todos los permisos concedidos a esta aplicación.

> [!NOTE]
> La revocación del permiso actual concedido no impedirá que los usuarios vuelvan a tener consentimiento en la aplicación. Si desea bloquear el consentimiento de los usuarios, lea [Configuración del consentimiento de los usuarios finales a las aplicaciones](configure-user-consent.md).

De manera opcional, puede deshabilitar la aplicación para impedir que los usuarios accedan a la aplicación y que la aplicación acceda a los datos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
3. Seleccione la aplicación a la que desea restringir el acceso.
4. Seleccione **Propiedades** y, después, establezca **¿Habilitado para que los usuarios inicien sesión?** en **No**.

## <a name="investigate-a-suspicious-application"></a>Investigación de una aplicación sospechosa

Para restringir el acceso a esta aplicación, se recomienda activar la opción **Asignación de usuario**. A continuación, revise los permisos que los usuarios y administradores hayan concedido a la aplicación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
3. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
5. Seleccione la aplicación a la que desea restringir el acceso.
6. Seleccione **Propiedades** y, después, establezca **User requirement required** (Requisito de usuario obligatorio) en **Sí**.
7. Seleccione **Permisos** y revise los permisos concedidos a administradores y usuarios.

Opcionalmente, mediante PowerShell, puede:

- Quitar a todos los usuarios asignados para impedir que inicien sesión en la aplicación.
- Invalidar los tokens de actualización para los usuarios que tienen acceso a la aplicación.
- Revocar todos los permisos de una aplicación.

O bien, puede deshabilitar la aplicación para bloquear el acceso de los usuarios y detener el acceso de las aplicaciones a los datos.


## <a name="disable-a-malicious-application"></a>Deshabilitar una aplicación malintencionada 

Se recomienda deshabilitar la aplicación para impedir que los usuarios accedan a la aplicación y que la aplicación tenga acceso a los datos. Si, en su lugar, elimina la aplicación, los usuarios pueden volver a dar su consentimiento a la aplicación y conceder acceso a los datos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
3. Seleccione la aplicación a la que desea restringir el acceso.
4. Seleccione **Propiedades** y después copie el identificador del objeto.

### <a name="powershell-commands"></a>Comandos de PowerShell


Recupere el identificador de objeto de la entidad de servicio.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales**.
3. Seleccione la aplicación a la que desea restringir el acceso.
4. Seleccione **Propiedades** y después copie el identificador del objeto.

```powershell
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $sp.ObjectId
```
Quite todos los usuarios asignados a la aplicación.
 ```powershell
    Connect-AzureAD

    # Get Service Principal using objectId
    $sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

    # Get Azure AD App role assignments using objectId of the Service Principal
    $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -All $true

    # Remove all users and groups assigned to the application
    $assignments | ForEach-Object {
        if ($_.PrincipalType -eq "User") {
            Remove-AzureADUserAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.ObjectId
        } elseif ($_.PrincipalType -eq "Group") {
            Remove-AzureADGroupAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.ObjectId
        }
    }
 ```

Revoque los permisos concedidos a la aplicación.

```powershell
    Connect-AzureAD

    # Get Service Principal using objectId
    $sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

    # Get all delegated permissions for the service principal
    $spOAuth2PermissionsGrants = Get-AzureADOAuth2PermissionGrant -All $true| Where-Object { $_.clientId -eq $sp.ObjectId }

    # Remove all delegated permissions
    $spOAuth2PermissionsGrants | ForEach-Object {
        Remove-AzureADOAuth2PermissionGrant -ObjectId $_.ObjectId
    }

    # Get all application permissions for the service principal
    $spApplicationPermissions = Get-AzureADServiceAppRoleAssignedTo -ObjectId $sp.ObjectId -All $true | Where-Object { $_.PrincipalType -eq "ServicePrincipal" }

    # Remove all delegated permissions
    $spApplicationPermissions | ForEach-Object {
        Remove-AzureADServiceAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.objectId
    }
```
Invalide los tokens de actualización.
```powershell
        Connect-AzureAD

        # Get Service Principal using objectId
        $sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

        # Get Azure AD App role assignments using objectID of the Service Principal
        $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -All $true | Where-Object {$_.PrincipalType -eq "User"}

        # Revoke refresh token for all users assigned to the application
        $assignments | ForEach-Object {
            Revoke-AzureADUserAllRefreshToken -ObjectId $_.PrincipalId
        }
```
## <a name="next-steps"></a>Pasos siguientes
- [Administración del consentimiento a las aplicaciones y evaluación de las solicitudes de consentimiento](manage-consent-requests.md)
- [Configuración del consentimiento del usuario](configure-user-consent.md)
- [Configuración del flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md)
