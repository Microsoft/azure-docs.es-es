---
title: 'Tutorial: Integración de Azure Active Directory con XaitPorter | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y XaitPorter.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/03/2019
ms.author: jeedes
ms.openlocfilehash: 80900f8a63d29e8c36ec190945efc3c44f713a3f
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296930"
---
# <a name="tutorial-azure-active-directory-integration-with-xaitporter"></a>Tutorial: Integración de Azure Active Directory con XaitPorter

En este tutorial, obtendrá información sobre cómo integrar XaitPorter con Azure Active Directory (Azure AD).
La integración de XaitPorter con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a XaitPorter.
* Puede permitir que los usuarios inicien sesión automáticamente en XaitPorter (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con XaitPorter, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único en XaitPorter

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* XaitPorter admite el inicio de sesión único iniciado por **SP**

## <a name="adding-xaitporter-from-the-gallery"></a>Adición de XaitPorter desde la galería

Para configurar la integración de XaitPorter en Azure AD, debe agregar XaitPorter desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar XaitPorter desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **XaitPorter**, seleccione **XaitPorter** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![XaitPorter en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con XaitPorter con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de XaitPorter.

Para configurar y probar el inicio de sesión único de Azure AD con XaitPorter, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de XaitPorter](#configure-xaitporter-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en XaitPorter](#create-xaitporter-test-user)** : para tener un homólogo de Britta Simon en XaitPorter que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con XaitPorter, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **XaitPorter**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de XaitPorter](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.xaitporter.com/saml/login`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<subdomain>.xaitporter.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de XaitPorter](https://www.xait.com/support/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

6. Proporcione la **dirección IP** o la **dirección URL de metadatos de federación de aplicación** al [equipo de soporte técnico de SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/), de modo que XaitPorter pueda garantizar que la dirección IP sea accesible desde la instancia de XaitPorter mediante su inclusión en la lista de aprobados. 

### <a name="configure-xaitporter-single-sign-on"></a>Configuración del inicio de sesión único de XaitPorter

1. Para automatizar la configuración de XaitPorter, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Install the extension** (Instalar la extensión).

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar XaitPorter** para ir a la aplicación XaitPorter. Desde allí, proporcione las credenciales de administrador para iniciar sesión en XaitPorter. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar XaitPorter manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de XaitPorter como administrador y realice los pasos siguientes:

4. Haga clic en **Admin** (Administrador).

    ![Configurar inicio de sesión único](./media/xaitporter-tutorial/user1.png)

5. Seleccione **Manage Single Sign-On** (Administrar inicio de sesión único) desde la lista desplegable **System Setup** (Configuración del sistema).

    ![Configurar inicio de sesión único](./media/xaitporter-tutorial/user2.png)

6. En la sección **MANAGE SINGLE SIGN-ON** (Administrar inicio de sesión único), lleve a cabo los pasos siguientes:

    ![Configurar inicio de sesión único](./media/xaitporter-tutorial/user3.png)

    a. Seleccione **Enable Single Sign-On Authentication** (Habilitar autenticación de inicio de sesión único).

    b. En el cuadro de texto **Identity Provider Settings** (Configuración del proveedor de identidades), pegue la **dirección URL de metadatos de federación de aplicación** que ha copiado de Azure Portal y haga clic en **Fetch** (Recuperar).

    c. Seleccione **Enable Autocreation of Users** (Habilitar la creación automática de usuarios).

    d. Haga clic en **OK**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a XaitPorter.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **XaitPorter**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **XaitPorter**.

    ![Vínculo a XaitPorter en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-xaitporter-test-user"></a>Creación de un usuario de prueba de XaitPorter

En esta sección, creará un usuario llamado Britta Simon en XaitPorter. Trabaje con el [equipo de soporte técnico al cliente de XaitPorter](https://www.xait.com/support/) para agregar los usuarios a la plataforma de XaitPorter. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de XaitPorter en el panel de acceso y debería iniciar sesión automáticamente en la versión de XaitPorter para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
