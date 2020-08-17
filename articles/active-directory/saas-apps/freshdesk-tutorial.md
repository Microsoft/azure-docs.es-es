---
title: 'Tutorial: Integración de Azure Active Directory con Freshdesk | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y FreshDesk.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: c2a3e5aa-7b5a-4fe4-9285-45dbe6e8efcc
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b7188708c44d00d075a02d54b0a3b10cbc81954
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513594"
---
# <a name="tutorial-azure-active-directory-integration-with-freshdesk"></a>Tutorial: Integración de Azure Active Directory con Freshdesk

En este tutorial, obtendrá información sobre cómo integrar FreshDesk con Azure Active Directory (Azure AD).
La integración de FreshDesk con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a FreshDesk.
* Puede permitir que los usuarios inicien sesión automáticamente en FreshDesk (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con FreshDesk, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en FreshDesk

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* FreshDesk admite el inicio de sesión único iniciado por **SP**.
* Una vez configurado FreshDesk, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-freshdesk-from-the-gallery"></a>Agregar FreshDesk desde la galería

Para configurar la integración de FreshDesk en Azure AD, deberá agregar FreshDesk desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **FreshDesk** en el cuadro de búsqueda.
1. Seleccione **FreshDesk** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-freshdesk"></a>Configuración y prueba del inicio de sesión único de Azure AD para FreshDesk

Configure y pruebe el inicio de sesión único de Azure AD con FreshDesk mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de FreshDesk.

Para configurar y probar el inicio de sesión único de Azure AD con FreshDesk, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de FreshDesk](#configure-freshdesk-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de FreshDesk](#create-freshdesk-test-user)** : para tener un homólogo de Britta Simon en FreshDesk que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **FreshDesk**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenant-name>.freshdesk.com` u otro valor que Freshdesk haya sugerido.

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<tenant-name>.freshdesk.com` u otro valor que Freshdesk haya sugerido.

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de FreshDesk](https://freshdesk.com/helpdesk-software?utm_source=Google-AdWords&utm_medium=Search-IND-Brand&utm_campaign=Search-IND-Brand&utm_term=freshdesk&device=c&gclid=COSH2_LH7NICFVUDvAodBPgBZg). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación FreshDesk espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. En la captura de pantalla siguiente se muestra la lista de atributos predeterminados; aunque al **identificador de usuario único** se le asigna **user.userprincipalname**, FreshDesk supone que a esta notificación se le asignará **user.mail**, por lo que debe editar la asignación de atributos haciendo clic en el icono Editar y cambiarla.

    ![imagen](common/edit-attribute.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar FreshDesk**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure, para lo cual le concederá acceso a FreshDesk.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **FreshDesk**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **FreshDesk**.

    ![Vínculo de FreshDesk en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-freshdesk-single-sign-on"></a>Configuración del inicio de sesión único de FreshDesk

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Freshdesk como administrador.

2. Seleccione el icono **Security** (Seguridad) y, en la sección **Security** (Seguridad), siga estos pasos:

    ![Inicio de sesión único](./media/freshdesk-tutorial/configure-1.png "Inicio de sesión único")
  
    a. En **Single Sign On** (Inicio de sesión único), seleccione **On** (Activado).

    b. En **Login Method** (Método de inicio de sesión), seleccione **SAML SSO** (SSO de SAML).

    c. En el cuadro de texto **Entity ID provided by the IdP** (Identificador de entidad proporcionado por el IdP), pegue el valor de **Entity ID** (Id. de entidad) que ha copiado de Azure Portal.

    d. En el cuadro de texto **SAML SSO URL** (Dirección URL de inicio de sesión único de SAML), pegue el valor de **URL de inicio de sesión** que copió en Azure Portal.

    e. En **Signing Options** (Opciones de firma), seleccione **Only Signed Assertions** (Solo aserciones firmadas) en la lista desplegable.

    f. En el cuadro de texto **Sign Out URL** (URL de cierre de sesión), pegue el valor de **Logout URL** (Dirección URL de cierre de sesión) que ha copiado de Azure Portal.

    g. En el cuadro de texto **Security Certificate** (Certificado de seguridad), pegue el valor de **Certificate (Base64)** (Certificado [Base64]), que ha obtenido anteriormente.
  
    h. Haga clic en **Save**(Guardar).

## <a name="create-freshdesk-test-user"></a>Creación de un usuario de prueba de FreshDesk

Para permitir que los usuarios de Azure AD inicien sesión en FreshDesk, deben aprovisionarse en FreshDesk.  
En el caso de FreshDesk, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **Freshdesk** .

1. En el menú de la izquierda, haga clic en **Admin** (Administrador) y, en la pestaña **General Settings** (Configuración general), haga clic en **Agents** (Agentes).
  
    ![Agentes](./media/freshdesk-tutorial/create-user-1.png "Agentes")

1. Haga clic en **Nuevo agente**.

    ![New Agent (Nuevo agente)](./media/freshdesk-tutorial/create-user-2.png "Nuevo agente")

1. En el cuadro de diálogo Agent Information (Información del agente), rellene los campos obligatorios y haga clic en **Create agent** (Crear agente).

    ![Agent Information (Información de agente)](./media/freshdesk-tutorial/create-user-3.png "Información de agente")

    >[!NOTE]
    >El titular de la cuenta de Azure AD recibirá un mensaje de correo electrónico que incluye un vínculo para confirmar la cuenta antes de que se active.
    >
    >[!NOTE]
    >Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Freshdesk que proporcione Freshdesk para aprovisionar cuentas de usuario de Azure AD en FreshDesk.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de FreshDesk en el Panel de acceso, iniciará sesión automáticamente en la versión de FreshDesk para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

