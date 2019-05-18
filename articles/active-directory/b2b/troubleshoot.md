---
title: Solución de problemas de colaboración de B2B - Azure Active Directory | Microsoft Docs
description: Recursos para solucionar problemas comunes con la colaboración B2B de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: mimart
author: v-miegge
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4185d29ff1770ed9549b4b63a2e5da579bcf054f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65767156"
---
# <a name="troubleshooting-azure-active-directory-b2b-collaboration"></a>Solución de problemas de colaboración B2B de Azure Active Directory

Estos son algunos de los recursos para solucionar problemas comunes relacionados con la colaboración B2B de Azure Active Directory (Azure AD).

## <a name="ive-added-an-external-user-but-do-not-see-them-in-my-global-address-book-or-in-the-people-picker"></a>He agregado un usuario externo, pero no lo veo en mi libreta de direcciones global o en el selector de personas

En los casos donde los usuarios externos no se rellenan en la lista, el objeto puede tardar unos minutos en replicarse.

## <a name="a-b2b-guest-user-is-not-showing-up-in-sharepoint-onlineonedrive-people-picker"></a>Un usuario invitado de B2B no aparece en el selector de personas de SharePoint Online/OneDrive

La posibilidad de buscar usuarios invitados existentes en el selector de personas de SharePoint Online (SPO) está desactivada de manera predeterminada para así coincidir con el comportamiento heredado.

Para habilitarla, use la configuración "ShowPeoplePickerSuggestionsForGuestUsers" en el nivel de inquilino y colección de sitios. Para configurarla, use los cmdlets Set-SPOTenant y Set-SPOSite, que permiten que los miembros busquen a todos los usuarios invitados existentes en el directorio. Los cambios en el ámbito del inquilino no afectan los sitios de SPO que ya se aprovisionaron.

## <a name="invitations-have-been-disabled-for-directory"></a>Las invitaciones se han deshabilitado para el directorio

Si recibe una notificación de que no tiene permisos para invitar a usuarios, vaya a User Settings (Configuración de usuario) y compruebe que la cuenta de usuario está autorizada para invitar a usuarios externos.

![Captura de pantalla que muestra los valores de los usuarios externos](media/troubleshoot/external-user-settings.png)

Si hace poco ha modificado estos valores o asignado el rol de invitador invitado a un usuario, podría haber un retraso de 15 a 60 minutos para que los cambios surtan efecto.

## <a name="the-user-that-i-invited-is-receiving-an-error-during-redemption"></a>El usuario al que he invitado está recibiendo un error durante el canje

Estos son algunos de los errores comunes:

### <a name="invitees-admin-has-disallowed-emailverified-users-from-being-created-in-their-tenant"></a>El administrador del invitado no permite que los usuarios de EmailVerified se creen en su inquilino

Cuando se invita a usuarios cuya organización usa Azure Active Directory, pero donde no existe la cuenta de usuario en concreto (por ejemplo, el usuario no existe en Azure AD contoso.com). El administrador de contoso.com puede tener una directiva en lugar de evitar que se creen usuarios. El usuario debe consultar al administrador y determinar si se permiten los usuarios externos. Es posible que el administrador del usuario externo necesite permitir usuarios comprobados por correo electrónico en el dominio (consulte este [artículo](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0) sobre cómo permitir usuarios comprobados por correo electrónico).

![Los usuarios comprobados de error que indica que el inquilino no permite el correo electrónico](media/troubleshoot/allow-email-verified-users.png)

### <a name="external-user-does-not-exist-already-in-a-federated-domain"></a>El usuario externo no se encuentra ya en un dominio federado

Si va a usar la autenticación de federación y el usuario no existe en Azure Active Directory, no se puede invitar al usuario.

Para resolver este problema, el administrador del usuario externo debe sincronizar la cuenta del usuario con Azure Active Directory.

## <a name="how-does--which-is-not-normally-a-valid-character-sync-with-azure-ad"></a>¿Cómo "\#", que habitualmente no es un carácter válido, se sincroniza con Azure AD?

"\#" es un carácter reservado en los UPN para la colaboración B2B de Azure AD o los usuarios externos, porque la cuenta de invitado user@contoso.com se convierte en user_contoso.com#EXT#@fabrikam.onmicrosoft.com. Por lo tanto, los UPN con \# que proceden del entorno local no tienen permiso para iniciar sesión en Azure Portal. 

## <a name="i-receive-an-error-when-adding-external-users-to-a-synchronized-group"></a>Recibo un error al agregar los usuarios externos a un grupo sincronizado

Los usuarios externos pueden agregarse únicamente a los grupos "Seguridad" o "Asignado", y no a aquellos que se controlan localmente.

## <a name="my-external-user-did-not-receive-an-email-to-redeem"></a>Mi usuario externo no ha recibido un correo electrónico para realizar el canje

El invitado debe ponerse en contacto con su ISP o comprobar su filtro de correo no deseado para asegurarse de que se permite la siguiente dirección: Invites@microsoft.com

## <a name="i-notice-that-the-custom-message-does-not-get-included-with-invitation-messages-at-times"></a>Tenga en cuenta que, en ocasiones, el mensaje personalizado no se incluye en los mensajes de invitación

Para cumplir con las leyes de privacidad, nuestras API no incluye mensajes personalizados en la invitación por correo electrónico cuando:

- El invitador no tiene una dirección de correo electrónico en el inquilino que invita.
- Cuando una entidad de seguridad de App Service envía la invitación

Si este escenario es importante para usted, puede suprimir nuestro correo electrónico de invitación de API y enviarlo a través del mecanismo de correo electrónico de su elección. Consulte al asesor legal de su organización para asegurarse de que cualquier correo electrónico que envíe de esta forma también cumple las leyes de privacidad.

## <a name="you-receive-an-aadsts65005-error-when-you-try-to-log-in-to-an-azure-resource"></a>Recibirá un error de "AADSTS65005" cuando intenta iniciar sesión en un recurso de Azure

Un usuario que tenga una cuenta de invitado no puede iniciar sesión y recibe el mensaje de error siguiente:

    AADSTS65005: Using application 'AppName' is currently not supported for your organization contoso.com because it is in an unmanaged state. An administrator needs to claim ownership of the company by DNS validation of contoso.com before the application AppName can be provisioned.

El usuario tiene una cuenta de usuario de Azure y es un inquilino viral que se ha abandonado o no administrados. Además, hay no globales o administradores en el inquilino de la empresa.

Para resolver este problema, debe asumir el inquilino abandonado. Consulte [adquirir un directorio no administrado como administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/domains-admin-takeover). También debe tener acceso el DNS con conexión a internet para el sufijo de dominio en cuestión con el fin de proporcionar evidencia directo que se encuentra el control del espacio de nombres. Después de que el inquilino se devuelve a un estado administrado, por favor, hable con el cliente si se deja a los usuarios y nombre de dominio comprobado es la mejor opción para su organización.

## <a name="a-guest-user-with-a-just-in-time-or-viral-tenant-is-unable-to-reset-their-password"></a>Un usuario invitado con un inquilino Just-In-Time o "viral" no puede restablecer su contraseña

Si el inquilino de la identidad es un inquilino Just-In-Time (JIT) o un inquilino "viral" (es decir, un inquilino de Azure que es independiente y no está administrado), solamente el usuario invitado podrá restablecer su contraseña. A veces, una organización [asumirá la administración de los inquilinos virales](https://docs.microsoft.com/azure/active-directory/users-groups-roles/domains-admin-takeover) que se crean cuando los empleados usan sus direcciones de correo electrónico del trabajo para registrarse en los servicios. Después de que la organización se haga cargo de un inquilino viral, solo un administrador de dicha organización puede restablecer la contraseña del usuario o habilitar SSPR. Si es necesario, como la organización que invita, puede quitar la cuenta del usuario invitado del directorio y volver a enviar una invitación.

## <a name="next-steps"></a>Pasos siguientes

[Obtención de soporte técnico para la colaboración B2B](get-support.md)
