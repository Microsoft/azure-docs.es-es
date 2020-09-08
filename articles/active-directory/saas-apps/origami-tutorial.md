---
title: 'Tutorial: Integración de Azure Active Directory con Origami | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Origami.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: 0fe7e0d43050120daa017669a96e90f96b873530
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543795"
---
# <a name="tutorial-azure-active-directory-integration-with-origami"></a>Tutorial: Integración de Azure Active Directory con Origami

En este tutorial, obtendrá información sobre cómo integrar Origami con Azure Active Directory (Azure AD).
Integrar Origami con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Origami.
* Puede permitir que los usuarios inicien sesión automáticamente en Origami (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Origami, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Origami

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Origami admite SSO iniciado por **SP**

## <a name="adding-origami-from-the-gallery"></a>Incorporación de Origami desde la galería

Para configurar la integración de Origami en Azure AD, deberá agregar Origami desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Origami desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Origami**, seleccione **Origami** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Origami en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Origami con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Origami.

Para configurar y probar el inicio de sesión único de Azure AD con Origami, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Origami](#configure-origami-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Origami](#create-origami-test-user)** : el objetivo es tener un homólogo de Britta Simon en Origami que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Origami, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Origami**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Origami](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://live.origamirisk.com/origami/account/login?account=<companyname>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de atención al cliente de Origami ](https://wordpress.org/support/theme/origami) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Origami** (Configurar Origami), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-origami-single-sign-on"></a>Configuración del inicio de sesión único en Origami

1. Inicie sesión en la cuenta de Origami con derechos de administrador.

2. En el menú de la parte superior, haga clic en **Administrador**.
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_51.png)

3. En la página del cuadro de diálogo Single Sign On Setup (Configuración de inicio de sesión único), siga estos pasos:
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_531.png)

    a. Seleccione **Enable Single Sign On**(Habilitar el inicio de sesión único).

    b. En el cuadro de texto **Identity Provider’s Sign-On URL** (Dirección URL de la página de inicio de sesión del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    c. En el cuadro de texto **Identity Provider’s Single Sign-out URL** (Dirección URL de la página de cierre de sesión del proveedor de identidades), pegue el valor de **Dirección URL de cierre de sesión**  que ha copiado de Azure Portal.

    d. Haga clic en **Browse** (Examinar) para cargar el certificado que ha descargado de Azure Portal.

    e. Haga clic en **Guardar cambios**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon@yourcompanydomain.extension**  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Origami.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y luego **Origami**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Origami**.

    ![Vínculo a Origami en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-origami-test-user"></a>Creación de un usuario de prueba en Origami

En esta sección, creará un usuario llamado Britta Simon en Origami. 

1. Inicie sesión en la cuenta de Origami con derechos de administrador.

2. En el menú de la parte superior, haga clic en **Administrador**.
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_51.png)

3. En el cuadro de diálogo **Users and Security** (Usuarios y seguridad), haga clic en **Users** (Usuarios).
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_54.png)

4. Haga clic en **Add New User**(Agregar nuevo usuario).
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_55.png)

5. En el cuadro de diálogo Add New User (Agregar nuevo usuario), realice los pasos siguientes:
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_56.png)

    a. En el cuadro de texto **User Name** (Nombre de usuario), escriba el correo electrónico del usuario; por ejemplo, **brittasimon\@contoso.com**.

    b. En el cuadro de texto **Password** (Contraseña), escriba una contraseña.

    c. En el cuadro de texto **Confirm Password** (Confirmar contraseña), escriba la contraseña de nuevo.

    d. En el cuadro de texto **First Name** (Nombre), escriba el nombre de usuario, en este caso **Britta**.

    e. En el cuadro de texto **Last Name** (Apellidos), escriba el apellido del usuario, en este caso **Simon**.

    f. Haga clic en **Save**(Guardar).
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_57.png)

6. Asigne **User Roles** (Roles de usuario) y **Client Access** (Acceso de cliente) al usuario. 
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/tutorial_origami_58.png)

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Origami en el Panel de acceso y debería iniciar sesión automáticamente en la versión de Origami para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

