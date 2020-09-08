---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Watch by Colors | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Watch by Colors.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/09/2019
ms.author: jeedes
ms.openlocfilehash: 79fd6c5212b2ad956259f2cfbfb36ec3e7e92844
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88523825"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-watch-by-colors"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Watch by Colors

En este tutorial, obtendrá información sobre cómo integrar Watch by Colors con Azure Active Directory (Azure AD). Al integrar Watch by Colors con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Watch by Colors.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Watch by Colors con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Watch by Colors.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Watch by Colors admite el inicio de sesión único iniciado por **SP e IDP**

## <a name="adding-watch-by-colors-from-the-gallery"></a>Adición de Watch by Colors desde la galería

Para configurar la integración de Watch by Colors en Azure AD, será preciso que agregue Watch by Colors desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Watch by Colors** en el cuadro de búsqueda.
1. Seleccione **Watch by Colors** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-watch-by-colors"></a>Configuración y prueba del inicio de sesión único de Azure AD para Watch by Colors

Configure y pruebe el inicio de sesión único de Azure AD con Watch by Colors mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Watch by Colors.

Para configurar y probar el inicio de sesión único de Azure AD con Watch by Colors, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Watch by Colors](#configure-watch-by-colors-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Watch by Colors](#create-watch-by-colors-test-user)** , para tener un homólogo de B. Simon en Watch by Colors vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Watch by Colors**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada en el modo iniciado por **IDP** y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón **Guardar**.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://app.colorscorporation.com/login`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección, va a permitir que B. Simon acceda a Watch by Colors mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Watch by Colors**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-watch-by-colors-sso"></a>Configuración del inicio de sesión único en Watch by Colors

1. Para automatizar la configuración en Watch by Colors, es preciso instalar la **extensión de inicio de sesión seguro de Mis aplicaciones** , para lo que es preciso hacer clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Watch by Colors** para ir a la aplicación Watch by Colors. En ella, escriba las credenciales de administrador para iniciar sesión en Watch by Colors. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

3. Si desea configurar Watch by Colors de forma manual, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Watch by Colors como administrador y haga lo siguiente:

4. En la esquina superior derecha de la página, haga clic en su **perfil** > **Account Settings** (Configuración de cuenta)  > **SSO (Single Sign On)** [SSO (Inicio de sesión único)].

    ![Configuración de Watch by Colors](./media/watch-by-colors-tutorial/config01.png)

5. Siga estos pasos en la página **SSO (Single Sign On)** [SSO (Inicio de sesión único)]:

    ![Configuración de Watch by Colors](./media/watch-by-colors-tutorial/config02.png)

    a. Alterne la configuración de la opción **Habilitar SAML** en **Activada**.

    b. En el cuadro de texto **Dirección URL**, pegue la **Dirección URL de metadatos de federación** que ha copiado de Azure Portal.

    c. Haga clic en **Importar** y, a continuación, los siguientes campos se rellenan automáticamente de forma automática en la página.

    d. Haga clic en **Save**(Guardar).

### <a name="create-watch-by-colors-test-user"></a>Creación de un usuario de prueba de Watch by Colors

Para permitir que los usuarios de Azure AD inicien sesión en Watch by Colors, tienen que aprovisionarse en Watch by Colors. En Watch by Colors, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Watch by Colors como administrador de seguridad.

1. En la esquina superior derecha de la página, haga clic en su **perfil** > **Usuarios** > **Agregar usuario**.

    ![Configuración de Watch by Colors](./media/watch-by-colors-tutorial/config03.png)

1. En la página **User Details** (Detalles del usuario), siga estos pasos:

    ![Configuración de Watch by Colors](./media/watch-by-colors-tutorial/config04.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **B**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

    c. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario, por ejemplo, `B.Simon@contoso.com`.

    d. En el cuadro de texto **Password** (Contraseña), escriba su contraseña.

    e. Seleccione **Account Permissions** (Permisos de cuenta) en función de su organización.

    f. Haga clic en **Save**(Guardar).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Watch by Colors en el Panel de acceso y debería iniciar sesión automáticamente en la versión de Watch by Colors para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Probar Watch by Colors con Azure AD](https://aad.portal.azure.com/)

