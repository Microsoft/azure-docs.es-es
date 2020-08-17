---
title: 'Introducción a la plataforma de identidad de Microsoft: Azure'
description: Información acerca de los componentes de la plataforma de identidad de Microsoft y cómo pueden ayudarle a crear administración de identidad y acceso (IAM) en sus aplicaciones.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: overview
ms.workload: identity
ms.date: 07/09/2020
ms.author: ryanwi
ms.reviewer: agirling, saeeda, benv
ms.custom: identityplatformtop40
ms.openlocfilehash: e34eeeca3b17087fb5bd43ae353e43b5ab355831
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119562"
---
# <a name="microsoft-identity-platform-overview"></a>Introducción a la plataforma de identidad de Microsoft

La plataforma de identidad de Microsoft le ayuda a compilar aplicaciones en las que los usuarios y los clientes inician sesión mediante sus identidades de Microsoft o cuentas de redes sociales, y a proporcionar acceso autorizado a sus API o a las API de Microsoft como Microsoft Graph.

La plataforma de identidad de Microsoft está formada por varios componentes:

- **Servicio de autenticación compatible con los estándares OAuth 2.0 y OpenID Connect** que permite a los desarrolladores autenticar varios tipos de identidades entre las que se incluyen:
  - Cuentas profesionales o educativas (aprovisionadas mediante Azure AD)
  - Cuentas personales de Microsoft (por ejemplo, Skype, Xbox y Outlook.com)
  - Cuentas locales o sociales mediante Azure AD B2C
- **Bibliotecas de código abierto**: bibliotecas de autenticación de Microsoft (MSAL) y compatibilidad con cualquier otra biblioteca que cumpla con los estándares.
- **Portal de administración de aplicaciones**: una experiencia de registro y configuración en Azure Portal, junto con las demás funcionalidades de administración de Azure.
- **PowerShell y API de configuración de aplicaciones**: configuración mediante programación de las aplicaciones con Microsoft Graph API y PowerShell, para que pueda automatizar las tareas de DevOps.
- **Contenido para desarrolladores**: documentación técnica que incluye inicios rápidos, tutoriales, guías paso a paso y ejemplos de código.

La plataforma de identidad de Microsoft ofrece a los desarrolladores una integración con innovaciones recientes en el espacio de identidad y seguridad, como la autenticación sin contraseña, la autenticación de nivel superior y el acceso condicional. No es necesario que implemente esta funcionalidad manualmente: las aplicaciones integradas de manera nativa con la Plataforma de identidad de Microsoft aprovechan estas innovaciones.

Con la plataforma de identidad de Microsoft, puede escribir código una vez y llegar a cualquier usuario. Puede compilar una aplicación una sola vez y conseguir que funcione en muchas plataformas, o bien compilar una aplicación que funcione como un cliente, así como una aplicación de recursos (API).

## <a name="getting-started"></a>Introducción

Elija el [escenario de aplicación](authentication-flows-app-scenarios.md) que desearía compilar. Cada una de estas rutas de acceso de escenario comienza con una introducción y vínculos a una guía de inicio rápido que le ayudarán a empezar a trabajar:

- [Aplicación de página única (SPA)](scenario-spa-overview.md)
- [Aplicación web que permite iniciar sesión a los usuarios](scenario-web-app-sign-user-overview.md)
- [Aplicación web que llama a las API web](scenario-web-app-call-api-overview.md)
- [API web protegida](scenario-protected-web-api-overview.md)
- [API web que llama a API web](scenario-web-api-call-api-overview.md)
- [Aplicación de escritorio](scenario-desktop-overview.md)
- [Aplicación de demonio](scenario-daemon-overview.md)
- [Aplicación móvil](scenario-mobile-overview.md)

Cuando trabaje con la plataforma de identidad de Microsoft para integrar autenticación y autorización en sus aplicaciones, puede acudir a esta imagen que describe los escenarios de aplicación más comunes y sus componentes de identidad. Seleccione la imagen para verla a tamaño completo.

[![Mapa tipo metro que muestra varios escenarios de aplicación en la plataforma de identidad de Microsoft](./media/v2-overview/application-scenarios-identity-platform.png)](./media/v2-overview/application-scenarios-identity-platform.svg#lightbox)

## <a name="learn-authentication-concepts"></a>Conceptos de autenticación

Aprenda cómo se aplican los conceptos básicos de autenticación y de Azure AD a la plataforma de identidad de Microsoft en este conjunto de artículos recomendado:

- [Conceptos básicos sobre autenticación](./authentication-vs-authorization.md)
- [Aplicaciones y entidades de servicio](app-objects-and-service-principals.md)
- [Audiencias](v2-supported-account-types.md)
- [Permisos y consentimiento](v2-permissions-and-consent.md)
- [Tokens de identificador](id-tokens.md)
- [Tokens de acceso](access-tokens.md)
- [Flujos de autenticación y escenarios de aplicaciones](authentication-flows-app-scenarios.md)

## <a name="more-identity-and-access-management-options"></a>Más opciones de administración de identidades y acceso

[Azure AD B2C](../../active-directory-b2c/overview.md): compile aplicaciones orientadas al cliente en las que los usuarios puedan iniciar sesión con sus cuentas de redes sociales, como Facebook o Google, o mediante una dirección de correo electrónico y una contraseña.

[Azure AD B2B](../external-identities/what-is-b2b.md): invite a usuarios externos a su inquilino Azure AD como usuarios "invitados" a los que puede asignar permisos para la autorización, al tiempo que les permite usar sus credenciales existentes para la autenticación.

> [!TIP]
> ¿Busca documentación sobre la *plataforma para desarrolladores de Azure Active Directory (v 1.0)* ? Consulte [Introducción a Azure Active Directory para desarrolladores (v1.0)](../azuread-dev/v1-overview.md).

## <a name="next-steps"></a>Pasos siguientes

Si tiene una cuenta de Azure ya tiene acceso a un inquilino de Azure Active Directory, pero la mayoría de los desarrolladores de la plataforma de identidad de Microsoft necesitan su propio inquilino de Azure AD, un "inquilino de desarrollo", para usarlo durante el desarrollo de aplicaciones.

Aprenda a crear su propio inquilino para usarlo cuando compile las aplicaciones:

[Inicio rápido: Configuración de un inquilino de Azure AD](quickstart-create-new-tenant.md)