---
title: Restricción de los permisos de acceso de usuarios invitados en Azure Active Directory | Microsoft Docs
description: Restricción de los permisos de acceso de usuarios invitados mediante Azure Portal, PowerShell o Microsoft Graph en Azure Active Directory
services: active-directory
author: curtand
ms.author: curtand
manager: daveba
ms.date: 08/20/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: krbain
ms.collection: M365-identity-device-management
ms.openlocfilehash: 28125ff55fe6ab3e68d56dc26a074d0498c2b413
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798438"
---
# <a name="restrict-guest-access-permissions-preview-in-azure-active-directory"></a>Restricción de los permisos de acceso de invitados (versión preliminar) en Azure Active Directory

Azure Active Directory (Azure AD) le permite restringir qué usuarios invitados externos se pueden ver en su organización de Azure AD. De forma predeterminada, los usuarios invitados se establecen en un nivel de permisos limitado en Azure AD, mientras que el valor predeterminado para los usuarios miembros es el conjunto completo de permisos de usuario predeterminados. Se trata de la versión preliminar de un nuevo nivel de permisos de usuarios invitados en la configuración de colaboración externa de la organización de Azure AD para un acceso aún más restringido, por lo que las opciones de acceso de invitado ahora son:

Nivel de permiso         | Nivel de acceso
----------------         | ------------
Igual que los usuarios miembros     | Los invitados tienen el mismo acceso a los recursos de Azure AD que los usuarios miembros
Acceso limitado (predeterminado) | Los invitados pueden ver la pertenencia de todos los grupos no ocultos
**Acceso restringido (nuevo)**  | **Los invitados no pueden ver la pertenencia de ningún grupo**

Cuando el acceso de invitado está restringido, los invitados solo pueden ver su propio perfil de usuario. No se admite el permiso para ver a otros usuarios aunque el invitado busque por nombre principal de usuario o identificador de objeto. El acceso restringido también impide que los usuarios vean la pertenencia de los grupos de los que forman parte. Para más información sobre los permisos generales de usuario predeterminados, incluidos los permisos de usuarios invitados, vea [¿Cuales son los permisos de usuario predeterminados en Azure Active Directory?](../fundamentals/users-default-permissions.md)

## <a name="permissions-and-licenses"></a>Permisos y licencias

Debe tener en el rol de administrador global para configurar las opciones de colaboración externa. No existen requisitos de licencia adicionales para restringir el acceso de invitado.

## <a name="update-in-the-azure-portal"></a>Actualización en Azure Portal

Se han introducido cambios en los controles de Azure Portal existentes para los permisos de usuario invitado.

> [!IMPORTANT]
> Por poco tiempo, estos nuevos controles del portal de los permisos de usuario invitado serán visibles solo con la dirección URL [https://aka.ms/AADRestrictedGuestAccess](https://aka.ms/AADRestrictedGuestAccess). Aún es posible utilizar PowerShell y Microsoft Graph para definir los controles, y los cambios se respetarán en el portal.

1. Inicie sesión en el [Centro de administración de Azure AD](https://aka.ms/AADRestrictedGuestAccess) con permisos de administrador global.
1. En la página de información general de **Azure Active Directory** de su organización, seleccione **Configuración de usuario**.
1. En **Usuarios externos**, seleccione **Administrar la configuración de colaboración externa**.
1. En la página **Configuración de colaboración externa**, seleccione la opción **El acceso de los usuarios invitados está restringido a las propiedades y pertenencias de sus propios objetos de directorio**.

    ![Página de la configuración de colaboración externa de Azure AD](./media/users-restrict-guest-permissions/external-collaboration-settings.png)

1. Seleccione **Guardar**. Los cambios pueden tardar hasta 15 minutos en surtir efecto para los usuarios invitados.

## <a name="update-with-the-microsoft-graph-api"></a>Actualización con Microsoft Graph API

Se ha agregado una nueva Microsoft Graph API para configurar los permisos de invitado en la organización de Azure AD. Se pueden realizar las siguientes llamadas API para asignar cualquier nivel de permiso. El valor de guestUserRoleId que se usa aquí es para ilustrar la configuración de usuario invitado más restringida. Para más información sobre el uso de Microsoft Graph para establecer permisos de invitado, vea [Tipo de recurso authorizationPolicy](/graph/api/resources/authorizationpolicy).

### <a name="configuring-for-the-first-time"></a>Configuración por primera vez

````PowerShell
POST https://graph.microsoft.com/beta/policies/authorizationPolicy/authorizationPolicy

{
  "guestUserRoleId": "2af84b1e-32c8-42b7-82bc-daa82404023b"
}
````

La respuesta debe ser Correcto 204.

### <a name="updating-the-existing-value"></a>Actualización del valor existente

````PowerShell
PATCH https://graph.microsoft.com/beta/policies/authorizationPolicy/authorizationPolicy

{
  "guestUserRoleId": "2af84b1e-32c8-42b7-82bc-daa82404023b"
}
````

La respuesta debe ser Correcto 204.

### <a name="view-the-current-value"></a>Visualización del valor actual

````PowerShell
GET https://graph.microsoft.com/beta/policies/authorizationPolicy/authorizationPolicy
````

Respuesta de ejemplo:

````PowerShell
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#policies/authorizationPolicy/$entity",
    "id": "authorizationPolicy",
    "displayName": "Authorization Policy",
    "description": "Used to manage authorization related settings across the company.",
    "enabledPreviewFeatures": [],
    "guestUserRoleId": "10dae51f-b6af-4016-8d66-8c2a99b929b3",
    "permissionGrantPolicyIdsAssignedToDefaultUserRole": [
        "user-default-legacy"
    ]
}
````

## <a name="update-with-powershell-cmdlets"></a>Actualización con los cmdlets de PowerShell

Con esta característica, hemos agregado la capacidad de configurar los permisos restringidos mediante cmdlets de PowerShell v2. Obtenga y defina los cmdlets de PowerShell publicados en la versión 2.0.2.85.

### <a name="get-command-get-azureadmsauthorizationpolicy"></a>Comando Get: Get-AzureADMSAuthorizationPolicy

Ejemplo:

````PowerShell
PS C:\WINDOWS\system32> Get-AzureADMSAuthorizationPolicy

Id                                                : authorizationPolicy
OdataType                                         :
Description                                       : Used to manage authorization related settings across the company.
DisplayName                                       : Authorization Policy
EnabledPreviewFeatures                            : {}
GuestUserRoleId                                   : 10dae51f-b6af-4016-8d66-8c2a99b929b3
PermissionGrantPolicyIdsAssignedToDefaultUserRole : {user-default-legacy}
````

### <a name="set-command-set-azureadmsauthorizationpolicy"></a>Comando Set: Set-AzureADMSAuthorizationPolicy

Ejemplo:

````PowerShell
PS C:\WINDOWS\system32> Set-AzureADMSAuthorizationPolicy -GuestUserRoleId '2af84b1e-32c8-42b7-82bc-daa82404023b'
````

> [!NOTE]
> Debe escribir authorizationPolicy como el identificador cuando se solicite.

## <a name="supported-microsoft-365-services"></a>Servicios de Microsoft 365 admitidos

### <a name="supported-services"></a>Servicios admitidos

Con "admitidos", se hace referencia a que la experiencia es la prevista, en particular que es la misma que la experiencia de invitado actual.

- Teams
- Outlook (OWA)
- SharePoint

### <a name="services-currently-not-supported"></a>Servicios no admitidos actualmente

Los servicios no admitidos actualmente pueden tener problemas de compatibilidad con la nueva configuración de la restricción de invitados.

- Formularios
- Planner en Teams
- Aplicación Planner
- Project
- Yammer

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

Pregunta | Respuesta
-------- | ------
¿Dónde se aplican estos permisos? | Estos permisos a nivel de directorio se aplican a través de portales y servicios de Azure AD, incluidos Microsoft Graph, PowerShell v2, Azure Portal y el portal Mis aplicaciones. Los servicios de Microsoft 365 que utilizan los grupos de Office 365 para escenarios de colaboración también se ven afectados, en concreto Outlook, Microsoft Teams y SharePoint.
¿A qué secciones del portal Mis aplicaciones afectará esta característica? | La funcionalidad de grupos del portal Mis aplicaciones respetará estos nuevos permisos. Esto incluye todas las rutas de acceso para ver la lista de grupos y la pertenencia a grupos en Mis aplicaciones. No se realizó ningún cambio en la disponibilidad del icono del grupo. La disponibilidad del icono del grupo todavía se controla mediante la configuración de grupo existente en el portal de administración de Azure.
¿Estos permisos invalidan la configuración de invitado de SharePoint o Microsoft Teams? | No. La configuración existente sigue controlando la experiencia con estas aplicaciones y el acceso a ellas. Por ejemplo, si ve problemas en SharePoint, compruebe la configuración de uso compartido externo.
¿Cuáles son los problemas de compatibilidad conocidos en Planner y Yammer? | <li>Con los permisos establecidos en "restringido", los invitados que han iniciado sesión en la aplicación Planner o que acceden a Planner en Microsoft Teams no podrán acceder a sus planes ni a ninguna tarea.<li>Con los permisos establecidos en "restringido", los invitados que han iniciado sesión en Yammer no podrán abandonar el grupo.
¿Se modificarán los permisos de invitado existentes en mi inquilino? | No se realizó ningún cambio en la configuración actual. Mantenemos la compatibilidad de la configuración existente con versiones anteriores. El usuario es quien decide cuándo realizar los cambios.
¿Estos permisos se establecerán de forma predeterminada? | No. Los permisos predeterminados existentes permanecen invariables. Opcionalmente, puede configurar los permisos para que sean más restrictivos.
¿Existen requisitos de licencia para esta característica? | No, no hay ningún requisito de licencia nuevo con esta característica.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los permisos de invitado existentes en Azure AD, vea [¿Cuales son los permisos de usuario predeterminados en Azure Active Directory?](../fundamentals/users-default-permissions.md)
- Para ver los métodos de Microsoft Graph API para restringir el acceso de invitado, vea [Tipo de recurso authorizationPolicy](/graph/api/resources/authorizationpolicy).
- Para revocar todo el acceso de un usuario, vea [Revocación del acceso de usuario en Azure Active Directory](users-revoke-access.md).