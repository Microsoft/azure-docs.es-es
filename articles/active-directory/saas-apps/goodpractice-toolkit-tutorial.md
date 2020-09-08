---
title: 'Tutorial: Integración de Azure Active Directory con Mind Tools Toolkit | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Mind Tools Toolkit.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/12/2020
ms.author: jeedes
ms.openlocfilehash: 10773a2a379291fa861dbb2adfdda2ba3f63117f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550646"
---
# <a name="tutorial-azure-active-directory-integration-with-mind-tools-toolkit"></a>Tutorial: Integración de Azure Active Directory con Mind Tools Toolkit

En este tutorial, obtendrá información sobre cómo integrar Mind Tools Toolkit con Azure Active Directory (Azure AD).

Con esta integración, puede:

* Controlar en Azure AD quién tiene acceso a Mind Tools Toolkit.
* Permitir que los usuarios inicien sesión automáticamente en Mind Tools Toolkit (inicio de sesión único) con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Mind Tools Toolkit, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Mind Tools Toolkit.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Mind Tools Toolkit admite el inicio de sesión único iniciado por SP.
* Mind Tools Toolkit admite el aprovisionamiento de usuarios Just-In-Time.
* Después de configurar Mind Tools Toolkit, puede aplicar el control de sesión. Este control protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-mind-tools-toolkit-from-the-gallery"></a>Adición de Mind Tools Toolkit desde la galería

Para configurar la integración de Mind Tools Toolkit en Azure AD, será preciso que agregue Mind Tools Toolkit desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Mind Tools Toolkit** en el cuadro de búsqueda.
1. Seleccione **Mind Tools Toolkit** en los resultados de la búsqueda y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección configurará y probará el inicio de sesión único de Azure AD con Mind Tools Toolkit mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Mind Tools Toolkit.

Para configurar y probar el inicio de sesión único de Azure AD con Mind Tools Toolkit, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Mind Tools Toolkit](#configure-mind-tools-toolkit-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Cree un usuario de prueba de Test Tools Toolkit](#create-a-mind-tools-toolkit-test-user)** , para tener un homólogo de B.Simon en Mind Tools Toolkit. Este homólogo está vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, va a configurar el inicio de sesión único de Azure AD con Mind Tools Toolkit mediante estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Mind Tools Toolkit**, seleccione **Inicio de sesión único**.

    ![Sección Administrar, con Inicio de sesión único resaltado](common/select-sso.png)

1. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Seleccione un método de inicio de sesión único, con SAML resaltado](common/select-saml-option.png)

1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

    ![Página Configuración del inicio de sesión único con SAML con el icono de lápiz de Configuración básica de SAML resaltado](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, en el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el patrón `https://app.goodpractice.net/#/<subscriptionUrl>/s/<locationId>`.

    > [!NOTE]
    > El valor de **URL de inicio de sesión** no es real. Actualícelo con la dirección URL de inicio de sesión real. Pónganse en contacto con el [equipo de soporte técnico de Mind Tools Toolkit](mailto:support@goodpractice.com) para obtener este valor.

1. En la página **Configuración del inicio de sesión único con SAML**, vaya a la sección **Certificado de firma de SAML**. A la derecha de **XML de metadatos de federación**, seleccione **Descargar** para descargar el texto XML y guardarlo en el equipo. El contenido XML depende de las opciones que seleccione.

    ![Sección Certificado de firma de SAML, con Descargar resaltado junto a XML de metadatos de federación](common/metadataxml.png)

1. En la sección **Configurar Mind Tools Toolkit**, copie cualquiera de las siguientes direcciones URL según sus necesidades.

    * **Dirección URL de inicio de sesión**

    * **Identificador de Azure AD**

    * **Dirección URL de cierre de sesión**

    ![Sección Configurar Mind Tools Toolkit, con las direcciones URL de configuración resaltadas](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, se crea un usuario llamado B.Simon en Azure Portal:

1. En Azure Portal, en el lado izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. En la parte superior de la pantalla, seleccione **Nuevo usuario**.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba **B.Simon**.  
   1. En el campo **Nombre de usuario**, escriba **B.Simon@** _dominioDeEmpresa_ **.** _extensión_. Por ejemplo, B.Simon@contoso.com.
   1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a Mind Tools Toolkit mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Mind Tools Toolkit**.
1. En la página de información general de la aplicación, vaya a la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Sección Administrar, con la opción Usuarios y grupos resaltada](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

   ![Ventana Usuarios y grupos, con Agregar usuario resaltado](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-mind-tools-toolkit-sso"></a>Configuración del inicio de sesión único de Mind Tools Toolkit

Para configurar el inicio de sesión único en **Mind Tools Toolkit**, es preciso enviar el texto **XML de metadatos de federación** descargado y las direcciones URL copiadas previamente de Azure Portal al [equipo de soporte técnico de Mind Tools Toolkit](mailto:support@goodpractice.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-a-mind-tools-toolkit-test-user"></a>Creación de un usuario de prueba de Mind Tools Toolkit

En esta sección, va a crear un usuario llamado B.Simon en Mind Tools Toolkit.

Mind Tools Toolkit admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. No es necesaria ninguna acción por su parte en esta sección. Si el usuario no existe todavía en Mind Tools Toolkit, se crea uno nuevo cuando se intenta acceder a Mind Tools Toolkit.

### <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Al seleccionar el icono de Mind Tools Toolkit en el portal Aplicaciones, debería iniciar sesión automáticamente en la instancia de Mind Tools Toolkit para la que configuró el inicio de sesión único. Para más información acerca del portal Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales acerca de la integración de aplicaciones SaaS en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Mind Tools Toolkit con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Mind Tools Toolkit con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
