---
title: 'Tutorial: integración de Azure Active Directory con Brightspace by Desire2Learn | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Brightspace by Desire2Learn.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: e2d3065b-1f6c-4c45-af78-0d5da3266999
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/08/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: e14ad53ad613e7cbc6631707b79f607cd39b3791
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87018497"
---
# <a name="tutorial-azure-active-directory-integration-with-brightspace-by-desire2learn"></a>Tutorial: Integración de Azure Active Directory con Brightspace by Desire2Learn

En este tutorial, obtendrá información sobre cómo integrar Brightspace by Desire2Learn con Azure Active Directory (Azure AD).
La integración de Brightspace by Desire2Learn con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a Brightspace by Desire2Learn.
* Puede permitir que los usuarios inicien sesión automáticamente en Brightspace by Desire2Learn (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Brightspace by Desire2Learn, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción que permita el inicio de sesión único de Brightspace by Desire2Learn

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Brightspace by Desire2Learn admite el inicio de sesión único iniciado por **IDP**

## <a name="adding-brightspace-by-desire2learn-from-the-gallery"></a>Incorporación de Brightspace by Desire2Learn desde la galería

Para configurar la integración de Brightspace by Desire2Learn en Azure AD, deberá agregar Brightspace by Desire2Learn desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Brightspace by Desire2Learn desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Brightspace by Desire2Learn**, seleccione **Brightspace by Desire2Learn** en el panel de resultados y, a continuación, haga clic en **Agregar** para agregar la aplicación.

     ![Brightspace by Desire2Learn en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Brightspace by Desire2Learn con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Brightspace by Desire2Learn.

Para configurar y probar el inicio de sesión único de Azure AD con Brightspace by Desire2Learn, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Brightspace by Desire2Learn](#configure-brightspace-by-desire2learn-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Brightspace by Desire2Learn](#create-brightspace-by-desire2learn-test-user)** : para tener un homólogo de Britta Simon en Brightspace by Desire2Learn que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Brightspace by Desire2Learn, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Brightspace by Desire2Learn**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Brightspace by Desire2Learn](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:
    
    ```http
    https://<companyname>.tenants.brightspace.com/samlLogin
    https://<companyname>.desire2learn.com/shibboleth-sp
    ```

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.desire2learn.com/d2l/lp/auth/login/samlLogin.d2l`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico del cliente de Brightspace by Desire2Learn](https://www.d2l.com/contact/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Brightspace by Desire2Learn** (Configurar Brightspace by Desire2Learn), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-brightspace-by-desire2learn-single-sign-on"></a>Configuración del inicio de sesión único de Brightspace by Desire2Learn

Para configurar el inicio de sesión único en **Brightspace by Desire2Learn**, es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Brightspace by Desire2Learn](https://www.d2l.com/contact/). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Brightspace by Desire2Learn.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Brightspace by Desire2Learn**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Brightspace by Desire2Learn**.

    ![Vínculo de Brightspace by Desire2Learn en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-brightspace-by-desire2learn-test-user"></a>Creación de un usuario de prueba de Brightspace by Desire2Learn

En esta sección, creará el usuario Britta Simon en Brightspace by Desire2Learn. Trabaje con el  [equipo de soporte técnico de Brightspace por Desire2Learn](https://www.d2l.com/contact/) para agregar los usuarios en la plataforma de Brightspace por Desire2Learn. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Brightspace by Desire2Learn que proporcione Brightspace by Desire2Learn para aprovisionar cuentas de usuario de Azure Active Directory.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Brightspace by Desire2Learn en el Panel de acceso, debería iniciar sesión automáticamente en la aplicación Brightspace by Desire2Learn para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)