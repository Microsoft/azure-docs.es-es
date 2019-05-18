---
title: Problema al iniciar sesión en el sitio web del Panel de acceso | Microsoft Docs
description: Instrucciones para solucionar problemas que pueden surgir al intentar iniciar sesión para usar el Panel de acceso.
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
ms.reviwer: japere,asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: a7c6a9c3f26c8939176197a2ecf2fcd6026e9928
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784304"
---
# <a name="problem-signing-in-to-the-access-panel-website"></a>Problema al iniciar sesión en el sitio web del Panel de acceso

El panel de acceso es un portal web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. Un usuario que posee ediciones de Azure AD también puede usar las funcionalidades de administración de grupos de autoservicio y aplicaciones a través del Panel de acceso. El Panel de acceso es independiente de Azure Portal y no requiere que los usuarios tengan una suscripción de Azure.

Los usuarios pueden iniciar sesión en el Panel de acceso si tienen una cuenta profesional o educativa en Azure AD.

-   Los usuarios se pueden autenticar directamente mediante Azure AD.

-   Los usuarios se pueden autenticar mediante los Servicios de federación de Active Directory (AD FS).

-   Los usuarios se pueden autenticar mediante Windows Server Active Directory.

Si un usuario tiene una suscripción de Azure u Office 365 y ha estado usando el portal de Azure o una aplicación de Office 365, dicho usuario podrá usar el Panel de acceso sin problemas sin necesidad de volver a iniciar sesión de nuevo. A los usuarios que no se han autenticado se les solicita que inicien sesión utilizando el nombre de usuario y la contraseña de su cuenta en Azure AD. Si la organización configuró la federación, bastará con escribir el nombre de usuario.

## <a name="general-issues-to-check-first"></a>Problemas generales para comprobar primero 

-   Asegurarse de que el usuario inicia sesión en la **dirección URL correcta**: <https://myapps.microsoft.com>

-   Asegurarse de que el explorador del usuario ha agregado la dirección URL a sus **sitios de confianza**

-   Que la cuenta del usuario está **habilitada** para los inicios de sesión.

-   Que la cuenta del usuario **no está bloqueada**.

-   Que la **contraseña del usuario no ha expirado o se ha olvidado**.

-   Que **Multi-Factor Authentication** no bloquea el acceso del usuario.

-   Que una **directiva de acceso condicional** o una directiva de **protección de identidad** no bloquea el acceso del usuario.

-   Que la **información de contacto de autenticación** del usuario está actualizada para permitir la aplicación de directivas de Multi-Factor Authentication o de acceso condicional.

-   Que se intenta borrar también las cookies del explorador y volver a iniciar sesión.

## <a name="meeting-browser-requirements-for-the-access-panel"></a>Cumplimiento de los requisitos del explorador para el Panel de acceso

El Panel de acceso requiere un explorador compatible con JavaScript y que tenga habilitado CSS. Para usar el inicio de sesión único (SSO) basado en contraseña en el Panel de acceso, se debe instalar la extensión del Panel de acceso en el explorador del usuario. Esta extensión se descarga automáticamente cuando un usuario selecciona una aplicación que está configurada para SSO basado en contraseña.

Para el SSO basado en contraseña, los exploradores del usuario final pueden ser:

-   Internet Explorer 8, 9, 10, 11 (en Windows 7 o posterior)

-   Microsoft Edge en Windows 10 Anniversary Edition o versión posterior 

-   Chrome (en Windows 7 o posterior y en Mac OS X o posterior)

-   Firefox 26.0 o posterior (en Windows XP SP2 o posterior y en Mac OS X 10.6 o posterior)


## <a name="problems-with-the-users-account"></a>Problemas con la cuenta del usuario

El acceso al Panel de acceso se puede bloquear debido a un problema con la cuenta del usuario. A continuación, se muestran algunas maneras de solucionar problemas con los usuarios y la configuración de sus cuentas:

-   [Comprobar si existe una cuenta de usuario en Azure Active Directory](#check-if-a-user-account-exists-in-azure-active-directory)

-   [Comprobar el estado de la cuenta de un usuario](#check-a-users-account-status)

-   [Restablecer la contraseña de un usuario](#reset-a-users-password)

-   [Habilitar el autoservicio de restablecimiento de contraseña](#enable-self-service-password-reset)

-   [Comprobar el estado de la autenticación multifactor de un usuario](#check-a-users-multi-factor-authentication-status)

-   [Comprobar la información de contacto de autenticación de un usuario](#check-a-users-authentication-contact-info)

-   [Comprobación de la pertenencia a grupos de un usuario](#check-a-users-group-memberships)

-   [Comprobar las licencias asignadas de un usuario](#check-a-users-assigned-licenses)

-   [Asignar una licencia a un usuario](#assign-a-user-a-license)

### <a name="check-if-a-user-account-exists-in-azure-active-directory"></a>Comprobar si existe una cuenta de usuario en Azure Active Directory

Para comprobar si una cuenta de usuario existe, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Compruebe las propiedades del objeto de usuario para asegurarse de que se muestran como esperaba y no falta ningún dato.

### <a name="check-a-users-account-status"></a>Comprobar el estado de la cuenta de un usuario

Para comprobar el estado de la cuenta de un usuario, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en **Perfil**.

8.  En **Configuración**, asegúrese de que **Bloquear inicio de sesión** esté establecido en **No**.

### <a name="reset-a-users-password"></a>Restablecer la contraseña de un usuario

Para restablecer la contraseña de un usuario, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en el botón **Restablecer contraseña** situado en la parte superior del panel de usuario.

8.  Haga clic en el botón **Restablecer contraseña** en el panel **Restablecer contraseña** que aparece.

9.  Copie la **contraseña temporal** o **escriba una contraseña nueva** para el usuario.

10. Comunique esta nueva contraseña al usuario, que deberá cambiarla durante el siguiente inicio de sesión en Azure Active Directory.

### <a name="enable-self-service-password-reset"></a>Habilitar el autoservicio de restablecimiento de contraseña

Para habilitar el autoservicio de restablecimiento de contraseña, siga estos pasos de implementación:

-   [Permitir que los usuarios restablezcan sus contraseñas de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started)

-   [Permitir que los usuarios restablezcan o cambien sus contraseñas de AD locales](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started)

### <a name="check-a-users-multi-factor-authentication-status"></a>Comprobar el estado de la autenticación multifactor de un usuario

Para comprobar el estado de la autenticación multifactor de un usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. Haga clic en el botón **Multi-Factor Authentication** en la parte superior del panel.

7. Después de que se ha cargado el **Portal de administración de Multi-factor Authentication**, asegúrese de que se encuentra en la pestaña **Usuarios**.

8. Para encontrar el usuario, filtre u ordene la lista de usuarios, o realice búsquedas en ella.

9. Seleccione el usuario de la lista de usuarios y **habilite**, **deshabilite** o **exija** autenticación multifactor, según desee.

   >[!NOTE]
   >Si un usuario se encuentra en un estado **Exigido**, puede establecerlo en **Deshabilitado** temporalmente para permitirle volver a su cuenta. Una vez de vuelta, puede cambiar de nuevo su estado a **Habilitado** para requerir que vuelva a registrar su información de contacto durante su siguiente inicio de sesión. Como alternativa, puede seguir los pasos descritos en [Comprobar la información de contacto de autenticación de un usuario](#check-a-users-authentication-contact-info) para comprobar o establecer estos datos por ellos.
   >
   >

### <a name="check-a-users-authentication-contact-info"></a>Comprobar la información de contacto de autenticación de un usuario

Para comprobar la información de contacto de autenticación de un usuario para Multi-Factor Authentication, acceso condicional, protección de identidad y restablecimiento de contraseña, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en **Perfil**.

8.  Desplácese hacia abajo hasta **Información de contacto para la autenticación**.

9.  **Revise** los datos registrados para el usuario y actualícelos si es necesario.

### <a name="check-a-users-group-memberships"></a>Comprobación de la pertenencia a grupos de un usuario

Para comprobar la pertenencia a grupos de un usuario, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en **Grupos** para ver de qué grupos es miembro el usuario.

### <a name="check-a-users-assigned-licenses"></a>Comprobar las licencias asignadas de un usuario

Para comprobar las licencias asignadas de un usuario, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en **Licencias** para ver qué licencias tiene asignadas actualmente el usuario.

### <a name="assign-a-user-a-license"></a>Asignar una licencia a un usuario 

Para asignar una licencia a un usuario, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los usuarios**.

6.  **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en **Licencias** para ver qué licencias tiene asignadas actualmente el usuario.

8.  Haga clic en el botón **Asignar**.

9.  Seleccione **uno o más productos** en la lista de productos disponibles.

10. **Opcional**: Haga clic en el elemento **Opciones de asignación** para asignar productos de forma granular. Cuando haya finalizado este procedimiento, haga clic en **Aceptar**.

11. Haga clic en el botón **Asignar** para asignar estas licencias a este usuario.

## <a name="if-these-troubleshooting-steps-do-not-resolve-the-issue"></a>Si después de seguir estos pasos el problema no se ha resuelto,

abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación

-   UPN (dirección de correo electrónico del usuario)

-   Id. de inquilino

-   Tipo de explorador

-   Zona horaria y hora/período de tiempo durante el que se produce el error

-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[Proporcionar un inicio de sesión único a las aplicaciones con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md)
