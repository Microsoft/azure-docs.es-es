---
title: 'Tutorial: Integración de Azure Active Directory con Trisotech Digital Enterprise Server | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Trisotech Digital Enterprise Server.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 3e8bf3d2407e2c3d0db4bb5d8591ba598aba68a3
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546447"
---
# <a name="tutorial-azure-active-directory-integration-with-trisotech-digital-enterprise-server"></a>Tutorial: Integración de Azure Active Directory con Trisotech Digital Enterprise Server

En este tutorial, aprenderá a integrar Trisotech Digital Enterprise Server con Azure Active Directory (Azure AD).
Integrar Trisotech Digital Enterprise Server con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Trisotech Digital Enterprise Server.
* Puede permitir que los usuarios inicien sesión automáticamente en Trisotech Digital Enterprise Server (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Trisotech Digital Enterprise Server, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción con inicio de sesión único de Trisotech Digital Enterprise Server

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Trisotech Digital Enterprise Server admite el inicio de sesión único iniciado por **SP**.

* Trisotech Digital Enterprise Server admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-trisotech-digital-enterprise-server-from-the-gallery"></a>Adición de Trisotech Digital Enterprise Server desde la galería

Para configurar la integración de Trisotech Digital Enterprise Server en Azure AD, será preciso que agregue Trisotech Digital Enterprise Server desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Trisotech Digital Enterprise Server desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Trisotech Digital Enterprise Server**, seleccione **Trisotech Digital Enterprise Server** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Trisotech Digital Enterprise Server en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Trisotech Digital Enterprise Server con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Trisotech Digital Enterprise Server.

Para configurar y probar el inicio de sesión único de Azure AD con Trisotech Digital Enterprise Server, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Trisotech Digital Enterprise Server](#configure-trisotech-digital-enterprise-server-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Trisotech Digital Enterprise Server](#create-trisotech-digital-enterprise-server-test-user)** : para tener un homólogo de Britta Simon en Trisotech Digital Enterprise Server que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Trisotech Digital Enterprise Server, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Trisotech Digital Enterprise Server**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de inicio de sesión único de dominio y direcciones URL de Trisotech Digital Enterprise Server](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.trisotech.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<companyname>.trisotech.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de cliente de Trisotech Digital Enterprise Server](mailto:support@trisotech.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-trisotech-digital-enterprise-server-single-sign-on"></a>Configuración del inicio de sesión único de Trisotech Digital Enterprise Server

1. En otra ventana del explorador web, inicie sesión en el sitio web de la empresa Trisotech Digital Enterprise Server Configuration como administrador.

2. Haga clic en el **icono de menú** y, después, seleccione **Administration** (Administración).

    ![Configurar inicio de sesión único](./media/trisotechdigitalenterpriseserver-tutorial/user1.png)

3. Seleccione **User Provider** (Proveedor de usuarios).

    ![Configurar inicio de sesión único](./media/trisotechdigitalenterpriseserver-tutorial/user2.png)

4. En la sección **User Provider Configurations** (Configuración de proveedor de usuarios), realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/trisotechdigitalenterpriseserver-tutorial/user3.png)

    a. Seleccione **Secured Assertion Markup Language 2 (SAML 2)** en la lista desplegable **Authentication Method** (Método de autenticación).

    b. En el cuadro de texto **URL de metadatos**, pegue el valor de **Dirección URL de metadatos de federación de la aplicación** que copió en Azure Portal.

    c. En el cuadro de texto **Application ID** (Application ID), escriba la dirección URL usando el patrón siguiente: `https://<companyname>.trisotech.com`.

    d. Haga clic en **Guardar**

    e. Escriba el nombre de dominio en el cuadro de texto **Dominios permitidos (vacío significa todos)** para asignar automáticamente licencias para usuarios que coinciden con los dominios permitidos.

    f. Haga clic en **Guardar**

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Trisotech Digital Enterprise Server.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Trisotech Digital Enterprise Server**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Trisotech Digital Enterprise Server**.

    ![Vínculo de Trisotech Digital Enterprise Server en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-trisotech-digital-enterprise-server-test-user"></a>Creación de un usuario de prueba de Trisotech Digital Enterprise Server

En esta sección se va a crear un usuario de prueba llamado Britta Simon en Trisotech Digital Enterprise Server. Trisotech Digital Enterprise Server admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Trisotech Digital Enterprise Server, se crea uno después de la autenticación.

>[!Note]
>Si tiene que crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Trisotech Digital Enterprise Server](mailto:support@trisotech.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Trisotech Digital Enterprise Server en el panel de acceso, debería iniciar sesión automáticamente en la aplicación Trisotech Digital Enterprise Server para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

