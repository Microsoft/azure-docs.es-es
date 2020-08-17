---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con JIRA SAML SSO by Microsoft | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y JIRA SAML SSO by Microsoft.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 4b663047-7f88-443b-97bd-54224b232815
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/11/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9f06f0f376c1ecafcff833aa4b6bece0a0e35d10
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475403"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-jira-saml-sso-by-microsoft"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con JIRA SAML SSO by Microsoft

En este tutorial aprenderá a integrar JIRA SAML SSO by Microsoft con Azure Active Directory (Azure AD). Al integrar JIRA SAML SSO by Microsoft con Azure AD, se puede realizar lo siguiente:

* Controlar en Azure AD quién tiene acceso a JIRA SAML SSO by Microsoft.
* Permitir que los usuarios inicien sesión automáticamente en JIRA SAML SSO by Microsoft con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="description"></a>Descripción

Use la cuenta de Microsoft Azure Active Directory con el servidor JIRA de Atlassian para habilitar el inicio de sesión único. De esta forma, todos los usuarios de la organización pueden usar las credenciales de Azure AD para iniciar sesión en la aplicación JIRA. Este complemento usa SAML 2.0 para la federación.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con JIRA SAML SSO by Microsoft, se necesitan los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Se deben instalar y configurar las versiones de 6.4 a 8.11.0 de JIRA Core y Software o las versiones de 3.0 a 4.8.0 de JIRA Service Desk en una versión de Windows de 64 bits.
- El servidor JIRA es compatible con HTTPS
- Tenga en cuenta que las versiones admitidas para el complemento JIRA se mencionan en la sección siguiente.
- El servidor JIRA es accesible en Internet, especialmente a la página de inicio de sesión de Azure AD para la autenticación y debe poder recibir el token de Azure AD
- Las credenciales de administrador se configuran en JIRA
- WebSudo está deshabilitado en JIRA
- Usuario de prueba creado en la aplicación de servidor JIRA

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción de JIRA. Pruebe primero la integración en el entorno de desarrollo o de ensayo de la aplicación y, después, use el entorno de producción.

Para empezar, necesita los siguientes elementos:

* No use el entorno de producción, salvo que sea necesario.
* Una suscripción habilitada para el inicio de sesión único(SSO) en JIRA SAML SSO by Microsoft.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="supported-versions-of-jira"></a>Versiones compatibles de JIRA

* JIRA Core y Software: 6.4 a 8.11.0
* JIRA Service Desk: de la versión 3.0.0 a la 4.8.0
* JIRA también admite 5.2. Para obtener más detalles, haga clic en [Microsoft Azure Active Directory single sign-on for JIRA 5.2](jira52microsoft-tutorial.md) (Inicio de sesión único de Microsoft Azure Active Directory para JIRA 5.2).

> [!NOTE]
> Tenga en cuenta que nuestro complemento de JIRA también funciona con Ubuntu 16.04 y Linux.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* JIRA SAML SSO by Microsoft admite el inicio de sesión único iniciado por **SP**

## <a name="adding-jira-saml-sso-by-microsoft-from-the-gallery"></a>Agregar JIRA SAML SSO by Microsoft desde la galería

Para configurar la integración de JIRA SAML SSO by Microsoft en Azure AD, deberá agregar JIRA SAML SSO by Microsoft desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **JIRA SAML SSO by Microsoft** en el cuadro de búsqueda.
1. Seleccione **JIRA SAML SSO by Microsoft** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-jira-saml-sso-by-microsoft"></a>Configuración y prueba del inicio de sesión único de Azure AD para JIRA SAML SSO by Microsoft

Configure y pruebe el inicio de sesión único de Azure AD con JIRA SAML SSO by Microsoft mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de JIRA SAML SSO by Microsoft.

Para configurar y probar el inicio de sesión único de Azure AD con JIRA SAML SSO by Microsoft, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de JIRA SAML SSO by Microsoft](#configure-jira-saml-sso-by-microsoft-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de JIRA SAML SSO by Microsoft](#create-jira-saml-sso-by-microsoft-test-user)** , para tener un homólogo de B.Simon en JIRA SAML SSO by Microsoft que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **JIRA SAML SSO by Microsoft**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<domain:port>/plugins/servlet/saml/auth`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<domain:port>/`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<domain:port>/plugins/servlet/saml/auth`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. En caso de que sea una dirección URL con nombre, el puerto es opcional. Estos valores se reciben durante la configuración del complemento de Jira, que se explica más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección, va a permitir que B.Simon use el inicio de sesión único de Azure concediéndole acceso a JIRA SAML SSO by Microsoft.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **JIRA SAML SSO by Microsoft**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-jira-saml-sso-by-microsoft-sso"></a>Configuración del inicio de sesión único de JIRA SAML SSO by Microsoft

1. En otra ventana del explorador web, inicie sesión en la instancia de JIRA como administrador.

2. Mantenga el mouse encima del icono de engranaje y haga clic en **Complementos**.

    ![Configurar inicio de sesión único](./media/jiramicrosoft-tutorial/addon1.png)

3. Descargue el complemento del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=56506). Cargue manualmente el complemento proporcionado por Microsoft mediante el menú **Cargar complemento**. La descarga del complemento está contemplada en el [Acuerdo de servicio de Microsoft](https://www.microsoft.com/servicesagreement/).

    ![Configurar inicio de sesión único](./media/jiramicrosoft-tutorial/addon12.png)

4. Para ejecutar el escenario de proxy inverso de JIRA o el escenario del equilibrador de carga, realice los pasos siguientes:

    > [!NOTE]
    > En primer lugar, debe configurar el servidor con las siguientes instrucciones y después instalar el complemento.

    a. Agregue el siguiente atributo en el puerto del **conector** del archivo **server.xml** de la aplicación de servidor JIRA.

    `scheme="https" proxyName="<subdomain.domain.com>" proxyPort="<proxy_port>" secure="true"`

    ![Configurar inicio de sesión único](./media/jiramicrosoft-tutorial/reverseproxy1.png)

    b. Cambie la **URL base** en **Configuración del sistema** en función del proxy o equilibrador de carga.

    ![Configurar inicio de sesión único](./media/jiramicrosoft-tutorial/reverseproxy2.png)

5. Una vez instalado el complemento, aparece en la sección de complementos **Instalados por el usuario** de **Administrar complemento**. Haga clic en **Configurar** para configurar el nuevo complemento.

    ![Configurar inicio de sesión único](./media/jiramicrosoft-tutorial/addon14.png)

6. Siga estos pasos en la página de configuración:

    ![Configurar inicio de sesión único](./media/jiramicrosoft-tutorial/addon54.png)

    > [!TIP]
    > Asegúrese de que hay un solo certificado asignado a la aplicación, de forma que no se produzca ningún error en la resolución de los metadatos. Si hay varios certificados, después de resolver los metadatos, el administrador recibe un error.

    1. En el cuadro de texto **URL de metadatos**, pegue la **dirección URL de metadatos de federación de aplicación** que copió en Azure Portal y haga clic en el botón **Resolver**. Se lee la dirección URL de metadatos de IdP y se rellena toda la información de campos.

    1. Copie los valores **Identificador, Dirección URL de respuesta y Dirección URL de inicio de sesión**, y péguelos en los cuadros de texto **Identificador, Dirección URL de respuesta y Dirección URL de inicio de sesión** respectivamente en la sección **Dominio y direcciones URL de JIRA SAML SSO by Microsoft** de Azure Portal.

    1. En **Nombre del botón de inicio de sesión** escriba el nombre del botón que la organización quiere que los usuarios vean en la pantalla de inicio de sesión.
    
    1. En **Descripción del botón de inicio de sesión** escriba el nombre del botón que la organización quiere que los usuarios vean en la pantalla de inicio de sesión.

    1. En **SAML User ID Locations** (Ubicaciones de Id. de usuario de SAML) seleccione **User ID is in the NameIdentifier element of the Subject statement** (El Id. de usuario está en el elemento NameIdentifier de la instrucción Subject) o **User ID is in an Attribute element** (El Id. de usuario está en un elemento Attribute).  Este identificador debe ser el identificador de usuario de JIRA. Si el identificador de usuario no coincide, el sistema no permitirá que los usuarios inicien sesión.

       > [!Note]
       > La ubicación del Id. de usuario de SAML predeterminada es el identificador de nombre. Puede cambiarlo a una opción de atributo y escribir el nombre de atributo adecuado.

    1. Si selecciona la opción **User ID is in an Attribute element** (El Id. de usuario está en un elemento Attribute), escriba el nombre del atributo cuando se espera el id. de usuario en el cuadro de texto **Nombre del atributo**.

    1. Si se usa el dominio federado (por ejemplo, ADFS, etc.) con Azure AD, haga clic en la opción **Habilitar detección de dominio principal** y configure el **nombre de dominio**.

    1. En **Nombre de dominio**, escriba el nombre del dominio en el caso de inicios de sesión basados en ADFS.

    1. Active **Enable Single Sign out** (Habilitar cierre de sesión único) si quiere que se cierre la sesión de Azure AD cuando un usuario cierre la sesión de JIRA.
    
    1. Habilite la casilla **Force Azure Login** (Forzar inicio de sesión en Azure) solo si desea conectarse con las credenciales de Azure AD.
    
       > [!Note]
       > Para habilitar el formulario de inicio de sesión predeterminado para el inicio de sesión de administrador en la página de inicio de sesión esté habilitada la opción de forzar inicio de sesión en Azure, agregue el parámetro de consulta a la dirección URL del explorador.
       > `https://<domain:port>/login.jsp?force_azure_login=false`

    1. Haga clic en el botón **Save** (Guardar) para guardar la configuración.

       > [!NOTE]
       > Para más información acerca de la instalación y la solución de problemas, visite [MS JIRA SSO Connector Admin Guide](../ms-confluence-jira-plugin-adminguide.md) (Guía de administración del conector SSO de MS JIRA). También hay una sección de [preguntas frecuentes](../ms-confluence-jira-plugin-faq.md) que puede servirle de ayuda.

### <a name="create-jira-saml-sso-by-microsoft-test-user"></a>Creación de un usuario de prueba en JIRA SAML SSO by Microsoft

Para permitir que los usuarios de Azure AD inicien sesión en el servidor local de JIRA, se deben aprovisionar en JIRA SAML SSO by Microsoft. Para JIRA SAML SSO by Microsoft, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el servidor local de JIRA.

2. Mantenga el mouse encima del icono de engranaje y haga clic en **Administración de usuarios**.

    ![Agregar empleado](./media/jiramicrosoft-tutorial/user1.png)

3. Se le redirigirá a la página de acceso de administrador para especificar la **contraseña** y haga clic en el botón **Confirmar**.

    ![Agregar empleado](./media/jiramicrosoft-tutorial/user2.png)

4. En la sección de la pestaña **Administración de usuarios**, haga clic en **Crear usuario**.

    ![Agregar empleado](./media/jiramicrosoft-tutorial/user3.png) 

5. En la página del cuadro de diálogo **"Create New User"** (Crear nuevo usuario), realice los pasos siguientes:

    ![Agregar empleado](./media/jiramicrosoft-tutorial/user4.png) 

    a. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, B.simon@contoso.com.

    b. En el cuadro de texto **Nombre completo**, escriba el nombre completo de un usuario, por ejemplo, B.Simon.

    c. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, B.simon@contoso.com.

    d. En el cuadro de texto **Contraseña**, escriba la contraseña del usuario.

    e. Haga clic en **Crear usuario**.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de JIRA SAML SSO by Microsoft del panel de acceso, debería iniciar sesión automáticamente en la aplicación JIRA SAML SSO by Microsoft para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe JIRA SAML SSO by Microsoft con Azure AD](https://aad.portal.azure.com/)
