---
title: 'Tutorial: Integración de Azure Active Directory con la plataforma de productividad ClickUp | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y la plataforma de productividad ClickUp.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/21/2019
ms.author: jeedes
ms.openlocfilehash: 794a952ba719faa16e3584afaf8f2f292deb6c03
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551826"
---
# <a name="tutorial-azure-active-directory-integration-with-clickup-productivity-platform"></a>Tutorial: Integración de Azure Active Directory con la plataforma de productividad ClickUp

En este tutorial aprenderá a integrar la plataforma de productividad ClickUp con Azure Active Directory (Azure AD).
La integración de la plataforma de productividad ClickUp con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a la plataforma de productividad ClickUp.
* Puede permitir que los usuarios inicien sesión automáticamente en ClickUp (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con ClickUp, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción que permita el inicio de sesión único en ClickUp

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ClickUp admite el inicio de sesión único iniciado por **SP**

## <a name="adding-clickup-productivity-platform-from-the-gallery"></a>Incorporación de ClickUp desde la galería

Para configurar la integración de ClickUp en Azure AD, deberá agregar la aplicación de la galería a la lista de aplicaciones SaaS administradas.

**Para agregar ClickUp desde la galería, realice los siguientes pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **ClickUp**, seleccione la plataforma de productividad **ClickUp**  en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![ClickUp en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con ClickUp con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de ClickUp.

Para configurar y probar el inicio de sesión único de Azure AD con ClickUp, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en ClickUp](#configure-clickup-productivity-platform-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de ClickUp](#create-clickup-productivity-platform-test-user)** : para tener un homólogo de Britta Simon en ClickUp vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con ClickUp, realice los siguientes pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **ClickUp** , seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de ClickUp](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://app.clickup.com/login/sso`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://api.clickup.com/v1/team/<team_id>/microsoft`

    > [!NOTE]
    > El valor del identificador no es real. Actualícelo con el valor del identificador real, que se explica más adelante en este tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-clickup-productivity-platform-single-sign-on"></a>Configuración del inicio de sesión único en ClickUp

1. En otra ventana del explorador web, inicie sesión en su inquilino de ClickUp como administrador.

2. Haga clic en **User profile** (Perfil de usuario) y, a continuación, seleccione **Settings** (Configuración).

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/configure0.png)

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/configure1.png)

3. Como proveedor de inicio de sesión único (SSO), seleccione **Microsoft**.

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/configure2.png)

4. En la página **Configure Microsoft Single Sign On** (Configurar inicio de sesión único de Microsoft), realice los siguientes pasos:

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/configure3.png)

    a. Haga clic en **Copy** (Copiar) para copiar el identificador de entidad y pegarlo en el cuadro de texto **Identificador (id. de entidad)** de la sección **Configuración básica de SAML** en Azure Portal.
    
    b. En el cuadro de texto **Azure Federation Metadata URL** (Dirección URL de los metadatos de federación de Azure), pegue el valor de la URL de los metadatos de federación de la aplicación que copió de Azure Portal y haga clic en **Save** (Guardar).

5. Para completar la instalación, haga clic en **Authenticate With Microsoft to complete setup** (Autenticar con Microsoft para completar la instalación) y autentíquese con la cuenta Microsoft.

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/configure4.png)

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ClickUp.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **ClickUp**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ClickUp**.

    ![Vínculo a ClickUp en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-clickup-productivity-platform-test-user"></a>Creación del usuario de prueba en ClickUp

1. En otra ventana del explorador web, inicie sesión en su inquilino de ClickUp como administrador.

2. Haga clic en **User profile** (Perfil de usuario) y, a continuación, seleccione **People** (Personas).
   
    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/configure0.png)

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/user1.png)

3. Escriba la dirección de correo electrónico del usuario en el cuadro de texto y haga clic en **Invite** (Invitar).

    ![Configuración de ClickUp](./media/clickup-productivity-platform-tutorial/user2.png)

    > [!NOTE]
    > El usuario recibirá la notificación y tendrá que aceptar la invitación para activar la cuenta.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de ClickUp del panel de acceso, debería iniciar sesión automáticamente en la aplicación ClickUp para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

