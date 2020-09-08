---
title: 'Tutorial: Integración de Azure Active Directory con Kantega SSO para FishEye/Crucible | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Kantega SSO para FishEye/Crucible.
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
ms.openlocfilehash: ce5572c4c5aca6f48496250c942d50b90551f36f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546954"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-fisheyecrucible"></a>Tutorial: Integración de Azure Active Directory con Kantega SSO para FishEye/Crucible

En este tutorial obtendrá información sobre cómo integrar Kantega SSO para FishEye/Crucible con Azure Active Directory (Azure AD).
La integración de Kantega SSO para FishEye/Crucible con Azure AD le proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a Kantega SSO para FishEye/Crucible.
* Puede permitir que los usuarios inicien sesión automáticamente en Kantega SSO para FishEye/Crucible (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Kantega SSO para FishEye/Crucible, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Suscripción de Kantega SSO para FishEye/Crucible con el inicio de sesión único habilitado

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Kantega SSO para FishEye/Crucible admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-kantega-sso-for-fisheyecrucible-from-the-gallery"></a>Incorporación de Kantega SSO para FishEye/Crucible desde la galería

Para configurar la integración de Kantega SSO para FishEye/Crucible en Azure AD, tiene que agregar Kantega SSO para FishEye/Crucible desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Kantega SSO para FishEye/Crucible desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Kantega SSO para FishEye/Crucible**, seleccione **Kantega SSO para FishEye/Crucible** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![Kantega SSO para FishEye/Crucible en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Kantega SSO para FishEye/Crucible con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Kantega SSO para FishEye/Crucible.

Para configurar y probar el inicio de sesión único de Azure AD con Kantega SSO para FishEye/Crucible, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Kantega SSO para FishEye/Crucible](#configure-kantega-sso-for-fisheyecrucible-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Kantega SSO para FishEye/Crucible](#create-kantega-sso-for-fisheyecrucible-test-user)** : para tener un homólogo de Britta Simon en Kantega SSO para FishEye/Crucible que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Kantega SSO para FishEye/Crucible, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Kantega SSO para FishEye/Crucible**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Kantega SSO para FishEye/Crucible](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de Kantega SSO para FishEye/Crucible](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Estos valores se reciben durante la configuración del complemento de FishEye/Crucible, que se explica más adelante en el tutorial.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up Kantega SSO for FishEye/Crucible** (Configurar Kantega SSO para FishEye/Crucible), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-kantega-sso-for-fisheyecrucible-single-sign-on"></a>Configuración del inicio de sesión único de Kantega SSO para FishEye/Crucible

1. En otra ventana del explorador web, inicie sesión en el servidor local de FishEye/Crucible como administrador.

1. Mantenga el mouse encima del icono de engranaje y haga clic en **Complementos**.

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon1.png)

1. En la sección Configuración del sistema, haga clic en **Find new add-ons** (Buscar nuevos complementos). 

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/add-on2.png)

1. Busque **Kantega SSO para Crucible** y haga clic en el botón **Instalar** para instalar el nuevo complemento SAML.

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon2.png)

1. Se inicia la instalación del complemento. 

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon33.png)

1. Una vez completada la instalación. Haga clic en **Cerrar**.

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon34.png)

1.  Haga clic en **Administrar**.

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon35.png)

1. Haga clic en **Configurar** para configurar el nuevo complemento. 

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon3.png)

1. En la sección **SAML**. Seleccione **Azure Active Directory (Azure AD)** en la lista desplegable **Agregar proveedor de identidades**.

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon4.png)

1. Seleccione el nivel de suscripción **Básica**.

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon5.png)

1. En la sección **Agregar propiedades**, siga estos pasos:

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon6.png)

    a. Copie el valor de **URI de id. de aplicación** y úselo en los campos **Identificador, URL de respuesta y URL de inicio de sesión** de la sección **Configuración básica de SAML**  de Azure Portal.

    b. Haga clic en **Next**.

1. En la sección **Importar metadatos**, siga estos pasos:

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon7.png)

    a. Seleccione **Archivo de metadatos en el equipo** y cargue el archivo de metadatos que descargó desde Azure Portal.

    b. Haga clic en **Next**.

1. En la sección**Name and SSO location** (Nombre y ubicación de SSO), siga estos pasos:

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon8.png)

    a. Agregue el nombre del proveedor de identidades en el cuadro de texto **Nombre del proveedor de identidades** (por ejemplo, Azure AD).

    b. Haga clic en **Next**.

1. Compruebe el certificado de firma y haga clic en **Siguiente**.   

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon9.png)

1. En la sección **Cuentas de usuario de FishEye**, siga estos pasos:

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon10.png)

    a. Seleccione **Create users in FishEye's internal Directory if needed** (Crear usuarios en el directorio interno de FishEye si es necesario) y escriba el nombre adecuado del grupo de usuarios (puede ser un número múltiple de grupos separados por coma).

    b. Haga clic en **Next**.

1. Haga clic en **Finalizar**

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon11.png)

1. En la sección **Known domains for Azure AD** (Dominios conocidos para Azure AD), siga estos pasos:  

    ![Configurar inicio de sesión único](./media/kantegassoforfisheyecrucible-tutorial/addon12.png)

    a. Seleccione **Known domains** (Dominios conocidos) en el panel izquierdo de la página.

    b. Escriba el nombre de dominio en el cuadro de texto **Known domains** (Dominios conocidos).

    c. Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Kantega SSO para FishEye/Crucible.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Kantega SSO para FishEye/Crucible**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Kantega SSO para FishEye/Crucible**.

    ![Vínculo a Kantega SSO para FishEye/Crucible en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-kantega-sso-for-fisheyecrucible-test-user"></a>Creación de un usuario de prueba de Kantega SSO para FishEye/Crucible

Para permitir que los usuarios de Azure AD inicien sesión en FishEye/Crucible, deben aprovisionarse en FishEye/Crucible. En Kantega SSO para FishEye/Crucible, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el servidor local de Crucible.

1. Mantenga el mouse encima del icono de engranaje y haga clic en **Usuarios**.

    ![Agregar empleado](./media/kantegassoforfisheyecrucible-tutorial/user1.png)

1. En la sección de la pestaña **Usuarios**, haga clic en **Agregar usuario**.

    ![Agregar empleado](./media/kantegassoforfisheyecrucible-tutorial/user2.png)

1. En la página del cuadro de diálogo **Agregar nuevo usuario**, realice los pasos siguientes:

    ![Agregar empleado](./media/kantegassoforfisheyecrucible-tutorial/user3.png)

    a. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    b. En el cuadro de texto **Nombre para mostrar**, escriba el nombre para mostrar del usuario, como Britta Simon.

    c. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    d. En el cuadro de texto **Contraseña**, escriba la contraseña del usuario.

    e. En el cuadro de texto **Confirmar contraseña**, escriba nuevamente la contraseña de usuario.

    f. Haga clic en **Agregar**.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Kantega SSO para FishEye/Crucible en el panel de acceso, debería iniciar sesión automáticamente en la versión de Kantega SSO para FishEye/Crucible para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)