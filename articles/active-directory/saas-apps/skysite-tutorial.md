---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SKYSITE | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SKYSITE.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/27/2019
ms.author: jeedes
ms.openlocfilehash: 1454be235c98e5348d1b57028d988252191ad270
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88528053"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-skysite"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SKYSITE

En este tutorial, obtendrá información sobre cómo integrar SKYSITE con Azure Active Directory (Azure AD). Al integrar SKYSITE con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SKYSITE.
* Permitir que los usuarios inicien sesión automáticamente en SKYSITE con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en SKYSITE.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SKYSITE admite el inicio de sesión único iniciado por **IDP**

* SKYSITE admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-skysite-from-the-gallery"></a>Adición de SKYSITE desde la galería

Para configurar la integración de SKYSITE en Azure AD, deberá agregar SKYSITE desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SKYSITE** en el cuadro de búsqueda.
1. Seleccione **SKYSITE** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-skysite"></a>Configuración y prueba del inicio de sesión único de Azure AD para SKYSITE

Configure y pruebe el inicio de sesión único de Azure AD con SKYSITE mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SKYSITE.

Para configurar y probar el inicio de sesión único de Azure AD con SKYSITE, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en SKYSITE](#configure-skysite-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SKYSITE](#create-skysite-test-user)** , para tener un homólogo de B.Simon en SKYSITE que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **SKYSITE**, haga clic en la pestaña **Propiedades** y realice el paso siguiente: 

    ![Propiedades del inicio de sesión único](./media/skysite-tutorial/config05.png)

    * Copie la **dirección URL de acceso de usuario** y péguela en la **sección Configurar dirección URL de inicio de sesión único de SKYSITE**, que se explica más adelante en el tutorial.

1. En la página de integración de la aplicación de **SKYSITE** , vaya a **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada en el modo iniciado por **IDP** y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón **Guardar**.

1. La aplicación SKYSITE espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación SKYSITE espera que se usen algunos atributos más en la respuesta de SAML. En la sección **User Attributes & Claims** (Atributos y notificaciones del usuario) del cuadro de diálogo **Group Claims (Preview)** (Notificaciones de grupo [versión preliminar]), siga estos pasos:

    a. Haga clic en el **lápiz** junto a **Groups returned in claim** (Grupos devueltos en la notificación).

    ![imagen](./media/skysite-tutorial/config01.png)

    ![imagen](./media/skysite-tutorial/config02.png)

    b. Seleccione **Todos los grupos** en la lista de selección.

    c. Seleccione **Atributo de origen** de **Id. de grupo**.

    d. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar SKYSITE**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a SKYSITE mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SKYSITE**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="configure-skysite-sso"></a>Configuración del inicio de sesión único de SKYSITE

1. Abra una nueva ventana del explorador web e inicie sesión en el sitio de empresa de SKYSITE como administrador y haga lo siguiente:

4. Haga clic en **Settings** (Configuración) en la parte superior derecha de la página y, luego, vaya a **Account setting** (Configuración de cuenta).

    ![Configuración](./media/skysite-tutorial/config03.png)

5. Cambie a la pestaña **Inicio de sesión único (SSO)** y siga estos pasos:

    ![Configuración](./media/skysite-tutorial/config04.png)

    a. En el cuadro de texto **Identity Provider sign in URL** (Dirección URL de inicio de sesión del proveedor de identidades), pegue el valor de la **dirección URL de acceso del usuario** que copió de la pestaña **Properties** (Propiedades), en Azure Portal.

    b. Haga clic en **Upload certificate** (Cargar certificado) y, a continuación, vaya al archivo de certificado codificado en Base64 que ha descargado de Azure Portal.

    c. Haga clic en **Save**(Guardar).

### <a name="create-skysite-test-user"></a>Creación de un usuario de prueba de SKYSITE

En esta sección, se crea un usuario llamado Britta Simon en SKYSITE. SKYSITE admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en SKYSITE, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SKYSITE en el panel de acceso, debería iniciar sesión automáticamente en la versión de SKYSITE para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de SKYSITE con Azure AD](https://aad.portal.azure.com/)

