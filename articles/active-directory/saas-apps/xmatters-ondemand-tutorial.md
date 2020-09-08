---
title: 'Tutorial: Integración de Azure Active Directory con xMatters OnDemand | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y xMatters OnDemand.
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
ms.openlocfilehash: 9e84be9cbc6aa50f4beb3ed333b47459b15fbcc1
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546294"
---
# <a name="tutorial-azure-active-directory-integration-with-xmatters-ondemand"></a>Tutorial: Integración de Azure Active Directory con xMatters OnDemand

En este tutorial, aprenderá a integrar xMatters OnDemand con Azure Active Directory (Azure AD).
Integrar xMatters OnDemand con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a xMatters OnDemand.
* Puede permitir que los usuarios inicien sesión automáticamente en xMatters OnDemand (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con xMatters OnDemand, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único en xMatters OnDemand

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* xMatters OnDemand admite el inicio de sesión único iniciado por **IDP**.

## <a name="adding-xmatters-ondemand-from-the-gallery"></a>Incorporación de xMatters OnDemand desde la galería

Para configurar la integración de xMatters OnDemand en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar xMatters OnDemand desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **xMatters OnDemand**, seleccione **xMatters OnDemand** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![xMatters OnDemand en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, va a configurar y probar el inicio de sesión único de Azure AD con xMatters OnDemand con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de xMatters OnDemand.

Para configurar y probar el inicio de sesión único de Azure AD con xMatters OnDemand, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de xMatters OnDemand](#configure-xmatters-ondemand-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de xMatters OnDemand](#create-xmatters-ondemand-test-user)** : para tener un homólogo de Britta Simon en xMatters OnDemand que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con xMatters OnDemand, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **xMatters OnDemand**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de xMatters OnDemand](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:

    - `https://<companyname>.au1.xmatters.com.au/`
    - `https://<companyname>.cs1.xmatters.com/`
    - `https://<companyname>.xmatters.com/`
    - `https://www.xmatters.com`
    - `https://<companyname>.xmatters.com.au/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    - `https://<companyname>.au1.xmatters.com.au`
    - `https://<companyname>.xmatters.com/sp/<instancename>`
    - `https://<companyname>.cs1.xmatters.com/sp/<instancename>`
    - `https://<companyname>.au1.xmatters.com.au/<instancename>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte al cliente de xMatters OnDemand](https://www.xmatters.com/company/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

    > [!IMPORTANT]
    > Debe reenviar el certificado al [equipo de soporte técnico de xMatters OnDemand](https://www.xmatters.com/company/contact-us/). El equipo de soporte técnico de xMatters debe cargar el certificado para que se pueda finalizar la configuración del inicio de sesión único.

6. En la sección **Configurar xMatters OnDemand**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-xmatters-ondemand-single-sign-on"></a>Configuración del inicio de sesión único en xMatters OnDemand

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de xMatters OnDemand como administrador.

2. En la barra de herramientas de la parte superior, haga clic en **Administrador** y luego en **Detalles de la compañía** en la barra de navegación del lado izquierdo.

    ![Administrador](./media/xmatters-ondemand-tutorial/IC776795.png "Administración")

3. En la página **Configuración de SAML** , realice los siguientes pasos:

    ![SAML Configuration (Configuración de SAML)](./media/xmatters-ondemand-tutorial/IC776796.png "Configuración de SAML")

    a. Seleccione **Habilitar SAML**.

    b. En el cuadro de texto **Id. de proveedor de identidad**, pegue el valor del **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Single Sign On URL** (Dirección URL de inicio de sesión único), pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Single Logout URL** (Dirección URL de cierre de sesión único), pegue la **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    e. En la página de detalles de la compañía, en la parte superior, haga clic en **Guardar cambios**.

    ![Company details](./media/xmatters-ondemand-tutorial/IC776797.png "Detalles de la compañía")

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a xMatters OnDemand.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **xMatters OnDemand**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione  **xMatters OnDemand**.

    ![Vínculo a xMatters OnDemand en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-xmatters-ondemand-test-user"></a>Creación de un usuario de prueba de xMatters OnDemand

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en xMatters OnDemand.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. Inicie sesión en su inquilino de **xMatters OnDemand**.

2. Haga clic en la pestaña **Users** (Usuarios) y, luego, en **Add User** (Agregar usuario).

    ![Usuarios](./media/xmatters-ondemand-tutorial/IC781048.png "Usuarios")

3. En la sección **Agregar un usuario** , realice estos pasos:

    ![Adición de un usuario](./media/xmatters-ondemand-tutorial/IC781049.png "Agregar un usuario")

    a. Seleccione **Active**(Activo).

    b. En el cuadro de texto **User ID** (Identificador de usuario), escriba el identificador de usuario, en este caso, Brittasimon@contoso.com.

    c. En el cuadro de texto **Nombre**, escriba el nombre del usuario, en este caso, Britta.

    d. En el cuadro de texto **Apellidos**, escriba el apellido del usuario, en este caso Simon.

    e. En el cuadro de texto **Sitio**, escriba el sitio válido de una cuenta de Azure AD válida que quiera aprovisionar.

    f. Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de xMatters OnDemand en el panel de acceso, debería iniciar sesión automáticamente en la instancia de xMatters OnDemand para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

