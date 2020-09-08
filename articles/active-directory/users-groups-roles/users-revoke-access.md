---
title: Revocación del acceso de usuario en un emergencia de Azure Active Directory | Microsoft Docs
description: Adición de usuarios en bloque en el Centro de administración de Azure AD de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.topic: how-to
author: curtand
ms.author: curtand
manager: daveba
ms.reviewer: krbain
ms.date: 07/15/2020
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1cc4a29c9d4b5ae93df81de5b77cb6355947813d
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798421"
---
# <a name="revoke-user-access-in-azure-active-directory"></a>Revocación del acceso de usuario en Azure Active Directory

Entre los escenarios que podrían requerir que un administrador revoque todo el acceso de un usuario, se incluyen cuentas en peligro, rescisión de empleados y otras amenazas internas. En función de la complejidad del entorno, los administradores pueden realizar varios pasos para garantizar la revocación del acceso. En algunos escenarios, puede haber un período entre el inicio de la revocación del acceso y el momento en que se revoca realmente.

Para mitigar los riesgos, debe comprender cómo funcionan los tokens. Hay muchos tipos de tokens, que se encuentran en uno de los patrones que se mencionan en las secciones siguientes.

## <a name="access-tokens-and-refresh-tokens"></a>Tokens de acceso y tokens de actualización

Los tokens de acceso y los tokens de actualización se usan con frecuencia con aplicaciones cliente pesado y también se usan en aplicaciones basadas en explorador, como las aplicaciones de una sola página.

- Cuando los usuarios se autentican en Azure AD, se evalúan las directivas de autorización para determinar si se puede conceder acceso al usuario para un recurso específico.  

- Si se autoriza, Azure AD emite un token de acceso y un token de actualización para el recurso.  

- Los tokens de acceso que emite Azure AD duran 1 hora de forma predeterminada. Si el protocolo de autenticación lo permite, la aplicación puede pasar el token de actualización a Azure AD cuando expire el token de acceso con la finalidad de volver a autenticar al usuario de forma silenciosa.

A continuación, Azure AD vuelve a evaluar sus directivas de autorización. Si el usuario sigue autorizado, Azure AD emite un nuevo token de acceso y un token de actualización.

Los tokens de acceso pueden ser un problema de seguridad si el acceso se debe revocar en un período de tiempo menor que la duración del token, que suele ser de una hora aproximadamente. Por esta razón, Microsoft trabaja activamente para incorporar la [evaluación del acceso continua](../fundamentals/concept-fundamentals-continuous-access-evaluation.md) a las aplicaciones de Office 365, lo que ayuda a garantizar la invalidación de los tokens de acceso casi en tiempo real.  

## <a name="session-tokens-cookies"></a>Tokens de sesión (cookies)

La mayoría de las aplicaciones basadas en explorador usan tokens de sesión en lugar de tokens de acceso y de actualización.  

- Cuando un usuario abre un explorador y se autentica en una aplicación a través de Azure AD, recibe dos tokens de sesión. Uno de Azure AD y otro de la aplicación.  

- Una vez que una aplicación emite su propio token de sesión, el acceso a la aplicación se rige por la sesión de la aplicación. En este momento, el usuario solo se ve afectado por las directivas de autorización de las que la aplicación es consciente.

- Las directivas de autorización de Azure AD se vuelven a evaluar con la frecuencia con la que la aplicación vuelve a enviar al usuario a Azure AD. Normalmente, la reevaluación se produce de forma silenciosa, aunque la frecuencia depende de cómo esté configurada la aplicación. Es posible que la aplicación nunca vuelva a enviar al usuario a Azure AD siempre y cuando el token de sesión sea válido.

- Para poder revocar un token de sesión, la aplicación debe revocar el acceso en función de sus propias directivas de autorización. Azure AD no puede revocar directamente un token de sesión emitido por una aplicación.  

## <a name="revoke-access-for-a-user-in-the-hybrid-environment"></a>Revocar el acceso de un usuario en el entorno híbrido

Para un entorno híbrido con Active Directory local sincronizado con Azure Active Directory, Microsoft recomienda a los administradores de TI que realicen las siguientes acciones.  

### <a name="on-premises-active-directory-environment"></a>Entorno de Active Directory local

Como administrador de Active Directory, conéctese a la red local, abra PowerShell y realice las siguientes acciones:

1. Deshabilite al usuario en Active Directory. Consulte [Disable-ADAccount](/powershell/module/addsadministration/disable-adaccount?view=win10-ps).

    ```PowerShell
    Disable-ADAccount -Identity johndoe  
    ```

1. Restablezca la contraseña del usuario dos veces en Active Directory. Consulte [Set-ADAccountPassword](/powershell/module/addsadministration/set-adaccountpassword?view=win10-ps).

    > [!NOTE]
    > La razón para cambiar la contraseña de un usuario dos veces es mitigar el riesgo de ataques Pass-the-hash, especialmente si hay retrasos en la replicación de contraseñas local. Si puede suponer con seguridad que esta cuenta no está en peligro, puede restablecer la contraseña solo una vez.

    > [!IMPORTANT] 
    > No use las contraseñas de ejemplo en los siguientes cmdlets. Asegúrese de cambiar las contraseñas por una cadena aleatoria.

    ```PowerShell
    Set-ADAccountPassword -Identity johndoe -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd1" -Force)
    Set-ADAccountPassword -Identity johndoe -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd2" -Force)
    ```

### <a name="azure-active-directory-environment"></a>Entorno de Azure Active Directory

Como administrador de Azure Active Directory, abra PowerShell, ejecute ``Connect-AzureAD`` y realice las siguientes acciones:

1. Deshabilite al usuario en Azure AD. Consulte [Set-AzureADUser](/powershell/module/azuread/Set-AzureADUser?view=azureadps-2.0).

    ```PowerShell
    Set-AzureADUser -ObjectId johndoe@contoso.com -AccountEnabled $false
    ```
1. Revoque los tokens de actualización de Azure AD del usuario. Consulte [Revoke-AzureADUserAllRefreshToken](/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0).

    ```PowerShell
    Revoke-AzureADUserAllRefreshToken -ObjectId johndoe@contoso.com
    ```

1. Deshabilite los dispositivos del usuario. Consulte [Get-AzureADUserRegisteredDevice](/powershell/module/azuread/get-azureaduserregistereddevice?view=azureadps-2.0).

    ```PowerShell
    Get-AzureADUserRegisteredDevice -ObjectId johndoe@contoso.com | Set-AzureADDevice -AccountEnabled $false
    ```

## <a name="optional-steps"></a>Pasos opcionales

- [Borre solo los datos corporativos de aplicaciones administradas por Intune](/mem/intune/apps/apps-selective-wipe).

- [Borre los dispositivos de propiedad corporativa mediante el restablecimiento del dispositivo a la configuración predeterminada de fábrica](/mem/intune/remote-actions/devices-wipe).

> [!NOTE]
> Los datos del dispositivo no se pueden recuperar después de un borrado.

## <a name="when-access-is-revoked"></a>Cuando se revoca el acceso

Una vez que los administradores han realizado los pasos anteriores, el usuario no puede obtener nuevos tokens para ninguna aplicación asociada a Azure Active Directory. El tiempo transcurrido entre la revocación y el momento en que el usuario pierde el acceso depende del modo en que la aplicación concede el acceso:

- Para las **aplicaciones que usan tokens de acceso**, el usuario pierde el acceso cuando expira el token de acceso.

- En el caso de las **aplicaciones que usan tokens de sesión**, las sesiones existentes finalizan cuando expira el token. Si el estado deshabilitado del usuario se sincroniza con la aplicación, la aplicación puede revocar automáticamente las sesiones existentes del usuario si está configurada para hacerlo.  El tiempo que se tarda depende de la frecuencia de sincronización entre la aplicación y Azure AD.

## <a name="next-steps"></a>Pasos siguientes

- [Procedimientos de acceso seguro para administradores de Azure AD](directory-admin-roles-secure.md)
- [Agregar o actualizar la información del perfil de usuario](../fundamentals/active-directory-users-profile-azure-portal.md)