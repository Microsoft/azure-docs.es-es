---
title: 'Tutorial: Integración de Azure Active Directory con Signagelive | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Signagelive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 1/11/2019
ms.author: jeedes
ms.openlocfilehash: 5a8fe24e5fa1935b2363ed5d2c15c72af8af11ff
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88539780"
---
# <a name="tutorial-azure-active-directory-integration-with-signagelive"></a>Tutorial: Integración de Azure Active Directory con Signagelive

En este tutorial, aprenderá a integrar Signagelive con Azure Active Directory (Azure AD).
La integración de Signagelive con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Signagelive.
* Puede permitir que los usuarios inicien sesión automáticamente en Signagelive (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis). Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Signagelive, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una evaluación gratuita de un mes](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción habilitada para el inicio de sesión único en Signagelive

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Signagelive admite el inicio de sesión único iniciado por SP.

## <a name="add-signagelive-from-the-gallery"></a>Adición de Signagelive desde la galería

Para configurar la integración de Signagelive en Azure AD, primero debe agregar Signagelive desde la galería a la lista de aplicaciones SaaS administradas.

Para agregar Signagelive desde la galería, siga los pasos que se indican a continuación:

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione el icono **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Signagelive**. 

     ![Signagelive en la lista de resultados](common/search-new-app.png)

5. En el panel de resultados, seleccione **Signagelive** y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Signagelive con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer un vínculo entre un usuario de Azure AD y el usuario relacionado de Signagelive.

Para configurar y probar el inicio de sesión único de Azure AD con Signagelive, primero debe completar los siguientes bloques de creación:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on), para permitir que los usuarios utilicen esta característica.
2. [Configuración del inicio de sesión único de Signagelive](#configure-signagelive-single-sign-on): para configurar el inicio de sesión único en la aplicación.
3. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con Britta Simon.
4. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. [Creación de un usuario de prueba de Signagelive](#create-a-signagelive-test-user): para tener un homólogo de Britta Simon en Signagelive que esté vinculado a la representación del usuario en Azure AD.
6. [Prueba del inicio de sesión único](#test-single-sign-on) para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Signagelive, siga los pasos que se indican a continuación:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Signagelive**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga los pasos que se indican a continuación:

    ![Información de dominio y direcciones URL de inicio de sesión único de Signagelive](common/sp-signonurl.png)

    En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://login.signagelive.com/sso/<ORGANIZATIONALUNITNAME>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Signagelive](mailto:support@signagelive.com) para obtener este valor. También puede hacer referencia a los patrones de la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **certificado (sin procesar)** de las opciones proporcionadas según sus requisitos. Luego, guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Set up Signagelive** (Configurar Signagelive), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-signagelive-single-sign-on"></a>Configuración del inicio de sesión único de Signagelive

Para configurar el inicio de sesión único en Signagelive, envíe el **certificado (sin procesar)** descargado y las direcciones URL copiadas de Azure Portal al [equipo de soporte técnico de Signagelive](mailto:support@signagelive.com). Estas garantizan que la conexión de inicio de sesión único de SAML está establecida correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba "brittasimon@yourcompanydomain.extension". Por ejemplo, en este caso, podría escribir "BrittaSimon@contoso.com".

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá a Britta Simon acceso a Signagelive para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, después, **Signagelive**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Signagelive**.

    ![Vínculo a Signagelive en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Seleccione el botón **Agregar usuario**. Después, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, en la lista **Usuarios** seleccione **Britta Simon**. A continuación, haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-a-signagelive-test-user"></a>Creación de un usuario de prueba de Signagelive

En esta sección, creará un usuario llamado Britta Simon en Signagelive. Colabore con el  [equipo de soporte técnico de Signagelive](mailto:support@signagelive.com) para agregar los usuarios en la plataforma de Signagelive. Los usuarios deben crearse y activarse antes de usar el inicio de sesión único.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal MyApps.

Al seleccionar el icono de **Signagelive** en el portal MyApps, debería iniciarse automáticamente su sesión. Para obtener más información sobre el portal MyApps, consulte [¿Qué es el portal MyApps?](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

