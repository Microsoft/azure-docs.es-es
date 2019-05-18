---
title: Una aplicación asignada no aparece en el panel de acceso | Microsoft Docs
description: Solucionar por qué una aplicación no aparece en el panel de acceso
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
ms.date: 09/09/2018
ms.author: mimart
ms.reviwer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9d16efbd8ea2fa37ac9572e9dc1ba3ef67deaeaf
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784395"
---
# <a name="an-assigned-application-is-not-appearing-on-the-access-panel"></a>Una aplicación asignada no aparece en el panel de acceso

El panel de acceso es un portal basado en web que permite que un usuario con una cuenta profesional o educativa de Azure Active Directory (Azure AD) vea e inicie las aplicaciones basadas en la nube a las que el administrador de Azure AD le ha concedido acceso. Estas aplicaciones se configuran en nombre del usuario en el portal de Azure AD. La aplicación debe configurarse correctamente y asignarse al usuario o al grupo del que el usuario es miembro para poder ver la aplicación en el Panel de acceso.

El tipo de aplicaciones que un usuario puede ver se dividen en las siguientes categorías:

-   Aplicaciones de Office 365

-   Aplicaciones de Microsoft y de terceros configuradas con SSO basado en federación

-   Aplicaciones de SSO basado en contraseña

-   Aplicaciones con soluciones SSO existentes

## <a name="general-issues-to-check-first"></a>Problemas generales para comprobar primero

-   Si se acaba de agregar una aplicación a un usuario, intente volver a iniciar sesión de entrada y salida en el panel de acceso del usuario después de unos minutos para ver si la aplicación se ha agregado.

-   Si se acaba de quitar una licencia de un usuario o grupo del que el usuario es miembro, los cambios pueden tardar tiempo, en función del tamaño y la complejidad del grupo. Espere un poco más de tiempo antes de iniciar sesión en el panel de acceso.

## <a name="problems-related-to-application-configuration"></a>Problemas relacionados con la adición o configuración de aplicaciones

Puede que una aplicación no aparezca en el panel de acceso de un usuario porque no está configurada correctamente. A continuación se muestran algunas maneras de solucionar problemas relacionados con la configuración de la aplicación:

-   [Configuración del inicio de sesión único federado para una aplicación de la galería de Azure AD](#how-to-configure-federated-single-sign-on-for-an-azure-ad-gallery-application)

-   [Configuración del inicio de sesión único federado para una aplicación ajena a la galería](#how-to-configure-federated-single-sign-on-for-a-non-gallery-application)

-   [Configuración del inicio de sesión único con contraseña para una aplicación de la galería de Azure AD](#how-to-configure-password-single-sign-on-for-a-non-gallery-application)

-   [Configuración del inicio de sesión único con contraseña para una aplicación ajena a la galería](#how-to-configure-password-single-sign-on-for-a-non-gallery-application)

### <a name="how-to-configure-federated-single-sign-on-for-an-azure-ad-gallery-application"></a>Configuración del inicio de sesión único federado para una aplicación de la galería de Azure AD

Todas las aplicaciones de la galería de Azure AD habilitadas con la funcionalidad de Enterprise Single Sign-On tienen un tutorial paso a paso disponible. Para instrucciones detalladas paso a paso, acceda a la [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-saas-tutorial-list/).

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

-   [Incorporación de una aplicación desde la galería de Azure AD](#add-an-application-from-the-azure-ad-gallery)

-   [Configuración de los valores de los metadatos de la aplicación en Azure AD (URL de inicio de sesión, identificador, URL de respuesta)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   [Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [Recuperación de los metadatos y el certificado de Azure AD](#download-the-azure-ad-metadata-or-certificate)

-   [Configuración de los valores de los metadatos de Azure AD en la aplicación (URL de inicio de sesión, emisor, URL de cierre de sesión y certificado)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

#### <a name="add-an-application-from-the-azure-ad-gallery"></a>Incorporación de una aplicación desde la galería de Azure AD

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

#### <a name="configure-single-sign-on-for-an-application-from-the-azure-ad-gallery"></a>Configuración del inicio de sesión único para una aplicación de la galería de Azure AD

Para configurar el inicio de sesión único para una aplicación, siga los pasos siguientes:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

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

#### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación

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
   >Azure AD selecciona el formato del atributo NameID (identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest).
   >
   >

9. Para agregar atributos de usuario, haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

   Para agregar un atributo:

   1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

   2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

#### <a name="download-the-azure-ad-metadata-or-certificate"></a>Descarga del certificado o los metadatos de Azure AD

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

### <a name="how-to-configure-federated-single-sign-on-for-a-non-gallery-application"></a>Configuración del inicio de sesión único federado para una aplicación ajena a la galería

Para configurar una aplicación ajena a la galería, debe tener Azure AD Premium y que la aplicación admita SAML 2.0. Para más información acerca de las versiones de Azure AD, visite [Precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

-   [Configuración de los valores de los metadatos de la aplicación en Azure AD (URL de inicio de sesión, identificador, URL de respuesta)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   [Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [Recuperación de los metadatos y el certificado de Azure AD](#download-the-azure-ad-metadata-or-certificate)

-   [Configuración de los valores de los metadatos de Azure AD en la aplicación (URL de inicio de sesión, emisor, URL de cierre de sesión y certificado)](#configure-the-application-for-password-single-sign-on-1)

#### <a name="configure-the-applications-metadata-values-in-azure-ad-sign-on-url-identifier-reply-url"></a>Configuración de los valores de los metadatos de la aplicación en Azure AD (URL de inicio de sesión, identificador, URL de respuesta)

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

    2.  **Opcional:** Para configurar la aplicación como SSO iniciado por el SP, la dirección URL de inicio de sesión es obligatoria.

12. En **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**.

13. **Opcional:** haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

    Para agregar un atributo:

    1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

    2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

14. Haga clic en **Configurar &lt;nombre de la aplicación&gt;** para acceder a documentación sobre cómo configurar el inicio de sesión único en la aplicación. Además, tiene las direcciones URL de Azure AD y los certificados necesarios para la aplicación.

#### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>Selección del identificador de usuario e incorporación de los atributos de usuario para enviarlos a la aplicación

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
   >Azure AD selecciona el formato del atributo NameID (identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest).
   >
   >

9. Para agregar atributos de usuario, haga clic en **Ver y editar todos los demás atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

   Para agregar un atributo:

   1. Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

   2. Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

#### <a name="download-the-azure-ad-metadata-or-certificate"></a>Descarga del certificado o los metadatos de Azure AD

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

### <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación de la galería de Azure AD

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

-   [Incorporación de una aplicación desde la galería de Azure AD](#add-an-application-from-the-azure-ad-gallery)

-   [Configuración de la aplicación para el inicio de sesión único con contraseña](#configure-the-application-for-password-single-sign-on)

#### <a name="add-an-application-from-the-azure-ad-gallery"></a>Incorporación de una aplicación desde la galería de Azure AD

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

#### <a name="configure-the-application-for-password-single-sign-on"></a>Configuración de la aplicación para el inicio de sesión único con contraseña

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

9. [Asigne usuarios a la aplicación](#how-to-assign-a-user-to-an-application-directly).

10. Además, también puede proporcionar credenciales en nombre del usuario; para ello, seleccione las filas de los usuarios, haga clic en **Actualizar credenciales** y escriba el nombre de usuario y la contraseña en nombre de los usuarios. En caso contrario, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.

### <a name="how-to-configure-password-single-sign-on-for-a-non-gallery-application"></a>Configuración del inicio de sesión único con contraseña para una aplicación ajena a la galería

Para configurar una aplicación desde la galería de Azure AD, realice los siguientes pasos:

-   [Incorporación de una aplicación ajena a la galería](#add-a-non-gallery-application)

-   [Configuración de la aplicación para el inicio de sesión único con contraseña](#configure-the-application-for-password-single-sign-on)

#### <a name="add-a-non-gallery-application"></a>Incorporación de una aplicación ajena a la galería

Para agregar una aplicación desde la galería de Azure AD, siga estos pasos:

1.  Abra [Azure Portal](https://portal.azure.com) e inicie sesión como **administrador global** o **coadministrador**.

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en el botón **Agregar** situado en la esquina superior derecha del panel **Aplicaciones empresariales**.

6.  Haga clic en **Aplicación situada fuera de la galería**.

7.  Escriba el nombre de la aplicación en el cuadro de texto **Nombre**. Seleccione **Agregar**.

Tras un breve período, podrá ver el panel de configuración de la aplicación.

#### <a name="configure-the-application-for-password-single-sign-on"></a>Configuración de la aplicación para el inicio de sesión único con contraseña

Para configurar el inicio de sesión único para una aplicación, siga los pasos siguientes:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

    1.  Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6.  Seleccione la aplicación que desea configurar para el inicio de sesión único.

7.  Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8.  Seleccione el modo de **Inicio de sesión con contraseña**.

9.  Escriba la **dirección URL de inicio de sesión**. Es la dirección URL en la que los usuarios escriben su nombre de usuario y contraseña para iniciar sesión. Asegúrese de que los campos de inicio de sesión estén visibles en la dirección URL.

10. [Asigne usuarios a la aplicación](#how-to-assign-a-user-to-an-application-directly).

11. Además, también puede proporcionar credenciales en nombre del usuario; para ello, seleccione las filas de los usuarios, haga clic en **Actualizar credenciales** y escriba el nombre de usuario y la contraseña en nombre de los usuarios. En caso contrario, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.

## <a name="problems-related-to-assigning-applications-to-users"></a>Problemas relacionados con la asignación de aplicaciones a los usuarios

Puede que un usuario no vea una aplicación en su Panel de acceso porque no ha sido asignado a la aplicación. A continuación se muestran algunas maneras de comprobarlo:

-   [Comprobación de si un usuario está asignado a la aplicación](#check-if-a-user-is-assigned-to-the-application)

-   [Asignación de un usuario a una aplicación directamente](#how-to-assign-a-user-to-an-application-directly)

-   [Comprobación de si un usuario posee una licencia relacionada con la aplicación](#check-if-a-user-is-under-a-license-related-to-the-application)

-   [Asignación de una licencia a un usuario](#how-to-assign-a-user-a-license)

### <a name="check-if-a-user-is-assigned-to-the-application"></a>Comprobación de si un usuario está asignado a la aplicación

Para comprobar si un usuario está asignado a la aplicación, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

6. **Busque** el nombre de la aplicación en cuestión.

7. Haga clic en **Usuarios y grupos**.

8. Compruebe si el usuario está asignado a la aplicación.

   * Si no, siga los pasos descritos en "Asignación de un usuario a una aplicación directamente" para hacerlo.

### <a name="how-to-assign-a-user-to-an-application-directly"></a>Asignación de un usuario a una aplicación directamente

Para asignar uno o varios usuarios a una aplicación directamente, siga los pasos siguientes:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global**.

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

### <a name="check-if-a-user-is-under-a-license-related-to-the-application"></a>Comprobación de si un usuario posee una licencia relacionada con la aplicación

Para comprobar las licencias asignadas de un usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7. Haga clic en **Licencias** para ver qué licencias tiene asignadas actualmente el usuario.

   * Si el usuario está asignado a una licencia de Office, esto permitirá que las aplicaciones de Office aparezcan en el panel de acceso del usuario.

### <a name="how-to-assign-a-user-a-license"></a>Asignación de una licencia a un usuario 

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

## <a name="problems-related-to-assigning-applications-to-groups"></a>Problemas relacionados con la asignación de aplicaciones a grupos

Puede que un usuario vea una aplicación en su Panel de acceso porque forme parte de un grupo al que se ha asignado esa aplicación. A continuación se muestran algunas maneras de comprobarlo:

-   [Comprobación de la pertenencia a grupos de un usuario](#check-a-users-group-memberships)

-   [Asignación de una aplicación a un grupo directamente](#how-to-assign-an-application-to-a-group-directly)

-   [Comprobación de si un usuario es miembro de un grupo asignado a una licencia](#check-if-a-user-is-part-of-group-assigned-to-a-license)

-   [Asignación de una licencia a un grupo](#how-to-assign-a-license-to-a-group)

### <a name="check-a-users-group-memberships"></a>Comprobación de la pertenencia a grupos de un usuario

Para comprobar la pertenencia de un grupo, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7. Haga clic en **Grupos**.

8. Compruebe si el usuario forma parte de un grupo que se ha asignado a la aplicación.

   * Si desea quitar el usuario del grupo, **haga clic en la fila** del grupo y seleccione Eliminar.

### <a name="how-to-assign-an-application-to-a-group-directly"></a>Asignación de una aplicación a un grupo directamente

Para asignar uno o varios grupos a una aplicación directamente, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global**.

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea asignar a un usuario de la lista.

7. Cuando se cargue la aplicación, haga clic en **Usuarios y grupos** desde el menú de navegación izquierdo de la aplicación.

8. Haga clic en el botón **Agregar** en la parte superior de la lista **Usuarios y grupos** para abrir el panel **Agregar asignación**.

9. Haga clic en el selector **Usuarios y grupos** del panel **Agregar asignación**.

10. Escriba el **nombre completo** del grupo al que quiere asignar en el cuadro de búsqueda **Buscar por nombre o dirección de correo**.

11. Mantenga el puntero sobre el **grupo** en la lista para que aparezca una **casilla**. Haga clic en la casilla situada junto a la foto o el logotipo del perfil del grupo para agregar ese usuario a la lista de **seleccionados**.

12. **Opcional:** si desea **agregar más de un grupo**, escriba otro **nombre de grupo completo** o en el cuadro de búsqueda **Buscar por nombre o dirección de correo** y haga clic en la casilla para agregar ese grupo a la lista de **seleccionados**.

13. Cuando haya terminado de seleccionar grupos, haga clic en el botón **Seleccionar** para agregarlos a la lista de usuarios y grupos que se asignarán a la aplicación.

14. **Opcional:** haga clic en el selector **Seleccionar rol** del panel **Agregar asignación** para seleccionar un rol que se asignará a los grupos que ha seleccionado.

15. Haga clic en el botón **Asignar** para asignar la aplicación a los grupos seleccionados.

Tras un breve período, los usuarios que seleccionó podrán iniciar estas aplicaciones en el panel de acceso.

### <a name="check-if-a-user-is-part-of-group-assigned-to-a-license"></a>Comprobación de si un usuario es miembro de un grupo asignado a una licencia

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Usuarios y grupos** en el menú de navegación.

5. Haga clic en **Todos los usuarios**.

6. **Busque** el usuario en el que está interesado y **haga clic en la fila** para seleccionarlo.

7. Haga clic en **Grupos**.

8. Haga clic en la fila de un grupo específico.

9. Haga clic en **Licencias** para ver qué licencias tiene asignadas el grupo.

   * Si el grupo está asignado a una licencia de Office, esto permitirá que determinadas aplicaciones de Office aparezcan en el Panel de acceso del usuario.

### <a name="how-to-assign-a-license-to-a-group"></a>Asignación de una licencia a un grupo

Para asignar una licencia a un grupo, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Usuarios y grupos** en el menú de navegación.

5.  Haga clic en **Todos los grupos**.

6.  **Busque** el grupo en el que está interesado y **haga clic en la fila** para seleccionarlo.

7.  Haga clic en **Licencias** para ver qué licencias tiene asignadas actualmente el grupo.

8.  Haga clic en el botón **Asignar**.

9.  Seleccione **uno o más productos** en la lista de productos disponibles.

10. **Opcional**: Haga clic en el elemento **Opciones de asignación** para asignar productos de forma granular. Cuando haya finalizado este procedimiento, haga clic en **Aceptar**.

11. Haga clic en el botón **Asignar** para asignar estas licencias a este grupo. Esta operación puede tardar mucho tiempo, en función del tamaño y la complejidad del grupo.

>[!NOTE]
>Para acelerar el proceso, considere la posibilidad de asignar temporalmente una licencia al usuario directamente. 
>
>

## <a name="next-steps"></a>Pasos siguientes
[Adición de nuevos usuarios a Azure Active Directory](./../fundamentals/add-users-azure-active-directory.md)

