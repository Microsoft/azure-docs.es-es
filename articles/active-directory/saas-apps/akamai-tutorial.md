---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Akamai | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Akamai.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/03/2020
ms.author: jeedes
ms.openlocfilehash: fbec82e25424fd5220aa992cf2dd0e8449e6a0a4
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88523140"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-akamai"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Akamai

En este tutorial, obtendrá información sobre cómo integrar Akamai con Azure Active Directory (Azure AD). Al integrar Akamai con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Akamai.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Akamai con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

La integración de Azure Active Directory y Akamai Enterprise Application Access permite un acceso sin problemas a las aplicaciones heredadas hospedadas en la nube o en el entorno local. La solución integrada aprovecha las ventajas de todas las funcionalidades modernas de Azure Active Directory como el [acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal), [Azure AD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) y [Azure AD Identity Governance](https://docs.microsoft.com/azure/active-directory/governance/identity-governance-overview) para el acceso a las aplicaciones heredadas sin necesidad de instalar agentes ni modificaciones en la aplicación.

En la imagen siguiente se describe cómo encaja Akamai EAA en el escenario de Acceso híbrido seguro más amplio.

![Akamai EAA encaja en el escenario de Acceso híbrido seguro más amplio](./media/header-akamai-tutorial/introduction01.png)

### <a name="key-authentication-scenarios"></a>Escenarios de autenticación de claves

Además de la compatibilidad con la integración nativa de Azure Active Directory con protocolos de autenticación modernos, como Open ID Connect, SAML y WS-Fed, Akamai EAA amplía el acceso seguro para las aplicaciones de autenticación heredadas tanto en el acceso interno como en el externo con Azure AD, lo que habilita escenarios modernos (por ejemplo, el acceso sin contraseña) en estas aplicaciones. Esto incluye:

* Aplicaciones de autenticación basadas en encabezados
* Escritorio remoto
* SSH (Secure Shell)
* Aplicaciones de autenticación Kerberos
* VNC (Virtual Network Computing)
* Aplicaciones sin autenticación integrada o autenticación anónima
* Aplicaciones de autenticación NTLM (protección con dos preguntas para el usuario)
* Aplicaciones basadas en formularios (protección con dos preguntas para el usuario)

### <a name="integration-scenarios"></a>Escenarios de integración

La asociación entre Microsoft y Akamai EAA permite la flexibilidad necesaria para satisfacer los requisitos empresariales, ya que admite varios escenarios de integración en función de las necesidades de las empresas. Se pueden usar para proporcionar cobertura desde el primer día en todas las aplicaciones y clasificar y configurar gradualmente las clasificaciones de directivas adecuadas.

#### <a name="integration-scenario-1"></a>Escenario de integración 1

Akamai EAA se configura como una sola aplicación en Azure AD. El administrador puede configurar la directiva de CA en la aplicación y, una vez que se cumplan las condiciones, los usuarios puedan obtener acceso al portal de Akamai EAA.

**Ventajas**:

* Solo tiene que configurar IDP una vez.

**Inconvenientes**:

* Los usuarios terminan teniendo dos portales de aplicaciones.

* Cobertura de la directiva de CA común única para todas las aplicaciones.

![Escenario de integración 1](./media/header-akamai-tutorial/scenario1.png)

#### <a name="integration-scenario-2"></a>Escenario de integración 2

La aplicación Akamai EAA se configura de forma individual en el portal de Azure AD. El administrador puede configurar una directiva de CA individual en las aplicaciones y, una vez que se cumplan las condiciones, los usuarios se pueden redirigir directamente a la aplicación específica.

**Ventajas**:

* Puede definir directivas de CA individuales.

* Todas las aplicaciones se representan en el panel de myApps.microsoft.com y 0365 Waffle.


**Inconvenientes**:

* Debe configurar varios IDP.

![Escenario de integración 2](./media/header-akamai-tutorial/scenario2.png)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Akamai.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Akamai admite el inicio de sesión único iniciado por IDP.

#### <a name="important"></a>Importante

Toda la configuración que se muestra a continuación es la misma para el **escenario de integración 1** y el **escenario 2**. En el **escenario de integración 2**, ha configurado un IDP individual en Akamai EAA. La propiedad URL debe modificarse para que apunte a la dirección URL de la aplicación.

![Importante](./media/header-akamai-tutorial/important.png)

## <a name="adding-akamai-from-the-gallery"></a>Adición de Akamai desde la galería

Para configurar la integración de Akamai en Azure AD, será preciso agregar Akamai desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Akamai** en el cuadro de búsqueda.
1. Seleccione **Akamai** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-akamai"></a>Configuración y prueba del inicio de sesión único de Azure AD para Akamai

Configure y pruebe el inicio de sesión único de Azure AD con Akamai mediante una usuaria de prueba llamada **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Akamai.

Para configurar y probar el inicio de sesión único de Azure AD con Akamai, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Akamai](#configure-akamai-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Configuración de IDP](#setting-up-idp)**
    * **[Autenticación basada en encabezados](#header-based-authentication)**
    * **[Escritorio remoto](#remote-desktop)**
    * **[SSH](#ssh)**
    * **[Autenticación Kerberos](#kerberos-authentication)**
    * **[Creación de un usuario de prueba de Akamai](#create-akamai-test-user)** , para tener un homólogo de B.Simon en Akamai que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Akamai**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<Yourapp>.login.go.akamai-access.com/saml/sp/response`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https:// <Yourapp>.login.go.akamai-access.com/saml/sp/response`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de cliente de Akamai](https://www.akamai.com/us/en/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up Akamai** (Configurar Akamai), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Akamai mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Akamai**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-akamai-sso"></a>Configuración del inicio de sesión único de Akamai

### <a name="setting-up-idp"></a>Configuración de IDP

**Configuración de IDP de AKAMAI EAA**

1. Inicie sesión en la consola **Enterprise Application Access de Akamai**.
1. En la **consola EAA de Akamai**, seleccione **Identity** (Identidad)  > **Identity Providers** (Proveedores de identidades) y haga clic en **Add Identity Provider** (Agregar proveedor de identidades).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure01.png)

1. En **Create New Identity Provider** (Crear proveedor de identidades), realice los pasos siguientes:

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure02.png)

    a. Especifique el valor de **Unique Name** (Nombre único).

    b. Elija **Third Party SAML** (SAML de terceros) y haga clic en **Create Identity Provider and Configure** (Crear proveedor de identidades y configurar).

### <a name="general-settings"></a>Configuración general

1. **Identity Intercept** (Interceptación de identidad): especifique el nombre de la dirección URL base del SP que se usará para la configuración de Azure AD.

    > [!NOTE]
    > Puede elegir tener su propio dominio personalizado (requerirá una entrada DNS y un certificado). En este ejemplo, vamos a usar el dominio de Akamai.

1. **Akamai Cloud Zone** (Zona de nube de Akamai): seleccione la zona de nube adecuada.
1. **Certificate Validation** (Validación de certificado): compruebe la documentación de Akamai (opcional)

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure03.png)

### <a name="authentication-configuration"></a>Configuración de autenticación

1. URL: especifique la misma dirección URL que la que indicó en la interceptación de identidad (aquí es donde se redirigirá a los usuarios después de la autenticación).
2. Logout URL (Dirección URL de cierre de sesión): actualice la dirección URL de cierre de sesión.
3. Sign SAML Request (Firmar solicitud SAML): el valor está desactivado de manera predeterminada.
4. Para el archivo de metadatos de IDP, agregue la aplicación en la consola de Azure AD.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure04.png)

### <a name="session-settings"></a>Configuración de la sesión

Deje la configuración predeterminada.

![Configuración de Akamai](./media/header-akamai-tutorial/sessionsettings.png)

### <a name="directories"></a>Directorios

Omita la configuración del directorio.

![Configuración de Akamai](./media/header-akamai-tutorial/directories.png)

### <a name="customization-ui"></a>IU de personalización

Puede agregar personalización a IDP.

![Configuración de Akamai](./media/header-akamai-tutorial/customizationui.png)

### <a name="advanced-settings"></a>Configuración avanzada

Omita la configuración avanzada o consulte la documentación de Akamai para obtener más detalles.

![Configuración de Akamai](./media/header-akamai-tutorial/advancesettings.png)

### <a name="deployment"></a>Implementación

1. Haga clic en Deploy Identity Provider (Implementar proveedor de identidades).

    ![Configuración de Akamai](./media/header-akamai-tutorial/deployment.png)

2. Comprobación de que la implementación se ha realizado correctamente

### <a name="header-based-authentication"></a>Autenticación basada en encabezados

Autenticación basada en encabezados de Akamai

1. Elija **Custom HTTP** (HTTP personalizado) en el Asistente para agregar aplicaciones.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure05.png)

2. Escriba **Application Name** (Nombre de aplicación) y **Description** (Descripción).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure06.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure07.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure08.png)

#### <a name="authentication"></a>Authentication

1. Seleccione **Authentication** (Autenticación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure09.png)

2. Asigne el **proveedor de identidades**.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure10.png)

#### <a name="services"></a>Servicios

Haga clic en Save and Go to Authentication (Guardar e ir a Autenticación).

![Configuración de Akamai](./media/header-akamai-tutorial/configure11.png)

#### <a name="advanced-settings"></a>Configuración avanzada

1. En los **Customer HTTP Headers** (Encabezados de HTTP del cliente), especifique **CustomerHeader** y **SAML Attribute** (Atributo de SAML).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure12.png)

1. Haga clic en el botón **Save and go to Deployment** (Guardar e ir a Implementación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure13.png)

#### <a name="deploy-the-application"></a>Implementación de la aplicación

1. Haga clic en el botón **Deploy Application** (Implementar aplicación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure14.png)

1. Compruebe que la aplicación se ha implementado correctamente.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure15.png)

1. Experiencia del usuario final.

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser01.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser02.png)

1. Acceso condicional.

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess01.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess02.png)

#### <a name="remote-desktop"></a>Escritorio remoto

1. Elija **RDP** en el Asistente para agregar aplicaciones.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure16.png)

1. Escriba **Application Name** (Nombre de aplicación) y **Description** (Descripción).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure17.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure18.png)

1. Especifique el conector que funcionará con esta aplicación.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure19.png)

#### <a name="authentication"></a>Authentication

Haga clic en **Save and go to Services** (Guardar e ir a Servicios).

![Configuración de Akamai](./media/header-akamai-tutorial/configure20.png)

#### <a name="services"></a>Servicios

Haga clic en **Save and go to Advanced Settings** (Guardar e ir a Configuración avanzada).

![Configuración de Akamai](./media/header-akamai-tutorial/configure21.png)

#### <a name="advanced-settings"></a>Configuración avanzada

1. Haga clic en **Save and go to Deployment** (Guardar e ir a Implementación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure22.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure23.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure24.png)

1. Experiencia del usuario final

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser03.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser02.png)

1. Acceso condicional

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess04.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess05.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess06.png)

1. También puede escribir directamente la dirección URL de la aplicación RDP.

#### <a name="ssh"></a>SSH

1. Vaya a Add Applications (Agregar aplicaciones) y elija **SSH**.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure25.png)

1. Escriba **Application Name** (Nombre de aplicación) y **Description** (Descripción).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure26.png)

1. Configure Application Identity (Identidad de aplicación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure27.png)

    a. Especifique un nombre y una descripción.

    b. Especifique la dirección IP y el nombre de dominio completo del servidor de aplicaciones y el puerto para SSH.

    c. Especifique el nombre de usuario y la frase de contraseña de SSH *Consulte la consola de acceso de aplicaciones empresariales de Akamai.

    d. Especifique el nombre de host externo.

    e. Especifique la ubicación del conector y elija el conector.

#### <a name="authentication"></a>Authentication

Haga clic en **Save and go to Services** (Guardar e ir a Servicios).

![Configuración de Akamai](./media/header-akamai-tutorial/configure28.png)

#### <a name="services"></a>Servicios

Haga clic en **Save and go to Advanced Settings** (Guardar e ir a Configuración avanzada).

![Configuración de Akamai](./media/header-akamai-tutorial/configure29.png)

#### <a name="advanced-settings"></a>Configuración avanzada

Haga clic en Save and go to Deployment (Guardar e ir a Implementación).

![Configuración de Akamai](./media/header-akamai-tutorial/configure30.png)

![Configuración de Akamai](./media/header-akamai-tutorial/configure31.png)

#### <a name="deployment"></a>Implementación

1. Haga clic en **Deploy Application** (Implementar aplicación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure32.png)

1. Experiencia del usuario final

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser03.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser04.png)

1. Acceso condicional

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess04.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess07.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess08.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess09.png)

### <a name="kerberos-authentication"></a>Autenticación Kerberos

En el ejemplo siguiente, publicaremos un servidor web interno <code>http://frp-app1.superdemo.live</code> y habilitaremos el inicio de sesión único con KCD.

#### <a name="general-tab"></a>Pestaña General

![Configuración de Akamai](./media/header-akamai-tutorial/generaltab.png)

#### <a name="authentication-tab"></a>Pestaña Autenticación

Asignación del proveedor de identidades

![Configuración de Akamai](./media/header-akamai-tutorial/authenticationtab.png)

#### <a name="services-tab"></a>Pestaña Servicios

![Configuración de Akamai](./media/header-akamai-tutorial/servicestab.png)

#### <a name="advanced-settings"></a>Configuración avanzada

![Configuración de Akamai](./media/header-akamai-tutorial/advancesettings02.png)

> [!NOTE]
> El SPN del servidor web tiene el formato SPN@Domain como, por ejemplo: `HTTP/frp-app1.superdemo.live@SUPERDEMO.LIVE` para esta demostración. Deje los valores predeterminados en el resto de la configuración.

#### <a name="deployment-tab"></a>Pestaña Implementación

![Configuración de Akamai](./media/header-akamai-tutorial/deploymenttab.png)

#### <a name="adding-directory"></a>Incorporación de directorio

1. Seleccione **AD** en la lista desplegable.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure33.png)

1. Proporcione los datos necesarios.

    ![Configuración de Akamai](./media/header-akamai-tutorial/configure34.png)

1. Compruebe la creación del directorio.

    ![Configuración de Akamai](./media/header-akamai-tutorial/directorydomain.png)

1. Agregue los grupos o unidades organizativas que requerirán acceso.

    ![Configuración de Akamai](./media/header-akamai-tutorial/addgroup.png)

1. En la parte inferior, el grupo se denomina EAAGroup y tiene 1 miembro.

    ![Configuración de Akamai](./media/header-akamai-tutorial/eaagroup.png)

1. Agregue el directorio al proveedor de identidades; para ello, haga clic en **Identity** (Identidad)  > **Identity Providers** (Proveedores de identidades), haga clic en la pestaña **Directories** (Directorios) y haga clic en **Assign directory** (Assign directory).

    ![Configuración de Akamai](./media/header-akamai-tutorial/assigndirectory.png)

### <a name="configure-kcd-delegation-for-eaa-walkthrough"></a>Tutorial de configuración de la delegación de KCD para EAA

#### <a name="step-1-create-an-account"></a>Paso 1: Creación de una cuenta 

1. En el ejemplo, usaremos una cuenta llamada **EAADelegation**. Para ello, puede usar el complemento **Usuarios y equipos de Active Directory**.

    ![Configuración de Akamai](./media/header-akamai-tutorial/assigndirectory.png)

    > [!NOTE]
    > El nombre de usuario tiene que estar en un formato específico basado en el **nombre de interceptación de identidad**. En la figura 1, vemos que es **corpapps.login.go.akamai-access.com**

1. El nombre de inicio de sesión de usuario será `HTTP/corpapps.login.go.akamai-access.com`

    ![Configuración de Akamai](./media/header-akamai-tutorial/eaadelegation.png)

#### <a name="step-2-configure-the-spn-for-this-account"></a>Paso 2: Configuración del SPN para esta cuenta

1. Según este ejemplo, el SPN será como el que se indica a continuación.

2. setspn -s **Http/corpapps.login.go.akamai-access.com eaadelegation**

    ![Configuración de Akamai](./media/header-akamai-tutorial/spn.png)

#### <a name="step-3-configure-delegation"></a>Paso 3: Configuración de la delegación

1. En la cuenta EAADelegation, haga clic en la pestaña Delegation (Delegación).

    ![Configuración de Akamai](./media/header-akamai-tutorial/spn.png)

    * Especificación de uso de cualquier protocolo de autenticación
    * Haga clic en Agregar y agregue la cuenta del grupo de aplicaciones para el sitio web de Kerberos. Se debe resolver automáticamente para corregir el SPN, si se ha configurado correctamente.

#### <a name="step-4-create-a-keytab-file-for-akamai-eaa"></a>Paso 4: Creación de un archivo keytab para AKAMAI EAA

1. Esta es la sintaxis genérica.

1. ktpass /out ActiveDirectorydomain.keytab  /princ `HTTP/yourloginportalurl@ADDomain.com`  /mapuser serviceaccount@ADdomain.com /pass +rdnPass  /crypto All /ptype KRB5_NT_PRINCIPAL

1. Explicación del ejemplo

    | Fragmento de código | Explicación |
    | - | - |
    | Ktpass /out EAADemo.keytab | // Nombre del archivo keytab de salida |
    | /princ HTTP/corpapps.login.go.akamai-access.com@superdemo.live | // HTTP/yourIDPName@YourdomainName |
    | /mapuser eaadelegation@superdemo.live | // Cuenta de delegación de EAA |
    | /pass RANDOMPASS | // Contraseña de la cuenta de delegación de EAA |
    | /crypto All ptype KRB5_NT_PRINCIPAL | // Consulte la documentación de Akamai EAA |
    | | |

1. Ktpass /out EAADemo.keytab  /princ HTTP/corpapps.login.go.akamai-access.com@superdemo.live /mapuser eaadelegation@superdemo.live /pass RANDOMPASS /crypto All ptype KRB5_NT_PRINCIPAL

    ![Configuración de Akamai](./media/header-akamai-tutorial/administrator.png)

#### <a name="step-5-import-keytab-in-the-akamai-eaa-console"></a>Paso 5: Importación de keytab en la consola de AKAMAI EAA

1. Haga clic en **System** (Sistema)  > **Keytabs**.

    ![Configuración de Akamai](./media/header-akamai-tutorial/keytabs.png)

1. En el tipo de keytab, elija **Delegación Kerberos**.

    ![Configuración de Akamai](./media/header-akamai-tutorial/keytabdelegation.png)

1. Asegúrese de que el archivo keytab se muestre como Deployed (implementado) y Verified (Comprobado).

    ![Configuración de Akamai](./media/header-akamai-tutorial/keytabs02.png)

1. Experiencia del usuario

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser03.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/enduser04.png)

1. Acceso condicional

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess04.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess10.png)

    ![Configuración de Akamai](./media/header-akamai-tutorial/conditionalaccess11.png)

### <a name="create-akamai-test-user"></a>Creación de un usuario de prueba de Akamai

En esta sección, creará un usuario llamado B.Simon en Akamai. Colabore con el [equipo de soporte técnico al cliente de Akamai](https://www.akamai.com/us/en/contact-us/) para agregar los usuarios a la plataforma de Akamai. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único. 

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Akamai en el panel de acceso, iniciará sesión automáticamente en la versión de Akamai para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Akamai con Azure AD](https://aad.portal.azure.com/)
