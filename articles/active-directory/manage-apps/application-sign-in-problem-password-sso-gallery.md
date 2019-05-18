---
title: Problemas al iniciar sesión en una aplicación de la galería de Azure AD configurada para inicio de sesión único | Microsoft Docs
description: Cómo solucionar problemas relacionados con aplicaciones de la galería de Azure AD configuradas para inicio de sesión único con contraseña
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0559213706c499878e0f14a0beeee22dcdbaf59a
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825145"
---
# <a name="problems-signing-in-to-an-azure-ad-gallery-application-configured-for-password-single-sign-on"></a>Problemas al iniciar sesión en una aplicación de la galería de Azure AD configurada para inicio de sesión único

El Panel de acceso es un portal web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. Un usuario que posee ediciones de Azure AD también puede usar las funcionalidades de administración de grupos de autoservicio y aplicaciones a través del Panel de acceso. El Panel de acceso es independiente de Azure Portal y no requiere que los usuarios tengan una suscripción de Azure.

Para usar el inicio de sesión único (SSO) basado en contraseña en el Panel de acceso, se debe instalar la extensión del Panel de acceso en el explorador del usuario. Esta extensión se descarga automáticamente cuando un usuario selecciona una aplicación que está configurada para SSO basado en contraseña.

## <a name="meeting-browser-requirements-for-the-access-panel"></a>Cumplimiento de los requisitos del explorador para el Panel de acceso

El Panel de acceso requiere un explorador compatible con JavaScript y que tenga habilitado CSS. Para usar el inicio de sesión único (SSO) basado en contraseña en el Panel de acceso, se debe instalar la extensión del Panel de acceso en el explorador del usuario. Esta extensión se descarga automáticamente cuando un usuario selecciona una aplicación que está configurada para SSO basado en contraseña.

Para el SSO basado en contraseña, los exploradores del usuario final pueden ser:

-   Internet Explorer 8, 9, 10, 11: en Windows 7 o versiones superiores

-   Chrome: en Windows 7 o versiones superiores y en MacOS X o versiones superiores

-   Firefox 26.0 o versiones superiores: en Windows XP SP2 o versiones superiores y en Mac OS X 10.6 o versiones superiores

>[!NOTE]
>La extensión de SSO basado en contraseña está disponible para Microsoft Edge en Windows 10 cuando se admiten extensiones de explorador para Microsoft Edge.
>
>

## <a name="how-to-install-the-access-panel-browser-extension"></a>Cómo instalar la extensión de explorador del Panel de acceso

Para instalar la extensión de explorador del panel de acceso, siga estos pasos:

1.  Abra el [Panel de acceso](https://myapps.microsoft.com) en uno de los exploradores admitidos e inicie sesión como **usuario** en su instancia de Azure AD.

2.  Haga clic en una **aplicación de SSO con contraseña** en el Panel de acceso.

3.  En el mensaje que le pregunta si desea instalar el software, seleccione **Instalar ahora**.

4.  Se le dirigirá al vínculo de descarga en función del explorador. **Agregue** la extensión al explorador.

5.  Si el explorador lo solicita, seleccione **Habilitar** o **Permitir** la extensión.

6.  Una vez instalada, **reinicie** la sesión del explorador.

7.  Inicie sesión en el Panel de acceso y vea si puede **iniciar** las aplicaciones de SSO con contraseña

También puede descargar la extensión para Chrome y Firefox desde los siguientes vínculos directos:

-   [Extensión del Panel de acceso para Chrome](https://chrome.google.com/webstore/detail/access-panel-extension/ggjhpefgjjfobnfoldnjipclpcfbgbhl)

-   [Extensión del Panel de acceso para Firefox](https://addons.mozilla.org/firefox/addon/access-panel-extension/)

## <a name="setting-up-a-group-policy-for-internet-explorer"></a>Configuración de una directiva de grupo para Internet Explorer

Puede configurar una directiva de grupo que le permita instalar de forma remota la extensión del Panel de acceso para Internet Explorer en las máquinas de los usuarios.

Los requisitos previos son:

-   Configuró [Active Directory Domain Services](https://msdn.microsoft.com/library/aa362244%28v=vs.85%29.aspx)y unió los equipos de los usuarios a su dominio.

-   Debe tener el permiso "Editar configuración" para editar el objeto de directiva de grupo (GPO). De forma predeterminada, los miembros de los siguientes grupos de seguridad tienen este permiso: administradores de dominio, administradores de empresa y propietarios del creador de directivas de grupo. [Más información](https://technet.microsoft.com/library/cc781991%28v=ws.10%29.aspx).

Siga el tutorial [Implementación de la extensión de panel de acceso para Internet Explorer mediante la directiva de grupo](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ie-group-policy) para obtener instrucciones paso a paso sobre cómo configurar la directiva de grupo e implementarla en los usuarios.

## <a name="troubleshoot-the-access-panel-in-internet-explorer"></a>Solución de problemas del Panel de acceso en Internet Explorer

Siga la guía de [solución de problemas de la extensión del Panel de acceso para Internet Explorer](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ie-troubleshooting) para acceder a la herramienta de diagnóstico, junto con instrucciones paso a paso sobre cómo configurar la extensión para IE.

## <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación de la galería de Azure AD

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

-   Incorporación de una aplicación desde la galería de Azure AD

-   [Configuración de la aplicación para el inicio de sesión único con contraseña](#configure-the-application-for-password-single-sign-on)

-   [Asignar usuarios a la aplicación](#assign-users-to-the-application)

### <a name="add-an-application-from-the-azure-ad-gallery"></a>Incorporación de una aplicación desde la galería de Azure AD

Para agregar una aplicación desde la galería de Azure AD, siga estos pasos:

1.  Abra [Azure Portal](https://portal.azure.com) e inicie sesión como **administrador global** o **coadministrador**.

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en el botón **Agregar** situado en la esquina superior derecha del panel **Aplicaciones empresariales**.

6.  En el cuadro de texto **Escriba un nombre** de la sección **Agregar desde la galería**, escriba el nombre de la aplicación.

7.  Seleccione la aplicación que desea configurar para el inicio de sesión único.

8.  Antes de agregar la aplicación, puede cambiar su nombre en el cuadro de texto **Nombre**.

9.  Haga clic en el botón **Agregar** para agregar la aplicación.

Tras un breve período, podrá ver el panel de configuración de la aplicación.

### <a name="configure-the-application-for-password-single-sign-on"></a>Configuración de la aplicación para el inicio de sesión único con contraseña

Para configurar el inicio de sesión único para una aplicación, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccionar la aplicación que desea configurar para el inicio de sesión único

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Seleccione el modo de **Inicio de sesión con contraseña**.

9. Asigne usuarios a la aplicación.

10. Además, también puede proporcionar credenciales en nombre del usuario; para ello, seleccione las filas de los usuarios, haga clic en **Actualizar credenciales** y escriba el nombre de usuario y la contraseña en nombre de los usuarios. En caso contrario, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.

### <a name="assign-users-to-the-application"></a>Asignación de usuarios a la aplicación

Para asignar uno o varios usuarios a una aplicación directamente, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea asignar a un usuario de la lista.

7. Cuando se cargue la aplicación, haga clic en **Usuarios y grupos** desde el menú de navegación izquierdo de la aplicación.

8. Haga clic en el botón **Agregar** en la parte superior de la lista **Usuarios y grupos** para abrir el panel **Agregar asignación**.

9. Haga clic en el selector **Usuarios y grupos** del panel **Agregar asignación**.

10. Escriba el **nombre completo** o la **dirección de correo electrónico** del usuario al que quiere asignar en el cuadro de búsqueda **Buscar por nombre o dirección de correo**.

11. Mantenga el puntero sobre el **usuario** en la lista para que aparezca una **casilla**. Haga clic en la casilla situada junto a la foto o el logotipo del perfil del usuario para agregar ese usuario a la lista de **seleccionados**.

12. **Opcional:** si desea **agregar más de un usuario**, escriba otro **nombre completo** o **dirección de correo electrónico** en el cuadro de búsqueda **Buscar por nombre o dirección de correo** y haga clic en la casilla para agregar ese usuario a la lista de **seleccionados**.

13. Cuando haya terminado de seleccionar usuarios, haga clic en el botón **Seleccionar** para agregarlos a la lista de usuarios y grupos que se asignarán a la aplicación.

14. **Opcional:** haga clic en el selector **Seleccionar rol** del panel **Agregar asignación** para seleccionar un rol que se asignará a los usuarios que ha seleccionado.

15. Haga clic en el botón **Asignar** para asignar la aplicación a los usuarios seleccionados.

Tras un breve período, los usuarios que seleccionó podrán iniciar estas aplicaciones en el Panel de acceso.

## <a name="if-these-troubleshoot-steps-dont-resolve-the-issue"></a>Si después de seguir estos pasos el problema no se ha resuelto, 
abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación

-   UPN (dirección de correo electrónico del usuario)

-   TenantID

-   Tipo de explorador

-   Zona horaria y hora/período de tiempo durante el que se produce el error

-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[Proporcionar un inicio de sesión único a las aplicaciones con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md)
