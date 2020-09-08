---
title: 'Tutorial: Integración de Azure Active Directory con Sprinklr | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Sprinklr.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: 02b9adcb72b4d26a19b6514d19ce94a70a2415ed
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88534224"
---
# <a name="tutorial-azure-active-directory-integration-with-sprinklr"></a>Tutorial: Integración de Azure Active Directory con Sprinklr

En este tutorial, obtendrá información sobre cómo integrar Sprinklr con Azure Active Directory (Azure AD).
La integración de Sprinklr con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Sprinklr.
* Puede permitir que los usuarios inicien sesión automáticamente en Sprinklr (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Sprinklr, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Sprinklr.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Sprinklr admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-sprinklr-from-the-gallery"></a>Incorporación de Sprinklr desde la galería

Para configurar la integración de Sprinklr en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Sprinklr desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Sprinklr**, seleccione **Sprinklr** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Sprinklr en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Sprinklr con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Sprinklr.

Para configurar y probar el inicio de sesión único de Azure AD con Sprinklr, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Sprinklr](#configure-sprinklr-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Sprinklr](#create-sprinklr-test-user)** : para tener un homólogo de Britta Simon en Sprinklr vinculado a la representación del usuario de Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Sprinklr, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Sprinklr**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de inicio de sesión único de Sprinklr: dominio y direcciones URL](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.sprinklr.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<subdomain>.sprinklr.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Sprinklr](https://www.sprinklr.com/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Sprinklr** (Configurar Sprinklr), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-sprinklr-single-sign-on"></a>Configuración del inicio de sesión único de Sprinklr

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de Sprinklr como administrador.

1. Vaya a **Administración \> Configuración**.

    ![Administración](./media/sprinklr-tutorial/ic782907.png "Administración")

1. Vaya a **Administrar socios \> Inicio de sesión único** en el panel de la izquierda.

    ![Administrar asociado](./media/sprinklr-tutorial/ic782908.png "Administrar socio")

1. Haga clic en **+Add Single Sign On**(+ Agregar inicios de sesión únicos).

    ![Inicios de sesión únicos](./media/sprinklr-tutorial/ic782909.png "Inicios de sesión únicos")

1. Siga estos pasos en la página **Inicio de sesión único** :

    ![Inicios de sesión únicos](./media/sprinklr-tutorial/ic782910.png "Inicios de sesión únicos")

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración (por ejemplo, *WAADSSOTest*).

    b. Seleccione **Habilitado**.

    c. Seleccione **Use new SSO Certificate**(Usar el nuevo certificado de SSO).

    d. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de proveedor de identidades**.

    e. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    f. En el cuadro de texto **URL de inicio de sesión del proveedor de identidades**, pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    g. En el cuadro de texto **URL de cierre de sesión del proveedor de identidades**, pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    h. Como **Tipo de Id. de usuario de SAML**, seleccione **La aserción contiene el nombre de usuario de sprinklr.com del usuario**.

    i. Para **Ubicación de Id. de usuario de SAML**, seleccione **El Id. de usuario está en el elemento NameIdentifier de la instrucción Subject**.

    j. Haga clic en **Save**(Guardar).

    ![SAML](./media/sprinklr-tutorial/ic782911.png "SAML")

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Sprinklr.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Sprinklr**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Sprinklr**.

    ![Vínculo a Sprinklr en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-sprinklr-test-user"></a>Creación de un usuario de prueba de Sprinklr

1. Inicie sesión en su sitio de la compañía de Sprinklr como administrador.

1. Vaya a **Administración \> Configuración**.

    ![Administración](./media/sprinklr-tutorial/ic782907.png "Administración")

1. Vaya a **Administrar clientes \> Usuarios** en el panel de la izquierda.

    ![Configuración](./media/sprinklr-tutorial/ic782914.png "Configuración")

1. Haga clic en **Agregar usuario**.

    ![Configuración](./media/sprinklr-tutorial/ic782915.png "Configuración")

1. En el cuadro de diálogo **Edit user** (Editar usuario), realice los siguientes pasos:

    ![Editar usuario](./media/sprinklr-tutorial/ic782916.png "Edit user")

    a. En los cuadros de texto **Correo electrónico**, **Nombre** y **Apellido**, escriba la información de una cuenta de usuario de Azure AD que desee aprovisionar.

    b. Seleccione **Password Disabled**(Contraseña deshabilitada).

    c. Seleccione **Language** (Lenguaje).

    d. Seleccione **User Type**(Tipo de usuario).

    e. Haga clic en **Update**(Actualizar).

    > [!IMPORTANT]
    > **Password Disabled** (Contraseña deshabilitada) debe estar seleccionada para que los usuarios puedan iniciar sesión a través de un proveedor de identidades. 

1. Vaya a **Role**(Rol) y luego lleve a cabo los siguientes pasos:

    ![Roles de asociados](./media/sprinklr-tutorial/ic782917.png "Roles de socios")

    a. En la lista **Global**, seleccione **ALL_Permissions** (Todos los permisos).  

    b. Haga clic en **Update**(Actualizar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Sprinklr ofrecida por Sprinklr para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Sprinklr en el panel de acceso y debería iniciar sesión automáticamente en la versión de Sprinklr para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
