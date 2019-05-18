---
title: Problemas de inicio de sesión con un vínculo profundo en una aplicación | Microsoft Docs
description: Cómo solucionar problemas de acceso a una aplicación desde una dirección URL de vínculo profundo con Azure AD
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
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 44825f32a13db0a221252c042dc9f23ec43a9c8f
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825426"
---
# <a name="problems-signing-in-to-an-application-using-a-deeplink"></a>Problemas de inicio de sesión con un vínculo profundo en una aplicación

El Panel de acceso es un portal basado en web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie las aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. 

Estas aplicaciones se configuran en nombre del usuario en el portal de Azure AD. La aplicación debe configurarse correctamente y asignarse al usuario o al grupo del que el usuario es miembro para poder ver la aplicación en el Panel de acceso.

Las direcciones URL de acceso de usuario o vínculos profundos son vínculos que los usuarios pueden utilizar para acceder a sus aplicaciones de SSO con contraseña directamente desde las barras de dirección URL de los exploradores. Al ir a este vínculo, los usuarios pueden iniciar sesión automáticamente en la aplicación sin tener que ir primero al Panel de acceso. Se trata del mismo vínculo que los usuarios utilizan para acceder a estas aplicaciones desde el iniciador de la aplicación de Office 365.

## <a name="general-issues-to-check-first"></a>Problemas generales para comprobar primero

-   Que se usa un **explorador** que cumpla los requisitos mínimos del Panel de acceso.

-   Que se ha agregado la dirección URL de la aplicación a los **sitios de confianza** del explorador del usuario.

-   Que la aplicación está **configurada** correctamente.

-   Que la cuenta del usuario está **habilitada** para los inicios de sesión.

-   Que la cuenta del usuario **no está bloqueada**.

-   Que la **contraseña del usuario no ha expirado o se ha olvidado**.

-   Que **Multi-Factor Authentication** no bloquea el acceso del usuario.

-   Que una **directiva de acceso condicional** o una directiva de **protección de identidad** no bloquea el acceso del usuario.

-   Que la **información de contacto de autenticación** del usuario está actualizada para permitir la aplicación de directivas de Multi-Factor Authentication o de acceso condicional.

-   Que se intenta borrar también las cookies del explorador y volver a iniciar sesión.

## <a name="checking-the-deeplink"></a>Comprobación del vínculo profundo

Para comprobar si tiene el vínculo profundo correcto, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

7. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

8. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

9. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

10. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

    * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

11. Seleccione la aplicación para la cual desea comprobar el vínculo profundo.

12. Busque la etiqueta **URL de acceso de usuario**. El vínculo profundo debe coincidir con esta dirección URL.

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

-   [Extensión del panel de acceso para Firefox](https://addons.mozilla.org/firefox/addon/access-panel-extension/)

## <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación de la galería de Azure AD

Para configurar una aplicación desde la galería de Azure AD, debe realizar los siguientes pasos:

-   [Incorporación de una aplicación desde la galería de Azure AD](#add-an-application-from-the-azure-ad-gallery)

-   [Configuración de la aplicación para el inicio de sesión único con contraseña](#configure-the-application-for-password-single-sign-on)

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

9.  Para agregar la aplicación, haga clic en el botón **Agregar**.

Tras un breve período, podrá ver el panel de configuración de la aplicación.

### <a name="configure-the-application-for-password-single-sign-on"></a>Configuración de la aplicación para el inicio de sesión único con contraseña

Para configurar el inicio de sesión único para una aplicación, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Seleccione el modo de **Inicio de sesión con contraseña**.

9. [Asigne usuarios a la aplicación](#how-to-assign-a-user-to-an-application-directly).

10. Además, también puede proporcionar credenciales en nombre del usuario; para ello, seleccione las filas de los usuarios, haga clic en **Actualizar credenciales** y escriba el nombre de usuario y la contraseña en nombre de los usuarios. En caso contrario, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.

## <a name="how-to-configure-password-single-sign-on-for-a-non-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación ajena a la galería

Para configurar una aplicación desde la galería de Azure AD, debe realizar los siguientes pasos:

-   [Incorporación de una aplicación ajena a la galería](#add-a-non-gallery-application)

-   [Configuración de la aplicación para el inicio de sesión único con contraseña](#configure-the-application-for-password-single-sign-on)

### <a name="add-a-non-gallery-application"></a>Incorporación de una aplicación ajena a la galería

Para agregar una aplicación desde la galería de Azure AD, siga estos pasos:

1.  Abra [Azure Portal](https://portal.azure.com) e inicie sesión como **administrador global** o **coadministrador**.

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en el botón **Agregar** situado en la esquina superior derecha del panel **Aplicaciones empresariales**.

6.  Haga clic en **Aplicación situada fuera de la galería**.

7.  Escriba el nombre de la aplicación en el cuadro de texto **Nombre**. Seleccione **Agregar**.

Tras un breve período, podrá ver el panel de configuración de la aplicación.

### <a name="configure-the-application-for-password-single-sign-on"></a>Configuración de la aplicación para el inicio de sesión único con contraseña

Para configurar el inicio de sesión único para una aplicación, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

    1.  Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6.  Seleccione la aplicación que desea configurar para el inicio de sesión único.

7.  Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8.  Seleccione el modo de **Inicio de sesión con contraseña**.

9.  Escriba la **dirección URL de inicio de sesión** donde los usuarios escribirán su nombre de usuario y contraseña para iniciar sesión. Asegúrese de que los campos de inicio de sesión estén visibles en la dirección URL.

10. Asigne usuarios a la aplicación.

11. Además, también puede proporcionar credenciales en nombre del usuario; para ello, seleccione las filas de los usuarios, haga clic en **Actualizar credenciales** y escriba el nombre de usuario y la contraseña en nombre de los usuarios. En caso contrario, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.

## <a name="how-to-assign-a-user-to-an-application-directly"></a>Asignación de un usuario a una aplicación directamente

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

11. Mantenga el puntero sobre el **usuario** en la lista para que aparezca una **casilla**. Para agregar su usuario a la lista de **seleccionados**, haga clic en la casilla situada junto a la foto o el logotipo del perfil del usuario.

12. **Opcional:** si desea **agregar más de un usuario**, escriba otro **nombre completo** o **dirección de correo electrónico** en el cuadro de búsqueda **Buscar por nombre o dirección de correo** y haga clic en la casilla para agregar ese usuario a la lista de **seleccionados**.

13. Cuando haya terminado de seleccionar usuarios, haga clic en el botón **Seleccionar** para agregarlos a la lista de usuarios y grupos que se asignarán a la aplicación.

14. **Opcional:** haga clic en el selector **Seleccionar rol** del panel **Agregar asignación** para seleccionar un rol que se asignará a los usuarios que ha seleccionado.

15. Haga clic en el botón **Asignar** para asignar la aplicación a los usuarios seleccionados.

Tras un breve período, los usuarios que seleccionó podrán iniciar estas aplicaciones en el panel de acceso.

## <a name="if-these-troubleshooting-steps-do-not-the-resolve-the-issue"></a>Si después de seguir estos pasos el problema no se ha resuelto, 

abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación

-   UPN (dirección de correo electrónico del usuario)

-   TenantID

-   Tipo de explorador

-   Zona horaria y hora/período de tiempo durante el que se produce el error

-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[Proporcionar un inicio de sesión único a las aplicaciones con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md)
