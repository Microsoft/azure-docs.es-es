---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ThousandEyes | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ThousandEyes.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/04/2019
ms.author: jeedes
ms.openlocfilehash: 2b99d5ea33693431fb3811af50385891684a366e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546464"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-thousandeyes"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ThousandEyes

En este tutorial aprenderá a integrar ThousandEyes con Azure Active Directory (Azure AD). Al integrar ThousandEyes con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a ThousandEyes.
* Permitir que los usuarios inicien sesión automáticamente en ThousandEyes con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en ThousandEyes.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ThousandEyes admite el inicio de sesión único iniciado por **SP e IDP**.
* ThousandEyes admite el [**aprovisionamiento** automático de usuarios](https://docs.microsoft.com/azure/active-directory/saas-apps/thousandeyes-provisioning-tutorial)

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-thousandeyes-from-the-gallery"></a>Agregar ThousandEyes desde la galería

Para configurar la integración de ThousandEyes en Azure AD, será preciso que agregue ThousandEyes desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ThousandEyes** en el cuadro de búsqueda.
1. Seleccione **ThousandEyes** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-thousandeyes"></a>Configuración y prueba del inicio de sesión único de Azure AD para ThousandEyes

Configure y pruebe el inicio de sesión único de Azure AD con ThousandEyes mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de ThousandEyes.

Para configurar y probar el inicio de sesión único de Azure AD con ThousandEyes, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de ThousandEyes](#configure-thousandeyes-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de ThousandEyes](#create-thousandeyes-test-user)** , para tener un homólogo de B.Simon en ThousandEyes que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ThousandEyes**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón **Guardar**.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://app.thousandeyes.com/login/sso`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar ThousandEyes**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a ThousandEyes mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ThousandEyes**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-thousandeyes-sso"></a>Configuración del inicio de sesión único de ThousandEyes

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de **ThousandEyes** como administrador.

2. En el menú de la parte superior, haga clic en **Configuración**.

    ![Configuración](./media/thousandeyes-tutorial/ic790066.png "Configuración")

3. Haga clic en **Cuenta**

    ![Cuenta](./media/thousandeyes-tutorial/ic790067.png "Cuenta")

4. Haga clic en la pestaña **Security & Authentication** (Seguridad y autenticación).

    ![Seguridad y autenticación](./media/thousandeyes-tutorial/ic790068.png "Seguridad y autenticación")

5. En la sección **Configurar inicio de sesión único** siga los pasos siguientes:

    ![Configurar inicio de sesión único](./media/thousandeyes-tutorial/ic790069.png "Configurar inicio de sesión único")

    a. Seleccione **Enable Single Sign-On**(Habilitar inicio de sesión único).

    b. En el cuadro de texto **Login Page URL** (Dirección URL de la página de inicio de sesión), pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Logout Page URL** (Dirección URL de la página de cierre de sesión), pegue la **dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Emisor de proveedor de identidades**, pegue el **Identificador de Azure AD** que ha copiado de Azure Portal.

    e. En **Certificado de verificación**, haga clic en **Elegir archivo** y cargue el certificado que ha descargado de Azure Portal.

    f. Haga clic en **Save**(Guardar).

### <a name="create-thousandeyes-test-user"></a>Creación de un usuario de prueba de ThousandEyes

El objetivo de esta sección es crear un usuario llamado Britta Simon en ThousandEyes. ThousandEyes admite el aprovisionamiento automático de usuarios, que está habilitado de forma predeterminada. [Aquí](thousandeyes-provisioning-tutorial.md) puede encontrar más información sobre cómo configurar el aprovisionamiento automático de usuarios.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. Inicie sesión en su sitio de la compañía de ThousandEyes como administrador.

2. Haga clic en **Configuración**.

    ![Configuración](./media/thousandeyes-tutorial/IC790066.png "Configuración")

3. Haga clic en **Cuenta**.

    ![Cuenta](./media/thousandeyes-tutorial/IC790067.png "Cuenta")

4. Haga clic en la pestaña **Accounts & Users** (Cuentas y usuarios).

    ![Cuentas y usuarios](./media/thousandeyes-tutorial/IC790073.png "Cuentas y usuarios")

5. En la sección **Add Users & Accounts** (Agregar usuarios y cuentas), realice los siguientes pasos:

    ![Agregar cuentas de usuario](./media/thousandeyes-tutorial/IC790074.png "Agregar cuentas de usuario")

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de un usuario; por ejemplo, **B.Simon**.

    b. En el cuadro de texto **Correo electrónico**, escriba el correo electrónico del usuario, en el ejemplo b.simon@contoso.com.

    b. Haga clic en **Agregar nuevo usuario a la cuenta**.

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo electrónico con un vínculo para confirmar la cuenta y activarla.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de ThousandEyes ofrecida por ThousandEyes para aprovisionar cuentas de usuario de Azure Active Directory.


## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de ThousandEyes en el panel de acceso y debería iniciar sesión automáticamente en la versión de ThousandEyes para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe ThousandEyes con Azure AD](https://aad.portal.azure.com/)

- [Configuración del aprovisionamiento de usuarios](https://docs.microsoft.com/azure/active-directory/saas-apps/thousandeyes-provisioning-tutorial)