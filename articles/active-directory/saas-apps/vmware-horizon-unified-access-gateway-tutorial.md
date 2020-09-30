---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con VMware Horizon - Unified Access Gateway | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y VMware Horizon - Unified Access Gateway.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/02/2020
ms.author: jeedes
ms.openlocfilehash: 9e9d98e8b68b4e01d7e8f19e7c7d89a88d9bf69f
ms.sourcegitcommit: 206629373b7c2246e909297d69f4fe3728446af5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/06/2020
ms.locfileid: "89500598"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-vmware-horizon---unified-access-gateway"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con VMware Horizon - Unified Access Gateway

En este tutorial, aprenderá a integrar VMware Horizon - Unified Access Gateway con Azure Active Directory (Azure AD). Al integrar VMware Horizon - Unified Access Gateway con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a VMware Horizon - Unified Access Gateway.
* Permitir que los usuarios inicien sesión automáticamente en VMware Horizon - Unified Access Gateway con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de VMware Horizon - Unified Access Gateway habilitada para el inicio de sesión único (SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* VMware Horizon - Unified Access Gateway admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-vmware-horizon---unified-access-gateway-from-the-gallery"></a>Adición de VMware Horizon - Unified Access Gateway desde la galería

Para configurar la integración de VMware Horizon - Unified Access Gateway en Azure AD, deberá agregar VMware Horizon - Unified Access Gateway desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **VMware Horizon - Unified Access Gateway** en el cuadro de búsqueda.
1. Seleccione **VMware Horizon - Unified Access Gateway** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-vmware-horizon---unified-access-gateway"></a>Configuración y prueba del inicio de sesión único de Azure AD para VMware Horizon - Unified Access Gateway

Configure y pruebe el inicio de sesión único de Azure AD con VMware Horizon - Unified Access Gateway mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de VMware Horizon - Unified Access Gateway.

Para configurar y probar el inicio de sesión único de Azure AD con VMware Horizon - Unified Access Gateway, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en VMware Horizon-Unified Access Gateway](#configure-vmware-horizon-unified-access-gateway-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de VMware Horizon-Unified Access Gateway](#create-vmware-horizon-unified-access-gateway-test-user)** : para tener un homólogo de B.Simon en VMware Horizon - Unified Access Gateway que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **VMware Horizon - Unified Access Gateway**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<HORIZON_UAG_FQDN>/portal`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<HORIZON_UAG_FQDN>/portal/samlsso`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<HORIZON_UAG_FQDN>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de VMware Horizon - Unified Access Gateway](mailto:support@vmware.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar VMware Horizon - Unified Access Gateway**, copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección, va a conceder a B.Simon acceso a VMware Horizon - Unified Access Gateway mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **VMware Horizon - Unified Access Gateway**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-vmware-horizon-unified-access-gateway-sso"></a>Configuración del inicio de sesión único de VMware Horizon-Unified Access Gateway

Para configurar el inicio de sesión único en **VMware Horizon - Unified Access Gateway**, es preciso enviar el archivo **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de VMware Horizon - Unified Access Gateway](mailto:support@vmware.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-vmware-horizon-unified-access-gateway-test-user"></a>Creación de un usuario de prueba de VMware Horizon-Unified Access Gateway

En esta sección creará un usuario llamado B.Simon en VMware Horizon - Unified Access Gateway. Colabore con el [equipo de soporte técnico de VMware Horizon - Unified Access Gateway](mailto:support@vmware.com) para agregar los usuarios a la plataforma de VMware Horizon - Unified Access Gateway. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de VMware Horizon - Unified Access Gateway.  

* Vaya directamente a la dirección URL de inicio de sesión de VMware Horizon - Unified Access Gateway e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal y debería iniciar sesión automáticamente en la instancia de VMware Horizon - Unified Access Gateway para la que configuró el inicio de sesión único. 

También puede usar el Panel de acceso de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de VMware Horizon - Unified Access Gateway en el panel de acceso, debería iniciar sesión automáticamente en la instancia de VMware Horizon - Unified Access Gateway para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez que configure VMware Horizon - Unified Access Gateway, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
