---
title: 'Tutorial: Integración de Azure Active Directory con Informes de gastos de N2F | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e Informes de gastos de N2F.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/01/2019
ms.author: jeedes
ms.openlocfilehash: 6036ff86c430422556ed4f7e1bc80fe122ed0a30
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552574"
---
# <a name="tutorial-azure-active-directory-integration-with-n2f---expense-reports"></a>Tutorial: Integración de Azure Active Directory con Informes de gastos de N2F

En este tutorial, aprenderá a integrar Informes de gastos de N2F con Azure Active Directory (Azure AD).
La integración de Informes de gastos de N2F con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Informes de gastos de N2F.
* Puede permitir que los usuarios inicien sesión automáticamente en Informes de gastos de N2F (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con los informes de gastos de N2F, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en Informes de gastos de N2F.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Informes de gastos de N2F admite el inicio de sesión único iniciado por **SP** e **IDP**.

## <a name="adding-n2f---expense-reports-from-the-gallery"></a>Incorporación de los informes de gastos de N2F desde la galería

Para configurar la integración de los informes de gastos de N2F con Azure AD, es preciso agregar dichos informes desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar los informes de gastos de N2F desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Informes de gastos de N2F**, seleccione **Informes de gastos de N2F** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Informes de gastos de N2F en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, puede configurar y probar el inicio de sesión único de Azure AD con los informes de gastos de N2F con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Informes de gastos de N2F.

Para configurar y probar el inicio de sesión único de Azure AD con Informes de gastos de N2F, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Informes de gastos de N2F](#configure-n2f---expense-reports-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Informes de gastos de N2F](#create-n2f---expense-reports-test-user)** : para tener un homólogo de Britta Simon en Informes de gastos de N2F que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Informes de gastos de N2F, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Informes de gastos de N2F**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, no es necesario realizar ningún paso porque la aplicación ya está integrada previamente con Azure.

    ![Información de dominio y direcciones URL de inicio de sesión único de Informes de gastos de N2F](common/preintegrated.png)

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de Informes de gastos de N2F](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://www.n2f.com/app/`

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

7. En la sección **Set up myPolicies** (Configurar myPolicies), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-n2f---expense-reports-single-sign-on"></a>Configuración del inicio de sesión único de Informes de gastos de N2F

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Informes de gastos de N2F.

2. Haga clic en **Settings** (Configuración) y seleccione **Advance Settings** (Configuración avanzada) en la lista desplegable.

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/configure1.png)

3. Seleccione la pestaña **Account settings** (Configuración de la cuenta).

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/configure2.png)

4. Seleccione **Authentication** (Autenticación) y luego seleccione la pestaña **+ Add an authentication method** (+Agregar un método de autenticación).

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/configure3.png)

5. Seleccione **SAML Microsoft Office 365** como método de autenticación.

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/configure4.png)

6. En la sección **Authentication method** (Método deautenticación), realice los pasos siguientes:

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/configure5.png)

    a. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    b. En el cuadro de texto **Metadata URL** (Dirección URL de metadatos), pegue el valor de **App Federation Metadata Url** (Dirección URL de metadatos de federación de aplicaciones) que copió de Azure Portal.

    c. Haga clic en **Save**(Guardar).

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

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a Informes de gastos de N2F.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Informes de gastos de N2F**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Informes de gastos de N2F**.

    ![El vínculo Informes de gastos de N2F en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-n2f---expense-reports-test-user"></a>Creación de un usuario de prueba de Informes de gastos de N2F

Para permitir que los usuarios de Azure AD inicien sesión en Informes de gastos de N2F, deben aprovisionarse en esta aplicación. En el caso de Informes de gastos de N2F, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el sitio de la compañía de Informes de gastos de N2F.

2. Haga clic en **Settings** (Configuración) y seleccione **Advance Settings** (Configuración avanzada) en la lista desplegable.

    ![Informes de gastos de N2F: agregar usuario](./media/n2f-expensereports-tutorial/configure1.png)

3. Seleccione la pestaña **Users** (Usuarios) en el panel de navegación izquierdo.

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/user1.png)

4. Seleccione la pestaña **+ New user** (+Nuevo usuario).

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/user2.png)

5. En la sección **User** (Usuario), realice los siguientes pasos:

    ![Configuración de Informes de gastos de N2F](./media/n2f-expensereports-tutorial/user3.png)

    a. En el cuadro de texto **Email address** (Dirección de correo electrónico), escriba la dirección de correo electrónico de un usuario, por ejemplo, **brittasimon\@contoso.com**.

    b. En el cuadro de texto **Nombre**, escriba el nombre de usuario, en este caso **Britta**.

    c. En el cuadro de texto **Name** (Nombre), escriba el nombre de usuario, por ejemplo, **BrittaSimon**.

    d. Elija **Role, Direct manager (N+1)** (Role, responsable directo [N+1]) y **Division** (División) según los requisitos de su organización.

    e. Haga clic en **Validate and send invitation** (Validar y enviar invitación).

    > [!NOTE]
    > Si tiene algún problema al agregar el usuario, póngase en contacto con el [equipo de soporte técnico de Informes de gastos de N2F](mailto:support@n2f.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Informes de gastos de N2F en el panel de acceso, debería iniciar sesión automáticamente en la versión de Informes de gastos de N2F para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

