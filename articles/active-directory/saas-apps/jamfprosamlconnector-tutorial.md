---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Jamf Pro | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Jamf Pro.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/11/2020
ms.author: jeedes
ms.openlocfilehash: 780421d93916c7da7897dfa15d09dc895cf56280
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552669"
---
# <a name="tutorial-azure-active-directory-sso-integration-with-jamf-pro"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con Jamf Pro

En este tutorial, aprenderá a integrar Jamf Pro con Azure Active Directory (Azure AD). Al integrar Jamf Pro con Azure AD, puede hacer lo siguiente:

* Use Azure AD para controlar quién tiene acceso a Jamf Pro.
* Haga que los usuarios inicien sesión automáticamente en Jamf Pro con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de Jamf Pro habilitada para el inicio de sesión único (SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 

* Jamf Pro admite el **inicio de sesión único iniciado por SP** e **IDP**.
* Una vez configurado Jamf Pro puede aplicar el control de sesión que proteja el filtrado externo e interno de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="add-jamf-pro-from-the-gallery"></a>Incorporación de Jamf Pro desde la galería

Para configurar la integración de Jamf Pro en Azure AD, deberá agregar Jamf Pro desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel izquierdo, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba *Jamf Pro* en el cuadro de búsqueda.
1. Seleccione **Jamf Pro** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-sso-in-azure-ad-for-jamf-pro"></a>Configuración y prueba del inicio de sesión único de Azure AD para Jamf Pro

Configure y pruebe el inicio de sesión único de Azure AD con Jamf Pro mediante un usuario de prueba llamado B.Simon. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Jamf Pro.

En esta sección, puede configurar y probar el inicio de sesión único de Azure AD con Jamf Pro.

1. [Configure el inicio de sesión único en Azure AD](#configure-sso-in-azure-ad) para que los usuarios puedan usar esta característica.
    1. [Cree un usuario de prueba de Azure AD](#create-an-azure-ad-test-user) para probar el inicio de sesión único de Azure AD con el usuario B.Simon.
    1. [Asigne el usuario de prueba de Azure AD](#assign-the-azure-ad-test-user) para que B.Simon pueda usar el inicio de sesión único en Azure AD.
1. [Configure el inicio de sesión único en Jamf Pro](#configure-sso-in-jamf-pro) para configurar el inicio de sesión único en la aplicación.
    1. [Cree un usuario de prueba de Jamf Pro](#create-a-jamf-pro-test-user) para tener un homólogo de B.Simon en Jamf Pro que esté vinculado a la representación del usuario en Azure AD.
1. [Pruebe la configuración del inicio de sesión único](#test-the-sso-configuration) para comprobar que la configuración funciona.

## <a name="configure-sso-in-azure-ad"></a>Configuración del inicio de sesión único en Azure AD

En esta sección va a habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Jamf Pro**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edite la página de configuración básica de SAML.](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo **iniciado por IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL que use la siguiente fórmula: `https://<subdomain>.jamfcloud.com/saml/metadata`.

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL que use la siguiente fórmula: `https://<subdomain>.jamfcloud.com/saml/SSO`.

1. Seleccione **Establecer direcciones URL adicionales**. Si quiere configurar la aplicación en modo **iniciado por SP**, en el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL que use la siguiente fórmula: `https://<subdomain>.jamfcloud.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador, la URL de respuesta y la dirección URL de inicio de sesión reales. Obtendrá el valor del identificador real de la sección **Single Sign-On** (Inicio de sesión único) en el portal de Jamf Pro, que se explica más adelante en el tutorial. Puede extraer el valor real del subdominio del valor del identificador y usar esa información del subdominio como las direcciones URL de inicio de sesión y de respuesta. También puede consultar las fórmulas que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, vaya a la sección **Certificado de firma de SAML**, seleccione el botón de **copia** para copiar el valor de **Dirección URL de metadatos de federación de aplicación** y guárdelo en su equipo.

    ![Vínculo de descarga del certificado de firma de SAML](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.
   1. En el campo **Nombre de usuario**, escriba [nombre]@[dominioDeEmpresa].[extensión]. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección le concederá acceso a B.Simon a Jamf Pro.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Jamf Pro**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Seleccionar el botón Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista Usuarios y, luego, pulse el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario. Haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sso-in-jamf-pro"></a>Configuración del inicio de sesión único en Jamf Pro

1. Para automatizar la configuración de Jamf Pro, instale la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, seleccione **Instale la extensión**.

    ![Página de la extensión del explorador de inicio de sesión seguro de Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, seleccione **Configuración de Jamf Pro**. Cuando se abra la aplicación Jamf Pro, proporcione las credenciales de administrador para iniciar sesión. La extensión del explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Página de configuración de Jamf Pro](common/setup-sso.png)

3. Para configurar Jamf Pro manualmente, abra una nueva ventana del explorador web e inicie sesión en el sitio de empresa de Jamf Pro como administrador. A continuación, siga estos pasos.

4. Seleccione el icono de **configuración** en la esquina superior derecha de la página.

    ![Seleccionar el icono de configuración en Jamf Pro](./media/jamfprosamlconnector-tutorial/configure1.png)

5. Seleccione **Single Sign-On** (Inicio de sesión único).

    ![Seleccionar inicio de sesión único en Jamf Pro](./media/jamfprosamlconnector-tutorial/configure2.png)

6. En la página **Inicio de sesión único** (Inicio de sesión único), siga estos pasos.

    ![La página de inicio de sesión único de Jamf Pro](./media/jamfprosamlconnector-tutorial/configure3.png)

    a. Seleccione **Editar**.

    b. Seleccione la casilla **Enable Single Sign-On Authentication** (Habilitar autenticación de inicio de sesión único).

  c. Seleccione la opción **Azure** en el menú desplegable **Identity Provider** (Proveedor de identidades).

  d. Copie el valor **ENTITY ID** (ID. DE ENTIDAD) y péguelo en el campo **Identificador (id. de entidad)** de la sección **Configuración básica de SAML** en Azure Portal.

> [!NOTE]
> Use el valor del campo `<SUBDOMAIN>` para completar las direcciones URL de inicio de sesión y respuesta en la sección **Configuración básica de SAML** en Azure Portal.

  e. Seleccione **Metadata URL** (URL de metadatos) en el menú desplegable **Identity Provider Metadata Source** (Origen de metadatos del proveedor de identidades). En el campo que aparece, pegue el valor de **Dirección URL de metadatos de federación de aplicación** que copió de Azure Portal.

  f. (Opcional) Edite el valor de expiración del token o seleccione "Disable SAML token expiration" (Deshabilitar expiración de token SAML).

7. En la misma página, desplácese hacia abajo hasta la sección **User Mapping** (Asignación de usuarios). A continuación, siga estos pasos.

    ![Sección User Mapping (Asignación de usuarios) de la página de inicio de sesión único de Jamf Pro.](./media/jamfprosamlconnector-tutorial/tutorial-jamfprosamlconnector-single.png)

    a. Seleccione la opción **NameID** para **Identity Provider User Mapping** (Asignación de usuario del proveedor de identidades). De forma predeterminada, esta opción se establece en **NameID**, pero se puede definir un atributo personalizado.

    b. Seleccione **Email** (Correo electrónico) para **Jamf Pro User Mapping** (Asignación de usuario de Jamf Pro). Jamf Pro asigna los atributos SAML que envía el proveedor de identidades primero por usuarios y, después, por grupos. Cuando un usuario intenta acceder a Jamf Pro, este obtiene información sobre el usuario del proveedor de identidades y la compara con las cuentas de usuario de Jamf Pro. Si no se encuentra la cuenta de usuario entrante, Jamf Pro intentará encontrar una coincidencia por el nombre del grupo.

    c. Pegue el valor `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups` en el campo **IDENTITY PROVIDER GROUP ATTRIBUTE NAME** (NOMBRE DE ATRIBUTO DEL PROVEEDOR DE IDENTIDADES).

    d. En la misma página, desplácese hacia abajo hasta la sección **Security** (Seguridad) y seleccione **Allow users to bypass the Single Sign-On authentication** (Permitir a los usuarios omitir la autenticación de inicio de sesión único). Como resultado, no se redirigirá a los usuarios a la página de inicio de sesión del proveedor de identidades para la autenticación y, en su lugar, podrán iniciar sesión en Jamf Pro directamente. Cuando un usuario intenta tener acceso a Jamf Pro a través del proveedor de identidades, se produce la autorización y autenticación mediante SSO iniciadas por el proveedor de identidades.

    e. Seleccione **Guardar**.

### <a name="create-a-jamf-pro-test-user"></a>Crear un usuario de prueba de Jamf Pro

Para que los usuarios de Azure AD inicien sesión en Jamf Pro, tienen que aprovisionarse en Jamf Pro. En el caso de Jamf Pro, el aprovisionamiento es una tarea manual.

Para aprovisionar una cuenta de usuario, realice estos pasos:

1. Inicie sesión en su sitio de la empresa de Jamf Pro como administrador.

2. Seleccione el icono **Settings** (Configuración) en la esquina superior derecha de la página.

    ![Icono de configuración de Jamf Pro](./media/jamfprosamlconnector-tutorial/configure1.png)

3. Seleccione **Jamf Pro User Accounts & Groups** (Grupos y cuentas de usuario de Jamf Pro).

    ![Icono Jamf Pro User Accounts & Groups (Grupos y cuentas de usuario de Jamf Pro) en la configuración de Jamf Pro](./media/jamfprosamlconnector-tutorial/user1.png)

4. Seleccione **Nuevo**.

    ![Página de configuración del sistema Jamf Pro User Accounts & Groups (Grupos y cuentas de usuario de Jamf Pro)](./media/jamfprosamlconnector-tutorial/user2.png)

5. Seleccione **Crear cuenta estándar**.

    ![Opción Create Standard Account (Crear cuenta estándar) en la página Jamf Pro User Accounts & Groups (Grupos y cuentas de usuario de Jamf Pro)](./media/jamfprosamlconnector-tutorial/user3.png)

6. En el cuadro de diálogo **New Account** (Cuenta nueva) realice los siguientes pasos:

    ![Nuevas opciones de configuración de cuenta en la configuración del sistema de Jamf Pro](./media/jamfprosamlconnector-tutorial/user4.png)

    a. En el campo **USERNAME** (NOMBRE DE USUARIO), escriba `Britta Simon`, el nombre completo del usuario de prueba.

    b. Seleccione las opciones **ACCESS LEVEL**, **PRIVILEGE SET**, and **ACCESS STATUS** (NIVEL DE ACCESO, CONJUNTO DE PRIVILEGIOS y ESTADO DE ACCESO) que cumplan las directrices de la organización.

    c. En el campo **FULL NAME** (NOMBRE COMPLETO), escriba `Britta Simon`.

    d. En el campo **EMAIL ADDRESS** (DIRECCIÓN DE CORREO ELECTRÓNICO), escriba la dirección de correo electrónico de la cuenta de Britta Simon.

    e. En el campo **PASSWORD** (CONTRASEÑA), escriba la contraseña del usuario.

    f. En el campo **VERIFY PASSWORD** (COMPROBAR CONTRASEÑA), escriba la contraseña del usuario.

    g. Seleccione **Guardar**.

## <a name="test-the-sso-configuration"></a>Prueba de la configuración de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Jamf Pro en el panel de acceso y debería iniciar sesión automáticamente en la cuenta de Jamf Pro para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
- [Prueba de Jamf Pro con Azure AD](https://aad.portal.azure.com/)