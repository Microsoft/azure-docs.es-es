---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ADP | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ADP.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/26/2019
ms.author: jeedes
ms.openlocfilehash: c170581496241ab3be0fe54bb484549ab4b1f976
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88542911"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-adp"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ADP

En este tutorial, aprenderá a integrar ADP en Azure Active Directory (Azure AD). Al integrar ADP en Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a ADP.
* Permitir que los usuarios puedan iniciar sesión automáticamente en ADP con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en ADP.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ADP admite inicio de sesión único iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-adp-from-the-gallery"></a>Adición de ADP desde la galería

Para configurar la integración de ADP en Azure AD, será preciso agregar ADP desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ADP** en el cuadro de búsqueda.
1. Seleccione **ADP** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-adp"></a>Configuración y prueba del inicio de sesión único de Azure AD para ADP

Configure y pruebe el inicio de sesión único de Azure AD con ADP mediante una usuaria de prueba llamada **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de ADP.

Para configurar y probar el inicio de sesión único de Azure AD con ADP, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único en ADP](#configure-adp-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de ADP](#create-adp-test-user)** , para tener un homólogo de B.Simon en ADP que esté vinculado a la representación de ella en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, vaya a la página de integración de la aplicación **ADP**, haga clic en la **pestaña Propiedades** y realice los pasos siguientes: 

    ![Propiedades del inicio de sesión único](./media/adpfederatedsso-tutorial/tutorial_adp_prop.png)

    a. Establezca el valor del campo **Habilitado para que los usuarios inicien sesión** en **Sí**.

    b. Copie la **dirección URL de acceso de usuario** y péguela en la **sección Configurar dirección URL de inicio de sesión** (este paso se explica más adelante en el tutorial).

    c. Establezca el valor del campo **Asignación de usuarios necesaria** en **Sí**.

    d. Establezca el valor del campo **Visible para los usuarios** en **No**.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **ADP**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL: `https://fed.adp.com`

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar ADP**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a ADP mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ADP**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-adp-sso"></a>Configuración del inicio de sesión único de ADP

Para configurar el inicio de sesión único en **ADP**, debe cargar el archivo **XML de metadatos** que descargó en el [sitio web de ADP](https://adpfedsso.adp.com/public/login/index.fcc).

> [!NOTE]  
> Este proceso puede tardar unos días.

### <a name="configure-your-adp-services-for-federated-access"></a>Configurar los servicios de ADP para obtener acceso federado

>[!Important]
> Los empleados que necesiten obtener acceso federado a los servicios de ADP deben asignarse a la aplicación de servicio de ADP y, posteriormente, deben volver a asignarse al servicio ADP específico.
Una vez recibida la confirmación del representante de ADP, configure los servicios de ADP y asigne o administre los usuarios para controlar su acceso al servicio de ADP especificado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ADP** en el cuadro de búsqueda.
1. Seleccione **ADP** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.
1. En Azure Portal, vaya a la página de integración de la aplicación **ADP**, haga clic en la **pestaña Propiedades** y realice los pasos siguientes:  

    ![Propiedades vinculadas de inicio de sesión único](./media/adpfederatedsso-tutorial/tutorial_adp_linkedproperties.png)

    a.  Establezca el valor del campo **Habilitado para que los usuarios inicien sesión** en **Sí**.

    b.  Establezca el valor del campo **Asignación de usuarios necesaria** en **Sí**.

    c.  Establezca el valor del campo **Visible para los usuarios** en **Sí**.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **ADP**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.

1. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, elija como **Modo** la opción **Vinculado**. Para vincular la aplicación a **ADP**.

    ![Inicio de sesión único vinculado](./media/adpfederatedsso-tutorial/tutorial_adp_linked.png)

1. Navegue hasta la sección **Configurar la dirección URL de inicio de sesión** y realice los pasos siguientes:

    ![Propiedades del inicio de sesión único](./media/adpfederatedsso-tutorial/tutorial_adp_linkedsignon.png)

    a. Pegue la **dirección URL de acceso de usuario** que copió de la **pestaña Propiedades** anterior (de la aplicación principal ADP).
                                                             
    b. Estas son las 5 aplicaciones que admiten diferentes **direcciones URL del estado de la retransmisión**. Tiene que anexar de forma manual el valor adecuado de la **dirección URL del estado de la retransmisión** de una aplicación en particular a la **dirección URL de acceso de usuarios**.
    
    * **ADP Workforce Now**
        
        `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?WFN`

    * **ADP Workforce Now Enhanced Time**
        
        `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?EETDC2`
    
    * **ADP Vantage HCM**
        
        `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?ADPVANTAGE`

    * **ADP Enterprise HR**

        `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?PORTAL`

    * **MyADP**

        `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?REDBOX`

9. **Guarde** los cambios.

10. Tras recibir la confirmación del representante de ADP, comience la prueba con uno o dos usuarios.

    a. Asigne unos cuantos usuarios a la aplicación del servicio de ADP para que prueben el acceso federado.

    b. La prueba es correcta cuando los usuarios acceden a la aplicación del servicio de ADP en la galería y pueden obtener acceso a su servicio ADP.
 
11. Cuando se confirme una prueba correcta, asigne el servicio de ADP federado a usuarios individuales o a grupos de usuarios (esto se explica más adelante en el tutorial) e implántelo entre sus empleados.

### <a name="create-adp-test-user"></a>Creación de un usuario de prueba de ADP

El objetivo de esta sección es crear una usuaria de prueba llamada B.Simon en ADP. Colabore con el [equipo de soporte técnico de ADP](https://www.adp.com/contact-us/overview.aspx) para agregar los usuarios a la cuenta de ADP. 

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de ADP en el Panel de acceso, debería iniciar sesión automáticamente en la solución ADP para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe ADP con Azure AD](https://aad.portal.azure.com)
