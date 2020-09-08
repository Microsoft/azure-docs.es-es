---
title: 'Tutorial: Integración de Azure Active Directory con Riskware | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Riskware.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: b147037a58e4af59c0c3ccd56b6f6b241d775308
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553349"
---
# <a name="tutorial-azure-active-directory-integration-with-riskware"></a>Tutorial: Integración de Azure Active Directory con Riskware

En este tutorial, aprenderá a integrar Riskware con Azure Active Directory (Azure AD).
La integración de Riskware con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Riskware.
* Puede habilitar a los usuarios para que inicien sesión automáticamente en Riskware (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Riskware, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Riskware

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Riskware admite el inicio de sesión único iniciado por **SP**

## <a name="adding-riskware-from-the-gallery"></a>Agregar Riskware desde la galería

Para configurar la integración de Riskware en Azure AD, será preciso que agregue Riskware desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Riskware desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Riskware**, seleccione **Riskware** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Riskware en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Riskware con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Riskware.

Para configurar y probar el inicio de sesión único de Azure AD con Riskware, es preciso completar los siguientes bloques de compilación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Riskware](#configure-riskware-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Riskware](#create-riskware-test-user)** : para tener un homólogo de Britta Simon en Riskware que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Riskware, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Riskware**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Riskware](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:
    
    | Entorno| Patrón de dirección URL|
    |--|--|
    | UAT|  `https://riskcloud.net/uat?ccode=<COMPANYCODE>` |
    | PROD| `https://riskcloud.net/prod?ccode=<COMPANYCODE>` |
    | DEMO| `https://riskcloud.net/demo?ccode=<COMPANYCODE>` |
    |||

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL:
    
    | Entorno| Patrón de dirección URL|
    |--|--|
    | UAT| `https://riskcloud.net/uat` |
    | PROD| `https://riskcloud.net/prod` |
    | DEMO| `https://riskcloud.net/demo` |
    |||

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Riskware](mailto:support@pansoftware.com.au) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar Riskware**, copie la dirección o direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-riskware-single-sign-on"></a>Configuración del inicio de sesión único en Riskware

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Riskware.

1. En la parte superior derecha, haga clic en **Maintenance** (Mantenimiento) para abrir la página de mantenimiento.

    ![Mantener la configuración de Riskware](./media/riskware-tutorial/tutorial_riskware_maintain.png)

1. En la página de mantenimiento, haga clic en **Authentication** (Autenticación).

    ![Autenticar la configuración de Riskware](./media/riskware-tutorial/tutorial_riskware_authen.png)

1. En la página **Authentication Configuration** (Configuración de la autenticación), realice los siguientes pasos:

    ![Autenticar la configuración de Riskware](./media/riskware-tutorial/tutorial_riskware_config.png)

    a. En **Type** (Tipo), seleccione **SAML** para la autenticación.

    b. En el cuadro de texto **Code** (Código), escriba el código, por ejemplo, AZURE_UAT.

    c. En el cuadro de texto **Description** (Descripción), escriba la descripción como configuración de AZURE para SSO.

    d. En el cuadro de texto **Single Sign On Page** (Dirección URL de inicio de sesión único), pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    e. En el cuadro de texto **Sign out Page** (Página de cierre de sesión), pegue el valor de **URL de cierre de sesión** que copió de Azure Portal.

    f. En el cuadro de texto **Post Form Field** (Campo de formulario Post), escriba el nombre del campo presente en la respuesta de post que contenga SAML como SAMLResponse.

    g. En el cuadro de texto **XML Identity Tag Name** (Nombre de la etiqueta de la identidad XML), escriba el atributo de tipo que contenga el identificador único de la respuesta SAML, como NameID.

    h. Abra **XML de metadatos** de Azure Portal en el Bloc de notas, copie el certificado del archivo de metadatos y péguelo en el cuadro de texto **Certificate** (Certificado).

    i. En el cuadro de texto **Consumer URL** (Dirección URL del consumidor), pegue el valor de **Reply URL** (Dirección URL de respuesta), que le proporcionará el equipo de soporte técnico.

    j. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Identifier** (Identificador), que le proporcionará el equipo de soporte técnico.

    > [!Note]
    > Póngase en contacto con el [equipo de soporte técnico de Riskware](mailto:support@pansoftware.com.au) para obtener estos valores.

    k. Seleccione la casilla **Use POST** (Usar POST).

    l. Seleccione la casilla **Use SAML Request** (Usar solicitud SAML).

    m. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Riskware.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Riskware**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Riskware**.

    ![Vínculo a Riskware en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-riskware-test-user"></a>Creación de un usuario de prueba de Riskware

Para permitir que los usuarios de Azure AD inicien sesión en Riskware, deben aprovisionarse en dicha solución. En Riskware, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Riskware como administrador de seguridad.

1. En la parte superior derecha, haga clic en **Maintenance** (Mantenimiento) para abrir la página de mantenimiento. 

    ![Mantener la configuración de Riskware](./media/riskware-tutorial/tutorial_riskware_maintain.png)

1. En la página de mantenimiento, haga clic en **People** (Usuarios).

    ![Configuración de personas en Riskware](./media/riskware-tutorial/tutorial_riskware_people.png)

1. Seleccione la pestaña **Details** (Detalles) y lleve a cabo los pasos siguientes:

    ![Detalles de configuración de Riskware](./media/riskware-tutorial/tutorial_riskware_details.png)

    a. Seleccione **Person Type** (Tipo de usuario), por ejemplo, Employee (Empleado).

    b. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **Britta**.

    c. En el cuadro de texto **Surname** (Apellido), escriba el apellido del usuario, en este caso **Simon**.

1. En la pestaña **Seguridad** , lleve a cabo estos pasos:

    ![Configuración de la seguridad en Riskware](./media/riskware-tutorial/tutorial_riskware_security.png)

    a. En la sección **Authentication** (Autenticación), seleccione el modo **Authentication** (Autenticación), que ha configurado como Configuración de AZURE para inicio de sesión único.

    b. En la sección **Logon Details** (Detalles de inicio de sesión), en el cuadro de texto **User ID** (Identificador del usuario), escriba la dirección de correo del usuario, por ejemplo, `brittasimon@contoso.com`.

    c. En el cuadro de texto **Password** (Contraseña), escriba la contraseña del usuario.

1. En la pestaña **Organization** (Organización), realice los pasos siguientes:

    ![Configuración de la organización en Riskware](./media/riskware-tutorial/tutorial_riskware_org.png)

    a. Seleccione la opción como organización **Level1**.

    b. En la sección **Person's Primary Workplace** (Área de trabajo principal del usuario), en el cuadro de texto **Location** (Ubicación), escriba la ubicación.

    c. En la sección **Employee** (Empleado), en **Employee Status** (Estado del empleado) seleccione Casual (Informal).

    d. Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Riskware en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Riskware para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
