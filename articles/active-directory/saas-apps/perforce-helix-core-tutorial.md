---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Perforce Helix Core - Helix Authentication Service | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Perforce Helix Core - Helix Authentication Service.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: jeedes
ms.openlocfilehash: 94464827f0a3cb797fa0e1b0249131e5a110dbad
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553876"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-perforce-helix-core---helix-authentication-service"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Perforce Helix Core - Helix Authentication Service

En este tutorial, aprenderá a integrar Perforce Helix Core - Helix Authentication Service con Azure Active Directory (Azure AD). Al integrar Perforce Helix Core - Helix Authentication Service con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Perforce Helix Core - Helix Authentication Service.
* Permitir que los usuarios inicien sesión automáticamente en Perforce Helix Core - Helix Authentication Service con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Perforce Helix Core - Helix Authentication Service.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Perforce Helix Core - Helix Authentication Service admite el inicio de sesión único iniciado por **SP**.
* Una vez que se ha configurado Perforce Helix Core - Helix Authentication Service, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración y la infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-perforce-helix-core---helix-authentication-service-from-the-gallery"></a>Adición de Perforce Helix Core - Helix Authentication Service desde la galería

Para configurar la integración de Perforce Helix Core - Helix Authentication Service en Azure AD, tiene que agregar Perforce Helix Core - Helix Authentication Service desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Perforce Helix Core - Helix Authentication Service** en el cuadro de búsqueda.
1. Seleccione **Perforce Helix Core - Helix Authentication Service** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-perforce-helix-core---helix-authentication-service"></a>Configuración y prueba del inicio de sesión único de Azure AD para Perforce Helix Core - Helix Authentication Service

Configure y pruebe el inicio de sesión único de Azure AD con Perforce Helix Core - Helix Authentication Service mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una vinculación entre un usuario de Azure AD y el usuario relacionado de Perforce Helix Core - Helix Authentication Service.

Para configurar y probar el inicio de sesión único de Azure AD con Perforce Helix Core - Helix Authentication Service, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Perforce Helix Core - Helix Authentication Service](#configure-perforce-helix-core---helix-authentication-service-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Perforce Helix Core - Helix Authentication Service](#create-perforce-helix-core---helix-authentication-service-test-user)** , para tener un usuario equivalente a B.Simon en Perforce Helix Core - Helix Authentication Service que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Perforce Helix Core - Helix Authentication Service**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<helix-auth-service>.<customer-hostname>.com/`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<helix-auth-service>.<customer-hostname>.com/saml`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<HELIX-AUTH-SERVICE>.<CUSTOMER-HOSTNAME>.com/saml/sso`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales. Póngase en contacto con el [equipo de soporte técnico al cliente de Perforce Helix Core - Helix Authentication Service](mailto:support@perforce.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

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

En esta sección va a permitir que B.Simon acceda a Perforce Helix Core - Helix Authentication Service mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Perforce Helix Core - Helix Authentication Service**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-perforce-helix-core---helix-authentication-service-sso"></a>Configuración del inicio de sesión único de Perforce Helix Core - Helix Authentication Service

Para configurar el inicio de sesión único en **Perforce Helix Core - Helix Authentication Service**, debe enviar el valor de **Dirección URL de metadatos de federación de aplicación** al [equipo de soporte técnico de Perforce Helix Core - Helix Authentication Service](mailto:support@perforce.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-perforce-helix-core---helix-authentication-service-test-user"></a>Creación de un usuario de prueba de Perforce Helix Core - Helix Authentication Service

En esta sección, creará un usuario llamado B.Simon en Perforce Helix Core - Helix Authentication Service. Colabore con el [equipo de soporte técnico de Perforce Helix Core - Helix Authentication Service](mailto:support@perforce.com) para agregar los usuarios a la plataforma de Perforce Helix Core - Helix Authentication Service. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Perforce Helix Core - Helix Authentication Service, debería iniciar sesión automáticamente en la aplicación Perforce Helix Core - Helix Authentication Service para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Perforce Helix Core - Helix Authentication Service con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Perforce Helix Core - Helix Authentication Service mediante visibilidad y controles avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
