---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con Cloud Academy - SSO'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory y Cloud Academy - SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: jeedes
ms.openlocfilehash: 37ed9bb09b6b15af0c32f489cbc3c02ec27c2827
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89461976"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-cloud-academy---sso"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con Cloud Academy - SSO

En este tutorial aprenderá a integrar Cloud Academy - SSO con Azure Active Directory (Azure AD). Al integrar Cloud Academy - SSO con Azure AD, puede hacer lo siguiente:

* Usar Azure AD para controlar quién puede acceder a Cloud Academy - SSO.
* Permitir que los usuarios inicien sesión automáticamente en Cloud Academy - SSO con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el inicio de sesión único (SSO)?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Cloud Academy - SSO.

## <a name="tutorial-description"></a>Descripción del tutorial

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

Cloud Academy - SSO admite el inicio de sesión único iniciado por SP.

Una vez configurado Cloud Academy - SSO, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-cloud-academy---sso-from-the-gallery"></a>Adición de Cloud Academy - SSO desde la galería

Para configurar la integración de Cloud Academy - SSO en Azure AD, es preciso agregar Cloud Academy - SSO desde la galería a la lista de aplicaciones SaaS administradas:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta profesional o educativa o con una cuenta Microsoft personal.
1. En el panel izquierdo, seleccione **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Cloud Academy - SSO** en el cuadro de búsqueda.
1. Seleccione **Cloud Academy - SSO** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-cloud-academy---sso"></a>Configuración y prueba del inicio de sesión único de Azure AD SSO para Cloud Academy - SSO

Configure y pruebe el inicio de sesión único de Azure AD con Cloud Academy - SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Cloud Academy - SSO.

Para configurar y probar el inicio de sesión único de Azure AD con Cloud Academy - SSO, complete los siguientes pasos generales:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
    1. **[Concesión de acceso al usuario de prueba](#grant-access-to-the-test-user)** , para que el usuario pueda usar el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único para Cloud Academy - SSO](#configure-single-sign-on-for-cloud-academy)** en la aplicación.
    1. **[Creación de un usuario de prueba de Cloud Academy - SSO](#create-a-cloud-academy-test-user)** como homólogo de la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** para comprobar que la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Cloud Academy - SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el botón de lápiz para **Configuración básica de SAML** para editar la configuración:

   ![Captura de pantalla que muestra el botón de lápiz para editar la configuración básica de SAML.](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, en el cuadro de texto **URL de inicio de sesión**, escriba `https://cloudacademy.com/login/enterprise/`.

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación**. Guarde la dirección URL.

    ![Captura de pantalla del botón de copia para el campo Dirección URL de metadatos de federación de aplicación.](common/copy-metadataurl.png)

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

En esta sección, va a permitir que B.Simon acceda a Cloud Academy - SSO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** y, a continuación, seleccione **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Cloud Academy - SSO**.
1. En la sección **Administrar** de la página de información general de la aplicación, seleccione **Usuarios y grupos**:

   ![Captura de pantalla que muestra la opción Usuarios y grupos.](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, a continuación, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**:

    ![Captura de pantalla que muestra el botón Agregar usuario.](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista **Usuarios** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-single-sign-on-for-cloud-academy"></a>Configuración del inicio de sesión único para Cloud Academy

1. En otra ventana del explorador, inicie sesión en el sitio de la compañía de Cloud Academy - SSO como administrador.

1. Seleccione el nombre de la compañía y, a continuación, seleccione **Settings & Integrations** (Configuración e integraciones) en el menú que aparece:

    ![Captura de pantalla que muestra la opción Configuración e integraciones.](./media/cloud-academy-sso-tutorial/config-1.PNG)

1. En la página **Settings & integraciones** (Configuración e integraciones), en la pestaña **Integrations** (Integraciones), seleccione la tarjeta **SSO**:

    ![Captura de pantalla que muestra la tarjeta SSO en la pestaña de integraciones.](./media/cloud-academy-sso-tutorial/config-2.PNG)

1. Realice los pasos siguientes en esta página:

    ![Captura de pantalla que muestra la página SSO en Integraciones.](./media/cloud-academy-sso-tutorial/config-3.PNG)

    a. En el cuadro **Entity ID URL** (Dirección URL del identificador de entidad), escriba el valor del identificador de entidad que copió de Azure Portal.

    b. En el cuadro **SSO URL** (Dirección URL de inicio de sesión único), pegue la dirección URL de inicio de sesión que copió de Azure Portal.

    c. Abra el certificado Base 64 descargado desde Azure Portal en el Bloc de notas. Pegue su contenido en el cuadro **Certificate** (Certificado).

    d. En el cuadro de texto **Name ID Format** (Formato del identificador de nombre), mantenga el valor predeterminado: `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`.

1. Seleccione **Guardar**.

    > [!NOTE]
    > Para más información sobre cómo configurar Cloud Academy - SSO, consulte [Configuración del inicio de sesión único](https://support.cloudacademy.com/hc/articles/360043908452-Setting-Up-Single-Sign-On).

### <a name="create-a-cloud-academy-test-user"></a>Creación de un usuario de prueba de Cloud Academy

1. Inicie sesión en Cloud Academy - SSO.

1. Seleccione el nombre de la compañía y, a continuación, seleccione **Members** (Miembros) en el menú que aparece:

    ![Captura de pantalla que muestra la opción Miembros.](./media/cloud-academy-sso-tutorial/create-user.PNG)

1. Seleccione **Invite Members** (Invitar a miembros) y, a continuación, seleccione **Invite a single Member** (Invitar a un solo miembro):

    ![Captura de pantalla que muestra la opción Invitar a un solo miembro.](./media/cloud-academy-sso-tutorial/create-user-1.PNG)

1. Escriba los valores en los campos obligatorios y, a continuación, seleccione **Invite** (Invitar):

    ![Captura de pantalla que muestra el cuadro de diálogo para invitar a un miembro.](./media/cloud-academy-sso-tutorial/create-user-2.PNG)

## <a name="test-sso"></a>Prueba de SSO 

Ahora va a probar la configuración de inicio de sesión único de Azure AD con el Panel de acceso.

Al seleccionar el icono de Cloud Academy - SSO en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Cloud Academy - SSO para la que configuró el inicio de sesión único. Para más información, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para la integración de aplicaciones SaaS con Azure Active Directory ](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Cloud Academy - SSO con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Cloud Academy - SSO con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)