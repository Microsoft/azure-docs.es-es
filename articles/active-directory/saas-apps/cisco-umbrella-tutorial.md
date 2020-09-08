---
title: 'Tutorial: Integración de Azure Active Directory con Cisco Umbrella| Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Cisco Umbrella.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: 86ba8527a6da3e995fc3cf63937b11f1915c82a1
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88556037"
---
# <a name="tutorial-azure-active-directory-integration-with-cisco-umbrella"></a>Tutorial: Integración de Azure Active Directory con Cisco Umbrella

En este tutorial, aprenderá a integrar Cisco Umbrella con Azure Active Directory (Azure AD).
Integrar Cisco Umbrella con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Cisco Umbrella.
* Puede permitir que los usuarios inicien sesión automáticamente en Cisco Umbrella (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Cisco Umbrella, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único de Cisco Umbrella

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cisco Umbrella admite SSO iniciado por **SP e IDP**

## <a name="adding-cisco-umbrella-from-the-gallery"></a>Incorporación de Cisco Umbrella desde la galería

Para configurar la integración de Cisco Umbrella en Azure AD, será preciso que añada Cisco Umbrella desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Cisco Umbrella desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Cisco Umbrella**, seleccione **Cisco Umbrella** en el panel de resultados y, luego, haga clic en el botón **Agregar** para añadir la aplicación.

     ![Cisco Umbrella en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con [nombre de aplicación] utilizando usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de [nombre de aplicación].

Para configurar y probar el inicio de sesión único de Azure AD con [nombre de aplicación], es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Cisco Umbrella](#configure-cisco-umbrella-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Cisco Umbrella](#create-cisco-umbrella-test-user)** : para tener un homólogo de Britta Simon en Cisco Umbrella que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con [nombre de aplicación], siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Cisco Umbrella**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

    ![Información de dominio y direcciones URL de inicio de sesión único de Cisco Umbrella](common/both-preintegrated-signon.png)

    a. Si quiere configurar la aplicación en modo iniciado por **SP**, realice los siguientes pasos:

    b. Haga clic en **Establecer direcciones URL adicionales**.

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://login.umbrella.com/sso`

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar **XML de metadatos** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Cisco Umbrella** (Configurar Cisco Umbrella), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-cisco-umbrella-single-sign-on"></a>Configurar el inicio de sesión único de Cisco Umbrella

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de Cisco Umbrella como administrador.

2. En el lado izquierdo del menú, haga clic en **Administrador**, vaya a **Autenticación** y, a continuación, haga clic en **SAML**.

    ![El administrador](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_admin.png)

3. Elija **Otro** y haga clic en **SIGUIENTE**.

    ![El otro](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_other.png)

4. En la página **Metadatos de Cisco Umbrella**, haga clic en **SIGUIENTE**.

    ![Los metadatos](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_metadata.png)

5. En la pestaña **Cargar metadatos**, si había preconfigurado SAML, seleccione la opción **Haga clic aquí para cambiarlos** y siga los pasos siguientes.

    ![El siguiente](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_next.png)

6. En la **Opción A: Cargar archivo XML**, cargue el archivo **XML de metadatos de federación** que descargó desde Azure Portal y, una vez que los siguientes valores se autorrellenen automáticamente tras la carga de los metadatos, haga clic en **SIGUIENTE**.

    ![El método ChooseFile](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_choosefile.png)

7. En la sección **Validación de la configuración de SAML**, haga clic en **PROBAR LA CONFIGURACIÓN DE SAML**.

    ![La prueba](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_test.png)

8. Haga clic en **GUARDAR**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Cisco Umbrella.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Cisco Umbrella**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Cisco Umbrella**.

    ![Vínculo a Cisco Umbrella en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-cisco-umbrella-test-user"></a>Creación de usuario de prueba de Cisco Umbrella

Para permitir que los usuarios de Azure AD inicien sesión en Cisco Umbrella, tienen que aprovisionarse en Cisco Umbrella.  
En el caso de Cisco Umbrella, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de Cisco Umbrella como administrador.

2. En el lado izquierdo del menú, haga clic en **Administrador** y vaya a **Cuentas**.

    ![La cuenta](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_account.png)

3. En la página **Cuentas**, haga clic en **Agregar** en el lado superior derecho de la página y siga los siguientes pasos.

    ![El usuario](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_createuser.png)

    a. En el campo **Nombre**, escriba el nombre, como **Britta**.

    b. En el campo **Apellido**, escriba el apellido, como **simon**.

    c. En **Elegir rol de administrador delegado**, seleccione su rol.
  
    d. En el campo **Email address** (Dirección de correo electrónico), escriba la dirección de correo electrónico de un usuario, por ejemplo, **brittasimon\@contoso.com**.

    e. En el campo **Contraseña**, escriba su contraseña.

    f. En el campo **Confirmar contraseña**, vuelva a escribir su contraseña.

    g. Haga clic en **CREATE** (Crear).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Cisco Umbrella en el panel de acceso, debería iniciar sesión automáticamente en la versión de Cisco Umbrella para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
