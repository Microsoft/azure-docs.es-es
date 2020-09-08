---
title: 'Tutorial: Integración de Azure Active Directory con Yodeck | Microsoft Docs'
description: Obtenga información para configurar el inicio de sesión único entre Azure Active Directory y Yodeck.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/29/2019
ms.author: jeedes
ms.openlocfilehash: 8eed884ed9d957ede4ae9fc192ae5a8e501b652e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546243"
---
# <a name="tutorial-azure-active-directory-integration-with-yodeck"></a>Tutorial: Integración de Azure Active Directory con Yodeck

En este tutorial se aprende a integrar Yodeck con Azure Active Directory (Azure AD).
La integración de Yodeck con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Yodeck.
* Puede permitir que los usuarios inicien sesión automáticamente en Yodeck (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Yodeck, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único en Yodeck

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Yodeck admite el inicio de sesión único iniciado por **SP** e **IDP**.

## <a name="adding-yodeck-from-the-gallery"></a>Incorporación de Yodeck desde la galería

Para configurar la integración de Yodeck en Azure AD, debe agregar Yodeck desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Yodeck desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Yodeck**, seleccione **Yodeck** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Yodeck en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección puede configurar y probar el inicio de sesión único de Azure AD con Yodeck con un usuario de prueba denominado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Yodeck.

Para configurar y probar el inicio de sesión único de Azure AD con Yodeck, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Yodeck](#configure-yodeck-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Yodeck](#create-yodeck-test-user)** : para tener un homólogo de Britta Simon en Yodeck que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Yodeck, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Yodeck**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    ![Información de dominio y direcciones URL de inicio de sesión único de Yodeck](common/idp-identifier.png)

    En el cuadro de texto **Identificador**, escriba una dirección URL: `https://app.yodeck.com/api/v1/account/metadata/`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![imagen](common/both-preintegrated-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://app.yodeck.com/login`

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-yodeck-single-sign-on"></a>Configuración del inicio de sesión único de Yodeck

1. Para automatizar la configuración en **Yodeck**, debe instalar la **extensión del explorador de inicio de sesión seguro Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![imagen](./media/target-process-tutorial/install_extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Yodeck** para ir a la aplicación Yodeck. En ella, escriba las credenciales de administrador para iniciar sesión en Yodeck. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

    **Si desea configurar la aplicación manualmente, realice los siguientes pasos:**

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa de Yodeck como administrador.

1. Haga clic en la opción **Configuración de usuario** de la esquina superior derecha de la página y seleccione **Configuración de cuenta**.

    ![Configuración de Yodeck](./media/yodeck-tutorial/configure1.png)

1. Seleccione **SAML** y realice los siguientes pasos:

    ![Configuración de Yodeck](./media/yodeck-tutorial/configure2.png)

    a. Seleccione **Importar desde URL**.

    b. En el cuadro de texto **URL**, pegue el valor **Dirección URL de metadatos de federación de aplicación** que ha copiado de Azure Portal y haga clic en **Importar**.
    
    c. Después de importar **Dirección URL de metadatos de federación de aplicación**, los campos restantes se rellenan automáticamente.

    d. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba brittasimon@yourcompanydomain.extension. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección se habilita a Britta Simon para que use el inicio de sesión único de Azure al concederle acceso a Yodeck.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Yodeck**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Yodeck**.

    ![Vínculo a Yodeck en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-yodeck-test-user"></a>Creación de un usuario de prueba de Yodeck

Para permitir que los usuarios de Azure AD inicien sesión en Yodeck, tienen que aprovisionarse en Yodeck. En el caso de Yodeck, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en el sitio de la empresa de Yodeck como administrador.

2. Haga clic en la opción **Configuración de usuario** de la esquina superior derecha de la página y seleccione **Usuarios**.

    ![Agregar empleado](./media/yodeck-tutorial/user1.png)

3. Haga clic en **+Usuario** para abrir la pestaña **Detalles del usuario**.

    ![Agregar empleado](./media/yodeck-tutorial/user2.png)

4. En la página de diálogo **Detalles del usuario**, lleve a cabo los pasos siguientes:

    ![Agregar empleado](./media/yodeck-tutorial/user3.png)

    a. En el cuadro de texto **Nombre**, escriba el nombre del usuario, en este caso, **Britta**.

    b. En el cuadro de texto **Apellidos**, escriba los apellidos del usuario, en este caso, **Simon**.

    c. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, brittasimon@contoso.com.

    d. Seleccione la opción **Permisos de cuenta** adecuada según los requisitos de la organización.
    
    e. Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Yodeck en el panel de acceso, debería iniciar sesión automáticamente en la versión de Yodeck para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

