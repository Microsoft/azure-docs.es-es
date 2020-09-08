---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Helpshift | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Helpshift.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/20/2019
ms.author: jeedes
ms.openlocfilehash: fb9c776061407669f27f61ac4bc536b859d04b66
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550628"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-helpshift"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Helpshift

En este tutorial, aprenderá a integrar Helpshift con Azure Active Directory (Azure AD). Al integrar Helpshift con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Helpshift.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Helpshift con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Helpshift.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Helpshift admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-helpshift-from-the-gallery"></a>Adición de Helpshift desde la galería

Para configurar la integración de Helpshift en Azure AD, será preciso que agregue Helpshift desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Helpshift** en el cuadro de búsqueda.
1. Seleccione **Helpshift** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-helpshift"></a>Configuración y prueba del inicio de sesión único de Azure AD para Helpshift

Configure y pruebe el inicio de sesión único de Azure AD con Helpshift mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Helpshift.

Para configurar y probar el inicio de sesión único de Azure AD con Helpshift, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Helpshift](#configure-helpshift-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en Helpshift](#create-helpshift-test-user)** : para tener un homólogo de B.Simon en Helpshift que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Helpshift**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<YourDOMAIN>.helpshift.com/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<YourDOMAIN>.helpshift.com/login/saml/acs/`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    d. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<YourDOMAIN>.helpshift.com/login/saml/idp-login/`

    e. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://<YourDOMAIN>.helpshift.com/`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de respuesta, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico de Helpshift](mailto:support@helpshift.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Helpshift**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Helpshift mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Helpshift**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-helpshift-sso"></a>Configuración del inicio de sesión único de Helpshift

1. En otra ventana del explorador web, inicie sesión en la aplicación Helpshift como administrador.

1. Abra el **Panel** de Helpshift y haga clic en el **icono de configuración**.

    ![Configuración de Helpshift](./media/helpshift-tutorial/configuration01.png)

1. Haga clic en la pestaña **Integrations** (Integraciones) y realice los pasos siguientes:

    ![Configuración de Helpshift](./media/helpshift-tutorial/configuration02.png)

    a. Active **Single Sign-On(SAML – SSO)** (Inicio de sesión único [SAML: SSO]).

    b. Seleccione **Identity Provider (IDP)** (Proveedor de identidades [IDP]) como **Azure Active Directory**.

    c. En el cuadro de texto **SAML 2.0 Endpoint URL** (Dirección URL del punto de conexión de SAML 2.0), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. Abra el archivo **Certificado (Base 64)** descargado en el Bloc de notas, copie el contenido del archivo (sin usar las líneas "—–BEGIN CERTIFICATE—–" y "—–END CERTIFICATE—–") y péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509).

    e. En el cuadro de texto **Issuer URL** (Dirección URL del emisor), pegue el valor del **Identificador de Azure AD** que ha copiado de Azure Portal.

    f. Haga clic en **APPLY CHANGES** (Aplicar cambios).

### <a name="create-helpshift-test-user"></a>Creación de un usuario de prueba de Helpshift

En esta sección, va a crear un usuario llamado B.Simon en Helpshift. Colabore con el  [equipo de soporte técnico de Helpshift](mailto:support@helpshift.com) para agregar los usuarios a la plataforma de Helpshift. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Helpshift en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Helpshift para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Helpshift con Azure AD](https://aad.portal.azure.com/).