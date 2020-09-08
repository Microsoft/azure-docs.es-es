---
title: 'Tutorial: Integración de Azure Active Directory con Vidyard | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Vidyard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 2c6b88e9c59fa7195e77275ad193975924df137f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88532184"
---
# <a name="tutorial-azure-active-directory-integration-with-vidyard"></a>Tutorial: Integración de Azure Active Directory con Vidyard

En este tutorial, obtendrá información sobre cómo integrar Vidyard con Azure Active Directory (Azure AD).
La integración de Vidyard con Azure AD proporciona las ventajas siguientes:

* Puede controlar en Azure AD quién tiene acceso a Vidyard.
* Puede permitir que los usuarios inicien sesión automáticamente en Vidyard (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Vidyard, necesita los elementos siguientes:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción de Vidyard habilitada para el inicio de sesión único

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Vidyard admite SSO iniciado por **SP** e **IDP**

* Vidyard admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-vidyard-from-the-gallery"></a>Adición de Vidyard desde la galería

Para configurar la integración de Vidyard en Azure AD, tendrá que agregar Vidyard desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Vidyard desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Vidyard**, seleccione **Vidyard** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Vidyard en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Vidyard con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Vidyard.

Para configurar y probar el inicio de sesión único de Azure AD con Vidyard, es preciso completar los bloques de creación siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Vidyard](#configure-vidyard-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Vidyard](#create-vidyard-test-user)** : para tener un homólogo de Britta Simon en Vidyard que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Vidyard, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Vidyard**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Vidyard](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://secure.vidyard.com/sso/saml/<unique id>/metadata`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://secure.vidyard.com/sso/saml/<unique id>/consume`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Vidyard](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://secure.vidyard.com/sso/saml/<unique id>/login`

    > [!NOTE]
    > Estos valores no son reales. Los valores se actualizan con el identificador, la dirección URL de inicio de sesión y la dirección URL de respuesta reales, lo que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up Vidyard** (Configurar Vidyard), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-vidyard-single-sign-on"></a>Configuración del inicio de sesión único de Vidyard

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de empresa de Vidyard Software.

2. En el panel Vidyard, seleccione **Grupo** > **Seguridad**

    ![Configuración de Vidyard](./media/vidyard-tutorial/configure1.png)

3. Haga clic en la pestaña **Nuevo perfil**.

    ![Configuración de Vidyard](./media/vidyard-tutorial/configure2.png)

4. En la sección **Configuración de SAML**, siga estos pasos:

    ![Configuración de Vidyard](./media/vidyard-tutorial/configure3.png)

    a. Escriba el nombre de perfil general en el cuadro de texto **Nombre de perfil**.

    b. Copie el valor de **SSO User Login Page** (Página de inicio de sesión de usuario de SSO) y péguelo en el cuadro de texto **Dirección URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.

    c. Copie el valor **ACS URL** (URL de ACS) y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** de Azure Portal.

    d. Copie el valor **Issuer/Metadata URL** (URL de emisor/metadatos) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** de Azure Portal.

    e. Abra el certificado descargado de Azure Portal en el Bloc de notas y luego péguelo en el cuadro de texto **Certificado X.509**.

    f. En el cuadro de texto **SAML Endpoint URL** (URL de punto de conexión de SAML 2.0), pegue el valor de **URL de inicio de sesión** copiado de Azure Portal.

    g. Haga clic en **Confirmar**.

5. En la pestaña Inicio de sesión único, haga clic en **Asignar** junto a un perfil existente.

    ![Configuración de Vidyard](./media/vidyard-tutorial/configure4.png)

    > [!NOTE]
    > Una vez que haya creado un perfil de SSO, asígnelo a los grupos para los que los usuarios necesitarán acceso a través de Azure. Si el usuario no existe en el grupo al que se asignó, Vidyard creará automáticamente una cuenta de usuario y asignará su rol en tiempo real.

6. Seleccione el grupo de la organización, que es visible en **Groups Available to Assign** (Grupos disponibles para asignar).

    ![Configuración de Vidyard](./media/vidyard-tutorial/configure5.png)

7. Puede ver los grupos asignados en **Groups Currently Assigned** (Grupos asignados actualmente). Seleccione un rol para el grupo de acuerdo con la organización y haga clic en **Confirmar**.

    ![Configuración de Vidyard](./media/vidyard-tutorial/configure6.png)

    > [!NOTE]
    > Para más información, consulte [este documento](https://knowledge.vidyard.com/hc/articles/360009990033-SAML-based-Single-Sign-On-SSO-in-Vidyard).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Vidyard.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y luego **Vidyard**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Vidyard**.

    ![Vínculo a Vidyard en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-vidyard-test-user"></a>Creación de un usuario de prueba de Vidyard

En esta sección, se crea un usuario llamado Britta Simon en Vidyard. Vidyard admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Vidyard, se crea otro después de la autenticación.

>[!Note]
>Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Vidyard](mailto:support@vidyard.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Vidyard en el panel de acceso debería iniciar sesión automáticamente en la versión de Vidyard para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

