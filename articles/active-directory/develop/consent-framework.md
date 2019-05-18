---
title: Marco de consentimiento de Azure Active Directory
description: Obtenga más información sobre el marco de consentimiento de Azure AD y cómo facilita el desarrollo de aplicaciones web multiinquilino y aplicaciones de cliente nativas.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/30/2018
ms.author: ryanwi
ms.reviewer: zachowd, lenalepa, jesakowi
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 93adedc5c1343df1eee05b653b60cfd7e810044c
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540422"
---
# <a name="azure-active-directory-consent-framework"></a>Marco de consentimiento de Azure Active Directory

El marco de consentimiento de Azure AD facilita el desarrollo de aplicaciones web multiinquilino y aplicaciones de cliente nativas. Estas aplicaciones permiten que las cuentas de usuario inicien sesión desde un inquilino de Azure AD diferente del inquilino en el que se ha registrado la aplicación. También pueden necesitar acceder a diferentes API web, como Microsoft Graph API (para tener acceso a Azure AD, Intune y los servicios de Office 365) y otras API de servicios de Microsoft, además de sus propias API web.

El marco se basa en que un usuario o un administrador da consentimiento a una aplicación que solicita el registro en su directorio, lo cual puede implicar el acceso a los datos de directorio. Por ejemplo, si una aplicación cliente web debe leer información del calendario sobre el usuario desde Office 365, se pide primero a ese usuario que de su consentimiento a la aplicación cliente. Después de dar su consentimiento, la aplicación cliente podrá llamar a Microsoft Graph API en nombre del usuario y usar la información de calendario según sea necesario. [Microsoft Graph API](https://developer.microsoft.com/graph) proporciona acceso a datos de Office 365 (como calendarios y mensajes de Exchange, sitios y listas de SharePoint, documentos de OneDrive, blocs de notas de OneNote, tareas de Planner y libros de trabajo de Excel), así como a usuarios y grupos de Azure AD y otros objetos de datos de más servicios en la nube de Microsoft.

El marco de consentimiento se basa en OAuth 2.0 y sus distintos flujos, como la concesión de credenciales de cliente y la concesión de código de autorización, mediante clientes públicos o confidenciales. Con OAuth 2.0, Azure AD permite crear muchos tipos diferentes de aplicaciones cliente, como en un teléfono, tableta, servidor o una aplicación web, y obtener acceso a los recursos necesarios.

Para más información sobre el uso del marco de consentimiento con concesiones de autorización de OAuth2.0, consulte [Autorización del acceso a aplicaciones web mediante OAuth 2.0 y Azure Active Directory](v1-protocols-oauth-code.md) y [Escenarios de autenticación para Azure AD](authentication-scenarios.md). Para más información sobre cómo obtener acceso autorizado a Office 365 mediante Microsoft Graph, consulte [Autenticación de la aplicación con Microsoft Graph](https://developer.microsoft.com/graph/docs/authorization/auth_overview).

## <a name="consent-experience---an-example"></a>Ejemplo de experiencia de consentimiento

Los siguientes pasos muestran cómo funciona la experiencia de consentimiento para el desarrollador de la aplicación y para el usuario.

1. Suponga que tiene una aplicación de cliente web que necesita solicitar permisos concretos para tener acceso a un recurso o API. Obtendrá información sobre cómo realizar esta configuración en la sección siguiente, pero básicamente Azure Portal se utiliza para declarar solicitudes de permisos en el momento de la configuración. Al igual que otras opciones de configuración, pasan a formar parte del registro de Azure AD de la aplicación:

    ![Permisos para otras aplicaciones](./media/quickstart-v1-integrate-apps-with-azure-ad/requiredpermissions.png)

1. Plantéese la posibilidad de que los permisos de la aplicación estén actualizados, la aplicación se ejecute y un usuario esté a punto de usarla por primera vez. En primer lugar, la aplicación debe obtener un código de autorización del punto de conexión `/authorize` de Azure AD. El código de autorización, a continuación, se puede utilizar para adquirir un nuevo acceso y un token de actualización.

1. Si el usuario no está aún autenticado, el punto de conexión `/authorize` de Azure AD le pide que inicie sesión.

    ![Inicio de sesión de usuario o administrador en Azure AD](./media/quickstart-v1-integrate-apps-with-azure-ad/usersignin.png)

1. Cuando el usuario inicie sesión, Azure AD determinará si se debe mostrar una página de consentimiento al usuario. Esta determinación se basa en que el usuario (o el administrador de la organización) ya concediese el consentimiento de la aplicación. Si todavía no se concedió el consentimiento, Azure AD se lo pide al usuario y muestra los permisos necesarios para que funcione. El conjunto de permisos que aparecen en el cuadro de diálogo de consentimiento coinciden con los que se seleccionaron en **Permisos delegados** en Azure Portal.

    ![Experiencia de consentimiento de usuario](./media/quickstart-v1-integrate-apps-with-azure-ad/consent.png)

1. Después de que el usuario concede el consentimiento, se devuelve un código de autorización a la aplicación, que se canjea para adquirir un token de acceso y un token de actualización. Para obtener más información acerca de este flujo, consulte [Tipo de aplicaciones de API web](web-api.md).

1. Como administrador, también puede dar su consentimiento para permisos delegados de una aplicación en nombre de todos los usuarios del inquilino. El consentimiento del administrador evita que el cuadro de diálogo de consentimiento aparezca para cada usuario del inquilino y lo pueden llevar a cabo los usuarios con el rol de administrador en [Azure Portal](https://portal.azure.com). Para información sobre qué roles de administrador pueden consentir los permisos delegados, consulte [Permisos de roles de administrador en Azure AD](../users-groups-roles/directory-assign-admin-roles.md).

    **Para consentir los permisos delegados de una aplicación**

   1. Vaya a la página **Configuración** de la aplicación.
   1. Seleccione **Permisos necesarios**.
   1. Haga clic en el botón **Conceder permisos**.

      ![Concesión de permisos para el consentimiento explícito del administrador](./media/quickstart-v1-integrate-apps-with-azure-ad/grantpermissions.png)

   > [!IMPORTANT]
   > Actualmente es obligatorio conceder explícitamente el consentimiento con el botón **Conceder permisos** para las aplicaciones de página única (SPA) que usan ADAL.js. En caso contrario, se produce un error en la aplicación cuando se solicita el token de acceso.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Conversión de una aplicación de inquilino único en una aplicación multiinquilino](howto-convert-app-to-be-multi-tenant.md)
* Para más detalles, aprenda [cómo se admite el consentimiento en la capa del protocolo OAuth 2.0 durante el flujo de concesión de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code#request-an-authorization-code).
