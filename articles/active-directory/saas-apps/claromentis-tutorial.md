---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Claromentis | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Claromentis.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/04/2019
ms.author: jeedes
ms.openlocfilehash: dba77045f8aef1799717c6f6af91046b3ca6119e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88520318"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-claromentis"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Claromentis

En este tutorial, aprenderá a integrar Claromentis con Azure Active Directory (Azure AD). Cuando integre Claromentis con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Claromentis.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Claromentis con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Claromentis.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Claromentis admite el inicio de sesión único iniciado por **SP e IDP**
* Claromentis admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-claromentis-from-the-gallery"></a>Incorporación de Claromentis desde la galería

Para configurar la integración de Claromentis en Azure AD, deberá agregar Claromentis desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Claromentis** en el cuadro de búsqueda.
1. Seleccione **Claromentis** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-claromentis"></a>Configuración y prueba del inicio de sesión único de Azure AD para Claromentis

Configure y pruebe el inicio de sesión único de Azure AD con Claromentis mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Claromentis.

Para configurar y probar el inicio de sesión único de Azure AD con Claromentis, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Claromentis](#configure-claromentis-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en Claromentis](#create-claromentis-test-user)** , para tener un homólogo de B.Simon en Claromentis que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Claromentis**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba el valor del identificador según los requisitos de su organización.

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<customer_site_url>/custom/loginhandler/simplesaml/www/module.php/saml/sp/saml2-acs.php/claromentis`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    ```https
    https://<customer_site_url>/login
    https://<customer_site_url>/login?no_auto=0
    ```

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador, la dirección URL de inicio de sesión y la dirección URL de respuesta reales, como se explica más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Claromentis**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Claromentis mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Claromentis**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-claromentis-sso"></a>Configuración del inicio de sesión único de Claromentis

1. En otra ventana del explorador, inicie sesión en el sitio web de Claromentis como administrador.

1. Haga clic en el **icono de aplicación** y seleccione **Administrador**.

    ![Configuración de Claromentis](./media/claromentis-tutorial/config1.png)

1. Seleccione la pestaña **Custom Login Handler** (Controlador del inicio de sesión personalizado).

    ![Configuración de Claromentis](./media/claromentis-tutorial/config2.png)

1. Seleccione **SAML Config** (Configuración de SAML).

    ![Configuración de Claromentis](./media/claromentis-tutorial/config3.png)

1. En la pestaña **SAML Config** (Configuración de SAML), desplácese hacia abajo a la sección **Config** (Configuración) y realice los pasos siguientes:

    ![Configuración de Claromentis](./media/claromentis-tutorial/config4.png)

    a. En el cuadro de texto **Nombre del contacto técnico**, escriba el nombre de la persona de contacto del soporte técnico.

    b. En el cuadro de texto **Correo del contacto técnico**, escriba la dirección de correo electrónico de la persona de contacto del soporte técnico.

    c. Proporcione la contraseña en el cuadro de texto **Auth Admin Password** (Autenticar contraseña del administrador).

1. Desplácese hacia abajo hasta la opción **Auth Sources** (Orígenes de autenticación) y realice los pasos siguientes:

    ![Configuración de Claromentis](./media/claromentis-tutorial/config5.png)

    a. En el cuadro de texto **IDP**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    b. En el cuadro de texto **Entity ID** (Id. de entidad), escriba el identificador de entidad.

    c. Cargue el archivo **XML de metadatos de federación** que descargó de Azure Portal.

    d. Haga clic en **Save**(Guardar).

1. Ahora observará que todas las direcciones URL se han rellenado en la sección **Proveedor de identidades** de **SAML Config** (Configuración de SAML).

    ![Configuración de Claromentis](./media/claromentis-tutorial/config6.png)

    a. Copie el valor de **Identificador (id. de entidad)** y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** de Azure Portal.

    b. Copie el valor de **URL de respuesta** y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** de Azure Portal.

    c. Copie el valor de **URL de inicio de sesión** y péguelo en el cuadro de texto **URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.

### <a name="create-claromentis-test-user"></a>Creación de un usuario de prueba de Claromentis

En esta sección, se crea un usuario llamado B.Simon en Claromentis. Claromentis admite el aprovisionamiento de usuarios Just-In-Time, el cual está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Claromentis, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Claromentis en el panel de acceso y debería iniciar sesión automáticamente en la versión de Claromentis para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Claromentis con Azure AD](https://aad.portal.azure.com/)