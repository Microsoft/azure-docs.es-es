---
title: 'Tutorial: Integración de Azure Active Directory con Carbonite Endpoint Backup | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Carbonite Endpoint Backup.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/06/2019
ms.author: jeedes
ms.openlocfilehash: 9888ae6f7d28f6fef5901ea246d71d7b9ae9f78f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88530620"
---
# <a name="tutorial-integrate-carbonite-endpoint-backup-with-azure-active-directory"></a>Tutorial: Integración de Carbonite Endpoint Backup con Azure Active Directory

En este tutorial, aprenderá a integrar Carbonite Endpoint Backup con Azure Active Directory (Azure AD). Al integrar Carbonite Endpoint Backup con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Carbonite Endpoint Backup.
* Permitir que los usuarios inicien sesión automáticamente en Carbonite Endpoint Backup con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) de Carbonite Endpoint Backup.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Carbonite Endpoint Backup admite el inicio de sesión único iniciado por **SP e IDP**

## <a name="adding-carbonite-endpoint-backup-from-the-gallery"></a>Incorporación de Carbonite Endpoint Backup desde la galería

Para configurar la integración de Carbonite Endpoint Backup en Azure AD, deberá agregar la aplicación desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Carbonite Endpoint Backup** en el cuadro de búsqueda.
1. Seleccione **Carbonite Endpoint Backup** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Carbonite Endpoint Backup mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Carbonite Endpoint Backup.

Para configurar y probar el inicio de sesión único de Azure AD con Carbonite Endpoint Backup, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración del inicio de sesión único en Carbonite Endpoint Backup](#configure-carbonite-endpoint-backup-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de Carbonite Endpoint Backup](#create-carbonite-endpoint-backup-test-user)** : para tener un homólogo de Britta Simon en Carbonite Endpoint Backup vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Carbonite Endpoint Backup**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una de las siguientes direcciones URL:

    ```http
    https://red-us.mysecuredatavault.com
    https://red-apac.mysecuredatavault.com
    https://red-fr.mysecuredatavault.com
    https://red-emea.mysecuredatavault.com
    https://kamino.mysecuredatavault.com
    ```

    b. En el cuadro de texto **URL de respuesta**, escriba una de las siguientes direcciones URL:

    ```http
    https://red-us.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-apac.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-fr.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-emea.mysecuredatavault.com/AssertionConsumerService.aspx
    ```

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una de las siguientes direcciones URL:

    ```http
    https://red-us.mysecuredatavault.com/
    https://red-apac.mysecuredatavault.com/
    https://red-fr.mysecuredatavault.com/
    https://red-emea.mysecuredatavault.com/
    ```

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Carbonite Endpoint Backup** (Configurar Carbonite Endpoint Backup), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-carbonite-endpoint-backup-sso"></a>Configuración del inicio de sesión único en Carbonite Endpoint Backup

1. Para automatizar la configuración en Carbonite Endpoint Backup, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Tras agregar la extensión al explorador, haga clic en **Configurar Carbonite Endpoint Backup**; se le dirigirá a la aplicación Carbonite Endpoint Backup. Desde allí, proporcione las credenciales de administrador para iniciar sesión en Carbonite Endpoint Backup. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar manualmente Carbonite Endpoint Backup, abra una nueva ventana del explorador web, inicie sesión en el sitio de la compañía de Carbonite Endpoint Backup como administrador y realice los pasos siguientes:

4. Haga clic en **Company** (Compañía) en el panel izquierdo.

    ![Configuración de Carbonite Endpoint Backup ](media/carbonite-endpoint-backup-tutorial/configure1.png)

5. Haga clic en **Single sign-on** (Inicio de sesión único).

    ![Configuración de Carbonite Endpoint Backup ](media/carbonite-endpoint-backup-tutorial/configure2.png)

6. Haga clic en **Enable** (Habilitar) y en **Edit settings** (Editar configuración) para la configuración.

    ![Configuración de Carbonite Endpoint Backup ](media/carbonite-endpoint-backup-tutorial/configure3.png)

7. En la página de configuración **Single sign-on** (Inicio de sesión único), siga estos pasos:

    ![Configuración de Carbonite Endpoint Backup ](media/carbonite-endpoint-backup-tutorial/configure4.png)

    1. En el cuadro de texto **Identity provider name** (Nombre del proveedor de identidades), pegue el valor de **Azure AD Identifier** (Identificador de Azure AD) que ha copiado de Azure Portal.

    1. En el cuadro de texto **Identity Provider URL** (Dirección URL del proveedor de identidades), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal.

    1. Haga clic en **Elegir archivo** para cargar el archivo **Certificado (Base64)** que ha descargado de Azure Portal.

    1. Haga clic en **Save**(Guardar).

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

En esta sección va a conceder acceso a B.Simon a Carbonite Endpoint Backup para permitirle usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Carbonite Endpoint Backup**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-carbonite-endpoint-backup-test-user"></a>Creación del usuario de prueba en Carbonite Endpoint Backup

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Carbonite Endpoint Backup.

1. Haga clic en **Usuarios** en el panel izquierdo y en **Agregar usuario**.

    ![Incorporación de un usuario a Carbonite Endpoint Backup](media/carbonite-endpoint-backup-tutorial/adduser1.png)

1. En la página **Agregar usuario**, siga estos pasos:

    ![Incorporación de un usuario a Carbonite Endpoint Backup](media/carbonite-endpoint-backup-tutorial/adduser2.png)

    1. Escriba el **correo electrónico**, el **nombre** y los **apellidos** del usuario y proporciónele los permisos necesarios los requisitos de la organización.

    1. Haga clic en **Agregar usuario**.

### <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Carbonite Endpoint Backup del panel de acceso, debería iniciar sesión automáticamente en la aplicación Carbonite Endpoint Backup para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)