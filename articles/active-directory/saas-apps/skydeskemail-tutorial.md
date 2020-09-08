---
title: 'Tutorial: Integración de Azure Active Directory con Skydesk Email | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Skydesk Email.
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
ms.openlocfilehash: 71150ea5ba7d6f25b07a9afaa1f1dc60adcb3e24
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88518777"
---
# <a name="tutorial-azure-active-directory-integration-with-skydesk-email"></a>Tutorial: Integración de Azure Active Directory con Skydesk Email

En este tutorial, aprenderá a integrar SkyDesk Email con Azure Active Directory (Azure AD).
Integrar Skydesk Email con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a SkyDesk Email.
* Puede permitir que los usuarios inicien sesión automáticamente en Skydesk Email (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Skydesk Email, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Skydesk Email

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Skydesk Email admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-skydesk-email-from-the-gallery"></a>Adición de Skydesk Email desde la galería

Para configurar la integración de Skydesk Email en Azure AD, será preciso que agregue Skydesk Email desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Skydesk Email desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Skydesk Email**, seleccione **Skydesk Email** en el panel de resultados y, a continuación, haga clic en el botón **Agregar** para agregar la aplicación.

     ![SkyDesk Email en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con SkyDesk Email con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Skydesk Email.

Para configurar y probar el inicio de sesión único de Azure AD con Skydesk Email, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Skydesk Email](#configure-skydesk-email-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Skydesk Email](#create-skydesk-email-test-user)** : para tener un homólogo de Britta Simon en Skydesk Email que esté vinculado a la representación de ella en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SkyDesk Email, realice los siguientes pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **SkyDesk Email**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Skydesk Email](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://mail.skydesk.jp/portal/<companyname>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de SkyDesk Email](https://www.skydesk.jp/apps/support/) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Skydesk Email** (Configurar Skydesk Email), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-skydesk-email-single-sign-on"></a>Configuración del inicio de sesión único de SkyDesk Email

1. En un explorador web diferente, inicie sesión en su cuenta de SkyDesk Email como administrador.

1. En el menú en la parte superior, haga clic en **Setup** (Configurar) y seleccione **Org** (Organización).

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_51.png)
  
1. En el panel izquierdo, haga clic en **Domains** (Dominios).

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_53.png)

1. Haga clic en **Add Domain** (Agregar dominio).

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_54.png)

1. Escriba el nombre de dominio y, a continuación, compruebe el dominio.

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_55.png)

1. Haga clic en **SAML Authentication** (Autenticación SAML) desde el panel izquierdo.

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_52.png)

1. En la página de diálogo **SAML Authentication** (Autenticación SAML), siga estos pasos:

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_56.png)

    > [!NOTE]
    > Para usar la autenticación basada en SAML, necesita **comprobar el dominio** o configurar la **dirección URL del portal**. Puede configurar la dirección URL del portal con el nombre único.

    ![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_57.png)

    a. En el cuadro de texto **Login URL** (Dirección URL de inicio de sesión), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Logout URL** (URL de cierre de sesión), pegue el valor de la **dirección URL de cierre de sesión** que copió de Azure Portal.

    c. **Cambiar dirección URL de contraseña** es opcional, déjelo en blanco.

    d. Haga clic en **Get Key From File** (Obtener clave de archivo) para seleccionar el certificado descargado desde Azure Portal y, después, haga clic en **Abrir** para cargar el certificado.

    e. En **Algoritmo**, seleccione **RSA**.

    f. Haga clic en **Aceptar** para guardar los cambios.

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

En esta sección, concederá acceso a Britta Simon a SkyDesk Email para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **SkyDesk Email**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Skydesk Email**.

    ![Vínculo de SkyDesk Email en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-skydesk-email-test-user"></a>Creación de un usuario de prueba de SkyDesk Email

En esta sección, creará un usuario llamado Britta Simon en Skydesk Email.

Haga clic en **User Access** (Acceso de usuario) en el panel de la izquierda de Skydesk Email y, a continuación, escriba su nombre de usuario.

![Configurar inicio de sesión único](./media/skydeskemail-tutorial/tutorial_skydeskemail_58.png)

> [!NOTE]
> Si necesita crear usuarios de forma masiva, debe ponerse en contacto con el [equipo de soporte al cliente de Skydesk Email](https://www.skydesk.jp/apps/support/).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SkyDesk Email en el Panel de acceso, debería iniciar sesión automáticamente en la versión de SkyDesk Email para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

