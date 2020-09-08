---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Slack | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Slack.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/28/2020
ms.author: jeedes
ms.openlocfilehash: fdea1f3b2d4cff0203951b6ec5ef6b86b62cdf9c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88527580"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-slack"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Slack

En este tutorial, obtendrá información sobre cómo integrar Slack con Azure Active Directory (Azure AD). Al integrar Slack con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Slack.
* Permitir que los usuarios inicien sesión automáticamente en Slack con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Slack.

> [!NOTE]
> Si necesita efectuar una integración con más de una instancia de Slack en un inquilino, el identificador de cada aplicación puede ser una variable.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Slack admite el inicio de sesión único iniciado por **SP**.
* Slack admite el aprovisionamiento de usuarios **Just-In-Time**
* Slack admite el [aprovisionamiento **automático** de usuarios](https://docs.microsoft.com/azure/active-directory/saas-apps/slack-provisioning-tutorial)
* Una vez configurado Slack, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-slack-from-the-gallery"></a>Adición de Slack desde la galería

Para configurar la integración de Slack en Azure AD, tendrá que agregar Slack desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Slack** en el cuadro de búsqueda.
1. Seleccione **Slack** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-slack"></a>Configuración y prueba del inicio de sesión único de Azure AD para Slack

Configure y pruebe el inicio de sesión único de Azure AD con Slack mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Slack.

Para configurar y probar el inicio de sesión único de Azure AD con Slack, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Slack](#configure-slack-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de Slack](#create-slack-test-user)** , para tener un homólogo de B. Simon en Slack que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Slack**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://< DOMAIN NAME>.slack.com/sso/saml/start`

    b. En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL: `https://slack.com`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualice el valor con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Slack](https://slack.com/help/contact) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.
    
    > [!NOTE]
    > El valor de **Identificador (Id. de entidad)** puede ser una variable si tiene más de una instancia de Slack para integrar con el inquilino. Utilice el patrón `https://<DOMAIN NAME>.slack.com`. En este escenario, debe emparejar otro valor en Slack utilizando el mismo valor.

1. La aplicación Slack espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Slack espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos. También debe agregar el atributo `email`. Si el usuario no tiene una dirección de correo electrónico, asigne **emailaddress** a **user.userprincipalname** y **email** a **user.userprincipalname**.

    | Nombre | Atributo de origen |
    | -----|---------|
    | emailaddress | user.userprincipalname |
    | email | user.userprincipalname |
    | | |

   > [!NOTE]
   > Para establecer la configuración del proveedor de servicios (SP), debe hacer clic en **Expand** (Expandir) junto a **Advanced Options** (Opciones avanzadas) en la página de configuración de SAML. En el cuadro de texto **Service Provider Issuer** (Emisor del proveedor de servicios), escriba la dirección URL del área de trabajo. El valor predeterminado es slack.com. 

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Slack**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a Slack mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Slack**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-slack-sso"></a>Configuración del inicio de sesión único en Slack

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de Slack como administrador.

2. Vaya a **Microsoft Azure AD** y luego a **Configuración del equipo**.

     ![Configuración del inicio de sesión único en App Side](./media/slack-tutorial/tutorial-slack-team-settings.png)

3. En la sección **Configuración del equipo**, haga clic en la pestaña **Autenticación** y luego en **Cambiar configuración**.

    ![Configuración del inicio de sesión único en App Side](./media/slack-tutorial/tutorial-slack-authentication.png)

4. En el cuadro de diálogo **Configuración de la autenticación SAML** , realice los pasos siguientes:

    ![Configuración del inicio de sesión único en App Side](./media/slack-tutorial/tutorial-slack-save-authentication.png)

    a.  En el cuadro de texto **SAML 2.0 Endpoint (HTTP)** [Punto de conexión SAML 2.0 (HTTP)], pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b.  En el cuadro de texto **Emisor de proveedor de identidades**, pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.

    c.  Abra el archivo de certificado descargado en el Bloc de notas, copie el contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado público**.

    d. Configure las tres opciones anteriores según corresponda para su equipo de Slack. Para obtener más información sobre la configuración, busque la **Guía de configuración de SSO de Slack** aquí. `https://get.slack.help/hc/articles/220403548-Guide-to-single-sign-on-with-Slack%60`

    ![Configuración del inicio de sesión único en App Side](./media/slack-tutorial/tutorial-slack-expand.png)

    e. Haga clic en **Expand** (Expandir) y escriba `https://slack.com` en el cuadro de texto **Service provider issuer** (Emisor del proveedor de servicios).

    f.  Haga clic en **Guardar configuración**.
    
    > [!NOTE]
    > Si tiene más de una instancia de Slack para integrar con Azure AD, establezca `https://<DOMAIN NAME>.slack.com` en **Service provider issuer** (Emisor del proveedor de servicios) para emparejarlo con el **identificador** de la aplicación de Azure.

### <a name="create-slack-test-user"></a>Creación de un usuario de prueba de Slack

El objetivo de esta sección es crear una usuaria de prueba llamada B.Simon en Slack. Slack admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Al intentar acceder a Slack, se crea un nuevo usuario, en caso de que no exista. Slack también admite el aprovisionamiento automático de usuarios. [Aquí](slack-provisioning-tutorial.md) puede encontrar más detalles sobre cómo configurar el aprovisionamiento automático de usuarios.

> [!NOTE]
> Si necesita crear manualmente un usuario, es preciso que se ponga en contacto con el [equipo de soporte técnico de Slack](https://slack.com/help/contact).

> [!NOTE]
> Azure AD Connect es la herramienta de sincronización que puede sincronizar usuarios locales de Active Directory Identity con Azure AD y estos usuarios sincronizados también podrán usar las aplicaciones como otros usuarios en la nube.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Slack en el panel de acceso y debería iniciar sesión automáticamente en la versión de Slack para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Slack con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
