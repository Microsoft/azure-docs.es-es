---
title: 'Tutorial: Integración de Azure Active Directory con LearnUpon | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y LearnUpon.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: b11c6315-c79d-4f34-9610-bd17070ab7c7
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 574c21dc2713f10513ac296e7db538e20a94c9d6
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406530"
---
# <a name="tutorial-azure-active-directory-integration-with-learnupon"></a>Tutorial: Integración de Azure Active Directory con LearnUpon

En este tutorial, aprenderá a integrar LearnUpon con Azure Active Directory (Azure AD).
La integración de LearnUpon con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a LearnUpon.
* Puede permitir que los usuarios inicien sesión automáticamente en LearnUpon (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con LearnUpon, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en LearnUpon

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.


* LearnUpon admite inicio de sesión único iniciado por **IDP**

* LearnUpon admite el aprovisionamiento de usuarios **Just In Time**.


## <a name="adding-learnupon-from-the-gallery"></a>Incorporación de LearnUpon desde la galería

Para configurar la integración de LearnUpon en Azure AD, es preciso agregar LearnUpon desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar LearnUpon desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **LearnUpon**, seleccione **LearnUpon** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![LearnUpon en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con LearnUpon con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de LearnUpon.

Para configurar y probar el inicio de sesión único de Azure AD con LearnUpon, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)**: para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de LearnUpon](#configure-learnupon-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**, para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**, para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de LearnUpon](#create-learnupon-test-user)**: el objetivo es tener un homólogo de Britta Simon en LearnUpon que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)**: para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con LearnUpon, siga estos pasos:

1. En la página de integración de la aplicación **LearnUpon** de [Azure Portal](https://portal.azure.com/), haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de LearnUpon](common/idp-reply.png)

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.learnupon.com/saml/consumer`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de respuesta real. Póngase en contacto con el [equipo de soporte técnico de LearnUpon](https://www.learnupon.com/features/support/) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, ubique la **huella digital**. Se agregará a su configuración de SAML de LearnUpon.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Configurar LearnUpon**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-learnupon-single-sign-on"></a>Configuración de inicio de sesión único de LearnUpon

1. Abra otra instancia del explorador e inicie sesión en LearnUpon con una cuenta de administrador.

1. Haga clic en la pestaña **settings** (Configuración).

    ![Configurar inicio de sesión único](./media/learnupon-tutorial/tutorial_learnupon_06.png)

1. Haga clic en **Single Sign On - SAML** (Inicio de sesión único - SAML) y, después, en **General Settings** (Configuración general) para configurar SAML.
   
    ![Configurar inicio de sesión único](./media/learnupon-tutorial/tutorial_learnupon_07.png) 

1. En la sección **General Settings** (Configuración general), siga estos pasos:
   
    ![Configurar inicio de sesión único](./media/learnupon-tutorial/tutorial_learnupon_08.png)  
  
     a. Seleccione **Habilitado**.

    b. Seleccione la **versión** **2.0**.

    c. En **Skip conditions** (Omitir condiciones), haga clic en **No**.

    d. En el cuadro de texto **SAML Token Post param name** (Nombre de parámetro POST de token SAML), escriba el nombre del parámetro POST de la solicitud en la dirección URL del consumidor de SAML indicada anteriormente que contenga la aserción SAML que se va a comprobar y autenticar (por ejemplo, **SAMLResponse**).

    e. En el cuadro de texto **Name Identifier Format** (Formato de identificador de nombre), escriba el valor que indica en qué lugar de la aserción SAML reside el identificador del usuario (dirección de correo electrónico) (por ejemplo, `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`).
  
    f. En el cuadro de texto **Identify Provider Location** (Identificar la ubicación del proveedor), escriba el valor que indica el lugar al que se envían los usuarios si hacen clic en el icono cargado desde la pantalla de inicio de sesión de Azure Portal.
  
    g. En el cuadro de texto **Sign Out URL** (URL de cierre de sesión), pegue el valor de la **URL de cierre de sesión** que ha copiado de Azure Portal.

    h. Haga clic en **Manage finger prints**(Administrar huellas dactilares) y, a continuación, cargue la huella digital del certificado descargado.

1. Haga clic en **User Settings**(Configuración del usuario) y siga estos pasos:

     ![Configurar inicio de sesión único](./media/learnupon-tutorial/tutorial_learnupon_11.png)  

     a. En el cuadro de texto **First Name Identifier Format** (Formato de identificador de nombre), escriba el valor que nos indica el lugar de la aserción SAML en que reside el nombre de los usuarios (por ejemplo: `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`).
  
    b. En el cuadro de texto **Last Name Identifier Format** (Formato de identificador de apellido), escriba el valor que nos indica el lugar de la aserción SAML en que reside el apellido de los usuarios (por ejemplo: `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`).

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

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a LearnUpon.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **LearnUpon**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **LearnUpon**.

    ![Vínculo a LearnUpon en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-learnupon-test-user"></a>Creación de un usuario de prueba de LearnUpon

En esta sección, se crea un usuario llamado Britta Simon en LearnUpon. LearnUpon admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en LearnUpon, se crea otro después de la autenticación. Si necesita crear un usuario manualmente, es preciso que se ponga contacto con el [equipo de soporte técnico de LearnUpon](https://www.learnupon.com/features/support/).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de LearnUpon en el panel de acceso y debería iniciar sesión automáticamente en la versión de LearnUpon para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)