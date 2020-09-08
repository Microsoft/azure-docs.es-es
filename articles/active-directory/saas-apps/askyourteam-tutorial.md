---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con AskYourTeam | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y AskYourTeam.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/28/2020
ms.author: jeedes
ms.openlocfilehash: ad68208f8993ce84c71c97d97de7e2b090f1e7da
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549430"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-askyourteam"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con AskYourTeam

En este tutorial aprenderá a integrar AskYourTeam con Azure Active Directory (Azure AD). Al integrar AskYourTeam con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a AskYourTeam.
* Permitir que los usuarios inicien sesión automáticamente en AskYourTeam con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en AskYourTeam.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* AskYourTeam admite el inicio de sesión único iniciado por **SP e IDP**.
* Una vez configurado AskYourTeam, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-askyourteam-from-the-gallery"></a>Incorporación de AskYourTeam desde la galería

Para configurar la integración de AskYourTeam en Azure AD, será preciso que agregue AskYourTeam desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **AskYourTeam** en el cuadro de búsqueda.
1. Seleccione **AskYourTeam** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-askyourteam"></a>Configuración y prueba del inicio de sesión único de Azure AD para AskYourTeam

Configure y pruebe el inicio de sesión único de Azure AD con AskYourTeam utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de AskYourTeam.

Para configurar y probar el inicio de sesión único de Azure AD con AskYourTeam, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en AskYourTeam](#configure-askyourteam-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de AskYourTeam](#create-askyourteam-test-user)** , para tener un homólogo de B.Simon en AskYourTeam que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **AskYourTeam**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY>.app.askyourteam.com/users/auth/saml/callback`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY>.app.askyourteam.com/login`

    > [!NOTE]
    > Estos valores no son reales. La actualización de estos valores por los valores reales de dirección URL de respuesta y dirección URL de inicio de sesión se explica más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar AskYourTeam**, copie las direcciones URL adecuadas en función de sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a AskYourTeam mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **AskYourTeam**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-askyourteam-sso"></a>Configuración del inicio de sesión único en AskYourTeam

1. En otra ventana del explorador web, inicie sesión en el sitio web de AskYourTeam como administrador.

1. Haga clic en **My Organisation** (Mi organización).

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/user1.png)

1. Haga clic en **Integrations** (Integraciones).

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/configure1.png)

1. Haga clic en **Edit Settings** (Editar configuración).

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/configure2.png)

1. En la página **Edit Single Sign-On Integration** (Editar la integración de inicio de sesión único), siga estos pasos: 

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/configure3.png)

    a. En el cuadro de texto **SAML Single Sign-On Service URL** (Dirección URL del servicio de inicio de sesión único de SAML), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **SAML Entity ID** (Id. de entidad SAML), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Sign-Out URL** (URL de cierre de sesión), pegue el valor de **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    d. Abra en el Bloc de notas el archivo de **certificado (base64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **SAML Signing Certificate - Base64** (Certificado de firma de SAML - Base64).

    > [!NOTE]
    > También puede cargar el archivo **XML de metadatos de federación** haciendo clic en la opción **Choose File** (Elegir archivo).

    e. Copie el valor de **Reply URL (Assertion Consumer Service URL)** (Dirección URL de respuesta [URL del Servicio de consumidor de aserciones]) y péguelo en el cuadro **URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    f. Copie el valor de **Sign on URL** (URL de inicio de sesión) y péguelo en el cuadro de texto **URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.

    g. Haga clic en **Save**(Guardar).

### <a name="create-askyourteam-test-user"></a>Creación de un usuario de prueba de AskYourTeam

1. En otra ventana del explorador web, inicie sesión en el sitio web de AskYourTeam como administrador.

1. Haga clic en **My Organisation** (Mi organización).

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/user1.png)

1. Haz clic en **Users** (Usuarios) y seleccione **New User** (Nuevo usuario).

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/user2.png)

1. En la sección **New user** (Nuevo usuario), lleve a cabo estos pasos:

    ![Configuración de AskYourTeam](./media/askyourteam-tutorial/user3.png)

    1. En el cuadro de texto **First name** (Nombre), escriba el nombre de usuario.

    1. En el cuadro de texto **Last name** (Apellidos), escriba los apellidos del usuario.

    1. En el cuadro de texto **Email** (Correo electrónico), escriba la dirección de correo electrónico del usuario, como B.Simon@contoso.com.

    1. Seleccione el valor de **Rol** para el usuario según el requisito de su organización.

    1. Haga clic en **Save**(Guardar).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de AskYourTeam en el Panel de acceso, debería iniciar sesión automáticamente en la versión de AskYourTeam para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe AskYourTeam con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
