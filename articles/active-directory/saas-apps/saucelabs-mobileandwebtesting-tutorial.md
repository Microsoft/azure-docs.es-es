---
title: 'Tutorial: Integración de Azure Active Directory con Sauce Labs - Mobile and Web Testing | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Sauce Labs - Mobile and Web Testing.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2019
ms.author: jeedes
ms.openlocfilehash: 8dd5ca148474a1fe1464217f60c79e3815a5b26d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547926"
---
# <a name="tutorial-azure-active-directory-integration-with-sauce-labs---mobile-and-web-testing"></a>Tutorial: Integración de Azure Active Directory con Sauce Labs - Mobile and Web Testing

En este tutorial, obtendrá información sobre cómo integrar Sauce Labs - Mobile and Web Testing con Azure Active Directory (Azure AD).
La integración de Sauce Labs - Mobile and Web Testing con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Sauce Labs - Mobile and Web Testing.
* Puede permitir que los usuarios inicien sesión automáticamente en Sauce Labs - Mobile and Web Testing (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Sauce Labs - Mobile and Web Testing, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único de Sauce Labs - Mobile and Web Testing

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Sauce Labs - Mobile and Web Testing admite el inicio de sesión único iniciado por **IDP**
* Sauce Labs - Mobile and Web Testing admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-sauce-labs---mobile-and-web-testing-from-the-gallery"></a>Adición de Sauce Labs - Mobile and Web Testing desde la galería

Para configurar la integración de Sauce Labs - Mobile and Web Testing en Azure AD, deberá agregar Sauce Labs - Mobile and Web Testing desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Sauce Labs - Mobile and Web Testing desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Sauce Labs - Mobile and Web Testing**, seleccione **Sauce Labs - Mobile and Web Testing** en el panel de resultados y después haga clic en **Agregar** para agregar la aplicación.

    ![Sauce Labs - Mobile and Web Testing en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Sauce Labs - Mobile and Web Testing con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Sauce Labs - Mobile and Web Testing.

Para configurar y probar el inicio de sesión único de Azure AD con Sauce Labs - Mobile and Web Testing, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Sauce Labs - Mobile and Web Testing](#configure-sauce-labs---mobile-and-web-testing-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Sauce Labs - Mobile and Web Testing](#create-sauce-labs---mobile-and-web-testing-test-user)** : para tener un homólogo de Britta Simon en Sauce Labs - Mobile and Web Testing que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Sauce Labs - Mobile and Web Testing, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Sauce Labs - Mobile and Web Testing**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

    ![Información de inicio de sesión único en Dominio y direcciones URL de Sauce Labs - Mobile and Web Testing](common/preintegrated.png)

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar Sauce Labs - Mobile and Web Testing**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-sauce-labs---mobile-and-web-testing-single-sign-on"></a>Configuración del inicio de sesión único de Sauce Labs - Mobile and Web Testing

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de Sauce Labs - Mobile and Web Testing como administrador.

2. Haga clic en el **icono de usuario** y seleccione la pestaña **Administración del equipo**.

    ![Configurar inicio de sesión único](./media/saucelabs-mobileandwebtesting-tutorial/configure1.png)

3. Escriba su **Nombre de dominio** en el cuadro de texto.

    ![Configurar inicio de sesión único](./media/saucelabs-mobileandwebtesting-tutorial/configure2.png)

4. Haga clic en la pestaña **Configurar**.

    ![Configurar inicio de sesión único](./media/saucelabs-mobileandwebtesting-tutorial/configure3.png)

5. En la sección **Configurar inicio de sesión único**, siga estos pasos.

    ![Configurar inicio de sesión único](./media/saucelabs-mobileandwebtesting-tutorial/configure4.png)

    a. Haga clic en **Examinar** y cargue el archivo de metadatos descargado de Azure AD.

    b. Seleccione la casilla **PERMITIR APROVISIONAMIENTO JUST-IN-TIME**.

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
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Sauce Labs - Mobile and Web Testing.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Sauce Labs - Mobile and Web Testing**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Sauce Labs - Mobile and Web Testing**.

    ![Vínculo de Sauce Labs - Mobile and Web Testing en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-sauce-labs---mobile-and-web-testing-test-user"></a>Creación de un usuario de prueba de Sauce Labs - Mobile and Web Testing

En esta sección, se crea un usuario llamado Britta Simon en Sauce Labs - Mobile and Web Testing. Sauce Labs - Mobile and Web Testing admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Sauce Labs - Mobile and Web Testing, se crea uno después de la autenticación.

> [!Note]
> Si necesita crear manualmente un usuario, póngase en contacto con el  [equipo de soporte técnico de Sauce Labs - Mobile and Web Testing](mailto:support@saucelabs.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el mosaico de Sauce Labs - Mobile and Web Testing en el panel de acceso, debe iniciar sesión automáticamente en la aplicación Sauce Labs - Mobile and Web Testing para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

