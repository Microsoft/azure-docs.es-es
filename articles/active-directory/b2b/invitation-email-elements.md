---
title: Los elementos del mensaje de correo de invitación de B2B - Azure Active Directory | Microsoft Docs
description: Plantilla de correo electrónico de invitación de colaboración B2B de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 02/06/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3f93586d46aa01116990f8f02f344c6952d3c1b1
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65768364"
---
# <a name="the-elements-of-the-b2b-collaboration-invitation-email---azure-active-directory"></a>Elementos del correo electrónico de invitación para la colaboración B2B: Azure Active Directory

Los correos electrónicos de invitación son un componente fundamental para incorporar a los asociados como usuarios de colaboración B2B en Azure AD. Puede usarlos para aumentar la confianza del destinatario. Puede agregar legitimidad y prueba social al correo electrónico para asegurarse de que el destinatario se sienta cómodo al seleccionar el botón **Empezar** para aceptar la invitación. Esta confianza es clave para reducir la fricción en el uso compartido. Y puede que también quiera que el correo electrónico tenga un buen aspecto.

![Captura de pantalla que muestra el correo electrónico de invitación de B2B](media/invitation-email-elements/invitation-email.png)

## <a name="explaining-the-email"></a>Explicación del correo electrónico
Se van a tratar algunos elementos del correo electrónico para saber cómo hacer el mejor uso de estas funcionalidades.

### <a name="subject"></a>Subject
El asunto del correo electrónico sigue este patrón: está invitado a la organización de &lt;nombreInquilino&gt;.

### <a name="from-address"></a>Dirección de origen
Se usa un patrón similar a LinkedIn para la dirección De.  Es necesario tener claro quién es el invitador y a qué empresa pertenece y también aclarar que el correo electrónico procede de una dirección de correo electrónico de Microsoft. El formato es: Microsoft Invitations <invites@microsoft.com> o &lt;nombre para mostrar del invitador&gt; desde &lt;tenantname&gt; (a través de Microsoft) <invites@microsoft.com>.

### <a name="reply-to"></a>Responder a
En la respuesta al correo electrónico se indica el correo electrónico del invitador si está disponible, para que al responder al correo electrónico se vuelva a enviar un correo al invitador.

### <a name="branding"></a>Personalización de marca
Los correos electrónicos de invitación del inquilino usan la personalización de marca de la empresa que puede haberse configurado para el inquilino. Si desea beneficiarse de esta funcionalidad, [aquí](https://docs.microsoft.com/azure/active-directory/active-directory-branding-custom-signon-azure-portal) se proporcionan los detalles sobre cómo configurarla. El logotipo del banner aparece en el correo electrónico. Siga el tamaño de la imagen y las instrucciones de calidad indicadas [aquí](https://docs.microsoft.com/azure/active-directory/active-directory-branding-custom-signon-azure-portal) para obtener los mejores resultados. Además, el nombre de la empresa también se presenta en la llamada a la acción.

### <a name="call-to-action"></a>Llamada a la acción
La llamada a la acción consta de dos partes: explicar por qué el destinatario ha recibido el correo electrónico y qué se pide al destinatario que haga al respecto.
- La sección “por qué” puede seguir este patrón: Se le ha invitado a acceder a aplicaciones en la organización de &lt;nombreInquilino&gt;.

- Y la sección “qué se le pide que haga” está indicada por la presencia del botón **Introducción**. Cuando el destinatario se agrega sin necesidad de invitaciones, este botón no se muestra.

### <a name="inviters-information"></a>Información sobre el invitador
El nombre para mostrar del invitador se incluye en el correo electrónico. Y, además, si ha configurado una imagen de perfil para la cuenta de Azure AD, el correo electrónico de invitación incluye también esa imagen. Ambas opciones están diseñadas para aumentar la confianza del destinatario en el correo electrónico.

Si aún no ha configurado la imagen del perfil, se muestra un icono con las iniciales del invitador en lugar de la imagen:

  ![Captura de pantalla muestra la invitación con el invitador iniciales que se muestran](media/invitation-email-elements/inviters-initials.png)

### <a name="body"></a>Cuerpo
El cuerpo contiene el mensaje que el invitador redacta al [invitar a un usuario invitado al directorio, grupo o aplicación](add-users-administrator.md) o al [usar la API de invitación](customize-invitation-api.md). Al ser un área de texto, por motivos de seguridad no se procesan las etiquetas HTML.

  ![Captura de pantalla muestra el cuerpo del correo electrónico de invitación](media/invitation-email-elements/invitation-email-body.png)

### <a name="footer-section"></a>Sección de pie de página
El pie de página contiene la marca de empresa de Microsoft y permite que el destinatario sepa si el correo electrónico se envía desde un alias no supervisado. 

Casos especiales:

- El invitador no tiene una dirección de correo electrónico en el espacio empresarial invitador.

  ![Captura de pantalla cuando un autor de la invitación no tiene el correo electrónico en el espacio empresarial invitador.](media/invitation-email-elements/inviter-no-email.png)


- El destinatario no necesita canjear la invitación.

  ![Captura de pantalla cuando el destinatario no necesita canjear la invitación](media/invitation-email-elements/when-recipient-doesnt-redeem.png)

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
