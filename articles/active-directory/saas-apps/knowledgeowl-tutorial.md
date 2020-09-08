---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con KnowledgeOwl | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y KnowledgeOwl.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: jeedes
ms.openlocfilehash: a35fb30ca19e08c68c99c9c9524231706b066e11
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549898"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-knowledgeowl"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con KnowledgeOwl

En este tutorial aprenderá a integrar KnowledgeOwl con Azure Active Directory (Azure AD). Al integrar KnowledgeOwl con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a KnowledgeOwl.
* Permitir que los usuarios inicien sesión automáticamente en KnowledgeOwl con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de KnowledgeOwl con el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* KnowledgeOwl admite el inicio de sesión único iniciado por **SP e IDP**.
* KnowledgeOwl admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-knowledgeowl-from-the-gallery"></a>Agregar KnowledgeOwl desde la galería

Para configurar la integración de KnowledgeOwl en Azure AD, es preciso agregar KnowledgeOwl desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **KnowledgeOwl** en el cuadro de búsqueda.
1. Seleccione **KnowledgeOwl** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-knowledgeowl"></a>Configuración y prueba del inicio de sesión único de Azure AD para KnowledgeOwl

Configure y pruebe el inicio de sesión único de Azure AD con KnowledgeOwl con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de KnowledgeOwl.

Para configurar y probar el inicio de sesión único de Azure AD con KnowledgeOwl, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en KnowledgeOwl](#configure-knowledgeowl-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Crear un usuario de prueba en KnowledgeOwl](#create-knowledgeowl-test-user)** : para tener un homólogo de B.Simon en KnowledgeOwl vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **KnowledgeOwl**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:
    
    ```http
    https://app.knowledgeowl.com/sp
    https://app.knowledgeowl.com/sp/id/<unique ID>
    ```

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:
    
    ```http
    https://subdomain.knowledgeowl.com/help/saml-login
    https://subdomain.knowledgeowl.com/docs/saml-login
    https://subdomain.knowledgeowl.com/home/saml-login
    https://privatedomain.com/help/saml-login
    https://privatedomain.com/docs/saml-login
    https://privatedomain.com/home/saml-login
    ```

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:
    
    ```http
    https://subdomain.knowledgeowl.com/help/saml-login
    https://subdomain.knowledgeowl.com/docs/saml-login
    https://subdomain.knowledgeowl.com/home/saml-login
    https://privatedomain.com/help/saml-login
    https://privatedomain.com/docs/saml-login
    https://privatedomain.com/home/saml-login
    ```

    > [!NOTE]
    > Estos valores no son reales. Debe actualizar este valor con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales, tal como se explica más adelante en el tutorial.

1. La aplicación KnowledgeOwl espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación KnowledgeOwl espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen | Espacio de nombres |
    | ------------ | -------------------- | -----|
    | ssoid | user.mail | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`|

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (sin procesar)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

1. En la sección **Set up KnowledgeOwl** (Configurar KnowledgeOwl), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a KnowledgeOwl mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **KnowledgeOwl**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-knowledgeowl-sso"></a>Configuración del inicio de sesión único en KnowledgeOwl

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa de KnowledgeOwl como administrador.

1. Haga clic en **Configuración** y, a continuación, seleccione **Seguridad**.

    ![Configuración de KnowledgeOwl](./media/knowledgeowl-tutorial/configure1.png)

1. Desplácese hasta la opción **SAML SSO Integration** (Integración de SSO de SAML) y siga los pasos a continuación:

    ![Configuración de KnowledgeOwl](./media/knowledgeowl-tutorial/configure2.png)

    a. Seleccione **Enable SAML SSO** (Habilitar SSO de SAML).

    b. Copie el valor **SP Entity ID** (Id. de identidad de SP) en **Identificador (Id. de entidad)** en la sección **Configuración básica de SAML** de Azure Portal.

    c. Copie el valor de **SP Login URL** (Dirección URL de inicio de sesión SP) y péguelo en el cuadro de texto **Sign-on URL and Reply URL** (Direcciones URL de inicio de sesión y respuesta) de la sección **Configuración básica de SAML** de Azure Portal.

    d. En el cuadro de texto **IdP entityID** (Id. de entidad de IdP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    e. En el cuadro de texto **IdP Login URL** (Dirección URL de inicio de sesión de IdP), pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    f. En el cuadro de texto **IdP Logout URL** (URL de cierre de sesión de IdP), pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal.

    g. Cargue el certificado que descargó de Azure Portal haciendo clic en **Upload IdP Certificate** (Cargar certificado de IdP).

    h. Haga clic en **Map SAML Attributes** (Asignar atributos de SAML) para asignar atributos y realice los pasos siguientes:

    ![Configuración de KnowledgeOwl](./media/knowledgeowl-tutorial/configure3.png)

    * Escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/ssoid` en el cuadro de texto **Id. de SSO**.
    * Escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` en el cuadro de texto **Nombre de usuario / correo electrónico**.
    * Escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` en el cuadro de texto **Nombre**.
    * Escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` en el cuadro de texto **Apellidos**.
    * Haga clic en **Guardar**

    i. Haga clic en **Guardar** en la parte inferior de la página.

    ![Configuración de KnowledgeOwl](./media/knowledgeowl-tutorial/configure4.png)

### <a name="create-knowledgeowl-test-user"></a>Creación de un usuario de prueba de KnowledgeOwl

En esta sección se crea un usuario llamado B.Simon en KnowledgeOwl. KnowledgeOwl admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en KnowledgeOwl, se crea otro después de la autenticación.

> [!Note]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de KnowledgeOwl](mailto:support@knowledgeowl.com).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de KnowledgeOwl en el panel de acceso, debería iniciar sesión automáticamente en la versión de KnowledgeOwl para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe KnowledgeOwl con Azure AD](https://aad.portal.azure.com/)