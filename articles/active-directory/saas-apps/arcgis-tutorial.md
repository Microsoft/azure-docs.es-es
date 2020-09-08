---
title: 'Tutorial: Integración de Azure Active Directory con ArcGIS Online | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y ArcGIS Online.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/19/2018
ms.author: jeedes
ms.openlocfilehash: 28e15cb7e0b519932eeee3a6a6445449affe2f35
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551377"
---
# <a name="tutorial-azure-active-directory-integration-with-arcgis-online"></a>Tutorial: Integración de Azure Active Directory con ArcGIS Online

En este tutorial, obtendrá información sobre cómo integrar ArcGIS Online con Azure Active Directory (Azure AD).
La integración de ArcGIS Online con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a ArcGIS Online.
* Puede permitir que los usuarios inicien sesión automáticamente en ArcGIS Online (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con ArcGIS Online, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en ArcGIS Online

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ArcGIS Online admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-arcgis-online-from-the-gallery"></a>Incorporación de ArcGIS Online desde la galería

Para configurar la integración de ArcGIS Online en Azure AD, deberá agregar ArcGIS Online desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar ArcGIS Online desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **ArcGIS Online**, seleccione **ArcGIS Online** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![ArcGIS Online en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección podrá configurar y probar el inicio de sesión único de Azure AD con ArcGIS Online y un usuario de prueba denominado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de ArcGIS Online.

Para configurar y probar el inicio de sesión único de Azure AD con ArcGIS Online, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de ArcGIS Online](#configure-arcgis-online-single-sign-on)** : permite configurar las opciones de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de ArcGIS Online](#create-arcgis-online-test-user)** : permite tener un homólogo de Britta Simon en ArcGIS Online que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con ArcGIS Online, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ArcGIS Online**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre el dominio y las direcciones URL de inicio de sesión único de ArcGIS Online](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.maps.arcgis.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `<companyname>.maps.arcgis.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico para clientes de ArcGIS Online](https://support.esri.com/en/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. Para automatizar la configuración en **ArcGIS Online**, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Install the extension** (Instalar la extensión).

    ![imagen](./media/arcgis-tutorial/install_extension.png)

7. Después de agregar la extensión al explorador, haga clic en **Setup ArcGIS Online** (Configurar ArcGIS Online) para ir a la aplicación ArcGIS Online. Desde allí, proporcione las credenciales de administrador para iniciar sesión en ArcGIS Online. La extensión de explorador web configurará automáticamente la aplicación y automatizará los pasos de la sección **Configuración del inicio de sesión único de ArcGIS Online**.

### <a name="configure-arcgis-online-single-sign-on"></a>Configuración del inicio de sesión único de ArcGIS Online

1. Si quiere configurar ArcGIS Online manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de ArcGIS Online como administrador y realice los pasos siguientes:

2. Haga clic en **EDITAR CONFIGURACIÓN**.

    ![Edit Settings](./media/arcgis-tutorial/ic784742.png "Edit Settings") (Editar configuración)

3. Haga clic en **Seguridad**.

    ![Seguridad](./media/arcgis-tutorial/ic784743.png "Seguridad")

4. En **Inicios de sesión de la empresa**, haga clic en **ESTABLECER PROVEEDOR DE IDENTIDADES**.

    ![Enterprise Logins (Inicios de sesión de la empresa)](./media/arcgis-tutorial/ic784744.png "Enterprise Logins (Inicios de sesión de la empresa)")

5. En la sección **Configurar proveedor de identidades** , realice los pasos siguientes:

    ![Set Identity Provider (Establecer proveedor de identidades)](./media/arcgis-tutorial/ic784745.png "Establecer proveedor de identidades")

    a. En el cuadro de texto **Nombre**, escriba el nombre de la organización.

    b. En **Metadata for the Enterprise Identity Provider will be supplied using** (Los metadatos para el proveedor de identidades de la empresa se proporcionarán con), seleccione **A File** (Un archivo).

    c. Haga clic en **Choose file**(Elegir archivo) para cargar el archivo de metadatos descargado.

    d. Haga clic en **ESTABLECER PROVEEDOR DE IDENTIDADES**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ArcGIS Online.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **ArcGIS Online**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **ArcGIS Online**.

    ![Vínculo a ArcGIS Online en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-arcgis-online-test-user"></a>Creación de un usuario de prueba de ArcGIS Online

Para permitir que los usuarios de Azure AD inicien sesión en ArcGIS Online, tienen que aprovisionarse en ArcGIS Online.  
En el caso de ArcGIS Online, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **ArcGIS** .

2. Haga clic en **INVITAR A MIEMBROS**.
   
    ![Invite Members (Invitar a miembros)](./media/arcgis-tutorial/ic784747.png "Invitar a miembros")

3. Seleccione **Agregar miembros automáticamente sin enviar un correo electrónico** y luego haga clic en **SIGUIENTE**.
   
    ![Add Members Automatically (Agregar miembros automáticamente)](./media/arcgis-tutorial/ic784748.png "Agregar miembros automáticamente")

4. En la página de diálogo **Miembros** , realice los pasos siguientes:
   
     ![Add and review (Agregar y revisar)](./media/arcgis-tutorial/ic784749.png "Agregar y revisar")
    
     a. Escriba los valores para **Email** (Correo electrónico), **First name** (Nombre) y **Last name** (Apellidos) de una cuenta de Azure AD válida que quiera aprovisionar.
  
     b. Haga clic en **AGREGAR Y REVISAR**.
5. Revise los datos que ha escrito y luego haga clic en **AGREGAR MIEMBROS**.
   
    ![Add members (Agregar miembros)](./media/arcgis-tutorial/ic784750.png "Agregar miembro")
        
    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de ArcGIS Online en el panel de acceso, debería iniciar sesión automáticamente en la versión de ArcGIS Online para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

