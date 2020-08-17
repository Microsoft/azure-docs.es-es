---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Michigan Data Hub Single Sign-On | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Michigan Data Hub Single Sign-On.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 8fd008c2-65d0-41c6-987f-f749a5a21787
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 07/23/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: eeebd306614b57eb721977b403b7daf7a258dc87
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293619"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-michigan-data-hub-single-sign-on"></a>Tutorial: Azure Active Directory integración de inicio de sesión único (SSO) con Michigan Data Hub Single Sign-On

En este tutorial, aprenderá a integrar Michigan Data Hub Single Sign-On con Azure Active Directory (Azure AD). Al integrar Michigan Data Hub Single Sign-On con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Michigan Data Hub Single Sign-On.
* Puede permitir que los usuarios inicien sesión automáticamente en Michigan Data Hub Single Sign-On con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Michigan Data Hub Single Sign-On.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Michigan Data Hub Single Sign-On admite el inicio de sesión único iniciado por **SP**
* Una vez que haya configurado Michigan Data Hub Single Sign-On, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-michigan-data-hub-single-sign-on-from-the-gallery"></a>Adición de Michigan Data Hub Single Sign-On desde la galería

Para configurar la integración de Michigan Data Hub Single Sign-On en Azure AD, será preciso que agregue Michigan Data Hub Single Sign-On desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Michigan Data Hub Single Sign-On** en el cuadro de búsqueda.
1. Seleccione **Michigan Data Hub Single Sign-On** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-michigan-data-hub-single-sign-on"></a>Configuración y prueba del inicio de sesión único de Azure AD para Michigan Data Hub Single Sign-On

Configure y pruebe el inicio de sesión único de Azure AD con Michigan Data Hub Single Sign-On mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Michigan Data Hub Single Sign-On.

Para configurar y probar el inicio de sesión único de Azure AD con Michigan Data Hub Single Sign-On, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Michigan Data Hub Single Sign-On](#configure-michigan-data-hub-single-sign-on-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Michigan Data Hub Single Sign-On](#create-michigan-data-hub-single-sign-on-test-user)** , para tener un homólogo de B.Simon en Michigan Data Hub Single Sign-On que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Michigan Data Hub Single Sign-On**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://launchpad.midatahub.org`

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

En esta sección, concederá a B.Simon acceso a Michigan Data Hub Single Sign-On para que pueda usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Michigan Data Hub Single Sign-On**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-michigan-data-hub-single-sign-on-sso"></a>Configuración del inicio de sesión único en Michigan Data Hub Single Sign-On

Para configurar el inicio de sesión único en **Michigan Data Hub Single Sign-On**, es preciso enviar el valor de **App Federation Metadata Url** (Dirección URL de metadatos de federación de la aplicación) al [equipo de soporte técnico de inicio de sesión de ](mailto:support@midatahub.org). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-michigan-data-hub-single-sign-on-test-user"></a>Creación de un usuario de prueba de Michigan Data Hub Single Sign-On

En esta sección, va a crear un usuario llamado B.Simon en Michigan Data Hub Single Sign-On. Trabaje con el  [equipo de soporte técnico de Michigan Data Hub Single Sign-On](mailto:support@midatahub.org) para agregar los usuarios a la plataforma de Michigan Data Hub Single Sign-On. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Michigan Data Hub Single Sign-On en el panel de acceso, debería iniciar sesión automáticamente en la versión de Michigan Data Hub Single Sign-On para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Michigan Data Hub Single Sign-On con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Procedimiento para proteger Michigan Data Hub Single Sign-On con visibilidad y controles avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)