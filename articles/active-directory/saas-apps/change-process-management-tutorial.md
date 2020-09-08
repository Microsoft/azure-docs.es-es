---
title: 'Tutorial: Integración del inicio de sesión único de Azure AD con Change Process Management'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Change Process Management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2020
ms.author: jeedes
ms.openlocfilehash: 501f19b2022d221ad6facd66c483790b13491ba0
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88529685"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-change-process-management"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Change Process Management

En este tutorial, aprenderá a integrar Change Process Management con Azure Active Directory (Azure AD). Al integrar Change Process Management con Azure AD, puede:

* Use Azure AD para controlar quién puede acceder a Change Process Management.
* Permitir que los usuarios inicien sesión automáticamente en Change Process Management con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de Change Process Management habilitada para el inicio de sesión único (SSO).

## <a name="tutorial-description"></a>Descripción del tutorial

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

Change Process Management admite el SSO iniciado por IDP.

Una vez configurado Change Process Management, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-change-process-management-from-the-gallery"></a>Incorporación de Change Process Management desde la galería

Para configurar la integración de Change Process Management en Azure AD, es preciso agregar Change Process Management desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta profesional o educativa o con una cuenta Microsoft personal.
1. En el panel izquierdo, seleccione **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Change Process Management** en el cuadro de búsqueda.
1. Seleccione **Change Process Management** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-change-process-management"></a>Configuración y prueba del inicio de sesión único de Azure AD para Change Process Management

Va a configurar y probar el inicio de sesión único de Azure AD con Change Process Management mediante un usuario de prueba llamado B.Simon. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Change Process Management.

Para configurar y probar el inicio de sesión único de Azure AD con Change Process Management, es preciso llevar a cabo estos pasos generales:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
    1. **[Concesión de acceso al usuario de prueba](#grant-access-to-the-test-user)** , para que el usuario pueda usar el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Change Process Management](#configure-change-process-management-sso)** en el lado de la aplicación.
    1. **[Creación de un usuario de prueba de Change Process Management](#create-a-change-process-management-test-user)** como homólogo de la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** para comprobar que la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Change Process Management**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el botón de lápiz para **Configuración básica de SAML** para editar la configuración:

   ![Botón de lápiz para Configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar el inicio de sesión único con SAML** realice estos pasos:

    a. En el cuadro **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<hostname>:8443/`

    b. En el cuadro **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<hostname>:8443/changepilot/saml/sso`

    > [!NOTE]
    > Los valores de **Identificador** y **Dirección URL de respuesta** anteriores no son los valores reales que debe usar. Póngase en contacto con el [equipo de soporte técnico de Change Process Management](mailto:support@realtech-us.com) para obtener los valores reales. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** para **Certificado (Base64)** para descargarlo y guardarlo en el equipo:

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Change Process Management**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**. Seleccione **Usuarios** y, a continuación, seleccione **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades de **usuario**, realice estos pasos:
   1. En el cuadro **Nombre**, escriba **B.Simon**.  
   1. En el cuadro **Nombre de usuario**, escriba \<username>@\<companydomain>.\<extension>. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione **Mostrar contraseña** y, a continuación, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="grant-access-to-the-test-user"></a>Concesión de acceso al usuario de prueba

En esta sección, va a conceder a B.Simon acceso a Change Process Management mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** y, a continuación, seleccione **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Change Process Management**.
1. En la sección **Administrar** de la página de información general de la aplicación, seleccione **Usuarios y grupos**:

   ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista **Usuarios** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-change-process-management-sso"></a>Configuración del inicio de sesión único de Change Process Management

Para configurar el inicio de sesión único en Change Process Management, tiene que enviar el certificado Base64 descargado y las direcciones URL apropiadas que copió de Azure Portal al [equipo de soporte técnico de Change Process Management](mailto:support@realtech-us.com). Ellos configuran la conexión de inicio de sesión único de SAML de manera correcta en ambas partes.

### <a name="create-a-change-process-management-test-user"></a>Creación de un usuario de prueba de Change Process Management
 Colabore con el [equipo de soporte técnico de Change Process Management](mailto:support@realtech-us.com) para agregar un usuario llamado B.Simon en Change Process Management. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Change Process Management en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Change Process Management para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para la integración de aplicaciones SaaS con Azure Active Directory ](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Change Process Management con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Cómo proteger Change Process Management con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)