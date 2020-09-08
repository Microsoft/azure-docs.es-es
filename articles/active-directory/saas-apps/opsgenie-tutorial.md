---
title: 'Tutorial: Integración de Azure Active Directory con OpsGenie | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y OpsGenie.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: jeedes
ms.openlocfilehash: 0ebcb746e10ae910c695e52fb053fc5d0b316e70
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543914"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-opsgenie"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con OpsGenie

En este tutorial aprenderá a integrar OpsGenie con Azure Active Directory (Azure AD). Al integrar OpsGenie con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a OpsGenie.
* Permitir que los usuarios inicien sesión automáticamente en OpsGenie con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en OpsGenie.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* OpsGenie admite el inicio de sesión único iniciado por **IDP**.
* Una vez configurado OpsGenie, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-opsgenie-from-the-gallery"></a>Adición de OpsGenie desde la galería

Para configurar la integración de OpsGenie en Azure AD, deberá agregar OpsGenie desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **OpsGenie** en el cuadro de búsqueda.
1. Seleccione **OpsGenie** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-opsgenie"></a>Configuración y prueba del inicio de sesión único de Azure AD para OpsGenie

Configure y pruebe el inicio de sesión único de Azure AD con OpsGenie mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de OpsGenie.

Para configurar y probar el inicio de sesión único de Azure AD con OpsGenie, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de OpsGenie](#configure-opsgenie-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de OpsGenie](#create-opsgenie-test-user)** , para tener un homólogo de B.Simon en OpsGenie vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **OpsGenie**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://app.opsginie.com/auth/saml/<UNIQUEID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://app.opsginie.com/auth/saml?id=<UNIQUEID>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la dirección URL de respuesta reales, que se explican más adelante en el tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

1. En la sección **Configurar OpsGenie**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a OpsGenie mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **OpsGenie**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-opsgenie-sso"></a>Configuración del inicio de sesión único de OpsGenie

1. Abra otra instancia del explorador y después inicie sesión en OpsGenie como administrador.

2. Haga clic en **Configuración** y después en la pestaña **Inicio de sesión único**.
   
    ![Inicio de sesión único de OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-06.png)

3. Para habilitar SSO, seleccione **Habilitado**.
   
    ![Configuración de OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-07.png) 

4. En la sección **Proveedor**, haga clic en la pestaña **Azure Active Directory**.
   
    ![Configuración de OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-08.png) 

5. En la página de diálogo de Azure Active Directory, realice los siguientes pasos:
   
    ![Configuración de OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-09.png)
    
    a. Copie el valor de **App ID URI** (URI de id. de aplicación) y péguelo en el cuadro de texto **Identificador (id. de entidad)** de la sección **Configuración básica de SAML** en Azure Portal.

    a. Copie el valor de **Reply URL** (URL de respuesta) y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** en Azure Portal.

    a. En el cuadro de texto **SAML 2.0 Endpoint** (Punto de conexión SAML 2.0), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.
    
    b. En el cuadro de texto **URL de metadatos**, pegue el valor de **Dirección URL de metadatos de federación de la aplicación** que copió en Azure Portal.
    
    c. Para habilitar el inicio de sesión único, active la alternancia **Enable single sign-on** (Habilitar el inicio de sesión único).

    d. Haga clic en **Apply SSO settings** (Aplicar la configuración de SSO).

### <a name="create-opsgenie-test-user"></a>Creación de un usuario de prueba de OpsGenie

El objetivo de esta sección es crear un usuario de prueba llamado B.Simon en OpsGenie. 

1. En una ventana del explorador web, inicie sesión en el inquilino de OpsGenie como administrador.

2. Vaya a la lista Usuarios haciendo clic en **Usuarios** en el panel izquierdo.
   
    ![Configuración de OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-10.png) 

3. Haga clic en **Agregar usuario**.

4. En el cuadro de diálogo **Agregar usuario** , realice los pasos siguientes:
   
    ![Configuración de OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-11.png)
   
    a. En el cuadro de texto **Email** (Correo electrónico), escriba la dirección de correo electrónico de B.Simon en Azure Active Directory.
   
    b. En el cuadro de texto **Full name** (Nombre completo), escriba **B.Simon**.
   
    c. Haga clic en **Save**(Guardar). 

> [!NOTE]
> B.Simon recibirá un correo electrónico con instrucciones para configurar su perfil.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de OpsGenie en el panel de acceso y debería iniciar sesión automáticamente en la versión de OpsGenie para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe OpsGenie con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)