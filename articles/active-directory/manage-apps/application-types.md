---
title: Visualización de aplicaciones con el inquilino de Azure Active Directory para la administración de identidades
description: Comprenda cómo visualizar todas las aplicaciones con el inquilino de Azure Active Directory para la administración de identidades.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: kenwith
ms.openlocfilehash: 955efe45be27fa2f4e738066bb3b69b3604be33a
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400728"
---
# <a name="viewing-apps-using-your-azure-ad-tenant-for-identity-management"></a>Visualización de las aplicaciones con el inquilino de Azure AD para la administración de identidades
En [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md) se explican los aspectos básicos. Aquí aprenderá cómo visualizar todas las aplicaciones con el inquilino de Azure AD para la administración de identidades. En este artículo se profundiza un poco más en los tipos de aplicaciones que encontrará.

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>¿Por qué aparece una aplicación específica en mi lista de todas las aplicaciones?
Cuando se filtra por **todas las aplicaciones**, la **lista** **Todas las aplicaciones** muestra todos los objetos de entidad de servicio del inquilino. Los objetos de entidad de servicio pueden aparecer en esta lista de varias formas:
- Cuando agrega cualquier aplicación desde la galería de aplicaciones, incluidas:
   - **Azure AD: Aplicaciones empresariales**. Aplicaciones agregadas al inquilino mediante la opción **Aplicaciones empresariales** en el portal de Azure AD. Normalmente, las aplicaciones se integran con el estándar SAML.
   - **Azure AD: Registros de aplicaciones**. Aplicaciones agregadas al inquilino con la opción **Registros de aplicaciones** en el portal de Azure AD. Normalmente, aplicaciones desarrolladas de forma personalizada con los estándares Open ID Connect y OAuth.
   - **Aplicaciones de proxy de aplicación**: una aplicación que se ejecuta en su entorno local en la que quiere que se proporcione inicio de sesión seguro externamente.
- Al registrarse o iniciar sesión en una aplicación de terceros integrada con Azure Active Directory. Un ejemplo de esto es [Smartsheet](https://app.smartsheet.com/b/home) o [DocuSign](https://www.docusign.net/member/MemberLogin.aspx).
- Aplicaciones de Microsoft como Microsoft 365 u Office 365.
- Al agregar un nuevo registro de aplicación mediante la creación de una aplicación desarrollada de forma personalizada con el [Registro de aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-app-registration).
- Al agregar un nuevo registro de aplicación mediante la creación de una aplicación desarrollada de forma personalizada con el [Portal de registro de aplicaciones V2.0](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-app-registration).
- Al agregar una aplicación, el desarrollo se lleva a cabo con los [métodos de autenticación de ASP.NET](https://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions) o los [servicios conectados](https://devblogs.microsoft.com/visualstudio/connecting-to-cloud-services/) de Visual Studio.
- Al crear un objeto de entidad de servicio mediante el [módulo de PowerShell de Azure AD](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).
- Al [dar su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) como administrador para que utilice datos de su inquilino.
- Cuando un [usuario da su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) para que utilice datos de su inquilino.
- Al habilitar determinados servicios que almacenan datos en el inquilino. Un ejemplo de esto es Password Reset, que se modela como entidad de servicio para almacenar la directiva de restablecimiento de contraseña de forma segura.

Para averiguar cómo y por qué las aplicaciones se agregan a su directorio, lea [Cómo se agregan aplicaciones a Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-how-applications-are-added).

## <a name="next-steps"></a>Pasos siguientes
[Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
