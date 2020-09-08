---
title: 'Cambio de la configuración y del método de verificación en dos fases: Azure Active Directory'
description: Obtenga información acerca de cómo cambiar el método de comprobación de seguridad y la configuración de su cuenta profesional o educativa en la página de comprobación de seguridad adicional.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: richagi
ms.assetid: d3372d9a-9ad1-4609-bdcf-2c4ca9679a3b
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 07/06/2020
ms.author: curtand
ms.openlocfilehash: e0a6c566e8e0dfb77b5899f735020d0f1facf3d1
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798387"
---
# <a name="change-your-two-factor-verification-method-and-settings"></a>Cambio de la configuración y del método de comprobación en dos pasos

Después de configurar los métodos de verificación de seguridad de su cuenta profesional o educativa, puede actualizar cualquiera de los detalles relacionados, entre los que se incluyen:

- Método de verificación de seguridad predeterminado

- Detalles del método de verificación de seguridad, por ejemplo, el número de teléfono

- Configuración de una aplicación de autenticación o eliminación de un dispositivo de esta

## <a name="using-the-additional-security-verification-page"></a>Uso de la página Comprobación de seguridad adicional

Si su organización le proporcionó pasos específicos sobre cómo activar y administrar la verificación en dos fases, primero debe seguir esas instrucciones. En caso contrario, puede acceder a la configuración del método de verificación de seguridad en la página [Comprobación de seguridad adicional](./multi-factor-authentication-end-user-first-time.md).

>[!Note]
>Si lo que ve en la pantalla no coincide con lo que se indica en este artículo, significa que el administrador ha activado la experiencia **Información de seguridad (versión preliminar)** o que la organización tiene su propio portal personalizado. Para más información sobre la nueva experiencia de información de seguridad, consulte [Introducción a la información de seguridad (versión preliminar)](./security-info-setup-signin.md). Para más información sobre el portal personalizado de su organización, debe ponerse en contacto con su departamento de soporte técnico.

### <a name="to-get-to-the-additional-security-verification-page"></a>Para ir a la página Comprobación de seguridad adicional

Puede seguir este vínculo a la [página Comprobación de seguridad adicional](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1).

![Página Comprobación de seguridad adicional, con los detalles del método de verificación de seguridad disponibles](./media/multi-factor-authentication-end-user-manage-settings/mfa-security-verification-page.png)

También puede ir a la página **Comprobación de seguridad adicional** siguiendo estos pasos:

1. Inicie sesión en [https://myapps.microsoft.com](https://myapps.microsoft.com).

1. Seleccione el nombre de cuenta en la parte superior derecha y, después, seleccione **Perfil**.

1. Seleccione **Comprobación de seguridad adicional**.  

    ![Vínculo Aplicaciones en la página Comprobación de seguridad adicional](./media/multi-factor-authentication-end-user-manage-settings/mfa-myapps-link.png)

>[!Note]
>Para más información sobre cómo uasr la sección **Contraseñas de aplicaciones** de la página **Comprobación de seguridad adicional**, consulte [Administración de contraseñas de aplicación para la verificación en dos fases](multi-factor-authentication-end-user-app-passwords.md). Las contraseñas de aplicaciones solo se deben usar si las aplicaciones no admiten la verificación en dos fases.

## <a name="change-your-default-security-verification-method"></a>Cambio del método de verificación de seguridad predeterminado

Después de iniciar sesión en su cuenta profesional o educativa con su nombre de usuario y contraseña, se le presentará automáticamente el método de verificación de seguridad elegido. En función de los requisitos de su organización, puede tratarse de una notificación o un código de verificación a través de una aplicación de autenticación, un mensaje de texto o una llamada de teléfono.

Si decide cambiar el método de verificación de seguridad predeterminado que está usando, puede hacerlo desde aquí.

### <a name="to-change-your-default-security-verification-method"></a>Cambio del método de verificación de seguridad predeterminado

1. En la página **Comprobación de seguridad adicional**, seleccione el método que desea usar en la lista desplegable **What's your preferred option** (¿Cuál es su opción preferida?). Verá todas las opciones, pero solo podrá elegir aquellas que la organización ponga a su disposición.

    - **Notificarme a través de la aplicación**: Se le notificará a través de la aplicación de autenticación que tiene un aviso de verificación en espera.

    - **Llamar a mi teléfono de autenticación**: Recibirá una llamada de teléfono en el dispositivo móvil pidiéndole que compruebe su información.

    - **Enviar un código a mi teléfono de autenticación**: Obtendrá un código de verificación como parte de un mensaje de texto en el dispositivo móvil. Debe escribir este código en el aviso de verificación de su cuenta profesional o educativa.

    - **Llamar al teléfono del trabajo**: Recibirá una llamada en el teléfono del trabajo pidiéndole que compruebe su información.

    - **Usar código de comprobación de la aplicación**: Use la aplicación de autenticación para obtener un código de verificación que escribirá en el aviso de verificación de su cuenta profesional o educativa.

2. Seleccione **Guardar**.

## <a name="add-or-change-your-phone-number"></a>Adición o cambio del número de teléfono

Puede agregar nuevos números de teléfono o actualizar los que ya existen en la página **Comprobación de seguridad adicional**.

>[!Important]
>Es muy recomendable que agregue un número de teléfono secundario para evitar que se bloquee su cuenta si pierde o le roban el teléfono principal, o si adquiere un nuevo teléfono y ya no tiene el número de teléfono principal original.

### <a name="to-change-your-phone-numbers"></a>Cambio de los números de teléfono

1. En la sección **¿Cómo desea responder?** de la página **Comprobación de seguridad adicional**, actualice el número de teléfono de **Teléfono de autenticación** (su dispositivo móvil principal) y **Teléfono de la oficina**.

1. Seleccione la casilla situada junto a la opción **Teléfono de autenticación alternativo** y, a continuación, escriba un número de teléfono secundario en el que pueda recibir llamadas telefónicas si no puede acceder al dispositivo principal.

1. Seleccione **Guardar**.

## <a name="add-a-new-account-to-the-microsoft-authenticator-app"></a>Adición de una nueva cuenta a la aplicación Microsoft Authenticator

Puede configurar su cuenta profesional o educativa en la aplicación Microsoft Authenticator para [Android](https://play.google.com/store/apps/details?id=com.azure.authenticator) o [iOS](https://apps.apple.com/app/microsoft-authenticator/id983156458).

Si ya ha configurado su cuenta profesional o educativa en la aplicación Microsoft Authenticator, no es necesario que lo haga de nuevo.

1. En la sección **¿Cómo desea responder?** de la página **Comprobación de seguridad adicional**, seleccione **Set up Authenticator app** (Configurar aplicación Authenticator).

    ![Configuración de la cuenta profesional o educativa en la aplicación Microsoft Authenticator](./media/multi-factor-authentication-end-user-manage-settings/mfa-security-verification-page-auth-app.png)

1. Siga las instrucciones que aparecen en pantalla, incluido el uso del dispositivo móvil para digitalizar el código QR y, después, seleccione **Siguiente**.

    Se le pedirá que apruebe una notificación mediante la aplicación Microsoft Authenticator para comprobar su información.

1. Seleccione **Guardar**.

## <a name="delete-your-account-or-device-from-the-microsoft-authenticator-app"></a>Eliminación de la cuenta o el dispositivo de la aplicación Microsoft Authenticator

Puede eliminar la cuenta de la aplicación Microsoft Authenticator y puede eliminar el dispositivo de su cuenta profesional o educativa. Normalmente, el dispositivo se elimina para quitar de forma permanente un dispositivo perdido, robado o antiguo de su cuenta, y se elimina la cuenta para tratar de corregir algunos problemas de conexión o para solucionar un cambio en la cuenta como, por ejemplo, un nuevo nombre de usuario.

### <a name="to-delete-your-device-from-your-work-or-school-account"></a>Eliminación del dispositivo de su cuenta profesional o educativa

1. En la sección **¿Cómo desea responder?** de la página **Comprobación de seguridad adicional**, seleccione el botón **Configurar aplicación autenticadora**.

1. Seleccione **Guardar**.

### <a name="to-delete-your-account-from-the-microsoft-authenticator-app"></a>Eliminación de la cuenta de la aplicación Microsoft Authenticator

En la aplicación Microsoft Authenticator, seleccione el botón **Eliminar** situado junto al dispositivo que desea eliminar.

## <a name="turn-on-two-factor-verification-prompts-on-a-trusted-device"></a>Activación de los avisos de verificación en dos fases en un dispositivo de confianza

Dependiendo de la configuración de la organización, es posible que vea una casilla que indica "Don't ask again for **X** days" (No volver a preguntar en X días) al realizar la verificación en dos fases en el explorador. Si ha seleccionado esta opción para detener los avisos de verificación en dos fases y posteriormente pierde el dispositivo o este puede estar en peligro, debe activar de nuevo la verificación en dos fases para ayudar a proteger su cuenta. Debe activar los avisos para todos los dispositivos al mismo tiempo. Lamentablemente, no puede volver a activar los avisos para un dispositivo específico.

### <a name="to-turn-two-factor-verification-prompts-back-on-for-your-devices"></a>Para volver a activar los avisos de verificación en dos fases en los dispositivos

En la página [**Comprobación de seguridad adicional**, seleccione ](#to-get-to-the-additional-security-verification-page)Restaurar **Multi-Factor Authentication en dispositivos en los que se confió anteriormente**. La próxima vez que inicie sesión en cualquier dispositivo, se le pedirá que realice la verificación en dos fases.

## <a name="next-steps"></a>Pasos siguientes

Después de agregar o actualizar la configuración de la verificación en dos fases, puede administrar las contraseñas de aplicación, iniciar sesión u obtener ayuda con algunos problemas comunes relacionados con ella.

- [Administración de contraseñas de aplicación para la verificación en dos fases](multi-factor-authentication-end-user-app-passwords.md) de las aplicaciones que no admiten la verificación en dos fases.

- [Inicio de sesión con la verificación en dos fases](multi-factor-authentication-end-user-signin.md)

- [Solución de problemas comunes con la verificación en dos fases](multi-factor-authentication-end-user-troubleshoot.md)