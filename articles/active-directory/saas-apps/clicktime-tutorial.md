---
title: 'Tutorial: integración de Azure Active Directory con ClickTime | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y ClickTime.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2019
ms.author: jeedes
ms.openlocfilehash: ed39064d4aee07554ce795be73732024e1b47f84
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549347"
---
# <a name="tutorial-azure-active-directory-integration-with-clicktime"></a>Tutorial: integración de Azure Active Directory con ClickTime

En este tutorial, aprenderá a integrar ClickTime con Azure Active Directory (Azure AD).
La integración de ClickTime con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a ClickTime.
* Puede permitir que los usuarios inicien sesión automáticamente en ClickTime (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con ClickTime, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en ClickTime

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ClickTime admite SSO iniciado por **IDP**

## <a name="adding-clicktime-from-the-gallery"></a>Agregar ClickTime desde la galería

Para configurar la integración de ClickTime en Azure AD, debe agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar ClickTime desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **ClickTime**, seleccione **ClickTime** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![ClickTime en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con ClickTime con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de ClickTime.

Para configurar y probar el inicio de sesión único de Azure AD con ADP ClickTime, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración de inicio de sesión único de ClickTime](#configure-clicktime-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de ClickTime](#create-clicktime-test-user)** : para tener un homólogo de Britta Simon en ClickTime que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con ClickTime, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ClickTime**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de ClickTime](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL: `https://app.clicktime.com/sp/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:
    
    ```http
    https://app.clicktime.com/Login/
    https://app.clicktime.com/App/Login/Consume.aspx
    ```

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar ClickTime**, copie las direcciones URL adecuadas según sus requisitos.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-clicktime-single-sign-on"></a>Configuración de inicio de sesión único de ClickTime

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de ClickTime como administrador.

1. En la barra de herramientas de la parte superior, haga clic en**Preferencias** y luego haga clic en **Configuración de seguridad**.

1. En la sección de configuración **Preferencias de inicio de sesión único** , siga estos pasos:
   
    ![Security Settings (Configuración de seguridad)](./media/clicktime-tutorial/tic777280.png "Configuración de seguridad")
   
    a.  Seleccione **Allow** sign-in using Single Sign-On (SSO) with **Azure AD** [Permitir inicio de sesión mediante inicio de sesión único (SSO) con Azure AD].
   
    b. En el cuadro de texto **Identity Provider Endpoint** (Punto de conexión del proveedor de identidades), pegue el valor de la **Dirección URL de inicio de sesión** que copió desde Azure Portal.
   
    c.  Abra el **certificado codificado en Base 64** que descargó de Azure Portal en el **Bloc de notas**, copie el contenido y, luego, péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509).
   
    d.  Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ClickTime.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **ClickTime**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ClickTime**.

    ![Vínculo de ClickTime en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-clicktime-test-user"></a>Creación de un usuario de prueba de ClickTime

Para permitir que los usuarios de Azure AD inicien sesión en ClickTime, deben aprovisionarse en ClickTime.  
En el caso de ClickTime, el aprovisionamiento es una tarea manual.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de ClickTime ofrecida por ClickTime para aprovisionar cuentas de usuario de Azure AD.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **ClickTime** .

1. En la barra de herramientas de la parte superior, haga clic en **Compañía** y, luego, en **Contactos**.
   
    ![Contactos](./media/clicktime-tutorial/tic777282.png "Personas")

1. Haga clic en **Add Person**(Agregar persona).
   
    ![Add Person (Agregar persona)](./media/clicktime-tutorial/tic777283.png "Agregar persona")

1. En la sección New Person (Nueva persona), lleve a cabo estos pasos:
   
    ![Contactos](./media/clicktime-tutorial/tic777284.png "Personas")
   
    a.  En el cuadro de texto de **nombre completo**, escriba el nombre completo de un usuario, por ejemplo, **Britta Simon**. 
  
    b.  En el cuadro de texto de **correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, **brittasimon\@contoso.com**.
       
    > [!NOTE]
    > Si lo desea, puede establecer propiedades adicionales del nuevo objeto persona.
   
    c.  Haga clic en **Save**(Guardar).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de ClickTime en el panel de acceso, debería iniciar sesión automáticamente en la versión de ClickTime para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

