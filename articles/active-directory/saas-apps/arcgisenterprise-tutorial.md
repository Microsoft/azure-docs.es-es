---
title: 'Tutorial: Integración de Azure Active Directory con ArcGIS Enterprise | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y ArcGIS Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2018
ms.author: jeedes
ms.openlocfilehash: a544a98b3a9d5c2d30f95240325e91018d2f40c9
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549625"
---
# <a name="tutorial-azure-active-directory-integration-with-arcgis-enterprise"></a>Tutorial: Integración de Azure Active Directory con ArcGIS Enterprise

En este tutorial, aprenderá a integrar ArcGIS Enterprise con Azure Active Directory (Azure AD).
La integración de ArcGIS Enterprise con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a ArcGIS Enterprise.
* Puede permitir que los usuarios inicien sesión automáticamente en ArcGIS Enterprise (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con ArcGIS Enterprise, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción de ArcGIS Enterprise con el inicio de sesión único habilitado

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ArcGIS Enterprise admite el inicio de sesión único iniciado por **SP e IDP**
* ArcGIS Enterprise admite el aprovisionamiento de usuarios **Just-In-Time**


## <a name="adding-arcgis-enterprise-from-the-gallery"></a>Adición de ArcGIS Enterprise desde la galería

Para configurar la integración de ArcGIS Enterprise en Azure AD, es preciso agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar ArcGIS Enterprise desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **ArcGIS Enterprise**, seleccione **ArcGIS Enterprise** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![ArcGIS Enterprise en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con [nombre de aplicación] utilizando usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de [nombre de aplicación].

Para configurar y probar el inicio de sesión único de Azure AD con [nombre de aplicación], es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de ArcGIS Enterprise](#configure-arcgis-enterprise-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en ArcGIS Enterprise](#create-arcgis-enterprise-test-user)**: el objetivo es tener un homólogo de Britta Simon en ArcGIS Enterprise que esté vinculado a la representación de ella en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con [nombre de aplicación], siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **ArcGIS Enterprise**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración de SAML básica**, realice los siguientes pasos si desea configurar la aplicación en el modo iniciado por **IDP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de ArcGIS Enterprise](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `<EXTERNAL_DNS_NAME>.portal`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<EXTERNAL_DNS_NAME>/portal/sharing/rest/oauth2/saml/signin2`

    c. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de ArcGIS Enterprise](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<EXTERNAL_DNS_NAME>/portal/sharing/rest/oauth2/saml/signin`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de ArcGIS Enterprise](mailto:support@esri.com) para obtener estos valores. El valor de Identificador se obtendrá en la **sección Set Identity Provider** (Definir proveedor de identidades), que se explica más adelante en este tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-arcgis-enterprise-single-sign-on"></a>Configuración del inicio de sesión único de ArcGIS Enterprise

1. Para automatizar la configuración en ArcGIS Enterprise, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Set up ArcGIS Enterprise** (Configurar ArcGIS Enterprise) para ir a la aplicación del mismo nombre. Desde allí, proporcione las credenciales de administrador para iniciar sesión en ArcGIS Enterprise. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

1. Si desea configurar ArcGIS Enterprise manualmente, inicie sesión en el sitio de la empresa ArcGIS Enterprise como administrador.


1. Seleccione **Organization >EDIT SETTINGS** (Organización > Editar configuración).

    ![Configuración de ArcGIS Enterprise](./media/arcgisenterprise-tutorial/configure1.png)

1. Seleccione la pestaña **Seguridad**.

    ![Configuración de ArcGIS Enterprise](./media/arcgisenterprise-tutorial/configure2.png)

1. Desplácese hacia abajo hasta la sección **Enterprise Logins via SAML** (Inicios de sesión empresariales mediante SAML) y seleccione **SET ENTERPRISE LOGIN** (Establecer inicio de sesión empresarial).

    ![Configuración de ArcGIS Enterprise](./media/arcgisenterprise-tutorial/configure3.png)

1. En la sección **Set Identity Provider** (Definición del proveedor de identidades), realice los siguientes pasos:

    ![Configuración de ArcGIS Enterprise](./media/arcgisenterprise-tutorial/configure4.png)

    a. Proporcione un nombre como **Azure Active Directory Test** (Prueba de Azure Active Directory) en el cuadro de texto **Name** (Nombre).

    b. En el cuadro de texto **URL**, pegue el valor de **Dirección URL de metadatos de federación de aplicación** que copió de Azure Portal.

    c. Haga clic en **Show advanced settings** (Mostrar configuración avanzada) y copie el valor de **Entity ID** (Id. de entidad) y péguelo en el cuadro de texto **Identificador** de la sección **Dominio y direcciones URL de ArcGIS Enterprise** de Azure Portal.
    
    ![Configuración de ArcGIS Enterprise](./media/arcgisenterprise-tutorial/configure5.png)

    d. Haga clic en **UPDATE IDENTITY PROVIDER** (Actualizar proveedor de identidades).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ArcGIS Enterprise.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **ArcGIS Enterprise**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **ArcGIS Enterprise**.

    ![Vínculo a ArcGIS Enterprise en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-arcgis-enterprise-test-user"></a>Creación de un usuario de prueba en ArcGIS Enterprise

En esta sección, se crea un usuario llamado a Britta Simon en ArcGIS Enterprise. ArcGIS Enterprise admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en ArcGIS Enterprise, se crea uno después de la autenticación.

> [!Note]
> Si necesita crear un usuario manualmente, póngase en contacto con el  [equipo de soporte técnico de ArcGIS Enterprise](mailto:support@esri.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de ArcGIS Enterprise del panel de acceso, debería iniciar sesión automáticamente en la versión de ArcGIS Enterprise para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

