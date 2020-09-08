---
title: 'Tutorial: Integración de Azure Active Directory con GitHub | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y GitHub.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/07/2020
ms.author: jeedes
ms.openlocfilehash: 7153a1279785cfe79d23b2b5ba843ec9f5cd4965
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550753"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub

En este tutorial, aprenderá a integrar GitHub con Azure Active Directory (Azure AD). Al integrar GitHub con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a la organización de GitHub Enterprise Cloud.
* Administrar el acceso a la organización de GitHub Enterprise Cloud en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con GitHub, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una organización GitHub creada en [GitHub Enterprise Cloud](https://help.github.com/articles/github-s-products/#github-enterprise), que requiere el [plan de facturación de GitHub Enterprise](https://help.github.com/articles/github-s-billing-plans/#billing-plans-for-organizations).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* GitHub admite el inicio de sesión único iniciado por **SP**

* GitHub admite el [**aprovisionamiento de usuarios (invitaciones de la organización)** automático](github-provisioning-tutorial.md).
* Una vez que haya configurado GitHub, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-github-from-the-gallery"></a>Adición de GitHub desde la galería

Para configurar la integración de GitHub en Azure AD, debe agregar GitHub desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **GitHub** en el cuadro de búsqueda.
1. Seleccione **GitHub** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-github"></a>Configuración y prueba del inicio de sesión único de Azure AD para GitHub

Configure y pruebe el inicio de sesión único de Azure AD con GitHub mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de GitHub.

Para configurar y probar el inicio de sesión único de Azure AD con GitHub, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en GitHub](#configure-github-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en GitHub](#create-github-test-user)** , para tener un homólogo de B.Simon en GitHub vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **GitHub**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://github.com/orgs/<Organization ID>/sso`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://github.com/orgs/<Organization ID>`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://github.com/orgs/<Organization ID>/saml/consume`


    > [!NOTE]
    > Tenga en cuenta que estos no son valores reales. Tendrá que actualizar estos valores con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Aquí le recomendamos que utilice el valor de cadena único en el identificador. Vaya a la sección de administración de GitHub para recuperar estos valores.

5. La aplicación GitHub espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde a **Unique User Identifier (Name ID)** (Identificador de usuario único [Identificador de nombre]) se le ha asignado **user.userprincipalname**. La aplicación GitHub espera que al **identificador de usuario único (Identificador de nombre)** se le haya asignado **user.mail**, por lo que se debe editar la asignación de atributos. Para ello, haga clic en el icono **Editar** y cambie la asignación de atributos.

    ![imagen](common/edit-attribute.png)

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up GitHub** (Configurar GitHub), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

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

En esta sección va a permitir que B.Simon acceda a GitHub mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **GitHub**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccione un rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

    ![rol de usuario](./media/github-tutorial/user-role.png)

    > [!NOTE]
    > La opción **Seleccione un rol** se deshabilitará y el rol predeterminado es USUARIO para el usuario seleccionado.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-github-sso"></a>Configuración del inicio de sesión único de GitHub

1. En otra ventana del explorador web, inicie sesión en el sitio de la organización de GitHub como administrador.

2. Vaya a **Configuración** y haga clic en **Seguridad**

    ![Configuración](./media/github-tutorial/security.png)

3. Active la casilla **Habilitar autenticación SAML** para ver los campos de configuración del inicio de sesión único. Realice los siguientes pasos:

    ![Configuración](./media/github-tutorial/saml-sso.png)

    a. Copie el valor de **Single sign-on URL** (Dirección URL de inicio de sesión único) y pegue el valor en el cuadro de texto **URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.
    
    b. Copie el valor de **Assertion Consumer Service URL** (Dirección URL del Servicio de consumidor de aserciones) y péguelo en el cuadro de texto **Dirección URL de respuesta** de **Configuración básica de SAML** en Azure Portal.

4. Configure los campos siguientes:

    ![Configuración](./media/github-tutorial/configure.png)

    a. En el cuadro de texto **Sign-On URL** (Dirección URL de inicio de sesión), pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Emisor**, pegue el valor del **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. Abra el certificado descargado de Azure Portal en el Bloc de notas y pegue el contenido en el cuadro de texto **Certificado público**.

    d. Haga clic en el icono **Editar** para editar el **Método de firma** y el **Método de resumen** desde **RSA-SHA1** y **SHA1** hasta **RSA-SHA256** y **SHA256** tal como se muestra a continuación.
    
    e. Actualice la **dirección URL del servicio de consumidor de aserciones (URL de respuesta)** con respecto a la dirección URL predeterminada, con el fin de que la dirección URL de GitHub coincida con la del registro de aplicaciones de Azure.

    ![imagen](./media/github-tutorial/tutorial_github_sha.png)

5. Haga clic en **Test SAML configuration** (Probar configuración de SAML) para configura que no hay errores de validación durante el SSO.

    ![Configuración](./media/github-tutorial/test.png)

6. Haga clic en **Guardar**

> [!NOTE]
> El inicio de sesión único en GitHub realiza la autenticación en una organización específica de GitHub, pero no reemplaza la autenticación propia de GitHub. Por tanto, si la sesión del usuario en github.com ha expirado, puede que se le pida que se autentique con la contraseña o el identificador de GitHub durante el proceso de inicio de sesión único.

### <a name="create-github-test-user"></a>Creación de un usuario de prueba en GitHub

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en GitHub. GitHub admite el aprovisionamiento automático de usuarios, que está habilitado de forma predeterminada. [Aquí](github-provisioning-tutorial.md) puede encontrar más información sobre cómo configurar el aprovisionamiento automático de usuarios.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. Inicie sesión en el sitio de la empresa de GitHub como administrador.

2. Haga clic en **Contactos**.

    ![Contactos](./media/github-tutorial/people.png "Personas")

3. Haga clic en **Invitar a miembros**.

    ![Invite Users (Invitar a usuarios)](./media/github-tutorial/invite-member.png "Invitar a usuarios")

4. En la página de diálogo **Invitar a miembros**, realice los siguientes pasos:

    a. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de la cuenta de Britta Simon.

    ![Invite People (Invitar a personas)](./media/github-tutorial/email-box.png "Invitar a contactos")

    b. Haga clic en **Enviar invitación**.

    ![Invite People (Invitar a personas)](./media/github-tutorial/send-invitation.png "Invitar a contactos")

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de GitHub del panel de acceso, debería iniciar sesión automáticamente en la versión de GitHub para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe GitHub con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
