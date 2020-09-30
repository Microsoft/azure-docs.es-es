---
title: 'Problemas comunes con la autenticación en dos fases de una cuenta: Azure AD'
description: Soluciones para algunos de los problemas más comunes de la verificación en dos fases y la cuenta profesional o educativa.
services: active-directory
author: curtand
manager: daveba
ms.assetid: 8f3aef42-7f66-4656-a7cd-d25a971cb9eb
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 09/01/2020
ms.author: curtand
ms.reviewer: kexia
metadata ms.custom: contperfq1
ms.openlocfilehash: 3d95ad4aa100b0a185bde015dfe34d747fc77ec9
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322609"
---
# <a name="common-problems-with-two-factor-verification-and-your-work-or-school-account"></a>Problemas comunes con la verificación en dos fases y la cuenta profesional o educativa

Hay algunos problemas comunes en la verificación en dos fases que parecen ocurrir con más frecuencia de la que nos gustaría. Este artículo pretende describir las correcciones para los problemas más comunes.

La organización de Azure Active Directory (Azure AD) puede activar la verificación en dos pasos para su cuenta. Cuando la verificación en dos pasos está activada, se requiere una combinación de los datos siguientes para iniciar sesión en la cuenta:

- Nombre de usuario
- La contraseña
- Un teléfono o dispositivo móvil

La verificación en dos pasos es más segura que solo una contraseña, ya que requiere algo que debe _saber_ más algo que debe _tener_. Ningún hacker tiene su teléfono físico.

>[!Important]
>Si es administrador, puede encontrar más información sobre cómo configurar y administrar su entorno de Azure AD en la [documentación de Azure AD](../index.yml).

Este contenido puede ayudarle con la cuenta profesional o educativa, que es la que le proporcionó su organización (por ejemplo, dritan@contoso.com). Si tiene problemas con la verificación en dos pasos de una cuenta personal de Microsoft, que es una cuenta que configuró por sí mismo (por ejemplo, danielle@outlook.com), vea [Activar o desactivar la verificación en dos pasos para la cuenta de Microsoft](https://support.microsoft.com/help/4028586/microsoft-account-turning-two-step-verification-on-or-off).

## <a name="i-dont-have-my-mobile-device-with-me"></a>No llevo conmigo mi dispositivo móvil

Pasa algunas veces. Olvidó su dispositivo móvil en casa y ahora no puede utilizarlo para verificar su identidad. Tal vez haya añadido anteriormente un método alternativo para iniciar sesión en su cuenta (por ejemplo, el teléfono del trabajo). En caso afirmativo, puede utilizar ese método alternativo ahora. Si nunca ha añadido un método de verificación alternativo, póngase en contacto con el departamento de soporte técnico de su organización para solicitar ayuda.

### <a name="to-sign-in-to-your-work-or-school-account-using-another-verification-method"></a>Para iniciar sesión en su cuenta profesional o educativa con otro método de verificación

1. Inicie sesión en la cuenta, pero seleccione **Sign in another way** (Iniciar sesión de otro modo) en la página **Two-step verification** (Verificación en dos fases).

    ![Cambio del método de comprobación de inicio de sesión](./media/multi-factor-authentication-end-user-troubleshoot/two-factor-auth-signin-another-way.png)

    Si no ve el vínculo **Sign in another way** (Iniciar sesión de otro modo) significa que no ha configurado ningún otro método de comprobación. Tendrá que ponerse en contacto con el administrador para que lo ayude a iniciar sesión en la cuenta.

2. Elija el método de comprobación alternativo y siga con el proceso de verificación en dos fases.

## <a name="i-cant-turn-two-factor-verification-off"></a>¿Por qué no se puede desactivar la verificación en dos fases?

- Si usa la verificación en dos pasos con una cuenta personal para un servicio de Microsoft, como alain@outlook.com, tiene la posibilidad de [activar y desactivar la característica](https://account.live.com/proofs/Manage).

- Si usa la verificación en dos pasos con su cuenta profesional o educativa, lo más probable es que la organización haya decidido que debe usar esta característica de seguridad adicional. No hay ninguna manera de desactivarla de modo individual.

Si no puede desactivar la comprobación en dos fases, también podría deberse a los valores predeterminados de seguridad que se aplicaron en el nivel de organización. Para obtener más información sobre los valores predeterminados de seguridad, consulte [¿Qué son los valores predeterminados de seguridad?](../fundamentals/concept-fundamentals-security-defaults.md)

## <a name="my-device-was-lost-or-stolen"></a>He perdido o me han robado mi dispositivo

Si ha perdido o le han robado el dispositivo móvil, puede realizar una de las siguientes acciones:

- Iniciar sesión mediante otro método.
- Pedir al departamento de soporte técnico de su organización que borre la configuración.

Si ha perdido o le han robado el teléfono, es muy recomendable que informe al departamento de soporte técnico de la organización. El departamento de soporte técnico puede hacer las actualizaciones pertinentes en la cuenta del usuario. Una vez borrada su configuración, se le pedirá que se [registre en la verificación en dos fases](multi-factor-authentication-end-user-first-time.md) la próxima vez que inicie sesión.

## <a name="im-not-receiving-the-verification-code-sent-to-my-mobile-device"></a>No recibo el código de verificación enviado a mi dispositivo móvil

Este es un problema común. Normalmente, está relacionado con su dispositivo móvil y la configuración. A continuación, se incluyen algunas acciones que puede probar.

Probar esto | Directrices
--------- | ------------
Reiniciar el dispositivo móvil | A veces, el dispositivo solo necesita una actualización. Al reiniciar el dispositivo, se cierran todos los procesos y servicios en segundo plano. El reinicio también apaga los componentes principales del dispositivo. Los servicios o componentes se actualizan después de reiniciar el dispositivo.
Comprobar que la información de seguridad sea correcta | Asegúrese de que la información de seguridad del método de verificación es correcta, especialmente los números de teléfono. Si pone un número de teléfono equivocado, todas las alertas irán a ese número incorrecto. Afortunadamente, ese usuario no podrá hacer nada con esas alertas, pero esto tampoco le permitirá iniciar sesión en su cuenta. Para asegurarse de que la información es correcta, consulte las instrucciones del artículo [Administración de la configuración del método de verificación en dos fases](multi-factor-authentication-end-user-manage-settings.md).
Comprobar que las notificaciones estén activadas | Asegúrese de que el dispositivo móvil tenga activadas las notificaciones. Compruebe que los siguientes modos de notificación están permitidos: <br/><br/> &bull; Llamadas de teléfono <br/> &bull; La aplicación de autenticación <br/> &bull; La aplicación de mensajería de texto <br/><br/> Asegúrese de que estos modos generan una alerta _visible_ en el dispositivo.
Asegurarse de que el dispositivo tenga cobertura y conexión a Internet | Asegúrese de que llegan llamadas telefónicas y mensajes de texto al dispositivo móvil. Pida a un amigo que le llame y le envíe un mensaje de texto para asegurarse de que recibe ambos. Si no recibe la llamada o el mensaje de texto, compruebe que el dispositivo móvil está encendido. Si el dispositivo está encendido, pero sigue sin recibir la llamada o el texto, puede que haya un problema con la red. Póngase en contacto con el proveedor. Si suele experimentar problemas relacionados con la cobertura, le recomendamos que instale y use la [aplicación Microsoft Authenticator](user-help-auth-app-download-install.md) en su dispositivo móvil. Esta aplicación puede generar códigos de seguridad aleatorios para iniciar sesión, sin necesidad de una conexión a Internet ni de cobertura del móvil.
Desactivar la opción No molestar | Asegúrese de que no ha activado la característica **No molestar** del dispositivo móvil. Si esta característica está activada, no permite que reciba notificaciones en su dispositivo móvil. Consulte el manual del dispositivo móvil para obtener instrucciones sobre cómo desactivar esta característica.
Desbloquear los números de teléfono | En Estados Unidos, las llamadas de voz de Microsoft proceden de los números siguientes: +1 (866) 539 4191, +1 (855) 330 8653 y +1 (877) 668 6536.
Comprobar la configuración relacionada con la batería | Si configuró la optimización de la batería para que impida que las aplicaciones menos usadas permanezcan activas en segundo plano, es probable que el sistema de notificaciones se haya visto afectado. Pruebe a desactivar la optimización de la batería para las aplicaciones de autenticación y de mensajería. Pruebe a iniciar sesión nuevamente en su cuenta.
Deshabilitación de aplicaciones de seguridad de terceros | Algunas aplicaciones de seguridad del teléfono bloquean los mensajes de texto y las llamadas telefónicas de autores de llamada desconocidos. Es posible que una aplicación de seguridad impida que el teléfono reciba el código de verificación. Pruebe a desactivar las aplicaciones de seguridad de terceros en el teléfono y, a continuación, solicite que le envíen otro código de verificación.

## <a name="im-not-being-prompted-for-my-second-verification-information"></a>No recibo la solicitud de información para la segunda verificación

Para iniciar sesión en la cuenta profesional o educativa, utilice el nombre de usuario y la contraseña. A continuación, se le solicitará la información adicional de verificación de seguridad. Si no se le solicita, es posible que aún no haya configurado el dispositivo. El dispositivo móvil debe estar configurado para que funcione con el método de verificación de seguridad adicional específico.

Puede que aún no haya configurado el dispositivo. El dispositivo móvil tiene que estar configurado para que funcione con el método de verificación de seguridad adicional específico. Si desea conocer los pasos necesarios para que el dispositivo móvil esté disponible para usarlo con el método de verificación, consulte [Administración de la configuración del método de verificación en dos pasos](multi-factor-authentication-end-user-manage-settings.md). Si sabe que no ha configurado el dispositivo o su cuenta todavía, puede seguir los pasos del artículo [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

## <a name="i-have-a-new-phone-number-and-i-want-to-add-it"></a>Tengo un nuevo número de teléfono y deseo añadirlo

Si tiene un nuevo número de teléfono, tiene que actualizar los detalles del método de verificación de seguridad. De este modo, sus solicitudes de verificación se dirigirán a la ubicación correcta. Para actualizar el método de verificación, siga los pasos de la sección **Adición o cambio del número de teléfono** en el artículo [Cambio de la configuración y del método de comprobación en dos pasos](multi-factor-authentication-end-user-manage-settings.md#add-or-change-your-phone-number).

## <a name="i-have-a-new-mobile-device-and-i-want-to-add-it"></a>Tengo un nuevo dispositivo móvil y me gustaría añadirlo

Si tiene un nuevo dispositivo móvil, deberá configurarlo para que funcione con la verificación en dos pasos. Esta solución consta de varios pasos:

1. Configure el dispositivo para que funcione con la cuenta profesional o educativa. Para ello, siga los pasos descritos en el artículo sobre [cómo configurar una cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md).

1. Actualice la información de la cuenta y del dispositivo en la página **Comprobación de seguridad adicional**. Para aplicar la actualización, elimine el dispositivo antiguo y añada el nuevo. Para más información, consulte el artículo [Administración de la configuración del método de verificación en dos fases](multi-factor-authentication-end-user-manage-settings.md).

Pasos opcionales:

- Instale la aplicación Microsoft Authenticator en el dispositivo móvil; para ello, siga los pasos del artículo [Descarga e instalación de la aplicación Microsoft Authenticator](user-help-auth-app-download-install.md).

- Active la verificación en dos fases para los dispositivos de confianza siguiendo los pasos descritos en la sección sobre cómo **activar los avisos de la verificación en dos fases en un dispositivo de confianza**, en el artículo [Administración de la configuración del método de verificación en dos fases](multi-factor-authentication-end-user-manage-settings.md#turn-on-two-factor-verification-prompts-on-a-trusted-device).

## <a name="im-having-problems-signing-in-on-my-mobile-device-while-traveling"></a>Tengo problemas para iniciar sesión en el dispositivo móvil mientras viajo

Es posible que le resulte más difícil usar un método de verificación relacionado con dispositivos móviles, como un mensaje de texto, mientras se encuentra en el extranjero. También es posible que el dispositivo móvil pueda provocar cargos de itinerancia. En esta situación, se recomienda usar la aplicación Microsoft Authenticator y, si se desea, conectarse a una zona activa Wi-Fi. Para obtener más información sobre cómo configurar la aplicación Microsoft Authenticator en el dispositivo móvil, consulte el artículo [Descarga e instalación de la aplicación Microsoft Authenticator](user-help-auth-app-download-install.md).

## <a name="i-cant-get-my-app-passwords-to-work"></a>No consigo que funcionen mis contraseñas de aplicación

Las contraseñas de aplicación reemplazan a las contraseñas normales de las aplicaciones de escritorio más antiguas que no admiten la verificación en dos fases. En primer lugar, asegúrese de escribir correctamente la contraseña. Si esto no soluciona el problema, intente crear una nueva contraseña para la aplicación. Para ello, siga los pasos descritos en la sección sobre cómo **crear y eliminar contraseñas de aplicación mediante el portal Aplicaciones** del artículo [Administración de las contraseñas de aplicaciones para la verificación en dos pasos](multi-factor-authentication-end-user-app-passwords.md#create-and-delete-app-passwords-from-the-additional-security-verification-page).

## <a name="i-cant-turn-off-two-factor-verification"></a>No se puede desactivar la verificación en dos pasos

Si usa la verificación en dos pasos con su cuenta profesional o educativa (por ejemplo, alain@contoso.com), lo más probable es que la organización haya decidido que debe usar esta característica de seguridad adicional. Si este es el caso, no hay ninguna manera de desactivarla individualmente. Sin embargo, si usa la verificación en dos pasos con una cuenta personal, como alain@outlook.com, tiene la posibilidad de activar y desactivar la característica. Para instrucciones sobre cómo controlar la verificación en dos fases en sus cuentas personales, consulte [Activar o desactivar la verificación en dos pasos para la cuenta de Microsoft](https://support.microsoft.com/help/4028586/microsoft-account-turning-two-step-verification-on-or-off).

Si no puede desactivar la comprobación en dos fases, también podría deberse a los valores predeterminados de seguridad que se aplicaron en el nivel de organización. Para obtener más información sobre los valores predeterminados de seguridad, consulte [¿Qué son los valores predeterminados de seguridad?](../fundamentals/concept-fundamentals-security-defaults.md)

## <a name="i-didnt-find-an-answer-to-my-problem"></a>No encuentro una respuesta a mi problema.

Si ya intentó estos pasos pero sigue teniendo problemas, póngase en contacto con el departamento de soporte técnico de la organización para que le ayude.

## <a name="related-articles"></a>Artículos relacionados

- [Administración de la configuración del método de verificación en dos fases](multi-factor-authentication-end-user-manage-settings.md)

- [Configuración de mi cuenta para la verificación en dos pasos](multi-factor-authentication-end-user-first-time.md)

- [Preguntas frecuentes de la aplicación Microsoft Authenticator](user-help-auth-app-faq.md)