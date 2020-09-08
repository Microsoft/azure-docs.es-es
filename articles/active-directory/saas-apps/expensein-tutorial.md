---
title: 'Tutorial: Integración de Azure Active Directory con ExpenseIn | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y ExpenseIn.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/17/2020
ms.author: jeedes
ms.openlocfilehash: 7d0b0a6c935182737a7f207b7c7831fefcd491ea
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555336"
---
# <a name="tutorial-integrate-expensein-with-azure-active-directory"></a>Tutorial: Integración de ExpenseIn con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar ExpenseIn con Azure Active Directory (Azure AD). Al integrar ExpenseIn con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a ExpenseIn.
* Permitir que los usuarios puedan iniciar sesión automáticamente en ExpenseIn con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de ExpenseIn con el inicio de sesión único (SSO) habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 
* ExpenseIn admite el inicio de sesión único iniciado por **SP e IDP**.
* Una vez que configure ExpenseIn, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


## <a name="adding-expensein-from-the-gallery"></a>Adición de ExpenseIn desde la galería

Para configurar la integración de ExpenseIn en Azure AD, deberá agregar ExpenseIn desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ExpenseIn** en el cuadro de búsqueda.
1. Seleccione **ExpenseIn** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-expensein"></a>Configuración y prueba del inicio de sesión único de Azure AD para ExpenseIn

Configure y pruebe el inicio de sesión único de Azure AD con ExpenseIn utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de ExpenseIn.

Para configurar y probar el inicio de sesión único de Azure AD con ExpenseIn, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en ExpenseIn](#configure-expensein-sso)** , para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en ExpenseIn](#create-expensein-test-user)** , para tener un homólogo de B.Simon en ExpenseIn que esté vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **ExpenseIn**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://app.expensein.com/saml`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **dirección URL de metadatos de federación de aplicación** y haga clic en **Descargar** para descargar el **certificado (Base64)** y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](./media/expensein-tutorial/copy-metdataurl-certificate.png)

1. En la sección **Set up ExpenseIn** (Configurar ExpenseIn), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a ExpenseIn mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ExpenseIn**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.


## <a name="configure-expensein-sso"></a>Configuración del inicio de sesión único de ExpenseIn

1. Para automatizar la configuración en ExpenseIn, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar ExpenseIn** para ir a la aplicación ExpenseIn. En ella, escriba las credenciales de administrador para iniciar sesión en ExpenseIn. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

1. Si desea configurar ExpenseIn manualmente, inicie sesión en el sitio de la empresa ExpenseIn como administrador.

1. Haga clic en **Admin** en la parte superior de la página, a continuación, vaya a **Single Sign-On** (Inicio de sesión único) y haga clic en **Add provider** (Agregar proveedor).

     ![Configuración de ExpenseIn](./media/expenseIn-tutorial/config01.png)

1. En el elemento emergente **New Identity Provider** (Nuevo proveedor de identidades), realice los pasos siguientes:

    ![Configuración de ExpenseIn](./media/expenseIn-tutorial/config02.png)

    a. En el cuadro de texto **Provider Name** (Nombre del proveedor), escriba el nombre (por ejemplo, Azure).

    b. Seleccione **Sí** en **Allow Provider Initiated Sign-On** (Permitir el inicio sesión iniciado por el proveedor).

    c. En el cuadro de texto **Target Url** (Dirección URL de destino), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.

    e. Abra el certificado (Base64) en el Bloc de notas, copie su contenido y péguelo en el cuadro de texto **Certificate** (Certificado).

    f. Haga clic en **Crear**.

### <a name="create-expensein-test-user"></a>Creación de un usuario de prueba de ExpenseIn

Para permitir que los usuarios de Azure AD inicien sesión en ExpenseIn, deben aprovisionarse en ExpenseIn. En ExpenseIn, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en ExpenseIn como administrador.

2. Haga clic en **Admin** en la parte superior de la página, a continuación, vaya a **Users** (Usuarios) y haga clic en **New User** (Nuevo usuario).

     ![Configuración de ExpenseIn](./media/expenseIn-tutorial/config03.png)

3. En el elemento emergente **Detalles**, lleve a cabo los pasos siguientes:

    ![Configuración de ExpenseIn](./media/expenseIn-tutorial/config04.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **B**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

    c. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario, por ejemplo, `B.Simon@contoso.com`.

    d. Haga clic en **Crear**.

## <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de ExpenseIn en el panel de acceso, debería iniciar sesión automáticamente en la versión de ExpenseIn para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de ExpenseIn con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de ExpenseIn con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
