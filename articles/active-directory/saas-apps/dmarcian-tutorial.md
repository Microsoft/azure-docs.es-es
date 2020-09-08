---
title: 'Tutorial: Integración de Azure Active Directory con dmarcian | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y dmarcian.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/01/2019
ms.author: jeedes
ms.openlocfilehash: 8868b17766513ba1e93b25bf2aeff6553c62ba62
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88536162"
---
# <a name="tutorial-integrate-dmarcian-with-azure-active-directory"></a>Tutorial: Integración de dmarcian con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar dmarcian con Azure Active Directory (Azure AD). Al integrar dmarcian con Azure AD, puede hacer lo siguiente:

* Controle en Azure AD quién tiene acceso a dmarcian.
* Permita que los usuarios inicien sesión automáticamente en dmarcian con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en dmarcian.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* dmarcian admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-dmarcian-from-the-gallery"></a>Adición de dmarcian desde la galería

Para configurar la integración de dmarcian en Azure AD, es preciso agregar dmarcian desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **dmarcian** en el cuadro de búsqueda.
1. Seleccione **dmarcian** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con dmarcian utilizando un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de dmarcian.

Para configurar y probar el inicio de sesión único de Azure AD con dmarcian, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración del inicio de sesión único en dmarcian](#configure-dmarcian-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en dmarcian](#create-dmarcian-test-user)** : para tener un homólogo de B. Simon en dmarcian que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **dmarcian**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:

    ```http
    https://us.dmarcian.com/sso/saml/<ACCOUNT_ID>/sp.xml
    https://dmarcian-eu.com/sso/saml/<ACCOUNT_ID>/sp.xml
    https://dmarcian-ap.com/sso/saml/<ACCOUNT_ID>/sp.xml
    ```

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    ```http
    https://us.dmarcian.com/login/<ACCOUNT_ID>/handle/
    https://dmarcian-eu.com/login/<ACCOUNT_ID>/handle/
    https://dmarcian-ap.com/login/<ACCOUNT_ID>/handle/
    ```

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:
    
    ```http
    https://us.dmarcian.com/login/<ACCOUNT_ID>
    https://dmarcian-eu.com/login/<ACCOUNT_ID>
    https://dmarciam-ap.com/login/<ACCOUNT_ID>
    ```
     
    > [!NOTE] 
    > Estos valores no son reales. Los valores se actualizan con el identificador, la dirección URL de inicio de sesión y la dirección URL de respuesta reales, lo que se explica más adelante en el tutorial.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-dmarcian-sso"></a>Configuración del inicio de sesión único de dmarcian

1. Para automatizar la configuración en dmarcian, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar dmarcian** para ir a la aplicación dmarcian. En ella, escriba las credenciales de administrador para iniciar sesión en dmarcian. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar dmarcian manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de dmarcian o administrador y haga lo siguiente:

4. Haga clic en **Profile** (Perfil) en la esquina superior derecha y vaya a **Preferences** (Preferencias).

    ![Preferencias](./media/dmarcian-tutorial/tutorial_dmarcian_pref.png)

5. Desplácese hacia abajo y haga clic en la sección **Single Sign-On** (Inicio de sesión único) y, después, haga clic en **Configure** (Configurar).

    ![El inicio de sesión único](./media/dmarcian-tutorial/tutorial_dmarcian_sso.png)

6. En la página **SAML Single Sign-On** (Inicio de sesión único de SAML), establezca **Status** (Estado) en **Enabled** (Habilitado) y realice los pasos siguientes:

    ![Autenticación](./media/dmarcian-tutorial/tutorial_dmarcian_auth.png)

    * En la sección **Add dmarcian to your Identity Provider** (Agregar dmarcian al proveedor de identidades), haga clic en **COPY** para copiar el valor de **Assertion Consumer Service URL** (Dirección URL del servicio de consumidor de aserciones) de la instancia y péguelo en el cuadro de texto **Dirección URL de respuesta** de la sección **Configuración básica de SAML** en Azure Portal.

    * En la sección **Add dmarcian to your Identity Provider** (Agregar dmarcian al proveedor de identidades), haga clic en **COPY** para copiar el valor de **Entity ID** (Identificador de entidad) de la instancia y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    * En la sección **Set up Authentication** (Configurar la autenticación), en el cuadro de texto **Identity Provider Metadata** (Metadatos del proveedor de identidades), pegue el valor de **Dirección URL de metadatos de federación de aplicación** que copió de Azure Portal.

    * En la sección **Set up Authentication** (Configurar autenticación), en el cuadro de texto **Attribute Statements** (Instrucciones de atributo), pegue la dirección URL `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    * En la sección **Set up Login URL** (Configurar dirección URL de inicio de sesión), copie el valor de **Login URL** (Dirección URL de inicio de sesión) de la instancia y péguelo en el cuadro de texto **Dirección URL de inicio de sesión** de la sección **Configuración básica de SAML** en Azure Portal.

        > [!Note]
        > Puede modificar la **dirección URL de inicio de sesión** según su organización.

    * Haga clic en **Save**(Guardar).

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

En esta sección, va a permitir que B. Simon acceda a dmarcian mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **dmarcian**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-dmarcian-test-user"></a>Creación de un usuario de prueba en dmarcian

Para permitir que los usuarios de Azure AD inicien sesión en dmarcian, deben aprovisionarse en dmarcian. En dmarcian, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en dmarcian como administrador de seguridad.

2. Haga clic en **Profile** (Perfil) en la esquina superior derecha y vaya a **Manage Users** (Administrar usuarios).

    ![Usuario](./media/dmarcian-tutorial/tutorial_dmarcian_user.png)

3. En el lado derecho de la sección **SSO Users** (Usuarios de SSO), haga clic en **Add New User** (Agregar nuevo usuario).

    ![Agregar usuario](./media/dmarcian-tutorial/tutorial_dmarcian_addnewuser.png)

4. En la ventana emergente **Add New user** (Agregar nuevo usuario), realice los siguientes pasos:

    ![Nuevo usuario](./media/dmarcian-tutorial/tutorial_dmarcian_save.png)

    a. En el cuadro de texto **New User Email** (Correo electrónico del nuevo usuario), escriba el correo electrónico del usuario; por ejemplo, **brittasimon\@contoso.com**.

    b. Si desea conceder derechos de administrador al usuario, seleccione **Make User an Admin** (Hacer que el usuario sea un administrador).

    c. Haga clic en **Agregar usuario**.

### <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de dmarcian en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de dmarcian para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

