---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con PureCloud by Genesys | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PureCloud by Genesys.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/03/2019
ms.author: jeedes
ms.openlocfilehash: 0d2953710c559deb094761410e6d46401e103d8b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553298"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-purecloud-by-genesys"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con PureCloud by Genesys

En este tutorial, aprenderá a integrar PureCloud by Genesys con Azure Active Directory (Azure AD). Después de hacerlo, podrá:

* Usar Azure AD para controlar qué usuarios pueden acceder a PureCloud by Genesys.
* Permitir que los usuarios inicien sesión automáticamente en PureCloud by Genesys con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una, puede obtener una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en PureCloud by Genesys.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* PureCloud by Genesys admite el inicio de sesión único iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-purecloud-by-genesys-from-the-gallery"></a>Incorporación de PureCloud by Genesys desde la galería

Para configurar la integración de PureCloud by Genesys en Azure AD, deberá agregar PureCloud by Genesys desde la galería a la lista de aplicaciones SaaS administradas. Para ello, siga estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante una cuenta profesional o educativa, o bien mediante una cuenta personal de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **PureCloud by Genesys** en el cuadro de búsqueda.
1. Seleccione **PureCloud by Genesys** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-purecloud-by-genesys"></a>Configuración y prueba del inicio de sesión único de Azure AD para PureCloud by Genesis

Configure y pruebe el inicio de sesión único de Azure AD con PureCloud by Genesys mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de PureCloud by Genesys.

Para configurar y probar el inicio de sesión único de Azure AD con PureCloud by Genesys, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en PureCloud by Genesys](#configure-purecloud-by-genesys-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de PureCloud by Genesys](#create-purecloud-by-genesys-test-user)** , para tener un homólogo de B.Simon en PureCloud by Genesys que esté vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Para habilitar el inicio de sesión único de Azure AD en Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **PureCloud by Genesys**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro **Identificador**, escriba una dirección URL que corresponda a su región:

    ```http
    https://login.mypurecloud.com/saml
    https://login.mypurecloud.de/saml
    https://login.mypurecloud.jp/saml
    https://login.mypurecloud.ie/saml
    https://login.mypurecloud.au/saml
    ```

    b. En el cuadro **URL de respuesta**, escriba una dirección URL que corresponda a su región:

    ```http
    https://login.mypurecloud.com/saml
    https://login.mypurecloud.de/saml
    https://login.mypurecloud.jp/saml
    https://login.mypurecloud.ie/saml
    https://login.mypurecloud.com.au/saml
    ```

1. Seleccione **Establecer direcciones URL adicionales** y lleve a cabo el siguiente paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro **URL de inicio de sesión**, escriba una dirección URL que corresponda a su región:
    
    ```http
    https://login.mypurecloud.com
    https://login.mypurecloud.de
    https://login.mypurecloud.jp
    https://login.mypurecloud.ie
    https://login.mypurecloud.com.au
    ```

1. La aplicación PureCloud by Genesys espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de atributos de token SAML. En la siguiente captura se muestra la lista de los atributos predeterminados:

    ![imagen](common/default-attributes.png)

1. Además, la aplicación PureCloud by Genesys espera que se devuelvan algunos atributos más en la respuesta de SAML, como se muestra en la tabla siguiente. Estos atributos también se rellenan previamente, pero puede revisarlos según necesite.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | Email | user.userprincipalname |
    | OrganizationName | `Your organization name` |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar PureCloud by Genesys**, copie las direcciones URL adecuadas según sus requisitos.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado B.Simon en Azure Portal:

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba el nombre de usuario con el siguiente formato: username@companydomain.extension. Por ejemplo: `B.Simon@contoso.com`.
   1. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a PureCloud by Genesys.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **PureCloud by Genesys**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios y, luego, elija el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y elija el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-purecloud-by-genesys-sso"></a>Configuración del inicio de sesión único en PureCloud by Genesys

1. En otra ventana del explorador web, inicie sesión en PureCloud by Genesys como administrador.

1. Seleccione **Admin** (Administración) en la parte superior y, a continuación, vaya a **Single Sign-On** (Inicio de sesión único) en **Integrations** (Integraciones).

    ![Configurar inicio de sesión único](./media/purecloud-by-genesys-tutorial/configure01.png)

1. Cambie a la pestaña **ADFS/Azure AD (Premium)** y, a continuación, siga estos pasos:

    ![Configurar inicio de sesión único](./media/purecloud-by-genesys-tutorial/configure02.png)

    a. Seleccione **Browse** (Examinar) para cargar en **ADFS Certificate** (Certificado de ADFS) el certificado codificado en base 64 que descargó de Azure Portal.

    b. En el cuadro de texto **ADFS Issuer URI** (URI de emisor de ADFS), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro de texto **Target URI** (URI de destino), pegue el valor de **URL de inicio de sesión** que copió de Azure Portal.

    d. Para el valor de **Relying Party Identifier** (Identificador de usuario de confianza), vaya a Azure Portal y, en la página de integración de aplicaciones de **PureCloud by Genesys**, seleccione la pestaña **Propiedades** y copie el valor de **Id. de aplicación**. Péguelo en el cuadro de texto **Relying Party Identifier** (Identificador de usuario de confianza).

    ![Configurar inicio de sesión único](./media/purecloud-by-genesys-tutorial/configure06.png)

    e. Seleccione **Guardar**.

### <a name="create-purecloud-by-genesys-test-user"></a>Creación de un usuario de prueba de PureCloud by Genesys

Para permitir que los usuarios de Azure AD inicien sesión en PureCloud by Genesys, deben aprovisionarse en PureCloud by Genesys. En PureCloud by Genesys, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, siga estos pasos:**

1. Inicie sesión en PureCloud by Genesys como administrador.

1. Seleccione **Admin** (Administrador) en la parte superior y vaya a **People** (Personas) en **People & Permissions** (Personas y permisos).

    ![Configurar inicio de sesión único](./media/purecloud-by-genesys-tutorial/configure03.png)

1. En la página **People** (Personas), haga clic en **Add Person** (Agregar persona).

    ![Configurar inicio de sesión único](./media/purecloud-by-genesys-tutorial/configure04.png)

1. En el cuadro de diálogo **Add People to the Organization** (Agregar personas a la organización), siga estos pasos:

    ![Configurar inicio de sesión único](./media/purecloud-by-genesys-tutorial/configure05.png)

    a. En el cuadro **Full name** (Nombre completo), escriba el nombre del usuario. Por ejemplo: **B.Simon**.

    b. En el cuadro **Email** (Correo electrónico), escriba el correo electrónico del usuario. Por ejemplo, **b.simon\@contoso.com**.

    c. Seleccione **Crear**.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de **PureCloud by Genesys** en el Panel de acceso, debería iniciar sesión automáticamente en la cuenta de PureCloud by Genesys para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [ Lista de tutoriales para integrar aplicaciones SaaS con Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único en Azure AD?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure AD?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe PureCloud by Genesis con Azure AD](https://aad.portal.azure.com/)
