---
title: 'Tutorial: Integración de Azure Active Directory con Procore SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Procore SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: e0a8e6786dbce1015d587cf8015e3624c97a5817
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553621"
---
# <a name="tutorial-azure-active-directory-integration-with-procore-sso"></a>Tutorial: Integración de Azure Active Directory con Procore SSO

En este tutorial aprenderá a integrar Procore SSO con Azure Active Directory (Azure AD).
La integración de Procore SSO con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Procore SSO.
* Puede permitir que los usuarios inicien sesión automáticamente en Procore SSO (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Procore SSO, necesita lo siguiente:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Procore SSO

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Procore SSO admite el inicio de sesión único iniciado por **IDP**

## <a name="adding-procore-sso-from-the-gallery"></a>Agregar Procore SSO desde la galería

Para configurar la integración de Procore SSO en Azure AD, tiene que agregar Procore SSO desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Procore SSO desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Procore SSO**, seleccione **Procore SSO** en el panel de resultados y, a continuación, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Procore SSO en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Procore SSO con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Procore SSO.

Para configurar y probar el inicio de sesión único de Azure AD con Procore SSO, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Procore SSO](#configure-procore-sso-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Procore SSO](#create-procore-sso-test-user)** : para tener un homólogo de Britta Simon en Procore SSO que esté vinculado a la representación de usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Procore SSO, siga los pasos que se indican a continuación:

1. En la página de integración de la aplicación **Procore SSO** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

    ![Información de dominio y direcciones URL de inicio de sesión único de Procore SSO](common/preintegrated.png)

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Procore SSO** (Configurar Procore SSO), copie la dirección URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-procore-sso-single-sign-on"></a>Configuración del inicio de sesión único de Procore SSO

1. Para configurar el inicio de sesión único en **Procore SSO**, inicie sesión como administrador en su sitio de la compañía de Procore.

2. En la lista desplegable del cuadro de herramientas, haga clic en **Administrador** para abrir la página de configuración del SSO.

    ![Configurar inicio de sesión único](./media/procoresso-tutorial/procore_tool_admin.png)

3. Pegue los valores en los cuadros tal y como se describe a continuación:

    ![Configurar inicio de sesión único](./media/procoresso-tutorial/procore_setting_admin.png)  

    a. En el cuadro de texto **Dirección URL del emisor de inicio de sesión único**, pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Dirección URL de destino de inicio de sesión único de SAML**, pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra ahora el **XML de metadatos de la federación** que descargó antes de Azure Portal y copie el certificado en el etiqueta denominada **X509Certificate**. Pegue el valor copiado en el cuadro **Single Sign On x509 Certificate** (Certificado x509 de inicio de sesión único).

4. Haga clic en **Guardar cambios**.

5. Al finalizar esta configuración, tiene que enviar el **nombre de dominio** (por ejemplo, **contoso.com**) con el que inicia sesión en Procore al [equipo de soporte técnico de Procore](https://support.procore.com/) y ellos activarán el SSO federado para ese dominio.

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

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a Procore SSO.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, después, **Procore SSO**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Procore SSO**.

    ![Vínculo a Procore SSO en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-procore-sso-test-user"></a>Creación de un usuario de prueba de Procore SSO

Siga estos pasos para crear un usuario de prueba en Procore del lado de Procore SSO.

1. Inicie sesión como administrador en su sitio de la compañía de Procore.    

2. En la lista desplegable del cuadro de herramientas, haga clic en **Directorio** para abrir la página de directorio de la compañía.

    ![Configurar inicio de sesión único](./media/procoresso-tutorial/Procore_sso_directory.png)

3. Haga clic en la opción **Add a Person** (Agregar una persona) para abrir el formulario y especificar la ejecución de las opciones siguientes:

    ![Configurar inicio de sesión único](./media/procoresso-tutorial/Procore_user_add.png)

    a. En el cuadro de texto **Nombre**, escriba el nombre del usuario, por ejemplo, **Britta**.

    b. En el cuadro de texto **Apellido**, escriba el apellido del usuario, por ejemplo, **Simon**.

    c. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo del usuario con el formato siguiente BrittaSimon@contoso.com.

    d. Seleccione **Permission Template** (Plantilla de permisos) como **Apply Permission Template Later** (Aplicar plantilla de permisos más tarde).

    e. Haga clic en **Crear**.

4. Compruebe los datos del contacto recién agregado y actualícelos.

    ![Configurar inicio de sesión único](./media/procoresso-tutorial/Procore_user_check.png)

5. Haga clic en **Save and Send Invitiation** (Guardar y enviar invitación), si se requiere una invitación por correo electrónico, o en **Guardar** (guardar directamente) para completar el registro del usuario.
    
    ![Configurar inicio de sesión único](./media/procoresso-tutorial/Procore_user_save.png)

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Procore SSO en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Procore SSO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

