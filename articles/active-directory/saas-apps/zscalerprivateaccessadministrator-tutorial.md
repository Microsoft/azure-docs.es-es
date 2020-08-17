---
title: 'Tutorial: Integración de Azure Active Directory con Zscaler Private Access Administrator | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler Private Access Administrator.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: c87392a7-e7fe-4cdc-a8e6-afe1ed975172
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 104583522c7041d89d9c63ba67a5633a3bd383f3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87016496"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-private-access-administrator"></a>Tutorial: Integración de Azure Active Directory con Zscaler Private Access Administrator

En este tutorial, aprenderá a integrar Zscaler Private Access Administrator con Azure Active Directory (Azure AD).
La integración de Zscaler Private Access Administrator con Azure AD ofrece las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Zscaler Private Access Administrator.
* Puede permitir que los usuarios inicien sesión automáticamente en Zscaler Private Access Administrator (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Zscaler Private Access Administrator, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único de Zscaler Private Access Administrator

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zscaler Private Access Administrator admite el inicio de sesión único iniciado por **SP** e **IDP**.

## <a name="adding-zscaler-private-access-administrator-from-the-gallery"></a>Adición de Zscaler Private Access Administrator desde la galería

Para configurar la integración de Zscaler Private Access Administrator con Azure AD, debe agregar Zscaler Private Access Administrator desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Zscaler Private Access Administrator desde la galería, realice los siguientes pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler Private Access Administrator**, seleccione **Zscaler Private Access Administrator** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![Zscaler Private Access Administrator en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Zscaler Private Access Administrator con un usuario de prueba llamado **Britta Simon**.
Para que funcione el inicio de sesión único, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado en Zscaler Private Access Administrator.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler Private Access Administrator, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Zscaler Private Access Administrator](#configure-zscaler-private-access-administrator-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Zscaler Private Access Administrator](#create-zscaler-private-access-administrator-test-user)** : para tener un homólogo de Britta Simon en Zscaler Private Access Administrator que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único en Azure AD con Zscaler Private Access Administrator, siga los pasos que se indican a continuación:

1. En la página de integración de la aplicación **Zscaler Private Access Administrator** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Zscaler Private Access Administrator](common/idp-relay.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<subdomain>.private.zscaler.com/auth/metadata`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.private.zscaler.com/auth/sso`

    c. Haga clic en **Establecer direcciones URL adicionales**.

    d. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL: `idpadminsso`

5.  Si desea configurar la aplicación en modo iniciado por **SP**, realice el siguientes paso:

    ![Información de dominio y direcciones URL de inicio de sesión único de Zscaler Private Access Administrator](common/both-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.private.zscaler.com/auth/sso`   

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Zscaler Private Access Administrator](https://help.zscaler.com/zpa-submit-ticket) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up Zscaler Private Access Administrator** (Configurar Zscaler Private Access Administrator), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-zscaler-private-access-administrator-single-sign-on"></a>Configuración del inicio de sesión único de Zscaler Private Access Administrator

1. En otra ventana del explorador web, inicie sesión en Zscaler Private Access Administrator como administrador.

2. En la parte superior, haga clic en **Administration** (Administración) y vaya a la sección **AUTHENTICATION** (Autenticación) y haga clic en **IdP Configuration** (Configuración de IdP).

    ![Administrador de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

3. En la esquina superior derecha, haga clic en **Add IdP Configuration** (Agregar configuración de IdP). 

    ![Agregar IdP de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addpidp.png)

4. En la página **Add IdP Configuration** (Agregar configuración de IdP), realice estos pasos:
 
    ![Seleccionar IdP de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_idpselect.png)

    a. Haga clic en **Select File** (Seleccionar archivo) para cargar el archivo de metadatos descargado de Azure AD en el campo **IdP Metadata File Upload** (Carga de archivo de metadatos de IdP).

    b. Lee los **metadatos de IdP** de Azure AD y rellena la información de todos los campos como se indica a continuación.

    ![Configurar IdP de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/idpconfig.png)

    c. Seleccione **Single Sign On** (Inicio de sesión único) como **Administrator** (Administrador).

    d. Seleccione el dominio en el campo **Domains** (Dominios).
    
    e. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a Zscaler Private Access Administrator.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, después, **Zscaler Private Access Administrator**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Zscaler Private Access Administrator**.

    ![Vínculo a Zscaler Private Access Administrator en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-zscaler-private-access-administrator-test-user"></a>Creación de un usuario de prueba en Zscaler Private Access Administrator

Para permitir que los usuarios de Azure AD inicien sesión en Zscaler Private Access Administrator, se les debe aprovisionar en Zscaler Private Access Administrator. El aprovisionamiento en Zscaler Private Access Administrator consiste en una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el sitio de la empresa de Zscaler Private Access Administrator.

2. En la parte superior, haga clic en **Administration** (Administración) y vaya a la sección **AUTHENTICATION** (Autenticación) y haga clic en **IdP Configuration** (Configuración de IdP).

    ![Administrador de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

3. Haga clic en **Administrators** (Administradores) en el lado izquierdo del menú.

    ![Administrador de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_adminstrator.png)

4. En la esquina superior derecha, haga clic en **Add Administrator** (Agregar administrador):

    ![Agregar administrador en Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addadmin.png)

5. En la página **Add Administrator** (Agregar administrador), siga estos pasos:

    ![Administrador de usuarios de Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_useradmin.png)

    a. En el cuadro de texto **Username** (Nombre de usuario), escriba el correo electrónico del usuario, en este caso BrittaSimon@contoso.com.

    b. En el cuadro de texto **Password** (Contraseña), escriba la contraseña.

    c. En el cuadro de texto **Confirm Password** (Confirmar contraseña), escriba la contraseña.

    d. Seleccione **Role** (Rol) como **Zscaler Private Access Administrator**.

    e. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario con el formato BrittaSimon@contoso.com.

    f. En el cuadro de texto **Phone** (Teléfono), escriba el número de teléfono.

    g. En el cuadro de texto **Timezone** (Zona horaria), seleccione la zona horaria.

    h. Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Zscaler Private Access Administrator en el Panel de acceso, debería iniciar sesión automáticamente en la aplicación Zscaler Private Access Administrator para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

