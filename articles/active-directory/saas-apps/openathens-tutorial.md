---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con OpenAthens | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y OpenAthens.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: jeedes
ms.openlocfilehash: fc60552fadf10dde88c0e6d8c6f46bd25c68cf2f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543948"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-openathens"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con OpenAthens

En este tutorial aprenderá a integrar OpenAthens con Azure Active Directory (Azure AD). Al integrar OpenAthens con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a OpenAthens.
* Permitir que los usuarios inicien sesión automáticamente en OpenAthens con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en OpenAthens.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* OpenAthens admite SSO iniciado por **IDP**
* OpenAthens admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-openathens-from-the-gallery"></a>Adición de OpenAthens desde la galería

Para configurar la integración de OpenAthens en Azure AD, será preciso que agregue OpenAthens desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **OpenAthens** en el cuadro de búsqueda.
1. Seleccione **OpenAthens** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-openathens"></a>Configuración y prueba del inicio de sesión único de Azure AD para OpenAthens

Configure y pruebe el inicio de sesión único de Azure AD con OpenAthens mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de OpenAthens.

Para configurar y probar el inicio de sesión único de Azure AD con OpenAthens, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en OpenAthens](#configure-openathens-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en OpenAthens](#create-openathens-test-user)** , para tener un homólogo de B.Simon en OpenAthens que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **OpenAthens**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, cargue el **archivo de metadatos del proveedor de servicios** (los pasos para ello se mencionan más adelante en este tutorial).

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![openathens cargar metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Openathens explora metadatos de carga](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, el valor de **Identificador** se rellena automáticamente en el cuadro de texto de la sección **Configuración básica de SAML**:

    ![Información de dominio y direcciones URL de inicio de sesión único de OpenAthens](common/idp-identifier.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar OpenAthens**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a OpenAthens.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **OpenAthens**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-openathens-sso"></a>Configuración del inicio de sesión único de OpenAthens

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa de OpenAthens como administrador.

1. Seleccione **Connections** (Conexiones) de la lista de la pestaña **Management** (Administración).

    ![Configurar inicio de sesión único](./media/openathens-tutorial/tutorial_openathens_application1.png)

1. Seleccione **SAML 1.1/2.0** y, a continuación, seleccione el botón **Configure** (Configurar).

    ![Configurar inicio de sesión único](./media/openathens-tutorial/tutorial_openathens_application2.png)

1. Para agregar la configuración, seleccione el botón **Browse** (Examinar) para cargar el archivo .xml de metadatos que descargó desde Azure Portal y, a continuación, seleccione **Add** (Agregar).

    ![Configurar inicio de sesión único](./media/openathens-tutorial/tutorial_openathens_application3.png)

1. Realice los pasos siguientes en la pestaña **Details** (Detalles).

    ![Configurar inicio de sesión único](./media/openathens-tutorial/tutorial_openathens_application4.png)

    a. En **Display name mapping** (Asignación de nombre para mostrar), seleccione **Use attribute** (Usar atributo).

    b. En el cuadro de texto **Display name attribute** (Atributo de nombre para mostrar), escriba el valor `http://schema.microsoft.com/identity/claims/displayname`.

    c. En **Unique user mapping** (Asignación de usuario único), seleccione **Use attribute** (Usar atributo).

    d. En el cuadro de texto **Unique user attribute** (Atributo de usuario único), introduzca el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    e. En **Status** (Estado), seleccione las tres casillas de verificación.

    f. En **Create local accounts** (Crear cuentas locales), seleccione **automatically** (automáticamente).

    g. Seleccione **Save changes** (Guardar los cambios).

    h. Desde la pestaña **Usuario de confianza**, copie la **Dirección URL de metadatos** y ábrala en el explorador para descargar el archivo **XML de metadatos de SP**. Cargue este archivo de metadatos de SP en la sección **Configuración básica de SAML** de Azure AD.

    ![Configurar inicio de sesión único](./media/openathens-tutorial/tutorial_openathens_application5.png)

### <a name="create-openathens-test-user"></a>Creación de un usuario de prueba de OpenAthens

En esta sección, se crea un usuario llamado a Britta Simon en OpenAthens. OpenAthens admite el **aprovisionamiento de usuarios Just-In-Time**, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en OpenAthens, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de OpenAthens en el panel de acceso, debería iniciar sesión automáticamente en la versión de OpenAthens para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe OpenAthens con Azure AD](https://aad.portal.azure.com/)