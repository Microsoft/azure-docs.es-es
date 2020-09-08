---
title: 'Tutorial: integración de Azure Active Directory con HackerOne | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y HackerOne.
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
ms.openlocfilehash: 530ef271091afc3e646d375ea471d8c48f0f8651
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544203"
---
# <a name="tutorial-azure-active-directory-integration-with-hackerone"></a>Tutorial: Integración de Azure Active Directory con HackerOne

En este tutorial se aprende a integrar HackerOne con Azure Active Directory (Azure AD).
La integración de HackerOne con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a HackerOne.
* Puede permitir que los usuarios inicien sesión automáticamente en HackerOne (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con HackerOne, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en HackerOne

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* HackerOne admite el inicio de sesión único iniciado por **SP**
* HackerOne admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-hackerone-from-the-gallery"></a>Incorporación de HackerOne desde la galería

Para configurar la integración de HackerOne en Azure AD, será preciso que agregue HackerOne desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar HackerOne desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **HackerOne**, seleccione **HackerOne** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![HackerOne en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección se configura y prueba el inicio de sesión único de Azure AD con HackerOne con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de HackerOne.

Para configurar y probar el inicio de sesión único de Azure AD con HackerOne, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en HackerOne](#configure-hackerone-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de HackerOne](#create-hackerone-test-user)** : para tener un homólogo de Britta Simon en HackerOne vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con HackerOne, realice los siguientes pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **HackerOne**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de HackerOne](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba lo siguiente: `https://hackerone.com/users/saml/sign_in?email=<configured domain>`

    b. En el cuadro de texto **Identificador (Id. de entidad)** , escriba lo siguiente:`hackerone.com`

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up HackerOne** (Configurar HackerOne), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-hackerone-single-sign-on"></a>Configuración del inicio de sesión único en HackerOne

1. Inicie sesión en el inquilino de HackerOne como administrador.

2. En el menú de la parte superior, haga clic en **Configuración**.

    ![Configurar inicio de sesión único](./media/hackerone-tutorial/tutorial_hackerone_001.png)

3. Vaya a **Authentication** (Autenticación) y haga clic en el botón **Add SAML settings** (Agregar configuración de SAML).

    ![Configurar inicio de sesión único](./media/hackerone-tutorial/tutorial_hackerone_003.png)

4. En el cuadro de diálogo **SAML Settings** (Configuración de SAML), siga estos pasos:

    ![Configurar inicio de sesión único](./media/hackerone-tutorial/tutorial_hackerone_004.png)

    a. En el cuadro de texto **Dominio de correo electrónico** , escriba un dominio registrado.

    b. En el cuadro de texto **Single Sign-On URL** (Dirección URL de inicio de sesión único), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el **archivo de certificado** que descargó de Azure Portal en el bloc de notas, copie su contenido en el portapapeles y péguelo en el cuadro de texto **Certificado X509**.

    d. Haga clic en **Save**(Guardar).

5. En el diálogo Configuración de autenticación, realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/hackerone-tutorial/tutorial_hackerone_005.png)

    a. Haga clic en **Ejecutar prueba**.

6. Cuando la prueba finalice correctamente y el campo **Status** (Estado) muestre **Last test status: success** (Último estado de la prueba: correcto), seleccione el botón **Request Verification** (Solicitar comprobación) para enviar a HackerOne para su aprobación.

    ![Envío a HackerOne para su aprobación](./media/hackerone-tutorial/tutorial-hackerone-006.png)

7. Una vez que HackerOne aprueba la configuración, puede seleccionar el botón **Migrate Users** (Migrar usuarios) para requerir la autenticación de inicio de sesión único para todos los usuarios.

    ![Habilitación de SAML](./media/hackerone-tutorial/tutorial-hackerone-007.png)

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

En esta sección se habilita a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a HackerOne.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **HackerOne**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **HackerOne**.

    ![Vínculo a HackerOne en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-hackerone-test-user"></a>Creación del usuario de prueba de HackerOne

En esta sección, se crea un usuario llamado Britta Simon en HackerOne. HackerOne admite el aprovisionamiento de usuarios Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario aún no existe en HackerOne, se crea uno después de la autenticación.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de HackerOne en el panel de acceso, debería iniciar sesión automáticamente en la versión de HackerOne para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
