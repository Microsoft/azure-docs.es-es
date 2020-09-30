---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Freshservice | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Freshservice.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: jeedes
ms.openlocfilehash: 8767444e7ea839b1bb89cbb52833283fa01f7b5a
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90055595"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-freshservice"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Freshservice

En este tutorial, aprenderá a integrar Freshservice con Azure Active Directory (Azure AD). Cuando integre Freshservice con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Freshservice.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Freshservice con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en FreshService (SSO).

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Freshservice admite el inicio de sesión único iniciado por **SP**
* Una vez configurado Freshservice, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-freshservice-from-the-gallery"></a>Agregar Freshservice desde la galería

Para configurar la integración de Freshservice en Azure AD, deberá agregar Freshservice desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Freshservice** en el cuadro de búsqueda.
1. Seleccione **Freshservice** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-freshservice"></a>Configuración y prueba del inicio de sesión único de Azure AD para Freshservice

Configure y pruebe el inicio de sesión único de Azure AD con Freshservice utilizando un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Freshservice.

Para configurar y probar el inicio de sesión único de Azure AD con Freshservice, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Freshservice](#configure-freshservice-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Freshservice](#create-freshservice-test-user)** , para tener un homólogo de B.Simon en Freshservice que esté vinculado a la representación de Azure AD de usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Freshservice**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<company-name>.freshservice.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<company-name>.freshservice.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<company-name>.freshservice.com/login/saml`
    
    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de clientes de Freshservice](https://support.freshservice.com/). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Freshservice** (Configurar Freshservice) de **Azure Portal**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, permitirá que B.Simon acceda a Freshservice para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Freshservice**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-freshservice-sso"></a>Configuración del inicio de sesión único de Freshservice

1. Para automatizar la configuración en Freshservice, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Setup Freshservice** (Configurar Freshservice) para acceder a la aplicación Freshservice. Una vez allí, especifique las credenciales del administrador para iniciar sesión en Freshservice. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

1. Si desea configurar Freshservice manualmente, inicie sesión en el sitio de la empresa Freshservice como administrador.

1. En el menú de la izquierda, haga clic en **Admin** (Administrador) y seleccione **Helpdesk Security** (Seguridad del departamento de soporte técnico) en **General Settings** (Configuración general).

    ![Administrador](./media/freshservice-tutorial/configure-1.png "Administración")

1. En **Security** (Seguridad), haga clic en **Go to Freshworks 360 Security** (Ir a la seguridad de Freshworks 360).

    ![Seguridad](./media/freshservice-tutorial/configure-2.png "Seguridad")

1. En la sección **Seguridad** , realice estos pasos:

    ![Inicio de sesión único](./media/freshservice-tutorial/configure-3.png "Inicio de sesión único")
  
    a. En **Single Sign On** (Inicio de sesión único), seleccione **On** (Activado).

    b. En **Login Method** (Método de inicio de sesión), seleccione **SAML SSO** (SSO de SAML).

    c. En el cuadro de texto **Entity ID provided by the IdP** (Identificador de entidad proporcionado por el IdP), pegue el valor de **Entity ID** (Id. de entidad) que ha copiado de Azure Portal.

    d. En el cuadro de texto **SAML SSO URL** (Dirección URL de inicio de sesión único de SAML), pegue el valor de **URL de inicio de sesión** que copió en Azure Portal.

    e. En **Signing Options** (Opciones de firma), seleccione **Only Signed Assertions** (Solo aserciones firmadas) en la lista desplegable.

    f. En el cuadro de texto **Sign Out URL** (URL de cierre de sesión), pegue el valor de **Logout URL** (Dirección URL de cierre de sesión) que ha copiado de Azure Portal.

    g. En el cuadro de texto **Security Certificate** (Certificado de seguridad), pegue el valor de **Certificate (Base64)** (Certificado [Base64]), que ha obtenido anteriormente.
  
    h. Haga clic en **Save**(Guardar).


## <a name="create-freshservice-test-user"></a>Creación de un usuario de prueba de Freshservice

Para permitir que los usuarios de Azure AD inicien sesión en FreshService, deben aprovisionarse en FreshService. En el caso de FreshService, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en el sitio de la compañía de **FreshService** como administrador.

2. En el menú de la izquierda, haga clic en **Admin** (Administrador).

3. En la sección **User Management** (Administración de usuarios), haga clic en **Requesters** (Solicitantes).

    ![Solicitantes](./media/freshservice-tutorial/create-user-1.png "Solicitantes")

4. Haga clic en **Nuevo solicitante**.

    ![Nuevos solicitantes](./media/freshservice-tutorial/create-user-2.png "Nuevos solicitantes")

5. En la sección **New Requester** (Nuevo solicitante), rellene los campos necesarios y haga clic en **Save** (Guardar).
    ![Nuevo solicitante](./media/freshservice-tutorial/create-user-3.png "Nuevo solicitante")  

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibe un mensaje de correo electrónico con un vínculo para confirmar la cuenta antes de que se active.
    >  

    > [!NOTE]
    > Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de FreshService que proporcione FreshService para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Freshservice en el panel de acceso, debería iniciar sesión automáticamente en la versión de Freshservice para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Probar Freshservice con Azure AD](https://aad.portal.azure.com/)