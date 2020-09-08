---
title: 'Tutorial: Integración de Azure Active Directory con InstaVR Viewer | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e InstaVR Viewer.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: ba3c504819eb30da8d21aa47aafc313d16eec51d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550388"
---
# <a name="tutorial-azure-active-directory-integration-with-instavr-viewer"></a>Tutorial: Integración de Azure Active Directory con InstaVR Viewer

En este tutorial, obtendrá información sobre cómo integrar InstaVR Viewer con Azure Active Directory (Azure AD).
La integración de InstaVR Viewer con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a InstaVR Viewer.
* Puede permitir que los usuarios inicien sesión automáticamente en InstaVR Viewer (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con InstaVR Viewer, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en InstaVR Viewer

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* InstaVR Viewer admite el inicio de sesión único iniciado por **SP**
* InstaVR Viewer admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-instavr-viewer-from-the-gallery"></a>Adición de InstaVR Viewer desde la galería

Para configurar la integración de InstaVR Viewer en Azure AD, deberá agregar InstaVR Viewer desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar InstaVR Viewer desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **InstaVR Viewer**, seleccione **InstaVR Viewer** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![InstaVR Viewer en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con InstaVR Viewer con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de InstaVR Viewer.

Para configurar y probar el inicio de sesión único de Azure AD con InstaVR Viewer, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de InstaVR Viewer](#configure-instavr-viewer-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en InstaVR Viewer](#create-instavr-viewer-test-user)** : para tener un homólogo de Britta Simon en InstaVR Viewer que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con InstaVR Viewer, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **InstaVR Viewer**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de InstaVR Viewer](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://console.instavr.co/auth/saml/login/<WEBPackagedURL>`

    > [!NOTE]
    > No hay ningún patrón fijo para URL de inicio de sesión. Se genera cuando el cliente de InstaVR Viewer lleva a cabo el empaquetado web. Es único para cada cliente y paquete. Para obtener la dirección URL de inicio de sesión exacta, necesita iniciar sesión en su instancia de InstaVR Viewer y llevar a cabo el empaquetado web.

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://console.instavr.co/auth/saml/sp/<WEBPackagedURL>`

    > [!NOTE]
    > El valor del identificador no es real. Actualícelo con el valor de identificador real, que se explica más adelante en este tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** y el **archivo de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadata-certificatebase64.png)

6. En la sección **Set up InstaVR Viewer** (Configurar InstaVR Viewer), copie las direcciones URL adecuadas según sus requisitos.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-instavr-viewer-single-sign-on"></a>Configuración de Inicio de sesión único de InstaVR Viewer

1. Abra una ventana del explorador web nueva e inicie sesión en el sitio de la compañía InstaVR Viewer como administrador.

2. Haga clic en **Icono de usuario** y seleccione **Cuenta**.

    ![Configuración de InstaVR Viewer](media/instavr-viewer-tutorial/tutorial-instavr-viewer-account.png)

3. Desplácese hacia abajo hasta la opción **SAML Auth** (Autenticación de SAML) y realice los pasos siguientes:

    ![Configuración de InstaVR Viewer](media/instavr-viewer-tutorial/tutorial-instavr-viewer-configure.png)

    a. En el cuadro de texto **URL de inicio de sesión único**, pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **URL de cierre de sesión**, pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    d. Para cargar el archivo de certificado descargado, haga clic en **Cargar**.

    e. Para cargar su archivo de federación descargado, haga clic en **Cargar**.

    f. Copie el valor **Id. de entidad** y péguelo en el cuadro de texto **Identificador (Id. de entidad)** de la sección **Configuración básica de SAML** en Azure Portal.

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

En esta sección, concederá acceso a Britta Simon a InstaVR Viewer para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** e **InstaVR Viewer**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **InstaVR Viewer**.

    ![Vínculo a InstaVR Viewer en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-instavr-viewer-test-user"></a>Creación de usuario de prueba de InstaVR Viewer

En esta sección, se crea un usuario llamado Britta Simon en InstaVR Viewer. InstaVR Viewer admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en InstaVR Viewer, se crea uno nuevo después de la autenticación. Si tiene algún problema, póngase en contacto con el [equipo de soporte técnico de InstaVR Viewer](mailto:contact@instavr.co).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

1. Abra una ventana del explorador web nueva e inicie sesión en el sitio de la compañía InstaVR Viewer como administrador.

2. Seleccione **Paquete** en el panel de navegación izquierdo y seleccione **Crear paquete para Web**.

    ![Configuración de InstaVR Viewer](media/instavr-viewer-tutorial/tutorial-instavr-viewer-testing1.png)

3. Seleccione **Descargar**.

    ![Configuración de InstaVR Viewer](media/instavr-viewer-tutorial/tutorial-instavr-viewer-testing2.png)

4. Seleccione **Abrir página hospedada**, tras lo cual se redirigirá a Azure AD para el inicio de sesión.

    ![Configuración de InstaVR Viewer](media/instavr-viewer-tutorial/tutorial-instavr-viewer-testing3.png)

5. Escriba sus credenciales de Azure AD para iniciar sesión correctamente en Azure AD a través de Inicio de sesión único.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
