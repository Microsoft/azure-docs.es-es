---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cloud Academy - SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Cloud Academy - SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: jeedes
ms.openlocfilehash: 88e626f9b3069b3b43d525914c017caf763a9047
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551768"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cloud-academy---sso"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cloud Academy - SSO

En este tutorial aprenderá a integrar Cloud Academy - SSO con Azure Active Directory (Azure AD). Al integrar Cloud Academy - SSO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Cloud Academy - SSO.
* Permitir que los usuarios inicien sesión automáticamente en Cloud Academy - SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Cloud Academy - SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cloud Academy - SSO admite el inicio de sesión único iniciado por **SP**

* Una vez que haya configurado Cloud Academy - SSO, puede aplicar el control de sesión, que protege a la organización en tiempo real contra la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-cloud-academy---sso-from-the-gallery"></a>Adición de Cloud Academy - SSO desde la galería

Para configurar la integración de Cloud Academy - SSO en Azure AD, es preciso agregar Cloud Academy - SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Cloud Academy - SSO** en el cuadro de búsqueda.
1. Seleccione **Cloud Academy - SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-cloud-academy---sso"></a>Configuración y prueba del inicio de sesión único de Azure AD SSO para Cloud Academy - SSO

Configure y pruebe el inicio de sesión único de Azure AD con Cloud Academy - SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Cloud Academy - SSO.

Para configurar y probar el inicio de sesión único de Azure AD con Cloud Academy - SSO, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Cloud Academy - SSO](#configure-cloud-academy-sso-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Cloud Academy - SSO](#create-cloud-academy-sso-test-user)** , para tener un homólogo de B.Simon en Prisma Cloud SSO que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Cloud Academy - SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://cloudacademy.com/login/enterprise/`

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

En esta sección, va a permitir que B.Simon acceda a Cloud Academy - SSO utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Cloud Academy - SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cloud-academy-sso-sso"></a>Configuración del inicio de sesión único en Cloud Academy - SSO

1. En otra ventana del explorador, inicie sesión en el sitio de la compañía de Cloud Academy - SSO como administrador.

1. Haga clic en el nombre de la empresa y seleccione **Settings & Integrations** (Configuración e integraciones) en el menú.

    ![Configuración ](./media/cloud-academy-sso-tutorial/config-1.PNG)

1. En la página **Settings & integraciones** (Configuración e integraciones), vaya a la pestaña **Integrations** (Integraciones) y haga clic en **SSO** (Inicio de sesión único).

    ![Configuración ](./media/cloud-academy-sso-tutorial/config-2.PNG)

1. En la página siguiente, realice estos pasos:

    ![Configuración ](./media/cloud-academy-sso-tutorial/config-3.PNG)

    a. En el cuadro de texto **Entity ID URL** (Dirección URL de identificador de entidades), pegue el valor del **Id. de entidad** que ha copiado desde Azure Portal.

    b. En el cuadro de texto **URL de inicio de sesión único**, pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra en el Bloc de notas el archivo del **certificado (Base64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **Certificate** (Certificado).

    d. En el cuadro de texto **Formato de identificador de nombre**, el valor predeterminado, `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`

1. Haga clic en el botón **Save** (Guardar).

    > [!NOTE]
    > Para más información sobre cómo configurar Cloud Academy - SSO, consulte el [artículo de soporte técnico](https://support.cloudacademy.com/hc/articles/360043908452-Setting-Up-Single-Sign-On).

### <a name="create-cloud-academy-sso-test-user"></a>Creación de un usuario de prueba de Cloud Academy - SSO

1. Inicie sesión en **Cloud Academy - SSO**.

1. Haga clic en el nombre de la empresa y seleccione **Members** (Miembros) en el menú.

    ![ Creación de un usuario de prueba ](./media/cloud-academy-sso-tutorial/create-user.PNG)

1. Haga clic en **Invite Members** (Invitar a miembros) y seleccione **Invite a Single Member** (Invitar a un solo miembro).

    ![ Creación de un usuario de prueba ](./media/cloud-academy-sso-tutorial/create-user-1.PNG)

1. Escriba los campos obligatorios y haga clic en **Invite** (Invitar).

    ![ Creación de un usuario de prueba ](./media/cloud-academy-sso-tutorial/create-user-2.PNG)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Cloud Academy - SSO en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Cloud Academy - SSO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Cloud Academy - SSO con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Cloud Academy - SSO con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)