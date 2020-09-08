---
title: 'Tutorial: Integración de Azure Active Directory con Adobe Sign | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Adobe Sign.
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
ms.openlocfilehash: b7d1ea6c1ee21970a598855471284130160e1d65
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88537964"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-sign"></a>Tutorial: integración de Azure Active Directory con Adobe Sign

En este tutorial, obtendrá información sobre cómo integrar Adobe Sign con Azure Active Directory (Azure AD).
La integración de Adobe Sign con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a Adobe Sign.
* Puede permitir que los usuarios inicien sesión automáticamente en Adobe Sign (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Adobe Sign, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en Adobe Sign

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Adobe Sign admite el inicio de sesión único iniciado por **SP**

## <a name="adding-adobe-sign-from-the-gallery"></a>Agregar el inicio de sesión de Adobe Sign desde la galería

Para configurar la integración de Adobe Sign en Azure AD, será preciso que agregue Adobe Sign desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Adobe Sign desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Adobe Sign**, seleccione **Adobe Sign** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Adobe Sign en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Adobe Sign con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Adobe Sign.

Para configurar y probar el inicio de sesión único de Azure AD con Adobe Sign, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Adobe Sign](#configure-adobe-sign-single-sign-on)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Adobe Sign](#create-adobe-sign-test-user)** : para tener un homólogo de Britta Simon en Adobe Sign que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Adobe Sign, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones **Adobe Sign**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Adobe Sign](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.echosign.com/`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<companyname>.echosign.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte al cliente de Adobe Sign](https://helpx.adobe.com/in/contact/support.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar Adobe Sign**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-adobe-sign-single-sign-on"></a>Configuración del inicio de sesión único de Adobe Sign

1. Antes de la configuración, póngase en contacto con el [equipo de soporte técnico al cliente de Adobe Sign](https://helpx.adobe.com/in/contact/support.html) para incluir su dominio en la lista de permitidos de Adobe Sign. Aquí se muestra cómo agregar el dominio:

    a. El [equipo de soporte técnico al cliente de Adobe Sign](https://helpx.adobe.com/in/contact/support.html) le envía un token generado aleatoriamente. Para el dominio, el token será similar a: **adobe-sign-verification= xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx**

    b. Publique el token de comprobación en un registro de texto DNS y notifique al [equipo de soporte técnico al cliente de Adobe Sign](https://helpx.adobe.com/in/contact/support.html).
    
    > [!NOTE]
    > Esto puede tardar unos cuantos días, o más. Tenga en cuenta que los retrasos de propagación de DNS significan que un valor que se publica en DNS puede no estar visible durante una hora o más. Su administrador de TI debe tener conocimientos sobre cómo publicar este token en un registro de texto DNS.
    
    c. Cuando notifique al [equipo de soporte técnico al cliente de Adobe Sign](https://helpx.adobe.com/in/contact/support.html) a través del vale de soporte, después de publicado el token, se valida el dominio y se agrega a su cuenta.
    
    d. Por lo general, estos son los pasos necesarios para publicar el token en un registro DNS:

    * Inicie sesión en su cuenta de dominio
    * Busque la página para actualizar el registro de DNS. Esta página se puede llamar Administración de DNS, Administración del servidor de nombres o Configuración avanzada.
    * Busque los registros TXT para el dominio.
    * Agregue un registro TXT con el valor completo del token proporcionado por Adobe.
    * Guarde los cambios.

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Adobe Sign como administrador.

1. En el menú SAML, seleccione **Configuración de cuenta** > **Configuración de SAML**.
   
    ![Captura de pantalla de la página Adobe Sign SAML Settings (Configuración de SAML de Adobe Sign)](./media/adobe-echosign-tutorial/ic789520.png "Cuenta")

1. En la sección **SAML Settings** (Configuración de SAML), realice los pasos siguientes:
  
   ![Captura de pantalla de SAML Settings (Configuración de SAML)](./media/adobe-echosign-tutorial/ic789521.png "Configuración de SAML")
   
   ![Captura de pantalla de SAML Settings (Configuración de SAML)](./media/adobe-echosign-tutorial/ic789522.png "Configuración de SAML")

   a. En **SAML Mode** (Modo de SAML), seleccione **SAML Mandatory** (SAML obligatorio).
   
   b. Seleccione **Allow EchoSign Account Administrators to log in using their EchoSign Credentials**(Permitir a los administradores de cuentas de EchoSign iniciar sesión con sus credenciales de EchoSign).
   
   c. En **User Creation** (Creación de usuario), seleccione **Automatically add users authenticated through SAML** (Agregar automáticamente usuarios autenticados a través de SAML).

   d. Pegue el **Identificador de Azure AD** que copió de Azure Portal en el cuadro de texto **Idp Entity ID** (Identificador de la entidad de IdP).
    
   e. Pegue la **URL de inicio de sesión** que ha copiado de Azure Portal en el cuadro de texto **Idp Login URL** (URL de inicio de sesión de IdP).
   
   f. Pegue la **URL de cierre de sesión** que ha copiado de Azure Portal en el cuadro de texto **Idp Logout URL** (URL de cierre de sesión de IdP).

   g. Abra el archivo **Certificado (Base64)** en el Bloc de notas. Copie el contenido del mismo en el Portapapeles y, después, péguelo en el cuadro de texto **IdP Certificate** (Certificado IDP).

   h. Seleccione **Save changes** (Guardar los cambios).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Adobe Sign.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**,  **Adobe Sign**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Adobe Sign**.

    ![El enlace de Adobe Sign en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-adobe-sign-test-user"></a>Creación de un usuario de prueba de Adobe Sign

Para permitir que los usuarios de Azure AD inicien sesión en Adobe Sign, tienen que aprovisionarse en esta aplicación. Se trata de una tarea manual.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Adobe Sign que esta le ofrezca para aprovisionar cuentas de usuario de Azure AD. 

1. Inicie sesión en el sitio de la compañía de **Adobe Sign** como administrador.

2. En el menú de la parte superior, seleccione **Account** (Cuenta). A continuación, en el panel izquierdo, seleccione **Users & Groups** (Usuarios y grupos) > **Create a new user** (Crear un usuario nuevo).
   
    ![Captura de pantalla del sitio empresarial de Adobe Sign, con Account (Cuenta), Users & Groups (Usuarios y grupos) y Create a new user (Crear nuevo usuario) resaltados](./media/adobe-echosign-tutorial/ic789524.png "Cuenta")
   
3. En la sección **Create New User** (Crear nuevo usuario), lleve a cabo estos pasos:
   
    ![Captura de pantalla de la sección Create New User (Crear nuevo usuario)](./media/adobe-echosign-tutorial/ic789525.png "Crear usuario")
   
    a. Escriba en los campos de texto pertinentes los datos de **Email Address** (Dirección de correo electrónico), **Name** (Nombre) y **Last Name** (Apellidos) de la cuenta de correo válida de Azure AD que desea aprovisionar.
   
    b. Seleccione **Create User** (Crear usuario).

>[!NOTE]
>El titular de la cuenta de Azure Active Directory recibe un mensaje de correo electrónico con un vínculo para confirmar la cuenta antes de que se active. 

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Adobe Sign en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Adobe Sign para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

