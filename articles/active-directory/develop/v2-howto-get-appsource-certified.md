---
title: Certificación de AppSource para Azure Active Directory | Microsoft Docs
description: Información detallada sobre cómo certificar su instancia de AppSource de la aplicación para Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/23/2020
ms.author: ryanwi
ms.reviewer: jeedes
ms.custom: aaddev
ms.openlocfilehash: 08acaf4f0171e586ff2cc3f52134395fb5925df7
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118814"
---
# <a name="get-appsource-certified-for-azure-active-directory"></a>Certificación de AppSource para Azure Active Directory

[Microsoft AppSource](https://appsource.microsoft.com/) es un destino para que los usuarios empresariales detecten, prueben y administren aplicaciones SaaS de línea de negocio (SaaS independiente y un complemento para productos SaaS de Microsoft existentes).

Para que una aplicación SaaS independiente se muestre en AppSource, la aplicación debe aceptar el inicio de sesión único desde cuentas profesionales de cualquier compañía u organización que cuente con Azure Active Directory (Azure AD). El proceso de inicio de sesión debe usar los protocolos [OpenID Connect](v2-protocols-oidc.md) o [OAuth 2.0](v2-oauth2-auth-code-flow.md). La integración de SAML no se acepta como certificación de AppSource.

## <a name="multi-tenant-applications"></a>Aplicaciones multiinquilino

Una *aplicación multiempresa* es una aplicación que acepta inicios de sesión de usuarios de cualquier compañía u organización que tenga Azure AD, sin que se requiera otra instancia, configuración o implementación. AppSource recomienda que las aplicaciones implementen una arquitectura multiempresa para habilitar la experiencia de evaluación gratuita *con un solo clic*.

Para habilitar la arquitectura multiempresa en la aplicación, siga estos pasos:
1. Establezca la propiedad `Multi-Tenanted` en `Yes` en la información de registro de la aplicación de [Azure Portal](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps). De forma predeterminada, las aplicaciones creadas en Azure Portal se configuran como *[inquilino único](#single-tenant-applications)* .
1. Actualice el código para enviar solicitudes al punto de conexión `common`. Para ello, actualice el punto de conexión de `https://login.microsoftonline.com/{yourtenant}` a `https://login.microsoftonline.com/common*`.
1. En algunas plataformas, como ASP.NET, también hay que actualizar el código para que acepte varios emisores.

Para más información sobre la arquitectura multiempresa, vea [Inicio de sesión de cualquier usuario de Azure Active Directory (Azure AD) mediante el patrón de aplicación multiempresa](howto-convert-app-to-be-multi-tenant.md).

### <a name="single-tenant-applications"></a>Aplicaciones de inquilino único

Una *aplicación de inquilino único* es una aplicación que solo aceptan inicios de sesión de usuarios de una instancia de Azure AD definida. Los usuarios externos (incluidas las cuentas profesionales o educativas de otras organizaciones o una cuenta personal) pueden iniciar sesión en una aplicación de inquilino único después de agregar cada usuario como cuenta de invitado a la instancia de Azure AD en la que esa aplicación está registrada. 

Puede agregar usuarios como cuentas de invitado a una instancia de Azure AD a través de la [Colaboración B2B en Azure AD](../external-identities/what-is-b2b.md), que se puede realizar [mediante programación](../../active-directory-b2c/code-samples.md). Al usar B2B, los usuarios pueden crear un portal de autoservicio que no requiera invitación para iniciar sesión. Para obtener más información, consulte [Portal de autoservicio para el registro en la colaboración B2B de Azure AD](../external-identities/self-service-portal.md).

Las aplicaciones de inquilino único pueden habilitar la experiencia *Ponerse en contacto conmigo*, pero si se quiere habilitar la experiencia de evaluación gratuita/con un solo clic que AppSource recomienda, habilite en su lugar la arquitectura multiempresa en la aplicación.

## <a name="appsource-trial-experiences"></a>Experiencias de evaluación gratuita de AppSource

### <a name="free-trial-customer-led-trial-experience"></a>Evaluación gratuita (experiencia de evaluación dirigida por el usuario)

La evaluación dirigida por el usuario es la experiencia que AppSource recomienda porque ofrece acceso con un solo clic a la aplicación. En el ejemplo siguiente se muestra el aspecto de esta experiencia:

1.  Un usuario busca la aplicación en el sitio web de AppSource y, a continuación, selecciona la opción **Evaluación gratuita**.

    ![Se muestra la experiencia de evaluación dirigida por el usuario de la evaluación gratuita](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step1.png)

2.  AppSource redirige al usuario a una dirección URL en su sitio web. El sitio web inicia el proceso de  *inicio de sesión único* automáticamente (al cargar la página).

    ![Se muestra cómo se redirige al usuario a una dirección URL en el sitio web](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step2.png)

3.  Se redirige al usuario a la página de inicio de sesión de Microsoft y este proporciona las credenciales para iniciar sesión.

    ![Se muestra la página de inicio de sesión de Microsoft](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step3.png)

4. El usuario da su consentimiento a la aplicación.

    ![Ejemplo: Página de consentimiento de una aplicación](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step4.png)

5.  Se completa el inicio de sesión y se redirige al usuario de vuelta al sitio web.  El usuario inicia la evaluación gratuita.

    ![Se muestra la experiencia que el usuario ve cuando se redirige de vuelta a su sitio](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step5.png)

### <a name="contact-me-partner-led-trial-experience"></a>Ponerse en contacto conmigo (experiencia de evaluación dirigida por el partner)

Puede usar la experiencia de evaluación de asociado cuando tiene que producirse una operación manual o a largo plazo para aprovisionar al usuario o la compañía: por ejemplo, la aplicación tiene que aprovisionar máquinas virtuales, instancias de base de datos u operaciones que tardan mucho en completarse. En este caso, después de que el usuario seleccione el botón **Solicitar versión de prueba** y rellene un formulario, AppSource le envía la información de contacto del usuario. Al recibir esta información, se aprovisiona el entorno y se envían al usuario las instrucciones sobre cómo acceder a la experiencia de evaluación:<br/><br/>

1. Un usuario busca la aplicación en el sitio web de AppSource y, a continuación, selecciona la opción **Ponerse en contacto conmigo**.

    ![Se muestra la opción Ponerse en contacto conmigo de la experiencia de evaluación dirigida por el partner](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step1.png)

2. El usuario rellena un formulario con información de contacto.

    ![Se muestra un formulario de ejemplo con información de contacto](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step2.png)

3. Recibirá la información del usuario, configurará una instancia de prueba y enviará al usuario un hipervínculo para acceder a la aplicación.

    ![Se muestra el marcador de posición de la información de usuario](./media/active-directory-devhowto-appsource-certified/usercontact.png)

4. El usuario accede a la aplicación y completa el proceso de inicio de sesión único.

    ![Se muestra la pantalla de inicio de sesión de la aplicación](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step3.png)

5. El usuario da su consentimiento a la aplicación.

    ![Se muestra un ejemplo de página de consentimiento de una aplicación](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step4.png)

6. Se completa el inicio de sesión y se redirige al usuario de vuelta al sitio web. El usuario inicia la evaluación gratuita.

    ![Se muestra la experiencia que el usuario ve cuando se redirige de vuelta a su sitio](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step5.png)

### <a name="more-information"></a>Más información

Para más información sobre la experiencia de evaluación de AppSource, vea [este vídeo](https://aka.ms/trialexperienceforwebapps). 

## <a name="get-support"></a>Obtención de soporte técnico

Para la integración de Azure AD, se usa [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-active-directory+appsource) con la comunidad para ofrecer soporte técnico.

Se recomienda muy especialmente que primero plantee sus preguntas sobre Stack Overflow y examine los problemas existentes para ver si algún usuario ha hecho esa pregunta antes. Asegúrese de que sus preguntas o comentarios se etiquetan con [`[azure-active-directory]` y `[appsource]`](https://stackoverflow.com/questions/tagged/azure-active-directory+appsource).

Use la siguiente sección de comentarios para proporcionar sus opiniones y ayudarnos a afinar y remodelar el contenido.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo generar aplicaciones que admitan inicios de sesión de Azure AD, vea [Escenarios de autenticación para Azure AD](authentication-flows-app-scenarios.md).
- Para información sobre cómo mostrar la aplicación SaaS en AppSource, vea [AppSource Partner Information](https://appsource.microsoft.com/partners) (Información sobre el partner de AppSource).