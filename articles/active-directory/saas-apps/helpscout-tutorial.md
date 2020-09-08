---
title: 'Tutorial: Integración de Azure Active Directory con Help Scout | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Help Scout.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: jeedes
ms.openlocfilehash: 16783d344c6a0dedec4e24d49150e0b4b2edbe3e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548152"
---
# <a name="tutorial-azure-active-directory-integration-with-help-scout"></a>Tutorial: integración de Azure Active Directory con Help Scout

En este tutorial, obtendrá información sobre cómo integrar Help Scout con Azure Active Directory (Azure AD).
La integración de Help Scout con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a Help Scout.
* Puede permitir que los usuarios inicien sesión automáticamente en Help Scout (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Help Scout, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Help Scout

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Help Scout admite el inicio de sesión único iniciado por **SP e IDP**
* Help Scout admite aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-help-scout-from-the-gallery"></a>Incorporación de Help Scout desde la galería

Para configurar la integración de Help Scout en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Help Scout** en el cuadro de búsqueda.
1. Seleccione **Help Scout** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección configurará y probará el inicio de sesión único de Azure AD con Help Scout mediante un usuario de prueba llamado **B.Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Help Scout.

Para configurar y probar el inicio de sesión único de Azure AD con Help Scout, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Help Scout](#configure-help-scout-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en Help Scout](#create-help-scout-test-user)** , para tener un homólogo de B.Simon en Help Scout que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Help Scout, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Help Scout**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

1. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Help Scout](common/idp-intiated.png)

    a. **Identificador** es la opción **Audience URI (Service Provider Entity ID)** [URI de audiencia (Identificador de entidad del proveedor de servicios)] de Help Scout y empieza por `urn:`

    b. **URL de respuesta** es la opción **Post-back URL (Assertion Consumer Service URL)** [URL de devolución (URL del Servicio de consumidor de aserciones)] de Help Scout y empieza por `https://` 

    > [!NOTE]
    > Los valores de estas direcciones URL se muestran solo con fines demostrativos. Tiene que actualizar estos valores con el identificador y la dirección URL de respuesta reales. Estos valores se obtienen en la pestaña **Single Sign-On** (Inicio de sesión único), en la sección Authentication (Autenticación), que se explica más adelante en el tutorial.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de Help Scout](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una URL como: `https://secure.helpscout.net/members/login/`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Help Scout** (Configurar Help Scout), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **B.Simon**.
  
    b. En el campo **Nombre de usuario**, escriba **B.Simon\@dominioDeSuEmpresa.extensión**.  
    Por ejemplo: B.Simon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a Help Scout.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Help Scout**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Help Scout**.

    ![Vínculo a Help Scout en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-help-scout-sso"></a>Configuración del inicio de sesión único de Help Scout

1. Para automatizar la configuración en Help Scout, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Help Scout** para ir a esta aplicación. En ella, escriba las credenciales de administrador para iniciar sesión en Help Scout. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar Help Scout manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Help Scout como administrador y lleve a cabo los siguientes pasos:

1. Haga clic en **Manage** (Administrar) en el menú superior y seleccione **Company** (Compañía) en el menú desplegable.

    ![Configurar inicio de sesión único](./media/helpscout-tutorial/settings1.png)

1. Seleccione **Autenticación** en el panel de navegación izquierdo.

    ![Configurar inicio de sesión único](./media/helpscout-tutorial/settings2.png)

1. Esto le lleva a la sección de configuración de SAML, donde debe seguir estos pasos:

    ![Configurar inicio de sesión único](./media/helpscout-tutorial/settings3.png)

    a. Copie el valor de **Post-back URL (Assertion Consumer Service URL)** [URL de devolución (URL del Servicio de consumidor de aserciones)] y péguelo en el cuadro **URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    b. Copie el valor de **Audience URI (Service Provider Entity ID)** [URI de audiencia (Identificador de entidad del proveedor de servicios)] y péguelo en el cuadro **Identificador**, en la sección **Configuración básica de SAML** de Azure Portal.

1. Active **Enable SAML** (Habilitar SAML) y siga estos pasos:

    ![Configurar inicio de sesión único](./media/helpscout-tutorial/settings4.png)

    a. En el cuadro de texto **Single Sign-on URL** (Dirección URL de inicio de sesión único), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. Haga clic en **Upload Certificate** (Cargar certificado) para cargar el **Certificado (Base64)** que descargó de Azure Portal.

    c. Especifique el dominio o dominios de correo electrónico de su organización, por ej. `contoso.com` en el cuadro de texto **Email Domains** (Dominios de correo electrónico). Puede separar varios dominios mediante comas. Siempre que un administrador o un usuario de Help Scout entre en ese dominio específico en la [página de inicio de sesión de Help Scout](https://secure.helpscout.net/members/login/), se le redirigirá al proveedor de identidades para que se autentique con sus credenciales.

    d. Por último, puede cambiar **Force SAML Sign-on** (Forzar inicio de sesión de SAML) si desea que los usuarios solo inicien sesión en Help Scout mediante este método. Si aún así desea dejar la opción para que puedan conectarse con sus credenciales de Help Scout, puede dejarla desactivada. Incluso si esta opción está habilitada, el propietario de la cuenta siempre podrá iniciar sesión en Help Scout con su contraseña de la cuenta.

    e. Haga clic en **Save**(Guardar).

### <a name="create-help-scout-test-user"></a>Creación de un usuario de prueba de Help Scout

En esta sección, se crea un usuario llamado B.Simon en Help Scout. Help Scout admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Help Scout, se crea otro después de la autenticación.

### <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Help Scout en el panel de acceso, debería iniciar sesión automáticamente en la versión de Help Scout para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Help Scout con Azure AD](https://aad.portal.azure.com/)