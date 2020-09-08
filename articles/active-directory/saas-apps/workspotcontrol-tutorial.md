---
title: 'Tutorial: integración de Azure Active Directory con Workspot Control | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workspot Control.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 3/11/2019
ms.author: jeedes
ms.openlocfilehash: bfbecc71638e6feaaf29809f09dda752dd29b2ae
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88526642"
---
# <a name="tutorial-azure-active-directory-integration-with-workspot-control"></a>Tutorial: integración de Azure Active Directory con Workspot Control

En este tutorial, aprenderá a integrar Workspot Control con Azure Active Directory (Azure AD). Al integrar Workspot Control con Azure AD, puede hacer lo siguiente:

* Usar Azure AD para controlar quién tiene acceso a Workspot Control.
* Permitir que los usuarios inicien sesión automáticamente en Workspot Control (inicio de sesión-único [SSO]) con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones con Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Workspot Control, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).

* Una suscripción habilitada para el inicio de sesión único en Workspot Control.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

> [!Note]
> Workspot Control admite el inicio de sesión único iniciado por SP e iniciado por IDP.


## <a name="adding-workspot-control-from-the-gallery"></a>Incorporación de Workspot Control desde la galería

Para configurar la integración de Workspot Control en Azure AD, debe agregar Workspot Control desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Workspot Control desde la galería, siga estos pasos:**

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

3. Seleccione **Nueva aplicación** en la parte superior de la ventana.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Workspot Control**, seleccione **Workspot Control** en el panel de resultados y, a continuación, seleccione **Agregar**.

     ![Ventana "Agregar desde la galería"](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Workspot Control con un usuario de prueba llamado "Britta Simon".
Para que el inicio de sesión único funcione, es preciso establecer un vínculo entre un usuario de Azure AD y el usuario relacionado de Workspot Control.

Para configurar y probar el inicio de sesión único de Azure AD con Workspot Control, debe completar las siguientes tareas:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on), para permitir que los usuarios utilicen esta característica.
2. [Configuración del inicio de sesión único de Workspot Control:](#configure-workspot-control-single-sign-on) para configurar el inicio de sesión único en la aplicación.
3. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD para Britta Simon.
4. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. [Creación de un usuario de prueba de Workspot Control](#create-a-workspot-control-test-user) para tener un homólogo de Britta Simon en Workspot Control que esté vinculado a la representación del usuario en Azure AD.
6. [Prueba del inicio de sesión único](#test-single-sign-on) para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Workspot Control, realice los pasos siguientes:

1. En la página de integración de la aplicación **Workspot Control** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En la ventana **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML** para habilitar el inicio de sesión único.

    ![Ventana Seleccionar un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** (lápiz) para acceder a la **Configuración básica de SAML**.

    ![Icono Editar resaltado de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por IDP, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Workspot Control](common/idp-intiated.png)

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:<br/>
    ***https://<<i></i>INSTANCENAME>-saml.workspot.com/saml/metadata***

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:<br/>
    ***https://<<i></i>INSTANCENAME>-saml.workspot.com/saml/assertion***

5. Seleccione **Establecer direcciones URL adicionales** si desea configurar la aplicación en el modo iniciado por SP.

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Workspot Control](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:<br/>
    ***https://<<i></i>INSTANCENAME>-saml.workspot.com/***

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico al cliente de Workspot Control](mailto:support@workspot.com) para obtener estos valores. También puede hacer referencia a los patrones de la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **certificado (Base 64)** de las opciones disponibles según sus requisitos. Guárdelo en el equipo.

    ![Vínculo de descarga del certificado (Base64)](common/certificatebase64.png)

7. En la sección **Configurar Workspot Control**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    - **Dirección URL de inicio de sesión**

    - **Identificador de Azure AD**

    - **Dirección URL de cierre de sesión**

### <a name="configure-workspot-control-single-sign-on"></a>Configuración del inicio de sesión único de Workspot Control

1. En otra ventana del explorador web, inicie sesión en Workspot Control como administrador de seguridad.

2. En la barra de herramientas de la parte superior de la página, seleccione **Setup** (Configurar) y, a continuación, **SAML**.

    ![Opciones de configuración](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_setup.png)

3. En la ventana **Security Assertion Markup Language Configuration** (Configuración de Lenguaje de marcado de aserción de seguridad), realice los pasos siguientes:
 
    ![Ventana de Configuración de Lenguaje de marcado de aserción de seguridad](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_saml.png)

    1. En el cuadro **Entity ID** (Identificador de entidad), pegue el **identificador de Azure AD** que copió de Azure Portal.

    1. En el cuadro **Signon Service URL** (Dirección URL del servicio de inicio de sesión), pegue la **Dirección URL de inicio de sesión** que copió de Azure Portal.

    1. En el cuadro **Logout Service URL** (Dirección URL del servicio de cierre de sesión), pegue la **Dirección URL de cierre de sesión** que copió de Azure Portal.

    1. Seleccione **Update File** (Actualizar archivo) para cargar en el certificado X.509 el certificado codificado en base 64 que descargó de Azure Portal.

    1. Seleccione **Guardar**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la ventana.

    ![Botón "Nuevo usuario"](common/new-user.png)

3. En las propiedades del usuario, siga estos pasos:

    ![Ventana Propiedades del usuario](common/user-properties.png)

    1. En el campo **Nombre**, escriba **BrittaSimon**.
  
    1. En el campo **Nombre de usuario**, escriba **brittasimon@* dominiodeempresa.extensión***. Por ejemplo, escriba **BrittaSimon@contoso.<i></i>com**.

    1. Active la casilla de verificación **Mostrar contraseña**. Y, después, anote el valor que se muestra en el cuadro **Password** (Contraseña).

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a Britta Simon a Workspot Control para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, a continuación, **Workspot Control**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Workspot Control**.

    ![Vínculo de Workspot Control en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Seleccione el botón **Agregar usuario**. A continuación, seleccione **Usuarios y grupos** en la ventana **Agregar asignación**.

    ![Ventana "Agregar asignación"](common/add-assign-user.png)

5. En la ventana **Usuarios y grupos**, en la lista **Usuarios**, seleccione **Britta Simon**. Después, haga clic en **Seleccionar**.

6. Si espera algún valor de rol en la aserción de SAML, seleccione el rol adecuado para el usuario en la lista de la ventana **Seleccionar rol**. A continuación, haga clic en **Seleccionar** en la parte inferior.

7. En la ventana **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-workspot-control-test-user"></a>Creación de un usuario de prueba de Workspot Control

Para permitir que los usuarios de Azure AD inicien sesión en Workspot Control, tienen que aprovisionarse en Workspot Control. El aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, siga estos pasos:**

1. Inicie sesión en Workspot Control como administrador de seguridad.

2. En la barra de herramientas de la parte superior de la página, seleccione **Users** (Usuarios) y, a continuación, **Add User** (Agregar usuario).

    ![Opciones de "Usuarios"](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_adduser.png)

3. En la ventana **Add a New User** (Agregar un nuevo usuario), siga estos pasos:

    ![Ventana "Agregar un nuevo usuario"](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_addnewuser.png)

    1. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **Britta**.

    1. En el cuadro de texto **Last Name** (Apellidos), escriba los apellidos del usuario, en este caso **Simon**.

    1. En el cuadro de texto **Email** (Correo electrónico), escriba la dirección de correo electrónico del usuario, como **Brittasimon@contoso.<i></i>com**.

    1. Seleccione el rol de usuario correspondiente de la lista desplegable **Role** (Rol).

    1. Seleccione el grupo de usuarios correspondiente de la lista desplegable **Group** (Grupo).

    1. Seleccione **Agregar usuario**.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el *Panel de acceso*.

Al hacer clic en el icono **Workspot Control** en el panel de acceso, debería iniciar sesión automáticamente en la versión de Workspot Control para la que configuró el inicio de sesión único. Para más información, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/user-help/my-apps-portal-end-user-access).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list)

- [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
