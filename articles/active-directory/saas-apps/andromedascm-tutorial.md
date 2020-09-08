---
title: 'Tutorial: Integración de Azure Active Directory con Andromeda | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Andromeda.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.openlocfilehash: 31fc1877dd0edd2fd670f0f9466692ac7bc8ad15
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88531198"
---
# <a name="tutorial-azure-active-directory-integration-with-andromeda"></a>Tutorial: Integración de Azure Active Directory con Andromeda

En este tutorial, aprenderá a integrar Andromeda con Azure Active Directory (Azure AD).
La integración de Andromeda con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Andromeda.
* Puede permitir que los usuarios inicien sesión automáticamente en Andromeda (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Andromeda, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción que permita el inicio de sesión único en Andromeda

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Andromeda admite el inicio de sesión único iniciado por **SP e IDP**
* Andromeda admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-andromeda-from-the-gallery"></a>Adición de Andromeda desde la galería

Para configurar la integración de Andromeda en Azure AD, será preciso que agregue Andromeda desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Andromeda desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Andromeda**, seleccione **Andromeda** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Andromeda en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Andromeda con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Andromeda.

Para configurar y probar el inicio de sesión único de Azure AD con Andromeda, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Andromeda](#configure-andromeda-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Andromeda](#create-andromeda-test-user)** : para tener un homólogo de Britta Simon en Andromeda que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Andromeda, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Andromeda**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Andromeda](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<tenantURL>.ngcxpress.com/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<tenantURL>.ngcxpress.com/SAMLConsumer.aspx`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de Andromeda](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenantURL>.ngcxpress.com/SAMLLogon.aspx`

    > [!NOTE]
    > Estos valores no son reales. El valor se actualizará con la dirección URL de inicio de sesión, la dirección URL de respuesta y el identificador reales, que se explican más adelante en el tutorial.

6. La aplicación Andromeda espera las aserciones de SAML en un formato específico. Configure las siguientes notificaciones para esta aplicación. Puede administrar los valores de estos atributos en la sección **Atributos de usuario** de la página de integración de aplicaciones. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![imagen](common/edit-attribute.png)

    > [!Important]
    > Desactive las definiciones de espacio de nombres al realizar esta configuración.

7. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, edite las notificaciones mediante el **icono Editar** o agregue notificaciones mediante **Agregar nueva notificación** para configurar el atributo Token SAML como muestra la imagen anterior y realice los siguientes pasos: 

    | Nombre | Atributo de origen|
    | ------ | -----------|
    | rol        | Rol específico de la aplicación |
    | type        | Tipo de aplicación |
    | company       | CompanyName |

    > [!NOTE]
    > Estos valores no son reales. Se facilitan solo con fines de demostración, por lo que debe usar los roles de su organización.

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](common/new-save-attribute.png)

    ![imagen](common/new-attribute-details.png)

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    f. Haga clic en **Aceptar**.

    g. Haga clic en **Save**(Guardar).

8. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

9. En la sección **Set up Andromeda** (Configurar Andromeda), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-andromeda-single-sign-on"></a>Configuración del inicio de sesión único en Andromeda

1. Inicie sesión como administrador en el sitio de la empresa Andromeda.

2. En la parte superior de la barra de menús, haga clic en **Admin** (Administrador) y vaya a **Administration** (Administración).

    ![Administración de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_admin.png)

3. En el lado izquierdo de la barra de herramientas, en la sección **Interfaces** (Interfaces), haga clic en **SAML Configuration** (Configuración de SAML).

    ![SAML de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_saml.png)

4. En la página **SAML Configuration** (Configuración de SAML), realice los siguientes pasos:

    ![Configuración de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_config.png)

    a. Marque **Enable SSO with SAML** (Habilitar SSO con SAML).

    b. En la sección **Andromeda Information** (Información de Andromeda), copie el valor **SP Identity** (Identidad de SP) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML**.

    c. Copie el valor de **Consumer URL** (Dirección URL de consumidor) y péguelo en el cuadro de texto **Reply URL** (URL de respuesta) de la sección **Configuración básica de SAML**.

    d. Copie el valor de **Logon URL** (Dirección URL de inicio de sesión) y péguelo en el cuadro de texto **Sign-on URL** (URL de inicio de sesión) de la sección **Configuración básica de SAML**.

    e. En la sección **SAML Identity Provider** (Proveedor de identidades SAML), escriba el nombre del proveedor de identidades.

    f. En el cuadro de texto **Single Sign On End Point** (Punto de conexión del inicio de sesión único), pegue el valor de **Login URL** (URL de inicio de sesión) que ha copiado de Azure Portal.

    g. Abra el **certificado codificado en Base64** descargado desde Azure Portal en el Bloc de notas y péguelo en el cuadro de texto **X 509 Certificate** (Certificado X509).
    
    h. Asigne los siguientes atributos con su respectivo valor para facilitar el inicio de sesión único desde Azure AD. El atributo **User ID** (Id. de usuario) es necesario para iniciar sesión. Para el aprovisionamiento, es necesario proporcionar los valores **Email** (Correo electrónico), **Company** (Compañía), **UserType** (Tipo de usuario) y **Role** (Rol). En esta sección, se define la asignación de atributos (nombre y valores) que se corresponden con los que se establecen en Azure Portal.

    ![Asignación de atributos de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_attbmap.png)

    i. Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Andromeda.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**,  **Andromeda**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Andromeda**.

    ![Vínculo a Andromeda en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-andromeda-test-user"></a>Creación de un usuario de prueba de Andromeda

En esta sección, se crea un usuario llamado Britta Simon en Andromeda. Andromeda admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe ya en Andromeda, se crea uno después de la autenticación. Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de Andromeda](https://www.ngcsoftware.com/support/).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Andromeda en el panel de acceso, debería iniciar sesión automáticamente en la versión de Andromeda para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)