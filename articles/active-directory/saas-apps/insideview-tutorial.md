---
title: 'Tutorial: Integración de Azure Active Directory con InsideView | Microsoft Docs'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory e InsideView.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/20/2019
ms.author: jeedes
ms.openlocfilehash: 17c9672663cae4df7eacb72779e6caf6476e3d24
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550493"
---
# <a name="tutorial-azure-active-directory-integration-with-insideview"></a>Tutorial: Integración de Azure Active Directory con InsideView

En este tutorial, obtendrá información sobre cómo integrar InsideView con Azure Active Directory (Azure AD).
Esta integración ofrece las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a InsideView.
* Puede permitir que los usuarios inicien sesión automáticamente en InsideView (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con InsideView, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de InsideView que tenga el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* InsideView admite SSO iniciado por IdP.

## <a name="add-insideview-from-the-gallery"></a>Incorporación de InsideView desde la galería

Para configurar la integración de InsideView en Azure AD, será preciso que agregue InsideView desde la galería a la lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**:

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **InsideView**. Seleccione **InsideView** en los resultados de búsqueda y, a continuación, seleccione **Agregar**.

    ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con InsideView con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, tendrá que establecer una relación entre un usuario de Azure AD y el usuario correspondiente de InsideView.

Para configurar y probar el inicio de sesión único de Azure AD con InsideView, debe hacer lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configure el inicio de sesión único en InsideView](#configure-insideview-single-sign-on)** en el lado de la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Cree un usuario de prueba de InsideView](#create-an-insideview-test-user)** que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con InsideView, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación InsideView, seleccione **Inicio de sesión único**:

    ![Selección de inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Configuración básica de SAML](common/idp-reply.png)

    En el cuadro **Dirección URL de respuesta**, escriba una dirección URL con este patrón:

    `https://my.insideview.com/iv/<STS Name>/login.iv`

    > [!NOTE]
    > Este valor es un marcador de posición. Debe utilizar la dirección URL real de respuesta. Póngase en contacto con el [equipo de soporte técnico de InsideView](mailto:support@insideview.com) para obtener este valor. También puede consultar los patrones que se muestran en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** situado junto a **Certificado (sin procesar)** , según sus requisitos, y guarde el certificado en el equipo:

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Configurar InsideView**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-insideview-single-sign-on"></a>Configuración del inicio de sesión único de InsideView

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de InsideView.

1. En la parte superior de la ventana, seleccione **Admin** (Administrador), **SingleSignOn Settings** (Configuración de inicio de sesión único) y **Add SAML** (Agregar SAML).
   
   ![Configuración del inicio de sesión único de SAML](./media/insideview-tutorial/ic794135.png "Configuración del inicio de sesión único de SAML")

1. En la sección **Add a New SAML** (Agregar un nuevo SAML), realice estos pasos.

    ![Sección Agregar un nuevo SAML](./media/insideview-tutorial/ic794136.png "Sección Agregar un nuevo SAML")

    1. En el cuadro de texto **STS Name** (Nombre STS), escriba el nombre de la configuración.

    1. En el cuadro **SamlP/WS-Fed Unsolicited EndPoint** (Punto de conexión no solicitado de SamlP/WS-Fed), pegue la **dirección URL de inicio de sesión** que copió de Azure Portal.

    1. Abra el certificado sin procesar que descargó desde Azure Portal. Copie el contenido del certificado en el Portapapeles y, a continuación, pegue el contenido en el cuadro **STS Certificate** (Certificado STS).

    1. En el cuadro de texto **Crm User Id Mapping** (Asignación de Id de usuario de Crm), escriba **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** .

    1. En el cuadro de texto **Crm Email Mapping** (Asignación de correo electrónico de Crm), escriba **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** .

    1. En el cuadro de texto **Crm First Name Mapping** (Asignación de nombre de Crm), escriba **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`** .

    1. En el cuadro de texto **Crm lastName Mapping** (Asignación de apellido de Crm), escriba **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`** .  

    1. Seleccione **Guardar**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**:

    ![Selección de Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la ventana:

    ![Selección de Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **BrittaSimon@\<yourcompanydomain>.\<extension>** (Por ejemplo, BrittaSimon@contoso.com).

    1. Seleccione **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a InsideView.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **InsideView**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **InsideView**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-an-insideview-test-user"></a>Creación de un usuario de prueba de InsideView

Para permitir que los usuarios de Azure AD inicien sesión en InsideView, deberá agregarlos a esta aplicación. Y los debe agregar manualmente.

Para crear usuarios o contactos en InsideView, póngase en contacto con el [equipo de soporte técnico de InsideView](mailto:support@insideview.com).

> [!NOTE]
> Puede usar cualquier API o herramienta de creación de cuentas de usuario que proporcione InsideView para aprovisionar cuentas de usuario de AAD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de InsideView en el panel de acceso, debería iniciar sesión automáticamente en la instancia de InsideView para la que configuró el inicio de sesión único. Para más información acerca del Panel de acceso, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
