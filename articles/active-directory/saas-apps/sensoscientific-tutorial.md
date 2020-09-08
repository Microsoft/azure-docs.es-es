---
title: 'Tutorial: Integración de Azure Active Directory con SensoScientific Wireless Temperature Monitoring System | Microsoft Docs'
description: Obtenga información sobre cómo configurar un inicio de sesión único entre Azure Active Directory y SensoScientific Wireless Temperature Monitoring System.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/10/2019
ms.author: jeedes
ms.openlocfilehash: 61807f90caad9e6be2a25e54dcaa837595afc729
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543185"
---
# <a name="tutorial-azure-active-directory-integration-with-sensoscientific-wireless-temperature-monitoring-system"></a>Tutorial: Integración de Azure Active Directory con SensoScientific Wireless Temperature Monitoring System

En este tutorial, aprenderá a integrar SensoScientific Wireless Temperature Monitoring System con Azure Active Directory (Azure AD).
La integración de SensoScientific Wireless Temperature Monitoring System con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién accede a SensoScientific Wireless Temperature Monitoring System.
* Puede habilitar que los usuarios inicien sesión automáticamente en SensoScientific Wireless Temperature Monitoring System (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con SensoScientific Wireless Temperature Monitoring System, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción que tenga habilitado el inicio de sesión único en SensoScientific Wireless Temperature Monitoring System

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SensoScientific Wireless Temperature Monitoring System admite el inicio de sesión único iniciado por **IDP**

## <a name="adding-sensoscientific-wireless-temperature-monitoring-system-from-the-gallery"></a>Adición de SensoScientific Wireless Temperature Monitoring System desde la galería

Para configurar la integración de SensoScientific Wireless Temperature Monitoring System en Azure AD, debe agregar dicho sistema desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar SensoScientific Wireless Temperature Monitoring System desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SensoScientific Wireless Temperature Monitoring System**, selecciónelo en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![SensoScientific Wireless Temperature Monitoring System en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con SensoScientific Wireless Temperature Monitoring System mediante un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de SensoScientific Wireless Temperature Monitoring System.

Para configurar y probar el inicio de sesión único de Azure AD con SensoScientific Wireless Temperature Monitoring System, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de SensoScientific Wireless Temperature Monitoring System](#configure-sensoscientific-wireless-temperature-monitoring-system-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en SensoScientific Wireless Temperature Monitoring System](#create-sensoscientific-wireless-temperature-monitoring-system-test-user)** : para encontrar un homólogo de Britta Simon en SensoScientific Wireless Temperature Monitoring System vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SensoScientific Wireless Temperature Monitoring System, siga estos pasos:

1. En la página de integración de la aplicación **SensoScientific Wireless Temperature Monitoring System** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

    ![Información sobre dominio y direcciones URL de inicio de sesión único de SensoScientific Wireless Temperature Monitoring System](common/preintegrated.png)

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up SensoScientific Wireless Temperature Monitoring System** (Configurar SensoScientific Wireless Temperature Monitoring System), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-sensoscientific-wireless-temperature-monitoring-system-single-sign-on"></a>Configuración del inicio de sesión único en SensoScientific Wireless Temperature Monitoring System

1. Inicie sesión en la aplicación SensoScientific Wireless Temperature Monitoring System como administrador.

1. En el menú de navegación de la parte superior, haga clic en **Configuration** (Configuración) y vaya a **Configure** (Configurar) en **Single Sign On** (Inicio de sesión único) para abrir la configuración del inicio de sesión:

    ![Configurar inicio de sesión único](./media/sensoscientific-tutorial/tutorial_sensoscientificwtms_admin.png)

    a. Seleccione **Issuer Name** (Nombre del emisor) como Azure AD.

    b. En el cuadro de texto **URL del emisor**, pegue el **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Dirección URL del servicio de inicio de sesión único**, pegue la **Dirección URL de inicio de sesión** que copió desde Azure Portal.

    d. En el cuadro de texto **Dirección URL del servicio de cierre de sesión único**, pegue la **URL de cierre de sesión** que copió de Azure Portal.

    e. Examine el certificado que ha descargado de Azure Portal y cárguelo aquí.

    f. Haga clic en **Save**(Guardar).

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

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a SensoScientific Wireless Temperature Monitoring System.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **SensoScientific Wireless Temperature Monitoring System**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **SensoScientific Wireless Temperature Monitoring System**.

    ![Vínculo a SensoScientific Wireless Temperature Monitoring System en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-sensoscientific-wireless-temperature-monitoring-system-test-user"></a>Creación de un usuario de prueba en SensoScientific Wireless Temperature Monitoring System

Para permitir que los usuarios de Azure AD inicien sesión en SensoScientific Wireless Temperature Monitoring System, es necesario aprovisionarlos en la aplicación. Colabore con el  [equipo de soporte técnico de SensoScientific Wireless Temperature Monitoring System](https://www.sensoscientific.com/contact-us/) para agregar usuarios a la plataforma de SensoScientific Wireless Temperature Monitoring System. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SensoScientific Wireless Temperature Monitoring System en el panel de acceso, debe iniciar sesión automáticamente en la versión de SensoScientific Wireless Temperature Monitoring System para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

