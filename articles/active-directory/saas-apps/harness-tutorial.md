---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Harness | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Harness.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/02/2019
ms.author: jeedes
ms.openlocfilehash: cfa81b52e8479851fc22b12cbc96a9507f0b2dc7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551396"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-harness"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Harness

En este tutorial, obtendrá información sobre cómo integrar Harness con Azure Active Directory (Azure AD). Al integrar Harness con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Harness.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Harness con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Harness

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Harness admite el SSO iniciado por **SP e IDP**

## <a name="adding-harness-from-the-gallery"></a>Incorporación de Harness desde la galería

Para configurar la integración de Harness en Azure AD, será preciso que agregue Harness desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Harness** en el cuadro de búsqueda.
1. Seleccione **Harness** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-harness"></a>Configuración y prueba del inicio de sesión único de Azure AD para Harness

Configure y pruebe el inicio de sesión único de Azure AD con Harness mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Harness.

Para configurar y probar el inicio de sesión único de Azure AD con Harness, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Harness](#configure-harness-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Harness](#create-harness-test-user)** , para tener un homólogo de B. Simon en Harness que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Harness** , busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://app.harness.io/gateway/api/users/saml-login?accountId=<harness_account_id>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://app.harness.io/`

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Obtendrá la dirección URL de respuesta real en la sección **Configuración del inicio de sesión único de Harness**, que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Harness**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a Harness mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Harness**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-harness-sso"></a>Configuración del inicio de sesión único de Harness

1. Para automatizar la configuración en Harness, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Harness** para ir a la aplicación Harness. En ella, escriba las credenciales de administrador para iniciar sesión en Harness. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Harness manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Harness como administrador y haga lo siguiente:

4. En la parte superior derecha de la página, haga clic en **Continuous Security (Seguridad continua)**  > **Access Management (Administración de acceso)**  > **Authentication Settings (Configuración de autenticación)** .

    ![Configuración de Harness](./media/harness-tutorial/configure01.png)

5. En la sección **SSO Providers** (Proveedores de inicio de sesión único), haga clic en **+ Add SSO Providers (+ Agregar proveedores de inicio de sesión único)**  > **SAML**.

    ![Configuración de Harness](./media/harness-tutorial/configure03.png)

6. En el menú desplegable **SAML Provider** (Proveedor de SAML), lleve a cabo estos pasos:

    ![Configuración de Harness](./media/harness-tutorial/configure02.png)

    a. Copie la instancia de **In your SSO Provider, please enable SAML-based login, then enter the following URL** (En su proveedor de SSO, habilite el inicio dirección basado en SAML y, después, escriba la siguiente dirección URL de respuesta) y péguelo en el cuadro de texto Dirección URL de respuesta en **Configuración básica de SAML** en Azure Portal.

    b. En el cuadro de texto **Nombre para mostrar**, escriba el nombre para mostrar.

    c. Haz clic en **Choose File** (Elegir archivo) para cargar el archivo XML de metadatos que haya descargado desde Azure AD.

    d. Haga clic en **ENVIAR**.

### <a name="create-harness-test-user"></a>Creación de un usuario de prueba de Harness

Para permitir que los usuarios de Azure AD inicien sesión en Harness, deben aprovisionarse en Harness. En Harness, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en Harness.

1. En la parte superior derecha de la página, haga clic en **Continuous Security (Seguridad continua)**  > **Access Management (Administración de acceso)**  > **Users (Usuarios)** .

    ![Configuración de Harness](./media/harness-tutorial/configure04.png)

1. En el lado derecho de la página, haga clic en **+ Add User** (Agregar usuario).

    ![Configuración de Harness](./media/harness-tutorial/configure05.png)

1. En el elemento emergente **Add User** (Agregar usuario), lleve a cabo los pasos siguientes:

    ![Configuración de Harness](./media/harness-tutorial/configure06.png)

    a. En el cuadro de texto **Email Address(es)** (Direcciones de correo electrónico), escriba el correo electrónico del usuario; por ejemplo, `B.simon@contoso.com`.

    b. Seleccione su **User Groups** (Grupos de usuarios).

    c. Haga clic en **Enviar**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Harness en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Harness para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Harness con Azure AD](https://aad.portal.azure.com/)

