---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Kanbanize | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Kanbanize.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/09/2019
ms.author: jeedes
ms.openlocfilehash: 686a186650bfe4066533b0f086c080b5ffb14eb6
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546841"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-kanbanize"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Kanbanize

En este tutorial, aprenderá a integrar Kanbanize con Azure Active Directory (Azure AD). Al integrar Kanbanize con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Kanbanize
* Permitir que los usuarios puedan iniciar sesión automáticamente en Kanbanize con sus cuentas de Azure AD
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Kanbanize.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Kanbanize admite el inicio de sesión único iniciado por **SP e IDP**.
* Kanbanize admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-kanbanize-from-the-gallery"></a>Adición de Kanbanize desde la galería

Para configurar la integración de Kanbanize en Azure AD, deberá agregar Kanbanize desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Kanbanize** en el cuadro de búsqueda.
1. Seleccione **Kanbanize** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-kanbanize"></a>Configuración y prueba del inicio de sesión único de Azure AD para Kanbanize

Configure y pruebe el inicio de sesión único de Azure AD con Kanbanize mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Kanbanize.

Para configurar y probar el inicio de sesión único de Azure AD con Kanbanize, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Kanbanize](#configure-kanbanize-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Kanbanize](#create-kanbanize-test-user)** , para tener un homólogo de B.Simon en Kanbanize que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Kanbanize**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

     a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<subdomain>.kanbanize.com/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.kanbanize.com/saml/acs`

    c. Haga clic en **Establecer direcciones URL adicionales**.

    d. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL: `/ctrl_login/saml_login`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.kanbanize.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Kanbanize](mailto:support@ms.kanbanize.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Kanbanize espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde nameidentifier se asigna con **user.userprincipalname**. La aplicación Kanbanize espera que nameidentifier se asigne con **user.mail**, así que debe hacer clic en el icono Editar para editar la asignación de atributos y cambiar dicha asignación.

    ![imagen](common/edit-attribute.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Kanbanize**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Kanbanize mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Kanbanize**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-kanbanize-sso"></a>Configuración del inicio de sesión único de Kanbanize

1. Para automatizar la configuración en Kanbanize, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Kanbanize** para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en Kanbanize. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Kanbanize manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Kanbanize como administrador y lleve a cabo los siguientes pasos:

4. Vaya a la parte superior derecha de la página y haga clic en el logotipo de **configuración**.

    ![Configuración de Kanbanize](./media/kanbanize-tutorial/tutorial-kanbanize-set.png)

5. En la página del panel de administración del lado izquierdo del menú, haga clic en **Integrations** (Integraciones) y, a continuación, habilite **Single Sign-On** (Inicio de sesión único).

    ![Integraciones de Kanbanize](./media/kanbanize-tutorial/tutorial-kanbanize-admin.png)

6. En la sección de integraciones, haga clic en **CONFIGURE** (CONFIGURAR) para abrir la página **Single Sign-On Integration** (Integración de inicio de sesión único).

    ![Configuración de Kanbanize](./media/kanbanize-tutorial/tutorial-kanbanize-config.png)

7. En la página **Single Sign-On Integration** (Integración de inicio de sesión único), en **Configurations** (Configuraciones), realice los siguientes pasos:

    ![Integraciones de Kanbanize](./media/kanbanize-tutorial/tutorial-kanbanize-save.png)

    a. En el cuadro de texto **Idp Entity ID** (Identificador de la entidad de IdP), pegue el valor de **Azure AD Identifier** (Identificador de Azure AD) que copió de Azure Portal.

    b. En el cuadro de texto **Idp Login Endpoint** (Punto de conexión de inicio de sesión de Idp), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que copió de Azure Portal.

    c. En el cuadro de texto **Idp Logout Endpoint** (Punto de conexión de cierre de sesión de Idp), pegue el valor de **Logout URL** (Dirección URL de cierre de sesión) que copió de Azure Portal.

    d. En **Attribute name for Email** (Nombre de atributo de correo electrónico), escriba este valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    e. En el cuadro de texto **Attribute name for First Name** (Nombre de atributo para nombre), escriba este valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`.

    f. En el cuadro de texto **Attribute name for Last Name** (Nombre de atributo para apellido), escriba este valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`.

    > [!Note]
    > Puede obtener estos valores mediante la combinación de valores de espacio de nombres y nombre del atributo correspondiente de la sección Atributos de usuario de Azure Portal.

    g. En el Bloc de notas, abra el certificado codificado en base 64 que descargó de Azure Portal, copie su contenido (sin los marcadores de inicio y fin) y, luego, péguelo en el cuadro de texto **Idp X.509 Certificate** (Certificado X.509 de IdP).

    h. Marque **Enable login with both SSO and Kanbanize** (Habilitar el inicio de sesión con SSO y Kanbanize).

    i. Haga clic en **Guardar configuración**.

### <a name="create-kanbanize-test-user"></a>Creación de un usuario de prueba de Kanbanize

En esta sección, se crea un usuario llamado B.Simon en Kanbanize. Kanbanize admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si aún no existe un usuario en Kanbanize, se crea uno después de la autenticación. Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de Kanbanize](mailto:support@ms.kanbanize.com).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Kanbanize en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Kanbanize para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Kanbanize con Azure AD](https://aad.portal.azure.com/)

