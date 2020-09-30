---
title: Marco de consentimiento de Azure AD
titleSuffix: Microsoft identity platform
description: Obtenga más información sobre el marco de consentimiento de Azure AD y cómo facilita el desarrollo de aplicaciones web multiinquilino y aplicaciones de cliente nativas.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/30/2018
ms.author: ryanwi
ms.reviewer: zachowd, lenalepa, jesakowi
ms.custom: aaddev, has-adal-ref
ms.openlocfilehash: e9780332ad6279deef63910c7e6ba95e1ccf43bd
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90706139"
---
# <a name="azure-active-directory-consent-framework"></a>Marco de consentimiento de Azure Active Directory

El marco de consentimiento de Azure AD facilita el desarrollo de aplicaciones web multiinquilino y aplicaciones de cliente nativas. Estas aplicaciones permiten que las cuentas de usuario inicien sesión desde un inquilino de Azure AD diferente del inquilino en el que se ha registrado la aplicación. También pueden necesitar acceder a diferentes API web, como Microsoft Graph API (para tener acceso a Azure AD, Intune y los servicios de Microsoft 365) y otras API de servicios de Microsoft, además de sus propias API web.

El marco se basa en que un usuario o un administrador da consentimiento a una aplicación que solicita el registro en su directorio, lo cual puede implicar el acceso a los datos de directorio. Por ejemplo, si una aplicación cliente web debe leer información del calendario sobre el usuario desde Microsoft 365, se pide primero a ese usuario que de su consentimiento a la aplicación cliente. Después de dar su consentimiento, la aplicación cliente podrá llamar a Microsoft Graph API en nombre del usuario y usar la información de calendario según sea necesario. [Microsoft Graph API](https://developer.microsoft.com/graph) proporciona acceso a datos de Microsoft 365 (como calendarios y mensajes de Exchange, sitios y listas de SharePoint, documentos de OneDrive, blocs de notas de OneNote, tareas de Planner y libros de trabajo de Excel), así como a usuarios y grupos de Azure AD y otros objetos de datos de más servicios en la nube de Microsoft.

El marco de consentimiento se basa en OAuth 2.0 y sus distintos flujos, como la concesión de credenciales de cliente y la concesión de código de autorización, mediante clientes públicos o confidenciales. Con OAuth 2.0, Azure AD permite crear muchos tipos diferentes de aplicaciones cliente, como en un teléfono, tableta, servidor o una aplicación web, y obtener acceso a los recursos necesarios.

Para más información sobre el uso del marco de consentimiento con concesiones de autorización de OAuth2.0, consulte [Autorización del acceso a aplicaciones web mediante OAuth 2.0 y Azure Active Directory](v2-oauth2-auth-code-flow.md) y [Escenarios de autenticación para Azure AD](./authentication-vs-authorization.md). Para obtener más información sobre cómo obtener acceso autorizado a Microsoft 365 mediante Microsoft Graph, consulte [Autenticación de la aplicación con Microsoft Graph](/graph/).

## <a name="consent-experience---an-example"></a>Ejemplo de experiencia de consentimiento

Los siguientes pasos muestran cómo funciona la experiencia de consentimiento para el desarrollador de la aplicación y para el usuario.

1. Suponga que tiene una aplicación de cliente web que necesita solicitar permisos concretos para tener acceso a un recurso o API. Obtendrá información sobre cómo realizar esta configuración en la sección siguiente, pero básicamente Azure Portal se utiliza para declarar solicitudes de permisos en el momento de la configuración. Al igual que otras opciones de configuración, pasan a formar parte del registro de Azure AD de la aplicación:

    ![Permisos para otras aplicaciones](./media/consent-framework/permissions.png)

1. Plantéese la posibilidad de que los permisos de la aplicación estén actualizados, la aplicación se ejecute y un usuario esté a punto de usarla por primera vez. En primer lugar, la aplicación debe obtener un código de autorización del punto de conexión `/authorize` de Azure AD. El código de autorización, a continuación, se puede utilizar para adquirir un nuevo acceso y un token de actualización.

1. Si el usuario no está aún autenticado, el punto de conexión `/authorize` de Azure AD le pide que inicie sesión.

    ![Inicio de sesión de usuario o administrador en Azure AD](./media/consent-framework/usersignin.png)

1. Cuando el usuario inicie sesión, Azure AD determinará si se debe mostrar una página de consentimiento al usuario. Esta determinación se basa en que el usuario (o el administrador de la organización) ya concediese el consentimiento de la aplicación. Si todavía no se concedió el consentimiento, Azure AD se lo pide al usuario y muestra los permisos necesarios para que funcione. El conjunto de permisos que aparecen en el cuadro de diálogo de consentimiento coinciden con los que se seleccionaron en **Permisos delegados** en Azure Portal.

    ![Se muestra un ejemplo de los permisos que aparecen en el cuadro de diálogo de consentimiento](./media/consent-framework/consent.png)

1. Después de que el usuario concede el consentimiento, se devuelve un código de autorización a la aplicación, que se canjea para adquirir un token de acceso y un token de actualización. Para más información sobre este flujo, consulte [Plataforma de identidad y flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md).

1. Como administrador, también puede dar su consentimiento para permisos delegados de una aplicación en nombre de todos los usuarios del inquilino. El consentimiento del administrador evita que el cuadro de diálogo de consentimiento aparezca para cada usuario del inquilino y lo pueden llevar a cabo los usuarios con el rol de administrador en [Azure Portal](https://portal.azure.com). Para información sobre qué roles de administrador pueden consentir los permisos delegados, consulte [Permisos de roles de administrador en Azure AD](../users-groups-roles/directory-assign-admin-roles.md).

    **Para consentir los permisos delegados de una aplicación**

   1. Vaya a la página **Permisos de API** de la aplicación.
   1. Haga clic en el botón **Conceder consentimiento de administrador**.

      ![Concesión de permisos para el consentimiento explícito del administrador](./media/consent-framework/grant-consent.png)

   > [!IMPORTANT]
   > Actualmente es obligatorio conceder explícitamente el consentimiento con el botón **Conceder permisos** para las aplicaciones de página única (SPA) que usan ADAL.js. En caso contrario, se produce un error en la aplicación cuando se solicita el token de acceso.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Conversión de una aplicación de inquilino único en una aplicación multiinquilino](howto-convert-app-to-be-multi-tenant.md)
* Para más detalles, aprenda [cómo se admite el consentimiento en la capa del protocolo OAuth 2.0 durante el flujo de concesión de código de autorización](../azuread-dev/v1-protocols-oauth-code.md#request-an-authorization-code).