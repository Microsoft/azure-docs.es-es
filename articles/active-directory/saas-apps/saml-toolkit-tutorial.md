---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Azure AD SAML Toolkit | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Azure AD SAML Toolkit.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/24/2020
ms.author: jeedes
ms.openlocfilehash: aa37cef84bb1d2cb92f2bb0e4a227c5be60fa345
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543421"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-azure-ad-saml-toolkit"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Azure AD SAML Toolkit

En este tutorial, aprenderás a integrar Azure AD SAML Toolkit en Azure Active Directory (Azure AD). Al integrar Azure AD SAML Toolkit en Azure AD, puedes:

* Controlar en Azure AD quién tiene acceso a Azure AD SAML Toolkit.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Azure AD SAML Toolkit con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en Azure AD SAML Toolkit.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Azure AD SAML Toolkit admite el inicio de sesión único iniciado por **SP**.
* Una vez configurado Azure AD SAML Toolkit, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-azure-ad-saml-toolkit-from-the-gallery"></a>Adición de Azure AD SAML Toolkit desde la galería

Para configurar la integración de Azure AD SAML Toolkit en Azure AD, será preciso que agregues Azure AD SAML Toolkit desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escribe **Azure AD SAML Toolkit** en el cuadro de búsqueda.
1. Selecciona **Azure AD SAML Toolkit** en el panel de resultados y agrega la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-azure-ad-saml-toolkit"></a>Configuración y prueba del inicio de sesión único de Azure AD para Azure AD SAML Toolkit

Configura y prueba el inicio de sesión único de Azure AD con Azure AD SAML Toolkit usando una usuaria de prueba llamada **B. Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Azure AD SAML Toolkit.

Para configurar y probar el inicio de sesión único de Azure AD con Azure AD SAML Toolkit, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Azure AD SAML Toolkit](#configure-azure-ad-saml-toolkit-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Azure AD SAML Toolkit](#create-azure-ad-saml-toolkit-test-user)** , para tener un homólogo de B.Simon en Azure AD SAML Toolkit que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Azure AD SAML Toolkit**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://samltoolkit.azurewebsites.net/`

    b. En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL: `https://samltoolkit.azurewebsites.net`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL: `https://samltoolkit.azurewebsites.net/SAML/Consume`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores de la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales, que se explican más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (sin procesar)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

1. En la sección **Configurar Azure AD SAML Toolkit**, copia las direcciones URL adecuadas según tus necesidades.

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

En esta sección, vas a permitir que B. Simon use el inicio de sesión único de Azure al concederle acceso a Azure AD SAML Toolkit.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, selecciona **Azure AD SAML Toolkit**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-azure-ad-saml-toolkit-sso"></a>Configurar el SSO en Azure AD SAML Toolkit

1. Abre una nueva ventana del explorador web, si no te has registrado en el sitio web de Azure AD SAML Toolkit, primero haz clic en **Register** (Registrarse) para registrarte. Si ya te has registrado, inicia sesión en el sitio de la compañía de Azure AD SAML Toolkit con las credenciales de inicio de sesión registradas.

    ![Registro en Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/register.png)

1. Ve a la **SAML Configuration** (Configuración de SAML).

    ![Configurar SAML de Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/saml-configure.png)

1. Haga clic en **Crear**.

    ![Crear SSO de Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/createsso.png)

1. En la página **SAML SSO Configuration** (Configuración SAML de SSO), sigue los pasos a continuación:

    ![Crear SSO de Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/fill-details.png)

    1. En el cuadro de texto **Login URL** (Dirección URL de inicio de sesión), pega el valor de **Dirección URL de inicio de sesión** que copiaste de Azure Portal.

    1. En el cuadro de texto **IdP entityID** (Id. de entidad de IdP), pega el valor de **Identificador de Azure AD** que copiaste de Azure Portal.

    1. En el cuadro de texto **URL de cierre de sesión**, pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal.

    1. Haga clic en **Choose File** (Elegir archivo) para cargar el archivo de **Certificate (Raw)** (Certificado [sin procesar]) que ha descargado desde Azure Portal.

    1. Haga clic en **Crear**.

    1. Copie los valores de dirección URL de inicio de sesión, identificador y dirección URL de ACS de la página de configuración de inicio de sesión único del kit de herramientas de SAML y péguelo en los cuadros de texto correspondientes de la **sección Configuración básica de SAML** en Azure Portal.

### <a name="create-azure-ad-saml-toolkit-test-user"></a>Crear un usuario de prueba de Azure AD SAML Toolkit

En esta sección, se crea una usuaria llamada B. Simon en Azure AD SAML Toolkit. Cree un usuario de prueba en la herramienta; para ello, registre un nuevo usuario y proporcione todos sus detalles. 

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Azure AD SAML Toolkit en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Azure AD SAML Toolkit para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Azure AD SAML Toolkit con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Azure AD SAML Toolkit con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/protect-azure)