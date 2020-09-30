---
title: Solución de problemas al iniciar sesión en una aplicación desde Aplicaciones de Azure AD
description: Solución de problemas al iniciar sesión en una aplicación desde Aplicaciones de Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 472ab82e6941be5c1485773b1115d91eda701666
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231187"
---
# <a name="troubleshoot-problems-signing-in-to-an-application-from-azure-ad-my-apps"></a>Solución de problemas al iniciar sesión en una aplicación desde Aplicaciones de Azure AD

Aplicaciones es un portal basado en web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie las aplicaciones basadas en la nube a las que el administrador de Azure AD le haya concedido acceso. 

Para más información sobre cómo usar Azure AD como proveedor de identidades de una aplicación, consulte [¿Qué es la administración de aplicaciones?](what-is-application-management.md). Para ponerse al día rápidamente, consulte la [serie de inicios rápidos sobre la administración de aplicaciones](view-applications-portal.md).

Estas aplicaciones se configuran en nombre del usuario en el portal de Azure AD. La aplicación debe configurarse correctamente y asignarse al usuario o al grupo del que el usuario sea miembro para poder ver la aplicación en Aplicaciones. 

El tipo de aplicaciones que un usuario puede ver se dividen en las siguientes categorías:
-   Aplicaciones Microsoft 365 y Office 365
-   Aplicaciones de Microsoft y de terceros configuradas con SSO basado en federación
-   Aplicaciones de SSO basado en contraseña
-   Aplicaciones con soluciones SSO existentes

Aquí se muestran algunas cuestiones que comprobar si una aplicación aparece o no aparece.
- Asegúrese de agregar la aplicación a Azure AD y de que el usuario esté asignado. Para más información, consulte la [serie de inicios rápidos sobre la administración de aplicaciones](view-applications-portal.md).
- Si se ha agregado una aplicación recientemente, haga que el usuario cierre la sesión y vuelva a intentarlo. 
- Si la aplicación requiere una licencia, como Office, asegúrese de que el usuario tenga asignada la adecuada.
- El tiempo necesario para realizar cambios en las licencias puede variar en función del tamaño y la complejidad del grupo.

## <a name="general-issues-to-check-first"></a>Problemas generales para comprobar primero

-   Asegúrese de que el explorador web cumpla los requisitos; consulte [Exploradores compatibles](../user-help/my-apps-portal-end-user-access.md).
-   Que se ha agregado la dirección URL de la aplicación a los **sitios de confianza** del explorador del usuario.
-   Que la aplicación está **configurada** correctamente.
-   Que la cuenta del usuario está **habilitada** para los inicios de sesión.
-   Que la cuenta del usuario **no está bloqueada**.
-   Que la **contraseña del usuario no ha expirado o se ha olvidado**.
-   Que **Multi-Factor Authentication** no bloquea el acceso del usuario.
-   Que una **directiva de acceso condicional** o una directiva de **protección de identidad** no bloquea el acceso del usuario.
-   Que la **información de contacto de autenticación** del usuario está actualizada para permitir la aplicación de directivas de Multi-Factor Authentication o de acceso condicional.
-   Que se intenta borrar también las cookies del explorador y volver a iniciar sesión.

## <a name="problems-with-the-users-account"></a>Problemas con la cuenta del usuario
El acceso a Aplicaciones se puede bloquear debido a un problema con la cuenta del usuario. A continuación, se muestran algunas maneras de solucionar problemas con los usuarios y la configuración de sus cuentas:
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
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Compruebe las propiedades del objeto de usuario para asegurarse de que se muestran como esperaba y no falta ningún dato.

### <a name="check-a-users-account-status"></a>Comprobar el estado de la cuenta de un usuario
Para comprobar el estado de la cuenta de un usuario, siga estos pasos:
1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Seleccione **Perfil**.
8.  En **Configuración**, asegúrese de que **Bloquear inicio de sesión** esté establecido en **No**.

### <a name="reset-a-users-password"></a>Restablecer la contraseña de un usuario
Para restablecer la contraseña de un usuario, siga estos pasos:
1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Seleccione el botón **Restablecer contraseña** situada en la parte superior del panel de usuario.
8.  Seleccione el botón **Restablecer contraseña** en el panel **Restablecer contraseña** que aparece.
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
4. Seleccione **Usuarios y grupos** en el menú de navegación.
5. Seleccione **Todos los usuarios**.
6. Seleccione el botón **Autenticación multifactor** en la parte superior del panel.
7. Después de que se ha cargado el **Portal de administración de Multi-factor Authentication**, asegúrese de que se encuentra en la pestaña **Usuarios**.
8. Para encontrar el usuario, filtre u ordene la lista de usuarios, o realice búsquedas en ella.
9. Seleccione el usuario de la lista de usuarios y **habilite**, **deshabilite** o **exija** autenticación multifactor, según desee.
   >[!NOTE]
   >Si un usuario se encuentra en un estado **Exigido**, puede establecerlo en **Deshabilitado** temporalmente para permitirle volver a su cuenta. Una vez de vuelta, puede cambiar de nuevo su estado a **Habilitado** para requerir que vuelva a registrar su información de contacto durante su siguiente inicio de sesión. Como alternativa, puede seguir los pasos descritos en [Comprobar la información de contacto de autenticación de un usuario](#check-a-users-authentication-contact-info) para comprobar o establecer estos datos por ellos.

### <a name="check-a-users-authentication-contact-info"></a>Comprobar la información de contacto de autenticación de un usuario
Para comprobar la información de contacto de autenticación de un usuario para Multi-Factor Authentication, acceso condicional, protección de identidad y restablecimiento de contraseña, siga estos pasos:
1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Seleccione **Perfil**.
8.  Desplácese hacia abajo hasta **Información de contacto para la autenticación**.
9.  **Revise** los datos registrados para el usuario y actualícelos si es necesario.

### <a name="check-a-users-group-memberships"></a>Comprobación de la pertenencia a grupos de un usuario
Para comprobar la pertenencia a grupos de un usuario, siga estos pasos:
1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Seleccione **Grupos** para ver de cuáles es miembro el usuario.

### <a name="check-a-users-assigned-licenses"></a>Comprobar las licencias asignadas de un usuario
Para comprobar las licencias asignadas de un usuario, siga estos pasos:
1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Seleccione **Licencias** para ver cuáles tiene asignadas actualmente el usuario.

### <a name="assign-a-user-a-license"></a>Asignar una licencia a un usuario 
Para asignar una licencia a un usuario, siga estos pasos:
1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4.  Seleccione **Usuarios y grupos** en el menú de navegación.
5.  Seleccione **Todos los usuarios**.
6.  **Busque** el usuario que le interesa y seleccione la **fila** para seleccionarlo.
7.  Seleccione **Licencias** para ver cuáles tiene asignadas actualmente el usuario.
8.  Seleccione el botón **Asignar**.
9.  Seleccione **uno o más productos** en la lista de productos disponibles.
10. **Opcional**: seleccione el elemento **Opciones de asignación** para asignar productos concretos. Seleccione **Aceptar**.
11. Seleccione el botón **Asignar** para asignar estas licencias a este usuario.

## <a name="troubleshooting-deep-links"></a>Solución de problemas de los vínculos profundos
Las direcciones URL de acceso de usuario o vínculos profundos son vínculos que los usuarios pueden utilizar para acceder a sus aplicaciones de SSO con contraseña directamente desde las barras de dirección URL de los exploradores. Al ir a este vínculo, los usuarios pueden iniciar sesión automáticamente en la aplicación sin tener que ir primero a Aplicaciones. Este es el mismo vínculo que los usuarios utilizan para acceder a estas aplicaciones desde el iniciador de la aplicación de Office 365.

### <a name="checking-the-deep-link"></a>Comprobación del vínculo profundo

Para comprobar si tiene el vínculo profundo correcto, siga estos pasos:
1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**
2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4. Seleccione **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.
5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**
6. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**
7. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
8. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
9. Seleccione **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.
10. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
    * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**
11. Seleccione la aplicación para la cual desea comprobar el vínculo profundo.
12. Busque la etiqueta **URL de acceso de usuario**. El vínculo profundo debe coincidir con esta dirección URL.

## <a name="contact-support"></a>Ponerse en contacto con soporte técnico
Abra una incidencia de soporte técnico con la información siguiente, si está disponible:
-   Id. de error de correlación
-   UPN (dirección de correo electrónico del usuario)
-   TenantID
-   Tipo de explorador
-   Zona horaria y hora/período de tiempo durante el que se produce el error
-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
- [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
