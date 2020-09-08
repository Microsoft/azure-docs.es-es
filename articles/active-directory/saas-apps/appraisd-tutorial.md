---
title: 'Tutorial: Integración de Azure Active Directory con Appraisd | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Appraisd.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/27/2019
ms.author: jeedes
ms.openlocfilehash: 2b8e8a6df2fc355ad6a8ef50d801dc7457e97c63
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547790"
---
# <a name="tutorial-integrate-appraisd-with-azure-active-directory"></a>Tutorial: Integración de Appraisd con Azure Active Directory

En este tutorial, aprenderá a integrar Appraisd con Azure Active Directory (Azure AD). Al integrar Appraisd con Azure AD, puede hacer lo siguiente:

* Controle en Azure AD quién tiene acceso a Appraisd.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Appraisd con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único de Appraisd.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. Appraisd admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-appraisd-from-the-gallery"></a>Adición de Appraisd desde la galería

Para configurar la integración de Appraisd en Azure AD, será preciso que agregue Appraisd desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Appraisd** en el cuadro de búsqueda.
1. Seleccione **Appraisd** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Appraisd utilizando un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Appraisd.

Para configurar y probar el inicio de sesión único de Azure AD con Appraisd, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración de Appraisd](#configure-appraisd)** , para configurar las opciones del inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B. Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B. Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Appraisd](#create-appraisd-test-user)** : para tener un homólogo de B. Simon en Appraisd que esté vinculado a la representación de ella en Azure AD.
6. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Appraisd**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada y las direcciones URL necesarias ya se han rellenado previamente con Azure. Para guardar la configuración, el usuario debe hacer clic en el botón Guardar y realizar los pasos siguientes:

    a. Haga clic en **Establecer direcciones URL adicionales**.

    b. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL: `<TENANTCODE>`

    c. Si quiere configurar la aplicación en modo iniciado por **SP**, en el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL siguiendo este patrón: `https://app.appraisd.com/saml/<TENANTCODE>`

    > [!NOTE]
    > La URL de inicio de sesión y el valor del estado de retransmisión se obtienen en la página de configuración de SSO de Appraisd, que se explica más adelante en el tutorial.

1. La aplicación Appraisd espera las aserciones de SAML en un formato específico, lo que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde **nameidentifier** se asigna con **user.userprincipalname**. La aplicación Appraisd espera que **nameidentifier** se asigne con **user.mail**, por lo que debe editar la asignación de atributos haciendo clic en el icono **Editar** y cambiar esa asignación.

    ![imagen](common/edit-attribute.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Appraisd** (Configurar Appraisd), copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-appraisd"></a>Configuración de Appraisd

1. Para automatizar la configuración en Appraisd, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Setup Appraisd** (Configurar Appraisd) para ir a la aplicación Appraisd. Desde allí, proporcione las credenciales de administrador para iniciar sesión en Appraisd. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Appraisd manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Appraisd como administrador y haga lo siguiente:

4. En la parte superior derecha de la página, haga clic en el icono **Configuración** y, después, vaya a **Configuración**.

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_sett.png)

5. En el lado izquierdo del menú, haga clic en el **inicio de sesión único SAML**.

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_single.png)

6. En la página **Configuración de inicio de sesión único de SAML 2.0**, siga estos pasos:

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_saml.png)

    a. Copie el valor **Default Relay State** (Estado retransmisión predeterminado) y péguelo en el cuadro de texto **Estado de la retransmisión** de **Basic SAML Configuration** (Configuración básica de SAML) en Azure Portal.

    b. Copie el valor **Service-initiated login URL** (URL de inicio de sesión iniciado por el servicio) y péguelo en el cuadro de texto **URL de inicio de sesión** de **Basic SAML Configuration** (Configuración básica de SAML) en Azure Portal.

7. En la página **Identificación de usuarios**, desplácese hacia abajo y realice los pasos siguientes:

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_identifying.png)

    a. En el cuadro de texto **Dirección URL del inicio de sesión único del proveedor de identidades**, pegue el valor de **URL de inicio de sesión** que ha copiado de Azure Portal y haga clic en **Guardar**.

    b. En el cuadro de texto **Identity Provider Issuer URL** (Dirección URL del emisor del proveedor de identidades), pegue el valor de **Azure AD Identifier** (Identificador de Azure AD) que ha copiado de Azure Portal y haga clic en **Save** (Guardar).

    c. En el Bloc de notas, abra el certificado codificado en Base 64 que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509) y haga clic en **Guardar**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B. Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B. Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B. Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B. Simon para que use el inicio de sesión único de Azure concediendo acceso a Appraisd.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Appraisd**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B. Simon** en la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-appraisd-test-user"></a>Creación de un usuario de prueba de Appraisd

Para permitir que los usuarios de Azure AD inicien sesión en Appraisd, deben aprovisionarse en Appraisd. En Appraisd, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Appraisd como administrador de seguridad.

2. En la parte superior derecha de la página, haga clic en el icono **Settings** (Configuración) y, después, vaya al **Centro de administración**.

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_admin.png)

3. En la barra de herramientas situada en la parte superior de la página, haga clic en **Personas** y, después, vaya a **Agregar un nuevo usuario**.

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_user.png)

4. En la página **Agregar nuevo usuario**, realice los pasos siguientes:

    ![imagen](./media/appraisd-tutorial/tutorial_appraisd_newuser.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **Britta**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba los apellidos del usuario, en este caso **Simon**.

    c. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario, por ejemplo, `B. Simon@contoso.com`.

    d. Haga clic en **Agregar usuario**.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Appraisd en el panel de acceso y debería iniciar sesión automáticamente en la versión de Appraisd para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
