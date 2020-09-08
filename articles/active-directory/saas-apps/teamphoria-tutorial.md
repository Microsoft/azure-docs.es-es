---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Teamphoria | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Teamphoria.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/09/2019
ms.author: jeedes
ms.openlocfilehash: 30971c9c5280bfba37f46aaab0aaf2c2506502c0
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88542350"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-teamphoria"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Teamphoria

En este tutorial aprenderá a integrar Teamphoria con Azure Active Directory (Azure AD). Al integrar Teamphoria con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Teamphoria.
* Permitir que los usuarios inicien sesión automáticamente en Teamphoria con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Teamphoria.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Teamphoria admite SSO iniciado por **SP**

## <a name="adding-teamphoria-from-the-gallery"></a>Agregar Teamphoria desde la galería

Para configurar la integración de Teamphoria en Azure AD, es preciso agregar Teamphoria desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Teamphoria** en el cuadro de búsqueda.
1. Seleccione **Teamphoria** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-teamphoria"></a>Configuración y prueba del inicio de sesión único de Azure AD para Teamphoria

Configure y pruebe el inicio de sesión único de Azure AD con Teamphoria mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Teamphoria.

Para configurar y probar el inicio de sesión único de Azure AD con Teamphoria, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Teamphoria](#configure-teamphoria-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Teamphoria](#create-teamphoria-test-user)** , para tener un homólogo de B.Simon en Teamphoria que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Teamphoria**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<sub-domain>.teamphoria.com/login`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Teamphoria Client](https://www.teamphoria.com/) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Teamphoria**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Teamphoria mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Teamphoria**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-teamphoria-sso"></a>Configuración del inicio de sesión único de Teamphoria

1. Para automatizar la configuración en Teamphoria, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Teamphoria** para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en Teamphoria. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Teamphoria manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Teamphoria como administrador y lleve a cabo los siguientes pasos:

4. Vaya a la opción **ADMIN SETTINGS** (Configuración de administración) de la barra de herramientas de la izquierda y, en la pestaña Configure (Configurar), haga clic en **SINGLE SIGN-ON** (Inicio de sesión único) para abrir la ventana de configuración de SSO.

    ![Configurar inicio de sesión único](./media/teamphoria-tutorial/admin_sso_configure.png)

5. Haga clic en la opción **ADD NEW IDENTITY PROVIDER** (Agregar nuevo proveedor de identidades) en la esquina superior derecha para abrir el formulario para agregar la configuración de SSO.

    ![Configurar inicio de sesión único](./media/teamphoria-tutorial/add_new_identity_provider.png)

6. Escriba la información en los campos, tal como se describe a continuación:

    ![Configurar inicio de sesión único](./media/teamphoria-tutorial/Teamphoria_sso_save.png)

    a. **DISPLAY NAME**: escriba el nombre para mostrar del complemento en la página de administración.

    b. **BUTTON NAME**: el nombre de la pestaña que se mostrará en la página de inicio de sesión mediante SSO.

    c. **CERTIFICATE**: abra el certificado descargado anteriormente desde Azure Portal con el Bloc de notas, copie el contenido del mismo y péguelo en este cuadro de texto.

    d. **ENTRY POINT**: pegue el valor de **Dirección URL de inicio de sesión** que copió anteriormente de Azure Portal.

    e. Cambie la opción a **ON** y haga clic en **SAVE**.

### <a name="create-teamphoria-test-user"></a>Creación de un usuario de prueba en Teamphoria

Para permitir que los usuarios de Azure AD inicien sesión en Teamphoria, deben aprovisionarse en Teamphoria. En el caso de Teamphoria, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su sitio de la compañía de Teamphoria como administrador.

1. Haga clic en **ADMIN** en la barra de herramientas izquierda y haga clic en la pestaña **MANAGE** en **USERS** para abrir la página de administración para los usuarios.

    ![Agregar empleado](./media/teamphoria-tutorial/admin_manage_users.png)

1. Haga clic en la opción **MANUAL INVITE**.

    ![Invitar a contactos](./media/teamphoria-tutorial/admin_manage_add_users.png)

1. En esta página, realizar las acciones siguientes.

    ![Invitar a contactos](./media/teamphoria-tutorial/manual_user_invite.png)

    a. En el cuadro de texto **EMAIL ADDRESS** (DIRECCIÓN DE CORREO ELECTRÓNICO), escriba la **dirección de correo electrónico** del usuario, en este caso, B.Simon.

    b. En el cuadro de texto **FIRST NAME** (NOMBRE), escriba el nombre de usuario, en este caso, **B**.

    c. En el cuadro de texto **LAST NAME** (APELLIDOS), escriba los apellidos del usuario, en este caso **Simon**.

    d. Haga clic en **INVITE 1 USER**. El usuario debe aceptar la invitación para su creación en el sistema.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Teamphoria en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Teamphoria para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Teamphoria con Azure AD](https://aad.portal.azure.com/)

