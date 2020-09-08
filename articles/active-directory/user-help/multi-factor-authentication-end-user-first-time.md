---
title: ¿Qué es la página Comprobación adicional? - Azure AD
description: Acceso a la página Comprobación de seguridad adicional para la comprobación en dos fases
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 05/28/2020
ms.author: curtand
ms.openlocfilehash: e5b07f8f7ae766d110c87a495a3e1623b815e526
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88795990"
---
# <a name="what-is-the-additional-verification-page"></a>¿Qué es la página Comprobación adicional?

La organización está realizando pasos adicionales para asegurarse de que usted es quien dice ser al iniciar sesión. Esta comprobación de seguridad adicional también se conoce como verificación en dos pasos. Consta de una combinación de nombre de usuario, contraseña y dispositivo móvil o teléfono. Si todo lo que desea hacer es desactivar la verificación en dos pasos en una cuenta de Microsoft como alain@outlook.com, use las instrucciones de [Activación o desactivación de la verificación en dos pasos para una cuenta de Microsoft](https://support.microsoft.com/help/4028586/microsoft-account-turning-two-step-verification-on-or-off).

<center>

![Imagen de los métodos de autenticación conceptuales](../authentication/media/concept-mfa-howitworks/methods.png)</center>

La verificación en dos fases es más segura que solo una contraseña, porque se basa en dos formas de autenticación:

- Algo que sabe, como la contraseña.
- Algo que tiene, como un teléfono u otro dispositivo que lleve consigo.

La verificación en dos fases puede servir de ayuda para evitar que hackers malintencionados se hagan pasar por usted, ya que, aunque tengan su contraseña, es muy improbable que tengan también el dispositivo.

>[!Important]
>Si es un administrador que busca información acerca de cómo activar la verificación en dos fases para los empleados u otros usuarios, consulte la [documentación de autenticación de Azure Active Directory](../authentication/index.yml). Este artículo está destinado a aquellos usuarios que quieren usar la verificación en dos fases con una cuenta profesional o educativa (por ejemplo, alain@contoso.com).

## <a name="who-decides-if-you-use-this-feature"></a>¿Quién decide si usa esta característica?

Quien decide si usa la verificación en dos fases depende del tipo de cuenta que tenga:

- **Cuenta profesional o educativa.** Si usa una cuenta profesional o educativa (por ejemplo, alain@contoso.com), su organización es la que decide si utiliza la verificación en dos fases, junto con los métodos de comprobación concretos. Si este es el caso, no hay manera de que pueda desactivarla.

- **Cuenta personal de Microsoft.** Se puede elegir configurar una verificación en dos fases para las cuentas personales de Microsoft (como alain@outlook.com). Puede activarla o desactivarla siempre que lo desee. Para ello, solo debe seguir las sencillas instrucciones que encontrará en [Activación o desactivación de la verificación en dos fases para una cuenta de Microsoft](https://support.microsoft.com/help/4028586/microsoft-account-turning-two-step-verification-on-or-off).

    >[!Note]
    >Si tiene otros problemas con la verificación en dos fases y alguna de sus cuentas de Microsoft personales, en [Cómo usar la verificación en dos pasos con la cuenta de Microsoft](https://support.microsoft.com/help/12408/microsoft-account-how-to-use-two-step-verification) encontrará más sugerencias.

## <a name="open-the-additional-security-verification-page"></a>Apertura de la página Comprobación de seguridad adicional

Una vez que la organización active la verificación en dos fases, cada vez que inicie sesión recibirá un mensaje que le indicará que proporcione más información para ayudar a mantener su cuenta segura.

![Solicitud de más información necesaria](media/multi-factor-authentication-verification-methods/multi-factor-authentication-initial-prompt.png)

### <a name="to-access-the-additional-security-verification-page"></a>Para acceder a la página Comprobación de seguridad adicional

1. Seleccione **Siguiente** en el mensaje de **Más información necesaria**.

    Aparece la página **Comprobación de seguridad adicional**.

2. En la página **Comprobación de seguridad adicional**, seleccione el método de verificación en dos fases que se va a usar para comprobar que es quien dice ser al iniciar sesión en su cuenta profesional o educativa. Puede seleccionar:

    | Método de contacto | Descripción |
    | --- | --- |
    | Aplicación móvil | <ul><li>**Recibir notificaciones de comprobación.** Esta opción inserta una notificación en la aplicación del autenticador en su smartphone o tableta. Vea la notificación y, si es legítima, seleccione **Autenticar** en la aplicación. Puede que su trabajo o escuela requiera que escriba un PIN para autenticarse.</li><li>**Usar el código de verificación.** En este modo, la aplicación genera un código de verificación que se actualiza cada 30 segundos. Escriba el código de verificación más reciente en la pantalla de inicio de sesión.<br>La aplicación Microsoft Authenticator está disponible para [Android](https://go.microsoft.com/fwlink/?linkid=866594) e [iOS](https://go.microsoft.com/fwlink/?linkid=866594).</li></ul> |
    | Teléfono de autenticación | <ul><li>**Llamada de teléfono** realiza una llamada de voz automatizada al número de teléfono que proporcione. El usuario responde a la llamada y pulsa la tecla de almohadilla (#) en el teclado del teléfono para autenticarse.</li><li>**Mensaje de texto** envía un mensaje de texto que contiene un código de verificación. Según la solicitud en el texto, responda al mensaje de texto o escriba el código de comprobación proporcionado en la interfaz de inicio de sesión.</li></ul> |
    | Teléfono del trabajo | Realiza una llamada de voz automatizada al número de teléfono que proporcione. El usuario responde a la llamada y pulsa la tecla de almohadilla (#) en el teclado del teléfono para autenticarse. |

## <a name="next-steps"></a>Pasos siguientes

Tras seleccionar un método de verificación en dos fases en la página **Comprobación de seguridad adicional**, debe configurarlo:

- [Configuración de un dispositivo móvil como método de comprobación](multi-factor-authentication-setup-phone-number.md)

- [Configuración de un teléfono como método de comprobación](multi-factor-authentication-setup-office-phone.md)

- [Configuración de la aplicación Microsoft Authenticator como método de comprobación](multi-factor-authentication-setup-auth-app.md)

## <a name="related-resources"></a>Recursos relacionados

- [Inicio de sesión con la verificación en dos pasos](multi-factor-authentication-end-user-signin.md)

- [Obtención de ayuda con la verificación en dos fases](multi-factor-authentication-end-user-troubleshoot.md)