---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Screencast-O-Matic | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Screencast-O-Matic.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/15/2019
ms.author: jeedes
ms.openlocfilehash: e56684c30d243e0b7848355234fef896ccd220ce
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543302"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-screencast-o-matic"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Screencast-O-Matic

En este tutorial aprenderá a integrar Screencast-O-Matic con Azure Active Directory (Azure AD). Al integrar Screencast-O-Matic con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Screencast-O-Matic.
* Permitir que los usuarios inicien sesión automáticamente en Screencast-O-Matic con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción con el inicio de sesión único habilitado de Screencast-O-Matic.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Screencast-O-Matic admite el inicio de sesión único iniciado por **SP**.
* Screencast-O-Matic admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-screencast-o-matic-from-the-gallery"></a>Adición de Screencast-O-Matic desde la galería

Para configurar la integración de Screencast-O-Matic en Azure AD, deberá agregar Screencast-O-Matic desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Screencast-O-Matic** en el cuadro de búsqueda.
1. Seleccione **Screencast-O-Matic** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-screencast-o-matic"></a>Configuración y prueba del inicio de sesión único de Azure AD para Screencast-O-Matic

Configure y pruebe el inicio de sesión único de Azure AD con Screencast-O-Matic mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Screencast-O-Matic.

Para configurar y probar el inicio de sesión único de Azure AD con Screencast-O-Matic, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Screencast-O-Matic](#configure-screencast-o-matic-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en Screencast-O-Matic](#create-screencast-o-matic-test-user)** : para tener un homólogo de B.Simon en Screencast-O-Matic vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Screencast-O-Matic**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://screencast-o-matic.com/<InstanceName>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de atención al cliente de Screencast-O-Matic](mailto:support@screencast-o-matic.com) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up Screencast-O-Matic** (Configurar Screencast-O-Matic), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a Screencast-O-Matic mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Screencast-O-Matic**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-screencast-o-matic-sso"></a>Configuración del inicio de sesión único en Screencast-O-Matic

1. Para automatizar la configuración en Screencast-O-Matic, es preciso instalar **My Apps Secure Sign-in browser extension** (Extensión del explorador de inicio de sesión seguro de Aplicaciones). Para ello, haga clic en **Install the extension** (Instalar la extensión).

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Set up Screencast-O-Matic** (Configurar Screencast-O-Matic) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en Screencast-O-Matic. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3-11.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar Screencast-O-Matic manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Screencast-O-Matic como administrador y lleve a cabo los siguientes pasos:

1. Haga clic en **Suscripción**.

    ![Suscripción](./media/screencast-tutorial/tutorial_screencast_sub.png)

1. En la sección **Access page** (Página de acceso), haga clic en **Configurar**.

    ![Acceso](./media/screencast-tutorial/tutorial_screencast_setup.png)

1. En **Setup Access Page** (Configurar página de acceso), realice los pasos siguientes.

1. En la sección **Access URL** (Dirección URL de acceso), escriba el nombre de instancia en el cuadro de texto indicado.

    ![Acceso](./media/screencast-tutorial/tutorial_screencast_access.png)

1. Seleccione **Require Domain User** (Requerir usuario de dominio) en la sección **SAML User Restriction (optional)** [Restricción de usuario SAML (opcional)].

1. En **Upload IDP Metadata XML File** (Cargar archivo XML de metadatos de IdP), haga clic en **Choose File** (Elegir archivo) para cargar los metadatos que descargó de Azure Portal.

1. Haga clic en **OK**.

    ![Acceso](./media/screencast-tutorial/tutorial_screencast_save.png)

### <a name="create-screencast-o-matic-test-user"></a>Creación de un usuario de prueba de Screencast-O-Matic

En esta sección, se crea un usuario llamado Britta Simon en Screencast-O-Matic. Screencast-O-Matic admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Screencast-O-Matic, se crea uno después de la autenticación. Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de Screencast-O-Matic](mailto:support@screencast-o-matic.com).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Screencast-O-Matic en el panel de acceso, debería iniciar sesión automáticamente en la versión de Screencast-O-Matic para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Screencast-O-Matic con Azure AD](https://aad.portal.azure.com/)