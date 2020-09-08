---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con PurelyHR | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PurelyHR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: jeedes
ms.openlocfilehash: 48db08f856407b2ceba32798ed0e39eab967b72d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553321"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-purelyhr"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con PurelyHR

En este tutorial, aprenderá a integrar PurelyHR con Azure Active Directory (Azure AD). Al integrar PurelyHR con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a PurelyHR.
* Permitir que los usuarios inicien sesión automáticamente en PurelyHR con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en PurelyHR.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* PurelyHR admite el inicio de sesión único iniciado por **SP e IDP**
* PurelyHR admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-purelyhr-from-the-gallery"></a>Adición de PurelyHR desde la galería

Para configurar la integración de PurelyHR en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **PurelyHR** en el cuadro de búsqueda.
1. Seleccione **PurelyHR** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-purelyhr"></a>Configuración y prueba del inicio de sesión único de Azure AD para PurelyHR

Configure y pruebe el inicio de sesión único de Azure AD con PurelyHR mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de PurelyHR.

Para configurar y probar el inicio de sesión único de Azure AD con PurelyHR, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en PurelyHR](#configure-purelyhr-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de PurelyHR](#create-purelyhr-test-user)** , para tener un homólogo de B. Simon en PurelyHR que esté vinculado a la representación de Azure AD de usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **PurelyHR**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyID>.purelyhr.com/sso-consume`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyID>.purelyhr.com/sso-initiate`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de PurelyHR](https://support.purelyhr.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar PurelyHR**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a PurelyHR mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **PurelyHR**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-purelyhr-sso"></a>Configuración del inicio de sesión único de PurelyHR

1. Para automatizar la configuración en PurelyHR, es preciso instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar PurelyHR** para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en PurelyHR. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

1. Si desea configurar PurelyHR manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de PurelyHR como administrador y haga lo siguiente:

1. Abra el **panel** en las opciones de la barra de herramientas y haga clic en **SSO Settings** (Configuración de SSO).

1. Pegue los valores en los cuadros tal y como se describe a continuación:

    ![Configurar inicio de sesión único](./media/purelyhr-tutorial/purelyhr-dashboard-sso-settings.png)  

    a. Abra el archivo **Certificate(Bas64)** que se ha descargado de Azure Portal en el bloc de notas y copie el valor del certificado. Pegue el valor copiado en el cuadro **X.509 Certificate** (Certificado X.509).

    b. En el cuadro **IdP Issuer URL** (URL del emisor de IdP), pegue el **identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro **IDP Endpoint URL** (URL de punto de conexión de IDP), pegue la **dirección URL de inicio de sesión** que copió de Azure Portal. 

    d. Active la casilla **Auto-Create Users** (Creación automática de usuarios) para habilitar el aprovisionamiento automático de usuarios en PurelyHR.

    e. Haga clic en **Guardar cambios** para guardar la configuración.

### <a name="create-purelyhr-test-user"></a>Creación de un usuario de prueba en PurelyHR

Este paso no suele ser necesario porque la aplicación admite aprovisionamiento de usuarios Just-In-Time. Si el aprovisionamiento automático de usuarios no se habilita, puede llevarse a cabo la creación manual de usuarios tal y como se describe a continuación.

Inicie sesión como administrador en su sitio de la compañía de Velpic SAML y realice los pasos siguientes:

1. Haga clic en la pestaña Manage (Administrar), vaya a la sección Users (Usuarios) y haga clic en el botón New (Nuevo) para agregar usuarios.

    ![agregar usuario](./media/velpicsaml-tutorial/velpic_7.png)

2. En el cuadro de diálogo **"Create New User"** (Crear nuevo usuario), realice los pasos siguientes.

    ![usuario](./media/velpicsaml-tutorial/velpic_8.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre de B.

    b. En el cuadro de texto **Apellido**, escriba el apellido Simon.

    c. En el cuadro de texto **User Name** (Nombre de usuario), escriba el nombre del usuario de B.Simon.

    d. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de la cuenta B.Simon@contoso.com.

    e. El resto de la información es opcional, puede rellenarla si es necesario.

    f. Haga clic en **GUARDAR**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de PurelyHR en el panel de acceso, debería iniciar sesión automáticamente en la versión de PurelyHR para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe PurelyHR con Azure AD](https://aad.portal.azure.com/)
