---
title: 'Invitar a usuarios internos a la colaboración B2B: Azure AD'
description: Si tiene cuentas de usuario internas para asociados, distribuidores, proveedores y otros invitados, puede cambiar a la colaboración B2B de Azure AD invitándolos a que inicien sesión con sus propias credenciales externas o su inicio de sesión. Use PowerShell o la API de invitación de Microsoft Graph.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 04/12/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: c03c2c55988df04cc45ef4a1d66d959513c1626d
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907688"
---
# <a name="invite-internal-users-to-b2b-collaboration"></a>Invitar a usuarios internos a la colaboración B2B

> [!NOTE]
> La invitación a los usuarios internos para usar la colaboración B2B es una característica en vista previa pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Antes de la disponibilidad de la colaboración B2B de Azure AD, las organizaciones pueden colaborar con distribuidores, proveedores y otros usuarios invitados mediante la configuración de credenciales internas para ellos. Si tiene usuarios invitados internos como este, puede invitarlos a usar la colaboración B2B para que pueda aprovechar las ventajas de B2B de Azure AD. Los usuarios invitados de B2B podrán usar sus propias identidades y credenciales para iniciar sesión, y no tendrá que mantener contraseñas ni administrar los ciclos de vida de las cuentas.

El envío de una invitación a una cuenta interna existente le permite conservar el identificador de objeto, el UPN, la pertenencia a grupos y las asignaciones de aplicaciones de ese usuario. No es necesario eliminar y volver a invitar al usuario manualmente ni reasignar los recursos. Para invitar al usuario, usará la API de invitación para pasar el objeto de usuario interno y la dirección de correo electrónico del usuario invitado junto con la invitación. Cuando el usuario acepta la invitación, el servicio B2B cambia el objeto de usuario interno existente a un usuario de B2B. A partir de ahí, el usuario debe iniciar sesión en los servicios de recursos de nube con sus credenciales de B2B. Todavía pueden usar sus credenciales internas para acceder a los recursos locales, pero puede evitarlo mediante el restablecimiento o la modificación de la contraseña de la cuenta interna.

> [!NOTE]
> La invitación es unidireccional. Puede invitar a usuarios internos a usar la colaboración B2B, pero no puede quitar las credenciales de B2B una vez agregadas. Para volver a cambiar el usuario a un usuario solo interno, deberá eliminar el objeto de usuario y crear otro.

En la versión preliminar pública, el método descrito en este artículo para invitar a usuarios internos a la colaboración B2B no se puede usar en estas instancias:

- Al usuario interno ya se le ha asignado una licencia de Exchange.
- El usuario pertenece a un dominio que está configurado para la federación directa del directorio.
- El usuario interno es una cuenta solo en la nube y su cuenta principal no está en Azure AD.

En estos casos, si el usuario interno debe cambiarse a un usuario de B2B, debe eliminar la cuenta interna y enviar al usuario una invitación para la colaboración B2B.

**Usuarios sincronizados locales**: para las cuentas de usuario sincronizadas entre el entorno local y la nube, el directorio local sigue siendo el origen de autoridad después de haber invitado a los usuarios a usar la colaboración B2B. Los cambios que realice en la cuenta local se sincronizarán con la cuenta en la nube, incluida la deshabilitación o eliminación de la cuenta. Por lo tanto, solo mediante la eliminación de la cuenta local, no puede evitar que el usuario inicie sesión en su cuenta local mientras conserva su cuenta en la nube. En su lugar, puede establecer la contraseña de la cuenta local en un GUID aleatorio u otro valor desconocido.

## <a name="how-to-invite-internal-users-to-b2b-collaboration"></a>Invitación de usuarios internos a la colaboración B2B

Puede usar PowerShell o la API de invitación para enviar una invitación a B2B al usuario interno. Asegúrese de que la dirección de correo electrónico que desea usar para la invitación está establecida como la dirección de correo electrónico externa en el objeto de usuario interno.

- Para un usuario solo en la nube, use la dirección de correo electrónico en la propiedad User.OtherMails de la invitación.
- Para un usuario sincronizado local, debe usar el valor de la propiedad User.Mail para la invitación.
- El dominio de la propiedad Mail del usuario debe coincidir con la cuenta que usan para iniciar sesión. De lo contrario, algunos servicios, como Teams, no podrán autenticar al usuario.

De forma predeterminada, se enviará un correo electrónico al usuario para notificarle que se le ha invitado, pero puede suprimir el mensaje y escribir uno personalizado.

> [!NOTE]
> Para enviar su propio correo electrónico u otra comunicación, puede usar New-AzureADMSInvitation with -SendInvitationMessage:$false para invitar a los usuarios de forma silenciosa y, a continuación, enviar su propio mensaje de correo electrónico al usuario convertido. Vea [Personalización y API de colaboración B2B de Active Directory Azure](customize-invitation-api.md).

## <a name="use-powershell-to-send-a-b2b-invitation"></a>Uso de PowerShell para enviar una invitación a B2B

Use el siguiente comando para invitar al usuario a la colaboración B2B:

```powershell
Uninstall-Module AzureADPreview
Install-Module AzureADPreview
$ADGraphUser = Get-AzureADUser -objectID "UPN of Internal User"
$msGraphUser = New-Object Microsoft.Open.MSGraph.Model.User -ArgumentList $ADGraphUser.ObjectId
New-AzureADMSInvitation -InvitedUserEmailAddress <<external email>> -SendInvitationMessage $True -InviteRedirectUrl "http://myapps.microsoft.com" -InvitedUser $msGraphUser
```

## <a name="use-the-invitation-api-to-send-a-b2b-invitation"></a>Uso de la API de invitación para enviar una invitación a B2B

En el ejemplo siguiente se muestra cómo llamar a la API de invitación para invitar a un usuario interno como un usuario de B2B.

```json
POST https://graph.microsoft.com/v1.0/invitations
Authorization: Bearer eyJ0eX...
ContentType: application/json
{
    "invitedUserEmailAddress": "<<external email>>",
    "sendInvitationMessage": true,
    "invitedUserMessageInfo": {
        "messageLanguage": "en-US",
        "ccRecipients": [
            {
                "emailAddress": {
                    "name": null,
                    "address": "<<optional additional notification email>>"
                }
            }
        ],
        "customizedMessageBody": "<<custom message>>"
    },
    "inviteRedirectUrl": "https://myapps.microsoft.com?tenantId=",
    "invitedUser": {
        "id": "<<ID for the user you want to convert>>"
    }
}
```

La respuesta a la API es la misma respuesta que se obtiene al invitar a un nuevo usuario invitado al directorio.

## <a name="next-steps"></a>Pasos siguientes

- [Canje de invitación de colaboración B2B](redemption-experience.md)
