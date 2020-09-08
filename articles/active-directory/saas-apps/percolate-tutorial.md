---
title: 'Tutorial: Integración de Azure Active Directory con Percolate | Microsoft Docs'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory y Percolate.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/01/2019
ms.author: jeedes
ms.openlocfilehash: deb64aa0c344e818b5fd85ca1a161293fd35d6f6
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554021"
---
# <a name="tutorial-azure-active-directory-integration-with-percolate"></a>Tutorial: Integración de Azure Active Directory con Percolate

En este tutorial, aprenderá a integrar Percolate con Azure Active Directory (Azure AD).

Esta integración ofrece las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a Percolate.
* Puede permitir que los usuarios inicien sesión automáticamente en Percolate (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Percolate, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de Percolate que tenga habilitado el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* Percolate admite el inicio de sesión único iniciado por SP e IDP.

## <a name="add-percolate-from-the-gallery"></a>Adición de Percolate desde la galería

Para configurar la integración de Percolate en Azure AD, es preciso que agregue Percolate desde la galería a su lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**:

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Percolate**. Seleccione **Percolate** en los resultados de búsqueda y, a continuación, seleccione **Agregar**.

     ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Percolate con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, tendrá que establecer una relación entre un usuario de Azure AD y el usuario correspondiente de Percolate.

Para configurar y probar el inicio de sesión único de Azure AD con Percolate, debe hacer lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configure el inicio de sesión único en Percolate](#configure-percolate-single-sign-on)** en el lado de la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Cree un usuario de prueba de Percolate](#create-a-percolate-test-user)** que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Percolate, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Percolate**, seleccione **Inicio de sesión único**:

    ![Selección de inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, no es necesario hacer nada para configurar la aplicación en modo iniciado por IdP. La aplicación ya está integrada con Azure.

    ![Información de dominio y direcciones URL de inicio de sesión único de Percolate](common/preintegrated.png)

5. Si quiere configurar la aplicación en modo iniciado por SP, seleccione **Establecer direcciones URL adicionales** y, en el cuadro **URL de inicio de sesión**, escriba **https://percolate.com/app/login** :

   ![Información de dominio y direcciones URL de inicio de sesión único de Percolate](common/metadata-upload-additional-signon.png)
6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el icono **Copiar** para copiar la **Dirección URL de metadatos de federación de aplicación**. Guarde esta dirección URL.

    ![Copia de la dirección URL de metadatos de federación de aplicación](common/copy-metadataurl.png)

7. En la sección **Configurar Percolate**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-percolate-single-sign-on"></a>Configuración del inicio de sesión único en Percolate

1. En una nueva ventana del explorador web, inicie sesión en Percolate como administrador.

2. En el lado izquierdo de la página principal, seleccione **Settings** (Configuración):
    
    ![Seleccionar Configuración](./media/percolate-tutorial/configure01.png)

3. En el panel izquierdo, seleccione **SSO** en **Organization** (Organización):

    ![Selección de SSO en Organization (Organización)](./media/percolate-tutorial/configure02.png)

    1. En el cuadro **Login URL** (URL de inicio de sesión), pegue el valor de la **Dirección URL de inicio de sesión** que copió de Azure Portal.

    1. En el cuadro **Entity ID** (Id. de entidad), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    1. En el Bloc de notas, abra el certificado codificado en base 64 que descargó de Azure Portal. Copie su contenido y péguelo en el cuadro **x509 certificates** (Certificados x509).

    1. En el cuadro **Email attribute** (Atributo de correo electrónico), escriba **emailaddress**.

    1. El cuadro **Identity provider metadata URL** (URL de metadatos del proveedor de identidades) es un campo opcional. Si copió una **dirección URL de metadatos de federación de aplicación** de Azure Portal, puede pegarla en este cuadro.

    1. En la lista **Should AuthNRequests be signed?** (¿Se debe firmar AuthNRequests?), seleccione **No**.

    1. En la lista **Enable SSO auto-provisioning** (Habilitar el aprovisionamiento automático del inicio de sesión único), seleccione **No**.

    1. Seleccione **Guardar**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**:

    ![Selección de Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Selección de Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **BrittaSimon@\<yourcompanydomain>.\<extension>** (Por ejemplo, BrittaSimon@contoso.com).

    1. Seleccione **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure AD, para lo cual le concederá acceso a Percolate.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Percolate**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Percolate**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Seleccionar Usuarios y grupos](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-percolate-test-user"></a>Creación de un usuario de prueba en Percolate

Para permitir que los usuarios de Azure AD inicien sesión en Percolate, deberá agregarlos a esta aplicación. Y los debe agregar manualmente.

Para crear una cuenta de usuario, siga estos pasos:

1. Inicie sesión en Percolate como administrador.

2. En el panel izquierdo, seleccione **Users** (Usuarios) en **Organization** (Organización). Seleccione **New users** (Nuevos usuarios):

    ![Selección de nuevos usuarios](./media/percolate-tutorial/configure03.png)

3. En la página **Create users** (Crear usuarios), siga estos pasos:

    ![Página de creación de usuarios](./media/percolate-tutorial/configure04.png)

    1. En el cuadro **Email** (Correo electrónico), escriba la dirección de correo electrónico del usuario. Por ejemplo, brittasimon@contoso.com.

    1. En el cuadro **Full name** (Nombre completo), escriba el nombre del usuario. Por ejemplo, **Brittasimon**.

    1. Seleccione **Create users** (Crear usuarios).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Percolate en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Percolate para la que configuró el inicio de sesión único. Para más información, consulte [Acceso y uso del aplicaciones en el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)