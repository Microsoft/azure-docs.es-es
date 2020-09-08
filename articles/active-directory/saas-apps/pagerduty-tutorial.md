---
title: 'Tutorial: Integración de Azure Active Directory con PagerDuty | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PagerDuty.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: 2bc486987dd25d899728af4861c0b71ef14d9f4a
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554098"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pagerduty"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con PagerDuty

En este tutorial, aprenderá a integrar PagerDuty con Azure Active Directory (Azure AD). Al integrar PagerDuty con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a PagerDuty.
* Permitir que los usuarios puedan iniciar sesión automáticamente en PagerDuty con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en PagerDuty.

> [!NOTE]
> Si usa MFA o la autenticación sin contraseña con Azure AD, desactive el valor de AuthnContext en la solicitud SAML. De lo contrario, Azure AD generará un error al no coincidir el valor de AuthnContext y no volverá a enviar el token a la aplicación.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* PagerDuty admite el inicio de sesión único iniciado por **SP**
* Después de configurar PagerDuty, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-pagerduty-from-the-gallery"></a>Agregación de PagerDuty desde la galería

Para configurar la integración de PagerDuty en Azure AD, deberá agregar PagerDuty desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **PagerDuty** en el cuadro de búsqueda.
1. Seleccione **PagerDuty** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-pagerduty"></a>Configuración y prueba del inicio de sesión único de Azure AD para PagerDuty

Configure y pruebe el inicio de sesión único de Azure AD con PagerDuty mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de PagerDuty.

Para configurar y probar el inicio de sesión único de Azure AD con PagerDuty, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en PagerDuty](#configure-pagerduty-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de PagerDuty](#create-pagerduty-test-user)** : para tener un homólogo de B.Simon en PagerDuty que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **PagerDuty**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenant-name>.pagerduty.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<tenant-name>.pagerduty.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<tenant-name>.pagerduty.com`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales. Póngase en contacto con el [equipo de soporte de cliente de PagerDuty](https://www.pagerduty.com/support/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configuración de PagerDuty**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a PagerDuty mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **PagerDuty**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-pagerduty-sso"></a>Configuración del inicio de sesión único de PagerDuty

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa de Pagerduty como administrador.

2. En el menú de la parte superior, haga clic en **Configuración de cuenta**.

    ![Account Settings (Configuración de cuenta)](./media/pagerduty-tutorial/ic778535.png "Configuración de cuenta")

3. Haga clic en **Inicio de sesión único**.

    ![Inicio de sesión único](./media/pagerduty-tutorial/ic778536.png "Inicio de sesión único")

4. En la página **Habilitar inicio de sesión único (SSO)** , siga estos pasos:

    ![Habilitar el inicio de sesión único](./media/pagerduty-tutorial/ic778537.png "Habilitar el inicio de sesión único")

    a. Abra el certificado codificado en Base 64 que descargó de Azure Portal en el Bloc de notas, copie el contenido en el Portapapeles y, luego, péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509)
  
    b. En el cuadro de texto **URL de inicio de sesión**, pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.
  
    c. En el cuadro de texto **URL de cierre de sesión**, pegue la **dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    d. Seleccione **Allow username/password login** (Permitir inicio de sesión con nombre de usuario/contraseña).

    e. Seleccione la casilla de verificación **Require EXACT authentication context comparison** (Requerir comparación de contexto de autenticación EXACT).

    f. Haga clic en **Guardar cambios**.

### <a name="create-pagerduty-test-user"></a>Creación de un usuario de prueba de PagerDuty

Para permitir que los usuarios de Azure AD inicien sesión en PagerDuty, se deben aprovisionar en PagerDuty. En el caso de PagerDuty, el aprovisionamiento es una tarea manual.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de PagerDuty ofrecida por PagerDuty para aprovisionar cuentas de usuario de Azure Active Directory.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **Pagerduty**.

2. En el menú de la parte superior, haga clic en **Usuarios**.

3. Haga clic en **Agregar usuarios**.
   
    ![Incorporación de usuarios](./media/pagerduty-tutorial/ic778539.png "Agregar usuarios")

4.  En el cuadro de diálogo **Invite your team** (Invitar a su equipo), realice los pasos siguientes:
   
    ![Invitar a su equipo](./media/pagerduty-tutorial/ic778540.png "Invitar a su equipo")

    a. En el cuadro de texto **First and Last Name** (Nombre y apellidos), escriba el nombre de un usuario, por ejemplo, **B.Simon**. 
   
    b. Escriba la dirección de **correo electrónico** del usuario, por ejemplo **b.simon\@contoso.com**.
   
    c. Haga clic en **Add** (Agregar) y después en **Send Invites** (Enviar invitaciones).
   
    > [!NOTE]
    > Todos los usuarios agregados recibirán una invitación para crear una cuenta de PagerDuty.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de PagerDuty en el panel de acceso y debería iniciar sesión automáticamente en la versión de PagerDuty para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de PagerDuty con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de PagerDuty con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

