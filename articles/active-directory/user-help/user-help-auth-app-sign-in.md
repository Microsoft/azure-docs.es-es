---
title: Inicio de sesión con la Azure AD de la aplicación Microsoft Authenticator
description: Use la aplicación Microsoft Authenticator para iniciar sesión en su cuenta profesional o educativa o en sus cuentas personales, tanto de Microsoft como ajenas a Microsoft, y que se use la comprobación en dos fases o el inicio de sesión en el teléfono.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 06/03/2020
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: 19e441334395f600c3adaa867660345164537f96
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799458"
---
# <a name="sign-in-to-your-accounts-using-the-microsoft-authenticator-app"></a>Inicio de sesión en sus cuentas mediante la aplicación Microsoft Authenticator

La aplicación Microsoft Authenticator le ayuda a iniciar sesión en sus cuentas si usa la comprobación en dos fases. La comprobación en dos pasos le ayuda a acceder a sus cuentas de manera más segura, en especial cuando se examina información confidencial. Como las contraseñas se pueden olvidar, poner en peligro o robar, la comprobación en dos pasos es un paso de seguridad adicional que ayuda a proteger su cuenta ya que es más difícil que otras personas accedan a ella.

Puede usar la aplicación Microsoft Authenticator de varias formas, por ejemplo:

- Para proporcionar un aviso de un segundo método de comprobación después de iniciar sesión con su nombre de usuario y contraseña.

- Para proporcionar inicio de sesión sin necesidad de una contraseña, mediante su nombre de usuario y su dispositivo móvil con su huella digital, cara o PIN.

  >[!Important]
  >Este método de inicio de sesión en el teléfono solo funciona con las cuentas profesionales o educativas y personales de Microsoft. Las cuentas que no sean de Microsoft requieren que se use el proceso estándar de comprobación en dos fases.

## <a name="prerequisites"></a>Requisitos previos

Para poder usar la aplicación Microsoft Authenticator, es preciso:

 1. Descargar e instalar la aplicación Microsoft Authenticator. Si aún no lo ha hecho, consulte [Introducción a la aplicación Microsoft Authenticator](user-help-auth-app-download-install.md).

 2. Agregar sus cuentas profesional o educativa, personal y de terceros a la aplicación Microsoft Authenticator. Para más información, consulte [Adición de una cuenta profesional o educativa](user-help-auth-app-add-work-school-account.md), [Adición de una cuenta Microsoft personal](user-help-auth-app-add-personal-ms-account.md) y [Adición de una cuenta que no sea de Microsoft](user-help-auth-app-add-non-ms-account.md).

## <a name="turn-on-and-use-phone-sign-in-for-your-work-or-school-account"></a>Activación y uso del inicio de sesión en el teléfono en las cuentas profesionales o educativas

El inicio de sesión en el teléfono es un tipo de verificación en dos pasos. En este caso, para verificar su identidad debe especificar algo que sepa y algo que tenga, pero el inicio de sesión en el teléfono permite omitir el paso de escribir la contraseña de la cuenta y realiza toda la comprobación de la identidad en el dispositivo móvil.

Para poder activar el inicio de sesión en el teléfono, es preciso activar la comprobación en dos fases. Para más información acerca de cómo activar la comprobación en dos fases en una cuenta, consulte [Adición de una cuenta profesional o educativa](user-help-auth-app-add-work-school-account.md) y [Adición de una cuenta Microsoft personal](user-help-auth-app-add-personal-ms-account.md).

El inicio de sesión en el teléfono solo está disponible en dispositivos iOS y Android con Android 6.0, o cualquier versión posterior.

### <a name="turn-on-phone-sign-in"></a>Activación del inicio de sesión en el teléfono

Abra la aplicación Microsoft Authenticator, vaya a la cuenta profesional o educativa y active el inicio de sesión en el teléfono.

- **Al pulsar el icono de la cuenta**, se ve una vista de pantalla completa de la cuenta. Si ve **Inicio de sesión en el teléfono habilitado**, significa que se ha configurado para iniciar sesión sin contraseña. Si ve la opción **Habilitar inicio de sesión en el teléfono**, púlsela para activar el inicio de sesión en el teléfono.
- **Si ya ha usado la aplicación para la verificación de dos fases**, puede pulsar el icono de la cuenta para ver una vista de pantalla completa de la cuenta. A continuación, pulse **Habilitar inicio de sesión en el teléfono**, para activar el inicio de sesión en el teléfono.
- **Si no encuentra su cuenta profesional o educativa** en la pantalla **Cuentas** de la aplicación, significa que aún no la ha agregado a la aplicación. Agregue su cuenta profesional o educativa siguiendo los pasos descritos en la ayuda [Adición de una cuenta profesional o educativa](user-help-auth-app-add-work-school-account.md).

Después de activar el inicio de sesión en el teléfono, puede iniciar sesión solo con la aplicación Microsoft Authenticator. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en su cuenta profesional o educativa.

    Después de escribir el nombre de usuario, aparece la pantalla **Aprobar el inicio de sesión**, en la que se muestra un número de dos dígitos y en el que se le pide que inicie sesión con la aplicación Microsoft Authenticator. Si no desea usar este inicio de sesión en el método, puede seleccionar **Use su contraseña en su lugar** e inicie sesión con su contraseña.

    ![Cuadro Aprobar el inicio de sesión en el equipo](media/user-help-auth-app-sign-in/microsoft-auth-app-sign-in.png)

2. Abra la notificación o la aplicación Microsoft Authenticator en su dispositivo y, después, pulse el número que coincida con el número que aparece la pantalla **Aprobar el inicio de sesión** del equipo.

    ![Cuadro Aprobar el inicio de sesión en dispositivo](media/user-help-auth-app-sign-in/microsoft-auth-app-sign-in-numbers.png)

3. Elija **Aprobar** si reconoce el intento de inicio de sesión. De lo contrario, elija **Rechazar**.

4. Usar el PIN o la clave biométrica del teléfono para completar la autenticación.

## <a name="turn-on-and-use-phone-sign-in-for-your-personal-microsoft-accounts"></a>Activación y uso del inicio de sesión en el teléfono para cuentas personales de Microsoft

Puede activar el inicio de sesión con teléfono para su cuenta Microsoft personal, por ejemplo la cuenta que usa para iniciar sesión en Outlook.com, Xbox o Skype.

>[!NOTE]
>Para ayudar a proteger su cuenta, la aplicación Microsoft Authenticator requiere un PIN o bloqueo biométrico en el dispositivo. Si mantiene el teléfono desbloqueado, la aplicación le pide que configure un bloqueo de seguridad antes de activar el inicio de sesión con el teléfono.

### <a name="turn-on-phone-sign-in"></a>Activación del inicio de sesión en el teléfono 

Abra la aplicación Microsoft Authenticator, vaya a la cuenta profesional o educativa y active el inicio de sesión en el teléfono.

- **Al pulsar el icono de la cuenta**, se ve una vista de pantalla completa de la cuenta. Si ve **Inicio de sesión en el teléfono habilitado**, significa que se ha configurado para iniciar sesión sin contraseña. Si ve la opción **Habilitar inicio de sesión en el teléfono**, púlsela para activar el inicio de sesión en el teléfono.
- **Si ya usa la aplicación para la verificación de dos fases**, puede pulsar el icono de la cuenta para ver una vista de pantalla completa de la cuenta. A continuación, pulse **Habilitar inicio de sesión en el teléfono**, para activar el inicio de sesión en el teléfono.
- **Si no encuentra su cuenta** en la pantalla **Cuentas** de la aplicación, significa que aún no la ha agregado a la aplicación. Agregue su cuenta personal de Microsoft, para lo que debe seguir los pasos que se describen en el artículo [Incorporación de cuentas personales Microsoft](user-help-auth-app-add-personal-ms-account.md).

### <a name="sign-in-to-your-account-using-phone-sign-in"></a>Inicio de sesión en su cuenta con ayuda del teléfono

1. Vaya a la página de inicio de sesión de su cuenta personal de Microsoft y, en lugar de escribir su contraseña, seleccione el vínculo **Usar la aplicación Microsoft Authenticator en su lugar**.

    Microsoft le enviará una notificación al teléfono.

2. Apruebe la notificación.

## <a name="sign-in-using-two-factor-verification-for-your-account"></a>Inicie sesión con la comprobación en dos fases en la cuenta

El método de verificación en dos fases requiere que escriba su nombre de usuario y contraseña en el dispositivo en el que va a iniciar sesión y luego elija si la aplicación Microsoft Authenticator recibe una notificación, o si quiere copiar el código de verificación de la aplicación Microsoft Authenticator. En un dispositivo Android, los códigos de verificación se pueden encontrar en la pantalla **Cuentas**. En un dispositivo iOS, estos códigos de verificación se pueden encontrar en la pantalla **Cuentas** o en la vista de pantalla completa de una cuenta, en función del tipo de cuenta. Active la verificación en dos fases para la cuenta al agregar la cuenta a la aplicación Microsoft Authenticator.

>[!Note]
>Si no ve su cuenta profesional o educativa, o su cuenta personal en la pantalla **Cuentas** de la aplicación Microsoft Authenticator, significa que no ha agregado la cuenta a la aplicación Microsoft Authenticator. Para agregar su cuenta, consulte [Adición de una cuenta profesional o educativa](user-help-auth-app-add-work-school-account.md) y [Adición de una cuenta Microsoft personal](user-help-auth-app-add-personal-ms-account.md).

Para conocer los pasos necesarios para iniciar sesión en su cuenta profesional o educativa, o en su cuenta personal usando los distintos métodos de la comprobación en dos fases, consulte [Inicio de sesión con la información de seguridad o la verificación en dos pasos](user-help-sign-in.md).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

| Pregunta | Solución |
| -------- | -------- |
| ¿Cómo es que iniciar sesión con el teléfono sea más seguro que escribir una contraseña? | Hoy en día mayoría de los usuarios inician sesión en sitios web o aplicaciones con nombre de usuario y contraseña. Desafortunadamente, las contraseñas se pueden perder, las roban o las adivinan los piratas informáticos.<br><br>Después de configurar la aplicación Microsoft Authenticator, esta crea una clave en el teléfono para desbloquear la cuenta que está protegida mediante el PIN o bloqueo biométrico de dicho teléfono. Después, esta clave se usa para demostrar su identidad al iniciar sesión.<br><br>**Importante**<br>Los datos solo se usan para proteger la clave de forma local. Nunca se envían a la nube o se almacenan en esta. |
| ¿Cómo reemplaza el inicio de sesión en el teléfono a la verificación en dos pasos? ¿Debo apagarla? | El inicio de sesión en el teléfono es un tipo de verificación en dos pasos en el que ambos pasos se realizan en el dispositivo móvil. Debe mantener la verificación en dos pasos activada para ayudar a proporcionar más seguridad a su cuenta. |
| Si mantengo activa la verificación en dos pasos para mi cuenta, ¿tendré que aprobar dos notificaciones? | No. El inicio de sesión en su cuenta Microsoft con el teléfono también cuenta como verificación en dos pasos, así que no se requiere ninguna segunda aprobación. |
| ¿Qué ocurre si pierdo mi teléfono o no lo tengo conmigo? ¿Cómo puedo acceder a mi cuenta? | Siempre puede seleccionar el uso de una contraseña en lugar del vínculo en la página de inicio de sesión para volver a utilizar la contraseña. Sin embargo, si usa la verificación en dos pasos, seguirá siendo necesario un segundo método para verificar la identidad.<br><br>**Importante**<br>Es muy recomendable que tenga más de un método de verificación asociado con su cuenta y que dicho método esté actualizado.<br><br>Puede administrar sus métodos de verificación para las cuentas personales de la página [Configuración de seguridad](https://account.live.com/proofs/manage). Para cuentas profesionales o educativas, puede ir a la página [Comprobación de seguridad adicional](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1) de su organización o a la página **Garantizar la seguridad de la cuenta** si el administrador ha activado la información de seguridad. Para más información sobre seguridad, consulte [Introducción a la información de seguridad (versión preliminar)](./security-info-setup-signin.md).<br><br>Si no puede administrar sus métodos de verificación, debe ponerse en contacto con su administrador. |
| ¿Cómo dejo de usar esta característica y vuelvo a usar mi contraseña? | Para las cuentas personales, seleccione el vínculo **Use su contraseña en su lugar** durante el inicio de sesión. Su elección más reciente se recuerda y se ofrece de forma predeterminada la próxima vez que inicie sesión. Si desea volver a usar el inicio de sesión con el teléfono, seleccione el vínculo **Use una aplicación en su lugar** durante el inicio de sesión.<br><br>Si la cuenta es profesional o educativa, debe anular el registro del dispositivo desde la página **Settings** (Configuración) de la aplicación Microsoft Authenticator o deshabilitar el dispositivo desde el área **Devices & activity** (Dispositivos y actividad) del perfil. Para más información sobre cómo deshabilitar el dispositivo de su perfil, consulte [Actualización de la información del perfil y de la cuente desde el portal Mis aplicaciones](my-apps-portal-end-user-update-profile.md#view-your-organization-related-profile-information). |
| ¿Por qué no puedo usar más de una cuenta profesional o educativa para el inicio de sesión con el teléfono? | Debe registrarse un teléfono en una única cuenta profesional o educativa. Si desea activar el inicio de sesión con el teléfono para una cuenta profesional o educativa diferente, debe anular el registro de la cuenta en el dispositivo desde la página **Configuración**. |
| ¿Puedo iniciar sesión en mi equipo con mi teléfono? | En el caso de su equipo, se recomienda iniciar sesión mediante Windows Hello en Windows 10. Windows Hello le permite usar su cara, la huella digital o el PIN para iniciar sesión. |

## <a name="next-steps"></a>Pasos siguientes

- Si tiene dificultades para obtener el código de verificación de su cuenta Microsoft personal, consulte la sección **Solución de problemas de código de seguridad** del artículo [Información de seguridad y códigos de seguridad de la cuenta de Microsoft](https://support.microsoft.com/help/12428/microsoft-account-security-info-verification-codes).

- Si tiene más preguntas generales sobre la aplicación, consulte [Preguntas más frecuentes de la aplicación Microsoft Authenticator](user-help-auth-app-faq.md).

- Si desea más información sobre la verificación en dos pasos, vea [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

- Para más información sobre seguridad, consulte [Introducción a la información de seguridad (versión preliminar)](./security-info-setup-signin.md).