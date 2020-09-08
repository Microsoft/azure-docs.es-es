---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Pipedrive | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Pipedrive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/06/2020
ms.author: jeedes
ms.openlocfilehash: b3b2032d8cefe881e59fe339786877c4f03c9305
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553791"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pipedrive"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Pipedrive

En este tutorial, aprenderá a integrar Pipedrive con Azure Active Directory (Azure AD). Al integrar Pipedrive con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Pipedrive.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Pipedrive con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Pipedrive.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Pipedrive admite el inicio de sesión único iniciado por **SP e IDP**.
* Una vez configurado el inicio de sesión único de Pipedrive, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


## <a name="adding-pipedrive-from-the-gallery"></a>Adición de Pipedrive desde la galería

Para configurar la integración de Pipedrive en Azure AD, es preciso agregar Pipedrive desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Pipedrive** en el cuadro de búsqueda.
1. Seleccione **Pipedrive** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-pipedrive"></a>Configuración y prueba del inicio de sesión único de Azure AD para Pipedrive

Configure y pruebe el inicio de sesión único de Azure AD con Pipedrive mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Pipedrive.

Para configurar y probar el inicio de sesión único de Azure AD con Pipedrive, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Pipedrive](#configure-pipedrive-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de Pipedrive](#create-pipedrive-test-user)** : para tener un homólogo de B.Simon en Pipedrive que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Pipedrive**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<COMPANY-NAME>.pipedrive.com/sso/auth/samlp/metadata.xml`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY-NAME>.pipedrive.com/sso/auth/samlp`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY-NAME>.pipedrive.com/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Pipedrive](mailto:support@pipedrive.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Pipedrive espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Pipedrive espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ------------ | --------- |
    | email | user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (base 64)** y seleccione **Descargar** para descargar el certificado y guardarlo en el equipo; copie también **Dirección URL de metadatos de federación de la aplicación** y guárdela en el equipo.

    ![Vínculo de descarga del certificado](./media/pipedrive-tutorial/certificate-data.png)

1. En la sección **Configurar Pipedrive**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Pipedrive mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Pipedrive**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-pipedrive-sso"></a>Configuración del inicio de sesión único de Pipedrive

1. En otra ventana del explorador, inicie sesión en el sitio web de Pipedrive como administrador.

1. Haga clic en **User profile** (Perfil de usuario) y seleccione **Settings** (Configuración).

    ![Configuración de Pipedrive](./media/pipedrive-tutorial/configure1.png)

1. Desplácese hacia abajo hasta Security Center y seleccione **Single sign-on** (Inicio de sesión único).

    ![Configuración de Pipedrive](./media/pipedrive-tutorial/configure2.png)

1. En la sección **SAML configuration for Pipedrive** (Configuración de SAML para Pipedrive), siga estos pasos:

    ![Configuración de Pipedrive](./media/pipedrive-tutorial/configure3.png)

    a. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Dirección URL de metadatos de federación de aplicación** que copió de Azure Portal.

    b. En el cuadro de texto **Single Sign On (SSO) URL** (Dirección URL de inicio de sesión único [SSO]), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Single Log Out (SLO) URL** (Dirección URL de cierre de sesión único [SLO]), pegue el valor de **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **X.509 Certificate** (Certificado X.509), abra el archivo **Certificado (base 64)** descargado de Azure Portal en el Bloc de notas, copie el contenido del mismo, péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509) y guarde los cambios.

### <a name="create-pipedrive-test-user"></a>Creación de un usuario de prueba de Pipedrive

1. En otra ventana del explorador, inicie sesión en el sitio web de Pipedrive como administrador.

1. Desplácese hacia abajo hasta empresa y seleccione **Manage users** (Administrar usuarios).

    ![Configuración de Pipedrive](./media/pipedrive-tutorial/user1.png)

1. Haga clic en **Add users** (Agregar usuarios).
    
    ![Configuración de Pipedrive](./media/pipedrive-tutorial/user2.png)

1. En la sección **Manage users** (Administrar usuarios), realice los siguientes pasos:

    ![Configuración de Pipedrive](./media/pipedrive-tutorial/user3.png)

    a. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico del usuario, en este caso, `B.Simon@contoso.com`.

    b. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario.

    c. En el cuadro de texto **Last name** (Apellidos), escriba los apellidos del usuario.

    d. Haga clic en **Confirm and invite users** (Confirmar e invitar a los usuarios).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Pipedrive en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Pipedrive para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Pipedrive con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)