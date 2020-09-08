---
title: 'Tutorial: integrar Azure Active Directory con Envi MMIS | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Envi MMIS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/06/2019
ms.author: jeedes
ms.openlocfilehash: aababc6ffd84540b99ba68f6bbb449d2aa78ed4b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551751"
---
# <a name="tutorial-azure-active-directory-integration-with-envi-mmis"></a>Tutorial: integrar Azure Active Directory con Envi MMIS

En este tutorial, obtendrá información sobre cómo integrar Envi MMIS en Azure Active Directory (Azure AD).
La integración de Envi MMIS con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Envi MMIS.
* Puede permitir que los usuarios inicien sesión automáticamente en Envi MMIS (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Envi MMIS, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Envi MMIS

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Envi MMIS admite el inicio de sesión único iniciado por **SP** e **IDP**.

## <a name="adding-envi-mmis-from-the-gallery"></a>Agregar Envi MMIS desde la galería

Para configurar la integración de Envi MMIS en Azure AD, será preciso que agregue Envi MMIS desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Envi MMIS desde la galería, siga los pasos a continuación:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Envi MMIS**, seleccione **Envi MMIS** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Envi MMIS en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Envi MMIS con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Envi MMIS.

Para configurar y probar el inicio de sesión único de Azure AD con Envi MMIS, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Envi MMIS](#configure-envi-mmis-single-sign-on)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Envi MMIS](#create-envi-mmis-test-user)** : para tener un homólogo de Britta Simon en Envi MMIS que esté vinculado a la representación de ella en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Envi MMIS, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Envi MMIS**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Set-up Single Sign-On with SAML** (Configurar inicio de sesión único con SAML), haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Envi MMIS](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://www.<CUSTOMER DOMAIN>.com/Account`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.<CUSTOMER DOMAIN>.com/Account/Acs`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de Envi MMIS](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.<CUSTOMER DOMAIN>.com/Account`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de cliente de Envi MMIS](mailto:support@ioscorp.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up Envi MMIS** (Configurar Envi MMIS), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-envi-mmis-single-sign-on"></a>Configuración del inicio de sesión único en Envi MMIS

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de Envi MMIS.

2. Haga clic en la pestaña **My Domain** (Mi dominio).

    ![Botón Configurar inicio de sesión único](./media/envimmis-tutorial/configure1.png)

3. Haga clic en **Editar**.

    ![Botón Configurar inicio de sesión único](./media/envimmis-tutorial/configure2.png)

4. Seleccione la casilla **Use remote authentication** (Usar autenticación remota) y, a continuación, seleccione **Redirección HTTP** (Redirección HTTP) desde la lista desplegable **Tipo de autenticación**.

    ![Botón Configurar inicio de sesión único](./media/envimmis-tutorial/configure3.png)

5. Seleccione la pestaña **Recursos** y, a continuación, haga clic en **Cargar metadatos**.

    ![Botón Configurar inicio de sesión único](./media/envimmis-tutorial/configure4.png)

6. En la ventana emergente **Cargar metadatos**, realice los siguientes pasos:

    ![Botón Configurar inicio de sesión único](./media/envimmis-tutorial/configure5.png)

    a. Seleccione la opción **Archivo** desde la lista desplegable **Upload From** (Cargar desde).

    b. Para cargar el archivo de metadatos descargado de Azure Portal, seleccione el **icono para elegir archivo**.

    c. Haga clic en **Aceptar**.

7. Después de cargar el archivo de metadatos descargado, los campos se rellenarán automáticamente. Haga clic en **Update** (Actualizar).

    ![Botón Configurar inicio de sesión único](./media/envimmis-tutorial/configure6.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure al concederle acceso a Envi MMIS.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **Envi MMIS**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Envi MMIS**.

    ![Vínculo de Envi MMIS en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario de la lista y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-envi-mmis-test-user"></a>Creación de un usuario de prueba de Envi MMIS

Para permitir que los usuarios de Azure AD inicien sesión en Envi MMIS, deben aprovisionarse en Envi MMIS. En el caso de Envi MMIS, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el sitio de la compañía de Envi MMIS.

2. Haga clic en la pestaña **Lista de usuarios**.

    ![Agregar empleado](./media/envimmis-tutorial/user1.png)

3. Haga clic en el botón **Agregar usuario**.

    ![Agregar empleado](./media/envimmis-tutorial/user2.png)

4. En la sección **Add User** (Agregar usuario), realice estos pasos:

    ![Agregar empleado](./media/envimmis-tutorial/user3.png)

    a. En el cuadro de texto **Nombre de usuario**, escriba el nombre del usuario de la cuenta de Britta Simon, como **brittasimon\@contoso.com**.
    
    b. En el cuadro de texto **Nombre**, escriba el nombre del usuario de BrittaSimon, como **Britta**.

    c. En el cuadro de texto **Apellido**, escriba el nombre del usuario de Britta Simon, como **Simon**.

    d. Escriba el tratamiento del usuario en el cuadro de texto **Título**.
    
    e. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo electrónico de la cuenta de Britta Simon, como **brittasimon\@contoso.com**.

    f. En el cuadro de texto **SSO User Name** (Nombre de usuario de SSO), escriba el nombre del usuario de la cuenta de Britta Simon, como **brittasimon\@contoso.com**.

    g. Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Envi MMIS en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Envi MMIS para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

