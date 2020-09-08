---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SAP Cloud for Customer | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAP Cloud for Customer.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/20/2019
ms.author: jeedes
ms.openlocfilehash: f9fd458ea19fa0dad2f630f94a67d5e1db96cee3
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543319"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-cloud-for-customer"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SAP Cloud for Customer

En este tutorial aprenderá a integrar SAP Cloud for Customer con Azure Active Directory (Azure AD). Al integrar SAP Cloud for Customer con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SAP Cloud for Customer.
* Permitir que los usuarios inicien sesión automáticamente en SAP Cloud for Customer con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en SAP Cloud for Customer.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAP Cloud for Customer el inicio de sesión único iniciado por **PS**

## <a name="adding-sap-cloud-for-customer-from-the-gallery"></a>Adición de SAP Cloud for Customer desde la galería

Para configurar la integración de SAP Cloud for Customer en Azure AD, es preciso agregar SAP Cloud for Customer desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SAP Cloud for Customer** en el cuadro de búsqueda.
1. Seleccione **SAP Cloud for Customer** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sap-cloud-for-customer"></a>Configuración y prueba del inicio de sesión único de Azure AD para SAP Cloud for Customer

Configure y pruebe el inicio de sesión único de Azure AD con SAP Cloud for Customer mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SAP Cloud for Customer.

Para configurar y probar el inicio de sesión único de Azure AD con SAP Cloud for Customer, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de SAP Cloud for Customer](#configure-sap-cloud-for-customer-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SAP Cloud for Customer](#create-sap-cloud-for-customer-test-user)** , para tener un homólogo de B.Simon en SAP Cloud for Customer que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **SAP Cloud for Customer**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<server name>.crm.ondemand.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<server name>.crm.ondemand.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de SAP Cloud for Customer](https://www.sap.com/about/agreements.sap-cloud-services-customers.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación SAP Cloud for Customer espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

1. En la sección **Atributos del usuario** del cuadro de diálogo **Atributos y notificaciones de usuario**, siga estos pasos:

    a. Haga clic en el **icono Editar** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](./media/sap-customer-cloud-tutorial/tutorial_usermail.png)

    ![imagen](./media/sap-customer-cloud-tutorial/tutorial_usermailedit.png)

    b. Seleccione **Transformación** como **origen**.

    c. En la lista **Transformación**, seleccione **ExtractMailPrefix()** .

    d. En la lista **Parámetro 1**, seleccione el atributo de usuario que desea usar en la implementación.
    Por ejemplo, si quiere usar EmployeeID como identificador de usuario único y ha almacenado el valor del atributo en ExtensionAttribute2, seleccione user.extensionattribute2.

    e. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar SAP Cloud for Customer**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a SAP Cloud for Customer mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SAP Cloud for Customer**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sap-cloud-for-customer-sso"></a>Configuración del inicio de sesión único de SAP Cloud for Customer

1. Abra una ventana del explorador web e inicie sesión en el sitio de empresa de SAP Cloud for Customer como administrador.

2. En el lado izquierdo del menú, haga clic en  **Identity Providers** > **Corporate Identity Providers** > **Add** (Proveedores de identidades > Proveedores de identidades corporativas > Agregar) y, en el elemento emergente, agregue el nombre del proveedor de identidades como **Azure AD**; después, haga clic en **Save** (Guardar) y en **SAML 2.0 Configuration** (Configuración de SAML 2.0).

    ![Configuración de SAP](./media/sap-customer-cloud-tutorial/configure01.png)

3. En la sección **SAML 2.0 Configuration** (Configuración de SAML 2.0), realice los pasos siguientes:

    ![Configuración de SAP](./media/sap-customer-cloud-tutorial/configure02.png)

    a. Haga clic en **Browse** (Examinar) para cargar el archivo XML de metadatos de federación que descargó de Azure Portal.

    b. Cuando el archivo XML se haya cargado correctamente, se rellenarán de forma automática los valores siguientes; después, haga clic en **Save** (Guardar).

### <a name="create-sap-cloud-for-customer-test-user"></a>Creación de un usuario de prueba de SAP Cloud for Customer

Para permitir que los usuarios de Azure AD inicien sesión en SAP Cloud for Customer, deben aprovisionarse en esta última aplicación. En SAP Cloud for Customer, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en SAP Cloud for Customer como administrador de seguridad.

2. En el lado izquierdo del menú, haga clic en  **Users & Authorizations** > **User administración** > **Add User** (Usuarios y autorizaciones > Administración de usuarios > Agregar usuario).

    ![Configuración de SAP](./media/sap-customer-cloud-tutorial/configure03.png)

3. En la sección **Add New User** (Agregar nuevo usuario), lleve a cabo estos pasos:

    ![Configuración de SAP](./media/sap-customer-cloud-tutorial/configure04.png)

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre del usuario, en este caso **B**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba el apellido del usuario, en este caso **Simon**.

    c. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario, en este caso `B.Simon@contoso.com`.

    d. En el cuadro de texto **Login Name** (Nombre de inicio de sesión), escriba el nombre de usuario de la siguiente manera: **B.Simon**.

    e. Seleccione el valor de **User Type** (Tipo de usuario) adecuado a sus necesidades.

    f. Seleccione la opción de **Account Activation** (Activación de cuentas) según sus necesidades.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SAP Cloud for Customer del panel de acceso, debería iniciar sesión automáticamente en la aplicación SAP Cloud for Customer para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe SAP Cloud for Customer con Azure AD](https://aad.portal.azure.com/)

