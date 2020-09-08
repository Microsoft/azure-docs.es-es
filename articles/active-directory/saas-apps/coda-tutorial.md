---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Coda | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Coda.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/23/2020
ms.author: jeedes
ms.openlocfilehash: 718afec8b36a27bfd36e2a018b39f480144bf822
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545359"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-coda"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Coda

En este tutorial, aprenderá a integrar Coda con Azure Active Directory (Azure AD). Al integrar Coda con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Coda.
* Permitir que los usuarios inicien sesión automáticamente en Coda con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Coda (Enterprise) con la integración de GDrive deshabilitada. Póngase en contacto con el [equipo de soporte técnico de Coda](mailto:support@coda.io) para deshabilitar la integración de GDrive en la organización si actualmente está habilitada.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Coda admite inicio de sesión único iniciado por **IDP**

* Coda admite el aprovisionamiento de usuarios **Just-In-Time**

* Una vez configurado Coda, puede aplicar controles de sesión, que protegen la filtración e infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-coda-from-the-gallery"></a>Adición de Coda desde la galería

Para configurar la integración de Coda en Azure AD, es preciso agregar Coda desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Coda** en el cuadro de búsqueda.
1. Seleccione **Coda** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-coda"></a>Configuración y prueba del inicio de sesión único de Azure AD para Coda

Configure y pruebe el inicio de sesión único de Azure AD con Coda con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Coda.

Para configurar y probar el inicio de sesión único de Azure AD con Coda, es preciso completar los siguientes bloques de creación:

1. **[Inicio de la configuración del SSO de Coda](#begin-configuration-of-coda-sso)** , para comenzar la configuración del inicio de sesión único en Coda.
1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Coda](#configure-coda-sso)** , para completar la configuración de inicio de sesión único en Coda.
   * **[Creación de un usuario de prueba en Coda](#create-coda-test-user)** : para tener un homólogo de B.Simon en Coda vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="begin-configuration-of-coda-sso"></a>Inicio de la configuración del SSO de Coda

Siga estos pasos en Coda para comenzar.

1. En Coda, abra el panel de **Organization settings** (Configuración de la organización).

   ![Abra Organization settings (Configuración de la organización)](media/coda-tutorial/org-settings.png)

1. Asegúrese de que la organización tiene desactivada la integración de GDrive. Si está habilitada actualmente, póngase en contacto con el [equipo de soporte técnico de Coda](mailto:support@coda.io) para que le ayude a migrar de GDrive.

   ![GDrive deshabilitado](media/coda-tutorial/gdrive-off.png)

1. En **Authenticate with SSO (SAML)** (Autenticación con inicio de sesión único [SAML]), seleccione la opción **Configure SAML** (Configurar SAML).

   ![Configuración de SAML](media/coda-tutorial/saml-settings-link.png)

1. Anote los valores de **Entity ID** (Id. de entidad) y **SAML Response URL** (Dirección URL de respuesta SAML), que necesitará en los pasos siguientes.

   ![Valores de Entity ID y SAML Response URL (Id. de entidad) y Dirección URL de respuesta SAML para usar en Azure](media/coda-tutorial/azure-settings.png)

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Coda**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar el inicio de sesión único con SAML**, escriba los valores de los siguientes campos:

   a. En el cuadro de texto **Identificador**, escriba el valor de "Entity ID" (Id. de entidad) anterior. Debe seguir el patrón: `https://coda.io/samlId/<CUSTOMID>`.

   b. En el cuadro de texto **URL de respuesta**, escriba el valor de "SAML Response URL" (Dirección URL de respuesta SAML) anterior. Debe seguir el patrón: `https://coda.io/login/sso/saml/<CUSTOMID>/consume`.

   > [!NOTE]
   > Sus valores diferirán de los anteriores; puede encontrar los valores en la consola "Configure SAML" (Configurar SAML) de Coda. Actualice estos valores con el identificador y la URL de respuesta reales.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Coda**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Coda mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Coda**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

   ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-coda-sso"></a>Configuración del inicio de sesión único de Coda

Para completar la configuración, escriba valores de Azure Active Directory en el panel **Configure SAML** (Configurar SAML) de Coda.

1. En Coda, abra el panel de **Organization settings** (Configuración de la organización).
1. En **Authenticate with SSO (SAML)** (Autenticación con inicio de sesión único [SAML]), seleccione la opción **Configure SAML** (Configurar SAML).
1. Establezca **SAML Provider** (Proveedor de SAML) en **Azure Active Directory**.
1. En **Identity Provider Login URL** (URL de inicio de sesión del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión** de la consola de Azure.
1. En **Identity Provider Issuer** (Emisor del proveedor de identidades), pegue el valor de **Identificador de Azure AD** de la consola de Azure.
1. En **Identity Provider Public Certificate** (Certificado público del proveedor de identidades), seleccione la opción **Upload Certificate** (Cargar certificado) y seleccione el archivo de certificado que descargó anteriormente.
1. Seleccione **Guardar**.

Esto completa el trabajo necesario para la configuración de la conexión de inicio de sesión único de SAML.

### <a name="create-coda-test-user"></a>Creación de un usuario de prueba de Coda

En esta sección, se crea el usuario B.Simon en Coda. Coda admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario ya no existe en Coda, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Coda en el panel de acceso, iniciará sesión automáticamente en la instancia de Coda para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Coda con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Cómo proteger Coda con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
