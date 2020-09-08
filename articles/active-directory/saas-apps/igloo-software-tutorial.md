---
title: 'Tutorial: Integración de Azure Active Directory con Igloo Software | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e Igloo Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/06/2019
ms.author: jeedes
ms.openlocfilehash: 857869b73e76c76075ac9607b64ffbb9d7358b5b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545296"
---
# <a name="tutorial-azure-active-directory-integration-with-igloo-software"></a>Tutorial: Integración de Azure Active Directory con Igloo Software

En este tutorial, obtendrá información sobre cómo integrar Igloo Software con Azure Active Directory (Azure AD).
La integración de Igloo Software con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Igloo Software.
* Puede permitir que los usuarios inicien sesión automáticamente en Igloo Software (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Igloo Software, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Igloo Software

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Igloo Software admite el inicio de sesión único iniciado por **SP**
* Igloo Software admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-igloo-software-from-the-gallery"></a>Agregar Igloo Software desde la galería

Para configurar la integración de Igloo Software en Azure AD, deberá agregar Igloo Software desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Igloo Software desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Igloo Software**, seleccione **Igloo Software** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Igloo Software en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Igloo Software con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Igloo Software.

Para configurar y probar el inicio de sesión único de Azure AD con Igloo Software, debe completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Igloo Software](#configure-igloo-software-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en Igloo Software](#create-igloo-software-test-user)** : para tener un homólogo de Britta Simon en Igloo Software vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Igloo Software, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Igloo Software**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Igloo Software](common/sp-identifier-reply.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<company name>.igloocommmunities.com`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<company name>.igloocommmunities.com/saml.digest`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<company name>.igloocommmunities.com/saml.digest`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con el [equipo de soporte de cliente de Igloo Software](https://www.igloosoftware.com/services/support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Igloo Software** (Configurar Igloo Software), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-igloo-software-single-sign-on"></a>Configuración del inicio de sesión único en Igloo Software

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de Igloo Software de la compañía.

2. Vaya a **Control Panel**(Panel de control).

     ![Control Panel (Panel de control)](./media/igloo-software-tutorial/ic799949.png "Panel de control")

3. En la pestaña **Membership** (Pertenencia), haga clic en **Sign In Settings** (Configuración de inicio de sesión).

    ![Configuración de inicio de sesión](./media/igloo-software-tutorial/ic783968.png "Sign In Settings")

4. En la sección SAML Configuration (Configuración de SAML), haga clic en **Configure SAML Authentication**(Configurar la autenticación de SAML).

    ![SAML Configuration (Configuración de SAML)](./media/igloo-software-tutorial/ic783969.png "Configuración de SAML")

5. En la sección de **General Configuration** (Configuración general), realice los pasos siguientes:

    ![Configuración general](./media/igloo-software-tutorial/ic783970.png "General Configuration")

    a. En el cuadro de texto **Connection Name** (Nombre de conexión), escriba el nombre personalizado de la configuración.

    b. En el cuadro de texto **IdP Login URL** (URL de inicio de sesión del IdP), pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **IdP Logout URL** (URL de cierre de sesión del IdP), pegue el valor de la **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    d. Seleccione **Logout Response and Request HTTP Type** (Respuesta de cierre de sesión y tipo de solicitud HTTP) como **POST**.

    e. Abra el certificado codificado en **base 64** descargado de Azure Portal en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado público**.

6. En **Response and Authentication Configuration**(Configuración de autenticación y respuesta), realice los pasos siguientes:

    ![Configuración de la respuesta y la autenticación](./media/igloo-software-tutorial/IC783971.png "Response and Authentication Configuration")
  
    a. En **Identity Provider** (Proveedor de identidades), seleccione **Microsoft ADFS**.

    b. En **Identifier Type** (Tipo de identificador), seleccione **Email Address** (Dirección de correo electrónico). 

    c. En el cuadro de texto **Atributo de correo electrónico**, escriba **emailaddress**.

    d. En el cuadro de texto **First Name Attribute** (Atributo de nombre), escriba **givenname**.

    e. En el cuadro de texto **Last Name Attribute** (Atributo de apellido), escriba **surname**.

7. Realice los pasos siguientes para completar la configuración:

    ![Creación de usuario al inicio de sesión](./media/igloo-software-tutorial/IC783972.png "Creación de usuario al inicio de sesión") 

    a. En **User creation on Sign in** (Creación de usuario al inicio de sesión), seleccione **Create a new user in your site when they sign in** (Crear un nuevo usuario en el sitio cuando se inicia sesión).

    b. En **Sign in Settings** (Configuración de inicio de sesión), seleccione **Use SAML button on "Sign in" screen** (Usar botón SAML en la pantalla "Iniciar sesión").

    c. Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Igloo Software.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Igloo Software**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Igloo Software**.

    ![Vínculo a Igloo Software en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-igloo-software-test-user"></a>Creación del usuario de prueba en Igloo Software

No hay ningún elemento de acción para que configure el aprovisionamiento de usuarios para Igloo Software.  

Cuando un usuario asignado intenta iniciar sesión en Igloo Software desde el panel de acceso, Igloo Software comprueba si el usuario existe.  Si no hay cuentas de usuario disponibles, Igloo Software crea una automáticamente.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Igloo Software en el panel de acceso, debería iniciar sesión automáticamente en la versión de Igloo Software para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)