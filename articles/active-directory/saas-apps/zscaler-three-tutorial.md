---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Zscaler | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler Three.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/17/2019
ms.author: jeedes
ms.openlocfilehash: 00585105350e9858bbfd649a41273def162febcb
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545784"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-zscaler-three"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Zscaler Three

En este tutorial, aprenderá a integrar Zscaler Three con Azure Active Directory (Azure AD). Al integrar Zscaler Three con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Zscaler Three.
* Permitir que los usuarios inicien sesión automáticamente en Zscaler Three con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Zscaler Three.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zscaler Three admite inicio de sesión único iniciado por **SP**

* Zscaler Three admite aprovisionamiento de usuarios **Just-In-Time**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-zscaler-three-from-the-gallery"></a>Incorporación de Zscaler Three desde la galería

Para configurar la integración de Zscaler Three en Azure AD, deberá agregar Zscaler Three desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Zscaler Three** en el cuadro de búsqueda.
1. Seleccione **Zscaler Three** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-zscaler-three"></a>Configuración y prueba del inicio de sesión único de Azure AD para Zscaler Three

Configure y pruebe el inicio de sesión único de Azure AD con Zscaler Three mediante una usuaria de prueba llamada **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Zscaler Three.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler Three, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Zscaler Three](#configure-zscaler-three-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba en Zscaler Three](#create-zscaler-three-test-user)** , para tener un homólogo de B. Simon en Zscaler Three vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Zscaler Three**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://login.zscalerthree.net/sfc_sso`

1. La aplicación Zscaler Three espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/edit-attribute.png)

6. Además de lo anterior, la aplicación Zscaler Three espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen |
    | ---------| ------------ |
    | memberOf     | user.assignedroles |

    > [!NOTE]
    > Haga clic [aquí](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-app-role-management) para saber cómo configurar el valor Role en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Zscaler Three**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B. Simon acceda a Zscaler Three mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Zscaler Three**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione el usuario, como **Britta Simon**, en la lista y luego haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

    ![imagen](./media/zscaler-three-tutorial/tutorial_zscalerthree_users.png)

1. En el cuadro de diálogo **Seleccionar rol**, elija el rol de usuario adecuado de la lista y, luego, haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.

    ![imagen](./media/zscaler-three-tutorial/tutorial_zscalerthree_roles.png)

1. En el cuadro de diálogo **Agregar asignación**, seleccione el botón **Asignar**.

    ![imagen](./media/zscaler-three-tutorial/tutorial_zscalerthree_assign.png)

## <a name="configure-zscaler-three-sso"></a>Configuración del inicio de sesión único de Zscaler Three

1. Para automatizar la configuración en Zscaler Three, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión** .

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Zscaler Three** para ir a la aplicación Zscaler Three. Desde allí, proporcione las credenciales del administrador para iniciar sesión en Zscaler Three. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configurar](common/setup-sso.png)

3. Si quiere configurar Zscaler Three manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Zscaler Three como administrador y realice los pasos siguientes:

4. Vaya a **Administración > Autenticación > Configuración de autenticación** y realice los siguientes pasos:
   
    ![Administración](./media/zscaler-three-tutorial/ic800206.png "Administración")

    a. En Tipo de autenticación, elija **SAML**.

    b. Haga clic en **Configurar SAML**.

5. En la ventana **Editar SAML**, realice los pasos siguientes y haga clic en Guardar.  
            
    ![Administración de usuarios y autenticación](./media/zscaler-three-tutorial/ic800208.png "Manage Users & Authentication")
    
    a. En el cuadro de texto **Dirección URL del portal de SAML**, pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Atributo de nombre de inicio de sesión**, escriba **NameID**.

    c. Haga clic en **Cargar** para cargar el certificado de firma de SAML de Azure que ha descargado desde Azure Portal en el **Certificado SSL público**.

    d. Alterne **Habilitar aprovisionamiento automático de SAML**.

    e. En el cuadro de texto **Atributo de nombre para mostrar del usuario**, escriba **displayName** si desea habilitar el aprovisionamiento automático de SAML para atributos displayName.

    f. En el cuadro de texto **Atributo de nombre del grupo**, escriba **memberOf** si desea habilitar el aprovisionamiento automático de SAML para atributos memberOf.

    g. En el cuadro de texto **Atributo de nombre del departamento**, escriba **department** si desea habilitar el aprovisionamiento automático de SAML para atributos department.

    h. Haga clic en **Save**(Guardar).

6. En la página del cuadro de diálogo **Configurar autenticación de usuario** , realice los pasos siguientes:

    ![Administración](./media/zscaler-three-tutorial/ic800207.png)

    a. Mantenga el puntero sobre el menú **Activación** situado cerca de la parte inferior izquierda.

    b. Haga clic en **Activar**.

## <a name="configuring-proxy-settings"></a>Configuración de los valores de proxy
### <a name="to-configure-the-proxy-settings-in-internet-explorer"></a>Para definir la configuración de proxy en Internet Explorer

1. Inicie **Internet Explorer**.

2. Seleccione **Opciones de Internet** en el menú **Herramientas** para abrir el diálogo **Opciones de Internet**.   
    
     ![Opciones de Internet](./media/zscaler-three-tutorial/ic769492.png "Opciones de Internet")

3. Haga clic en la pestaña **Conexiones** .   
  
     ![Conexiones](./media/zscaler-three-tutorial/ic769493.png "Conexiones")

4. Haga clic en **Configuración de LAN** para abrir el diálogo **Configuración de LAN**.

5. En la sección del servidor proxy, lleve a cabo estos pasos:   
   
    ![Servidor proxy](./media/zscaler-three-tutorial/ic769494.png "Servidor proxy")

    a. Seleccione **Usar un servidor proxy para la LAN**.

    b. En el cuadro de texto Dirección, escriba **gateway.ZscalerThree.net**.

    c. En el cuadro de texto Puerto, escriba **80**.

    d. Seleccione **No usar servidor proxy para direcciones locales**.

    e. Haga clic en **Aceptar** para cerrar el diálogo **Configuración de red de área local (LAN)** .

6. Haga clic en **Aceptar** para cerrar el diálogo **Opciones de Internet**.

### <a name="create-zscaler-three-test-user"></a>Creación de un usuario de prueba de Zscaler Three

En esta sección se crea un usuario llamado B. Simon en Zscaler Three. Zscaler Three admite el aprovisionamiento Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Zscaler Three, se crea uno al intentar acceder a la aplicación.

>[!Note]
>Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Zscaler Three](https://www.zscaler.com/company/contact).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Zscaler Three en el panel de acceso, debería iniciar sesión automáticamente en la versión de Zscaler Three para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Zscaler Three con Azure AD](https://aad.portal.azure.com/)

