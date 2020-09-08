---
title: 'Tutorial: Integración del inicio de sesión único de Azure AD con Smart Global Governance'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Smart Global Governance.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/04/2020
ms.author: jeedes
ms.openlocfilehash: d93cd2efe49f819c8adeb53674b6d4f465312e06
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88527427"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-smart-global-governance"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Smart Global Governance

En este tutorial, aprenderá a integrar Smart Global Governance con Azure Active Directory (Azure AD). Al integrar Smart Global Governance con Azure AD, puede hacer lo siguiente:

* Usar Azure AD para controlar quién puede acceder a Smart Global Governance.
* Permitir que los usuarios inicien sesión automáticamente en Smart Global Governance con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Smart Global Governance.

## <a name="tutorial-description"></a>Descripción del tutorial

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

Smart Global Governance admite el inicio de sesión único iniciado por SP e IDP.

Después de configurar Smart Global Governance, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-smart-global-governance-from-the-gallery"></a>Incorporación de Smart Global Governance desde la galería

Para configurar la integración de Smart Global Governance en Azure AD, deberá agregar Smart Global Governance desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta profesional o educativa o con una cuenta Microsoft personal.
1. En el panel izquierdo, seleccione **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Smart Global Governance** en el cuadro de búsqueda.
1. Seleccione **Smart Global Governance** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-smart-global-governance"></a>Configuración y prueba del inicio de sesión único de Azure AD para Smart Global Governance

Va a configurar y probar el inicio de sesión único de Azure AD con Smart Global Governance mediante un usuario de prueba llamado B.Simon. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Smart Global Governance.

Para configurar y probar el inicio de sesión único de Azure AD con Smart Global Governance, es preciso llevar a cabo estos pasos generales:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
    1. **[Concesión de acceso al usuario de prueba](#grant-access-to-the-test-user)** , para que el usuario pueda usar el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Smart Global Governance](#configure-smart-global-governance-sso)** en el lado de la aplicación.
    1. **[Creación de un usuario de prueba de Smart Global Governance](#create-a-smart-global-governance-test-user)** como homólogo de la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** para comprobar que la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Smart Global Governance**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el botón de lápiz para **Configuración básica de SAML** para editar la configuración:

   ![Botón de lápiz para Configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por IDP, siga estos pasos.

    a. En el cuadro **Identificador**, escriba una de estas direcciones URL:

    - `https://eu-fr-south.console.smartglobalprivacy.com/platform/authentication-saml2/metadata`
    - `https://eu-fr-south.console.smartglobalprivacy.com/dpo/authentication-saml2/metadata`

    b. En el cuadro **URL de respuesta**, escriba una de estas direcciones URL:

    - `https://eu-fr-south.console.smartglobalprivacy.com/platform/authentication-saml2/acs`
    - `https://eu-fr-south.console.smartglobalprivacy.com/dpo/authentication-saml2/acs`

1. Si quiere configurar la aplicación en modo iniciado por SP, seleccione **Establecer direcciones URL adicionales** y haga lo siguiente.

   - En el cuadro **Dirección URL de inicio de sesión**, escriba una de estas direcciones URL:

    - `https://eu-fr-south.console.smartglobalprivacy.com/dpo`
    - `https://eu-fr-south.console.smartglobalprivacy.com/platform`

1. En la página **Configurar inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** para **Certificado (sin procesar)** para descargarlo y guardarlo en el equipo:

    ![Vínculo de descarga del certificado](common/certificateraw.png)

1. En la sección **Configurar Smart Global Governance**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, se crea un usuario llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**. Seleccione **Usuarios** y, a continuación, seleccione **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades de **usuario**, realice estos pasos:
   1. En el cuadro **Nombre**, escriba **B.Simon**.  
   1. En el cuadro **Nombre de usuario**, escriba \<username>@\<companydomain>.\<extension>. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione **Mostrar contraseña** y, a continuación, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="grant-access-to-the-test-user"></a>Concesión de acceso al usuario de prueba

En esta sección va a permitir que B.Simon acceda a Smart Global Governance mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** y, a continuación, seleccione **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Smart Global Governance**.
1. En la sección **Administrar** de la página de información general de la aplicación, seleccione **Usuarios y grupos**:

   ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, a continuación, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**:

    ![Selección de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista **Usuarios** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-smart-global-governance-sso"></a>Configuración del inicio de sesión único de Smart Global Governance

Para configurar el inicio de sesión único en Smart Global Governance, es preciso enviar el certificado (sin procesar) descargado y las direcciones URL adecuadas que ha copiado de Azure Portal al [equipo de soporte técnico de Smart Global Governance](mailto:support.tech@smartglobal.com). Ellos configuran la conexión de inicio de sesión único de SAML de manera correcta en ambas partes.

### <a name="create-a-smart-global-governance-test-user"></a>Creación de un usuario de prueba de Smart Global Governance

Colabore con el [equipo de soporte técnico de Smart Global Governance](mailto:support.tech@smartglobal.com) para agregar un usuario llamado B.Simon a Smart Global Governance. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Smart Global Governance en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Smart Global Governance para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para la integración de aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Smart Global Governance con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Smart Global Governance con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)