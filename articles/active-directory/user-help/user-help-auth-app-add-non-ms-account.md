---
title: 'Incorporación de cuentas que no son de Microsoft a la aplicación Microsoft Authenticator: Azure AD'
description: Agregue cuentas que no son de Microsoft, como Google o Facebook, a la aplicación Microsoft Authenticator para comprobar su identidad mientras usa la verificación en dos fases.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 01/24/2019
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: 04076ed5b91f8e840a084b784477d07eea5adee7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015977"
---
# <a name="add-non-microsoft-accounts-to-the-microsoft-authenticator-app"></a>Incorporación de cuentas que no son de Microsoft a la aplicación Microsoft Authenticator

Agregue sus cuentas que no sean de Microsoft, como las de Google, Facebook o GitHub, a la aplicación Microsoft Authenticator para la verificación en dos fases. La aplicación Microsoft Authenticator funciona con cualquier aplicación que utilice la verificación de dos fases y con cualquier cuenta que admita los estándares de contraseña de un solo uso y duración definida (TOTP).

>[!Important]
>Antes de agregar su cuenta, debe descargar e instalar la aplicación Microsoft Authenticator. Si no lo ha hecho aún, siga los pasos descritos en el artículo para [descargar e instalar la aplicación](user-help-auth-app-download-install.md).

## <a name="add-personal-accounts"></a>Adición de cuentas personales

Por lo general, para todas las cuentas personales, debe realizar los siguientes pasos:

1. Inicie sesión en la cuenta y, a continuación, active la verificación en dos fases con el dispositivo o su equipo.

2. Agregue la cuenta a la aplicación Microsoft Authenticator. Puede que se le pida que lea un código QR como parte de este proceso.

    >[!Note]
    >Si es la primera vez que configura la aplicación Microsoft Authenticator, es posible que reciba un mensaje en el que se le pregunta si quiere permitir que la aplicación acceda a la cámara (iOS) o permitir que la aplicación tome fotografías y grabe vídeo (Android). Debe seleccionar **Permitir** para que la aplicación de autenticación pueda acceder a la cámara y tomar una fotografía del código QR en el paso siguiente. Aunque no otorgue permiso a la cámara, podrá configurar la aplicación de autenticación, pero tendrá que agregar la información de código manualmente. Para más información sobre cómo agregar manualmente código, consulte [Agregar manualmente una cuenta a la aplicación](user-help-auth-app-add-account-manual.md).

En este artículo, ofrecemos el proceso para las cuentas de Facebook, Google, GitHub y Amazon, pero dicho proceso es el mismo para cualquier otra aplicación, como Adobe, Netflix o Instagram.

## <a name="add-your-google-account"></a>Adición de la cuenta de Google

Puede agregar su cuenta de Google activando primero la verificación en dos fases y, luego, agregando la cuenta a la aplicación.

### <a name="turn-on-two-factor-verification"></a>Activación de la verificación en dos fases

1. En el equipo, vaya a https://myaccount.google.com/signinoptions/two-step-verification/enroll-welcome, seleccione **Empezar** y, a continuación, verifique su identidad.

2. Siga los pasos que se muestran en la página para activar la verificación en dos fases para su cuenta personal de Google.

### <a name="add-your-google-account-to-the-app"></a>Adición de la cuenta de Google a la aplicación

1. En la página de seguridad de la cuenta de Google del equipo (https://myaccount.google.com/security), vaya a la sección **Add more second steps to verify it's you** (Agregar más segundos pasos para comprobar su identidad) y elija **Set up** (Configurar) en la sección **Authenticator app** (Aplicación Authenticator).

2. En la página **Get codes from the Authenticator app** (Obtener códigos de la aplicación autenticadora), seleccione **Android** o **iPhone** según su tipo de teléfono y, a continuación, seleccione **Siguiente**.

    Se le asignará un código QR que puede usar para asociar automáticamente la cuenta con la aplicación Microsoft Authenticator. No cierre esta ventana.

3. Abra la aplicación Microsoft Authenticator, seleccione **Agregar cuenta** en el icono **Personalizar y controlar** en la esquina superior derecha y, a continuación, seleccione **Otra cuenta (Google, Facebook, etc.)** .

4. Use la cámara del dispositivo para detectar el código QR desde la página **Configurar Authenticator** en el equipo.

    >[!Note]
    >Si la cámara no funciona correctamente, puede escribir manualmente la dirección URL y el código QR.

5. Revise la página **Cuentas** de la aplicación Microsoft Authenticator en el dispositivo para asegurarse de que la información de la cuenta es correcta y de que hay un código de verificación de seis dígitos asociado.

    Para mayor seguridad, el código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces.

6. Seleccione **Siguiente** en la página **Configurar Authenticator** del equipo, escriba el código de verificación de seis dígitos que se proporciona en la aplicación de su cuenta de Google y, a continuación, seleccione **Verify** (Comprobar).

7. Se ha comprobado la cuenta y puede seleccionar **Listo** para cerrar la página **Configurar Authenticator**.

    >[!NOTE]
    >Para obtener más información acerca de la verificación en dos fases y su cuenta de Google, consulte [Activar la verificación en dos pasos](https://support.google.com/accounts/answer/185839) y [Más información acerca de la verificación en dos pasos](https://www.google.com/landing/2step/help.html).

## <a name="add-your-facebook-account"></a>Adición de la cuenta de Facebook

Puede agregar su cuenta de Facebook activando primero la verificación en dos fases y, luego, agregando la cuenta a la aplicación.

### <a name="turn-on-two-factor-verification"></a>Activación de la verificación en dos fases

1. En el equipo, abra Facebook, seleccione el menú desplegable en la esquina superior derecha y, a continuación, vaya a **Configuración** > **Seguridad e inicio de sesión**.

    Se muestra la página **Seguridad e inicio de sesión**.

2. Vaya a la opción **Usar la autenticación en dos pasos** en la sección **Autenticación en dos pasos** y, a continuación, seleccione **Editar**.

    Se muestra la página **Autenticación en dos pasos**.

3. Seleccione **Empezar**.

### <a name="add-your-facebook-account-to-the-app"></a>Adición de la cuenta de Facebook a la aplicación

1. En la página de Facebook del equipo, vaya a la sección **Add a backup** (Agregar una copia de seguridad) y, a continuación, elija **Configurar** en el área **Aplicación de autenticación**.

    Se le asignará un código QR que puede usar para asociar automáticamente la cuenta con la aplicación Microsoft Authenticator. No cierre esta ventana.

2. Abra la aplicación Microsoft Authenticator, seleccione **Agregar cuenta** en el icono **Personalizar y controlar** en la esquina superior derecha y, a continuación, seleccione **Otra cuenta (Google, Facebook, etc.)** .

3. Use la cámara del dispositivo para detectar el código QR desde la página **Autenticación en dos pasos** en el equipo.

    >[!Note]
    >Si la cámara no funciona correctamente, puede escribir manualmente la dirección URL y el código QR.

4. Revise la página **Cuentas** de la aplicación Microsoft Authenticator en el dispositivo para asegurarse de que la información de la cuenta es correcta y de que hay un código de verificación de seis dígitos asociado.

    Para mayor seguridad, el código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces.

5. Seleccione **Siguiente** en la página **Autenticación en dos fases** del equipo y, a continuación, escriba el código de verificación de seis dígitos que se proporciona en la aplicación de su cuenta de Facebook.

    Se verifica la cuenta y, a continuación, ya puede usar la aplicación para verificar su cuenta.

    >[!NOTE]
    >Para obtener más información acerca de la verificación en dos fases y su cuenta de Facebook, consulte [¿Qué es la autenticación en dos pasos y cómo funciona?](https://www.facebook.com/help/148233965247823).

## <a name="add-your-github-account"></a>Adición de la cuenta de GitHub

Puede agregar su cuenta de GitHub activando primero la verificación en dos fases y, luego, agregando la cuenta a la aplicación.

### <a name="turn-on-two-factor-verification"></a>Activación de la verificación en dos fases

1. En el equipo, abra GitHub, seleccione la imagen de la esquina superior derecha y, a continuación, seleccione **Settings** (Configuración).

    Se muestra la página **Two-factor authentication** (Autenticación en dos fases).

2. Seleccione **Security** (Seguridad) desde la barra lateral **Personal settings** (Configuración personal) y, a continuación, seleccione **Enable two-factor authentication** (Habilitar autenticación en dos fases) desde el área **Two-factor authentication** (Autenticación en dos fases).

### <a name="add-your-github-account-to-the-app"></a>Adición de la cuenta de GitHub a la aplicación

1. En la página **Two-factor authentication** (Autenticación en dos fases) del equipo, seleccione **Set up using an app** (Configurar mediante una aplicación).

2. Guarde los códigos de recuperación para poder recuperar el acceso a su cuenta en caso de perderlo y, a continuación, seleccione **Next** (Siguiente). 

    Puede guardar los códigos descargándolos en el dispositivo mediante la impresión de una copia impresa o copiándolos en una herramienta de administrador de contraseñas.

3. En la página **Two-factor authentication** (Autenticación en dos fases), seleccione **Set up using an app** (Configuración mediante una aplicación).

    La página cambia para mostrar un código QR. No cierre esta página.

4. Abra la aplicación Microsoft Authenticator, seleccione **Agregar cuenta** en el icono **Personalizar y controlar** en la esquina superior derecha, después, seleccione **Otra cuenta (Google, Facebook, etc.)** y, a continuación, seleccione **enter this text code** (Escribir este código de texto) en el texto de la parte superior de la página.

    La aplicación Microsoft Authenticator no puede examinar el código QR, por lo que debe escribirlo manualmente.

5. Escriba un **nombre de cuenta** (por ejemplo, GitHub) y escriba la **clave secreta** del paso 4; a continuación, seleccione **Finish** (Finalizar).

6. En la página del **autenticador en dos fases** del equipo, escriba el código de verificación de seis dígitos que se proporciona en la aplicación de su cuenta de GitHub y, a continuación, seleccione **Enable** (Habilitar).

    En la página **Cuentas** de la aplicación aparece el nombre de su cuenta y un código de verificación de seis dígitos. Para mayor seguridad, el código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces.

    >[!NOTE]
    >Para obtener más información acerca de la verificación en dos fases y su cuenta de GitHub, consulte [About two-factor authentication](https://help.github.com/articles/about-two-factor-authentication/) (Acerca de la autenticación en dos fases).

## <a name="add-your-amazon-account"></a>Adición de la cuenta de Amazon

Agregue su cuenta personal de Amazon activando primero la verificación en dos fases y, luego, agregando la cuenta a la aplicación.

### <a name="turn-on-two-factor-verification"></a>Activación de la verificación en dos fases

1. En el equipo, abra Amazon, seleccione el menú desplegable **Cuenta y listas** y, a continuación, seleccione **Tu cuenta**.

2. Seleccione **Inicio de sesión y seguridad**, inicie sesión en su cuenta de Amazon y, a continuación, seleccione **Editar** en el área **Ajustes de seguridad avanzada**.

    Se muestra la página **Ajustes de seguridad avanzada**.

3. Seleccione **Comenzar**.

4. Seleccione **App de autenticación** desde la página **Elige la forma de recibir los códigos**.

    La página cambia para mostrar un código QR. No cierre esta página.

5. Abra la aplicación Microsoft Authenticator, seleccione **Agregar cuenta** en el icono **Personalizar y controlar** en la esquina superior derecha y, a continuación, seleccione **Otra cuenta (Google, Facebook, etc.)** .

6. Use la cámara del dispositivo para detectar el código QR de la página **Elige la forma de recibir los códigos** del equipo.

    >[!Note]
    >Si la cámara no funciona correctamente, puede escribir manualmente la dirección URL y el código QR.

7. Revise la página **Cuentas** de la aplicación Microsoft Authenticator en el dispositivo para asegurarse de que la información de la cuenta es correcta y de que hay un código de verificación de seis dígitos asociado.

    Para mayor seguridad, el código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces.

8. En la página **Elige la forma de recibir los códigos** del equipo, escriba el código de verificación de seis dígitos que se proporciona en la aplicación de su cuenta de Amazon y, a continuación, seleccione **Verificar código y continuar**.

9. Complete el resto del proceso de registro, incluida la operación para agregar un método de verificación de copia de seguridad, como un mensaje de texto y, a continuación, seleccione **Enviar código**.

10. En la página **Add a backup verification method** (Añadir método de verificación de copia de seguridad) en el equipo, escriba el código de verificación de seis dígitos que proporciona el método de verificación de copia de seguridad para su cuenta de Amazon y, a continuación, seleccione **Verificar código y continuar**.

11. En la página **Ya casi estamos**, decida si quiere que su equipo sea un dispositivo de confianza y, después, seleccione **Got it. Turn on Two-Step Verification** (Entendido. Activar la verificación en dos pasos).

    A continuación aparecerá la página **Ajustes de seguridad avanzada**, y se mostrarán los detalles de la verificación en dos fases actualizada.

    >[!NOTE]
    >Para obtener más información acerca de la verificación en dos fases y su cuenta de Amazon, consulte [Acerca de Verificación en dos pasos](https://www.amazon.com/gp/help/customer/display.html?nodeId=201596330) e [Ingresa a tu cuenta con Verificación en dos pasos](https://www.amazon.com/gp/help/customer/display.html?nodeId=201962440).

## <a name="next-steps"></a>Pasos siguientes

- Después de agregar las cuentas a la aplicación, puede iniciar sesión con la aplicación Authenticator en su dispositivo. Para más información, consulte cómo [iniciar sesión con la aplicación](user-help-auth-app-sign-in.md).

- En cuanto a los dispositivos con iOS, también puede hacer una copia de seguridad en la nube de las credenciales de su cuenta y de la configuración relacionada de la aplicación, como el orden de las cuentas. Para obtener más información, consulte [Copia seguridad y recuperación con la aplicación Microsoft Authenticator](user-help-auth-app-backup-recovery.md).
