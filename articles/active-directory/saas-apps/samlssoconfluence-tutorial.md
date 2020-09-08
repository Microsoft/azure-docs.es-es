---
title: 'Tutorial: Integración de Azure Active Directory con SAML SSO for Confluence by resolution GmbH | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAML SSO for Confluence by resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/24/2018
ms.author: jeedes
ms.openlocfilehash: 87c24cf61974c284772aae23e48ffc907792895b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543489"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-confluence-by-resolution-gmbh"></a>Tutorial: Integración de Azure Active Directory con SAML SSO for Confluence by resolution GmbH

En este tutorial, aprenderá a integrar SAML SSO for Confluence by resolution GmbH con Azure Active Directory (Azure AD).
La integración de SAML SSO for Confluence by resolution GmbH con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a SAML SSO for Confluence by resolution GmbH.
* Puede permitir que los usuarios inicien sesión automáticamente en SAML SSO for Confluence by resolution GmbH (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con SAML SSO for Confluence by resolution GmbH, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único de SAML SSO for Confluence by resolution GmbH

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAML SSO for Confluence by resolution GmbH admite el inicio de sesión único iniciado por **SP** e **IDP**

## <a name="adding-saml-sso-for-confluence-by-resolution-gmbh-from-the-gallery"></a>Adición de SAML SSO for Confluence by resolution GmbH desde la galería

Para configurar la integración de SAML SSO for Confluence by resolution GmbH en Azure AD, debe agregar SAML SSO for Confluence by resolution GmbH desde la galería a la lista de aplicaciones administradas de SaaS.

**Para agregar SAML SSO for Confluence by resolution GmbH desde la galería, siga los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SAML SSO for Confluence by resolution GmbH**, seleccione **SAML SSO for Confluence by resolution GmbH** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![SAML SSO for Confluence by resolution GmbH en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, se configura y se prueba el inicio de sesión único de Azure AD con SAML SSO for Confluence by resolution GmbH con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de SAML SSO for Confluence by resolution GmbH.

Para configurar y probar el inicio de sesión único de Azure AD con SAML SSO for Confluence by resolution GmbH, se deben completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de SAML SSO for Confluence by resolution GmbH](#configure-saml-sso-for-confluence-by-resolution-gmbh-single-sign-on)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de SAML SSO for Confluence by resolution GmbH](#create-saml-sso-for-confluence-by-resolution-gmbh-test-user)** : para tener un homólogo de Britta Simon en SAML SSO for Confluence by resolution GmbH vinculado a la representación del usuario de Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SAML SSO for Confluence by resolution GmbH, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **SAML SSO for Confluence by resolution GmbH**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración de SAML básica**, realice los siguientes pasos si desea configurar la aplicación en el modo iniciado por **IDP**:

    ![Dominio y direcciones URL de SAML SSO for Confluence by resolution GmbH: información de inicio de sesión único](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<server-base-url>/plugins/servlet/samlsso`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/samlsso`

    c. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por SP:

    ![Dominio y direcciones URL de SAML SSO for Confluence by resolution GmbH: información de inicio de sesión único](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/samlsso`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de clientes de SAML SSO for Confluence by resolution GmbH](https://www.resolution.de/go/support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="configure-saml-sso-for-confluence-by-resolution-gmbh-single-sign-on"></a>Configuración del inicio de sesión único de SAML SSO for Confluence by resolution GmbH

1. En otra ventana del explorador web, inicie sesión en el **Portal de administración de SAML SSO for Confluence by resolution GmbH** como administrador.

2. Mantenga el mouse encima del icono de engranaje y haga clic en **Complementos**.
    
    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon1.png)

3. Se le redirigirá a la página de acceso de administrador. Escriba la contraseña y haga clic en el botón **Confirmar**.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon2.png)

4. En la pestaña **ATLASSIAN MARKETPLACE** (MARKETPLACE DE ATLASSIAN), haga clic en **Find new add-ons** (Buscar nuevos complementos). 

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon.png)

5. Busque **SAML Single Sign On (SSO) for Confluence** y haga clic en el botón **Instalar** para instalar el nuevo complemento SAML.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon7.png)

6. Se iniciará la instalación del complemento. Haga clic en **Cerrar**.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon8.png)

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon9.png)

7.  Haga clic en **Administrar**.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon10.png)
    
8. Haga clic en **Configurar** para configurar el nuevo complemento.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon11.png)

9. Este nuevo complemento también puede encontrarse en la pestaña **USERS & SECURITY** (USUARIOS Y SEGURIDAD).

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon3.png)
    
10. En la página **SAML SingleSignOn Plugin Configuration** (Configuración del complemento SingleSignOn de SAML), haga clic en el botón **Add new IdP** (Agregar IdP nuevo) para configurar los valores del proveedor de identidades.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon4.png)

11. En la página **Choose your SAML Identity Provider** (Elegir el proveedor de identidades SAML), siga estos pasos:

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon5a.png)
 
    a. Establezca **Azure AD** como el tipo de IdP.
    
    b. Agregue el **nombre** del proveedor de identidades (p. ej., Azure AD).
    
    c. Agregue la **descripción** del proveedor de identidades (p. ej., Azure AD).
    
    d. Haga clic en **Next**.
    
12. En la página **Identity provider configuration** (Configuración del proveedor de identidades), haga clic en el botón **Siguiente**.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon5b.png)

13. En la página **Import SAML IdP Metadata** (Importar los metadatos del IdP de SAML), siga los pasos siguientes:

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon5c.png)

    a. Haga clic en el botón **Cargar archivo** y seleccione el archivo XML de metadatos que descargó en el paso 5.

    b. Haga clic en el botón **Importar**.
    
    c. Espere un momento hasta que la importación se realice correctamente.
    
    d. A continuación, haga clic en el botón **Siguiente**.
    
14. En la página **User ID attribute and transformation** (Transformación y atributo del id. de usuario), haga clic en el botón **Siguiente**.

    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon5d.png)
    
15. En la página **User creation and update** (Creación y actualización de usuarios), haga clic en **Guardar y siguiente** para guardar la configuración.   
    
    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon6a.png)
    
16. En la página **Test your settings** (Probar su configuración), haga clic en **Skip test & configure manually** (Omitir la prueba y configurar manualmente) para omitir la prueba de usuario por ahora. Esto se realizará en la sección siguiente y requiere la configuración de algunos valores en Azure Portal. 
    
    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon6b.png)
    
17. En el cuadro que aparece con el texto **Skipping the test means...** (Omitir la prueba significa...), haga clic en **OK** (Aceptar).
    
    ![Configurar inicio de sesión único](./media/samlssoconfluence-tutorial/addon6c.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a SAML SSO for Confluence by resolution GmbH.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **SAML SSO for Confluence by resolution GmbH**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **SAML SSO for Confluence by resolution GmbH**.

    ![Vínculo de SAML SSO for Confluence by resolution GmbH en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-saml-sso-for-confluence-by-resolution-gmbh-test-user"></a>Creación del usuario de prueba de SAML SSO for Confluence by resolution GmbH

Para permitir que los usuarios de Azure AD inicien sesión en SAML SSO for Confluence by resolution GmbH, deben estar aprovisionados en SAML SSO for Confluence by resolution GmbH.  
En SAML SSO for Confluence by resolution GmbH, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en el sitio de la empresa de SAML SSO for Confluence by resolution GmbH como administrador.

2. Mantenga el mouse encima del icono de engranaje y haga clic en **Administración de usuarios**.

    ![Agregar empleado](./media/samlssoconfluence-tutorial/user1.png) 

3. En la sección Usuarios, haga clic en la pestaña **Agregar usuarios**. En la página del cuadro de diálogo **Agregar un usuario**, realice los siguientes pasos:

    ![Agregar empleado](./media/samlssoconfluence-tutorial/user2.png) 

    a. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, Britta Simon.

    b. En el cuadro de texto **Nombre completo**, escriba el nombre completo de un usuario, por ejemplo, Britta Simon.

    c. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    d. En el cuadro de texto **Contraseña**, escriba la contraseña de Britta Simon.

    e. Haga clic en **Confirmar contraseña** y vuelva a escribir la contraseña.
    
    f. Haga clic en el botón **Agregar**.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SAML SSO for Confluence by resolution GmbH en el panel de acceso, debería iniciar sesión automáticamente en la aplicación SAML SSO for Confluence by resolution GmbH para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

