---
title: 'Tutorial: Integración de Azure Active Directory con SAP Analytics Cloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAP Analytics Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/17/2019
ms.author: jeedes
ms.openlocfilehash: 9edbb499788b61135d761c5dfa69ce42c9275932
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548759"
---
# <a name="tutorial-integrate-sap-analytics-cloud-with-azure-active-directory"></a>Tutorial: Integración de SAP Analytics Cloud con Azure Active Directory

En este tutorial, aprenderá a integrar SAP Analytics Cloud con Azure Active Directory (Azure AD). Al integrar SAP Analytics Cloud con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SAP Analytics Cloud.
* Permitir que los usuarios puedan iniciar sesión automáticamente en SAP Analytics Cloud con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción de SAP Analytics Cloud con el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAP Analytics Cloud admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-sap-analytics-cloud-from-the-gallery"></a>Adición de SAP Analytics Cloud desde la galería

Para configurar la integración de SAP Analytics Cloud en Azure AD, necesita agregar SAP Analytics Cloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SAP Analytics Cloud** en el cuadro de búsqueda.
1. Seleccione **SAP Analytics Cloud** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con SAP Analytics Cloud utilizando un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SAP Analytics Cloud.

Para configurar y probar el inicio de sesión único de Azure AD con SAP Analytics Cloud, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración del inicio de sesión único de SAP Analytics Cloud](#configure-sap-analytics-cloud-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en SAP Analytics Cloud](#create-sap-analytics-cloud-test-user)** , para tener un homólogo de B.Simon en SAP Analytics Cloud que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **SAP Analytics Cloud**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    - `https://<sub-domain>.sapanalytics.cloud/`
    - `https://<sub-domain>.sapbusinessobjects.cloud/`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón:

    - `<sub-domain>.sapbusinessobjects.cloud`
    - `<sub-domain>.sapanalytics.cloud`

    > [!NOTE] 
    > Los valores de estas direcciones URL se muestran solo con fines demostrativos. Debe actualizarlos con las direcciones URL reales de inicio de sesión único y de identificador. Para obtener la dirección URL de inicio de sesión, póngase en contacto con el [equipo de soporte técnico de SAP Analytics Cloud](https://help.sap.com/viewer/product/SAP_BusinessObjects_Cloud/release/). Para obtener la dirección URL de identificador, descargue los metadatos de SAP Analytics Cloud desde la consola de administración. Esto se explica posteriormente en este tutorial.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up SAP Analytics Cloud** (Configurar SAP Analytics Cloud), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-sap-analytics-cloud-sso"></a>Configuración del inicio de sesión único en SAP Analytics Cloud

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de SAP Analytics Cloud como administrador.

2. Seleccione **Menu** > **System** > **Administration** (Menú > Sistema > Administración).
    
    ![Seleccionar Menu (Menú), System (Sistema) y, luego, Administration (Administración)](./media/sapboc-tutorial/config1.png)

3. En la pestaña **Security** (Seguridad), seleccione el icono **Edit** (Editar) (lápiz).
    
    ![En la pestaña Security (Seguridad), seleccionar el icono Edit (Editar)](./media/sapboc-tutorial/config2.png)  

4. Como **Authentication Method** (Método de autenticación), seleccione **SAML Single Sign-On (SSO)** (Inicio de sesión único [SSO] de SAML).

    ![Seleccionar SAML Single Sign-On (SSO) (Inicio de sesión único [SSO] de SAML) como método de autenticación](./media/sapboc-tutorial/config3.png)  

5. Para descargar los metadatos del proveedor de servicios (Paso 1), haga clic en **Download** (Descargar). En el archivo de metadatos, busque y copie el valor **entityID**. En Azure Portal, en el cuadro de diálogo **Configuración básica de SAML**, pegue el valor en el cuadro **Identificador**.

    ![Copiar y pegar el valor entityID](./media/sapboc-tutorial/config4.png)  

6. Para cargar los metadatos del proveedor de servicios (Paso 2) en el archivo que descargó desde Azure Portal en **Upload Identity Provider metadata** (Cargar metadatos del proveedor de identidades), seleccione **Upload** (Cargar).  

    ![En Upload Identity Provider metadata (Cargar metadatos del proveedor de identidades), seleccionar Upload (Cargar)](./media/sapboc-tutorial/config5.png)

7. En la lista **User Attribute** (Atributo de usuario), seleccione el atributo de usuario (Paso 3) que quiere usar en su implementación. Este atributo de usuario se asigna al proveedor de identidades. Para escribir un atributo personalizado en la página del usuario, use la opción **Custom SAML Mapping** (Personalizar asignación de SAML). O bien, puede seleccionar **Email** (Correo electrónico) o **USER ID** (Id. de usuario) como atributo de usuario. En nuestro ejemplo, hemos seleccionado **Email** (Correo electrónico) porque hemos asignado la notificación de identificador de usuario con el atributo **userprincipalname** en la sección **Atributos y notificaciones de usuario** de Azure Portal. Esto proporciona un correo electrónico de usuario único, que se envía a la aplicación SAP Analytics Cloud en cada respuesta SAML correcta.

    ![Seleccionar Atributos de usuario](./media/sapboc-tutorial/config6.png)

8. Para comprobar la cuenta con el proveedor de identidades (Paso 4), en el cuadro **Login Credential (Email)** [Credenciales de inicio de sesión (correo electrónico)], escriba la dirección de correo electrónico del usuario. A continuación, seleccione **Verify Account** (Comprobar cuenta). El sistema agrega credenciales de inicio de sesión a la cuenta de usuario.

    ![Escribir Email (Correo electrónico) y seleccionar Verify Account (Comprobar cuenta)](./media/sapboc-tutorial/config7.png)

9. Seleccione el icono **Save** (Guardar).

    ![Icono Save (Guardar)](./media/sapboc-tutorial/save.png)

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

En esta sección, va a conceder acceso a B.Simon a SAP Analytics Cloud para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SAP Analytics Cloud**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-sap-analytics-cloud-test-user"></a>Creación de un usuario de prueba de SAP Analytics Cloud

Se debe aprovisionar a los usuarios de Azure AD en SAP Analytics Cloud antes de que puedan iniciar sesión en SAP Analytics Cloud. En SAP Analytics Cloud, el aprovisionamiento es una tarea manual.

Para aprovisionar una cuenta de usuario:

1. Inicie sesión como administrador en su sitio de la compañía de SAP Analytics Cloud.

2. Seleccione **Menu** > **Security** > **Users** (Menú > Seguridad > Usuarios).

    ![Agregar empleado](./media/sapboc-tutorial/user1.png)

3. En la página **Users** (Usuarios), para agregar nuevos detalles de usuario, seleccione **+** . 

    ![Página Add Users (Agregar usuarios)](./media/sapboc-tutorial/user4.png)

    Después, complete los siguientes pasos:

    a. En el cuadro **USER ID** (Id. de usuario), escriba el identificador del usuario, en este caso **B**.

    b. En el cuadro **FIRST NAME** (Nombre), escriba el nombre del usuario, en este caso **B**.

    c. En el cuadro **LAST NAME** (Apellido), escriba el apellido del usuario, en este caso **Simon**.

    d. En el cuadro **DISPLAY NAME** (Nombre para mostrar), escriba el nombre completo del usuario, en este caso **B.Simon**.

    e. En el cuadro **E-MAIL** (Correo electrónico), escriba la dirección de correo electrónico del usuario, en este caso `b.simon@contoso.com`.

    f. En la página **Select Roles** (Seleccionar roles), seleccione el rol adecuado para el usuario y, luego, **OK** (Aceptar).

      ![Seleccionar rol](./media/sapboc-tutorial/user3.png)

    g. Seleccione el icono **Save** (Guardar).

### <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SAP Analytics Cloud del panel de acceso, debería iniciar sesión automáticamente en la aplicación SAP Analytics Cloud para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

