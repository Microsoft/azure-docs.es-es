---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Shopify Plus | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Shopify Plus.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/18/2020
ms.author: jeedes
ms.openlocfilehash: 61d2eb05f7953cda16664f6b4e3317feb46aa8ad
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548602"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-shopify-plus"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Shopify Plus

En este tutorial, aprenderá a integrar Shopify Plus con Azure Active Directory (Azure AD). Al integrar Shopify Plus con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Shopify Plus.
* Permitir que los usuarios inicien sesión automáticamente en Shopify Plus con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Shopify Plus.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Shopify Plus admite el inicio de sesión único iniciado por **SP e IDP**.

* Una vez configurado Shopify Plus, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-shopify-plus-from-the-gallery"></a>Adición de Shopify Plus desde la galería

Para configurar la integración de Shopify Plus con Azure AD, será preciso que agregue Shopify Plus desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Shopify Plus** en el cuadro de búsqueda.
1. Seleccione **Shopify Plus** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-shopify-plus"></a>Configuración y prueba del inicio de sesión único de Azure AD para Shopify Plus

Configure y pruebe el inicio de sesión único de Azure AD con Shopify Plus con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Shopify Plus.

Para configurar y probar el inicio de sesión único de Azure AD con Shopify Plus, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Shopify Plus](#configure-shopify-plus-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Shopify Plus](#create-shopify-plus-test-user)** : para tener un homólogo de B.Simon en Shopify Plus vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Shopify Plus**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://accounts.shopify.com/saml/consume/organization/<ORGANIZATION_ID>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://shopify.plus/login`

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Actualícelo con la dirección URL de respuesta real. Póngase en contacto con el [equipo de soporte técnico al cliente de Shopify Plus](mailto:plus-user-management@shopify.com) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Shopify Plus espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Shopify Plus espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ---- | --------------- |
    | email | user.mail |

1. Cambie el formato de **Identificador de nombre** a **Persistente**. Seleccione la opción **Identificador de usuario único (id. de nombre)** y después el formato **Identificador de nombre**. Seleccione **Persistente** en esta opción. Guarde los cambios.
1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección, va a permitir que B.Simon acceda a Shopify Plus mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Shopify Plus**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-shopify-plus-sso"></a>Configuración del inicio de sesión único de Shopify Plus

Para ver los pasos completos, consulte la [documentación de Shopify sobre cómo configurar integraciones de SAML](https://help.shopify.com/en/manual/shopify-plus/saml).

Para configurar el inicio de sesión único en **Shopify Plus**, copie el valor de **Dirección URL de metadatos de federación de aplicación** de Azure Active Directory. A continuación, inicie sesión en la [administración de organizaciones](https://shopify.plus) y vaya a **Users** > **Security** (Usuarios > Seguridad). Seleccione **Set up configuration** (Establecer configuración) y, a continuación, pegue la dirección URL de metadatos de federación de la aplicación en la sección **Identity provider metadata URL** (Dirección URL de metadatos del proveedor de identidad). Seleccione **Add** (Agregar) para completar este paso.

### <a name="create-shopify-plus-test-user"></a>Creación de un usuario de prueba de Shopify Plus

En esta sección, creará un usuario llamado B.Simon en Shopify Plus. Vuelva a la sección **Users** (Usuarios) y agregue un usuario indicando su correo electrónico y sus permisos. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

### <a name="enforce-saml-authentication"></a>Aplicación de la autenticación SAML

> [!NOTE]
> Se recomienda probar la integración con usuarios individuales antes de aplicarla globalmente.

Usuarios individuales:
1. Vaya a la página de un usuario individual en Shopify Plus con un dominio de correo electrónico administrado por Azure AD y comprobado en Shopify Plus.
1. En la sección de autenticación SAML, seleccione **Edit** (Editar), **Required** (Obligatorio) y, por último, **Save** (Guardar).
1. Compruebe que este usuario puede iniciar sesión correctamente con los flujos iniciados por IdP y por SP.

Para todos los usuarios de un dominio de correo electrónico:
1. Vuelva a la página **Security** (Seguridad).
1. Seleccione **Required** (Obligatorio) en el valor de autenticación SAML. Esto aplica SAML en todos los usuarios con ese dominio de correo electrónico de Shopify Plus.
1. Seleccione **Guardar**.

> [!IMPORTANT]
> La habilitación de SAML para todos los usuarios de un dominio de correo electrónico afecta a todos los usuarios que usan esa aplicación. Los usuarios no podrán iniciar sesión con su página de inicio de sesión habitual. Solo podrán acceder a la aplicación mediante Azure Active Directory. Shopify no ofrece una dirección URL de inicio de sesión de copia de seguridad en la que los usuarios puedan iniciar sesión con su nombre de usuario y contraseña normales. Puede ponerse en contacto con el soporte técnico de Shopify para desactivar SAML, si es necesario.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Shopify Plus en el Panel de acceso, iniciará sesión automáticamente en la instancia de Shopify Plus para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Shopify Plus con Azure AD](https://aad.portal.azure.com/).

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Shopify Plus con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
