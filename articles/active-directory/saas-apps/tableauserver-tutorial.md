---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Tableau Server | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Tableau Server.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2020
ms.author: jeedes
ms.openlocfilehash: cb052f4ce962b1757f47f64e3f26fa5db2a731fa
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88999689"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-tableau-server"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Tableau Server

En este tutorial, aprenderá a integrar Tableau Server con Azure Active Directory (Azure AD). Mediante la integración de Tableau Server y Azure AD puede:

* Controlar en Azure AD quién tiene acceso a Tableau Server.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Tableau Server con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Tableau Server.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Tableau Server admite SSO iniciado por **SP**
* Una vez configurado Tableau Server, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-tableau-server-from-the-gallery"></a>Incorporación de Tableau Server desde la galería

Para configurar la integración de Tableau Server en Azure AD, será preciso que agregue Tableau Server desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Tableau Server** en el cuadro de búsqueda.
1. Seleccione **Tableau Server** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-tableau-server"></a>Configuración y prueba del inicio de sesión único de Azure AD para Tableau Server

Configure y pruebe el inicio de sesión único en Azure AD con Tableau Server mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Tableau Server.

Para configurar y probar el inicio de sesión único de Azure AD con Tableau Server es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Tableau Server](#configure-tableau-server-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Tableau Server](#create-tableau-server-test-user)** : para tener un homólogo de B. Simon en Tableau Server vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Tableau Server**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://azure.<domain name>.link`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://azure.<domain name>.link`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://azure.<domain name>.link/wg/saml/SSO/index.html`

    > [!NOTE]
    > Los valores anteriores no son valores reales. Actualice los valores con la dirección URL y el identificador releas de la página de configuración de Tableau Server que se explica más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up Tableau Server** (Configurar Tableau Server), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B. Simon use el inicio de sesión único de Azure concediéndole acceso a Tableau Server.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Tableau Server**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-tableau-server-sso"></a>Configuración del inicio de sesión único en Tableau Server

1. Para configurar SSO para la aplicación, debe iniciar sesión en su inquilino de Tableau Server como administrador.

2. En la pestaña **CONFIGURACIÓN**, seleccione **User Identity & Access** (Identidad de usuario y acceso) y, a continuación, seleccione la pestaña del método de **Autenticación**.

    ![Configurar inicio de sesión único](./media/tableauserver-tutorial/tutorial-tableauserver-auth.png)

3. En la página **CONFIGURACIÓN**, realice los siguientes pasos:

    ![Configurar inicio de sesión único](./media/tableauserver-tutorial/tutorial-tableauserver-config.png)

    a. Como **Authentication Method** (Método de autenticación), seleccione SAML.

    b. Seleccione la casilla **Enable SAML Authentication for the server**(Habilitar autenticación SAML para el servidor).

    c. Dirección URL de retorno de Tableau Server: dirección URL a la que accederán los usuarios de Tableau Server; por ejemplo, `http://tableau_server`. No se recomienda usar `http://localhost`. No se admiten las direcciones URL que tienen una barra diagonal al final (por ejemplo, `http://tableau_server/`). Copie el valor de **Tableau Server return URL** (Dirección URL de retorno de Tableau Server) y péguela en el cuadro de texto **URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal

    d. SAML entity ID (Id. de entidad SAML): el identificador de entidad identifica de forma exclusiva la instalación de Tableau Server en el IdP. Aquí, si quiere, puede especificar de nuevo la dirección URL de Tableau Server, pero no tiene que ser esa misma URL. Copie el valor de **SAML entity ID** (Id. de entidad SAML) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** de Azure Portal

    e. Haga clic en **Download XML Metadata File** (Descargar archivo de metadatos XML) y ábralo en la aplicación del editor de texto. Busque la URL del Servicio de consumidor de aserciones con Http Post e índice 0 y copie la URL. Ahora, péguelo en el cuadro de texto **URL de respuesta**  de la sección **Configuración básica de SAML** de Azure Portal

    f. Busque el archivo de metadatos de federación descargado desde Azure Portal y cárguelo en el **SAML Idp metadata file**(Archivo de metadatos del proveedor de identidades SAML).

    g. Escriba los nombres de los atributos que el proveedor de identidades usa para contener los nombres de usuario, los nombres para mostrar y las direcciones de correo electrónico.

    h. Haga clic en **Guardar**

    > [!NOTE]
    > El cliente tiene que cargar un archivo de certificado x509 con codificación PEM, con una extensión .crt y un archivo de clave privada RSA o DSA que tenga la extensión .key, como un archivo de claves de certificados. Para más información sobre el archivo de certificado y el archivo de claves de certificados, consulte [este documento](https://help.tableau.com/current/server/en-us/saml_requ.htm). Si necesita ayuda para configurar SAML en Tableau Server, consulte el artículo [Configure Server Wide SAML](https://help.tableau.com/current/server/en-us/config_saml.htm) (Configuración de SAML en todo el servidor).

### <a name="create-tableau-server-test-user"></a>Crear usuario de prueba de Tableau Server

El objetivo de esta sección es crear un usuario de prueba llamado B. Simon en Tableau Server. Debe aprovisionar todos los usuarios en el servidor de Tableau.

Tenga en cuenta que el nombre de usuario debe coincidir con el valor que ha configurado en el atributo personalizado de **nombre de usuario** de Azure AD. Con la asignación correcta, la integración debería funcionar la configuración del inicio de sesión único de Azure AD.

> [!NOTE]
> Si necesita crear un usuario manualmente, póngase en contacto con el administrador de Tableau Server de su organización.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Tableau Server en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Tableau Server para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Probar Tableau Server con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)