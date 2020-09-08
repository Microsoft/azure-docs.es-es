---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SAP Fiori | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAP Fiori.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/05/2019
ms.author: jeedes
ms.openlocfilehash: 4d2f6766fa32beb9cebaa8f77c04f6865e15e14d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543401"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-fiori"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SAP Fiori

En este tutorial aprenderá a integrar SAP Fiori con Azure Active Directory (Azure AD). Al integrar SAP Fiori con Azure AD, puede hacer lo siguiente:

* Controlar quién tiene acceso en Azure AD a SAP Fiori.
* Permitir que los usuarios puedan iniciar sesión automáticamente en SAP Fiori con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en SAP Fiori.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAP Fiori admite el inicio de sesión único iniciado por **SP**

> [!NOTE]
> En el caso de la autenticación de iFrame iniciada por SAP Fiori, se recomienda usar el parámetro **IsPassive** de AuthnRequest de SAML para la autenticación silenciosa. Para más detalles sobre el parámetro **isPassive**, consulte la información de [Inicio de sesión único de SAML de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/single-sign-on-saml-protocol).

## <a name="adding-sap-fiori-from-the-gallery"></a>Incorporación de SAP Fiori desde la galería

Para configurar la integración de SAP Fiori en Azure AD, será preciso que agregue SAP Fiori desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SAP Fiori** en el cuadro de búsqueda.
1. Seleccione **SAP Fiori** en el panel de resultados y, después, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sap-fiori"></a>Configuración y prueba del inicio de sesión único de Azure AD para SAP Fiori

Configure y pruebe el inicio de sesión único de Azure AD con SAP Fiori mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SAP Fiori.

Para configurar y probar el inicio de sesión único de Azure AD con SAP Fiori, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en SAP Fiori](#configure-sap-fiori-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de SAP Fiori](#create-sap-fiori-test-user)** , para tener un homólogo de B. Simon en SAP Fiori vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. Abra una ventana del explorador web e inicie sesión como administrador en el sitio SAP Fiori de su compañía.

1. Asegúrese de que los servicios **http** y **https** están activos y de que se asignan los puertos pertinentes al código de transacción **SMICM**.

1. Inicie sesión en SAP Business Client para el sistema SAP **T01**, en donde el inicio de sesión único es obligatorio. A continuación, activar la administración de sesiones de seguridad de HTTP.

    1. Vaya al código de transacción **SICF_SESSIONS**. Se muestran todos los parámetros de perfil pertinentes con sus valores actuales. Deben tener un aspecto similar al ejemplo siguiente:

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
        > Ajuste los parámetros en base a los requisitos de la organización. Los parámetros anteriores se proporcionan únicamente como ejemplo.

    1. Si es necesario, ajuste los parámetros en el perfil de instancia (predeterminado) del sistema SAP y reinicie el sistema SAP.

    1. Haga doble clic en el cliente apropiado para habilitar la sesión de seguridad HTTP.

        ![Página con los valores actuales de los parámetros de perfil pertinentes en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-profileparameter.png)

    1. Active los servicios SICF siguientes:

        ```
        /sap/public/bc/sec/saml2
        /sap/public/bc/sec/cdc_ext_service
        /sap/bc/webdynpro/sap/saml2
        /sap/bc/webdynpro/sap/sec_diag_tool (This is only to enable / disable trace)
        ```

1. Vaya al código de transacción **SAML2** en el cliente empresarial del sistema SAP [**T01/122**]. La interfaz de usuario de configuración se abre en una nueva ventana del explorador. En este ejemplo, utilizamos el cliente empresarial del sistema SAP 122.

    ![La página de inicio de sesión de cliente empresarial de SAP Fiori](./media/sapfiori-tutorial/tutorial-sapnetweaver-sapbusinessclient.png)

1. Escriba su nombre de usuario y contraseña, y haga clic en **Log On** (Entrar al sistema).

    ![La página de configuración SAML 2.0 de ABAP sistema T01/122 en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-userpwd.png)

1. En el cuadro **Provider Name** (Nombre de proveedor), remplace **T01122** con **http:\//T01122** y después seleccione **Save** (Guardar).

    > [!NOTE]
    > De forma predeterminada, el nombre del proveedor se encuentra en el formato \<sid>\<client>. Azure AD espera el nombre en el formato \<protocol>://\<name>. Se recomienda mantener el nombre del proveedor como https\://\<sid>\<client> para poder configurar varios motores de ABAP de SAP Fiori en Azure AD.

    ![El nombre del proveedor actualizado en la página de configuración SAML 2.0 de ABAP sistema T01/122 en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-providername.png)

1. Seleccione la **pestaña Local Provider** (Proveedor local) > **Metadata** (Metadatos).

1. En el cuadro de diálogo **SAML 2.0 Metadata** (Metadatos de SAML 2.0), descargue el archivo XML de metadatos generado y guárdelo en su equipo.

    ![El vínculo de descarga de metadatos en el cuadro de diálogo de metadatos de SAP SAML 2.0](./media/sapfiori-tutorial/tutorial-sapnetweaver-generatesp.png)

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **SAP Fiori** , busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![Carga del archivo de metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Elección del archivo de metadatos](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, los valores de **Identificador** y **URL de respuesta** se rellenan automáticamente en el panel **Configuración básica de SAML**. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https:\//\<your company instance of SAP Fiori\>`.

    > [!NOTE]
    > Algunos clientes han informado sobre errores relacionados con la configuración incorrecta de los valores de **URL de respuesta**. Si ve este error, puede usar el siguiente script de PowerShell para establecer la dirección URL de respuesta correcta para la instancia:
    >
    > ```
    > Set-AzureADServicePrincipal -ObjectId $ServicePrincipalObjectId -ReplyUrls "<Your Correct Reply URL(s)>"
    > ``` 
    > 
    > Puede establecer el identificador de objeto `ServicePrincipal` usted mismo antes de ejecutar el script, o puede pasarlo aquí.

1. La aplicación SAP Fiori espera que las aserciones de SAML estén en un formato específico. Configure las siguientes notificaciones para esta aplicación. Para administrar estos valores de atributo, en el panel **Configurar el inicio de sesión único con SAML**, seleccione **Editar**.

    ![El panel Atributos de usuario](common/edit-attribute.png)

1. En el panel **User Attributes & Claims** (Atributos y notificaciones del usuario), configure los atributos del token de SAML, tal como se muestra en la imagen anterior. Después, complete los siguientes pasos:

    1. Seleccione **Editar** para abrir el panel **Administrar las notificaciones del usuario**.

    1. En la lista **Transformación**, seleccione **ExtractMailPrefix()** .

    1. En la lista **Parámetro 1**, seleccione **user.userprincipalname**.

    1. Seleccione **Guardar**.

       ![El panel Administrar las notificaciones del usuario](./media/sapfiori-tutorial/nameidattribute.png)

       ![La sección Transformación en el panel Administrar las notificaciones del usuario](./media/sapfiori-tutorial/nameidattribute1.png)
    
1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up SAP Fiori** (Configurar SAP Fiori), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a SAP Fiori mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SAP Fiori**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sap-fiori-sso"></a>Configuración de inicio de sesión único en SAP Fiori

1. Inicie sesión en el sistema SAP y vaya al código de transacción **SAML2**. Se abre una nueva ventana del explorador con la página de configuración de SAML.

1. Para configurar los puntos de conexión para un proveedor de identidades de confianza (Azure AD), vaya a la pestaña **Trusted Providers** (Proveedores de confianza).

    ![La pestaña de proveedores de confianza en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-samlconfig.png)

1. Seleccione **Add** (Agregar) y luego **Upload Metadata File** (Cargar archivo de metadatos) en el menú contextual.

    ![Las opciones para agregar y cargar el archivo de metadatos en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-uploadmetadata.png)

1. Cargue el archivo de metadatos que descargó de Azure Portal. Seleccione **Next** (Siguiente).

    ![Seleccione el archivo de metadatos para cargar en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-metadatafile.png)

1. En la página siguiente, en el cuadro **Alias**, escriba el nombre de alias. Por ejemplo, **aadsts**. Seleccione **Next** (Siguiente).

    ![El cuadro de alias en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-aliasname.png)

1. Asegúrese de que el valor en el cuadro **Digest Algorithm** (Algoritmo de codificación) es **SHA-256**. Seleccione **Next** (Siguiente).

    ![Comprobación del valor del algoritmo de codificación en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-identityprovider.png)

1. En **Single Sign-On Endpoints** (Puntos de conexión del inicio de sesión único), seleccione **HTTP POST**y, a continuación, seleccione **Next** (Siguiente).

    ![Opciones de los puntos de conexión del inicio de sesión único en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-httpredirect.png)

1. En **Single Logout Endpoints** (Puntos de conexión del cierre de sesión único), seleccione **HTTP Redirect** y, a continuación, seleccione **Next** (Siguiente).

    ![Opciones de los puntos de conexión del cierre de sesión único en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-httpredirect1.png)

1. En **Artifact Endpoints** (Puntos de conexión de artefacto), seleccione **Next** (Siguiente) para continuar.

    ![Opciones de los puntos de conexión de artefacto en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-artifactendpoint.png)

1. En **Authentication Requirements** (Requisitos de autenticación), haga clic en **Finish** (Finalizar).

    ![Opciones de los requisitos de autenticación y la opción de finalizar en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-authentication.png)

1. Vaya a la pestaña **Trusted Provider** (Proveedor de confianza) > **Identity Federation** (Federación de identidades) en la parte inferior de la página. Seleccione **Editar**.

    ![Las pestañas de proveedor de confianza y federación de identidades en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-trustedprovider.png)

1. Seleccione **Agregar**.

    ![La opción Agregar en la pestaña de la federación de identidades](./media/sapfiori-tutorial/tutorial-sapnetweaver-addidentityprovider.png)

1. En el cuadro de diálogo **Supported NameID Formats** (Formatos de NameID admitidos), seleccione **Unspecified** (No especificado). Seleccione **Aceptar**.

    ![El cuadro de diálogo y opciones de los formatos de NameID admitidos en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-nameid.png)

    Los valores **User ID Source** (Origen del identificador de usuario) y **User ID Mapping Mode** (Modo de asignación de identificador de usuario) determinan el vínculo entre el usuario SAP y la notificación de Azure AD.  

    **Escenario 1**: Asignación de usuario de SAP a usuario de Azure AD

    1. En SAP, en **Details of NameID Format "Unspecified"** (Detalles del formato de NameID "Sin especificar"), tome nota de los detalles:

        ![El cuadro de diálogo de los detalles del formato de NameID "Sin especificar" en SAP](./media/sapfiori-tutorial/nameiddetails.png)

    1. En Azure Portal, en **User Attributes & Claims** (Atributos y notificaciones del usuario), tome nota de las notificaciones de Azure AD.

        ![El cuadro de diálogo de atributos y notificaciones del usuario en Azure Portal](./media/sapfiori-tutorial/claimsaad1.png)

    **Escenario 2**: Seleccione el identificador de usuario SAP según la dirección de correo electrónico configurada en SU01. En este caso, el identificador de correo electrónico debe configurarse en SU01 para cada usuario que necesite el inicio de sesión único.

    1.  En SAP, en **Details of NameID Format "Unspecified"** (Detalles del formato de NameID "Sin especificar"), tome nota de los detalles:

        ![El cuadro de diálogo de los detalles del formato de NameID "Sin especificar" en SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-nameiddetails1.png)

    1. En Azure Portal, en **User Attributes & Claims** (Atributos y notificaciones del usuario), tome nota de las notificaciones de Azure AD.

       ![El cuadro de diálogo de atributos y notificaciones del usuario en Azure Portal](./media/sapfiori-tutorial/claimsaad2.png)

1. Seleccione **Save** (Guardar) y, a continuación, **Enable** (Habilitar) para habilitar el proveedor de identidades.

    ![Las opciones de guardar y habilitar en SAP](./media/sapfiori-tutorial/configuration1.png)

1. Cuando se le solicite, seleccione **OK** (Aceptar).

    ![La opción de aceptar en el cuadro de diálogo de configuración de SAML 2.0 en SAP](./media/sapfiori-tutorial/configuration2.png)

### <a name="create-sap-fiori-test-user"></a>Creación de un usuario de prueba en SAP Fiori

En esta sección, creará un usuario llamado Britta Simon en SAP Fiori. Colabore con su equipo interno de expertos en SAP o con el asociado SAP de su organización para agregar el usuario en la plataforma de SAP Fiori.

## <a name="test-sso"></a>Prueba de SSO

1. Una vez activado el proveedor de identidades de Azure AD en SAP Fiori, intente obtener acceso a una de las siguientes direcciones URL para probar el inicio de sesión (no se le debería pedir ni nombre de usuario ni contraseña):

    * https:\//\<sapurl\>/sap/bc/bsp/sap/it00/default.htm
    * https:\//\<sapurl\>/sap/bc/bsp/sap/it00/default.htm

    > [!NOTE]
    > Reemplace *sapurl* con el nombre de host real de SAP.

1. La dirección URL de prueba le debe llevar a la siguiente página de aplicación de prueba en SAP. Si se abre la página es que el inicio de sesión único en Azure AD está correctamente configurado.

    ![La página de aplicación de prueba estándar en SAP](./media/sapfiori-tutorial/testingsso.png)

1. Si se le pide un nombre de usuario y contraseña, habilite el trace de sistema para ayudar con el diagnostico del problema. Use la siguiente dirección URL para el trace: https:\//\<sapurl\>/sap/bc/webdynpro/sap/sec_diag_tool?sap-client=122&sap-language=EN#.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe SAP Fiori con Azure AD](https://aad.portal.azure.com/)
