---
title: 'Tutorial: Integración de Azure Active Directory con Periscope Data | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Periscope Data.
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
ms.openlocfilehash: ca0bfcb0c90142924285215c427a7071297be730
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553927"
---
# <a name="tutorial-azure-active-directory-integration-with-periscope-data"></a>Tutorial: Integración de Azure Active Directory con Periscope Data

En este tutorial aprenderá a integrar Periscope Data con Azure Active Directory (Azure AD).
La integración de Periscope Data con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Periscope Data.
* Puede permitir que los usuarios inicien sesión automáticamente en Periscope Data (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Periscope Data, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción de Periscope Data habilitada para el inicio de sesión único

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Periscope Data admite el inicio de sesión único iniciado por **SP**

## <a name="adding-periscope-data-from-the-gallery"></a>Incorporación de Periscope Data desde la galería

Para configurar la integración de Periscope Data en Azure AD, deberá agregar la aplicación de la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Periscope Data desde la galería, realice los siguientes pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Periscope Data**, seleccione **Periscope Data** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Periscope Data en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Periscope Data con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Periscope Data.

Para configurar y probar el inicio de sesión único de Azure AD con Periscope Data, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Periscope Data](#configure-periscope-data-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba en Periscope Data](#create-periscope-data-test-user)** : para tener un homólogo de Britta Simon en Periscope Data vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Periscope Data, realice los siguientes pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Periscope Data**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Periscope Data](common/sp-identifier.png)

    a. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba cualquiera de estas direcciones URL:
    
    ```https
    https://app.periscopedata.com/
    https://app.periscopedata.com/app/<SITENAME>
    ```

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://app.periscopedata.com/<SITENAME>/sso`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualice estos valores con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico para clientes de Periscope Data](mailto:support@periscopedata.com) para obtener este valor y el valor del identificador, cuya explicación se muestra en la sección **Configuración del inicio de sesión único en Periscope Data** más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-periscope-data-single-sign-on"></a>Configuración del inicio de sesión único en Periscope Data

1. En otra ventana del explorador web, inicie sesión en Periscope Data como administrador.

2. Abra el menú de engranaje de la parte inferior izquierda, abra el menú **Billing** > **Security** (Facturación > Seguridad) y realice los siguientes pasos. Solo los administradores tienen acceso a esta configuración.

    ![Configuración de la información de Periscope Data](./media/periscope-data-tutorial/configure01.png)

    a. Copie el valor de **Dirección URL de metadatos de federación de aplicación** del paso 5 de **Certificado de firma de SAML** y ábrala en un explorador. Se abrirá un documento XML.

    b. En el cuadro de texto **Inicio de sesión único**, seleccione **Azure Active Directory**.

    c. Busque la etiqueta **SingleSignOnService** y pegue el valor de **Location** (Ubicación) en el cuadro de texto **SSO URL** (URL de SSO).

    d. Busque la etiqueta **SingleLogoutService** y pegue el valor de **Location** (Ubicación) en el cuadro de texto **SLO URL** (URL de SLO).

    e. Copie el valor de **Identificador** de su instancia y péguelo en el cuadro de texto **Identificador (id. de entidad)** de la sección **Configuración básica de SAML** de Azure Portal.

    f. Busque la primera etiqueta del archivo XML, copie el valor de **entityID** y péguelo en el cuadro de texto **Issuer** (Emisor).

    g. Busque la etiqueta **IDPSSODescriptor** con el protocolo SAML. Dentro de esa sección, busque la etiqueta **KeyDescriptor** con **use=signing**. Copie el valor de **X509Certificate** y péguelo en el cuadro de texto **Certificado**.

    h. Los sitios con varios espacios pueden elegir el espacio predeterminado en la lista desplegable **Default Space** (Espacio predeterminado). Este será el espacio al que se agregarán nuevos usuarios cuando inicien sesión en Periscope Data por primera vez; estos se aprovisionarán mediante el inicio de sesión único de Active Directory.

    i. Por último, escriba **Logout** y haga clic en **Guardar** y **Confirmar** para guardar y confirmar la configuración de inicio de sesión único.

    ![Configuración de la información de Periscope Data](./media/periscope-data-tutorial/configure02.png)

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Periscope Data.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Periscope Data**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Periscope Data**.

    ![Vínculo a Periscope Data en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-periscope-data-test-user"></a>Creación del usuario de prueba en Periscope Data

Para permitir que los usuarios de Azure AD inicien sesión en Periscope Data, deben aprovisionarse en Periscope Data. En Periscope Data, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Periscope Data como administrador.

2. Haga clic en el icono **Settings** (Configuración) de la parte inferior izquierda del menú y vaya a **Permissions** (Permisos).

    ![Configuración de la información de Periscope Data](./media/periscope-data-tutorial/configure03.png)

3. Haga clic en **ADD USER** (AGREGAR USUARIO) y realice los siguientes pasos:

      ![Configuración de la información de Periscope Data](./media/periscope-data-tutorial/configure04.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre de usuario, en este caso **Britta**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

    c. En el cuadro de texto **Email** (Correo electrónico), escriba el correo electrónico del usuario; por ejemplo, **brittasimon\@contoso.com**.

    d. Haga clic en **ADD** (AGREGAR).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Periscope Data en el panel de acceso, debería iniciar sesión automáticamente en la versión de Periscope Data para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

