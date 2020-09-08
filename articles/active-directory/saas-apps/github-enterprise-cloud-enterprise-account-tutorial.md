---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub Enterprise Cloud - Enterprise Account | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y GitHub Enterprise Cloud - Enterprise Account.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/29/2020
ms.author: jeedes
ms.openlocfilehash: 4e23c8a8497459ce7cb3cab3d1469359f80ad846
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550782"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github-enterprise-cloud---enterprise-account"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub Enterprise Cloud - Enterprise Account

En este tutorial, aprenderá a integrar la cuenta de GitHub Enterprise Cloud - Enterprise Account con Azure Active Directory (Azure AD). Al integrar la cuenta de GitHub Enterprise Cloud - Enterprise Account con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a la cuenta de GitHub Enterprise Cloud - Enterprise Account.
* Permita que los usuarios inicien sesión automáticamente en la cuenta de GitHub Enterprise Cloud-Enterprise Account con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) de GitHub Enterprise Cloud - Enterprise Account.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* GitHub Enterprise Cloud - Enterprise Account admite el inicio de sesión único (SSO) iniciado por **SP** e **IDP**.
* GitHub Enterprise Cloud - Enterprise Account admite el aprovisionamiento de usuarios **Just-in-Time**.
* Una vez configurado GitHub Enterprise Cloud - Enterprise Account, puede aplicar el control de sesión, que protege contra la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-github-enterprise-cloud---enterprise-account-from-the-gallery"></a>Adición de GitHub Enterprise Cloud - Enterprise Account desde la galería

Para configurar la integración de GitHub Enterprise Cloud - Enterprise Account en Azure AD, es preciso agregar GitHub Enterprise Cloud - Enterprise Account desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la Galería**, escriba **GitHub Enterprise Cloud - Enterprise Account** en el cuadro de búsqueda.
1. Seleccione **GitHub Enterprise Cloud - Enterprise Account** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-github-enterprise-cloud---enterprise-account"></a>Configuración y prueba del inicio de sesión único (SSO) de Azure AD para GitHub Enterprise Cloud - Enterprise Account

Configure y pruebe el inicio de sesión único (SSO) de Azure AD en GitHub Enterprise Cloud - Enterprise Account con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una vinculación entre un usuario de Azure AD y el usuario correspondiente de GitHub Enterprise Cloud - Enterprise Account.

Para configurar y probar el inicio de sesión único (SSO) de Azure AD con GitHub Enterprise Cloud - Enterprise Account, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único (SSO) de GitHub Enterprise Cloud - Enterprise Account](#configure-github-enterprise-cloud-enterprise-account-sso)** , para configurar la configuración del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de GitHub Enterprise Cloud - Enterprise Account](#create-github-enterprise-cloud-enterprise-account-test-user)** , para tener un homólogo de B.Simon en GitHub Enterprise Cloud - Enterprise Account que esté vinculado a la representación en Azure AD del usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de  **GitHub Enterprise Cloud - Enterprise Account**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://github.com/enterprises/<ENTERPRISE-SLUG>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://github.com/enterprises/<ENTERPRISE-SLUG>/saml/consume`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

     En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://github.com/enterprises/<ENTERPRISE-SLUG>/sso`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, la dirección URL de respuesta y el identificador reales. Póngase en contacto con el [equipo de soporte técnico de GitHub Enterprise Cloud - Enterprise Account](mailto:support@github.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateBase64.png)

1. En la sección **Configurar GitHub Enterprise Cloud - Enterprise Account**, copie las direcciones URL adecuadas en función de sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a GitHub Enterprise Cloud - Enterprise Account mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **GitHub Enterprise Cloud - Enterprise Account**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-github-enterprise-cloud-enterprise-account-sso"></a>Configuración del inicio de sesión único (SSO) de GitHub Enterprise Cloud - Enterprise Account

Para configurar el inicio de sesión único en **GitHub Enterprise Cloud-Enterprise Account**, es preciso enviar el **certificado (Base64)** descargado y las direcciones URL copiadas adecuadas de Azure Portal al [equipo de soporte técnico de GitHub Enterprise Cloud - Enterprise](mailto:support@github.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-github-enterprise-cloud-enterprise-account-test-user"></a>Creación del usuario de prueba de GitHub Enterprise Cloud - Enterprise Account

En esta sección, se crea un usuario llamado B.Simon en GitHub Enterprise Cloud - Enterprise Account. GitHub Enterprise Cloud - Enterprise Account admite el aprovisionamiento de usuarios Just-in-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario ya no existe en GitHub Enterprise Cloud - Enterprise Account, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de GitHub Enterprise Cloud - Enterprise Account en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de GitHub Enterprise Cloud - Enterprise Account para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe GitHub Enterprise Cloud - Enterprise Account con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Procedimiento para proteger GitHub Enterprise Cloud - Enterprise Account con visibilidad y controles avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)