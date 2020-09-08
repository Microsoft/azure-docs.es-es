---
title: 'Tutorial: Integración de Azure Active Directory con Zoho One | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zoho One.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.openlocfilehash: baa33e79b3bba8eccb2d0fc03e110e1a3c4defde
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546013"
---
# <a name="tutorial-azure-active-directory-integration-with-zoho-one"></a>Tutorial: Integración de Azure Active Directory con Zoho One

En este tutorial se aprende a integrar Zoho One con Azure Active Directory (Azure AD).
La integración de Zoho One con Azure AD proporciona las siguientes ventajas:

* En Azure AD, puede controlar quién tiene acceso a Zoho One.
* Puede permitir que los usuarios inicien sesión automáticamente en Zoho One (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Zoho One, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para inicio de sesión único en Zoho One

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zoho One admite el inicio de sesión único iniciado por **SP** e **IDP**.

## <a name="adding-zoho-one-from-the-gallery"></a>Incorporación de Zoho One desde la galería

Para configurar la integración de Zoho One en Azure AD, deberá agregar Zoho One desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Zoho One desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zoho One**, seleccione **Zoho One** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Zoho One en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Zoho One con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Zoho One.

Para configurar y probar el inicio de sesión único de Azure AD con Zoho One, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Zoho One](#configure-zoho-one-single-sign-on)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Zoho One](#create-zoho-one-test-user)** : para tener un homólogo de Britta Simon en Zoho One que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Zoho One, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Zoho One**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información de inicio de sesión único de Dominio y direcciones URL de Zoho One](common/idp-relay.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL: `one.zoho.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://accounts.zoho.com/samlresponse/<saml-identifier>`

    > [!NOTE]
    > El valor anterior de **URL de respuesta** no es real. El valor de `<saml-identifier>` lo obtendrá del paso 4 de la sección **Configuración del inicio de sesión único en Zoho One**, que se explica más adelante en el tutorial.

    c. Haga clic en **Establecer direcciones URL adicionales**.

    d. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL: `https://one.zoho.com`

5. Si desea configurar la aplicación en modo iniciado por **SP**, realice el siguientes paso:


    ![Información de inicio de sesión único de Dominio y direcciones URL de Zoho One](common/both-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://accounts.zoho.com/samlauthrequest/<domain_name>?serviceurl=https://one.zoho.com` 

    > [!NOTE] 
    > El valor de **URL de inicio de sesión** anterior no es real. Lo actualizará con la dirección URL de inicio de sesión real de la sección **Configuración del inicio de sesión único en Zoho One**, que se explica más adelante en el tutorial. 

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Configurar Zoho One**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-zoho-one-single-sign-on"></a>Configuración del inicio de sesión único en Zoho One

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa de Zoho One como administrador.

2. En la pestaña **Organización**, haga clic en **Configuración** en **Autenticación SAML**.

    ![Org. Zoho One](./media/zohoone-tutorial/tutorial_zohoone_setup.png)

3. En la página emergente, realice los pasos siguientes:

    ![Ini. Zoho One](./media/zohoone-tutorial/tutorial_zohoone_save.png)

    a. En el cuadro de texto **Sign-in URL** (Dirección URL de inicio de sesión), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

    b. En el cuadro de texto **Sign-out URL** (URL de cierre de sesión), pegue el valor de **Logout Out URL** (URL de cierre de sesión) que copió de Azure Portal.

    c. Haga clic en **Examinar** para cargar el **Certificado (Base64)** que ha descargado de Azure Portal.

    d. Haga clic en **Save**(Guardar).

4. Después de guardar la configuración de autenticación de SAML, copie el valor de **SAML-Identifier** (Identificador de SAML) y anéxelo a la **dirección URL de respuesta** en lugar de `<saml-identifier>`, por ejemplo, `https://accounts.zoho.com/samlresponse/one.zoho.com` y pegue el valor generado en el cuadro de texto **URL de respuesta** en la sección **Basic SAML Configuration** (Configuración básica de SAML).

    ![SAML de Zoho One](./media/zohoone-tutorial/tutorial_zohoone_samlidenti.png)

5. Vaya a la pestaña **Dominios** y haga clic en **Agregar dominio**.

    ![Dominio de Zoho One](./media/zohoone-tutorial/tutorial_zohoone_domain.png)

6. En la página **Agregar dominio**, siga estos pasos:

    ![Agregar dominio de Zoho One](./media/zohoone-tutorial/tutorial_zohoone_adddomain.png)

    a. En el cuadro de texto **Domain Name** (Nombre de dominio), escriba el dominio, por ejemplo, contoso.com.

    b. Haga clic en **Agregar**.

    >[!Note]
    >Después de agregar el dominio, siga [estos pasos](https://www.zoho.com/one/help/admin-guide/domain-verification.html) para comprobar el dominio. Una vez comprobado el dominio, use el nombre de dominio de **Dirección URL de inicio de sesión** de la sección **Basic SAML Configuration** (Configuración básica de SAML) de Azure Portal.

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

En esta sección se habilita a Britta Simon para que use el inicio de sesión único de Azure al concederle acceso a Zoho One.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **Zoho One**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Zoho One**.

    ![Vínculo de Zoho One en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-zoho-one-test-user"></a>Creación de un usuario de prueba de Zoho One

Para permitir que los usuarios de Azure AD inicien sesión en Zoho One, deben aprovisionarse en Zoho One. En Zoho One, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Zoho One como administrador de seguridad.

2. En la pestaña **Usuarios**, haga clic en **Logotipo de usuario**.

    ![Usuario de Zoho One](./media/zohoone-tutorial/tutorial_zohoone_users.png)

3. En la página **Add User** (Agregar usuario), siga estos pasos:

    ![Agregar usuario de Zoho One](./media/zohoone-tutorial/tutorial_zohoone_adduser.png)
    
    a. En el cuadro de texto **Nombre**, escriba el nombre de un usuario, por ejemplo, **Britta Simon**.
    
    b. En el cuadro de texto **Dirección de correo electrónico**, escriba el correo electrónico del usuario; por ejemplo, brittasimon@contoso.com.

    >[!Note]
    >Seleccione el dominio verificado en la lista de dominios.

    c. Haga clic en **Agregar**.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Zoho One en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Zoho One para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

