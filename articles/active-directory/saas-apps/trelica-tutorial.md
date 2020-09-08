---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Trelica | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Trelica.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: jeedes
ms.openlocfilehash: dcc304febe6fcebe3aba3047d1773e695b80f9e2
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551948"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-trelica"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Trelica

En este tutorial, aprenderá a integrar Trelica con Azure Active Directory (Azure AD).

Con esta integración, puede:

* Controlar en Azure AD quién tiene acceso a Trelica.
* Permitir que los usuarios inicien sesión automáticamente en Trelica con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Trelica.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Trelica admite el inicio de sesión único iniciado por IDP.
* Trelica admite el aprovisionamiento de usuarios Just-In-Time.
* Después de configurar Trelica, puede aplicar el control de sesión. Este control protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-trelica-from-the-gallery"></a>Adición de Trelica desde la galería

Para configurar la integración de Trelica en Azure AD, deberá agregar Trelica desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Trelica** en el cuadro de búsqueda.
1. Seleccione **Trelica** en los resultados de la búsqueda y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-trelica"></a>Configuración y prueba del inicio de sesión único de Azure AD para Trelica

Configure y pruebe el inicio de sesión único de Azure AD con Trelica mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculo entre un usuario de Azure AD y el usuario relacionado de Trelica.

Para configurar y probar el inicio de sesión único de Azure AD con Trelica, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Trelica](#configure-trelica-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Trelica](#create-a-trelica-test-user)** , para tener un homólogo de B.Simon en Trelica. Este homólogo está vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Trelica**, busque la sección **Administrar**. Seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Página Configuración del inicio de sesión único con SAML con el icono de lápiz de Configuración básica de SAML resaltado](common/edit-urls.png)

1. En la página **Configuración del inicio de sesión único con SAML**, escriba los valores siguientes:

    1. En el cuadro de texto **Identificador**, escriba la dirección URL **https://app.trelica.com** .

    1. En el cuadro de texto **Dirección URL de respuesta**, escriba una dirección URL con el patrón `https://app.trelica.com/Id/Saml2/<CUSTOM_IDENTIFIER>/Acs`.

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Actualice este valor con la dirección URL de respuesta real (también llamada ACS).
    > Para encontrarla, inicie sesión en Trelica y vaya a la [página de configuración de proveedores de identidades SAML](https://app.trelica.com/Admin/Profile/SAML) (Admin [Administrador] > Account [Cuenta] > SAML). Haga clic en el botón Copiar junto a la **dirección URL del servicio de consumidor de aserciones (ACS)** para colocar la información en el portapapeles, lista para pegarla en el cuadro de texto **Dirección URL de respuesta** de Azure AD.
    > Si tiene alguna duda, consulte la [documentación de ayuda de Trelica](https://docs.trelica.com/admin/saml/azure-ad) o póngase en contacto con el [equipo de soporte técnico del cliente de Trelica](mailto:support@trelica.com).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Sección Certificado de firma de SAML, con el botón de copia resaltado junto a Dirección URL de metadatos de federación de aplicación](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, se crea un usuario llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. En la parte superior de la pantalla, seleccione **Nuevo usuario**.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba **B.Simon**.
   1. En el campo **Nombre de usuario**, escriba **B.Simon@** _dominioDeEmpresa_ **.** _extensión_. Por ejemplo, B.Simon@contoso.com.
   1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Trelica mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Trelica**.
1. En la página de información general de la aplicación, vaya a la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Sección Administrar, con la opción Usuarios y grupos resaltada](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

   ![Ventana Usuarios y grupos, con Agregar usuario resaltado](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-trelica-sso"></a>Configuración del inicio de sesión único de Trelica

Para configurar el inicio de sesión único en **Trelica**, vaya a la [página de configuración de proveedores de identidades de SAML](https://app.trelica.com/Admin/Profile/SAML) (Admin [Administrador] > Account [Cuenta] > SAML). Haga clic en el botón **Nuevo**. Escriba **Azure AD** como nombre y, para el tipo de metadatos, elija **Metadata from url** (Metadatos de la dirección URL). Pegue la **dirección URL de metadatos de federación de aplicación** que tomó de Azure AD en el campo de la **dirección URL de metadatos** de Trelica.

Si tiene alguna duda, consulte la [documentación de ayuda de Trelica](https://docs.trelica.com/admin/saml/azure-ad) o póngase en contacto con el [equipo de soporte técnico del cliente de Trelica](mailto:support@trelica.com).

### <a name="create-a-trelica-test-user"></a>Creación de un usuario de prueba de Trelica

Trelica admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No es necesaria ninguna acción por su parte en esta sección. Si el usuario no existe ya en Trelica, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Al seleccionar el icono de Trelica en el portal Aplicaciones, debería iniciar sesión automáticamente en la instancia de Trelica para la que configuró el inicio de sesión único. Para más información acerca del portal Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales acerca de la integración de aplicaciones SaaS en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Trelica con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Trelica con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
