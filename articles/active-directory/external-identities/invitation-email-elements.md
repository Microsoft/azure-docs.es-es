---
title: 'Elementos del correo electrónico de invitación de B2B: Azure Active Directory | Microsoft Docs'
description: Plantilla de correo electrónico de invitación de colaboración B2B de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0429cfb62c319675806d76b4759b776a7b32dbcb
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907690"
---
# <a name="the-elements-of-the-b2b-collaboration-invitation-email---azure-active-directory"></a>Elementos del correo electrónico de invitación para la colaboración B2B: Azure Active Directory

Los correos electrónicos de invitación son un componente fundamental para incorporar a los asociados como usuarios de colaboración B2B en Azure AD. Si bien [no es necesario que envíe un correo electrónico para invitar a alguien a usar la colaboración B2B](add-user-without-invite.md), al hacerlo, se le proporciona al usuario toda la información que necesita para tomar una decisión sobre si aceptar la invitación. También le proporciona un vínculo al que siempre puede hacer referencia en el futuro cuando necesiten volver a los recursos.

![Captura de pantalla que muestra el correo electrónico de invitación de B2B](media/invitation-email-elements/invitation-email.png)

> [!NOTE]
> Esta plantilla de correo electrónico nueva todavía se está implementando en todos los inquilinos, por lo que algunos inquilinos siguen usando un diseño anterior. A fines de mayo de 2020, las invitaciones de todos los inquilinos utilizarán esta plantilla.

## <a name="explaining-the-email"></a>Explicación del correo electrónico

Se van a tratar algunos elementos del correo electrónico para saber cómo hacer el mejor uso de estas funcionalidades.

### <a name="subject"></a>Asunto

El asunto del correo electrónico sigue este patrón:

&lt;username&gt; lo ha invitado a acceder a las aplicaciones de su organización.

### <a name="from-address"></a>Dirección De

Se usa un patrón similar a LinkedIn para la dirección De. Este patrón debe aclarar que, aunque el correo electrónico procede de invites@microsoft.com, la invitación viene de otra organización. El formato es: Microsoft Invitations <invites@microsoft.com> o invitaciones de Microsoft en nombre de &lt;tenantname&gt; <invites@microsoft.com>. 

### <a name="reply-to"></a>Responder a

En la respuesta al correo electrónico se indica el correo electrónico del invitador si está disponible, para que al responder al correo electrónico se vuelva a enviar un correo al invitador.

### <a name="phishing-warning"></a>Advertencia de suplantación de identidad (phishing)

El correo electrónico comienza con una breve advertencia para el usuario sobre la suplantación de identidad (phishing) y lo alerta de que solo debe aceptar las invitaciones que espera. Es recomendable asegurarse de que los asociados a los que está invitando no se vean sorprendidos por su invitación, así es que es bueno que lo mencione con tiempo.

![Imagen de la advertencia de suplantación de identidad (phishing) en el correo electrónico](media/invitation-email-elements/phishing-warning.png)

### <a name="inviters-information"></a>Información sobre el invitador

El correo electrónico incluye información sobre el invitador y la organización de la que está enviando la invitación. Esto incluye el nombre y la dirección de correo electrónico del remitente, así como el nombre y el dominio principal asociados con la organización. Toda esta información debe ayudar al invitado a tomar una decisión informada sobre si aceptar la invitación.

![Imagen de la información del invitador en el correo electrónico](media/invitation-email-elements/inviters-information.png)

### <a name="invitation-message"></a>Mensaje de invitación

Si el invitador incluye un mensaje como parte de su invitación cuando [invita a un usuario invitado al directorio, grupo o aplicación](add-users-administrator.md) o cuando [usa la API de invitación](customize-invitation-api.md), el mensaje aparece resaltado en la sección principal del correo electrónico. También se incluye el nombre y la imagen de perfil del invitador, si se estableció. El mensaje mismo es un área de texto por lo que, por motivos de seguridad, no procesa etiquetas HTML.

![Imagen del mensaje de invitación en el correo electrónico](media/invitation-email-elements/invitation-message.png)

### <a name="accept-button-and-redirect-url"></a>Botón Aceptar y URL de redireccionamiento

La sección siguiente del correo electrónico contiene información sobre adónde irá el invitado después de aceptar la invitación, así como un botón para hacerlo.  En el futuro, el invitado siempre podrá usar este vínculo para volver directamente a los recursos.

![Imagen del botón Aceptar y la URL de redireccionamiento en el correo electrónico](media/invitation-email-elements/accept-button.png)

### <a name="footer-section"></a>Sección de pie de página

El pie de página contiene más información sobre la invitación que se envía. Siempre debe haber una opción para que el invitado bloquee las invitaciones futuras. Si la organización [estableció una declaración de privacidad](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-properties-area), aquí debe aparecer el vínculo a dicha declaración.  De lo contrario, una nota indica que la organización no ha establecido una declaración de privacidad.

![Imagen de la sección de pie de página del correo electrónico](media/invitation-email-elements/footer-section.png)
 
## <a name="how-the-language-is-determined"></a>Cómo se determina el idioma

Las opciones siguientes determinan el idioma que el usuario invitado ve en el correo electrónico de invitación. Estas opciones de configuración se muestran en el siguiente orden de prioridad. Si una opción no está configurada, la siguiente de la lista será la que determine el idioma.

- La propiedad **messageLanguage** del objeto [guestUserMessageInfo](https://docs.microsoft.com/graph/api/resources/invitedusermessageinfo?view=graph-rest-1.0), si se usa la API para crear una invitación.
-   La propiedad **preferredLanguage** especificada en el [objeto de usuario](https://docs.microsoft.com/graph/api/resources/user?view=graph-rest-1.0) del invitado.
-   El **idioma de notificación** establecido en las propiedades del inquilino principal del usuario invitado (solo para inquilinos de Azure AD).
-   El **idioma de notificación** establecido en las propiedades del inquilino del recurso.

Si ninguna de estas opciones está configurada, el idioma se establece de forma predeterminada en inglés (EE. UU.).

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [¿Cómo agregan los administradores de Azure Active Directory usuarios de colaboración B2B?](add-users-administrator.md)
- [¿Cómo agregan los trabajadores de la información usuarios de colaboración B2B?](add-users-information-worker.md)
- [Canje de invitación de colaboración B2B](redemption-experience.md)
- [Incorporación de usuarios de colaboración B2B sin invitación](add-user-without-invite.md)
