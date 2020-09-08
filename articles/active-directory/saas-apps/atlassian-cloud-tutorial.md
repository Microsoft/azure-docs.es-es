---
title: 'Tutorial: Integración de Azure Active Directory con Atlassian Cloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Atlassian Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/04/2020
ms.author: jeedes
ms.openlocfilehash: bdd0530aa580f8f1a1cfdffb81d070827a714115
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686078"
---
# <a name="tutorial-integrate-atlassian-cloud-with-azure-active-directory"></a>Tutorial: Integración de Atlassian Cloud con Azure Active Directory

En este tutorial, aprenderá a integrar Atlassian Cloud con Azure Active Directory (Azure AD). Al integrar Atlassian Cloud con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Atlassian Cloud.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Atlassian Cloud con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede obtener la evaluación gratuita de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Atlassian Cloud.
* Para habilitar el inicio de sesión único en Lenguaje de marcado de aserción de seguridad (SAML) para productos de Atlassian Cloud, debe configurar Atlassian Access. Más información sobre [Atlassian Access]( https://www.atlassian.com/enterprise/cloud/identity-manager).

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 

* Atlassian Cloud admite SSO iniciado por **SP e IDP**
* Atlassian Cloud admite el [aprovisionamiento y desaprovisionamiento automático de usuarios](atlassian-cloud-provisioning-tutorial.md).
* Una vez configurado Atlassian Cloud, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-atlassian-cloud-from-the-gallery"></a>Incorporación de Atlassian Cloud desde la galería

Para configurar la integración de Atlassian Cloud en Azure AD, necesita agregar Atlassian Cloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Atlassian Cloud** en el cuadro de búsqueda.
1. Seleccione **Atlassian Cloud** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Atlassian Cloud utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Atlassian Cloud.

Para configurar y probar el inicio de sesión único de Azure AD con Atlassian Cloud, es preciso completar los siguientes bloques de creación:

1. **[Configuración de Azure AD con el inicio de sesión único de Atlassian Cloud](#configure-azure-ad-sso)** para permitir a los usuarios utilizar el inicio de sesión único de SAML basado en Azure AD con Atlassian Cloud.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Creación de un usuario de prueba en Atlassian Cloud](#create-atlassian-cloud-test-user)** , para tener un homólogo de B.Simon en Atlassian Cloud que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. Para automatizar la configuración en Atlassian Cloud, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Install the extension** (Instalar la extensión).

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Atlassian Cloud** para ir a la aplicación Atlassian Cloud. Desde allí, proporcione las credenciales de administrador para iniciar sesión en Atlassian Cloud. La extensión de explorador configurará automáticamente la aplicación.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar Atlassian Cloud manualmente, inicie sesión en el sitio de empresa de Atlassian Cloud como administrador y realice los pasos siguientes.

1. Antes de empezar, vaya a la instancia de Atlassian y copie y guarde la dirección URL de la instancia
   > [!NOTE]
   > la dirección URL debe ajustarse al patrón `https://<instancename>.atlassian.net`

   ![imagen](./media/atlassian-cloud-tutorial/get-atlassian-instance-name.png)

1. Abra el [portal de administración de Atlassian](https://admin.atlassian.com/) y haga clic en el nombre de su organización

   ![imagen](./media/atlassian-cloud-tutorial/click-on-organization-in-atlassian-access.png)

1. Debe comprobar el dominio antes de configurar el inicio de sesión único. Para más información, consulte el documento [Atlassian domain verification](https://confluence.atlassian.com/cloud/domain-verification-873871234.html) (Comprobación del dominio Atlassian).
1. En el portal de administración de Atlassian, seleccione **Security** (Seguridad) en el cajón de la izquierda

   ![imagen](./media/atlassian-cloud-tutorial/click-on-security-in-atlassian-access.png)

1. En la pantalla de seguridad del portal de administración de Atlassian, seleccione **SAML single sign on** (Inicio de sesión único de SAML) en el cajón de la izquierda

   ![imagen](./media/atlassian-cloud-tutorial/click-on-saml-sso-in-atlassian-access-security.png)

1. Haga clic en **Add SAML Configuration** (Agregar configuración de SAML) y mantenga abierta la página

   ![imagen](./media/atlassian-cloud-tutorial/saml-configuration-in-atlassian-access-security-saml-sso.png)

   ![imagen](./media/atlassian-cloud-tutorial/add-saml-configuration.png)

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Atlassian Cloud**, busque la sección **Administrar** y seleccione **Configurar inicio de sesión único**.

   ![imagen](./media/atlassian-cloud-tutorial/set-up-sso.png)

1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.

   ![imagen](./media/atlassian-cloud-tutorial/saml-in-azure.png)

1. En la página **Configuración del inicio de sesión único con SAML**, desplácese hacia abajo hasta **Configurar Atlassian Cloud**
   
   a. Haga clic en **Direcciones URL de configuración**

   ![imagen](./media/atlassian-cloud-tutorial/configuration-urls.png)
   
   b. Copie el valor de **Identificador de Azure AD** en Azure Portal y péguelo en el cuadro de texto **Identity Provider Entity ID** (Id. de entidad del proveedor de identidades) en Atlassian
   
   c. Copie el valor de **Dirección URL de inicio de sesión** en Azure Portal y péguelo en el cuadro de texto **Identity Provider SSO URL** (Dirección URL de inicio de sesión único del proveedor de identidades) en Atlassian

   ![imagen](./media/atlassian-cloud-tutorial/configuration-urls-azure.png)

   ![imagen](./media/atlassian-cloud-tutorial/entity-id-and-ss.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![imagen](./media/atlassian-cloud-tutorial/certificate.png)

   ![imagen](./media/atlassian-cloud-tutorial/certificate-1.png)

1. **Agregar o guardar** la configuración de SAML en Atlassian

1. Si desea configurar la aplicación en el modo iniciado por **IDP**, edite la sección de **configuración básica de SAML** de la página **Configuración del inicio de sesión único con SAML** de Azure y abra la página **Inicio de sesión único de SAML** en el portal de administración de Atlassian

   a. Copie el valor de **SP Entity ID** (Identificador de entidad del proveedor de servicios) de Atlassian, péguelo en el cuadro **Identificador (id. de entidad)** en Azure y establézcalo como predeterminado
   
   b. Copie el valor de **SP Assertion Consumer Service URL** (URL del Servicio de consumidor de aserciones) de Atlassian, péguelo en el cuadro **Dirección URL de respuesta (URL del Servicio de consumidor de aserciones)** de Azure y establézcalo como valor predeterminado
   
   c. Copie el valor de **Dirección URL de la instancia**, que copió en el paso 1 y péguelo en el cuadro **Estado de la retransmisión** de Azure

   ![imagen](./media/atlassian-cloud-tutorial/copy-urls.png)

   ![imagen](./media/atlassian-cloud-tutorial/edit-button.png)

   ![imagen](./media/atlassian-cloud-tutorial/urls.png)
   
1. Si desea configurar la aplicación en el modo iniciado por **IDP**, edite la sección de **configuración básica de SAML** de la página **Configuración del inicio de sesión único con SAML** de Azure. Copie la **Dirección URL de la instancia** (del paso 1) y péguela en el cuadro **URL de inicio de sesión** de Azure

   ![imagen](./media/atlassian-cloud-tutorial/edit-button.png)

   ![imagen](./media/atlassian-cloud-tutorial/sign-on-URL.png)
   
1. La aplicación Atlassian Cloud espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. Para editar la asignación de atributos, haga clic en el icono de **edición**. 

   ![imagen](./media/atlassian-cloud-tutorial/default-attributes.png)
   
   1. Asignación de atributos para un inquilino de Azure AD con una licencia de Office 365
      
      a. Haga clic en la notificación **Identificador de usuario único (id. de nombre)**

      ![imagen](./media/atlassian-cloud-tutorial/user-attributes-and-claims.png)
      
      b. Atlassian Cloud espera que este **id. de nombre** (**Identificador de usuario único**) se asigne al correo electrónico del usuario (**user.email**). Edite el **atributo de origen** y cámbielo a **user.mail**. Guarde los cambios de la notificación.

      ![imagen](./media/atlassian-cloud-tutorial/unique-user-identifier.png)
      
      c. Las asignaciones de atributo finales deben tener el aspecto siguiente.

      ![imagen](common/default-attributes.png)
      
   1. Asignación de atributos para un inquilino de Azure AD sin una licencia de Office 365 

      a. Haga clic en la notificación `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` .

      ![imagen](./media/atlassian-cloud-tutorial/email-address.png)
         
      b. Aunque Azure no rellena el atributo **user.mail** de los usuarios creados en inquilinos de Azure AD sin licencias de Office 365, almacena el correo electrónico de esos usuarios en el atributo **userprincipalname**. Atlassian Cloud espera que este **id. de nombre** (**Identificador de usuario único**) se asigne al correo electrónico del usuario (**user.userprincipalname**).  Edite el **atributo de origen** y cámbielo a **user.userprincipalname**. Guarde los cambios de la notificación.

      ![imagen](./media/atlassian-cloud-tutorial/set-email.png)
         
      c. Las asignaciones de atributo finales deben tener el aspecto siguiente.

      ![imagen](common/default-attributes.png)
     
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

En esta sección, va a conceder acceso a B.Simon a Atlassian Cloud para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Atlassian Cloud**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](./media/atlassian-cloud-tutorial/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-atlassian-cloud-test-user"></a>Creación de un usuario de prueba de Atlassian Cloud

Para permitir que los usuarios de Azure AD inicien sesión en Atlassian Cloud, aprovisione las cuentas de usuario manualmente en Atlassian Cloud haciendo lo siguiente:

1. En el panel **Administración**, seleccione **Usuarios**.

    ![Vínculo de usuarios de Atlassian Cloud](./media/atlassian-cloud-tutorial/tutorial-atlassiancloud-14.png)

1. Para crear un usuario en Atlassian Cloud, seleccione **Invitar usuario**.

    ![Creación de un usuario de Atlassian Cloud](./media/atlassian-cloud-tutorial/tutorial-atlassiancloud-15.png)

1. En el cuadro **Dirección de correo electrónico**, escriba la dirección de correo electrónico del usuario y asigne el acceso a la aplicación.

    ![Creación de un usuario de Atlassian Cloud](./media/atlassian-cloud-tutorial/tutorial-atlassiancloud-16.png)

1. Para enviar una invitación por correo electrónico al usuario, seleccione **Invitar a usuarios**. Se envía una invitación por correo electrónico al usuario y, después de aceptar la invitación, el usuario se activa en el sistema.

> [!NOTE]
> También puede crear usuarios de forma masiva mediante la selección del botón **Bulk Create** (Crear de forma masiva) en la sección **Usuarios**.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Atlassian Cloud en el panel de acceso y debería iniciar sesión automáticamente en la versión de Atlassian Cloud para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Atlassian Cloud con Azure AD](https://aad.portal.azure.com/)

- [Protección de Atlassian Cloud con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)