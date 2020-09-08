---
title: ¿Qué es la aplicación Microsoft Authenticator? Azure Active Directory | Microsoft Docs
description: Obtenga información acerca de la aplicación Microsoft Authenticator, lo que incluye qué es, cómo funciona y qué información se incluye en esta sección del contenido.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 01/15/2020
ms.author: curtand
ms.openlocfilehash: 7d630fac1f848e691ddb96dc47596a2e9927309f
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799441"
---
# <a name="what-is-the-microsoft-authenticator-app"></a>¿Qué es la aplicación Microsoft Authenticator?

La aplicación Microsoft Authenticator le ayuda a iniciar sesión en sus cuentas si usa la comprobación en dos pasos. La comprobación en dos pasos le ayuda a acceder a sus cuentas de manera más segura, en especial cuando se examina información confidencial. Como las contraseñas se pueden olvidar, poner en peligro o robar, la comprobación en dos pasos es un paso de seguridad adicional que ayuda a proteger su cuenta ya que es más difícil que otras personas accedan a ella.

Puede usar la aplicación Microsoft Authenticator de varias formas, por ejemplo:

- Responda a una solicitud de autenticación después de iniciar sesión con el nombre de usuario y la contraseña.

- Inicie sesión sin escribir contraseña, con su nombre de usuario, la aplicación de autenticación y el dispositivo móvil con su huella digital, cara o PIN.

- Como generador de código de cualquier otra cuenta que admite aplicaciones de autenticación.

> [!Important]
> La aplicación Microsoft Authenticator funciona con cualquier cuenta que use la comprobación en dos fases y admita los estándares de contraseña de un solo uso y duración definida (TOTP).
>
>Este artículo está destinado a los usuarios que intentan descargar y usar la aplicación Microsoft Authenticator como método de comprobación de seguridad. Si es un administrador que desea obtener información acerca de cómo activar el inicio de sesión sin contraseña mediante la aplicación Authenticator para los empleados y otros usos, consulte la [Habilitar el inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator (versión preliminar)](../authentication/howto-authentication-passwordless-phone.md).

## <a name="terminology"></a>Terminología

| Término|Descripción|
| ----|-----------|
| Comprobación en dos fases | Un proceso de comprobación que requiere que se usen específicamente únicamente dos factores de comprobación, como por ejemplo una contraseña y un PIN. La aplicación Microsoft Authenticator admite tanto la comprobación en dos fases estándar como el inicio de sesión sin contraseña. |
| Multi-Factor Authentication (MFA) | Toda comprobación en dos fases es una autenticación multifactor, que requiere que use *al menos* dos factores de comprobación, según los requisitos de su organización. |
| Cuenta Microsoft (también denominada MSA) | Cree sus propias cuentas personales para poder acceder a los productos y servicios en la nube de Microsoft orientados al consumidor, como Outlook, OneDrive, Xbox LIVE u Office 365. Su cuenta Microsoft se crean y almacenan en el sistema de cuentas de identidad de consumidor de Microsoft que ejecuta Microsoft. |
| Cuenta profesional o educativa | Su organización crea una cuenta profesional o educativa (como alain@contoso.com) para permitirle acceder a recursos internos y potencialmente restringidos, como Microsoft Azure, Windows Intune y Office 365. |
| Código de verificación | El código de seis dígitos que aparece en la aplicación autenticadora en cada cuenta agregada. El código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces. Esto también se denomina código de acceso de un solo uso (OTP). |

## <a name="how-two-factor-verification-works-with-the-app"></a>Funcionamiento de la comprobación en dos fases con la aplicación

Así funciona la comprobación mediante dos factores con la aplicación Microsoft Authenticator:

- **Notificación.** Escriba el nombre de usuario y la contraseña en el dispositivo en el que va a iniciar sesión con su cuenta profesional o educativa o con su cuenta personal de Microsoft y, después, la aplicación Microsoft Authenticator envía una notificación que le pide que **apruebe el inicio de sesión**. Elija **Aprobar** si reconoce el intento de inicio de sesión. De lo contrario, elija **Rechazar**. Si elige **Rechazar**, también puede marcar la solicitud como fraudulenta.

- **Código de verificación.** Escriba el nombre de usuario y la contraseña en el dispositivo en que va a iniciar sesión con su cuenta profesional o educativa, o con su cuenta personal de Microsoft y, después, copie el código de verificación asociado en la pantalla **Cuentas** de la aplicación Microsoft Authenticator. El código de verificación también se denomina autenticación de código de acceso de un solo uso (OTP).

- **Inicio de sesión sin contraseña.** Escriba el nombre de usuario en el dispositivo en que va a iniciar una sesión con su cuenta profesional o educativa o su cuenta personal de Microsoft y, después, use su dispositivo móvil para comprobar que es usted mediante el uso de la huella digital, la cara o un PIN. Para este método, no es preciso que escriba su contraseña.

### <a name="whether-to-use-your-devices-biometric-capabilities"></a>Si usa las funcionalidades biométricas del dispositivo

Si usa un PIN para completar el proceso de autenticación, puede configurar la aplicación Microsoft Authenticator para usar en su lugar las funcionalidades de reconocimiento facial (biométrica) o de la huella del dispositivo. Puede realizar esta configuración la primera vez que use la aplicación Authenticator para verificar su cuenta; para ello, seleccione la opción para usar las funcionalidades biométricas del dispositivo como identificación en lugar del PIN.

## <a name="who-decides-if-you-use-this-feature"></a>¿Quién decide si usa esta característica?

En función del tipo de cuenta, la organización puede decidir que se debe usar una comprobación en dos fases o el usuario puede decidir por sí mismo si se usa o no.

- **Cuenta profesional o educativa.** Si se usa una cuenta profesional o educativa (por ejemplo, alain@contoso.com), la organización es la que decide si se debe usar una comprobación en dos fases, junto con los métodos de comprobación concretos. Para más información acerca de cómo agregar una cuenta profesional o educativa a la aplicación Microsoft Authenticator, consulte [Adición de una cuenta profesional o educativa](user-help-auth-app-add-work-school-account.md).

- **Cuenta personal de Microsoft.** Se puede elegir configurar una comprobación en dos fases para las cuentas personales de Microsoft (por ejemplo, alain@outlook.com). Para más información acerca de cómo agregar una cuenta Microsoft personal, consulte [Adición de cuentas personales de Microsoft](user-help-auth-app-add-personal-ms-account.md).

- **Cuenta que no es de Microsoft.** Puede elegir configurar una verificación en dos pasos para sus cuentas que no sean de Microsoft (por ejemplo, alain@gmail.com). Es posible que las cuentas que no sean de Microsoft no usen el término comprobación en dos fases, pero la característica debería encontrarse dentro de la configuración de **Seguridad** o **Inicio de sesión**. La aplicación Microsoft Authenticator funciona con todas las cuentas compatibles con los estándares TOTP. Para más información acerca de cómo agregar cuentas que no sean de Microsoft, consulte [Adición de cuentas que no sean de Microsoft](user-help-auth-app-add-non-ms-account.md).

## <a name="in-this-section"></a>En esta sección

| Artículo | Descripción |
| ------ | ------------ |
| [Descarga e instalación de la aplicación](user-help-auth-app-download-install.md) | Describe dónde y cómo obtener e instalar la aplicación Microsoft Authenticator para dispositivos con Android e iOS. |
| [Adición de cuentas profesionales o educativas](user-help-auth-app-add-work-school-account.md) | Se describe cómo agregar varias cuentas profesionales o educativas, y personales a la aplicación Microsoft Authenticator. |
| [Adición de cuentas personales](user-help-auth-app-add-personal-ms-account.md) | Se describe cómo agregar sus cuentas personales Microsoft a la aplicación Microsoft Authenticator. |
| [Adición de cuentas que no sean de Microsoft](user-help-auth-app-add-non-ms-account.md) | Describe cómo agregar sus cuentas personales que no sean Microsoft a la aplicación Microsoft Authenticator. |
| [Adición manual de cuentas](user-help-auth-app-add-account-manual.md) | Se describe cómo agregar manualmente cuentas a la aplicación Microsoft Authenticator si no se puede leer el código QR. |
| [Inicio de sesión mediante la aplicación](user-help-auth-app-sign-in.md) | Describe cómo iniciar sesión en varias cuentas mediante la aplicación Microsoft Authenticator.|
| [Credenciales de la cuenta de copia de seguridad y recuperación](user-help-auth-app-backup-recovery.md) | Proporciona información acerca de cómo realizar copias de seguridad y recuperar las credenciales de su cuenta con la aplicación Microsoft Authenticator. |
| [Preguntas frecuentes de la aplicación Microsoft Authenticator](user-help-auth-app-faq.md) | Proporciona respuestas a las preguntas más frecuentes acerca de la aplicación. |