---
title: 'Tutorial: Integración de Azure Active Directory con Citrix ShareFile | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Citrix ShareFile.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/18/2020
ms.author: jeedes
ms.openlocfilehash: cfbb704799a1884c689bd0de547526a33f1ba7ce
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88651928"
---
# <a name="tutorial-azure-active-directory-integration-with-citrix-sharefile"></a>Tutorial: Integración de Azure Active Directory con Citrix ShareFile

En este tutorial, aprenderá a integrar Citrix ShareFile con Azure Active Directory (Azure AD).
La integración de Citrix ShareFile con Azure AD le proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a Citrix ShareFile.
* Puede permitir que los usuarios inicien sesión automáticamente en Citrix ShareFile (Inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Citrix ShareFile, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Citrix ShareFile

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Citrix ShareFile admite el inicio de sesión único iniciado por **SP**.
* Una vez que haya configurado Citrix ShareFile, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-citrix-sharefile-from-the-gallery"></a>Adición de Citrix ShareFile desde la galería

Para configurar la integración de Citrix ShareFile en Azure AD, deberá agregar Citrix ShareFile desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Citrix ShareFile** en el cuadro de búsqueda.
1. Seleccione **Citrix ShareFile** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Citrix ShareFile con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Citrix ShareFile.

Para configurar y probar el inicio de sesión único de Azure AD con Citrix ShareFile, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de Citrix ShareFile](#configure-citrix-sharefile-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de Citrix ShareFile](#create-citrix-sharefile-test-user)**: para tener un homólogo de Britta Simon en Citrix ShareFile que esté vinculado a la representación del usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Citrix NetScaler**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenant-name>.sharefile.com/saml/login`

    b. En el cuadro de texto **Identificador (id. de entidad)**, escriba una dirección URL con el siguiente patrón:

    - `https://<tenant-name>.sharefile.com`
    - `https://<tenant-name>.sharefile.com/saml/info`
    - `https://<tenant-name>.sharefile1.com/saml/info`
    - `https://<tenant-name>.sharefile1.eu/saml/info`
    - `https://<tenant-name>.sharefile.eu/saml/info`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: .
    
    - `https://<tenant-name>.sharefile.com/saml/acs`
    - `https://<tenant-name>.sharefile.eu/saml/<URL path>`
    - `https://<tenant-name>.sharefile.com/saml/<URL path>`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Citrix ShareFile](https://www.citrix.co.in/products/citrix-content-collaboration/support.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Citrix ShareFile** (Configurar Citrix ShareFile), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a conceder a B.Simon acceso a Citrix ShareFile utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Citrix ShareFile**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-citrix-sharefile-sso"></a>Configuración del inicio de sesión único de Citrix ShareFile

1. En otra ventana del explorador web, inicie sesión en el el sitio de su compañía de **Citrix ShareFile** como administrador.

1. En el menú del lado izquierdo, haga clic en el **icono de configuración** -> **Security (Seguridad)**  -> **Login & Security Policy (Inicio de sesión y directiva de seguridad)** .
   
    ![Account Administration (Administración de cuentas)](./media/sharefile-tutorial/settings-security.png "Administración de cuentas")

1. En la página de diálogo **Configuración de Inicio de sesión único/SAML 2.0**, en **Configuración básica**, siga estos pasos:
   
    ![Inicio de sesión único](./media/sharefile-tutorial/saml-configuration.png "Inicio de sesión único")
   
    a. Seleccione **YES** (SÍ) en **Enable SAML** (Habilitar SAML).

    b. Copie el valor de **ShareFile Issuer/ Entity ID** (Emisor/Id. de entidad de ShareFile) y péguelo en el cuadro **URL del identificador** del cuadro de diálogo **Configuración básica de SAML** de Azure Portal.
    
    c. En el cuadro de texto **Your IDP Issuer/Entity ID** (Emisor del IDP/Identificador de entidad), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    d. Haga clic en **Cambiar** junto al campo **Certificado X.509** y luego cargue el certificado que descargó desde Azure Portal.
    
    e. En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.
    
    f. En el cuadro de texto **URL de cierre de sesión**, pegue el valor de **Sign-Out URL** (Dirección URL de cierre de sesión) que copió de Azure Portal.

5. Haga clic en **Guardar** en el portal de administración de Citrix ShareFile.

## <a name="create-citrix-sharefile-test-user"></a>Creación de un usuario de prueba de Citrix ShareFile

1. Inicie sesión en el inquilino de **Citrix ShareFile**.

2. Haga clic en **People (Personas)**  -> **Manage Users Home (Administrar página de inicio de usuarios)**  -> **Create New Users (Crear nuevos usuarios)**  -> **Create Employee (Crear empleado)** .
   
    ![Create Employee (Crear empleado)](./media/sharefile-tutorial/create-user.png "Crear empleado")

3. En la sección **Basic Information** (Información básica), siga los siguientes pasos:
   
    ![Basic Information (Información básica)](./media/sharefile-tutorial/user-form.png "Información básica")
   
    a. En el cuadro de texto **First name** (Nombre), escriba el **nombre** del usuario, en este caso, **Britta**.
   
    b.  En el cuadro de texto **Last name** (Apellido), escriba el **apellido** del usuario, en este caso **Simon**.
   
    c. En el cuadro de texto **Email Address** (Dirección de correo electrónico), escriba la dirección de correo electrónico de Britta Simon, como **brittasimon\@contoso.com**.

4. Haga clic en **Agregar usuario**.
  
    >[!NOTE]
    >El titular de la cuenta de Azure AD recibirá un correo electrónico y seguirá un vínculo para confirmar la cuenta antes de que se active. Puede usar cualquier otra herramienta de creación de cuentas de usuario de Citrix ShareFile o API que Citrix ShareFile proporcione para aprovisionar las cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Citrix ShareFile en el panel de acceso, debería iniciar sesión automáticamente en la versión de Citrix ShareFile para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

