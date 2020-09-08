---
title: 'Tutorial: Integración de Azure Active Directory con Syncplicity | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Syncplicity.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2019
ms.author: jeedes
ms.openlocfilehash: da532adfa2d4ab97edb44de9ae49c646ccdff381
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544900"
---
# <a name="tutorial-integrate-syncplicity-with-azure-active-directory"></a>Tutorial: Integración de Syncplicity con Azure Active Directory

En este tutorial, aprenderá a integrar Syncplicity con Azure Active Directory (Azure AD). Al integrar Syncplicity con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Syncplicity.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Syncplicity con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede obtener la evaluación gratuita de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Syncplicity.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. Syncplicity admite inicio de sesión único iniciado por **SP**.

## <a name="adding-syncplicity-from-the-gallery"></a>Incorporación de Syncplicity desde la galería

Para configurar la integración de Syncplicity en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Syncplicity** en el cuadro de búsqueda.
1. Seleccione **Syncplicity** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Syncplicity mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Syncplicity.

Para configurar y probar el inicio de sesión único de Azure AD con Syncplicity, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración del inicio de sesión único en Syncplicity](#configure-syncplicity-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Syncplicity](#create-syncplicity-test-user)** : para tener un homólogo de B.Simon en Syncplicity vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Syncplicity**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.syncplicity.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<companyname>.syncplicity.com/sp`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de Syncplicity](https://www.syncplicity.com/contact-us) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configuración de Syncplicity**, copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-syncplicity-sso"></a>Configuración del inicio de sesión único de Syncplicity

1. Inicie sesión en su inquilino de **Syncplicity** .

1. En el menú en la parte superior, haga clic en **admin** (Administración), seleccione **settings** (Configuración) y luego haga clic en **Custom domain and single sign-on** (Dominio personalizado e inicio de sesión único).

    ![Syncplicity](./media/syncplicity-tutorial/ic769545.png "Syncplicity")

1. En la página del cuadro de diálogo **Single Sign-On (SSO)** (Configuración de inicio de sesión único [SSO]), siga estos pasos:

    ![Inicio de sesión único \(SSO\)](./media/syncplicity-tutorial/ic769550.png "Single Sign-On \\\(SSO\\\)")

    a. En el cuadro de texto **Custom Domain** (Dominio personalizado), escriba el nombre de su dominio.
  
    b. Seleccione **Enabled** (Habilitado) como **Single Sign-On Status** (Estado de inicio de sesión único).

    c. En el cuadro de texto **Identificador de entidad**, pegue el valor del **identificador (identificador de entidad)** que usó en la **configuración básica de SAML** de Azure Portal.

    d. En el cuadro de texto **Dirección URL de la página de inicio de sesión**, pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    e. En el cuadro de texto **Dirección URL de la página de cierre de sesión**, pegue el valor de la **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    f. En **Identity Provider Certificate** (Certificado del proveedor de identidades), haga clic en **Elegir archivo** y cargue el certificado que ha descargado de Azure Portal.

    g. Haga clic en **GURDAR CAMBIOS**.

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

En esta sección, va a permitir que B.Simon acceda a Syncplicity utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Syncplicity**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-syncplicity-test-user"></a>Creación de un usuario de prueba de Syncplicity

Para que los usuarios de Azure AD puedan iniciar sesión, estos deben aprovisionarse en la aplicación Syncplicity. En esta sección se describe cómo crear cuentas de usuario de Azure AD en Syncplicity.

**Para aprovisionar cuentas de usuario en Syncplicity, realice los siguientes pasos:**

1. Inicie sesión en su inquilino de **Syncplicity** (por ejemplo, `https://company.Syncplicity.com`).

1. Haga clic en **admin** y seleccione **cuentas de usuario** y, a continuación, haga clic en **ADD A USER** (AGREGAR UN USUARIO).

    ![Administración de usuarios](./media/syncplicity-tutorial/ic769764.png "Administrar usuarios")

1. Escriba las **Email Addresses** (Direcciones de correo electrónico) de una cuenta de Azure AD que quiera aprovisionar, seleccione **User** (Usuario) como **Role** (Rol) y haga clic en **NEXT** (SIGUIENTE).

    ![Información de cuenta](./media/syncplicity-tutorial/ic769765.png "Información de cuenta")

    > [!NOTE]
    > El titular de la cuenta de Azure AD recibirá un mensaje de correo electrónico junto con un vínculo para confirmar y activar la cuenta.

1. Seleccione un grupo de la compañía de la que debe convertirse en miembro su nuevo usuario y haga clic en **NEXT**(SIGUIENTE).

    ![Pertenencia a grupos](./media/syncplicity-tutorial/ic769772.png "Pertenencia a grupos")

    > [!NOTE]
    > Si no se muestra ningún grupo, haga clic en **NEXT**(SIGUIENTE).

1. Seleccione las carpetas que desea colocar bajo el control de Syncplicity en el equipo del usuario y haga clic en **NEXT**(SIGUIENTE).

    ![Carpetas de Syncplicity](./media/syncplicity-tutorial/ic769773.png "Carpetas de Syncplicity")

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Syncplicity ofrecida por Syncplicity para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Syncplicity en el panel de acceso y debería iniciar sesión automáticamente en la versión de Syncplicity para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)