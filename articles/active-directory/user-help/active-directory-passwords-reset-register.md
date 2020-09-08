---
title: 'Registro de la información de autenticación para llevar a cabo el restablecimiento de la contraseña: Azure AD'
description: Registre la información sobre su método de verificación para el autoservicio de restablecimiento de contraseña de Azure AD. De este modo, podrá restablecer su propia contraseña sin la ayuda del administrador.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.assetid: ''
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 05/28/2020
ms.author: curtand
ms.openlocfilehash: e460f3fd383286b1e61a979b87a9dfa09f272313
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799033"
---
# <a name="register-your-verification-method-info-to-reset-your-own-password"></a>Registro de la información del método de verificación para restablecer la propia contraseña

Si alguna vez ha olvidado su contraseña en el trabajo o en el centro educativo, en su organización no ha tenido nunca contraseña o le han bloqueado la cuenta, puede usar la información de seguridad y el dispositivo móvil para restablecer su contraseña profesional o educativa.

Para poder registrar sus datos y restablecer su contraseña es preciso que el administrador active esta característica. Si no ve la opción **He olvidado mi contraseña**, significa que el administrador no ha activado la característica para su organización. Si cree que no es correcto, solicite ayuda al departamento de soporte técnico.

>[!Important]
>Este artículo está destinado a los usuarios que intentan utilizar el registro para el autoservicio de restablecimiento de contraseña. Esto significa que podrá restablecer su contraseña profesional o educativa (por ejemplo, alain@contoso.com), sin necesidad de pedir ayuda al administrador. Si es un administrador que desea obtener información acerca de cómo activar el autoservicio de restablecimiento de contraseña para los empleados u otros usuarios, consulte [Implementar el autoservicio de restablecimiento de contraseña de Azure AD y otros artículos](../authentication/howto-sspr-deployment.md).

## <a name="set-up-your-password-reset-verification-method"></a>Configuración del método de verificación de restablecimiento de contraseña

1. Abra el explorador web en el dispositivo y vaya a la [página de información de seguridad](https://account.activedirectory.windowsazure.com/PasswordReset/Register.aspx?regref=ssprsetup).

2. En función de cómo haya configurado el administrador la organización, una o varias de las opciones siguientes estarán disponibles para su configuración como método de verificación de seguridad. Si hay varias opciones disponibles, es muy recomendable usar más de una como método de verificación de seguridad, en caso de que uno de los métodos deje de estar disponible.

    - **Aplicación de autenticación.** Elija usar la aplicación Microsoft Authenticator u otra aplicación autenticadora como método de verificación de seguridad. Para más información sobre la configuración de la aplicación, consulte [Configuración de la aplicación Microsoft Authenticator como método de comprobación](security-info-setup-auth-app.md).

    - **Mensajes de texto.** Elija enviar mensajes de texto a su dispositivo móvil. Para más información acerca de cómo configurar la mensajería de texto, consulte [Configuración de mensajería de texto como su método de comprobación](security-info-setup-text-msg.md).

    - **Llamadas de teléfono.** Elija recibir una llamada de teléfono en el número de teléfono registrado. Para más información acerca de cómo configurar llamadas telefónicas, consulte [Configuración de un número de teléfono como método de comprobación](security-info-setup-phone-number.md).

    - **Clave de seguridad.** Elija esta opción para usar una clave de seguridad compatible con Microsoft. Para más información, consulte [Configuración de una clave de seguridad como su método de comprobación](security-info-setup-security-key.md).

    - **Dirección de correo electrónico.** Elija usar una dirección de correo electrónico alternativa que se pueda utilizar sin requerir la contraseña olvidada o perdida. Esto solo funciona para el restablecimiento de la contraseña, no como método de verificación de seguridad. Para más información acerca de la configuración de una dirección de correo electrónico, consulte [Configuración de una dirección de correo electrónico como método de comprobación](security-info-setup-email.md).

    - **Preguntas de seguridad.** Elija configurar y responder a preguntas de seguridad predefinidas configuradas por el administrador. Esto solo funciona para el restablecimiento de la contraseña, no como método de verificación de seguridad. Para más información acerca de las preguntas de seguridad, consulte [Configuración de preguntas de seguridad como método de comprobación](security-info-setup-questions.md).

3. Después de seleccionar y configurar los métodos, elija **Finalizar** para completar el proceso.

    > [!Note]
    > La información agregada para el número de teléfono o la dirección de correo electrónico no se comparte con el directorio global de su organización. Las únicas personas que pueden ver esta información son el propio usuario y el administrador. Solo el propio usuario puede ver las respuestas a las preguntas de seguridad.

## <a name="common-problems-and-their-solutions"></a>Problemas comunes y sus soluciones

 A continuación se presentan algunos casos de errores comunes y sus soluciones:

| Mensaje de error |  Posible solución |
| --- | --- | --- |
| Póngase en contacto con el administrador.<br>Hemos detectado que la contraseña de su cuenta de usuario no está administrada por Microsoft. Como consecuencia, no podemos restablecer automáticamente su contraseña.<br>Póngase en contacto con el personal de TI para obtener ayuda adicional.| Si recibe este mensaje de error después de escribir el identificador de usuario, significa que su organización administra internamente la contraseña y no desea que usted pueda restablecer la contraseña desde el vínculo **No se puede tener acceso a la cuenta**. Para restablecer la contraseña en esta situación, debe ponerse en contacto con el departamento de soporte técnico de su organización o con el administrador para obtener ayuda. |
| La cuenta no está habilitada para restablecer la contraseña.<br>Su personal de TI no ha configurado la cuenta para utilizarla con este servicio.<br>Si lo desea, podemos ponernos en contacto con un administrador de su organización para que restablezca la contraseña. | Si recibe este mensaje de error después de escribir el identificador de usuario, significa que su organización no ha activado la característica de restablecimiento de contraseña o que no está autorizado para usarla. Para restablecer la contraseña, seleccione el vínculo **Póngase en contacto con un administrador**. Después de hacer clic en el vínculo, se envía un mensaje de correo electrónico al departamento de soporte técnico de su organización o al administrador, que les informa que desea restablecer la contraseña. |
| No se ha podido comprobar su cuenta.<br>Si lo desea, podemos ponernos en contacto con un administrador de su organización para que restablezca la contraseña. | Si recibe este mensaje de error después de escribir el identificador de usuario, significa que su organización ha activado el restablecimiento de contraseña y que puede usarlo, pero que no se ha registrado para el servicio. En esta situación, debe ponerse en contacto con el departamento de soporte técnico de su organización o el administrador para restablecer la contraseña. Para más información sobre cómo registrarse para restablecer la contraseña después de volver al dispositivo, consulte el proceso anterior en este artículo. |

## <a name="next-steps"></a>Pasos siguientes

- [Cambiar la contraseña mediante el autoservicio de restablecimiento de contraseñas](active-directory-passwords-update-your-own-password.md)

- [Página Información de seguridad](https://mysignins.microsoft.com/security-info)

- [Portal de restablecimiento de contraseña](https://passwordreset.microsoftonline.com/)

- [Cuando no puedes iniciar sesión en tu cuenta Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)