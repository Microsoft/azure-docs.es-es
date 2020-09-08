---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con 8x8 | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y 8x8.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/20/2020
ms.author: jeedes
ms.openlocfilehash: 52b45bf7457a81b605df91e63a7c6ad539309dd8
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88538661"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-8x8"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con 8x8

En este tutorial, aprenderá a integrar 8x8 con Azure Active Directory (Azure AD). Al integrar 8x8 con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a 8x8.
* Permitir que los usuarios inicien sesión automáticamente en 8x8 con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de 8x8.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* 8x8 admite el inicio de sesión único iniciado por **SP e IDP**.

* Una vez configurado 8x8, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-8x8-from-the-gallery"></a>Adición de 8x8 desde la galería

Para configurar la integración de 8x8 en Azure AD, será preciso agregar 8x8 desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **8x8** en el cuadro de búsqueda.
1. Seleccione **8x8** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-8x8"></a>Configuración y prueba del inicio de sesión único de Azure AD para 8x8

Configure y pruebe el inicio de sesión único de Azure AD con 8x8 mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de 8x8.

Para configurar y probar el inicio de sesión único de Azure AD con 8x8, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en 8x8](#configure-8x8-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de 8x8](#create-8x8-test-user)** : para tener un homólogo de B.Simon en 8x8 vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **8x8**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL: `https://sso.8x8.com/saml2`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL: `https://sso.8x8.com/saml2`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo. Utilizará el certificado más adelante en este tutorial en la sección **Configuración del inicio de sesión único de 8x8**.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar 8x8**, copie las direcciones URL, ya que utilizará estos valores más adelante en el tutorial.

    ![Copiar direcciones URL de configuración](./media/8x8virtualoffice-tutorial/copy-configuration-urls.png)

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

En esta sección, va a permitir que B.Simon acceda a 8x8 mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **8x8**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-8x8-sso"></a>Configuración del inicio de sesión único de 8x8

La siguiente parte del tutorial depende del tipo de suscripción que tenga de 8x8.

* En el caso de los clientes de 8x8 Editions y X Series que utilizan Configuration Manager para la administración, consulte [Configuración de Configuration Manager de 8x8](#configure-8x8-configuration-manager).
* En el caso de los clientes de Virtual Office que utilizan Account Manager para la administración, consulte [Configuración de Account Manager de 8x8](#configure-8x8-account-manager).

### <a name="configure-8x8-configuration-manager"></a>Configuración de Configuration Manager de 8x8

1. Para automatizar la configuración en 8x8, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Set up 8x8** (Configurar 8x8) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en 8x8. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

1. Si desea configurar 8x8 manualmente, inicie sesión en el [administrador de configuración](https://vo-cm.8x8.com/) de 8x8 como administrador.

1. En la página principal, haga clic en **Identity Management** (Administración de identidades).

    ![8x8 Configuration Manager](./media/8x8virtualoffice-tutorial/configure1.png)

1. Active **Single Sign On (SSO)** (Inicio de sesión único [SSO]) y, a continuación, seleccione **Microsoft Azure AD**.

    ![8x8 Configuration Manager](./media/8x8virtualoffice-tutorial/configure2.png)

1. Copie las tres direcciones URL y el certificado de firma de la página **Configurar inicio de sesión único con SAML** de Azure AD en la sección **Microsoft Azure AD SAML Settings** (Configuración de SAML de Microsoft Azure AD) de Configuration Manager de 8x8.

    ![8x8 Configuration Manager](./media/8x8virtualoffice-tutorial/configure3.png)

    a. Copie **Dirección URL de inicio de sesión** en **IDP Login URL** (Dirección URL de inicio de sesión de IDP).

    b. Copie **Identificador de Azure AD** en **IDP Issuer URL/URN** (URL/URN del emisor de IDP).

    c. Copie **Dirección URL de cierre de sesión** en **IDP Logout URL** (Dirección URL de cierre de sesión de IDP).

    d. Descargue el **Certificado (Base64)** y cárguelo en **Certificate** (Certificado).

    e. Haga clic en **Save**(Guardar).

### <a name="configure-8x8-account-manager"></a>Configuración de Account Manager de 8x8

1. Inicie la sesión en el inquilino de 8x8 Virtual Office como administrador.

1. Seleccione el **administrador de cuentas de Virtual Office** en el panel de la aplicación.

    ![Configurar en la aplicación](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_001.png)

1. Seleccione la cuenta **empresarial** que se va a administrar y haga clic en el botón **Iniciar sesión**.

    ![Configurar en la aplicación](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_002.png)

1. Haga clic en la pestaña **CUENTAS** en la lista de menús.

    ![Configurar en la aplicación](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_003.png)

1. Haga clic en **Inicio de sesión único** en la lista de cuentas.

    ![Configurar en la aplicación](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_004.png)

1. Seleccione **Inicio de sesión único** en los método de autenticación y haga clic en **SAML**.

    ![Configurar en la aplicación](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_005.png)

1. En la sección **Inicio de sesión único de SAML**, siga estos pasos:

    ![Configurar en la aplicación](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_006.png)

    a. En el cuadro de texto **URL de inicio de sesión**, pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Sign Out URL** (URL de cierre de sesión), pegue el valor de la **URL de cierre de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **URL del emisor**, pegue el valor del **identificador de Azure AD** que ha copiado de Azure Portal.

    d. Haga clic en el botón **Examinar** para cargar el certificado que descargó de Azure Portal.

    e. Haga clic en el botón **Save** (Guardar).

### <a name="create-8x8-test-user"></a>Creación de un usuario de prueba de 8x8

En esta sección, creará un usuario llamado Britta Simon en 8x8. Trabaje con el [equipo de soporte técnico de 8x8](https://www.8x8.com/about-us/contact-us) para agregar los usuarios en la plataforma de 8x8. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de 8x8 en el panel de acceso, debería iniciar sesión automáticamente en la instancia de 8x8 para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de 8x8 con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de 8x8 con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)