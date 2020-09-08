---
title: 'Tutorial: Integración de Azure Active Directory con Tableau Online | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Tableau Online.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/31/2020
ms.author: jeedes
ms.openlocfilehash: 035bc13ee44a8a003ae860eb2bdd67432fa91f14
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88542531"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-tableau-online"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Tableau Online

En este tutorial, aprenderá a integrar Tableau Online con Azure Active Directory (Azure AD). Al integrar Tableau Online con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Tableau Online.
* Permitir que los usuarios inicien sesión automáticamente en Tableau Online con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Tableau Online.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Tableau Online admite SSO iniciado por **SP**
* Una vez configurado Tableau Online, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-tableau-online-from-the-gallery"></a>Incorporación de Tableau Online desde la galería

Para configurar la integración de Tableau Online en Azure AD, será preciso que agregue Tableau Online desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Tableau Online** en el cuadro de búsqueda.
1. Seleccione **Tableau Online** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Tableau Online con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Tableau Online.

Para configurar y probar el inicio de sesión único de Azure AD con Tableau Online, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Tableau Online](#configure-tableau-online-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Tableau Online](#create-tableau-online-test-user)** , para tener un homólogo de B.Simon en Tableau Online que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Tableau Online, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Tableau Online**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información del dominio y las direcciones URL de inicio de sesión único de Tableau Online](common/sp-identifier.png)

    a. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://sso.online.tableau.com/public/sp/login?alias=<entityid>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL: `https://sso.online.tableau.com/public/sp/metadata?alias=<entityid>`.

    > [!NOTE]
    > El valor `<entityid>` lo obtendrá en la sección **Configuración de Tableau Online** de este tutorial. El valor del Id. de entidad será el valor de **Identificador de Azure AD** de la sección **Configuración de Tableau Online**.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Tableau Online** (Configurar Tableau Online), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

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
    Por ejemplo, BrittaSimon\@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Tableau Online.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Tableau Online**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Tableau Online**.

    ![Vínculo a Tableau Online en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-tableau-online-sso"></a>Configuración del inicio de sesión único de Tableau Online

1. En una ventana de explorador diferente, inicie sesión en su aplicación de Tableau Online como administrador. Vaya a **Settings** (Configuración) y luego a **Authentication** (Autenticación).

    ![Configurar inicio de sesión único](./media/tableauonline-tutorial/tutorial_tableauonline_09.png)

2. Para habilitar SAML, en la sección **Authentication types** (Tipos de autenticación) seleccione **Enable an additional authentication method** (Habilitar un método de autenticación adicional) y, después, seleccione la casilla **SAML**.

    ![Configurar inicio de sesión único](./media/tableauonline-tutorial/tutorial_tableauonline_12.png)

3. Desplácese hacia abajo hasta la sección **Import metadata file into Tableau Online** (Importar archivo de metadatos en Tableau Online).  Haga clic en Examinar e importe el archivo de metadatos que ha descargado desde Azure AD. A continuación, haga clic en **Aplicar**.

   ![Configurar inicio de sesión único](./media/tableauonline-tutorial/tutorial_tableauonline_13.png)

4. En la sección **Match assertions** (Aserciones de coincidencia), inserte el nombre de aserción del proveedor de identidades correspondiente para la **dirección de correo electrónico**, el **nombre** y los **apellidos**. Para obtener esta información a partir de Azure AD: 
  
    a. Vaya a la página de integración de aplicaciones de **Tableau Online** de Azure Portal.

    b. En la sección **User Attributes & Claims** (Atributos de usuario y notificaciones), haga clic en el icono de edición.

   ![Configurar inicio de sesión único](./media/tableauonline-tutorial/attributesection.png)

    c. Copie el valor del espacio de nombres de estos atributos: givenname, email y surname mediante los pasos siguientes:

   ![Inicio de sesión único de Azure AD](./media/tableauonline-tutorial/tutorial_tableauonline_10.png)

    d. Haga clic en el valor **user.givenname**

    e. Copie el valor del cuadro de texto **Namespace** (Espacio de nombres).

    ![Configurar inicio de sesión único](./media/tableauonline-tutorial/attributesection2.png)

    f. Para copiar los valores del espacio de nombres del correo electrónico y el apellido, repita los pasos anteriores.

    g. Abra la aplicación Tableau Online y en la sección **Tableau Online Attributes** (Atributos de Tableau Online) seleccione las siguientes opciones:

    * Correo electrónico: **mail** o **userprincipalname**

    * Nombre: **givenname**

    * Apellidos: **surname**

    ![Configurar inicio de sesión único](./media/tableauonline-tutorial/tutorial_tableauonline_14.png)

### <a name="create-tableau-online-test-user"></a>Creación de un usuario de prueba en Tableau Online

En esta sección, creará una usuaria llamada Britta Simon en Tableau Online.

1. En **Tableau Online**, haga clic en **Settings** (Configuración) y en la sección **Authentication** (Autenticación). Desplácese hacia abajo hasta la sección **Manage Users** (Administrar usuarios). Haga clic en **Add users** (Agregar usuarios) y, después, en **Enter Email Addresses** (Especificar direcciones de correo electrónico).
  
    ![Creación de un usuario de prueba de Azure AD](./media/tableauonline-tutorial/tutorial_tableauonline_15.png)

2. Seleccione **Add users for (SAML) authentication** [Agregar usuarios para la autenticación (SAML)]. En el cuadro de texto **Especificar direcciones de correo electrónico**, agregue britta.simon\@contoso.com.
  
    ![Creación de un usuario de prueba de Azure AD](./media/tableauonline-tutorial/tutorial_tableauonline_11.png)

3. Haga clic en **Agregar usuarios**.

### <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Tableau Online en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Tableau Online para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)