---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Single Sign-on for Skytap | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Single Sign-on for Skytap.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/13/2020
ms.author: jeedes
ms.openlocfilehash: 8d34ca8ed01144ee282f6411640894807a09ef08
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88527883"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-single-sign-on-for-skytap"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Single Sign-on for Skytap

En este tutorial, aprenderá a integrar Single Sign-on for Skytap con Azure Active Directory (Azure AD). Al integrar Single Sign-on for Skytap con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Single Sign-on for Skytap.
* Permitir que los usuarios inicien sesión automáticamente en Single Sign-on for Skytap con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central, Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Single Sign-on for Skytap.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Single Sign-on for Skytap admite el inicio de sesión único iniciado por SP e IDP.
* Después de configurar Single Sign-on for Skytap, puede aplicar el control de sesión. Esto protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-single-sign-on-for-skytap-from-the-gallery"></a>Adición de Single Sign-on for Skytap desde la galería

Para configurar la integración de Single Sign-on for Skytap en Azure AD, es preciso que agregue Single Sign-on for Skytap desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Single Sign-on for Skytap** en el cuadro de búsqueda.
1. Seleccione **Single Sign-on for Skytap** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-single-sign-on-for-skytap"></a>Configuración y prueba del inicio de sesión único de Azure AD para Single Sign-on for Skytap

Configure y pruebe el inicio de sesión único de Azure AD con Single Sign-on for Skytap mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Single Sign-on for Skytap.

Estos son los pasos generales para configurar y probar el inicio de sesión único de Azure AD con Single Sign-on for Skytap:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.

    a. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.

    b. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Single Sign-on for Skytap](#configure-single-sign-on-for-skytap-sso)** para configurar los valores de inicio de sesión único en la aplicación.

    a. **[Creación de un usuario de prueba en Single Sign-on for Skytap](#create-single-sign-on-for-skytap-test-user)** para tener un homólogo de B.Simon en Single Sign-on for Skytap. Este homólogo está vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Single Sign-on for Skytap**, busque la sección **Administrar**. Seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Captura de pantalla de la página Configurar el inicio de sesión único con SAML, con el icono de lápiz resaltado](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `http://pingone.com/<custom EntityID>`.

    b. En el cuadro de texto **Dirección URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://sso.connect.pingidentity.com/sso/sp/ACS.saml2`.

1. Seleccione **Establecer direcciones URL adicionales** y siga estos pasos si desea configurar la aplicación en el modo iniciado por **SP**:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://sso.connect.pingidentity.com/sso/sp/initsso?saasid=<saasid>&idpid=<idpid>`.

    
    b. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://pingone.com/1.0/<custom ID>`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de respuesta, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico al cliente de Single Sign-on for Skytap](mailto:support@skytap.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación**. Seleccione **Descargar** para descargar el archivo de metadatos y guárdelo en el equipo.

    ![Captura de pantalla del vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Single Sign-on for Skytap**, copie las direcciones URL adecuadas según sus necesidades.

    ![Captura de pantalla de la copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el campo **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a Single Sign-on for Skytap mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Single Sign-on for Skytap**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Captura de pantalla de la sección Administrar, con la opción Usuarios y grupos resaltada](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Captura de pantalla de la página Usuarios y grupos, con Agregar usuario resaltado](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-single-sign-on-for-skytap-sso"></a>Configuración del inicio de sesión único en Single Sign-on for Skytap

Para configurar el inicio de sesión único en Single Sign-on for Skytap, es preciso enviar el contenido de **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Single Sign-on for Skytap](mailto:support@skytap.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.


### <a name="create-single-sign-on-for-skytap-test-user"></a>Creación de un usuario de prueba en Single Sign-on for Skytap

En esta sección, va a crear un usuario llamado B.Simon en Single Sign-on for Skytap. Colabore con el [equipo de soporte técnico de Single Sign-on for Skytap](mailto:support@skytap.com) para agregar los usuarios en la plataforma de Single Sign-on for Skytap. No puede usar el inicio de sesión único hasta que cree y active los usuarios.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Single Sign-on for Skytap en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Single Sign-on for Skytap para la que configuró el inicio de sesión único. Para más información, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Slack con Azure AD](https://aad.portal.azure.com/)

