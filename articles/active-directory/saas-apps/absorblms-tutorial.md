---
title: 'Tutorial: Integración de Azure Active Directory con Absorb LMS | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Absorb LMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/02/2019
ms.author: jeedes
ms.openlocfilehash: 59fb36765ad3cd584af4d6459cd78e2886d0edce
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88538712"
---
# <a name="tutorial-azure-active-directory-integration-with-absorb-lms"></a>Tutorial: Integración de Azure Active Directory con Absorb LMS

En este tutorial, aprenderá a integrar Absorb LMS con Azure Active Directory (Azure AD).
La integración de Absorb LMS con Azure AD ofrece las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Absorb LMS.
* Puede permitir que los usuarios inicien sesión automáticamente en Absorb LMS (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Absorb LMS, necesita lo siguiente:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para inicio de sesión único en Absorb LMS

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Absorb LMS admite SSO iniciado por **IDP**

## <a name="adding-absorb-lms-from-the-gallery"></a>Adición de Absorb LMS desde la galería

Para configurar la integración de Absorb LMS en Azure AD, será preciso que agregue Absorb LMS desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Absorb LMS desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Absorb LMS**, seleccione **Absorb LMS** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Absorb LMS en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Absorb LMS con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Absorb LMS.

Para configurar y probar el inicio de sesión único de Azure AD con Absorb LMS, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Absorb LMS](#configure-absorb-lms-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Absorb LMS](#create-absorb-lms-test-user)**: para tener un homólogo de Britta Simon en Absorb LMS que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Absorb LMS, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Absorb LMS**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Información de dominio y direcciones URL de inicio de sesión único de Absorb LMS](common/idp-intiated.png)

    Si usa **Absorb 5 - UI** (Absorb 5 - IU), utilice la siguiente configuración:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://company.myabsorb.com/account/saml`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://company.myabsorb.com/account/saml`

    Si usa **Absorb 5 - New Learner Experience** (Absorb 5 - Nueva experiencia de aprendiz), utilice la siguiente configuración:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://company.myabsorb.com/api/rest/v2/authentication/saml`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://company.myabsorb.com/api/rest/v2/authentication/saml`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Absorb LMS](https://support.absorblms.com/hc/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde **nameidentifier** se asigna con **user.userprincipalname**.

    ![imagen](common/edit-attribute.png)

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up Absorb LMS** (Configurar Absorb LMS), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-absorb-lms-single-sign-on"></a>Configuración del inicio de sesión único de Absorb LMS

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Absorb LMS.

2. Haga clic en el botón **Cuenta** que se encuentra en la esquina superior derecha.

    ![Botón de cuenta](./media/absorblms-tutorial/1.png)

3. En el panel Cuenta, seleccione **Configuración del portal**.

    ![Vínculo de configuración del portal](./media/absorblms-tutorial/2.png)

4. Seleccione la pestaña **Manage SSO Settings** (Administrar configuración de SSO).

    ![La pestaña Usuarios](./media/absorblms-tutorial/managesso.png)

5. Siga estos pasos en la página **Manage Single Sign-On Settings** (Administrar configuración de SSO).

    ![Página de configuración de inicio de sesión único](./media/absorblms-tutorial/settings.png)

    a. En el cuadro de texto **Nombre**, escriba el nombre como inicio de sesión único de Azure AD Marketplace.

    b. Seleccione **SAML** como **Método**.

    c. En el Bloc de notas, abra el certificado que descargó desde Azure Portal. Quite las etiquetas **---BEGIN CERTIFICATE---** y **---END CERTIFICATE---**. A continuación, pegue el resto del contenido en el cuadro de texto **Clave**.

    d. En el cuadro **Modo**, seleccione **Identity Provider Initiated (Iniciado por el proveedor de identidades)**.

    e. En el cuadro de texto **Propiedad de id.**, seleccione el atributo adecuado que ha configurado como identificador de usuario en Azure AD. Por ejemplo, si se selecciona *nameidentifier* en Azure AD, deberá seleccionar **Nombre de usuario**.

    f. Seleccione **Sha256** como **Tipo de firma**.

    g. En el cuadro **dirección URL de inicio de sesión**, pegue la **dirección URL de acceso de usuario** de la página **Propiedades** de la aplicación en Azure Portal.

    h. En **Dirección URL de cierre de sesión**, pegue el valor de la **dirección URL de cierre de sesión** que copió de la ventana **Configurar inicio de sesión** de Azure Portal.

    i. Cambie **Automatically Redirect** (Redirigir automáticamente) a **Activado**.

6. Seleccione **Guardar.**

    ![Opción Only Allow SSO Login (Solo permitir inicio de sesión SSO)](./media/absorblms-tutorial/save.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon\@yourcompanydomain.extension`.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Absorb LMS.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Absorb LMS**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Absorb LMS**.

    ![Vínculo a Absorb LMS en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-absorb-lms-test-user"></a>Creación de un usuario de prueba de Absorb LMS

Para permitir que los usuarios de Azure AD inicien sesión en Absorb LMS, tienen configurar Absorb LMS. En el caso de Absorb LMS, el aprovisionamiento es una tarea manual.

**Siga estos pasos para configurar el aprovisionamiento de usuario:**

1. Inicie sesión como administrador en el sitio de la compañía de Absorb LMS.

2. En el panel **Usuarios**, seleccione **Usuarios**.

    ![Vínculo de usuarios](./media/absorblms-tutorial/absorblms_userssub.png)

3. Seleccione la pestaña **Usuario**.

    ![Lista desplegable Agregar nuevo](./media/absorblms-tutorial/absorblms_createuser.png)

4. En la página **Agregar usuario**, siga estos pasos:

    ![página Agregar usuario](./media/absorblms-tutorial/user.png)

    a. En el cuadro de texto **Nombre**, escriba el nombre **Britta**.

    b. En el cuadro de texto **Apellido**, escriba el apellido **Simon**.

    c. En el cuadro de texto **Nombre de usuario**, escriba el nombre completo **Britta Simon**.

    d. En el cuadro de texto **Contraseña**, escriba la contraseña del usuario.

    e. En el cuadro **Confirmar contraseña** , vuelva a escribir la contraseña.

    f. Establezca la opción **Está activo** en **Activo**.

5. Seleccione **Guardar.**

    ![Opción Only Allow SSO Login (Solo permitir inicio de sesión SSO)](./media/absorblms-tutorial/save.png)

    > [!NOTE]
    > De forma predeterminada, el aprovisionamiento de usuarios no está habilitado en el inicio de sesión único. Si el cliente desea habilitar esta característica, debe establecerla como se mencionó en [esta](https://support.absorblms.com/hc/en-us/articles/360014083294-Incoming-SAML-2-0-SSO-Account-Provisioning) documentación. También tenga en cuenta que el aprovisionamiento de usuario solo está disponible en **Absorb 5 - New Learner Experience** (Absorb 5 - Nueva experiencia de aprendiz) con la dirección URL de ACS: `https://company.myabsorb.com/api/rest/v2/authentication/saml`

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Absorb LMS en el panel de acceso, debería iniciar sesión automáticamente en la versión de Absorb LMS para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
