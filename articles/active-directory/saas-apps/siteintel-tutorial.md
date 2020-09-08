---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SiteIntel | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SiteIntel.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/02/2020
ms.author: jeedes
ms.openlocfilehash: f681dd2931300ec00fd6388b4636015c87f38170
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88525103"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-siteintel"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con SiteIntel

En este tutorial, aprenderá a integrar SiteIntel con Azure Active Directory (Azure AD). Al integrar SiteIntel con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a SiteIntel.
* Permitir que los usuarios puedan iniciar sesión automáticamente en SiteIntel con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central, Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en SiteIntel.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SiteIntel admite el inicio de sesión único iniciado por SP e IDP.
* Una vez configurado SiteIntel, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-siteintel-from-the-gallery"></a>Incorporación de SiteIntel desde la galería

Para configurar la integración de SiteIntel en Azure AD, deberá agregar SiteIntel desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel izquierdo, seleccione **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En el cuadro **Agregar desde la galería**, escriba **SiteIntel**.
1. En la lista de resultados, seleccione **SiteIntel** y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-siteintel"></a>Configuración y prueba del inicio de sesión único de Azure AD para SiteIntel

Configure y pruebe el inicio de sesión único de Azure AD con SiteIntel mediante un usuario de prueba llamado *B.Simon*. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de SiteIntel.

Para configurar y probar el inicio de sesión único de Azure AD con SiteIntel, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.  

    a. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD con el usuario B.Simon.  

    b. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para que el usuario B.Simon pueda usar el inicio de sesión único de Azure AD.

1. **[Configuración del inicio de sesión único en SiteIntel](#configure-siteintel-sso)** , para configurar los valores de inicio de sesión único en la aplicación.

    * **[Creación de un usuario de prueba de SiteIntel](#create-a-siteintel-test-user)** para tener un homólogo de B.Simon en SiteIntel vinculado a la representación del usuario en Azure AD.

1. **[Prueba del inicio de sesión único](#test-sso)** para comprobar que la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Para habilitar el inicio de sesión único de Azure AD en Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones **SiteIntel**, vaya a la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, junto a **Configuración básica de SAML**, seleccione **Editar** (icono del lápiz).

   ![Captura de pantalla del panel "Configurar el inicio de sesión único con SAML"](common/edit-urls.png)

1. Para configurar la aplicación en modo iniciado por IdP, en la sección **Configuración básica de SAML**, haga lo siguiente:

    a. En el cuadro **Identificador**, escriba una dirección URL con el siguiente formato: `urn:amazon:cognito:sp:<REGION>_<USERPOOLID>`.

    b. En el cuadro **URL de respuesta**, escriba una dirección URL con el siguiente formato: `https://<CLIENT>.auth.siteintel.com/saml2/idpresponse`.

    c. En el cuadro **Estado de la retransmisión**, escriba una dirección URL con el siguiente formato: `https://<CLIENT>.siteintel.com`.

1. Para configurar la aplicación en modo iniciado por SP, seleccione **Establecer direcciones URL adicionales** y, después, haga lo siguiente:

   * En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://<CLIENT>.siteintel.com`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de respuesta, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico de SiteIntel](mailto:support@intalytics.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el botón **Copiar** para copiar la **Dirección URL de metadatos de federación de aplicación**.

    ![Captura de pantalla del botón Copiar de "Dirección URL de metadatos de federación de aplicación"](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior del panel.
1. En las propiedades de **Usuario**, haga lo siguiente:

   a. En el cuadro **Nombre**, escriba **B.Simon**.  

   b. En el cuadro **Nombre de usuario**, escriba el nombre de usuario con el siguiente formato: `username@companydomain.extension` (por ejemplo, `B.Simon@contoso.com`).

   c. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.

   d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a SiteIntel mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de **aplicaciones**, seleccione **SiteIntel**.
1. En la sección **Administrar** de la página de información general de la aplicación, seleccione **Usuarios y grupos**.

   ![Captura de pantalla del vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, en el panel **Agregar asignación**, **Usuarios y grupos**.

    ![Captura de pantalla del botón "Agregar usuario"](common/add-assign-user.png)

1. En el panel **Usuarios y grupos**, seleccione **B.Simon** y, a continuación, el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el panel **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista y, a continuación, pulse el botón **Seleccionar**.
1. En el panel **Agregar asignación**, seleccione el botón **Asignar**.

## <a name="configure-siteintel-sso"></a>Configuración del inicio de sesión único de SiteIntel

Para configurar el inicio de sesión único en el lado de SiteIntel, envíe la dirección URL que copió en el cuadro **Dirección URL de metadatos de federación de aplicación** al [equipo de soporte técnico de SiteIntel](mailto:support@intalytics.com). Ellos configurarán este valor para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-a-siteintel-test-user"></a>Creación de un usuario de prueba de SiteIntel

En esta sección, creará un usuario llamado *Britta Simon* en SiteIntel. Trabaje con el  [equipo de soporte técnico de SiteIntel](mailto:support@intalytics.com) para agregar a los usuarios en la plataforma de SiteIntel. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de **SiteIntel** en el panel de acceso, debería iniciar sesión automáticamente en la versión de SiteIntel para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
- [Pruebe SiteIntel con Azure AD](https://aad.portal.azure.com/)
- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
- [Protección de SiteIntel con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
