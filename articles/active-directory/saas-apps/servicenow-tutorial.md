---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ServiceNow | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ServiceNow.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: celested
ms.assetid: a5a1a264-7497-47e7-b129-a1b5b1ebff5b
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 06/19/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: a09cc83259a6fbb07b8c88e1d8f82b720400c0d1
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86507750"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-servicenow"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ServiceNow

En este tutorial, aprenderá a integrar ServiceNow con Azure Active Directory (Azure AD). Al integrar ServiceNow con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a ServiceNow.
* Permitir que los usuarios inicien sesión automáticamente en ServiceNow con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de ServiceNow con inicio de sesión único (SSO) habilitado.
* Para ServiceNow, una instancia o inquilino de ServiceNow, la versión Calgary o posterior.
* Para ServiceNow Express, una instancia de ServiceNow Express, versión Helsinki o posterior.
* El inquilino de ServiceNow debe tener habilitado el [complemento de inicio de sesión único en varios proveedores](https://wiki.servicenow.com/index.php?title=Multiple_Provider_Single_Sign-On#gsc.tab=0). Puede hacer esto si [envía una solicitud de servicio](https://hi.service-now.com).
* Para la configuración automática, habilite el complemento de varios proveedores para ServiceNow.
* Para instalar la aplicación ServiceNow Classic (móvil), vaya a la tienda de aplicaciones adecuada y busque la aplicación ServiceNow Classic. A continuación, descárguela.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 

* ServiceNow admite el inicio de sesión único iniciado por **SP**.

* ServiceNow admite el [aprovisionamiento de usuarios automatizado](servicenow-provisioning-tutorial.md).

* Una vez configurado ServiceNow, puede aplicar controles de sesión, que protegen la filtración y la infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

* Puede configurar la aplicación de ServiceNow Classic (móvil) con Azure AD para habilitar el inicio de sesión único. Admite tanto a usuarios de iOS como de Android. En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

## <a name="add-servicenow-from-the-gallery"></a>Adición de ServiceNow desde la galería

Para configurar la integración de ServiceNow en Azure AD, es preciso agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante una cuenta profesional o educativa, o bien una cuenta personal de Microsoft.
1. En el panel izquierdo, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ServiceNow** en el cuadro de búsqueda.
1. Seleccione **ServiceNow** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-servicenow"></a>Configuración y prueba del inicio de sesión único de Azure AD para ServiceNow

Configure y pruebe el inicio de sesión único de Azure AD con ServiceNow mediante un usuario de prueba llamado **B. Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de ServiceNow.

Para configurar y probar el inicio de sesión único de Azure AD con ServiceNow, es preciso completar los siguientes bloques de creación:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para permitir que los usuarios puedan utilizar esta característica.
    1. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con B. Simon.
    1. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para que B. Simon pueda usar el inicio de sesión único de Azure AD.
    1. [Configuración del inicio de sesión único de Azure AD para ServiceNow Express](#configure-azure-ad-sso-for-servicenow-express), para que los usuarios puedan usar esta característica.
2. [Configuración de ServiceNow](#configure-servicenow), para configurar el inicio de sesión único en la aplicación.
    1. [Creación de un usuario de prueba en ServiceNow](#create-servicenow-test-user), para tener un homólogo de B. Simon en ServiceNow que esté vinculado a la representación de ella en Azure AD.
    1. [Configuración del inicio de sesión único en ServiceNow Express](#configure-servicenow-express-sso), para configurar los valores de inicio de sesión único en la aplicación.  
3. [Comprobación del inicio de sesión único](#test-sso), para verificar que la configuración funciona correctamente.
4. [Prueba del inicio de sesión único para ServiceNow Classic (móvil)](#test-sso-for-servicenow-classic-mobile), para comprobar si funciona la configuración.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ServiceNow**, busque la sección **Administrar**. Seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Captura de pantalla de la página Configurar el inicio de sesión único con SAML, con el icono de lápiz resaltado](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://instance.service-now.com/login_with_sso.do?glide_sso_id=<sys_id of the sso configuration>`

    b. En **Identificador (Id. de entidad)** , escriba una dirección URL con el siguiente formato: `https://<instance-name>.service-now.com`

    c. En **URL de respuesta**, escriba una de las siguientes direcciones URL:

    | URL de respuesta|
    |----------|
    | `https://instancename.service-now.com/navpage.do` |
    | `https://instancename.service-now.com/customer.do` | 

    > [!NOTE]
    > Estos valores no son reales. Tiene que actualizar estos valores con la dirección URL de inicio de sesión y el identificador reales, que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** . 

   ![Captura de pantalla de la sección Certificado de firma de SAML, con la opción Descargar resaltada](common/certificatebase64.png)

   a. Seleccione el botón Copiar para copiar la **dirección URL de los metadatos de federación de la aplicación** y péguela en el Bloc de notas. Esta dirección URL se utilizará más adelante en el tutorial.

    b. Seleccione **Descargar** para descargar el **Certificado (Base64)** y, luego, guarde el archivo de certificado en el equipo.

1. En la sección **Configurar ServiceNow**, copie las direcciones URL que necesite.

   ![Captura de pantalla de la sección Configurar ServiceNow con las direcciones URL resaltadas](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba, llamado B. Simon, en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En **Nombre**, escriba `B.Simon`.  
   1. En **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B. Simon acceda a ServiceNow mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ServiceNow**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Captura de pantalla de la sección Administrar, con la opción Usuarios y grupos resaltada](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Captura de pantalla Usuarios y grupos, con Agregar usuario resaltado](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B. Simon** en la lista de usuarios y, después, elija **Seleccionar**.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija **Seleccionar**.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="configure-azure-ad-sso-for-servicenow-express"></a>Configuración de inicio de sesión único de Azure AD para ServiceNow Express

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ServiceNow**, haga clic en **Inicio de sesión único**.

    ![Captura de pantalla de la página de integración de aplicaciones de ServiceNow, con Inicio de sesión único resaltado](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Captura de pantalla Seleccione un método de inicio de sesión único con SAML resaltado](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Captura de pantalla de la página Configurar el inicio de sesión único con SAML, con el icono de lápiz resaltado](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://instance.service-now.com/login_with_sso.do?glide_sso_id=<sys_id of the sso configuration>`

    b. En **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente formato: `https://<instance-name>.service-now.com`

    c. En **URL de respuesta**, escriba una de las siguientes direcciones URL:

    | URL de respuesta |
    |-----------|
    | `https://instancename.service-now.com/navpage.do` |
    | `https://instancename.service-now.com/customer.do` |

    > [!NOTE]
    > Estos valores no son reales. Tiene que actualizar estos valores con la dirección URL de inicio de sesión y el identificador reales, que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **Certificado (Base64)** de las opciones especificadas según sus requisitos. Guárdelo en el equipo.

    ![Captura de pantalla de la sección Certificado de firma de SAML, con la opción Descargar resaltada](common/certificatebase64.png)

6. Puede hacer que Azure AD configure automáticamente ServiceNow para la autenticación basada en SAML. Para habilitar este servicio, vaya a la sección **Configurar ServiceNow** y haga clic en **Ver instrucciones detalladas** para abrir la ventana **Configurar inicio de sesión**.

    ![Captura de pantalla de la sección Configurar ServiceNow, con las instrucciones paso a paso resaltadas](./media/servicenow-tutorial/tutorial_servicenow_configure.png)

7. En el formulario **Configurar inicio de sesión**, escriba el nombre de la instancia de ServiceNow, el nombre de usuario y la contraseña de administrador. Seleccione **Configurar ahora**. El nombre de usuario administrador proporcionado debe tener asignado el rol **security_admin** en ServiceNow para que esto funcione. De lo contrario, para configurar manualmente ServiceNow de modo que se use Azure AD como proveedor de identidades de SAML, seleccione **Configurar manualmente el inicio de sesión único**. Copie la **dirección URL de cierre de sesión, el identificador de Azure AD y la dirección URL de inicio de sesión** de la sección Referencia rápida.

    ![Captura de pantalla del formulario de configuración de inicio de sesión con Configurar ahora resaltado](./media/servicenow-tutorial/configure.png "Configurar dirección URL de la aplicación")

## <a name="configure-servicenow"></a>Configuración de ServiceNow

1. Inicie sesión en la aplicación ServiceNow como administrador.

1. Active el complemento **Integration - Multiple Provider Single Sign-On Installer** (Integración: instalador del inicio de sesión único multiproveedor), para lo cual debe seguir estos pasos:

    a. En el panel izquierdo, busque la sección **Definition System** (Definición del sistema) en el cuadro de búsqueda y, después, seleccione **Plugins** (Complementos).

    ![Captura de pantalla de la sección System Definition (Definición del sistema), con la sección System Definition and Plugins (Definición del sistema y complementos) resaltada](./media/servicenow-tutorial/tutorial_servicenow_03.png "Activar complemento")

    b. Busque **Integration - Multiple Provider Single Sign-On Installer**.

     ![Captura de pantalla de la página System Plugins (Complementos del sistema), con Integration - Multiple Provider Single Sign-On Installer (Integración: instalador del inicio de sesión único multiproveedor) resaltado](./media/servicenow-tutorial/tutorial_servicenow_04.png "Activar complemento")

    c. Seleccione el complemento. Haga clic con el botón derecho y seleccione **Activate/Upgrade** (Activar o actualizar).

     ![Captura de pantalla del menú contextual del complemento, con Active/Upgrade (Activar/Actualizar) resaltado](./media/servicenow-tutorial/tutorial_activate.png "Activar complemento")

    d. Seleccione **Activar**.

     ![Captura de pantalla del cuadro de diálogo Activate Plugin (Activar complemento) resaltado](./media/servicenow-tutorial/tutorial_activate1.png "Activar complemento")

1. En el panel izquierdo, busque la sección **Multi-Provider SSO** (Inicio de sesión único multiproveedor) en la barra de búsqueda; después, seleccione **Properties** (Propiedades).

    ![Captura de pantalla de la sección Multi-Provider SSO (Inicio de sesión único multiproveedor) y Properties (Propiedades) resaltados](./media/servicenow-tutorial/tutorial_servicenow_06.png "Configurar dirección URL de la aplicación")

1. En el cuadro de diálogo **Multiple Provider SSO Properties** (Propiedades de inicio de sesión único multiproveedor), siga estos pasos:

    ![Captura de pantalla del cuadro de diálogo Multiple Provider SSO Properties (Propiedades de inicio de sesión único multiproveedor)](./media/servicenow-tutorial/ic7694981.png "Configurar dirección URL de la aplicación")

    * En **Enable multiple provider SSO** (Habilitar inicio de sesión único multiproveedor), seleccione **Yes** (Sí).
  
    * En **Enable Auto Importing of users from all identity providers into the user table** (Habilitar la importación automática de usuarios de todos los proveedores de identidades en la tabla de usuarios), seleccione **Yes** (Sí).

    * En **Enable debug logging for the multiple provider SSO integration** (Habilitar el registro de depuración para la integración de SSO multiproveedor), seleccione **Yes** (Sí).

    * En **The field on the user table that...** (El campo en la tabla de usuario que...), escriba **email** (correo electrónico).
  
    * Seleccione **Guardar**.

1. Puede configurar ServiceNow de forma automática o manual. Para configurar ServiceNow automáticamente, siga estos pasos:

    1. Vuelva a la página de inicio de sesión único de **ServiceNow** en Azure Portal.

    1. Se proporciona un servicio de configuración con un solo clic para ServiceNow. Para habilitar este servicio, vaya a la sección **Configuración de ServiceNow** y seleccione **Configurar ServiceNow** para abrir la ventana **Configurar inicio de sesión**.

        ![Captura de pantalla de la sección Configuración de ServiceNow, con las instrucciones paso a paso resaltadas](./media/servicenow-tutorial/tutorial_servicenow_configure.png)

    1. En el formulario **Configurar inicio de sesión**, escriba el nombre de la instancia de ServiceNow, el nombre de usuario y la contraseña de administrador. Seleccione **Configurar ahora**. El nombre de usuario administrador proporcionado debe tener asignado el rol **security_admin** en ServiceNow para que esto funcione. De lo contrario, para configurar manualmente ServiceNow de modo que se use Azure AD como proveedor de identidades de SAML, seleccione **Configurar manualmente el inicio de sesión único**. Copie la **URL del servicio de inicio de sesión único de SAML, el identificador de entidad de SAML y la dirección URL de cierre de sesión de SAML** de la sección Referencia rápida.

        ![Captura de pantalla del formulario de configuración de inicio de sesión con Configurar ahora resaltado](./media/servicenow-tutorial/configure.png "Configurar dirección URL de la aplicación")

    1. Inicie sesión en la aplicación ServiceNow como administrador.

       * En la configuración automática se configuran todas las opciones necesarias de **ServiceNow**, pero el **certificado X.509** no está habilitado de forma predeterminada. Tiene que asignarlo manualmente al proveedor de identidades en ServiceNow. Siga estos pasos:

         1. En el panel izquierdo, busque la sección **Multi-Provider SSO** (Inicio de sesión único multiproveedor) en la barra de búsqueda y, después, seleccione **Identity Providers** (Proveedores de identidades).

            ![Captura de pantalla de la sección Multi-Provider SSO (Inicio de sesión único multiproveedor), con Identity Providers (Proveedores de identidades) resaltado](./media/servicenow-tutorial/tutorial_servicenow_07.png "Configurar inicio de sesión único")

         1. Seleccione el proveedor de identidades generado automáticamente.

            ![Captura de pantalla de los proveedores de identidades, con el proveedor de identidad generado resaltado](./media/servicenow-tutorial/tutorial_servicenow_08.png "Configurar inicio de sesión único")

         1.  En la sección **Proveedor de identidades**, realice los siguientes pasos:

             ![Captura de pantalla de la sección Identity Provider (Proveedor de identidades)](./media/servicenow-tutorial/automatic_config.png "Configurar inicio de sesión único")

               * En **Name** (Nombre), escriba el nombre de la configuración, por ejemplo, **Microsoft Azure Federated single sign-on** (Inicio de sesión único federado de Microsoft Azure).

               * Quite el valor rellenado de **Identity Provider's SingleLogoutRequest** (Solicitud de cierre de sesión único del proveedor de identidades) del cuadro de texto.

               * Copie el valor de **ServiceNow Homepage** (Página principal de ServiceNow) y péguelo en **Dirección URL de inicio de sesión**, en la sección **Configuración básica de SAML de ServiceNow** de Azure Portal.

                  > [!NOTE]
                  > La página principal de la instancia de ServiceNow es una concatenación de su **URL de inquilino de ServiceNow** y **/navpage.do** (por ejemplo: `https://fabrikam.service-now.com/navpage.do`).

              * Copie el valor de **Entity ID / Issuer** (Identificador de entidad/emisor) y péguelo en **Identifier** (Identificador), en la sección **Configuración básica de SAML de ServiceNow** de Azure Portal.

              * Confirme que el valor de **NameID Policy** (Directiva de id. de nombre) esté establecido en `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`. 

         1. Desplácese hacia abajo a la sección **Certificado X.509** y seleccione **Editar**.

             ![Captura de pantalla de la sección Certificado X.509, con la opción Edición resaltada](./media/servicenow-tutorial/tutorial_servicenow_09.png "Configurar inicio de sesión único")

         1. Seleccione el certificado y el icono de flecha derecha para agregar el certificado.

            ![Captura de pantalla de Colección, con el certificado y el icono de flecha derecha resaltados](./media/servicenow-tutorial/tutorial_servicenow_11.png "Configurar inicio de sesión único")

          1. Seleccione **Guardar**.

          1. En la esquina superior derecha de la página, seleccione **Test Connection** (Probar conexión).

             ![Captura de pantalla de la página con Test Connection (Probar conexión) resaltado](./media/servicenow-tutorial/tutorial_activate2.png "Activar complemento")

             > [!NOTE]
             > Si se produce un error en la prueba de conexión y no puede activar esta conexión, ServiceNow ofrece el modificador de invalidación. Tiene que escribir **Sys_properties.LIST** en el cuadro de **navegación por búsqueda** y se abrirá la nueva página de propiedades del sistema. Aquí debe crear una nueva propiedad con un nombre como **glide.authenticate.multisso.test.connection.mandatory** con **dataType** como **True/False** y establecer el **valor** en **False**.

             > ![Captura de pantalla de la página Resultados de pruebas](./media/servicenow-tutorial/testconnection-fail.png "Configurar inicio de sesión único")
        
          1. Cuando se le pidan sus credenciales, escríbalas. Verá la página siguiente. Se espera el error **SSO Logout Test Results** (Resultados de la prueba del cierre de sesión de SSO). Omita el error y seleccione **Activate** (Activar).

             ![Captura de pantalla de la página Resultados de pruebas](./media/servicenow-tutorial/servicenowactivate.png "Configurar inicio de sesión único")
  
1. Para configurar **ServiceNow** manualmente, siga estos pasos:

    1. Inicie sesión en la aplicación ServiceNow como administrador.

    1. En el panel izquierdo, seleccione **Identity Providers** (Proveedores de identidades).

        ![Captura de pantalla de Multi-Provider SSO (Inicio de sesión único multiproveedor), con Identity Providers (Proveedores de identidades) resaltado](./media/servicenow-tutorial/tutorial_servicenow_07.png "Configurar inicio de sesión único")

    1. En el cuadro de diálogo **Identity Providers** (Proveedores de identidades), haga clic en **New** (Nuevo).

        ![Captura de pantalla de Identity Providers (Proveedores de identidades) con New (Nuevo) resaltado](./media/servicenow-tutorial/ic7694977.png "Configurar inicio de sesión único")

    1. En el cuadro de diálogo **Identity Providers** (Proveedores de identidades), seleccione **SAML**.

        ![Captura de pantalla de Identity Providers (Proveedores de identidades) con SAML resaltado](./media/servicenow-tutorial/ic7694978.png "Configurar inicio de sesión único")

    1. En **Import Identity Provider Metadata** (Importar metadatos del proveedor de identidades), realice los siguientes pasos:

        ![Captura de pantalla del cuadro de diálogo Identity Providers Metadata (Metadatos de proveedores de identidades) con URL e Import (Importar) resaltados](./media/servicenow-tutorial/idp.png "Configurar inicio de sesión único")

        1. Escriba el valor de **App Federation Metadata Url** (URL de metadatos de federación de la aplicación) que haya copiado de Azure Portal.

        1. Seleccione **Import** (Importar).

    1. Se lee la dirección URL de los metadatos de IdP y se rellena toda la información de los campos.

        ![Captura de pantalla de Identity Provider (Proveedor de identidades)](./media/servicenow-tutorial/ic7694982.png "Configurar inicio de sesión único")

        * En **Name** (Nombre), escriba el nombre de la configuración, por ejemplo, **Microsoft Azure Federated single sign-on** (Inicio de sesión único federado de Microsoft Azure).

        * Quite el valor rellenado en **Identity Provider's SingleLogoutRequest** (Solicitud de cierre de sesión único del proveedor de identidades) del cuadro de texto.

        * Copie el valor de **ServiceNow Homepage** (Página principal de ServiceNow). Péguelo en **URL de inicio de sesión** en la sección **Configuración de SAML básica de ServiceNow** de Azure Portal.

            > [!NOTE]
            > La página principal de la instancia de ServiceNow es una concatenación de su **URL de inquilino de ServiceNow** y **/navpage.do** (por ejemplo: `https://fabrikam.service-now.com/navpage.do`).

        * Copie el valor de **Entity ID / Issuer** (Id. de entidad/emisor). Péguelo en **Identificador**, en la sección **Configuración de SAML básica de ServiceNow** de Azure Portal.

        * Confirme que el valor de **NameID Policy** (Directiva de id. de nombre) esté establecido en `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`.

        * Seleccione **Advanced** (Avanzadas). En el **campo Usuario**, escriba **email** (correo electrónico).

            > [!NOTE]
            > Puede configurar Azure AD para emitir el identificador de usuario de Azure AD (nombre principal de usuario) o la dirección de correo electrónico como identificador único en el token SAML. Para ello, vaya a la sección **ServiceNow** > **Attributes** > **Single sign-on** (ServiceNow>Atributos>Inicio de sesión único) de Azure Portal y asigne el campo deseado al atributo **nameidentifier** (identificador de nombre). El valor almacenado para el atributo seleccionado en Azure AD (por ejemplo, el nombre de usuario principal) debe coincidir con el valor almacenado en ServiceNow para el campo especificado (por ejemplo, user_name).

        * Seleccione **Test Connection** (Probar conexión) en la esquina superior derecha de la página.

          > [!NOTE]
          > Si se produce un error en la prueba de conexión y no puede activar esta conexión, ServiceNow ofrece el modificador de invalidación. Tiene que escribir **Sys_properties.LIST** en el cuadro de **navegación por búsqueda** y se abrirá la nueva página de propiedades del sistema. Aquí debe crear una nueva propiedad con un nombre como **glide.authenticate.multisso.test.connection.mandatory** con **dataType** como **True/False** y establecer el **valor** en **False**.

          > ![Captura de pantalla de la página Resultados de pruebas](./media/servicenow-tutorial/testconnection-fail.png "Configurar inicio de sesión único")

        * Cuando se le pidan sus credenciales, escríbalas. Verá la página siguiente. Se espera el error **SSO Logout Test Results** (Resultados de la prueba del cierre de sesión de SSO). Omita el error y seleccione **Activate** (Activar).

          ![Captura de pantalla de la página Resultados de pruebas](./media/servicenow-tutorial/servicenowactivate.png "Configurar inicio de sesión único")

### <a name="create-servicenow-test-user"></a>Creación de un usuario de prueba en ServiceNow

El objetivo de esta sección es crear una usuaria de prueba llamada B. Simon en ServiceNow. ServiceNow admite el aprovisionamiento automático de usuarios, que está habilitado de forma predeterminada.

> [!NOTE]
> Si necesita crear un usuario manualmente, póngase en contacto con el [equipo de soporte técnico de ServiceNow](https://www.servicenow.com/support/contact-support.html).

### <a name="configure-servicenow-express-sso"></a>Configuración del inicio de sesión único de ServiceNow Express

1. Inicie sesión en la aplicación ServiceNow Express como administrador.

2. En el panel izquierdo, haga clic en **Single Sign-On** (Inicio de sesión único).

    ![Captura de pantalla de la aplicación ServiceNow Express con Single Sign-On (Inicio de sesión único) resaltado](./media/servicenow-tutorial/ic7694980ex.png "Configurar dirección URL de la aplicación")

3. En el cuadro de diálogo **Single Sign-On** (Inicio de sesión único), seleccione el icono de configuración en la parte superior derecha y establezca las siguientes propiedades:

    ![Captura de pantalla del cuadro de diálogo Inicio de sesión único](./media/servicenow-tutorial/ic7694981ex.png "Configurar dirección URL de la aplicación")

    a. Cambie **Enable multiple provider SSO** (Habilitar SSO de varios proveedores), hacia la derecha.

    b. Cambie **Enable debug logging got the multiple provider SSO integration** (Habilitar registro de depuración para integración de SSO de varios proveedores) hacia la derecha.

    c. En **The field on the user table that...** (El campo en la tabla de usuario que...), escriba **user_name** (nombre de usuario).

4. En el cuadro de diálogo **Single Sign-On** (Inicio de sesión único), seleccione **Add New Certificate** (Agregar nuevo certificado).

    ![Captura de pantalla del cuadro de diálogo Inicio de sesión único con Add New Certificate (Agregar nuevo certificado) resaltado](./media/servicenow-tutorial/ic7694973ex.png "Configurar inicio de sesión único")

5. En el cuadro de diálogo **X.509 Certificates** (Certificados X.509), siga estos pasos:

    ![Captura de pantalla del cuadro de diálogo X.509 Certificates (Certificados X.509)](./media/servicenow-tutorial/ic7694975.png "Configurar inicio de sesión único")

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración (por ejemplo, **TestSAML2.0**).

    b. Seleccione **Active**(Activo).

    c. En **Format** (Formato), seleccione **PEM**.

    d. En **Type** (Tipo), seleccione **Trust Store Cert** (Confiar en certificados de almacén).

    e. Abra en el Bloc de notas su certificado codificado en Base64 que ha descargado de Azure Portal. Copie el contenido en el Portapapeles y, después, péguelo en el cuadro de texto **PEM Certificate** (Certificado PEM).

    f. Seleccione **Update** (Actualizar).

6. En el cuadro de diálogo **Single Sign-On** (Inicio de sesión único), seleccione **Add New IdP** (Agregar nuevo IdP).

    ![Captura de pantalla del cuadro de diálogo Inicio de sesión único con Add New IdP (Agregar nuevo proveedor de identidades) resaltado](./media/servicenow-tutorial/ic7694976ex.png "Configurar inicio de sesión único")

7. En el cuadro de diálogo **Add New Identity Provider** (Agregar nuevo proveedor de identidades), en **Configure Identity Provider** (Configurar proveedor de identidades), siga estos pasos:

    ![Captura de pantalla del cuadro de diálogo Add New Identity Provider (Agregar nuevo proveedor de identidades)](./media/servicenow-tutorial/ic7694982ex.png "Configurar inicio de sesión único")

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración (por ejemplo, **SAML 2.0**).

    b. En **Identity provider URL** (Dirección URL del proveedor de identidades), pegue el valor del identificador del proveedor de identidades que ha copiado de Azure Portal.

    c. En **Identity provider AuthnRequest** (Solicitud de autenticación del proveedor de identidades), pegue la dirección URL de la solicitud de autenticación que ha copiado de Azure Portal.

    d. En **Identity provider SingleLogoutRequest** (Solicitud de cierre de sesión única del proveedor de identidades), pegue la dirección URL de cierre de sesión que ha copiado de Azure Portal.

    e. Como **Identity Provider Certificate** (Certificado del proveedor de identidades), seleccione el certificado que ha creado en el paso anterior.

8. Seleccione **Advanced Settings** (Configuración avanzada). En **Additional Identity Provider Properties** (Propiedades adicionales del proveedor de identidades), siga estos pasos:

    ![Captura de pantalla de Add New Identity Provider (Agregar nuevo proveedor de identidades) con Advanced Settings (Configuración avanzada) resaltado](./media/servicenow-tutorial/ic7694983ex.png "Configurar inicio de sesión único")

    a. En **Protocol Binding for the IDP's SingleLogoutRequest** (Enlace de protocolo para la solicitud de cierre de sesión único del proveedor de identidades), escriba **urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect**.

    b. En el cuadro de texto **NameID Policy** (Directiva de identificador de nombre), escriba **urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified**.

    c. En **AuthnContextClassRef Method** (Método AuthnContextClassRef), escriba `http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/password`.

    d. En **Create an AuthnContextClass** (Crear AuthnContextClass), cambie a off (desactivado).

9. En **Additional Identity Provider Properties** (Propiedades adicionales del proveedor de identidades), siga estos pasos:

    ![Captura de pantalla de Add New Identity Provider (Agregar nuevo proveedor de identidades) con diversas propiedades resaltadas](./media/servicenow-tutorial/ic7694984ex.png "Configurar inicio de sesión único")

    a. En el cuadro de texto **ServiceNow Homepage** (Página principal de ServiceNow), escriba la dirección URL de la página principal de la instancia de ServiceNow.

    > [!NOTE]
    > La página principal de la instancia de ServiceNow es una concatenación de su **URL de inquilino de ServiceNow** y **/navpage.do** (por ejemplo: `https://fabrikam.service-now.com/navpage.do`).

    b. En **Entity ID / Issuer** (Id. de entidad/emisor), escriba la dirección URL de su inquilino de ServiceNow.

    c. En **Audience URI** (URI de audiencia), escriba la dirección URL de su inquilino de ServiceNow.

    d. En **Clock Skew** (Sesgo de reloj), escriba **60**.

    e. En el **campo Usuario**, escriba **email** (correo electrónico).

    > [!NOTE]
    > Puede configurar Azure AD para emitir el identificador de usuario de Azure AD (nombre principal de usuario) o la dirección de correo electrónico como identificador único en el token SAML. Para ello, vaya a la sección **ServiceNow** > **Attributes** > **Single sign-on** (ServiceNow>Atributos>Inicio de sesión único) de Azure Portal y asigne el campo deseado al atributo **nameidentifier** (identificador de nombre). El valor almacenado para el atributo seleccionado en Azure AD (por ejemplo, el nombre de usuario principal) debe coincidir con el valor almacenado en ServiceNow para el campo especificado (por ejemplo, user_name).

    f. Seleccione **Guardar**.

## <a name="test-sso"></a>Prueba de SSO

Al seleccionar en el icono de ServiceNow del panel de acceso, debería iniciar sesión automáticamente en la versión de ServiceNow para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="test-sso-for-servicenow-classic-mobile"></a>Prueba del inicio de sesión único para ServiceNow Classic (móvil)

1. Abra la aplicación **ServiceNow Classic (móvil)** y realice los pasos siguientes:

    a. Seleccione el signo más en la esquina inferior derecha.

    ![Captura de pantalla de la aplicación ServiceNow Classic con el signo más resaltado](./media/servicenow-tutorial/test03.png)

    b. Escriba el nombre de la instancia de ServiceNow y haga clic en **Continue** (Continuar).

    ![Captura de pantalla de la página Add Instance (Agregar instancia) con Continue (Continuar) resaltado](./media/servicenow-tutorial/test04.png)

    c. En la página **Log in** (Iniciar sesión), siga estos pasos:

    ![Captura de pantalla de la página Login (Iniciar sesión), con Use external login (Usar inicio de sesión externo) resaltado.](./media/servicenow-tutorial/test01.png)

    *  Escriba el valor de **Username** (Nombre de usuario), por ejemplo B.simon@contoso.com.

    *  Seleccione **USE EXTERNAL LOGIN** (USAR INICIO DE SESIÓN EXTERNO). Se le redirigirá a la página de Azure AD para iniciar sesión.

    *  Escriba sus credenciales. Si hay alguna autenticación de terceros o cualquier otra característica de seguridad habilitada, el usuario debe responder como corresponda. Aparece la **página principal** de la aplicación.

        ![Captura de pantalla de la página principal de la aplicación](./media/servicenow-tutorial/test02.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Configuración del aprovisionamiento de usuarios](servicenow-provisioning-tutorial.md)

- [Pruebe ServiceNow con Azure AD](https://aad.portal.azure.com)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/protect-servicenow)

- [Protección de ServiceNow con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
