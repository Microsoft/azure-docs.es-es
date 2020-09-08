---
title: 'Inicio de sesión con la información de autenticación de identidad: Azure AD'
description: Obtenga información sobre cómo iniciar sesión con los distintos métodos de verificación de identidad en la información de seguridad.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: librown
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 07/18/2019
ms.author: curtand
ms.custom: user-help
ms.openlocfilehash: 42ced6632ebfa56af8fe13a02f531b9835e13df3
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799390"
---
# <a name="sign-in-using-two-step-verification-or-security-info"></a>Inicio de sesión con la información de seguridad o la verificación en dos pasos

Una vez configurada la verificación en dos pasos o la información de seguridad, podrá iniciar sesión en la cuenta con el método de verificación que especifique.

> [!Note]
> Si sigue usando la experiencia de verificación en dos pasos, deberá configurar los métodos de verificación siguiendo las instrucciones del artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).
>
> Si el administrador ha activado la experiencia de información de seguridad, deberá establecer los métodos de verificación con estos artículos detallados:<ul><li>[Configuración de la información de seguridad para usar una aplicación autenticadora](security-info-setup-auth-app.md)</li><li>[Configuración de la información de seguridad para usar la mensajería de texto](security-info-setup-text-msg.md)</li><li>[Configuración de la información de seguridad para usar las llamadas de teléfono](security-info-setup-phone-number.md)</li><li>[Configuración de la información de seguridad para usar una clave de seguridad](security-info-setup-security-key.md)</li></ul>

## <a name="sign-in-using-an-authenticator-app-notification-on-your-mobile-device"></a>Inicio de sesión con una notificación de la aplicación Authenticator en el dispositivo móvil

1. Inicie sesión en la cuenta con su nombre de usuario y contraseña.

2. Seleccione **Aprobar** en la notificación de aprobación enviada a su dispositivo móvil.

## <a name="sign-in-using-an-authenticator-app-code-on-your-mobile-device"></a>Inicio de sesión con un código de la aplicación Authenticator en el dispositivo móvil

1. Inicie sesión en la cuenta con su nombre de usuario y contraseña.

2. Abra la aplicación Authenticator y escriba el código generado aleatoriamente para la cuenta en el cuadro **Especificar el código**.

## <a name="sign-in-using-your-phone-number"></a>Inicio de sesión con su número de teléfono

1. Inicie sesión en la cuenta con su nombre de usuario y contraseña.

2. Responda al teléfono y siga las instrucciones.

## <a name="sign-in-using-a-text-message"></a>Inicio de sesión con un mensaje de texto

1. Inicie sesión en la cuenta con su nombre de usuario y contraseña.

2. Abra el mensaje de texto y escriba el código del mensaje de texto en el cuadro **Especificar el código**.

## <a name="sign-in-using-a-security-key-at-the-lock-screen"></a>Inicio de sesión con una clave de seguridad en la pantalla de bloqueo

1. Una vez que registre la clave de seguridad, seleccione la imagen de la clave de seguridad en la pantalla de bloqueo de Windows 10.

2. Inserte la clave de seguridad en el puerto USB del dispositivo e inicie sesión en Windows con el PIN de la clave de seguridad.

    ![Inicio de sesión de una clave de seguridad en la pantalla de bloqueo de Windows 10](./media/security-info/security-info-windows-10-lock-screen-security-key.png)

## <a name="sign-in-using-a-security-key-and-the-microsoft-edge-browser"></a>Inicio de sesión con una clave de seguridad y el explorador Microsoft Edge

1. Una vez que registre la clave de seguridad, abra el explorador Microsoft Edge.

2. Cuando se le pida iniciar sesión, inserte la clave de seguridad en el puerto USB del dispositivo e inicie sesión en Windows con el PIN de la clave de seguridad.

    ![Inicio de sesión de una clave de seguridad con el explorador Microsoft Edge](./media/security-info/security-info-edge-security-key.png)

    >[!NOTE]
    >Para obtener información sobre el inicio de sesión mediante la aplicación Microsoft Authenticator, vea el artículo [Inicio de sesión en sus cuentas mediante la aplicación Microsoft Authenticator](user-help-auth-app-sign-in.md).

## <a name="sign-in-using-another-verification-method"></a>Inicio de sesión con otro método de verificación

Si por algún motivo no puede usar su método de inicio de sesión principal, puede usar otro método de verificación configurado previamente.

1. Inicie sesión en la cuenta de la forma habitual y luego elija el vínculo **Iniciar sesión de otro modo** en la página **Verificación en dos pasos**.

    ![Cambio del método de comprobación de inicio de sesión](media/security-info/two-factor-auth-signin-another-way.png)

    >[!Note]
    >Si no ve el vínculo **Iniciar sesión de otro modo**, significa que no ha configurado ningún otro método de verificación y que tendrá que ponerse en contacto con el administrador para solicitar ayuda para iniciar sesión en su cuenta. Después de que el administrador le ayude a iniciar sesión, asegúrese de agregar métodos de comprobación adicionales. Para más información sobre cómo agregar métodos de verificación, consulte el artículo [Administración de la configuración de la verificación en dos pasos](multi-factor-authentication-end-user-manage-settings.md).
    >
    >Si ve el vínculo **Iniciar sesión de otro modo** pero sigue sin ver ningún otro método de verificación, tendrá que ponerse en contacto con el administrador para que lo ayude a iniciar sesión en su cuenta.

2. Elija el método de verificación alternativo y siga con el proceso de verificación en dos pasos.

3. Una vez de vuelta en su cuenta, podrá actualizar los métodos de verificación (si es necesario). Para más información sobre cómo agregar o cambiar los métodos, consulte el artículo [Administración de la configuración de la verificación en dos pasos en dos pasos](multi-factor-authentication-end-user-manage-settings.md).

## <a name="next-steps"></a>Pasos siguientes

- Consulte los aspectos de la información de seguridad en el artículo [Introducción a la información de seguridad (versión preliminar)](./security-info-setup-signin.md).

- Obtenga información sobre la verificación en dos pasos en el artículo [Introducción a la verificación en dos pasos](./multi-factor-authentication-end-user-first-time.md).

- Si ha perdido u olvidado la contraseña, restablézcala en el [portal de restablecimiento de contraseña](https://passwordreset.microsoftonline.com/).

- Obtenga soluciones de problemas, sugerencias y ayuda para los problemas de inicio de sesión en el artículo [Cuando no puedes iniciar sesión en tu cuenta de Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).