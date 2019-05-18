---
title: Aplicación inesperada en mi lista de aplicaciones | Microsoft Docs
description: Cómo ver todas las aplicaciones de su inquilino y comprender cómo aparecen estas en la lista Todas las aplicaciones en Aplicaciones empresariales
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: b1cb4eeb52d0680695bda266ad1a563b2ef5ee02
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65781093"
---
# <a name="unexpected-application-in-my-applications-list"></a>Aplicación inesperada en mi lista de aplicaciones

Este artículo le ayudará comprender cómo aparecen las aplicaciones en la lista **Todas las aplicaciones** en **Aplicaciones empresariales**. 

## <a name="how-to-see-all-applications-in-your-tenant"></a>Visualización de todas las aplicaciones del inquilino

Para ver todas las aplicaciones de su inquilino, debe usar el **Filtro** control para que aparezcan **todas las aplicaciones** en la lista **Todas las aplicaciones**. Siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

6.  Haga clic en el control **Filtro** en la parte superior de la **lista Todas las aplicaciones**.

7.  En el panel **Filtrar**, establezca la opción **Mostrar** en **Todas las aplicaciones**.

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>¿Por qué aparece una aplicación específica en mi lista de todas las aplicaciones?

Cuando se filtra por **todas las aplicaciones**, la **lista** **Todas las aplicaciones** muestra todos los objetos de entidad de servicio del inquilino. Los objetos de entidad de servicio pueden aparecer en esta lista de varias formas:

1. Cuando agrega cualquier aplicación desde la galería de aplicaciones, incluidas:

   1. **Aplicaciones de la galería de Azure AD**: una aplicación que se ha integrado previamente para el inicio de sesión único con Azure AD.

   2. **Aplicaciones de proxy de aplicación**: una aplicación que se ejecuta en su entorno local en la que quiere que se proporcione inicio de sesión seguro externamente.

   3. **Aplicaciones personalizadas**: una aplicación que su organización quiere desarrollar en la plataforma de desarrollo de aplicaciones de Azure AD, pero que puede que no exista aún.

   4. **Aplicaciones que no son de la Galería**: ¡traiga sus propias aplicaciones! Cualquier vínculo web que quiera, o cualquier aplicación que represente un campo de nombre de usuario y contraseña, admita los protocolos SAML u OpenID Connect, o admita SCIM, que quiera integrar para el inicio de sesión único con Azure AD.

2. Al registrarse o iniciar sesión en una aplicación de <sup>terceros</sup> integrada con Azure Active Directory. Un ejemplo de esto es [Smartsheet](https://app.smartsheet.com/b/home) o [DocuSign](https://www.docusign.net/member/MemberLogin.aspx).

3. Al registrarse o agregar una licencia a un usuario o un grupo para una aplicación original de Microsoft, como [Microsoft Office 365](https://products.office.com/).

4. Al agregar un nuevo registro de aplicación mediante la creación de una aplicación desarrollada de forma personalizada con el [Registro de aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-app-registration).

5. Al agregar un nuevo registro de aplicación mediante la creación de una aplicación desarrollada de forma personalizada con el [Portal de registro de aplicaciones V2.0](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-app-registration).

6. Al agregar una aplicación que está desarrollando con los [métodos de autenticación de ASP.NET](https://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions) o los [servicios conectados](https://blogs.msdn.com/b/visualstudio/archive/2014/11/19/connecting-to-cloud-services.aspx) de Visual Studio.

7. Al crear un objeto de entidad de servicio mediante el [módulo de PowerShell de Azure AD](/powershell/azure/install-adv2?view=azureadps-2.0).

8. Al [dar su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) como administrador para que utilice datos de su inquilino.

9. Cuando un [usuario da su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) para que utilice datos de su inquilino.

10. Al habilitar determinados servicios que almacenan datos en el inquilino. Un ejemplo de esto es Password Reset, que se modela como entidad de servicio para almacenar la directiva de restablecimiento de contraseña de forma segura.

Para obtener más detalles sobre cómo las aplicaciones se agregan a su directorio, lea [Cómo y por qué se agregan aplicaciones a Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-how-applications-are-added).

## <a name="i-want-to-remove-a-specific-users-or-groups-assignment-to-an-application"></a>Deseo quitar la asignación de un usuario o grupo específicos a una aplicación

Para quitar la asignación de un usuario o grupo a una aplicación, siga los pasos indicados en el artículo [Eliminación de asignaciones de usuario o grupo de una aplicación empresarial en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-remove-assignment-azure-portal).

## <a name="i-want-to-disable-all-access-to-an-application-for-every-user"></a>Quiero deshabilitar el acceso a una aplicación para todos los usuarios

Para deshabilitar todos los inicios de sesión de usuarios a una aplicación, siga los pasos indicados en el artículo [Deshabilitación de los inicios de sesión de usuario de una aplicación empresarial en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-disable-app-azure-portal).

## <a name="i-want-to-delete-an-application-entirely"></a>Quiero eliminar una aplicación por completo

Para **eliminar una aplicación**, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve que la aplicación que desea aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea eliminar.

7. Una vez que se cargue la aplicación, haga clic en el icono **Eliminar** del panel **Información general** de la aplicación situada en la parte superior.

## <a name="i-want-to-disable-all-future-user-consent-operations-to-any-application"></a>Quiero deshabilitar todas las operaciones de consentimiento de usuario futuras para todas las aplicaciones

Deshabilitar el consentimiento del usuario para todo el directorio impide que los usuarios finales den consentimiento a cualquier aplicación. Los administradores aún pueden dar su consentimiento en nombre de usuario. Para más información sobre el consentimiento de aplicación y los motivos por los que es posible que quiera o no quiera hacer esto, lea el artículo [Descripción del consentimiento de usuario y administrador](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview).

Para **deshabilitar todas las operaciones de consentimiento de usuario futuras en todo el directorio**, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global**.

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Configuración de usuario**.

6.  Deshabilite todas las operaciones de consentimiento de usuario futuras estableciendo la opción **Los usuarios pueden permitir que las aplicaciones accedan a sus datos** en **No**. Después, haga clic en el botón **Guardar**.

## <a name="next-steps"></a>Pasos siguientes
[Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
