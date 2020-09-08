---
title: 'Tutorial: integración de Azure Active Directory con Way We Do | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Way We Do.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/20/2019
ms.author: jeedes
ms.openlocfilehash: 310a42d25ce7fb7970777e8f36abbbee562ab01d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88523897"
---
# <a name="tutorial-integrate-way-we-do-with-azure-active-directory"></a>Tutorial: Integración de Way We Do con Azure Active Directory

En este tutorial, aprenderá a integrar Way We Do con Azure Active Directory (Azure AD). Al integrar Way We Do con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Way We Do.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Way We Do con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede obtener la evaluación gratuita de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción habilitada para el inicio de sesión único (SSO) de Way We Do.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Way We Do admite el inicio de sesión único iniciado por **SP**.
* Way We Do admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-way-we-do-from-the-gallery"></a>Incorporación de Way We Do desde la galería

Para configurar la integración de Way We Do en Azure AD, deberá agregar Way We Do desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Way We Do** en el cuadro de búsqueda.
1. Seleccione **Way We Do** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Way We Do mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Way We Do.

Para configurar y probar el inicio de sesión único de Azure AD con Way We Do, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración del inicio de sesión único en Way We Do](#configure-way-we-do-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Way We Do](#create-way-we-do-test-user)** : para tener un homólogo de Britta Simon en Way We Do que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Way We Do**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.waywedo.com/Authentication/ExternalSignIn`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.waywedo.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Way We Do](mailto:support@waywedo.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (sin procesar)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificateraw.png)

1. En la sección **Set up Way We Do** (Configurar Way We Do), copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-way-we-do-sso"></a>Configuración del inicio de sesión único de Way We Do

1. Para automatizar la configuración de Way We Do, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Install the extension** (Instalar la extensión).

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Way We Do** para ir a la aplicación Way We Do. En ella, escriba las credenciales de administrador para iniciar sesión en Way We Do. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar Way We Do manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Way We Do como administrador y haga lo siguiente:

1. Haga clic en el **icono de persona** en la esquina superior derecha de cualquier página de Way We Do y, a continuación, haga clic en **Account** (Cuenta) en el menú desplegable.

    ![Cuenta de Way We Do](./media/waywedo-tutorial/tutorial_waywedo_account.png)

1. Haga clic en el **icono de menú** para abrir el menú de navegación de inserción y haga clic en **Single Sign On** (Inicio de sesión único).

    ![Way We Do único](./media/waywedo-tutorial/tutorial_waywedo_single.png)

1. En la página **Single sign-on setup** (Configuración de inicio de sesión único), siga estos pasos:

    ![Way We Do guardar](./media/waywedo-tutorial/tutorial_waywedo_save.png)

    a. Haga clic en el botón de alternar de **Turn on single sign-on** (Activar inicio de sesión único) para cambiar a **Sí** y habilitar el inicio de sesión único.

    b. En el cuadro de texto **Single sign-on name**  (Nombre de inicio de sesión único), escriba su nombre.

    c. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    d. En el cuadro de texto **SAML SSO URL** (Dirección URL de inicio de sesión único de SAML), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

    e. Para cargar el certificado, haga clic en el botón **select** (seleccionar) situado junto a **Certificate** (Certificado).

    f. **Configuración opcional** -
    
    * Enable Passwords (Habilitar las contraseñas): cuando esta opción está deshabilitada, la contraseña normal funciona para Way We Do de modo que los usuarios solo pueden usar el inicio de sesión único.

    * Enable Auto-provisioning (Habilitar aprovisionamiento automático): cuando esta opción está habilitada, la dirección de correo electrónico usada para el inicio de sesión se compara automáticamente con la lista de los usuarios de Way We Do. Si la dirección de correo electrónico no coincide con un usuario activo de Way We Do, agrega automáticamente una nueva cuenta de usuario para la persona que inicia sesión y se solicita la información que falta.

      > [!NOTE]
      > Los usuarios agregados mediante el inicio de sesión único se agregan como usuarios generales y no se les asigna ningún rol en el sistema. Un administrador puede modificar su rol de seguridad como editor o administrador, y también puede asignar uno o varios roles de organigrama.

    g. Haga clic en **Guardar** para guardar la configuración.

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

En esta sección, va a permitir que B.Simon acceda a Way We Do utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Way We Do**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-way-we-do-test-user"></a>Creación de un usuario de prueba de Way We Do

En esta sección, se crea un usuario llamado a Britta Simon en Way We Do. Way We Do admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Way We Do, se crea después de la autenticación.

> [!Note]
> Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de Way We Do](mailto:support@waywedo.com).

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Way We Do en el panel de acceso y debería iniciar sesión automáticamente en la versión de Way We Do para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)