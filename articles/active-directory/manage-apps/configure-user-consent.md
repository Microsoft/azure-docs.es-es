---
title: Configuración del consentimiento del usuario en una aplicación en Azure Active Directory | Microsoft Docs
description: Aprenda a administrar el modo en que los usuarios dan su consentimiento a los permisos de la aplicación. Puede simplificar la experiencia del usuario proporcionando consentimiento de administrador. Estos métodos se aplican a todos los usuarios finales del inquilino de Azure Active Directory (Azure AD).
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/22/2018
ms.author: mimart
ms.reviewer: arvindh
ms.collection: M365-identity-device-management
ms.openlocfilehash: bd84b44c89c856036640757b7760d7c10bd36173
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824842"
---
# <a name="configure-the-way-end-users-consent-to-an-application-in-azure-active-directory"></a>Configure el modo en que los usuarios finales dan su consentimiento a una aplicación en Azure Active Directory.
Aprenda a configurar el modo en que los usuarios dan su consentimiento en los permisos de la aplicación. Puede simplificar la experiencia del usuario proporcionando consentimiento de administrador. En este artículo, se explican diferentes métodos para configurar el consentimiento del usuario. Estos métodos se aplican a todos los usuarios finales del inquilino de Azure Active Directory (Azure AD). 

Para más información acerca de cómo dar consentimiento a las aplicaciones, consulte [Marco de consentimiento de Azure Active Directory](../develop/consent-framework.md).

## <a name="prerequisites"></a>Requisitos previos

Para otorgar consentimiento de administrador, debe iniciar sesión como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.

Para restringir el acceso a las aplicaciones, debe requerir la asignación de usuarios y, luego, asignar usuarios o grupos a la aplicación.  Para más información, consulte [Métodos para asignar usuarios y grupos](methods-for-assigning-users-and-groups.md).

## <a name="grant-admin-consent-to-enterprise-apps-in-the-azure-portal"></a>Concesión de consentimiento en aplicaciones empresariales mediante Azure Portal

Para otorgar consentimiento a una aplicación empresarial:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global, administrador de una aplicación o administrador de una aplicación en la nube.
2. Haga clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo. Se abrirá la **extensión de Azure Active Directory**
3. En el cuadro de búsqueda del filtro, escriba **"Azure Active Directory"** y seleccione el elemento **Azure Active Directory**.
4. En el menú de navegación, haga clic en **Aplicaciones empresariales**.
5. Haga clic en **Conceder consentimiento de administrador**. Se le pedirá que inicie sesión para administrar la aplicación.
6. Inicie sesión con una cuenta que tenga permisos para conceder consentimiento de administrador a la aplicación. 
7. Dé su consentimiento a los permisos de la aplicación.

Esta opción solo funciona si la aplicación: 

- está registrada en su inquilino; o bien,
- está registrada en otro inquilino de Azure AD y ha recibido el consentimiento, como mínimo, de un usuario final. Una vez que un usuario final ha dado su consentimiento a una aplicación, Azure AD muestra la aplicación bajo **Aplicaciones empresariales** en Azure Portal.

## <a name="grant-admin-consent-when-registering-an-app-in-the-azure-portal"></a>Concesión de consentimiento de administrador al registrar una aplicación en Azure Portal

Para conceder consentimiento de administrador al registrar una aplicación: 

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.
2. Vaya a la hoja **Registros de aplicaciones**.
3. Seleccione la aplicación a la que quiere dar su consentimiento.
4. Seleccione **Permisos necesarios**.
5. Haga clic en **Conceder permisos** en la parte superior de la hoja.


## <a name="grant-admin-consent-through-a-url-request"></a>Concesión de consentimiento de administrador mediante una solicitud URL

Para otorgar consentimiento de administrador mediante una solicitud URL:

1. Cree una solicitud para *login.microsoftonline.com* con la configuración de la aplicación y anéxela a `&prompt=admin_consent`. 
2. Una vez que la sesión se haya iniciado con las credenciales del administrador, la aplicación tendrá consentimiento para todos los usuarios.


## <a name="force-user-consent-through-a-url-request"></a>Aplicación forzosa del consentimiento del administrador mediante una solicitud URL

Para hacer que los usuarios finales den su consentimiento en una aplicación cada vez que se autentiquen, anexe `&prompt=consent` a la dirección URL de la solicitud de autenticación.

## <a name="next-steps"></a>Pasos siguientes

[Consentimiento e integración de aplicaciones en Azure AD](../develop/quickstart-v1-integrate-apps-with-azure-ad.md)

[Consentimiento y permisos para las aplicaciones convergentes v2.0 de Azure AD](../develop/active-directory-v2-scopes.md)

[Azure AD en StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)
