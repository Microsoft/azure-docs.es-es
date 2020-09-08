---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con WEDO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y WEDO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/22/2020
ms.author: jeedes
ms.openlocfilehash: 364bb71ff9ec1cc948f9631f50ac4444c16ea4d1
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88523412"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-wedo"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con WEDO

En este tutorial, aprenderá a integrar WEDO con Azure Active Directory (Azure AD). Al integrar WEDO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a WEDO.
* Permitir que los usuarios inicien sesión automáticamente en WEDO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en WEDO. Póngase en contacto con el [equipo de soporte técnico al cliente de WEDO](mailto:info@wedo.swiss) para obtener esta suscripción.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* WEDO admite el inicio de sesión único iniciado por **SP e IDP**.

* [Una vez configurado WEDO, puede aplicar controles de sesión, que protegen la filtración e infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-wedo-from-the-gallery"></a>Incorporación de WEDO desde la galería

Para configurar la integración de WEDO en Azure AD, deberá agregar WEDO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **WEDO** en el cuadro de búsqueda.
1. Seleccione **WEDO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-wedo"></a>Configuración y prueba del inicio de sesión único de Azure AD para WEDO

Configure y pruebe el inicio de sesión único de Azure AD con WEDO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de WEDO.

Para configurar y probar el inicio de sesión único de Azure AD con WEDO, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de WEDO](#configure-wedo-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en WEDO](#create-wedo-test-user)** , para tener un homólogo de B.Simon en WEDO que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **WEDO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<SUBDOMAIN>.wedo.swiss/sp/acs`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.wedo.swiss/sp/acs`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.wedo.swiss/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico al cliente de WEDO](mailto:info@wedo.swiss) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación WEDO espera las aserciones de SAML en un formato específico, lo que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    | Nombre | Atributo de origen|
    | ------------ | --------- |
    | email | user.email |
    | firstName | user.firstName |
    | lastName | user.lasttName |
    | userName | user.userName |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar WEDO**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a WEDO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **WEDO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-wedo-sso"></a>Configuración del inicio de sesión único de WEDO

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en WEDO.

1. Inicie sesión en [WEDO](https://login.wedo.swiss/). Debe tener **rol de administrador**.
1. En la configuración del perfil, seleccione el menú **Authentication** (Autenticación) en la sección **Network settings** (Configuración de red).
1. En la página **SAML Authentication** (Autenticación SAML), realice los siguientes pasos:

   ![Vínculo de autenticación SAML](media/wedo-tutorial/network-security-authentification.png)

   a. Active la opción **Enable SAML Authentication** (Habilitar autenticación SAML).

   b. Seleccione la pestaña **Identity Provider metadata (XML)** (Metadatos del proveedor de identidades [XML]).

   c. Abra el archivo **XML de metadatos de federación** descargado de Azure Portal en el Bloc de notas, copie el contenido del XML de metadatos y péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509).

   d. Haga clic en **Guardar**

### <a name="create-wedo-test-user"></a>Creación de un usuario de prueba en WEDO

En esta sección, va a crear un usuario de prueba llamado Bob Simon en WEDO. La información debe coincidir con la indicada en *Creación de un usuario de prueba de Azure AD*.

1. En la configuración del perfil en WEDO, seleccione **Users** (Usuarios) en la sección *Network settings* (Configuración de red).
1. Haga clic en **Agregar usuario**.
1. En el menú emergente Add user (Agregar usuario), rellene la información del usuario:

    a. First name (Nombre): `B`.

    b. Last name (Apellidos): `Simon`.

    c. Escriba la dirección de correo electrónico `username@companydomain.extension`. Por ejemplo, `B.Simon@contoso.com`. Es obligatorio tener un correo electrónico con el mismo dominio que el nombre corto de la empresa.

    d. User type (Tipo de usuario): `User`.

    e. Haga clic en **Crear usuario**.

    f. En la página *Select teams* (Seleccionar equipos), haga clic en **Save** (Guardar).

    g.  En la página *Invite user* (Invitar usuario), haga clic en **Yes** (Sí).

1. Valide el usuario mediante el vínculo recibido por correo electrónico.

> [!NOTE]
> Si desea crear un usuario falso (el correo electrónico anterior no existe en la red), póngase en contacto con [nuestro equipo de soporte técnico](mailto:info@wedo.swiss) para validar al usuario*.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de WEDO en el Panel de acceso, debería iniciar sesión automáticamente en la versión de WEDO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe WEDO con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de WEDO con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
