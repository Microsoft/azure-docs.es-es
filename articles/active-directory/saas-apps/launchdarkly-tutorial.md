---
title: 'Tutorial: Integración de Azure Active Directory con LaunchDarkly | Microsoft Docs'
description: Obtenga información para configurar el inicio de sesión único entre Azure Active Directory y LaunchDarkly.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/02/2019
ms.author: jeedes
ms.openlocfilehash: bae3f4a0d8c793061bb3833b04717d378e86f85f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88519895"
---
# <a name="tutorial-azure-active-directory-integration-with-launchdarkly"></a>Tutorial: Integración de Azure Active Directory con LaunchDarkly

En este tutorial se aprende a integrar LaunchDarkly con Azure Active Directory (Azure AD).
La integración de LaunchDarkly con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a LaunchDarkly.
* Puede permitir que los usuarios inicien sesión automáticamente en LaunchDarkly (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con LaunchDarkly, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en LaunchDarkly

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* LaunchDarkly admite SSO iniciado por **SP e IDP**
* LaunchDarkly admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-launchdarkly-from-the-gallery"></a>Incorporación de LaunchDarkly desde la galería

Para configurar la integración de LaunchDarkly en Azure AD, debe agregar LaunchDarkly desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar LaunchDarkly desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **LaunchDarkly**, seleccione **LaunchDarkly** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

     ![LaunchDarkly en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con [nombre de aplicación] utilizando usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de [nombre de aplicación].

Para configurar y probar el inicio de sesión único de Azure AD con [nombre de aplicación], es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de LaunchDarkly](#configure-launchdarkly-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de LaunchDarkly](#create-launchdarkly-test-user)** : para tener un homólogo de Britta Simon en LaunchDarkly que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con [nombre de aplicación], siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **LaunchDarkly**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de LaunchDarkly](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL: `app.launchdarkly.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://app.launchdarkly.com/trust/saml2/acs/<customers-unique-id>`

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. El valor se actualizará con la dirección URL de respuesta real, que se explica más adelante en el tutorial. Si intenta usar la aplicación en el modo **IDP**, debe dejar en blanco el campo **Dirección URL de inicio de sesión**; de lo contrario, no podrá empezar el inicio de sesión desde **IDP**. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://app.launchdarkly.com`

    ![Información de dominio y direcciones URL de inicio de sesión único de LaunchDarkly](common/metadata-upload-additional-signon.png)

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up LaunchDarkly** (Configurar LaunchDarkly), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-launchdarkly-single-sign-on"></a>Configuración del inicio de sesión único de LaunchDarkly

1. En otra ventana del explorador web, inicie sesión en el sitio de LaunchDarkly de la compañía como administrador.

2. Seleccione **Configuración de cuenta** en el panel de navegación izquierdo.

    ![Configuración de LaunchDarkly](./media/launchdarkly-tutorial/configure1.png)

3. Haga clic en la pestaña **Seguridad**.

    ![Configuración de LaunchDarkly](./media/launchdarkly-tutorial/configure2.png)

4. Haga clic en **HABILITAR SSO** y luego en **EDITAR CONFIGURACIÓN DE SAML**.

    ![Configuración de LaunchDarkly](./media/launchdarkly-tutorial/configure3.png)

5. En la sección **Editar configuración de SAML**, realice los siguientes pasos:

    ![Configuración de LaunchDarkly](./media/launchdarkly-tutorial/configure4.png)

    a. Copie la **dirección URL del servicio del consumidor de SAML** de la instancia y péguela en el cuadro de texto URL de respuesta de la sección **Dominio y direcciones URL de LaunchDarkly** de Azure Portal.

    b. En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el certificado descargado desde Azure Portal en el Bloc de notas, copie el contenido y luego péguelo en el cuadro **Certificado X.509** o cárguelo directamente al hacer clic **Cargar**.

    d. Haga clic en **Guardar**

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

En esta sección se habilita a Britta Simon para que use el inicio de sesión único de Azure al concederle acceso a LaunchDarkly.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **LaunchDarkly**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **LaunchDarkly**.

    ![Vínculo a LaunchDarkly en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-launchdarkly-test-user"></a>Creación de un usuario de prueba de LaunchDarkly

El objetivo de esta sección es crear un usuario de prueba denominado Britta Simon en LaunchDarkly. LaunchDarkly admite el aprovisionamiento Just-In-Time, que está habilitado de manera predeterminada. No hay ningún elemento de acción para usted en esta sección. Durante un intento de acceder a LaunchDarkly, se crea un usuario, en caso de que no exista.

> [!Note]
> Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el  [equipo de soporte técnico de LaunchDarkly](mailto:support@launchdarkly.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de LaunchDarkly en el panel de acceso y debería iniciar sesión automáticamente en la versión de LaunchDarkly para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
