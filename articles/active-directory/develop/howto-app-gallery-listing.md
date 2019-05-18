---
title: Exposición de una aplicación en la galería de aplicaciones de Azure Active Directory | Microsoft Docs
description: Información sobre cómo mostrar una aplicación compatible con el inicio de sesión único en la galería de aplicaciones de Azure Active Directory
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/08/2019
ms.author: ryanwi
ms.reviewer: elisol, bryanla
ms.custom: aaddev, seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: 005e5c92a16760d8eec5dc37526f4b1f2dbd751c
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540547"
---
# <a name="how-to-list-your-application-in-the-azure-active-directory-application-gallery"></a>Procedimientos para: Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory

En este artículo se muestra cómo agregar una aplicación de la Galería de aplicaciones de Azure AD, implementar el inicio de sesión único (SSO) y administrar la lista.

## <a name="what-is-the-azure-ad-application-gallery"></a>¿Qué es la galería de aplicaciones de Azure AD?

- Los clientes pueden disfrutar de la mejor experiencia posible de inicio de sesión único.
- La configuración de la aplicación resulta sencilla y apenas requiere esfuerzo.
- Basta realizar una búsqueda rápida para encontrar la aplicación en la galería.
- Todos los clientes de Azure AD de los niveles Gratis, Básico y Premium pueden disfrutar de esta integración.
- Los clientes mutuos pueden utilizar un tutorial de configuración paso a paso.
- Los clientes que usen SCIM pueden utilizar el aprovisionamiento de la misma aplicación.

## <a name="prerequisites"></a>Requisitos previos

- Para las aplicaciones federadas (Open ID y SAML/WS-Fed), la aplicación debe admitir el modelo SaaS para que se incluya en la galería de Azure AD. Las aplicaciones de la galería empresarial deben admitir varias configuraciones de cliente y no la de un cliente específico.

- Para OpenID Connect, la aplicación debe ser de varios inquilinos y para ella se debe  implementar correctamente el [marco de consentimiento de Azure AD](consent-framework.md). El usuario puede enviar la solicitud de inicio de sesión a un punto de conexión común para que los clientes puedas proporcionar su consentimiento a la aplicación. Puede controlar el acceso de usuario en función del identificador del inquilino y el UPN del usuario que se recibieron en el token.

- Para SAML 2.0 o WS-Fed, la aplicación debe ser capaz de realizar la integración de SSO de SAML o WS-Fed en modo SP o IDP. Asegúrese de que esto funcione correctamente antes de enviar la solicitud.

- Para que el SSO de contraseña, asegúrese de que la aplicación admita la autenticación por formulario para que se pueda realizar el almacenamiento de contraseña y hacer que el inicio de sesión único funcione del modo previsto.

- Para las solicitudes de aprovisionamiento automático de usuarios, la aplicación debe figurar en la galería con la característica de inicio de sesión único habilitada mediante SAML 2.0/WS-Fed. Puede solicitar SSO y aprovisionamiento de usuario juntos en el portal, si aún no aparecen.

>[!NOTE]
>Estamos ejecutando con un gran número de solicitudes del conector SCIM, por lo que nos hemos detenido teniendo las nuevas solicitudes en nuestro portal. Espere las solicitudes hasta nuevo aviso. Le pedimos disculpas por este retraso y las molestias que pueda haberle causado.

## <a name="submit-the-request-in-the-portal"></a>Envío de la solicitud en el portal

Cuando haya comprobado que la integración de aplicaciones funciona con Azure AD, envíe la solicitud de acceso en nuestro [portal de red de aplicaciones](https://microsoft.sharepoint.com/teams/apponboarding/Apps). Si tiene una cuenta de Office 365, utilícela para iniciar sesión en este portal. De lo contrario, utilice una cuenta Microsoft (como Outlook o Hotmail) para iniciar sesión.

Si aparece la siguiente página después del inicio de sesión, póngase en contacto con el [equipo de integración de SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>) y proporcione la cuenta de correo electrónico que desea usar para enviar la solicitud. A continuación, el equipo de Azure AD agregará la cuenta en el portal de redes de aplicaciones de Microsoft.

![Solicitud de acceso en el portal de SharePoint](./media/howto-app-gallery-listing/errorimage.png)

Una vez que se agrega la cuenta, puede iniciar sesión en el portal de redes de aplicaciones de Microsoft.

Y si aparece la siguiente página después de iniciar sesión, escriba en el cuadro de texto la justificación empresarial por la que necesita obtener acceso y seleccione **Solicitar acceso**.

  ![Solicitud de acceso en el portal de SharePoint](./media/howto-app-gallery-listing/accessrequest.png)

Nuestro equipo revisará los detalles y le proporcionará el acceso según corresponda. Una vez aprobada su solicitud, puede iniciar sesión en el portal y enviar la solicitud. Para ello, haga clic en el icono **Enviar solicitud (ISV)** de la página principal.

![Página principal del Portal de SharePoint](./media/howto-app-gallery-listing/homepage.png)

> [!NOTE]
> Si tiene algún problema para obtener acceso, póngase en contacto con el [equipo de integración del SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="implementing-sso-using-federation-protocol"></a>Implementación de SSO mediante el protocolo de federación

Para agregar una aplicación a la galería de Azure AD, debe implementar uno de los siguientes protocolos de federación compatibles con Azure AD y aceptar los términos y condiciones de la galería de aplicaciones de Azure AD. Lea los términos y condiciones de la galería de aplicaciones de Azure AD [aquí](https://azure.microsoft.com/support/legal/active-directory-app-gallery-terms/).

- **OpenID Connect**: para integrar la aplicación con Azure AD mediante el protocolo OpenID Connect, siga las [instrucciones de los desarrolladores](authentication-scenarios.md).

    ![Escala de tiempo para agregar la aplicación de OpenID Connect en la galería](./media/howto-app-gallery-listing/openid.png)

    * Si desea agregar la aplicación a la lista en la galería con OpenID Connect, seleccione **OpenID Connect & OAuth 2.0** (OAuth 2.0 y OpenID Connect) como arriba.
    * Si tiene algún problema para obtener acceso, póngase en contacto con el [equipo de integración del SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>). 

- **SAML 2.0** o **WS-Fed**: Si la aplicación es compatible con SAML 2.0, se puede integrar directamente con un inquilino de Azure AD; siga [estas instrucciones para agregar una aplicación personalizada](../active-directory-saas-custom-apps.md).

  ![Escala de tiempo para exponer la aplicación de SAML 2.0 o WS-Fed en la galería](./media/howto-app-gallery-listing/saml.png)

  * Si desea agregar la aplicación a la lista en la galería mediante **SAML 2.0** o **WS-Fed**, seleccione **SAMl 2.0/WS-Fed** como arriba.
  * Si tiene algún problema para obtener acceso, póngase en contacto con el [equipo de integración del SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="implementing-sso-using-password-sso"></a>Implementación de SSO mediante SSO de contraseña

Cree una aplicación web que tenga una página de inicio de sesión HTML para configurar el [inicio de sesión único basado en contraseña](../manage-apps/what-is-single-sign-on.md). El SSO basado en contraseña, también conocido como almacenamiento de contraseñas, permite administrar el acceso y las contraseñas de los usuarios en aplicaciones web que no admiten la federación de identidades. También es útil para escenarios en los que varios usuarios deben compartir la misma cuenta, como las cuentas de las aplicaciones de redes sociales de la organización.

![Escala de tiempo para agregar la aplicación de SSO con contraseña a la galería](./media/howto-app-gallery-listing/passwordsso.png)

* Si desea agregar la aplicación a la lista en la galería mediante SSO con contraseña, seleccione **Password SSO** (SSO con contraseña) como arriba.
* Si tiene algún problema para obtener acceso, póngase en contacto con el [equipo de integración del SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="updateremove-existing-listing"></a>Actualización o eliminación de una lista existente

Para actualizar o eliminar una aplicación existente en la galería de aplicaciones de Azure AD, primero debe enviar la solicitud al [portal de red de aplicaciones](https://microsoft.sharepoint.com/teams/apponboarding/Apps). Si tiene una cuenta de Office 365, utilícela para iniciar sesión en este portal. De lo contrario, utilice una cuenta Microsoft (como Outlook o Hotmail) para iniciar sesión.

- Seleccione la opción adecuada, tal como se muestra en la siguiente imagen:

    ![Escala de tiempo para agregar la aplicación SAML a la lista de la galería](./media/howto-app-gallery-listing/updateorremove.png)

    * Si quiere actualizar una aplicación existente, seleccione **Update existing application listing** (Actualizar la lista de aplicaciones existentes).
    * Si quiere quitar una aplicación existente de la galería de Azure AD, seleccione **Remove existing application listing** (Quitar lista de aplicaciones existentes).
    * Si tiene algún problema para obtener acceso, póngase en contacto con el [equipo de integración del SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>). 

## <a name="listing-requests-by-customers"></a>Lista de las solicitudes de clientes

Los clientes pueden enviar la solicitud de una solicitud de lista, haga clic en **aplicación solicita clientes** -> **enviar solicitud nueva**.

![Icono aplicaciones solicitadas de cliente](./media/howto-app-gallery-listing/customer-submit-request.png)

A continuación se presenta que el flujo de cliente solicitado aplicaciones:

![Flujo de las aplicaciones cliente ha solicitado](./media/howto-app-gallery-listing/customer-request.png)

## <a name="timelines"></a>Escalas de tiempo

El período de tiempo que dura el proceso para mostrar una aplicación de SAML 2.0 o WS-Fed en la galería es de entre 7 y 10 días laborables.

   ![Escala de tiempo para agregar la aplicación SAML a la lista de la galería](./media/howto-app-gallery-listing/timeline.png)

El período de tiempo que dura el proceso para mostrar una aplicación de OpenID Connect en la galería es de entre 2 y 5 días laborables.

   ![Escala de tiempo para agregar la aplicación SAML a la lista de la galería](./media/howto-app-gallery-listing/timeline2.png)

## <a name="escalations"></a>Extensiones

Si necesita ponerse en contacto con una instancia superior, envíe un correo electrónico al [equipo de integración de SSO de Azure AD](mailto:SaaSApplicationIntegrations@service.microsoft.com) a la dirección de correo electrónico SaaSApplicationIntegrations@service.microsoft.com y le responderemos lo antes posible.
