---
title: Problemas de inicio de sesión en una aplicación desde el panel de acceso | Microsoft Docs
description: Cómo solucionar problemas de acceso a una aplicación desde el Panel de acceso de Microsoft Azure AD de myapps.microsoft.com
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
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 51e486e8eb2fef04c1b876dff3de644dda4aaf2e
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65781131"
---
# <a name="problems-signing-in-to-an-application-from-the-access-panel"></a>Problemas de inicio de sesión en una aplicación desde el panel de acceso

El panel de acceso es un portal basado en web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie las aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. 

Estas aplicaciones se configuran en nombre del usuario en el portal de Azure AD. La aplicación debe configurarse correctamente y asignarse al usuario o al grupo del que el usuario es miembro para poder ver la aplicación en el Panel de acceso.

El tipo de aplicaciones que un usuario puede ver se dividen en las siguientes categorías:

-   Aplicaciones de Office 365

-   Aplicaciones de Microsoft y de terceros configuradas con SSO basado en federación

-   Aplicaciones de SSO basado en contraseña

-   Aplicaciones con soluciones SSO existentes

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

## <a name="meeting-browser-requirements-for-the-access-panel"></a>Cumplimiento de los requisitos del explorador para el Panel de acceso

El Panel de acceso requiere un explorador compatible con JavaScript y que tenga habilitado CSS. Para usar el inicio de sesión único (SSO) basado en contraseña en el Panel de acceso, se debe instalar la extensión del Panel de acceso en el explorador del usuario. Esta extensión se descarga automáticamente cuando un usuario selecciona una aplicación que está configurada para SSO basado en contraseña.

Para el SSO basado en contraseña, los exploradores del usuario final pueden ser:

-   Internet Explorer 8, 9, 10, 11 (en Windows 7 o posterior)

-   Microsoft Edge en Windows 10 Anniversary Edition o versión posterior

-   Chrome (en Windows 7 o posterior y en Mac OS X o posterior)

-   Firefox 26.0 o posterior (en Windows XP SP2 o posterior y en Mac OS X 10.6 o posterior)

## <a name="how-to-install-the-access-panel-browser-extension"></a>Cómo instalar la extensión de explorador del Panel de acceso

Para instalar la extensión de explorador del Panel de acceso, siga estos pasos:

1.  Abra el [Panel de acceso](https://myapps.microsoft.com) en uno de los exploradores admitidos e inicie sesión como **usuario** en su instancia de Azure AD.

2.  Haga clic en una **aplicación de SSO con contraseña** en el Panel de acceso.

3.  En el mensaje que le pregunta si desea instalar el software, seleccione **Instalar ahora**.

4.  Se le dirigirá al vínculo de descarga en función del explorador. **Agregue** la extensión al explorador.

5.  Si el explorador lo solicita, seleccione **Habilitar** o **Permitir** la extensión.

6.  Una vez instalada, **reinicie** la sesión del explorador.

7.  Inicie sesión en el Panel de acceso y vea si puede **iniciar** las aplicaciones de SSO con contraseña

También puede descargar la extensión para Chrome y Microsoft Edge desde los siguientes vínculos directos:

-   [Extensión del Panel de acceso para Chrome](https://chrome.google.com/webstore/detail/access-panel-extension/ggjhpefgjjfobnfoldnjipclpcfbgbhl)

-   [Extensión del Panel de acceso para Microsoft Edge](https://www.microsoft.com/store/apps/9pc9sckkzk84)

## <a name="how-to-configure-federated-single-sign-on-for-an-azure-ad-gallery-application"></a>Configuración del inicio de sesión único federado para una aplicación de la galería de Azure AD

Todas las aplicaciones de la galería de Azure AD habilitadas con funcionalidad de Enterprise Single Sign-On tienen un tutorial paso a paso disponible. Para instrucciones detalladas paso a paso, acceda a la [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-saas-tutorial-list/).

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

-   Incorporación de una aplicación desde la galería de Azure AD

-   [Configuración de los valores de los metadatos de la aplicación en Azure AD (URL de inicio de sesión, identificador, URL de respuesta)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   [Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [Recuperación de los metadatos y el certificado de Azure AD](#download-the-azure-ad-metadata-or-certificate)

-   [Configuración de los valores de los metadatos de Azure AD en la aplicación (URL de inicio de sesión, emisor, URL de cierre de sesión y certificado)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   Asignación de usuarios a la aplicación

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

### <a name="configure-single-sign-on-for-an-application-from-the-azure-ad-gallery"></a>Configuración del inicio de sesión único para una aplicación de la galería de Azure AD

Para configurar el inicio de sesión único para una aplicación, siga los pasos siguientes:

1. <span id="_Hlk477187909" class="anchor"><span id="_Hlk477001983" class="anchor"></span></span>Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. En la lista desplegable **Modo**, seleccione **Inicio de sesión basado en SAML**.

9. Especifique los valores obligatorios en **Dominio y direcciones URL**. El proveedor de la aplicación le proporcionará estos valores.

   1. Para configurar la aplicación como SSO iniciado por el SP, la dirección URL de inicio de sesión es obligatoria. En algunas aplicaciones, el identificador también es obligatorio.

   2. Para configurar la aplicación como SSO iniciado por el IdP, la dirección URL de respuesta es obligatoria. En algunas aplicaciones, el identificador también es obligatorio.

10. **Opcional:** haga clic en **Mostrar configuración avanzada de URL** si desea ver los valores opcionales.

11. En **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**.

12. **Opcional:** haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

    Para agregar un atributo:

    1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

    2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

13. Haga clic en **Configurar &lt;nombre de la aplicación&gt;** para acceder a documentación sobre cómo configurar el inicio de sesión único en la aplicación. Además, tiene las direcciones URL de los metadatos y el certificado necesario para la instalación de SSO con la aplicación.

14. Para guardar la configuración, haga clic en **Guardar**.

15. Asigne usuarios a la aplicación.

### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación

Para seleccionar el identificador de usuario o agregar atributos de usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones**.

6. Seleccione la aplicación para la que ha configurado el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. En la sección **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**. La opción seleccionada debe coincidir con el valor esperado de la aplicación para autenticar al usuario.

   >[!NOTE]
   >AD Azure selecciona el formato del atributo NameID (Identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest).
   >
   >

9. Para agregar atributos de usuario, haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

   Para agregar un atributo:

   1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

   2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

### <a name="download-the-azure-ad-metadata-or-certificate"></a>Descarga del certificado o los metadatos de Azure AD

Para descargar el certificado o los metadatos de la aplicación de Azure AD, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación para la que ha configurado el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Vaya a la sección **Certificado de firma de SAML** y haga clic en el valor de la columna **Descargar**. Según lo que necesite la aplicación para configurar el inicio de sesión único, verá la opción para descargar el archivo XML de metadatos o el certificado.

   Azure AD no proporciona una dirección URL para obtener los metadatos. Solo se pueden recuperar como un archivo XML.

## <a name="how-to-configure-federated-single-sign-on-for-a-non-gallery-application"></a>Configuración del inicio de sesión único federado para una aplicación ajena a la galería

Para configurar una aplicación ajena a la galería, debe tener Azure AD Premium y que la aplicación admita SAML 2.0. Para más información acerca de las versiones de Azure AD, visite [Precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

-   Configuración de los valores de los metadatos de la aplicación en Azure AD (URL de inicio de sesión, identificador, URL de respuesta)

-   [Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [Recuperación de los metadatos y el certificado de Azure AD](#download-the-azure-ad-metadata-or-certificate)

-   Configuración de los valores de los metadatos de Azure AD en la aplicación (URL de inicio de sesión, emisor, URL de cierre de sesión y certificado)

### <a name="configure-the-applications-metadata-values-in-azure-ad-sign-on-url-identifier-reply-url"></a>Configuración de los valores de los metadatos de la aplicación en Azure AD (URL de inicio de sesión, identificador, URL de respuesta)

Para configurar el inicio de sesión único para una aplicación ajena a la galería de Azure AD, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en el botón **Agregar** situado en la esquina superior derecha del panel **Aplicaciones empresariales**.

6. En la sección **Agregar aplicación propia**, haga clic en **Aplicación situada fuera de la galería**.

7. Escriba el nombre de la aplicación en el cuadro de texto **Nombre**.

8. Haga clic en el botón **Agregar** para agregar la aplicación.

9. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

10. En la lista desplegable **Modo**, seleccione **Inicio de sesión basado en SAML**.

11. Especifique los valores obligatorios en **Dominio y direcciones URL**. El proveedor de la aplicación le proporcionará estos valores.

    1. Para configurar la aplicación como SSO iniciado por el IdP, escriba la dirección URL de respuesta y el identificador.

    2. **Opcional:** Para configurar la aplicación como SSO iniciado por el SP, la dirección URL de inicio de sesión es obligatoria.

12. En **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**.

13. **Opcional:** haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

    Para agregar un atributo:

    1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

    2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

14. Haga clic en **Configurar &lt;nombre de la aplicación&gt;** para acceder a documentación sobre cómo configurar el inicio de sesión único en la aplicación. Además, tiene las direcciones URL de Azure AD y el certificado necesarios para la aplicación.

### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación

Para seleccionar el identificador de usuario o agregar atributos de usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación para la que ha configurado el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. En la sección **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**. La opción seleccionada debe coincidir con el valor esperado de la aplicación para autenticar al usuario.

   >[!NOTE]
   >AD Azure selecciona el formato del atributo NameID (Identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest).
   >
   >

9. Para agregar atributos de usuario, haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

   Para agregar un atributo:

   1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

   2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

### <a name="download-the-azure-ad-metadata-or-certificate"></a>Descarga del certificado o los metadatos de Azure AD

Para descargar el certificado o los metadatos de la aplicación de Azure AD, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación para la que ha configurado el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Vaya a la sección **Certificado de firma de SAML** y haga clic en el valor de la columna **Descargar**. Según lo que necesite la aplicación para configurar el inicio de sesión único, verá la opción para descargar el archivo XML de metadatos o el certificado.

   Azure AD no proporciona una dirección URL para obtener los metadatos. Solo se pueden recuperar como un archivo XML.

## <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación de la galería de Azure AD

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

-   Incorporación de una aplicación desde la galería de Azure AD

-   Configuración de la aplicación para el inicio de sesión único con contraseña

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

Para configurar el inicio de sesión único para una aplicación, siga los pasos siguientes:

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

## <a name="how-to-configure-password-single-sign-on-for-a-non-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación ajena a la galería

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

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

Para configurar el inicio de sesión único para una aplicación, siga los pasos siguientes:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Seleccione el modo de **Inicio de sesión con contraseña**.

9. Escriba la **dirección URL de inicio de sesión**. Es la dirección URL en la que los usuarios escriben su nombre de usuario y contraseña para iniciar sesión. Asegúrese de que los campos de inicio de sesión estén visibles en la dirección URL.

10. Asigne usuarios a la aplicación.

11. Además, también puede proporcionar credenciales en nombre del usuario; para ello, seleccione las filas de los usuarios, haga clic en **Actualizar credenciales** y escriba el nombre de usuario y la contraseña en nombre de los usuarios. En caso contrario, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.

## <a name="how-to-assign-a-user-to-an-application-directly"></a>Asignación de un usuario a una aplicación directamente

Para asignar uno o varios usuarios a una aplicación directamente, siga los pasos siguientes:

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

Tras un breve período, los usuarios que seleccionó podrán iniciar estas aplicaciones en el panel de acceso.

## <a name="if-these-troubleshooting-steps-do-not-the-resolve-the-issue"></a>Si después de seguir estos pasos, el problema no se ha resuelto,

abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación

-   UPN (dirección de correo electrónico del usuario)

-   TenantID

-   Tipo de explorador

-   Zona horaria y hora/período de tiempo durante el que se produce el error

-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[Proporcionar un inicio de sesión único a las aplicaciones con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md)

