---
title: 'Restablecimiento de la contraseña con la información de seguridad: Azure Active Directory | Microsoft Docs'
description: Cómo restablecer la contraseña si la olvida, mediante la información de seguridad y la verificación en dos pasos.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 05/28/2020
ms.author: curtand
ms.openlocfilehash: b83cdf0b069eac2e46cfdd54edc11a1aa5a41075
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799050"
---
# <a name="reset-your-work-or-school-password-using-security-info"></a>Restablecimiento de la contraseña profesional o educativa con la información de seguridad

Si alguna vez ha olvidado su contraseña en el trabajo o en el centro educativo, en su organización no ha tenido nunca contraseña o le han bloqueado la cuenta, puede usar la información de seguridad y el dispositivo móvil para restablecer su contraseña profesional o educativa. Para poder configurar sus datos y [restablecer su contraseña](./active-directory-passwords-reset-register.md), es preciso que el administrador active esta característica.

Si conoce la contraseña, pero desea cambiarla, consulte las secciones [Para cambiar la contraseña](#how-to-change-your-password) de este artículo.

>[!Important]
>Este artículo está destinado a los usuarios que quieren usar el restablecimiento de contraseñas de cuentas profesionales o educativas olvidadas o desconocidas. Si es un administrador que desea obtener información acerca de cómo activar el autoservicio de restablecimiento de contraseña para los empleados u otros usuarios, consulte [Implementar el autoservicio de restablecimiento de contraseña de Azure AD y otros artículos](../authentication/howto-sspr-deployment.md).

## <a name="how-to-reset-or-unlock-your-password-for-a-work-or-school-account"></a>Restablecimiento o desbloqueo de la contraseña de una cuenta profesional o educativa

Si no puede acceder a su cuenta de Azure Active Directory (Azure AD), el motivo puede ser alguno de estos:

- La contraseña no funciona y quiere restablecerla.

- Conoce la contraseña, pero la cuenta está bloqueada y necesita desbloquearla.

### <a name="to-reset-your-password-and-get-back-into-your-account"></a>Para restablecer la contraseña y volver a su cuenta

1. En la pantalla **Escribir contraseña**, seleccione **He olvidado mi contraseña**.

2. En la pantalla **Vuelva a su cuenta**, escriba el **Id. de usuario** profesional o educativo (por ejemplo, la dirección de correo electrónico), demuestre que no es un robot escribiendo los caracteres que observa en la pantalla y luego seleccione **Siguiente**.

   ![Pantalla Vuelva a su cuenta](media/security-info/security-info-back-into-acct.png)

   >[!NOTE]
   >Si su administrador no ha activado la capacidad para permitirle restablecer su contraseña, verá un vínculo **Póngase en contacto con el administrador** en lugar de la pantalla **Vuelva a su cuenta**. Este vínculo le permite ponerse en contacto con su administrador para que le indique cómo restablecer su contraseña, bien por correo electrónico o en un portal web.

3. Elija uno de los métodos siguientes para verificar su identidad y cambiar la contraseña. Según cómo el administrador haya configurado su organización, puede que deba realizar este proceso una segunda vez, con la adición de información para un segundo paso de verificación.

    ![Vuelva a su cuenta, paso de verificación 1](media/security-info/security-info-back-into-acct2.png)

    >[!NOTE]
    >Según cómo el administrador haya configurado su organización, puede que algunas de estas opciones de verificación no estén disponibles. Debe haber configurado previamente el dispositivo móvil para la verificación al menos con uno de estos métodos.<br><br>Además, puede que la nueva contraseña tenga que cumplir determinados requisitos de seguridad. Normalmente, las contraseñas seguras tienen entre ocho y dieciséis caracteres, incluidos los caracteres en mayúsculas y minúsculas, al menos un número y al menos un carácter especial.

- **Restablezca la contraseña con una dirección de correo electrónico.** Se envía un correo electrónico a la dirección de correo electrónico que configuró previamente en la verificación en dos pasos o en la información de seguridad. Si el administrador ha activado la experiencia de información de seguridad, puede encontrar más información acerca de cómo configurar una dirección de correo electrónico en el artículo [Configuración de la información de seguridad para usar el correo electrónico (versión preliminar)](security-info-setup-email.md). Si aún no usa la información de seguridad, puede encontrar más información acerca de cómo configurar una dirección de correo electrónico en el artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md). 

    1. Seleccione **Enviar un mensaje de correo electrónico a mi dirección alternativa** y después seleccione **Correo electrónico**.

    2. Escriba en el cuadro el código de verificación enviado en el correo electrónico y luego seleccione **Siguiente**.

    3. Escriba y confirme la nueva contraseña, y luego seleccione **Finalizar**.

- **Restablezca la contraseña con un mensaje de texto.** Se envía un mensaje de texto al número de teléfono que configuró previamente en la información de seguridad. Si el administrador ha activado la experiencia de información de seguridad, puede encontrar más información acerca de cómo configurar la mensajería de texto en el artículo [Configuración de la información de seguridad para usar la mensajería de texto (versión preliminar)](security-info-setup-text-msg.md). Si aún no usa la información de seguridad, puede encontrar más información acerca de cómo configurar la mensajería de texto en el artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

    1. Seleccione **Enviar mensaje de texto a mi teléfono móvil**, escriba el número de teléfono y después seleccione **Texto**.

    2. Escriba en el cuadro el código de verificación enviado en el mensaje de texto y luego seleccione **Siguiente**.

    3. Escriba y confirme la nueva contraseña, y luego seleccione **Finalizar**.

- **Restablezca la contraseña con un número de teléfono.** Se envía un mensaje de texto al número de teléfono que configuró previamente en la información de seguridad. Si el administrador ha activado la experiencia de información de seguridad, puede encontrar más información acerca de cómo configurar un número de teléfono en el artículo [Configuración de la información de seguridad para usar una llamada telefónica (versión preliminar)](security-info-setup-phone-number.md). Si aún no usa la información de seguridad, puede encontrar más información acerca de cómo configurar un número de teléfono en el artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

    1. Seleccione **Llamar al teléfono móvil**, escriba el número de teléfono y después seleccione **Llamar**.

    2. Responda a la llamada telefónica y siga las instrucciones para verificar su identidad y luego seleccione **Siguiente**.

    3. Escriba y confirme la nueva contraseña, y luego seleccione **Finalizar**.

- **Restablezca su contraseña mediante preguntas de seguridad.** Se muestra la lista de preguntas de seguridad configuradas en la información de seguridad. Si el administrador ha activado la experiencia de información de seguridad, puede encontrar más información acerca de cómo configurar sus preguntas de seguridad en el artículo [Configuración de la información de seguridad para usar preguntas de seguridad predefinidas (versión preliminar)](security-info-setup-questions.md). Si aún no usa la información de seguridad, puede encontrar más información acerca de cómo configurar pregunta de seguridad en el artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

    1. Seleccione **Responder a mis preguntas de seguridad**, responda a las preguntas y después seleccione **Siguiente**.

    2. Escriba y confirme la nueva contraseña, y luego seleccione **Finalizar**.

- **Restablezca su contraseña mediante una notificación de la aplicación autenticadora.** Envía una notificación de aprobación a la aplicación autenticadora. Si el administrador ha activado la experiencia de información de seguridad, puede encontrar más información acerca de cómo configurar una aplicación autenticadora en el artículo [Configuración de la información de seguridad para usar una aplicación autenticadora (versión preliminar)](security-info-setup-auth-app.md). Si aún no usa la información de seguridad, puede encontrar más información acerca de cómo configurar una aplicación autenticadora en el artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

    1. Seleccione **Aprobar una notificación en la aplicación autenticadora** y después seleccione **Enviar notificación**.

    2. Apruebe el inicio de sesión desde la aplicación autenticadora.

    3. Escriba y confirme la nueva contraseña, y luego seleccione **Finalizar**.

- **Restablezca su contraseña mediante un código de la aplicación autenticadora.** Se acepta un código aleatorio proporcionado por la aplicación autenticadora. Si el administrador ha activado la experiencia de información de seguridad, puede encontrar más información acerca de cómo configurar una aplicación autenticadora para que proporcione un código en el artículo [Configuración de la información de seguridad para usar una aplicación autenticadora (versión preliminar)](security-info-setup-auth-app.md). Si aún no usa la información de seguridad, puede encontrar más información acerca de cómo configurar una aplicación autenticadora para que proporcione un código en el artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

  1. Seleccione **Escribir un código de mi aplicación autenticadora** y después seleccione **Enviar notificación**.

  2. Abra la aplicación autenticadora, escriba el código de verificación de la cuenta en el cuadro y después seleccione **Siguiente**.

  3. Escriba y confirme la nueva contraseña, y luego seleccione **Finalizar**.

  4. Después de obtener el mensaje que confirma que se ha restablecido su contraseña, puede iniciar sesión en su cuenta con la nueva contraseña.

     Si todavía no puede acceder a su cuenta, debe ponerse en contacto con el administrador de la organización para obtener más ayuda.

Después de restablecer la contraseña, puede que reciba un correo electrónico de confirmación que procede de una cuenta como "Microsoft en nombre de \<*your_organization*>". Si recibe un correo electrónico similar pero no ha restablecido la contraseña recientemente, póngase en contacto de inmediato con el administrador de la organización.

## <a name="how-to-change-your-password"></a>Cambio de la contraseña

Si desea cambiar la contraseña, puede hacerlo mediante el portal de Office 365, el panel de acceso de Azure o la página de inicio de sesión de Windows 10.

### <a name="to-change-your-password-using-the-office-365-portal"></a>Para cambiar la contraseña desde el portal de Office 365

Use este método si normalmente accede a las aplicaciones mediante el Portal de Office:

1. Inicie sesión en su [cuenta de Office 365](https://portal.office.com) con la contraseña existente.

2. Seleccione el perfil que se encuentra en la parte superior derecha y luego elija **Ver cuenta**.

3. Seleccione **Seguridad y privacidad** > **Contraseña**.

4. Escriba la contraseña anterior, establezca la nueva contraseña y confírmela, y luego seleccione **Enviar**.

### <a name="to-change-your-password-from-the-azure-access-panel"></a>Para cambiar la contraseña en el panel de acceso de Azure

Use este método si normalmente accede a las aplicaciones mediante el panel de acceso de Azure (MyApps):

1. Inicie sesión en el [panel de acceso de Azure](https://myapps.microsoft.com/) usando su contraseña existente.

2. Seleccione el perfil que se encuentra en la parte superior derecha y luego elija **Perfil**.

3. Seleccione **Cambiar contraseña**.

4. Escriba la contraseña anterior, establezca la nueva contraseña y confírmela, y luego seleccione **Enviar**.

### <a name="to-change-your-password-at-windows-sign-in"></a>Para cambiar la contraseña en el inicio de sesión de Windows

Si el administrador activa la funcionalidad, puede ver un vínculo a **Restablecer contraseña** en la pantalla de inicio de sesión de Windows 7, Windows 8, Windows 8.1 o Windows 10.

1. Seleccione el vínculo **Restablecer contraseña** para iniciar el proceso de restablecimiento de la contraseña sin tener que usar la experiencia normal basada en Web.

2. Confirme su identificador de usuario y seleccione **Siguiente**.

3. Seleccione un método de contacto para la comprobación y confírmelo. Si es necesario, elija una segunda opción de verificación diferente de la anterior y rellene la información necesaria.

4. En la página **Crear una nueva contraseña**, escriba la nueva contraseña, confírmela y luego seleccione **Siguiente**.

    Normalmente, las contraseñas seguras tienen entre ocho y dieciséis caracteres, incluidos los caracteres en mayúsculas y minúsculas, al menos un número y al menos un carácter especial.

5. Después de obtener el mensaje que confirma que se ha restablecido su contraseña, puede seleccionar **Finalizar**.

    Si todavía no puede acceder a su cuenta, debe ponerse en contacto con el administrador de la organización para obtener más ayuda.

## <a name="common-problems-and-their-solutions"></a>Problemas comunes y sus soluciones

A continuación se presentan algunos casos de errores comunes y sus soluciones:

|Problema|Descripción|Solución|
| --- | --- | --- |
|Cuando intento cambiar la contraseña, me aparece un error. |La contraseña contiene una palabra, frase o patrón que permite adivinar la contraseña fácilmente.| Inténtelo de nuevo con una contraseña más segura.|
|Después de escribir mi identificador de usuario, se me remite a una página que dice "Póngase en contacto con el administrador".|Microsoft ha determinado que la contraseña de su cuenta de usuario la administra su administrador en un entorno local. Por tanto, no puede restablecer la contraseña desde el vínculo "No se puede tener acceso a su cuenta". |Póngase en contacto con el administrador para obtener más ayuda.|
|Después de escribir mi identificador de usuario, recibo el error "Su cuenta no está habilitada para el restablecimiento de contraseña".|El administrador no ha configurado su cuenta, por lo que puede restablecer la contraseña.|Su administrador no ha habilitado el restablecimiento de contraseña para su organización desde el vínculo "No se puede tener acceso a su cuenta", o bien no le ha autorizado para usar la característica.<br><br> Para restablecer la contraseña, debe seleccionar el vínculo "Póngase en contacto con un administrador" para enviar un correo electrónico al administrador de la compañía y decirle que quiere restablecer su contraseña.|
|Después de escribir mi identificador de usuario, obtengo un error que dice "No hemos podido verificar su cuenta".|El proceso de inicio de sesión no pudo verificar la información de la cuenta.|Este mensaje puede aparecer por dos motivos.<br><br>1. El administrador activó el restablecimiento de contraseña para la organización, pero no la registró para usar el servicio. Para registrarse para un restablecimiento de contraseña, consulte uno de los siguientes artículos, según su método de verificación: [Configuración de la información de seguridad para usar una aplicación autenticadora (versión preliminar)](security-info-setup-auth-app.md), [Configuración de la información de seguridad para usar una llamada telefónica (versión preliminar)](security-info-setup-phone-number.md), [Configuración de la información de seguridad para usar la mensajería de texto (versión preliminar)](security-info-setup-text-msg.md), [Configuración de la información de seguridad para usar el correo electrónico (versión preliminar)](security-info-setup-email.md) o [Configuración de la información de seguridad para usar preguntas de seguridad (versión preliminar)](security-info-setup-questions.md).<br><br>2. El administrador no activado el restablecimiento de contraseña para su organización. En esta situación, debe seleccionar el vínculo "Póngase en contacto con un administrador" para enviar un correo electrónico a la organización, a fin de pedirle que restablezca su contraseña.|

## <a name="next-steps"></a>Pasos siguientes

- Consulte los aspectos de la información de seguridad en el artículo [Introducción a la información de seguridad (versión preliminar)](./security-info-setup-signin.md).

- Si quiere volver a una cuenta personal, como Xbox, hotmail.com o outlook.com, pruebe las sugerencias que se encuentran en el [artículo Cuando no consigues iniciar sesión en tu cuenta Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).