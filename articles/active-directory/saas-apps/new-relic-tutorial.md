---
title: 'Tutorial: Integración de Azure Active Directory con New Relic by Account | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y New Relic by Account.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2020
ms.author: jeedes
ms.openlocfilehash: 19a471b9339828250baa190b1de57c86eb22b30c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554471"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-new-relic-by-account"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con New Relic by Account

En este tutorial aprenderá a integrar New Relic by Account con Azure Active Directory (Azure AD). Al integrar New Relic by Account con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a New Relic by Account.
* Permitir que los usuarios inicien sesión automáticamente en New Relic by Account con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en New Relic by Account.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* New Relic by Account admite el inicio de sesión único iniciado por **SP**.

* Una vez configurado New Relic by Account, puede aplicar controles de sesión, que protegen la filtración e infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-new-relic-by-account-from-the-gallery"></a>Incorporación de New Relic by Account desde la galería

Para configurar la integración de New Relic by Account en Azure AD, deberá agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **New Relic by Account** en el cuadro de búsqueda.
1. Seleccione **New Relic by Account** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-new-relic-by-account"></a>Configuración y prueba del inicio de sesión único de Azure AD para New Relic by Account

Configure y pruebe el inicio de sesión único de Azure AD con New Relic by Account mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de New Relic by Account.

Para configurar y probar el inicio de sesión único de Azure AD con New Relic by Account, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en New Relic by Account](#configure-new-relic-by-account-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de New Relic by Account](#create-new-relic-by-account-test-user)** , para tener un homólogo de B.Simon en New Relic by Account vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **New Relic by Account**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)
1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    `https://rpm.newrelic.com/accounts/{acc_id}/sso/saml/login`; asegúrese de sustituir `acc_id` por su propio identificador de cuenta de New Relic by Account.

    b. En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL: `rpm.newrelic.com`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar New Relic by Account**, copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

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

En esta sección va a permitir que B.Simon acceda a New Relic by Account mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **New Relic by Account**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-new-relic-by-account-sso"></a>Configuración del inicio de sesión único en New Relic by Account

1. En otra ventana del explorador web, inicie sesión en el sitio web de empresa de **New Relic by Account** como administrador.

2. En el menú de la parte superior, haga clic en **Configuración de cuenta**.
   
    ![Account Settings (Configuración de cuenta)](./media/new-relic-tutorial/ic797036.png "Configuración de cuenta")

3. Haga clic en la pestaña **Seguridad y autenticación** y, luego, haga clic en la pestaña **Inicio de sesión único**.
   
    ![Inicio de sesión único](./media/new-relic-tutorial/ic797037.png "Inicio de sesión único")

4. En la página de diálogo SAML, realice los pasos siguientes:
   
    ![SAML](./media/new-relic-tutorial/ic797038.png "SAML")
   
    a. Haga clic en **Elegir archivo** para cargar el certificado de Azure Active Directory descargado.

    b. En el cuadro de texto **Dirección URL de inicio de sesión remoto**, pegue el valor de **dirección de inicio de sesión** que copió de Azure Portal.
   
    c. En el cuadro de texto **Logout landing URL** (Dirección URL de cierre de sesión), pegue el valor de **dirección URL de cierre de sesión** que copió de Azure Portal.

    d. Haga clic en **Guardar los cambios**.

### <a name="create-new-relic-by-account-test-user"></a>Creación de un usuario de prueba de New Relic by Account

1. Inicie sesión en el sitio web de empresa de **New Relic by Account** como administrador.

2. En el menú de la parte superior, haga clic en **Configuración de cuenta**.
   
    ![Account Settings (Configuración de cuenta)](./media/new-relic-tutorial/ic797040.png "Configuración de cuenta")

3. En el panel **Cuenta** que se encuentra en el lado izquierdo, haga clic en **Resumen** y, luego, haga clic en **Agregar usuario**.
   
    ![Account Settings (Configuración de cuenta)](./media/new-relic-tutorial/ic797041.png "Configuración de cuenta")

4. En el cuadro de diálogo **Usuarios activos** , realice los pasos siguientes:
   
    ![Usuarios activos](./media/new-relic-tutorial/ic797042.png "Usuarios activos")
   
    a. En el cuadro de texto **Correo electrónico** , escriba la dirección de correo electrónico de un usuario válido de Azure Active Directory que quiera aprovisionar.

    b. Como **Rol**, seleccione **Usuario**.

    c. Haga clic en **Agregar este usuario**.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de New Relic by Account ofrecida por New Relic by Account para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de New Relic by Account en el Panel de acceso, debería iniciar sesión automáticamente en la versión de New Relic by Account para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe New Relic by Account con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)