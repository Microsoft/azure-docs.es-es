---
title: 'Tutorial: Integración de Azure Active Directory con Questetra BPM Suite | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Questetra BPM Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 11a6df348ae623ccf75e61bc1abbb5e28c832237
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548980"
---
# <a name="tutorial-azure-active-directory-integration-with-questetra-bpm-suite"></a>Tutorial: Integración de Azure Active Directory con Questetra BPM Suite

En este tutorial, aprenderá a integrar Questetra BPM Suite con Azure Active Directory (Azure AD).
La integración de Questetra BPM Suite con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Questetra BPM Suite.
* Puede permitir que los usuarios inicien sesión automáticamente en Questetra BPM Suite (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Questetra BPM Suite, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Questetra BPM Suite

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Questetra BPM Suite admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-questetra-bpm-suite-from-the-gallery"></a>Incorporación de Questetra BPM Suite desde la galería

Para configurar la integración de Questetra BPM Suite en Azure AD, deberá agregar Questetra BPM Suite desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Questetra BPM Suite desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Questetra BPM Suite**, seleccione **Questetra BPM Suite** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Questetra BPM Suite en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Questetra BPM Suite con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Questetra BPM Suite.

Para configurar y probar el inicio de sesión único de Azure AD con Questetra BPM Suite, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Questetra BPM Suite](#configure-questetra-bpm-suite-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Questetra BPM Suite](#create-questetra-bpm-suite-test-user)** : para tener un homólogo de Britta Simon en Questetra BPM Suite vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Questetra BPM Suite, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Questetra BPM Suite**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Questetra BPM Suite](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.questetra.net/saml/SSO/alias/bpm`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<subdomain>.questetra.net/`

    > [!NOTE]
    > Estos valores no son reales. Debe actualizarlos con la dirección URL y el identificador reales de inicio de sesión. Puede obtener estos valores de la sección **SP Information** (Información de soporte técnico) del sitio de **Questetra BPM Suite** de la empresa, que se explica más adelante en el tutorial, o al ponerse en contacto con el [equipo de atención al cliente de Questetra BPM Suite](https://www.questetra.com/contact/). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar Questetra BPM Suite**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-questetra-bpm-suite-single-sign-on"></a>Configuración del inicio de sesión único de Questetra BPM Suite

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de **Questetra BPM Suite** como administrador.

2. En el menú de la parte superior, haga clic en **Configuración del sistema**. 
   
    ![Inicio de sesión único de Azure AD][10]

3. Para abrir la página **SingleSignOnSAML**, haga clic en **SSO (SAML)** . 
   
    ![Inicio de sesión único de Azure AD][11]

4. En el sitio **Questetra BPM Suite** de la empresa, en la sección **SP información** (Información de soporte técnico), lleve a cabo los pasos siguientes:

    a. Copie el valor de **ACS URL** (URL de ACS) y péguelo en el cuadro de texto **URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.
    
    b. Copie el valor de **Entity Id** (Identificador de entidad) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** de Azure Portal.

5. En el sitio **Questetra BPM Suite** de la empresa, realice los pasos siguientes: 
   
    ![Configurar inicio de sesión único][15]
   
    a. Seleccione **Enable Single Sign-On**(Habilitar inicio de sesión único).
   
    b. En el cuadro de texto **Entity Id** (Identificador de entidad), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.
    
    c. En el cuadro de texto **Dirección URL de la página de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.
    
    d. En el cuadro de texto **Dirección URL de la página de cierre de sesión**, pegue el valor de **dirección URL de cierre de sesión** que copió de Azure Portal.
    
    e. En el cuadro de texto**NameID format** (Formato de id. de nombre), escriba `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

    f. Abra el certificado codificado en **base 64** descargado de Azure Portal en el bloc de notas, copie su contenido en el portapapeles y péguelo en el cuadro de texto **Certificado de verificación**. 

    g. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba brittasimon@yourcompanydomain.extension. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure al concederle acceso a Questetra BPM Suite.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Questetra BPM Suite**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Questetra BPM Suite**.

    ![En la lista de aplicaciones, seleccione Questetra BPM Suite.](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-questetra-bpm-suite-test-user"></a>Creación de un usuario de prueba de Questetra BPM Suite

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en Questetra BPM Suite.

**Para crear un usuario llamado Britta Simon en Questetra BPM Suite, realice los pasos siguientes:**

1. Inicie sesión en el sitio Questetra BPM Suite de la empresa como administrador.

2. Vaya a **System Settings > User List > New User** (Configuración del sistema > Lista de usuarios > Nuevo usuario).
 
3. En el cuadro de diálogo Nuevo usuario, realice los pasos siguientes: 
   
    ![Creación de un usuario de prueba][300] 
   
    a. En el cuadro de texto **Name** (Nombre), escriba el **nombre** del usuario: britta.simon@contoso.com.
   
    b. En el cuadro de texto **Email** (Correo electrónico), escriba el **correo electrónico** del usuario: britta.simon@contoso.com.
   
    c. En el cuadro de texto **Password** (Contraseña), escriba una **contraseña** para el usuario.
    
    d. Haga clic en **Add new user**(Agregar nuevo usuario).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Questetra BPM Suite en el panel de acceso y debería iniciar sesión automáticamente en la versión de Questetra BPM Suite para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

<!--Image references-->

[10]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_03.png
[11]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_04.png
[15]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_08.png
[300]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_11.png