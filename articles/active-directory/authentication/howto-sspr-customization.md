---
title: 'Personalización del autoservicio de restablecimiento de contraseña: Azure Active Directory'
description: Obtenga información sobre cómo personalizar opciones de experiencia y visualización del usuario para el autoservicio de restablecimiento de contraseña de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/17/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8690b1a4b8527a397fcf4c04892e167b5332358d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035407"
---
# <a name="customize-the-user-experience-for-azure-active-directory-self-service-password-reset"></a>Personalización de la experiencia del usuario para el autoservicio de restablecimiento de contraseña de Azure Active Directory

El autoservicio de restablecimiento de contraseña (SSPR) ofrece a los usuarios de Azure Active Directory (Azure AD) la posibilidad de cambiar o restablecer su contraseña sin necesidad de que intervenga el administrador o el departamento de soporte técnico. Si la cuenta de un usuario está bloqueada o se ha olvidado su contraseña, puede seguir las indicaciones para desbloquearse y volver al trabajo. Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación.

Para mejorar la experiencia de SSPR de los usuarios, puede personalizar la apariencia de la página de restablecimiento de contraseña, las notificaciones por correo electrónico o las páginas de inicio de sesión. Estas opciones de personalización permiten que el usuario sepa con certeza que se encuentra en el lugar correcto y tenga la seguridad de que esta obteniendo acceso a recursos de la empresa.
    
En este artículo se muestra cómo personalizar el vínculo de correo electrónico de SSPR para los usuarios, la personalización de marca de empresa y el vínculo de la página de inicio de sesión de AD FS.

## <a name="customize-the-contact-your-administrator-link"></a>Personalizar el vínculo para ponerse en contacto con el administrador

Para que los usuarios obtengan ayuda con el autoservicio de restablecimiento de contraseña, se muestra un vínculo "Póngase en contacto con el administrador" en el portal de restablecimiento de contraseña. Si un usuario selecciona este vínculo, hará una de estas dos cosas:

* Si se deja este vínculo de contacto en el estado predeterminado, se envía un correo electrónico a los administradores en el que se solicita asistencia para cambiar la contraseña del usuario. En el ejemplo de correo electrónico siguiente se muestra este mensaje de correo electrónico predeterminado:

    ![Ejemplo de solicitud para restablecer el correo electrónico que se envía al administrador](./media/howto-sspr-customization/sspr-contact-admin.png)

* En caso de personalizarse, se envía al usuario a una página web o dirección de correo electrónico especificada por el administrador para obtener asistencia.
    * Si se personaliza, se recomienda establecerlo en algo con lo que los usuarios ya estén familiarizados para soporte técnico.

    > [!WARNING]
    > Si personaliza esta configuración con una dirección de correo electrónico y una cuenta que necesita un restablecimiento de contraseña, es posible que el usuario no pueda pedir asistencia.

### <a name="default-email-behavior"></a>Comportamiento del correo electrónico predeterminado

El correo electrónico de contacto predeterminado se envía a los destinatarios en el orden siguiente:

1. Si el rol *Administrador del departamento de soporte técnico* o *Administrador de contraseñas* está asignado, se notificará a todos administradores que los tengan.
1. Si no hay asignado ningún administrador del departamento de soporte técnico o de contraseñas, entonces se notificará a los que tengan el rol de *administrador de usuarios*.
1. Si no se ha asignado ninguno de los roles anteriores, se notificará a los *administradores globales*.

En todos los casos, se notificará a un máximo de 100 destinatarios en total.

Para obtener más información sobre los diferentes roles de administrador y cómo asignarlos, consulte [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md).

### <a name="disable-contact-your-administrator-emails"></a>Deshabilitar los correos electrónicos para contactar con los administradores

Si la organización no quiere que se notifique a los administradores de las solicitudes de restablecimiento de contraseña, se pueden usar las opciones de configuración siguientes:

* Personalice el vínculo del departamento de soporte técnico para proporcionar una dirección URL web o una dirección mailto: que los usuarios puedan usar para obtener asistencia. Esta opción se encuentra en **Restablecimiento de contraseña** > **Personalización** > **Dirección URL o correo electrónico personalizados del departamento de soporte técnico**.
* Habilite el autoservicio de restablecimiento de contraseña para todos los usuarios. Esta opción está disponible en **Restablecimiento de contraseña** > **Propiedades**. Si no quiere que los usuarios restablezcan sus propias contraseñas, puede aumentar el ámbito del acceso a un grupo vacío. *(Esta opción no se recomienda).*

## <a name="customize-the-sign-in-page-and-access-panel"></a>Personalización del panel de acceso y la página de inicio de sesión

Puede personalizar la página de inicio de sesión, por ejemplo, para agregar un logotipo que aparezca junto a la imagen que se adapte a la personalización de marca de empresa. Para obtener más información sobre cómo configurar la personalización de marca de empresa, consulte [Incorporación de personalización de marca de empresa a su página de inicio de sesión de Azure AD](../fundamentals/customize-branding.md).

Los gráficos que elija se muestran en los siguientes casos:

* Después de que un usuario escriba su nombre de usuario.
* Si el usuario accede a la dirección URL personalizada:
   * Pasando el parámetro `whr` a la página de restablecimiento de contraseña, como `https://login.microsoftonline.com/?whr=contoso.com`.
   * Pasando el parámetro `username` a la página de restablecimiento de contraseña, como `https://login.microsoftonline.com/?username=admin@contoso.com`.

### <a name="directory-name"></a>Nombre de directorio

Para que todo resulte más sencillo, puede cambiar el nombre de la organización en el portal y en las comunicaciones automatizadas. Para cambiar el atributo de nombre de directorio en Azure Portal, vaya a **Azure Active Directory** > **Propiedades**. Esta opción de nombre descriptivo de la organización es la más visible en correos electrónicos automatizados, tal como se muestra en los ejemplos siguientes:

* Nombre descriptivo del correo electrónico, por ejemplo "*Demostración de Microsoft en nombre de CONTOSO*"
* Línea de asunto del correo electrónico, por ejemplo "*Código de verificación del correo electrónico de la cuenta de demostración CONTOSO*"

## <a name="customize-the-ad-fs-sign-in-page"></a>Personalizar la página de inicio de sesión de AD FS

Si usa Servicios de federación de Active Directory (AD FS) para eventos de inicio de sesión de usuario, puede agregar un vínculo a la página de inicio de sesión mediante las directrices del artículo [Agregar la descripción de la página de inicio de sesión](/windows-server/identity/ad-fs/operations/add-sign-in-page-description).

Proporcione a los usuarios un vínculo a la página para que especifiquen el flujo de trabajo de SSPR, como *https://passwordreset.microsoftonline.com* . Para agregar un vínculo a la página de inicio de sesión de AD FS, use el siguiente comando en el servidor de AD FS:

``` powershell
Set-ADFSGlobalWebContent -SigninPageDescriptionText "<p><a href='https://passwordreset.microsoftonline.com' target='_blank'>Can't access your account?</a></p>"
```

## <a name="next-steps"></a>Pasos siguientes

Para comprender el uso de SSPR en su entorno, consulte [Opciones de creación de informes para la administración de contraseñas de Azure AD](howto-sspr-reporting.md).

Si usted o los usuarios tienen problemas con SSPR, consulte [Solución de problemas de autoservicio de restablecimiento de contraseña](active-directory-passwords-troubleshoot.md).
