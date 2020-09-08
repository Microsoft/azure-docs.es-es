---
title: 'Tutorial: Integración de Azure Active Directory con Work.com | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Work.com.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: 5fe8697327203a84f91b42e8742db75150b6cb19
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88541468"
---
# <a name="tutorial-azure-active-directory-integration-with-workcom"></a>Tutorial: Integración de Azure Active Directory con Work.com

En este tutorial, obtendrá información sobre cómo integrar Work.com con Azure Active Directory (Azure AD).
La integración de Work.com con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Work.com.
* Puede permitir que los usuarios inicien sesión automáticamente en Work.com (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Work.com, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Work.com

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Work.com admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-workcom-from-the-gallery"></a>Agregar Work.com desde la galería

Para configurar la integración de Work.com en Azure AD, será preciso que agregue Work.com desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Work.com desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Work.com**, seleccione **Work.com** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Work.com en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Work.com con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Work.com.

Para configurar y probar el inicio de sesión único de Azure AD con Work.com, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Work.com](#configure-workcom-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Work.com](#create-workcom-test-user)** : para tener un homólogo de Britta Simon en Work.com que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

>[!NOTE]
>Para configurar el inicio de sesión único, deberá configurar todavía un nombre de dominio personalizado de Work.com. Deberá definir al menos un nombre de dominio, probar su nombre de dominio e implementarlo en toda la organización.

Para configurar el inicio de sesión único de Azure AD con Work.com, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Work.com**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Work.com](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `http://<companyname>.my.salesforce.com`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte al cliente de Work.com](https://help.salesforce.com/articleView?id=000159855&type=3) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Work.com** (Configurar Work.com), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-workcom-single-sign-on"></a>Configuración del inicio de sesión único de Work.com

1. Inicie sesión en su inquilino de Work.com como administrador.

2. Acceda a **Setup**(Configuración).
   
    ![Configuración](./media/work-com-tutorial/ic794108.png "Configurar")

3. En el panel de navegación izquierdo, en la sección **Administer** (Administrar), haga clic en **Domain Management** (Administración de dominios) para expandir la sección relacionada y, luego, haga clic en **My Domain** (Mi dominio) para abrir la página **My Domain** (Mi dominio). 
   
    ![Mi dominio](./media/work-com-tutorial/ic767825.png "Mi dominio")

4. Para comprobar que el dominio se configuró correctamente, asegúrese de que está en "**Step 4 Deployed to Users**" (Paso 4 Dominio implementado para usuarios) y revise la sección "**My Domain Settings**" (Mi configuración de dominio).
   
    ![Dominio implementado al usuario](./media/work-com-tutorial/ic784377.png "Dominio implementado al usuario")

5. Inicie sesión en su inquilino de Work.com.

6. Acceda a **Setup**(Configuración).
    
    ![Configuración](./media/work-com-tutorial/ic794108.png "Configurar")

7. Expanda el menú **Security Controls** (Controles de seguridad) y luego haga clic en **Single Sign-On Settings** (Configuración de inicio de sesión único).
    
    ![Configuración de inicio de sesión único](./media/work-com-tutorial/ic794113.png "Configuración de inicio de sesión único")

8. En la página del cuadro de diálogo **Single Sign-On Settings** (Configuración de inicio de sesión único), siga estos pasos:
    
    ![SAML habilitado](./media/work-com-tutorial/ic781026.png "SAML habilitado")
    
    a. Seleccione **SAML habilitado**.
    
    b. Haga clic en **Nueva**.

9. En la sección **SAML Single Sign-On Settings** (Configuración del inicio de sesión único de SAML), siga estos pasos:
    
    ![Configuración del inicio de sesión único de SAML](./media/work-com-tutorial/ic794114.png "Configuración de inicio de sesión único SAML")
    
    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración.  
       
    > [!NOTE]
    > Si se proporciona un valor para **Name** (Nombre), el cuadro de texto **API Name** (Nombre de API) se completa automáticamente.
    
    b. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.
    
    c. Para cargar el certificado descargado de Azure Portal, haga clic en **Browse** (Examinar).
    
    d. En el cuadro de texto **Id. de identidad**, escriba `https://salesforce-work.com`.
    
    e. Como **SAML Identity Type** (Tipo de identidad SAML), seleccione **Assertion contains the Federation ID from the User object** (La aserción contiene el id. de federación del objeto Usuario).
    
    f. Para **///SAML Identity Location** (Ubicación de identidad SAML), seleccione **///Identity is in the NameIdentfier element of the Subject statement** (La identidad está en el elemento NameIdentifier de la instrucción Subject).
    
    g. En el cuadro de texto **Identity Provider Login URL** (Dirección URL de inicio de sesión del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    h. En el cuadro de texto **Identity Provider Logout URL** (Dirección URL de cierre de sesión del proveedor de identidades), pegue el valor de **Dirección URL de cierre de sesión**  que ha copiado de Azure Portal.
    
    i. Para **Service Provider Initiated Request Binding** (Enlace de solicitud iniciada por el proveedor de servicio), seleccione **HTTP Post** (Método HTTP Post).
    
    j. Haga clic en **Save**(Guardar).

10. En el portal de Work.com, en el panel de navegación izquierdo, haga clic en **Domain Management** (Administración de dominios) para expandir la sección relacionada y luego haga clic en la página **My Domain** (Mi dominio) para abrir la página **My Domain** (Mi dominio). 
    
    ![Mi dominio](./media/work-com-tutorial/ic794115.png "Mi dominio")

11. En la página **My domain** (Mi dominio), en la sección **Login Page Branding** (Personalización de marca de la página de inicio de sesión), haga clic en **Edit** (Editar).
    
    ![Personalización de marca de la página de inicio de sesión](./media/work-com-tutorial/ic767826.png "Personalización de marca de la página de inicio de sesión")

12. En la página **Login Page Branding** (Personalización de marca de la página de inicio de sesión), en la sección **Authentication Service** (Servicio de autenticación), se muestra el nombre de su **SAML SSO Settings** (Configuración de SSO de SAML). Selecciónelo y luego haga clic en **Save**(Guardar).
    
    ![Personalización de marca de la página de inicio de sesión](./media/work-com-tutorial/ic784366.png "Personalización de marca de la página de inicio de sesión")

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Work.com.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**,  **Work.com**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Work.com**.

    ![Vínculo a Work.com en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-workcom-test-user"></a>Creación de un usuario de prueba de Work.com

Para que los usuarios de Azure Active Directory puedan iniciar sesión, deben aprovisionarse a Work.com. En el caso de Work.com, el aprovisionamiento es una tarea manual.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Siga estos pasos para configurar el aprovisionamiento de usuario:

1. Inicie sesión en su sitio de la compañía de Work.com como administrador.

2. Acceda a **Setup**(Configuración).
   
    ![Configuración](./media/work-com-tutorial/IC794108.png "Configurar")

3. Vaya a **Manage Users \> (Administrar usuarios) Users** (Usuarios).
   
    ![Administración de usuarios](./media/work-com-tutorial/IC784369.png "Administrar usuarios")

4. Haga clic en **Nuevo usuario**.
   
    ![Todos los usuarios](./media/work-com-tutorial/IC794117.png "Todos los usuarios")

5. En la sección Edición de usuarios, realice los pasos siguientes, en los atributos de una cuenta de Azure AD válida que desee aprovisionar en los cuadros de texto relacionados:
   
    ![Edición de usuarios](./media/work-com-tutorial/ic794118.png "Edición de usuarios")
   
    a. En el cuadro de texto **Nombre**, escriba el **nombre** del usuario **Britta**.
    
    b. En el cuadro de texto **Apellido**, escriba el **apellido** del usuario **Simon**.
    
    c. En el cuadro de texto **Alias**, escriba el **nombre** del usuario **Britta**.
    
    d. En el cuadro de texto **Correo electrónico**, escriba la **dirección de correo electrónico** del usuario, Brittasimon@contoso.com.
    
    e. En el cuadro de texto **Nombre de usuario**, escriba un nombre de usuario como Brittasimon@contoso.com.
    
    f. En el cuadro de texto **Sobrenombre**, escriba un **sobrenombre** del usuario, como **Simon**.
    
    g. Seleccione **Role** (Rol), **User License** (Licencia de usuario) y **Profile** (Perfil).
    
    h. Haga clic en **Save**(Guardar).  
      
    > [!NOTE]
    > El titular de la cuenta de Azure AD recibirá un mensaje de correo electrónico con un vínculo para confirmar la cuenta antes de que se active.
    > 

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Work.com en el panel de acceso y debería iniciar sesión automáticamente en la versión de Work.com para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

