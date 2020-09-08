---
title: 'Tutorial: Integración de Azure Active Directory con Clarizen | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Clarizen.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2019
ms.author: jeedes
ms.openlocfilehash: 4e312c70e8d1630c1e8d1bb9a893b338383c7966
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88520726"
---
# <a name="tutorial-azure-active-directory-integration-with-clarizen"></a>Tutorial: Integración de Azure Active Directory con Clarizen

En este tutorial, aprenderá a integrar Clarizen con Azure Active Directory (Azure AD).
La integración de Clarizen con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Clarizen.
* Puede permitir que los usuarios inicien sesión automáticamente en Clarizen (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Clarizen, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Clarizen

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Clarizen admite inicio de sesión único iniciado por **IDP**

## <a name="adding-clarizen-from-the-gallery"></a>Incorporación de Clarizen desde la Galería

Para configurar la integración de Clarizen en Azure AD, es preciso agregar Clarizen desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Clarizen desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Clarizen**, seleccione **Clarizen** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Clarizen en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único en Azure AD con Clarizen con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Clarizen.

Para configurar y probar el inicio de sesión único de Azure AD con Clarizen, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Clarizen](#configure-clarizen-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de Clarizen](#create-clarizen-test-user)** , para tener un homólogo de Britta Simon en Clarizen vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único en Azure AD con Clarizen, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Clarizen**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información sobre el dominio y las direcciones URL de inicio de sesión único en Clarizen](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba un valor: `Clarizen`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://.clarizen.com/Clarizen/Pages/Integrations/SAML/SamlResponse.aspx`

    > [!NOTE]
    > Estos valores no son reales. Tiene que usar el identificador y la dirección URL de respuesta reales. Aquí se recomienda que utilice el valor de cadena único en el identificador. Para obtener los valores reales, póngase en contacto con el [equipo de soporte técnico de Clarizen](https://success.clarizen.com/hc/en-us/requests/new).

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Clarizen** (Configurar Clarizen), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-clarizen-single-sign-on"></a>Configuración del inicio de sesión único en Clarizen

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Clarizen como administrador.

1. Haga clic en el nombre de usuario y luego haga clic en **Settings** (Configuración).

    ![Clic en "Settings" (Configuración) debajo del nombre de usuario](./media/clarizen-tutorial/tutorial_clarizen_001.png "Configuración")

1. Haga clic en la pestaña **Global Settings** (Configuración global). A continuación, junto a **Federated Authentication** (Autenticación federada), haga clic en **edit** (Editar).

    ![Pestaña Global Settings (Configuración global)](./media/clarizen-tutorial/tutorial_clarizen_002.png "Configuración global")

1. En el cuadro de diálogo **Federated Authentication** (Autenticación federada), realice los pasos siguientes:

    ![Cuadro de diálogo "Federated Authentication" (Autenticación federada)](./media/clarizen-tutorial/tutorial_clarizen_003.png "Federated Authentication")

    a. Seleccione **Habilitar autenticación federada**.

    b. Para cargar el certificado descargado, haga clic en **Cargar** .

    c. En el cuadro **URL de inicio de sesión**, escriba el valor de **Dirección URL de inicio de sesión** de la ventana de configuración de aplicaciones de Azure AD.

    d. En el cuadro **Sign-out URL** (Dirección URL de cierre de sesión), escriba el valor de **Logout URL** (Dirección URL de cierre de sesión) de la ventana de configuración de aplicaciones de Azure AD.

    e. Seleccione **Usar POST**.

    f. Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Clarizen.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Clarizen**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Clarizen**.

    ![Vínculo a Clarizen en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-clarizen-test-user"></a>Creación del usuario de prueba de Clarizen

El objetivo de esta sección es crear un usuario llamado Britta Simon en Clarizen.

**Para crear un usuario manualmente, siga los pasos siguientes:**

Para permitir que los usuarios de Azure AD inicien sesión en Clarizen, debe aprovisionar las cuentas de usuario. En el caso de Clarizen, el aprovisionamiento es una tarea manual.

1. Inicie sesión en el sitio de la compañía de Clarizen como administrador.

2. Haga clic en **Contactos**.

    ![Clic en "People" (Personas)](./media/clarizen-tutorial/create_aaduser_001.png "Personas")

3. Haga clic en **Invitar a usuario**.

    ![Botón "Invite user" (Invitar a usuario)](./media/clarizen-tutorial/create_aaduser_002.png "Invitar a usuarios")

1. En el cuadro de diálogo **Invite People** (Invitar a personas), siga estos pasos:

    ![Cuadro de diálogo "Invite People" (Invitar a personas)](./media/clarizen-tutorial/create_aaduser_003.png "Invitar a contactos")

    a. En el cuadro **Email** (Correo electrónico), escriba la dirección de correo electrónico de la cuenta de Britta Simon.

    b. Haga clic en **Invitar**.

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.


### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Clarizen del panel de acceso, debe iniciar sesión automáticamente en la aplicación Clarizen para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
