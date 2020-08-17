---
title: 'Canje de invitación en colaboración B2B: Azure AD'
description: Describe la experiencia de canje de invitación de colaboración B2B de Azure AD para los usuarios finales, incluido el acuerdo con los términos de privacidad.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 05/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisol
ms.collection: M365-identity-device-management
ms.openlocfilehash: d6bf5d40262c5991504d3dc62490fb50f6a20592
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907659"
---
# <a name="azure-active-directory-b2b-collaboration-invitation-redemption"></a>Experiencia de invitación de colaboración B2B de Azure Active Directory

En este artículo se describen las maneras en las que los usuarios invitados pueden acceder a los recursos y el proceso de consentimiento que se encuentran. Si envía un correo electrónico de invitación al invitado, la invitación incluye un vínculo que este puede canjear para acceder a la aplicación o al portal. El correo electrónico de invitación es solo uno de los modos de acceso de los invitados a los recursos. Como alternativa, puede agregar invitados a su directorio y proporcionarles un vínculo directo al portal o a la aplicación que desee compartir. Independientemente del método que usen, la primera vez se les guiará por un proceso de consentimiento. Este proceso garantiza que los invitados acepten los términos de privacidad y los [términos de uso](https://docs.microsoft.com/azure/active-directory/governance/active-directory-tou) que haya configurado.

Al agregar un usuario invitado al directorio, la cuenta de este tiene un estado de consentimiento (visible en PowerShell) que se establece inicialmente en **PendingAcceptance**. Esta configuración permanece hasta que el invitado acepta la invitación, la política de privacidad y los términos de uso. Después de eso, el estado de consentimiento cambia a **Accepted** y las páginas de consentimiento dejan de aparecer para el invitado.

   > [!IMPORTANT]
   > **A partir del 31 de marzo de 2021**, Microsoft dejará de admitir el canje de invitaciones mediante la creación de cuentas de Azure AD no administradas e inquilinos para escenarios de colaboración B2B. Como preparación, se recomienda a los clientes que opten por la [autenticación de código de acceso de un solo uso por correo electrónico](one-time-passcode.md). Agradecemos sus comentarios sobre esta característica en vista previa pública. Nos alegra poder crear más formas de colaborar.

## <a name="redemption-through-the-invitation-email"></a>Canje a través del correo electrónico de invitación

Al agregar un usuario invitado al directorio [mediante Azure Portal](https://docs.microsoft.com/azure/active-directory/b2b/b2b-quickstart-add-guest-users-portal), se envía un correo electrónico de invitación al invitado. También puede enviar correos electrónicos de invitación cuando [use PowerShell](https://docs.microsoft.com/azure/active-directory/b2b/b2b-quickstart-invite-powershell) para agregar usuarios invitados al directorio. Esta es una descripción de la experiencia del invitado cuando canjea el vínculo del correo electrónico.

1. El invitado recibe un [correo electrónico de invitación](https://docs.microsoft.com/azure/active-directory/b2b/invitation-email-elements) que se envía desde **Microsoft Invitations**.
2. El invitado selecciona **Aceptar invitación** en el correo electrónico.
3. El invitado usará sus propias credenciales para iniciar sesión en el directorio. Si el invitado no tiene una cuenta que se pueda federar al directorio y la característica [código de acceso de un solo uso (OTP) de correo electrónico](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode) no está habilitada, se solicita al invitado que cree una cuenta [MSA](https://support.microsoft.com/help/4026324/microsoft-account-how-to-create) personal o una [cuenta de autoservicio de Azure AD](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-self-service-signup). Consulte [Flujo de canje de invitación](#invitation-redemption-flow) para más información.
4. Al invitado se le guiará por la [experiencia de consentimiento](#consent-experience-for-the-guest) que se describe a continuación.

## <a name="redemption-through-a-direct-link"></a>Canje a través de un vínculo directo

Como alternativa a la invitación por correo electrónico, puede darle al invitado un vínculo directo a la aplicación o al portal. Primero debe agregar el usuario invitado a su directorio mediante [Azure Portal](https://docs.microsoft.com/azure/active-directory/b2b/b2b-quickstart-add-guest-users-portal) o [PowerShell](https://docs.microsoft.com/azure/active-directory/b2b/b2b-quickstart-invite-powershell). Puede usar cualquiera de las [maneras personalizables para implementar las aplicaciones para los usuarios](https://docs.microsoft.com/azure/active-directory/manage-apps/end-user-experiences), incluidos los vínculos de inicio de sesión en directo. Cuando un invitado usa un vínculo directo en lugar de la invitación por correo electrónico, también se le guía por la experiencia de consentimiento inicial.

> [!IMPORTANT]
> El vínculo directo debe ser específico del inquilino. En otras palabras, debe incluir un identificador de inquilino o dominio comprobado de manera que el invitado se puede autenticar en el inquilino donde se encuentra la aplicación compartida. Una dirección URL típica, como https://myapps.microsoft.com, no funcionará para un invitado, ya que redirige al inquilino principal para la autenticación. Estos son algunos ejemplos de vínculos directos con el contexto del inquilino:
 > - Panel de acceso de aplicaciones: `https://myapps.microsoft.com/?tenantid=<tenant id>`
 > - Panel de acceso de aplicaciones para un dominio comprobado: `https://myapps.microsoft.com/<;verified domain>`
 > - Azure Portal: `https://portal.azure.com/<tenant id>`
 > - Aplicación individual: consulte cómo usar un [vínculo de inicio de sesión en directo](../manage-apps/end-user-experiences.md#direct-sign-on-links)

En algunos casos se recomienda el correo electrónico de invitación en lugar del vínculo directo. Si estos casos especiales son importantes para su organización, se recomienda que invite a los usuarios mediante métodos que aún envíen el correo electrónico de invitación:
 - El usuario no tiene una cuenta de Azure AD, MSA o de correo electrónico en una organización federada. A menos que use la característica de código de acceso de un solo uso, el invitado deberá canjear la invitación por correo electrónico para que se le guíe para crear una MSA.
 - A veces, el objeto de usuario invitado puede no tener una dirección de correo electrónico debido a un conflicto con un objeto de contacto (por ejemplo, un objeto de contacto de Outlook). En este caso, el usuario debe hacer clic en la dirección URL de canje en el correo electrónico de invitación.
 - El usuario puede iniciar sesión con un alias de la dirección de correo electrónico a la que se invitó. (Un alias es una dirección de correo electrónico adicional asociada con una cuenta de correo electrónico). En este caso, el usuario debe hacer clic en la dirección URL de canje en el correo electrónico de invitación.

## <a name="invitation-redemption-flow"></a>Flujo del canje de invitación

Cuando un usuario hace clic en el vínculo **Aceptar invitación** de un [correo electrónico de invitación](invitation-email-elements.md), Azure AD canjea automáticamente la invitación basándose en el flujo de canje, del modo que se muestra a continuación:

![Captura de pantalla que muestra el diagrama de flujo del canje](media/redemption-experience/invitation-redemption-flow.png)

**Si el nombre principal de usuario (UPN) del usuario coincide con una cuenta existente de Azure AD y de MSA personal, se le pedirá al usuario que elija la cuenta con la que desea realizar el canje.*

1. Azure AD realiza la detección basada en usuarios para determinar si el usuario existe en un [inquilino de Azure AD existente](https://docs.microsoft.com/azure/active-directory/b2b/what-is-b2b#easily-add-guest-users-in-the-azure-ad-portal).

2. Si un administrador ha habilitado la [federación directa](https://docs.microsoft.com/azure/active-directory/b2b/direct-federation), Azure AD comprueba si el sufijo de dominio del usuario coincide con el dominio de un proveedor de identidades SAML/WS-FED configurado y redirige al usuario al proveedor de identidades configurado previamente.

3. Si un administrador ha habilitado la [federación de Google](https://docs.microsoft.com/azure/active-directory/b2b/google-federation), Azure AD comprueba si el sufijo de dominio del usuario es gmail.com o googlemail.com y redirige al usuario a Google.

4. El proceso de canje comprueba si el usuario tiene una [cuenta de Microsoft (MSA)](https://support.microsoft.com/help/4026324/microsoft-account-how-to-create) personal existente.

5. Una vez identificado el **directorio principal** del usuario, el usuario se envía al proveedor de identidades correspondiente para iniciar sesión.  

6. Si en los pasos 1 a 4 no se encuentra el directorio principal del usuario al que se ha enviado la invitación, Azure AD determina si el inquilino que realiza la invitación tiene habilitada la característica de [código de acceso de un solo uso (OTP) de correo electrónico](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode) para los invitados.

7. Si [el código de acceso de un solo uso de correo electrónico para invitados está habilitado](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode#when-does-a-guest-user-get-a-one-time-passcode), se envía un código de acceso al usuario por medio del correo electrónico de invitación. El usuario recuperará el código de acceso y lo escribirá en la página de inicio de sesión de Azure AD.

8. Si el código de acceso de un solo uso de correo electrónico para invitados está deshabilitado, Azure AD comprueba el sufijo de dominio para determinar si pertenece a una cuenta de consumidor. Si es así, se pedirá al usuario que cree una [cuenta Microsoft](https://support.microsoft.com/help/4026324/microsoft-account-how-to-create) personal. En caso contrario, se pedirá al usuario que cree una [cuenta de autoservicio de Azure AD](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-self-service-signup).

9. Azure AD intenta crear una [cuenta de autoservicio de Azure AD](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-self-service-signup) mediante la verificación del acceso al correo electrónico. Para realizar la comprobación de la cuenta, se envía un código al correo electrónico, código que el usuario debe recuperar y enviar a Azure AD. Sin embargo, si el inquilino del usuario al que se ha enviado la invitación está federado o si el campo AllowEmailVerifiedUsers está establecido en false en el inquilino del usuario invitado, este no puede completar el canje y el flujo da como resultado un error. Para más información, consulte [Solución de problemas de colaboración de Azure Active Directory B2B](https://docs.microsoft.com/azure/active-directory/b2b/troubleshoot#the-user-that-i-invited-is-receiving-an-error-during-redemption).

10. Se solicita al usuario que cree una [cuenta Microsoft (MSA)](https://support.microsoft.com/help/4026324/microsoft-account-how-to-create) personal.

11. Después de autenticarse en el proveedor de identidades correcto, se redirige al usuario a Azure AD para completar la [experiencia de consentimiento](https://docs.microsoft.com/azure/active-directory/b2b/redemption-experience#consent-experience-for-the-guest).  

En el caso de canjes Just-in-Time (JIT), en los que el canje se realiza por medio de un vínculo de aplicación con inquilinos, no están disponibles los pasos del 8 al 10. Si un usuario alcanza el paso 6 y la característica de código de acceso de un solo uso de correo electrónico no está habilitada, recibe un mensaje de error y no puede canjear la invitación. Para evitar este error, los administradores deben [habilitar el código de acceso de un solo uso de correo electrónico](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode#when-does-a-guest-user-get-a-one-time-passcode) o asegurarse de que el usuario haga clic en un vínculo de invitación.

## <a name="consent-experience-for-the-guest"></a>Experiencia de consentimiento para el invitado

Cuando un invitado iniciar sesión por primera vez para acceder a los recursos de una organización asociada, se le guía por las siguientes páginas. 

1. El invitado debe revisar la página **Revisar permisos**, donde se describe la declaración de privacidad de la organización anfitriona. Para poder continuar, el usuario debe **Aceptar** el uso de su información de acuerdo con las directivas de privacidad de la organización anfitriona.

   ![Captura de pantalla que muestra la página Revisar permisos](media/redemption-experience/review-permissions.png) 

   > [!NOTE]
   > Para obtener información sobre cómo puede, como administrador de inquilinos, vincular con la declaración de privacidad de su organización, vea [Procedimiento: Incorporación de información de privacidad de su organización en Azure Active Directory](https://aka.ms/adprivacystatement).

2. Si se han configurado términos de uso, el invitado debe abrirlos y revisarlos y seleccionar **Aceptar**. 

   ![Captura de pantalla en la que se muestran los nuevos términos de uso](media/redemption-experience/terms-of-use-accept.png) 

   Puede configurar los [Términos de uso](../governance/active-directory-tou.md) en **External Identities** > **Términos de uso**.

3. A menos que se especifique otra cosa, al invitado se le redirige al panel de acceso a las aplicaciones, que enumera las aplicaciones a las que puede acceder.

   ![Captura de pantalla que muestra el panel de acceso a las aplicaciones](media/redemption-experience/myapps.png) 

En su directorio, el valor de **Invitación aceptada** cambia a **Sí**. Si se creó una MSA, el **Origen** del usuario muestra **Cuenta Microsoft**. Para más información sobre las propiedades de la cuenta de usuario invitado, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](user-properties.md). 

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](add-users-administrator.md)
- [¿Cómo agregan los trabajadores de la información usuarios de colaboración B2B a Azure Active Directory?](add-users-information-worker.md)
- [Incorporación de usuarios de colaboración B2B de Azure Active Directory con PowerShell](customize-invitation-api.md#powershell)
- [Salir de una organización como usuario invitado](leave-the-organization.md)
