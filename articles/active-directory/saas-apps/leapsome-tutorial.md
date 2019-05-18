---
title: 'Tutorial: integración de Azure Active Directory con Leapsome | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Leapsome.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: cb523e97-add8-4289-b106-927bf1a02188
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/10/2019
ms.author: jeedes
ms.openlocfilehash: 602e3145a003a0413287b08151abf472ecf4ade0
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406720"
---
# <a name="tutorial-azure-active-directory-integration-with-leapsome"></a>Tutorial: integración de Azure Active Directory con Leapsome

En este tutorial, obtendrá información sobre cómo integrar Leapsome con Azure Active Directory (Azure AD).
La integración de Leapsome con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Leapsome.
* Puede permitir que los usuarios inicien sesión automáticamente en Leapsome (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Leapsome, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción de Leapsome habilitada para el inicio de sesión único

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Leapsome admite SSO iniciado por **SP e IDP**

## <a name="adding-leapsome-from-the-gallery"></a>Incorporación de Leapsome desde la galería

Para configurar la integración de Leapsome en Azure AD, deberá agregar Leapsome desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Leapsome desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Leapsome**, seleccione **Leapsome** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Leapsome en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Leapsome con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Leapsome.

Para configurar y probar el inicio de sesión único de Azure AD con Leapsome, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)**: para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Leapsome](#configure-leapsome-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**, para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**, para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Leapsome](#create-leapsome-test-user)**: para tener un homólogo de Britta Simon en Leapsome que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)**: para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Leapsome, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Leapsome**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Atlassian Cloud](common/idp-intiated.png)

     a. En el cuadro de texto **Identificador**, escriba una dirección URL: `https://www.leapsome.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.leapsome.com/api/users/auth/saml/<CLIENTID>/assert`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Atlassian Cloud](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.leapsome.com/api/users/auth/saml/<CLIENTID>/login`

    > [!NOTE]
    > El valor de URL de respuesta y URL de inicio de sesión no es un valor real. Estos se actualizarán con los valores reales, lo que se explica más adelante en el tutorial.

6. La aplicación Leapsome espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![imagen](common/edit-attribute.png)

7. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, edite las notificaciones mediante el **icono Editar** o agregue notificaciones mediante **Agregar nueva notificación** para configurar el atributo Token SAML como muestra la imagen anterior y realice los siguientes pasos: 

    | NOMBRE | Atributo de origen | Espacio de nombres |
    | ---------------| --------------- | --------- |  
    | firstname | user.givenname | https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | lastname | user.surname | https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | título | user.jobtitle | https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | picture | URL de una fotografía del empleado | https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | | |

    > [!Note]
    > El valor del atributo picture no es real. Actualice este valor con la URL de la imagen real. Para obtener este valor, póngase en contacto con el [equipo de soporte técnico de Leapsome](mailto:support@leapsome.com).

     a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](common/new-save-attribute.png)

    ![imagen](common/new-attribute-details.png)

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. En el cuadro de texto **Espacio de nombres**, escriba el URI de espacio de nombres para esa fila.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    f. Haga clic en **Aceptar**.

    g. Haga clic en **Save**(Guardar).

8. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

9. En la sección **ConfigurarLeapsome**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-leapsome-single-sign-on"></a>Configuración del inicio de sesión único de Leapsome

1. En otra ventana del explorador web, inicie sesión en Leapsome como administrador de seguridad.

1. En la parte superior derecha, haga clic en el logotipo de configuración y luego en **Configuración de administración**.

    ![Conjunto de Leapsome](./media/leapsome-tutorial/tutorial_leapsome_admin.png)

1. En la barra de menús izquierda, haga clic en **Inicio de sesión único (SSO)** y en la página **Inicio de sesión único basado en SAML (SSO)** siga estos pasos:

    ![SAML de Leapsome](./media/leapsome-tutorial/tutorial_leapsome_samlsettings.png)

     a. Seleccione **Enable SAML-based single sign-on** (Habilitar inicio de sesión único basado en SAML).

    b. Copie el valor **Login URL (point your users here to start login)** [URL de inicio de sesión (apuntar a los usuarios aquí para iniciar el inicio de sesión)] y péguelo en el cuadro de texto **URL de inicio de sesión** en la sección **Configuración básica de SAML** de Azure Portal.

    c. Copie el valor **Reply URL (receives response from your identity provider)** [URL de respuesta (recibe respuesta desde el proveedor de identidades)] y péguelo en el cuadro de texto **URL de respuesta** en la sección **Configuración básica de SAML** de Azure Portal.

    d. En el cuadro de texto **SSO Login URL (provided by identity provider)** [Dirección URL de inicio de sesión único (proporcionada por el proveedor de identidades)], pegue el valor de **URL de inicio de sesión**, que ha copiado de Azure Portal.

    e. Copie el certificado que ha descargado desde Azure Portal sin los comentarios `--BEGIN CERTIFICATE and END CERTIFICATE--` y péguelo en el cuadro de texto **Certificate (provided by identity provider)** [Certificado (proporcionado por el proveedor de identidades)].

    f. Haga clic en **UPDATE SSO SETTINGS** (ACTUALIZAR CONFIGURACIÓN DE SSO).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure mediante la concesión de acceso a Leapsome.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Leapsome**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Leapsome**.

    ![Vínculo a Leapsome en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-leapsome-test-user"></a>Creación de un usuario de prueba de Leapsome

En esta sección, creará un usuario llamado Britta Simon en Leapsome. Colabore con el [equipo de soporte técnico de Leapsome](mailto:support@leapsome.com) para agregar los usuarios o el dominio que debe incluirse en la lista blanca en la plataforma de Leapsome. Si el dominio lo agrega el equipo, los usuarios se aprovisionarán automáticamente en la plataforma de Leapsome. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Leapsome en el panel de acceso y debería iniciar sesión automáticamente en la versión de Leapsome para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)