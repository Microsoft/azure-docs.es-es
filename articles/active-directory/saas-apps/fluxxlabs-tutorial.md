---
title: 'Tutorial: integración de Azure Active Directory con Fluxx Labs | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Fluxx Labs.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/21/2020
ms.author: jeedes
ms.openlocfilehash: 756bbaab1cbf359def01e371e4370607bee67ce3
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554869"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-fluxx-labs"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Fluxx Labs

En este tutorial, obtendrá información sobre cómo integrar Fluxx Labs con Azure Active Directory (Azure AD). Al integrar Fluxx Labs con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Fluxx Labs.
* Permitir que los usuarios inicien sesión automáticamente en Fluxx Labs con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Fluxx Labs.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Fluxx Labs admite el inicio de sesión único iniciado por **IDP**
* Una vez configurado Fluxx Labs, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-fluxx-labs-from-the-gallery"></a>Adición de Fluxx Labs desde la galería

Para configurar la integración de Fluxx Labs en Azure AD, será preciso que agregue Fluxx Labs desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Fluxx Labs** en el cuadro de búsqueda.
1. Seleccione **Fluxx Labs** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-fluxx-labs"></a>Configuración y prueba del inicio de sesión único de Azure AD para Fluxx Labs

Configure y pruebe el inicio de sesión único de Azure AD con Fluxx Labs mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Fluxx Labs.

Para configurar y probar el inicio de sesión único de Azure AD con Fluxx Labs, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Fluxx Labs](#configure-fluxx-labs-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Fluxx Labs](#create-fluxx-labs-test-user)** : para tener un homólogo de B.Simon en Fluxx Labs que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Fluxx Labs**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:

    | Entorno | Patrón de dirección URL|
    |-------------|------------|
    | Producción | `https://<subdomain>.fluxx.io` |
    | Preproducción | `https://<subdomain>.preprod.fluxxlabs.com`|

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    | Entorno | Patrón de dirección URL|
    |-------------|------------|
    | Producción | `https://<subdomain>.fluxx.io/auth/saml/callback` |
    | Preproducción | `https://<subdomain>.preprod.fluxxlabs.com/auth/saml/callback`|

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Fluxx Labs](https://fluxx.zendesk.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Fluxx Labs**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Fluxx Labs mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Fluxx Labs**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-fluxx-labs-sso"></a>Configuración del inicio de sesión único de Fluxx Labs

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Fluxx Labs como administrador.

2. Seleccione **Administrador** después de la sección **Configuración**.

    ![Configuración de Fluxx Labs](./media/fluxxlabs-tutorial/config1.png)

3. En el panel de administración, seleccione **Plug-ins** (Complementos) > **Integrations** (Integraciones) y, a continuación, seleccione **SAML SSO-(Disabled)** [SAML SSO-(Deshabilitado)]

    ![Configuración de Fluxx Labs](./media/fluxxlabs-tutorial/config2.png)

4. En la sección de atributos, realice los siguientes pasos:

    ![Configuración de Fluxx Labs](./media/fluxxlabs-tutorial/config3.png)

    a. Seleccione la casilla de verificación **SAML SSO** (SSO de SAML).

    b. En el cuadro de texto **Request Path** (Ruta de acceso de la solicitud), escriba **/auth/saml**.

    c. En el cuadro de texto **Callback Path** (Ruta de acceso de devolución de llamada), escriba **/auth/saml/callback**.

    d. En el cuadro **Assertion Consumer Service Url (Single Sign-On URL)** (URL del Servicio de consumidor de aserciones (URL de inicio de sesión único)), escriba el valor de **Reply URL** (URL de respuesta) que introdujo en Azure Portal.

    e. En el cuadro de texto **Audience(SP Entity ID)** (Público (ID de entidad SP)), introduzca el valor **Identificador**, que introdujo en Azure Portal.

    f. En el cuadro de texto **Identity Provider SSO Target URL** (Dirección URL de destino del inicio de sesión único del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    g. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de proveedor de identidades**.

    h. En el cuadro de texto **Name identifier Format** (Formato de identificador de nombre), escriba el valor `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

    i. Haga clic en **Save**(Guardar).

    > [!NOTE]
    > Una vez guardado el contenido, el campo aparecerá en blanco por seguridad, pero se ha guardado el valor en la configuración.

### <a name="create-fluxx-labs-test-user"></a>Creación de un usuario de prueba de Fluxx Labs

Para permitir que los usuarios de Azure AD inicien sesión en Fluxx Labs, deben aprovisionarse en Fluxx Labs. En el caso de Fluxx Labs, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el sitio de la compañía de Fluxx Labs.

2. Haga clic en el **icono** que aparece a continuación.

    ![Configuración de Fluxx Labs](./media/fluxxlabs-tutorial/config6.png)

3. En el panel, haga clic en el icono que se muestra a continuación para abrir la tarjeta **New PEOPLE** (Nueva persona).

    ![Configuración de Fluxx Labs](./media/fluxxlabs-tutorial/config4.png)

4. En la sección **NEW PEOPLE** (Nueva persona), lleve a cabo estos pasos:

    ![Configuración de Fluxx Labs](./media/fluxxlabs-tutorial/config5.png)

    a. Fluxx Labs usa la dirección de correo electrónico como identificador único para los inicios de sesión SSO. Rellene el campo **SSO UID** con la dirección de correo electrónico del usuario, que coincide con la dirección de correo electrónico que usan como inicio de sesión con SSO.

    b. Haga clic en **Save**(Guardar).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Fluxx Labs en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Fluxx Labs para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Fluxx Labs con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Fluxx Labs con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)