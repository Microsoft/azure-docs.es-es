---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cisco Webex Meetings | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Cisco Webex Meetings.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 26704ca7-13ed-4261-bf24-fd6252e2072b
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/21/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7a6182159275236f023a1647275ed1fb8c8f4112
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905795"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cisco-webex-meetings"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cisco Webex Meetings

En este tutorial, aprenderá a integrar Cisco Webex Meetings con Azure Active Directory (Azure AD). Al integrar Cisco Webex Meetings con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Cisco Webex Meetings.
* Permitir que los usuarios inicien sesión automáticamente en Cisco Webex Meetings con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Cisco Webex Meetings.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cisco Webex Meetings admite el inicio de sesión único iniciado por **SP e IDP**

* Cisco Webex Meetings admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-cisco-webex-meetings-from-the-gallery"></a>Adición de Cisco Webex Meetings desde la galería

Para configurar la integración de Cisco Webex Meetings en Azure AD, será preciso que agregue Cisco Webex Meetings desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Cisco Webex Meetings** en el cuadro de búsqueda.
1. Seleccione **Cisco Webex Meetings** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-cisco-webex-meetings"></a>Configuración y prueba del inicio de sesión único de Azure AD para Cisco Webex Meetings

Configure y pruebe el inicio de sesión único de Azure AD con Cisco Webex Meetings utilizando un usuario de prueba llamado **B. Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Cisco Webex Meetings.

Para configurar y probar el inicio de sesión único de Azure AD con Cisco Webex Meetings, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de Cisco Webex Meetings](#configure-cisco-webex-meetings-sso)**, para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Cisco Webex Meetings](#create-cisco-webex-meetings-test-user)**, para tener un homólogo de B. Simon en Cisco Webex Meetings vinculado a la representación del usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Cisco Webex Meetings**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, puede configurar la aplicación en modo iniciado por**IDP** mediante la carga del archivo de **metadatos del proveedor de servicios** de la siguiente manera:

    a. Haga clic en **Cargar el archivo de metadatos**.

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    c. Cuando el archivo de metadatos del proveedor de servicios se haya cargado correctamente, los valores **Identificador** y **URL de respuesta** se rellenarán automáticamente en la sección **Configuración básica de SAML**.

    >[!Note]
    >Obtendrá el archivo de metadatos del proveedor de servicios en la sección **Configuración del inicio de sesión único de Cisco Webex Meetings**, que se explica más adelante en este tutorial. 

1. Si desea configurar la aplicación en modo iniciado por **SP**, realice los siguientes pasos:  

    a. En la sección **Configuración básica de SAML**, haga clic en el icono de edición o con forma de lápiz.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)
    
    b. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL con el siguiente patrón: ` https://<customername>.my.webex.com`

5. La aplicación Cisco Webex Meetings espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

6. Además de lo anterior, la aplicación Cisco Webex Meetings espera que se usen algunos atributos más en la respuesta de SAML. En la sección Notificaciones del usuario del cuadro de diálogo Atributos de usuario, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguiente: 

    | Nombre | Atributo de origen|
    | ---------------|  --------- |
    |   firstname    | user.givenname |
    |   lastname    | user.surname |
    |   email       | user.mail |
    |   uid    | user.mail |

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, seleccione el valor de atributo que se muestra para esa fila en la lista desplegable.

    f. Haga clic en **Save**(Guardar).

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Cisco Webex Meetings** (Configurar Cisco Webex Meetings), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B. Simon acceda a Cisco Webex Meetings mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Cisco Webex Meetings**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cisco-webex-meetings-sso"></a>Configuración del inicio de sesión único de Cisco Webex Meetings

1. Vaya a la dirección URL `https://<customername>.webex.com/admin` con sus credenciales de administración.

2. Vaya a **Common Site Setting** (Configuración del sitio común) y a **SSO Configuration** (Configuración de inicio de sesión único).
 
    ![Configurar inicio de sesión único](./media/cisco-webex-tutorial/tutorial-cisco-webex-11.png)

3. En la página **Webex Administration** (Administración de Webwex), siga estos pasos:

    ![Configurar inicio de sesión único](./media/cisco-webex-tutorial/tutorial-cisco-webex-10.png)

    a. Seleccione **SAML 2.0** como **Federation Protocol** (Protocolo de Federación).

    b. Haga clic en el enlace **Import SAML Metadata** (Importar metadatos de SAML) para cargar el archivo de metadatos que descargó de Azure Portal.

    c. Haga clic en el botón **Export** (Exportar) para descargar el archivo de metadatos del proveedor de servicios y cárguelo en la sección **Configuración básica de SAML** de Azure Portal.

    d. En el cuadro de texto **AuthContextClassRef**, escriba `urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified` y, si desea habilitar la autenticación multifactor con Azure AD, escriba los dos valores, como `urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport;urn:oasis:names:tc:SAML:2.0:ac:classes:X509`.

    e. Seleccione **Auto Account Creation** (Creación de cuenta automática).

    >[!NOTE]
    >Para habilitar el aprovisionamiento de usuarios **Just-In-Time**, debe seleccionar **Auto Account Creation** (Creación de cuenta automática). Además, es necesario pasar atributos de token de SAML en la respuesta SAML.

    f. Haga clic en **Save**(Guardar).

    >[!NOTE]
    >Esta configuración es solo para los clientes que usan el UserID de Webex en formato de correo electrónico.

### <a name="create-cisco-webex-meetings-test-user"></a>Creación del usuario de prueba de Cisco Webex Meetings

El objetivo de esta sección es crear un usuario llamado B. Simon en Cisco Webex Meetings. Cisco Webex Meetings admite el aprovisionamiento **Just-In-Time**, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Cisco Webex Meetings, se crea uno al intentar acceder.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Cisco Webex Meetings en el panel de acceso, debería iniciar sesión automáticamente en la versión de Cisco Webex Meetings para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe ServiceNow con Azure AD](https://aad.portal.azure.com)
