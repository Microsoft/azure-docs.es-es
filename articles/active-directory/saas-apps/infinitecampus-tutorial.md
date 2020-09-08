---
title: 'Tutorial: Integración de Azure Active Directory con Infinite Campus | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e Infinite Campus.
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
ms.openlocfilehash: ca154caf67d8dd715ad1341e9fe3c6cfde20fde0
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553117"
---
# <a name="tutorial-azure-active-directory-integration-with-infinite-campus"></a>Tutorial: Integración de Azure Active Directory con Infinite Campus

En este tutorial, obtendrá información sobre cómo integrar Infinite Campus con Azure Active Directory (Azure AD).
La integración de Infinite Campus con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Infinite Campus.
* Puede permitir que los usuarios inicien sesión automáticamente en Infinite Campus (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Infinite Campus, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Infinite Campus
* Como mínimo, debe ser un administrador de Azure Active Directory y tener un rol de seguridad de productos Campus de "sistema de información de estudiantes (SIS)" para completar la configuración.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Infinite Campus admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-infinite-campus-from-the-gallery"></a>Adición de Infinite Campus desde la galería

Para configurar la integración de Infinite Campus en Azure AD, debe agregar Infinite Campus desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Infinite Campus desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Haga clic en el botón **Nueva aplicación** en la parte superior del cuadro de diálogo para agregar una nueva aplicación.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Infinite Campus**, seleccione **Infinite Campus** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Infinite Campus en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Infinite Campus con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Infinite Campus.

Para configurar y probar el inicio de sesión único de Azure AD con Infinite Campus, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Infinite Campus](#configure-infinite-campus-single-sign-on)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Infinite Campus](#create-infinite-campus-test-user)** : para tener un homólogo de Britta Simon en Infinite Campus que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Infinite Campus, siga los pasos que se indican a continuación:

1. En la página de integración de la aplicación **Infinite Campus** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección Configuración básica de SAML, siga los pasos que se indican a continuación (tenga en cuenta que el dominio varía según el modelo de hospedaje, pero el valor **FULLY-QUALIFIED-DOMAIN** debe coincidir con la instalación de Infinite Campus):

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN>.infinitecampus.com/campus/SSO/<DISTRICTNAME>/SIS`.

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN>.infinitecampus.com/campus/<DISTRICTNAME>`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN>.infinitecampus.com/campus/SSO/<DISTRICTNAME>`.

    ![Información de dominio y direcciones URL de inicio de sesión único de Infinite Campus](common/sp-identifier-reply.png)

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-infinite-campus-single-sign-on"></a>Configuración del inicio de sesión único de Infinite Campus

1. En otra ventana del explorador web, inicie sesión en Infinite Campus como Administrador de seguridad.

2. En el lado izquierdo del menú, haga clic en **System Administration** (Administración del sistema).

    ![El administrador](./media/infinitecampus-tutorial/tutorial_infinitecampus_admin.png)

3. Vaya a **User Security** (Seguridad de usuarios) > **SAML Management** (Administración de SAML) > **SSO Service Provider Configuration** (Configuración del proveedor de servicios de SSO).

    ![SAML](./media/infinitecampus-tutorial/tutorial_infinitecampus_saml.png)

4. En la página **SSO Service Provider Configuration** (Configuración del proveedor de servicios de SSO), siga estos pasos:

    ![Inicio de sesión único](./media/infinitecampus-tutorial/tutorial_infinitecampus_sso.png)

    a. Seleccione **Enable SAML v2 Single Sign On**(Habilitar inicio de sesión único de SAML v2).

    b. Edite el **nombre del atributo opcional** para que contenga el **nombre**.

    c. En la sección **Select an option to retrieve Identity Provider (IDP) server data** (Seleccionar una opción para recuperar los datos del servidor del proveedor de identidades), seleccione **Metadata URL** (URL de metadatos), pegue el valor de **App Federation Metadata Url** (Dirección URL de metadatos de federación de aplicaciones) que copió de Azure Portal en el cuadro y, después, haga clic en **Sync** (Sincronizar).

    d. Después de hacer clic en **Sync** (Sincronizar), los valores se rellenan automáticamente en la página **SSO Service Provider Configuration** (Configuración del proveedor de servicios de SSO). Estos valores se pueden comprobar para que coincidan con los valores del paso 4 anterior.

    e. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Por ejemplo, BrittaSimon@contoso.com.

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

> [!NOTE]
> Si quiere que todos los usuarios de Azure tengan acceso de inicio de sesión único a Infinite Campus y confiar al sistema de permisos internos de Infinite Campus el control de acceso, puede establecer la propiedad **Asignación de usuarios necesaria** de la aplicación en No y omitir los pasos siguientes.

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Infinite Campus.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, después, **Infinite Campus**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Infinite Campus**.

    ![Vínculo a Infinite Campus en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-infinite-campus-test-user"></a>Creación de un usuario de prueba de Infinite Campus

Infinite Campuss tiene una arquitectura centrada en datos demográficos. Póngase en contacto con el [equipo de soporte técnico de Infinite Campus](mailto:sales@infinitecampus.com) para agregar los usuarios a la plataforma de Infinite Campus.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Infinite Campus en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Infinite Campus para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
