---
title: 'Tutorial: Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SAP NetWeaver | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAP NetWeaver.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/14/2019
ms.author: jeedes
ms.openlocfilehash: 38ac4f1bf6a1dd4656b4e7d5783051f3b381940c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546821"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-netweaver"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SAP NetWeaver

En este tutorial, aprenderá a integrar SAP NetWeaver con Azure Active Directory (Azure AD). Al integrar SAP NetWeaver con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SAP NetWeaver.
* Permitir que los usuarios inicien sesión automáticamente en SAP NetWeaver con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en SAP NetWeaver
* Se requiere SAP NetWeaver V7.20 como mínimo

## <a name="scenario-description"></a>Descripción del escenario

SAP NetWeaver admite tanto **SAML** (**SSO iniciado por el SP**) como **OAuth**. En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 

> [!NOTE]
> Configure la aplicación en SAML o en OAuth, según el requisito de la organización. 

## <a name="adding-sap-netweaver-from-the-gallery"></a>Adición de SAP NetWeaver desde la galería

Para configurar la integración de SAP NetWeaver en Azure AD, deberá agregar esta solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SAP NetWeaver** en el cuadro de búsqueda.
1. Seleccione **SAP NetWeaver** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sap-netweaver"></a>Configuración y prueba del inicio de sesión único de Azure AD para SAP NetWeaver

Configure y pruebe el inicio de sesión único de Azure AD con SAP NetWeaver mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SAP NetWeaver.

Para configurar y probar el inicio de sesión único de Azure AD con SAP NetWeaver, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración de SAP NetWeaver con SAML](#configure-sap-netweaver-using-saml)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SAP NetWeaver](#create-sap-netweaver-test-user)** , para tener un homólogo de B.Simon en SAP NetWeaver vinculado a su representación en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.
1. **[Configuración de SAP NetWeaver para OAuth](#configure-sap-netweaver-for-oauth)** , para configurar las opciones de OAuth en la aplicación.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SAP NetWeaver, realice los pasos siguientes:

1. Abra una ventana del explorador web e inicie sesión en el sitio de la compañía SAP NetWeaver como administrador.

1. Asegúrese de que los servicios **http** y **https** están activos y de que se asignan los puertos adecuados en el código de transacción **SMICM**.

1. Inicie sesión en el cliente empresarial del sistema de SAP (T01) donde se requiere el inicio de sesión único y active la administración de sesiones de seguridad HTTP.

    a. Vaya al código de transacción **SICF_SESSIONS**. Se muestran todos los parámetros de perfil correspondientes con sus valores actuales. Se ven como se muestra a continuación:
    ```
    login/create_sso2_ticket = 2
    login/accept_sso2_ticket = 1
    login/ticketcache_entries_max = 1000
    login/ticketcache_off = 0  login/ticket_only_by_https = 0 
    icf/set_HTTPonly_flag_on_cookies = 3
    icf/user_recheck = 0  http/security_session_timeout = 1800
    http/security_context_cache_size = 2500
    rdisp/plugin_auto_logout = 1800
    rdisp/autothtime = 60
    ```
    >[!NOTE]
    > Ajuste los parámetros anteriores según las necesidades de la organización. Los parámetros anteriores se proporcionan solo como indicación.

    b. De ser necesario, ajuste los parámetros en el perfil de instancia o predeterminado del sistema SAP y reinicie el sistema SAP.

    c. Haga doble clic en el cliente pertinente para habilitar la sesión de seguridad HTTP.

    ![Vínculo de descarga del certificado](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_profileparameter.png)

    d. Active los siguientes servicios SICF:
    ```
    /sap/public/bc/sec/saml2
    /sap/public/bc/sec/cdc_ext_service
    /sap/bc/webdynpro/sap/saml2
    /sap/bc/webdynpro/sap/sec_diag_tool (This is only to enable / disable trace)
    ```
1. Vaya al código de transacción **SAML2** en el cliente empresarial del sistema SAP [T01/122]. Se abrirá una interfaz de usuario en el explorador. En este ejemplo, se supone a 122 como el cliente empresarial SAP.

    ![Vínculo de descarga del certificado](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_sapbusinessclient.png)

1. Indique su nombre de usuario y contraseña para ingresar a la interfaz de usuario y haga clic en **Editar**.

    ![Vínculo de descarga del certificado](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_userpwd.png)

1. Reemplace el **nombre del proveedor** de T01122 por `http://T01122` y haga clic en **Guardar**.

    > [!NOTE]
    > De forma predeterminada, el nombre de proveedor tiene el formato `<sid><client>`, pero Azure AD espera que el nombre tenga el formato `<protocol>://<name>`. Se recomienda que el nombre del proveedor se mantenga como `https://<sid><client>` para permitir la configuración de varios motores ABAP de SAP NetWeaver en Azure AD.

    ![Vínculo de descarga del certificado](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_providername.png)

1. **Generación de metadatos del proveedor de servicio**: una vez que hemos terminado con la configuración del **proveedor local** y de los **proveedores de confianza** en la interfaz de usuario de SAML 2.0, el siguiente paso sería generar el archivo de metadatos del proveedor de servicios (que contendrá todas las configuraciones, contextos de autenticación y otras configuraciones de SAP). Una vez que se genera este archivo, es necesario cargarlo en Azure AD.

    ![Vínculo de descarga del certificado](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_generatesp.png)

    a. Vaya a **la pestaña Proveedor local**.

    b. Haga clic en **Metadatos**.

    c. Guarde el **archivo de metadatos XML** generado en el equipo y cárguelo en la sección **Configuración básica de SAML** para rellenar de forma automática los valores **Identificador** y **URL de respuesta** en Azure Portal.

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **SAP NetWeaver**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    a. Haga clic en **Cargar archivo de metadatos** para cargar el **archivo de metadatos del proveedor de servicios** que obtuvo anteriormente.

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    c. Una vez que se haya cargado correctamente el archivo de metadatos, los valores **Identificador** y **Dirección URL de respuesta** se rellenan automáticamente en el cuadro de texto de la sección **Configuración básica de SAML**, como se muestra a continuación:

    d. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<your company instance of SAP NetWeaver>`

    > [!NOTE]
    > Hemos visto que algunos clientes informan de que se ha producido un error al configurar una dirección URL de respuesta incorrecta para su instancia. Si recibe algún error de ese tipo, puede usar el siguiente script de PowerShell como forma provisional de establecer la dirección URL de respuesta correcta para la instancia:
    > ```
    > Set-AzureADServicePrincipal -ObjectId $ServicePrincipalObjectId -ReplyUrls "<Your Correct Reply URL(s)>"
    > ``` 
    > Puede establecer usted mismo el identificador de objeto de ServicePrincipal en primer lugar o bien puede pasarlo también aquí.

1. La aplicación SAP NetWeaver espera las aserciones de SAML en un formato específico, lo que requiere que se agreguen asignaciones de atributo personalizadas a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

1. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, configure el atributo Token SAML como muestra la imagen anterior y realice los siguientes pasos:

    a. Haga clic en el **icono Editar** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](./media/sapnetweaver-tutorial/nameidattribute.png)

    ![imagen](./media/sapnetweaver-tutorial/nameidattribute1.png)

    b. En la lista **Transformación**, seleccione **ExtractMailPrefix()** .

    c. En la lista **Parámetro 1**, seleccione **user.userprincipalname**.

    d. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up SAP NetWeaver** (Configurar SAP NetWeaver), copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
    1. En el campo **Nombre**, escriba `B.Simon`.  
    1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
    1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
    1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a SAP NetWeaver mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SAP NetWeaver**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sap-netweaver-using-saml"></a>Configuración de SAP NetWeaver con SAML

1. Inicie sesión en el sistema SAP y vaya al código de transacción SAML2. Se abre una nueva ventana del explorador con la pantalla de configuración de SAML.

2. Para configurar puntos de conexión para el proveedor de identidades de confianza (Azure AD), vaya a la pestaña **Proveedores de confianza**.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_samlconfig.png)

3. Presione **Agregar** y seleccione **Cargar archivo de metadatos** en el menú contextual.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_uploadmetadata.png)

4. Cargue el archivo de metadatos que ha descargado desde Azure Portal.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_metadatafile.png)

5. En la siguiente pantalla, escriba el nombre de alias. Por ejemplo, aadsts, y presione **Siguiente** para continuar.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_aliasname.png)

6. Asegúrese de que su **algoritmo de síntesis** sea **SHA-256** y no requiera de ningún cambio y presione **Siguiente**.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_identityprovider.png)

7. En **Single Sign-On Endpoints** (Puntos de conexión de inicio de sesión único), utilice **HTTP POST** y haga clic en **Siguiente** para continuar.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_httpredirect.png)

8. En **Single Logout Endpoints** (Puntos de conexión de cierre de sesión único), seleccione **HTTPRedirect** y haga clic en **Siguiente** para continuar.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_httpredirect1.png)

9. En **Artifact Endpoints** (Puntos de conexión de artefacto), presione **Siguiente** para continuar.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_artifactendpoint.png)

10. En **Requisitos de autenticación**, haga clic en **Finalizar**.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_authentication.png)

11. Vaya a la pestaña **Proveedor de confianza** > **Federación de identidades** (desde la parte inferior de la pantalla). Haga clic en **Editar**.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_trustedprovider.png)

12. Haga clic en **Agregar** bajo la pestaña **Federación de identidades** (ventana de la parte inferior).

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_addidentityprovider.png)

13. En la ventana emergente, seleccione **Sin especificar** en **Formatos de NameID admitidos** y haga clic en Aceptar.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_nameid.png)

14. Tenga en cuenta que los valores **origen del id. de usuario** y **modo de asignación de id. de usuario** determinan el vínculo entre el usuario SAP y la notificación de Azure AD.  

    #### <a name="scenario-sap-user-to-azure-ad-user-mapping"></a>Escenario: Usuario SAP para la asignación de usuario de Azure AD.

    a. Captura de pantalla de los detalles de NameID de SAP.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/nameiddetails.png)

    b. Captura de pantalla en la que se mencionan las notificaciones necesarias de Azure AD.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/claimsaad1.png)

    #### <a name="scenario-select-sap-user-id-based-on-configured-email-address-in-su01-in-this-case-email-id-should-be-configured-in-su01-for-each-user-who-requires-sso"></a>Escenario: Seleccione el identificador de usuario SAP según la dirección de correo electrónico configurada en SU01. En este caso, el identificador de correo electrónico debe configurarse en su01 para cada usuario que requiera del inicio de sesión único.

    a.  Captura de pantalla de los detalles de NameID de SAP.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_nameiddetails1.png)

    b. Captura de pantalla en la que se mencionan las notificaciones necesarias de Azure AD.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/claimsaad2.png)

15. Haga clic en **Guardar** y, a continuación, haga clic en **Habilitar** para habilitar el proveedor de identidades.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/configuration1.png)

16. Haga clic en **Aceptar** cuando se le solicite.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/configuration2.png)

    ### <a name="create-sap-netweaver-test-user"></a>Creación de un usuario de prueba de SAP NetWeaver

    En esta sección, va a crear un usuario llamado B.Simon en SAP NetWeaver. Colabore con su equipo experto en SAP interno o con el asociado SAP de su organización para agregar usuarios en la plataforma de SAP NetWeaver.

## <a name="test-sso"></a>Prueba de SSO

1. Una vez que active el proveedor de identidades de Azure AD, intente acceder a la siguiente dirección URL para comprobar el inicio de sesión único (no se solicitará su nombre de usuario y contraseña)

    `https://<sapurl>/sap/bc/bsp/sap/it00/default.htm`

    (o) use la siguiente dirección URL

    `https://<sapurl>/sap/bc/bsp/sap/it00/default.htm`

    > [!NOTE]
    > Reemplace sapurl por el nombre de host real de SAP.

2. La dirección URL anterior debería dirigirlo a la pantalla que se menciona a continuación. Si puede acceder a la página a continuación, la configuración de SSO de Azure AD se realizó correctamente.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/testingsso.png)

3. Si se le solicita el nombre de usuario y contraseña, diagnostique el problema al habilitar el seguimiento mediante la siguiente dirección URL

    `https://<sapurl>/sap/bc/webdynpro/sap/sec_diag_tool?sap-client=122&sap-language=EN#`

## <a name="configure-sap-netweaver-for-oauth"></a>Configuración de SAP NetWeaver para OAuth

1. El proceso documentado de SAP está disponible en la ubicación: [NetWeaver Gateway Service Enabling and OAuth 2.0 Scope Creation](https://wiki.scn.sap.com/wiki/display/Security/NetWeaver+Gateway+Service+Enabling+and+OAuth+2.0+Scope+Creation) (Habilitación de un servicio de NetWeaver Gateway y creación del ámbito de OAuth 2.0).

2. Vaya a SPRO y busque **Activate and Maintain Services** (Activar y mantener servicios).

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth01.png)

3. En este ejemplo, queremos conectar el servicio de OData: `DAAG_MNGGRP` con OAuth para el inicio de sesión único de Azure AD. Use la búsqueda de nombre de servicio técnico para localizar el servicio `DAAG_MNGGRP` y actívelo si no lo está ya (busque el estado `green` en la pestaña ICF Nodes [Nodos de ICF]). Asegúrese de que el alias del sistema (el sistema back-end conectado en el que se ejecuta realmente el servicio) es correcto.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth02.png)

    * Después, haga clic en el botón **OAuth** en la barra de botones superior y asigne el valor de `scope` (mantenga el nombre predeterminado tal como se ofrece).

4. En nuestro ejemplo, el ámbito es `DAAG_MNGGRP_001`, se genera a partir del nombre del servicio agregando automáticamente un número. Se puede usar el informe `/IWFND/R_OAUTH_SCOPES` para cambiar el nombre del ámbito o crearlo manualmente.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth03.png)

    > [!NOTE]
    > El mensaje `soft state status is not supported` se puede pasar por alto, no supone ningún problema. Para más detalles, consulte [aquí](https://help.sap.com/doc/saphelp_nw74/7.4.16/1e/c60c33be784846aad62716b4a1df39/content.htm?no_cache=true).

### <a name="create-a-service-user-for-the-oauth-20-client"></a>Creación de un usuario de servicio para el cliente de OAuth 2.0

1. OAuth2 usa un valor de `service ID` para obtener el token de acceso para el usuario final en su nombre. Existe una restricción importante que impone el diseño de OAuth: el valor de `OAuth 2.0 Client ID` debe ser idéntico al valor de `username` que utiliza el cliente de OAuth 2.0 para iniciar sesión cuando se solicita un token de acceso. Por lo tanto, en nuestro ejemplo vamos a registrar un cliente de OAuth 2.0 con el nombre CLIENT1; como requisito previo, en el sistema SAP deberá existir un usuario con el mismo nombre (CLIENT1). Este usuario se configurará para su uso en la aplicación a la que se hace referencia. 

2. Al registrar un cliente de OAuth, se usa el valor de `SAML Bearer Grant type`.

    >[!NOTE]
    >Para más información, consulte el artículo sobre el registro de cliente de OAuth 2.0 para el tipo de concesión de portador de SAML [aquí](https://wiki.scn.sap.com/wiki/display/Security/OAuth+2.0+Client+Registration+for+the+SAML+Bearer+Grant+Type).

3. En el código de transacción SU01, cree el usuario CLIENT1 como `System type` y asígnele una contraseña; guárdela como sea necesario para proporcionar la credencial al programador de la API, que debe grabarla con el nombre de usuario en el código de llamada. No se debe asignar ningún perfil ni rol.

### <a name="register-the-new-oauth-20-client-id-with-the-creation-wizard"></a>Registro del nuevo identificador de cliente de OAuth 2.0 con el asistente para creación

1. Para registrar un nuevo **cliente de OAuth 2.0**, inicie la transacción **SOAUTH2**. La transacción mostrará información general acerca de los clientes de OAuth 2.0 que ya se han registrado. Elija **Create** (Crear) para iniciar el asistente para el nuevo cliente de OAuth, denominado CLIENT1 en este ejemplo.

2. Vaya al código de transacción **SOAUTH2** y proporcione la descripción; después, haga clic en **Next** (Siguiente).

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth04.png)

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth05.png)

3. En la lista desplegable, seleccione el proveedor de identidades **SAML2 IdP – Azure AD** ya agregado y guarde.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth06.png)

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth07.png)

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth08.png)

4. Haga clic en **Add** (Agregar) en la asignación de ámbito para agregar el ámbito creado anteriormente: `DAAG_MNGGRP_001`.

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth09.png)

    ![Configurar inicio de sesión único](./media/sapnetweaver-tutorial/oauth10.png)

5. Haga clic en **Finish** (Finalizar).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe SAP NetWeaver con Azure AD](https://aad.portal.azure.com/).