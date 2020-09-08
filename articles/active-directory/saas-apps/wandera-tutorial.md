---
title: 'Tutorial: Integración de Azure Active Directory con Wandera RADAR Admin | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Wandera RADAR Admin.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/27/2020
ms.author: jeedes
ms.openlocfilehash: c8ec4b68dc774b52ad621ff3e965481de680b6ff
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89080006"
---
# <a name="tutorial-integrate-wandera-radar-admin-with-azure-active-directory"></a>Tutorial: Integración de Wandera RADAR Admin con Azure Active Directory

En este tutorial, aprenderá a integrar Wandera RADAR Admin con Azure Active Directory (Azure AD). Al integrar Wandera RADAR Admin con Azure AD, se puede:

* Controlar en Azure AD quién tiene acceso a Wandera RADAR Admin.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Wandera RADAR Admin con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Wandera RADAR Admin.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* El Wandera RADAR Admin admite el inicio de sesión único iniciado por el **proveedor de identidades**.
* Una vez que ha configurado Wandera RADAR Admin, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-wandera-radar-admin-from-the-gallery"></a>Adición de Wandera RADAR Admin desde la galería

Para configurar la integración de Wandera RADAR Admin en Azure AD, es preciso agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Wandera RADAR Admin** en el cuadro de búsqueda.
1. Seleccione **Wandera RADAR Admin** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Wandera RADAR Admin mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Wandera RADAR Admin.

Para configurar y probar el inicio de sesión único de Azure AD con Wandera RADAR Admin, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Wandera RADAR Admin](#configure-wandera-radar-admin-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
   * **[Creación de un usuario de prueba de Wandera RADAR Admin](#create-wandera-radar-admin-test-user)** , para tener un homólogo de B.Simon en Wandera RADAR Admin que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Wandera RADAR Admin**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://radar.wandera.com/saml/acs/<tenant id>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de respuesta real. Póngase en contacto con el [equipo de soporte técnico al cliente de Wandera RADAR Admin](https://www.wandera.com/about-wandera/contact/#supportsection) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Certificado de firma de SAML** y modificar la configuración.

    ![Opción de firma](common/signing-option.png)

    1. Seleccione **Opción de firma** como **Firmar respuesta y aserción SAML**.

    1. Seleccione **Algoritmo de firma** como **SHA-256**.

1. En la sección **Configurar Wandera RADAR Admin** , copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Wandera RADAR Admin mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Wandera RADAR Admin**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-wandera-radar-admin-sso"></a>Configuración del inicio de sesión único de Wandera RADAR Admin

1. Para automatizar la configuración en Wandera RADAR Admin, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Wandera RADAR Admin** para ir a la aplicación Wandera RADAR Admin. En ella, escriba las credenciales de administrador para iniciar sesión en Wandera RADAR Admin. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 y 4.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Wandera RADAR Admin manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Wandera RADAR Admin como administrador y haga lo siguiente:

4. En la esquina superior derecha de la página, haga clic en **Configuración** > **Administración** > **Inicio de sesión único** y, a continuación, active la opción **Habilitar SAML 2.0** para realizar los pasos siguientes.

    ![Configuración de Wandera RADAR Admin](./media/wandera-tutorial/config01.png)

    a. Haga clic en **O bien escriba manualmente los campos requeridos**.

    b. En el cuadro de texto **IdP EntityId** (Id. de entidad de IdP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    c. Abra el XML de metadatos de federación en el Bloc de notas, copie su contenido y péguelo en el cuadro de texto **IdP Public X.509 Certificate** (Certificado X.509 público de IdP).

    d. Haga clic en **Save**(Guardar).

### <a name="create-wandera-radar-admin-test-user"></a>Creación de un usuario de prueba de Wandera RADAR Admin

En esta sección, creará un usuario llamado B.Simon en Wandera RADAR Admin. Colabore con el  [equipo de soporte técnico de Wandera RADAR Admin](https://www.wandera.com/about-wandera/contact/#supportsection) para agregar los usuarios en la plataforma de Wandera RADAR Admin. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Wandera RADAR Admin en el panel de acceso, debería iniciar sesión automáticamente en la versión de Wandera RADAR Admin para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

