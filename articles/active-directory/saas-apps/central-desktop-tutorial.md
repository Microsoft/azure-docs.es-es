---
title: 'Tutorial: Integración de Azure Active Directory con Central Desktop | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Central Desktop.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: dc444c0179078713f9586c47c0138fe15f246a5f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88530484"
---
# <a name="tutorial-azure-active-directory-integration-with-central-desktop"></a>Tutorial: integración de Azure Active Directory con Central Desktop

En este tutorial, obtendrá información sobre cómo integrar Central Desktop con Azure Active Directory (Azure AD).
Integrar Central Desktop con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Central Desktop.
* Puede permitir que los usuarios inicien sesión automáticamente en Central Desktop (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Central Desktop, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción a Central Desktop con el inicio de sesión único habilitado

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Central Desktop admite inicio de sesión único iniciado por **SP**

## <a name="adding-central-desktop-from-the-gallery"></a>Adición de Central Desktop desde la galería

Para configurar la integración de Central Desktop en Azure AD, deberá agregar Central Desktop desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Central Desktop desde la galería, realice los siguientes pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Central Desktop**, seleccione **Central Desktop** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Central Desktop en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Central Desktop con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Central Desktop.

Para configurar y probar el inicio de sesión único de Azure AD con Central Desktop, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Central Desktop](#configure-central-desktop-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de Central Desktop](#create-central-desktop-test-user)** : para tener un homólogo de Britta Simon en Central Desktop vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Central Desktop, siga estos pasos:

1. En la página de integración de la aplicación [Central Desktop](https://portal.azure.com/) de **Azure Portal**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información acerca del inicio de sesión único de dominio y direcciones URL de Central Desktop](common/sp-identifier-reply.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.centraldesktop.com`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón:
    
    ```http
    https://<companyname>.centraldesktop.com/saml2-metadata.php
    https://<companyname>.imeetcentral.com/saml2-metadata.php
    ```

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.centraldesktop.com/saml2-assertion.php`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Central Desktop](https://imeetcentral.com/contact-us) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (sin procesar)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Set up Central Desktop** (Configurar Central Desktop), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-central-desktop-single-sign-on"></a>Configuración del inicio de sesión único en Central Desktop

1. Inicie sesión en su inquilino de **Central Desktop**.

2. Vaya a **Configuración**. Seleccione **Avanzado** y, a continuación, seleccione **Inicio de sesión único**.

    ![Setup - Advanced (Configuración - Avanzado)](./media/central-desktop-tutorial/ic769563.png "Programa de instalación: avanzado")

3. En la página **Configuración de inicio de sesión único**, siga estos pasos:

    ![Single sign-on settings (Configuración de inicio de sesión único)](./media/central-desktop-tutorial/ic769564.png "Configuración de inicio de sesión único")

    a. Seleccione **Enable SAML v2 Single Sign On**(Habilitar inicio de sesión único de SAML).

    b. En el cuadro **SSO URL** (URL de SSO), pegue el **Identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro **SSO Login URL** (URL de inicio de sesión de SSO), pegue la **Dirección URL de inicio de sesión** que copió de Azure Portal.

    d. En el cuadro **SSO Logout URL** (URL de cierre de sesión de SSO), pegue la **Dirección URL de cierre de sesión** que copió de Azure Portal.

4. En la sección **Message Signature Verification Method** (Método de autenticación de firma de mensaje), realice los pasos siguientes:

    ![Message Signature Verification Method (Método de autenticación de firma de mensaje)](./media/central-desktop-tutorial/ic769565.png "Método de autenticación de firma de mensaje")
    
    a. Seleccione **Certificado**.

    b. En la lista **SSO Certificate** (Certificado de SSO), seleccione **RSH SHA256**.

    c. Abra el certificado descargado en el Bloc de notas. A continuación, copie el contenido del certificado y péguelo en el campo **SSO Certificate** (Certificado de SSO).

    d. Seleccione **Mostrar un vínculo a la página de inicio de sesión SAMLv2**.

    e. Seleccione **Actualizar**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Central Desktop.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Central Desktop**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Central Desktop**.

    ![Vínculo a Central Desktop en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-central-desktop-test-user"></a>Creación del usuario de prueba de Central Desktop

Para que los usuarios de Azure AD puedan iniciar sesión, deben aprovisionarse en la aplicación Central Desktop. En esta sección se describe cómo crear cuentas de usuario de Azure AD en Central Desktop.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Central Desktop que ofrezca Central Desktop para aprovisionar cuentas de usuario de Azure AD.

**Para el aprovisionamiento de cuentas de usuario a Central Desktop:**

1. Inicie sesión en su inquilino de Central Desktop.

2. Seleccione **People** (Personas) y **Add Internal Members** (Agregar miembros internos).

    ![Contactos](./media/central-desktop-tutorial/ic781051.png "Personas")

3. En el cuadro de texto **Email Address of New Members** (Dirección de correo electrónico de nuevos miembros), escriba una cuenta de Azure AD que quiera aprovisionar y luego haga clic en **Siguiente**.

    ![Email Address of New Members (Dirección de correo electrónico de nuevos miembros)](./media/central-desktop-tutorial/ic781052.png "Email Address of New Members (Dirección de correo electrónico de nuevos miembros)")

4. Seleccione **Add Internal Members** (Agregar miembros internos).

    ![Add internal members (Agregar miembros internos)](./media/central-desktop-tutorial/ic781053.png "Add internal members (Agregar miembros internos)")
  
   > [!NOTE]
   > Los usuarios que se agregan reciben un correo electrónico que incluye un vínculo de confirmación para activar sus cuentas.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Central Desktop en el panel de acceso, debería iniciar sesión automáticamente en la versión de Central Desktop para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
