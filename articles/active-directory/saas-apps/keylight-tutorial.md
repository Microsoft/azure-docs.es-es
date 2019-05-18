---
title: 'Tutorial: Integración de Azure Active Directory con LockPath Keylight | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y LockPath Keylight.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 234a32f1-9f56-4650-9e31-7b38ad734b1a
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7a03bb2626525bf022b109105a7c6bc0dee23aea
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407051"
---
# <a name="tutorial-azure-active-directory-integration-with-lockpath-keylight"></a>Tutorial: Integración de Azure Active Directory con LockPath Keylight

En este tutorial, obtendrá información sobre cómo integrar LockPath Keylight con Azure Active Directory (Azure AD).
La integración de LockPath Keylight con Azure AD le proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a LockPath Keylight.
* Puede permitir que los usuarios inicien sesión automáticamente en LockPath Keylight (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con LockPath Keylight, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en LockPath Keylight

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* LockPath Keylight admite el inicio de sesión único iniciado por **SP**.
* LockPath Keylight admite aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-lockpath-keylight-from-the-gallery"></a>Incorporación de LockPath Keylight desde la galería

Para configurar la integración de LockPath Keylight en Azure AD, es preciso agregar LockPath Keylight desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar LockPath Keylight desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **LockPath Keylight**, seleccione **LockPath Keylight** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![LockPath Keylight en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con LockPath Keylight con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de LockPath Keylight.

Para configurar y probar el inicio de sesión único de Azure AD con LockPath Keylight, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)**: para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de LockPath Keylight](#configure-lockpath-keylight-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**, para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**, para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de LockPath Keylight](#create-lockpath-keylight-test-user)**: para tener un homólogo de Britta Simon en LockPath Keylight que esté vinculado a la representación de ella en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)**: para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con LockPath Keylight, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **LockPath Keylight**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de LockPath Keylight](common/sp-identifier-reply.png)

     a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<company name>.keylightgrc.com/`

    b. En el cuadro de texto **Identificador (id. de entidad)**, escriba una dirección URL con el siguiente patrón: `https://<company name>.keylightgrc.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<company name>.keylightgrc.com/Login.aspx`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales. Póngase en contacto con el [equipo de soporte técnico de cliente de LockPath Keylight](https://www.lockpath.com/contact/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (sin procesar)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Set up LockPath Keylight** (Configurar LockPath Keylight), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-lockpath-keylight-single-sign-on"></a>Configuración del inicio de sesión único en LockPath Keylight

1. Realice los pasos siguientes para habilitar el inicio de sesión único en LockPath Keylight:

     a. Inicie sesión en su cuenta de LockPath Keylight como administrador.

    b. En el menú de la parte superior, haga clic en el icono de la **persona** y seleccione **Keylight Setup** (Configuración de Keylight).

    ![Configurar inicio de sesión único](./media/keylight-tutorial/401.png)

    c. En la vista de árbol de la izquierda, haga clic en **SAML**.

    ![Configurar inicio de sesión único](./media/keylight-tutorial/402.png)

    d. En el cuadro de diálogo **SAML Settings** (Configuración de SAML), haga clic en **Edit** (Editar).

    ![Configurar inicio de sesión único](./media/keylight-tutorial/404.png)

1. En la página del cuadro de diálogo **Edit SAML Settings** (Editar la configuración de SAML), realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/keylight-tutorial/405.png)

     a. Establezca **SAML authentication** (Autenticación SAML) como **Active** (Activada).

    b. En el cuadro de texto **Identity Provider Login URL** (Dirección URL de inicio de sesión único del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    c. En el cuadro de texto **Identity Provider Logout URL** (Dirección URL de cierre de sesión único del proveedor de identidades), pegue el valor de **Dirección URL de cierre de sesión**  que ha copiado de Azure Portal.

    d. Haga clic en **Elegir archivo** para seleccionar el certificado de LockPath Keylight descargado y después haga clic en **Abrir** para cargarlo.

    e. Establezca **SAML User Id location** (Ubicación de id. de usuario de SAML) en **NameIdentifier element of the subject statement** (Elemento NameIdentifier de la instrucción Subject).

    f. Proporcione el **proveedor de servicios de Keylight** mediante el siguiente patrón: `https://<CompanyName>.keylightgrc.com`.

    g. Establezca **Auto-provision users** (Usuarios de aprovisionamiento automático) en **Active** (Activado).

    h. Establezca **Auto-provision account type** (Tipo de cuenta de aprovisionamiento automático) en **Full User** (Usuario completo).

    i. Establezca **Auto-provision security role** (Rol de seguridad de aprovisionamiento automático) y seleccione **Standard User with SAML** (Usuario estándar con SAML).

    j. Establezca **Auto-provision security config** (Configuración de seguridad de aprovisionamiento automático) y seleccione **Standard User Configuration** (Configuración de usuario estándar).

    k. En el cuadro **Atributo de correo electrónico**, escriba `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    l. En el cuadro de texto **Atributo de nombre**, escriba `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`.

    m. En el cuadro de texto **Atributo de apellido**, escriba `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`.

    n. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a Britta Simon a LockPath Keylight para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **LockPath Keylight**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **LockPath Keylight**.

    ![Vínculo a LockPath Keylight en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-lockpath-keylight-test-user"></a>Creación de un usuario de prueba de LockPath Keylight

En esta sección, creará un usuario llamado Britta Simon en LockPath Keylight. LockPath Keylight admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe aún en LockPath Keylight, se crea después de la autenticación. Si necesita crear manualmente un usuario, es preciso que se ponga en contacto con el [equipo de soporte técnico de cliente de LockPath Keylight](https://www.lockpath.com/contact/).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de LockPath Keylight en el panel de acceso, debería iniciar sesión automáticamente en la versión de LockPath Keylight para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)