---
title: 'Tutorial: Integración de Azure Active Directory con BlueJeans | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y BlueJeans.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: dfc634fd-1b55-4ba8-94a8-b8288429b6a9
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 39e9f52948d035c72a6a019558915d8c92ceebeb
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65463518"
---
# <a name="tutorial-azure-active-directory-integration-with-bluejeans"></a>Tutorial: Integración de Azure Active Directory con BlueJeans

En este tutorial, obtendrá información sobre cómo integrar BlueJeans con Azure Active Directory (Azure AD).
La integración de BlueJeans con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a BlueJeans.
* Puede permitir que los usuarios inicien sesión automáticamente en BlueJeans (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con BlueJeans, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Suscripción habilitada para el inicio de sesión único en BlueJeans

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* BlueJeans admite el inicio de sesión único iniciado por **SP**

* BlueJeans admite el [**aprovisionamiento** automático de usuarios](bluejeans-provisioning-tutorial.md)

## <a name="adding-bluejeans-from-the-gallery"></a>Incorporación de BlueJeans desde la galería

Para configurar la integración de BlueJeans en Azure AD, será preciso que agregue BlueJeans desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar BlueJeans desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **BlueJeans**, seleccione **BlueJeans** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![BlueJeans en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con BlueJeans con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de BlueJeans.

Para configurar y probar el inicio de sesión único de Azure AD con BlueJeans, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)**: para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de BlueJeans](#configure-bluejeans-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**, para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**, para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de BlueJeans](#create-bluejeans-test-user)**: para tener un homólogo de Britta Simon en BlueJeans que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)**: para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con BlueJeans, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **BlueJeans**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](media/bluejeans-tutorial/edit-urls-bluejeans.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, escriba los valores siguientes:

    ![Información de dominio y direcciones URL de inicio de sesión único de BlueJeans](media/bluejeans-tutorial/tutorial_bluejeans-basic-configuration.png)

   - En el cuadro de texto **Identificador**, escriba lo siguiente: `https://samlsp.bluejeans.com`
    
   - En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL de la página de aterrizaje suministrada por BlueJeans (para obtener este valor, puede ponerse en contacto con el [equipo de soporte técnico de BlueJeans](https://support.bluejeans.com/contact)): `https://<companyname>.bluejeans.com`
    
   - Haga clic en **Save**(Guardar).

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up BlueJeans** (Configurar BlueJeans), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-bluejeans-single-sign-on"></a>Configuración del inicio de sesión único de BlueJeans

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la empresa de **BlueJeans**.

2. Vaya a **ADMIN \> CONFIGURACIÓN DE GRUPO \> SEGURIDAD**.

    ![Administración](./media/bluejeans-tutorial/ic785868.png "Administración")

3. En la sección **SEGURIDAD**, realice estos pasos:

    ![Inicio de sesión único de SAML](./media/bluejeans-tutorial/ic785869.png "Inicio de sesión único de SAML")

     a. Seleccione **SAML Single Sign On**(Inicio de sesión único de SAML).

    b. Seleccione **Enable automatic provisioning**(Habilitar aprovisionamiento automático).

4. Continúe con los siguiente pasos:

    ![Ruta del certificado](./media/bluejeans-tutorial/ic785870.png "Ruta del certificado")

     a. Haga clic en **Elegir archivo** para cargar el certificado codificado en base 64 que descargó de Azure Portal.

    b. En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Password Change URL** (Dirección URL de cambio de contraseña), pegue el valor de **Cambiar dirección URL de contraseña** que copió de Azure Portal.

    d. En el cuadro de texto **URL de cierre de sesión**, pegue el valor de **Sign-Out URL** (Dirección URL de cierre de sesión) que copió de Azure Portal.

5. Continúe con los siguiente pasos:

    ![Guardar cambios](./media/bluejeans-tutorial/ic785874.png "Guardar cambios")

     a. En el cuadro de texto **Id. de usuario**, escriba `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    b. En el cuadro de texto **Correo electrónico**, escriba `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    c. Haga clic en **GURDAR CAMBIOS**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon\@yourcompanydomain.extension`. Por ejemplo, BrittaSimon@contoso.com.

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a BlueJeans.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **BlueJeans**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **BlueJeans**.

    ![Vínculo a BlueJeans en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-bluejeans-test-user"></a>Creación de un usuario de prueba de BlueJeans

El objetivo de esta sección es crear un usuario llamado Britta Simon en BlueJeans. BlueJeans admite el aprovisionamiento automático de usuarios, que está habilitado de forma predeterminada. [Aquí](bluejeans-provisioning-tutorial.md) puede encontrar más información sobre cómo configurar el aprovisionamiento automático de usuarios.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. Inicie sesión como administrador en el sitio de la compañía de **BlueJeans**.

2. Vaya a **ADMIN \> ADMINISTRAR USUARIOS \> AGREGAR USUARIO**.

    ![Administración](./media/bluejeans-tutorial/ic785877.png "Administración")

    > [!IMPORTANT]
    > La pestaña **AGREGAR USUARIO** está disponible solo si en la pestaña **SEGURIDAD**, está desactivada la opción **Habilitar aprovisionamiento automático**.

3. En la sección **AGREGAR USUARIO**, lleve a cabo estos pasos:

    ![Agregar usuario](./media/bluejeans-tutorial/ic785886.png "Agregar usuario")

     a. En el cuadro de texto **First Name** (Nombre), escriba el nombre de usuario, en este caso **Britta**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

    c. En el cuadro de texto **Elegir un nombre de usuario de BlueJeans**, escriba el nombre de usuario del usuario, como **Brittasimon**

    d. En el cuadro de texto **Crear una contraseña**, escriba su contraseña.

    e. En el cuadro de texto **Compañía**, escriba su compañía.

    f. En el cuadro de texto **Dirección de correo electrónico**, escriba el correo electrónico del usuario; por ejemplo, `brittasimon\@contoso.com`.

    g. En el cuadro de texto **Create a BlueJeans Meeting I.D** (Crear un id. de reunión de BlueJeans), escriba el identificador de la reunión.

    h. En el cuadro de texto **Pick a Moderator Passcode** (Elegir un código de acceso de moderador), escriba el código de acceso.

    i. Haga clic en **CONTINUE** (Continuar).

    ![Agregar usuario](./media/bluejeans-tutorial/ic785887.png "Agregar usuario")

    J. Haga clic en **ADD USER** (Agregar usuario).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de BlueJeans que proporcione BlueJeans para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de BlueJeans en el panel de acceso y debería iniciar sesión automáticamente en la versión de BlueJeans para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
