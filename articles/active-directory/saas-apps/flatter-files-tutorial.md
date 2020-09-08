---
title: 'Tutorial: integración de Azure Active Directory con Flatter Files | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Flatter Files.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/15/2019
ms.author: jeedes
ms.openlocfilehash: 7dd13df6222f6441b7edb2bd9a789c1f102168d7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554981"
---
# <a name="tutorial-azure-active-directory-integration-with-flatter-files"></a>Tutorial: integración de Azure Active Directory con Flatter Files

En este tutorial, aprenderá a integrar Flatter Files con Azure Active Directory (Azure AD).
Integrar Flatter Files con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Flatter Files.
* Puede permitir que los usuarios inicien sesión automáticamente en Flatter Files (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Flatter Files, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Flatter Files

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Flatter Files admite el inicio de sesión único iniciado por **IDP**

## <a name="adding-flatter-files-from-the-gallery"></a>Adición de Flatter Files desde la galería

Para configurar la integración de Flatter Files en Azure AD, deberá agregar Flatter Files desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Flatter Files desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Flatter Files**, seleccione **Flatter Files** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Flatter Files en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, va a configurar y probar el inicio de sesión único de Azure AD con Flatter Files con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Flatter Files.

Para configurar y probar el inicio de sesión único de Azure AD con Flatter Files, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Flatter Files](#configure-flatter-files-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Flatter Files](#create-flatter-files-test-user)** : para tener un homólogo de Britta Simon en Flatter Files que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Flatter Files, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Flatter Files**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

    ![Información de dominio y direcciones URL de inicio de sesión único de Flatter Files](common/preintegrated.png)

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Flatter Files** (Configurar Flatter Files), copie las direcciones URL copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-flatter-files-single-sign-on"></a>Configuración del inicio de sesión único de Flatter Files

1. Inicie sesión en su aplicación de Flatter Files como administrador.

2. Haga clic en **PANEL**. 
   
    ![Configurar inicio de sesión único](./media/flatter-files-tutorial/tutorial_flatter_files_05.png)  

3. Haga clic en **Settings** (Configuración) y, después, siga estos pasos siguientes en la pestaña **Company** (Compañía): 
   
    ![Configurar inicio de sesión único](./media/flatter-files-tutorial/tutorial_flatter_files_06.png)  
    
    a. Seleccione **Usar SAML 2.0 para autenticación**.
    
    b. Haga clic en **Configurar SAML**.

4. En el cuadro de diálogo **Configuración de SAML** , realice los siguientes pasos: 
   
    ![Configurar inicio de sesión único](./media/flatter-files-tutorial/tutorial_flatter_files_08.png)  
   
    a. En el cuadro de texto **Dominio**, escriba el dominio registrado.
   
   > [!NOTE]
   > Si no dispone de un dominio registrado, póngase en contacto con el equipo de soporte técnico de Flatter Files a través de [support@flatterfiles.com](mailto:support@flatterfiles.com). 
    
    b. En el cuadro de texto **Identity Provider URL** (Dirección URL del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.
   
    c.  Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de proveedor de identidades**.

    d. Haga clic en **Update**(Actualizar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Flatter Files.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Flatter Files**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Flatter Files**.

    ![Vínculo de Flatter Files en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-flatter-files-test-user"></a>Creación de un usuario de prueba de Flatter Files

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en Flatter Files.

**Para crear una usuaria llamada Britta Simon en Flatter Files, realice los pasos siguientes:**

1. Inicie sesión en su sitio de la compañía de **Flatter Files** como administrador.

2. En el panel de navegación de la izquierda, haga clic en **Configuración** y, luego, en la pestaña **Usuarios**.
   
    ![Creación de un usuario de Flatter Files](./media/flatter-files-tutorial/tutorial_flatter_files_09.png)

3. Haga clic en **Agregar usuario**. 

4. En el cuadro de diálogo **Agregar usuario** , realice los pasos siguientes:
   
    ![Creación de un usuario de Flatter Files](./media/flatter-files-tutorial/tutorial_flatter_files_10.png)

    a. En el cuadro de texto **Nombre**, escriba **Britta**.
   
    b. En el cuadro de texto **Apellidos**, escriba **Simon**. 
   
    c. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo electrónico de Britta en Azure Portal.
   
    d. Haga clic en **Enviar**.   


### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Flatter Files del panel de acceso, debería iniciar sesión automáticamente en la versión de Flatter Files para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

