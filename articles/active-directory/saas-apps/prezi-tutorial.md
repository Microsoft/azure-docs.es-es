---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Prezi | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Prezi.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/30/2020
ms.author: jeedes
ms.openlocfilehash: 994d427fb91f04ea0d2002a2c95e12366f4c6420
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553574"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-prezi"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con Prezi

En este tutorial aprenderá a integrar Prezi con Azure Active Directory (Azure AD). Al integrar Prezi con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Prezi.
* Permitir que los usuarios inicien sesión automáticamente en Prezi con sus cuentas de Azure AD.
* Administrar las cuentas en Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Prezi.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Prezi admite el inicio de sesión único iniciado por SP e IDP.
* Prezi admite el aprovisionamiento de usuarios Just-In-Time.
* Una vez configurado Prezi, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. Para más información, consulte [Cómo aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-prezi-from-the-gallery"></a>Adición de Prezi desde la galería

Para configurar la integración de Prezi en Azure AD, será preciso agregar Prezi desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel izquierdo, seleccione **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Prezi** en el cuadro de búsqueda.
1. Seleccione **Prezi** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-prezi"></a>Configuración y prueba del inicio de sesión único de Azure AD para Prezi

Configure y pruebe el inicio de sesión único de Azure AD con Prezi mediante un usuario de prueba llamado B.Simon. Para que el inicio de sesión único funcione, establezca una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Prezi.

Para configurar y probar el inicio de sesión único de Azure AD con Prezi, es preciso completar estos bloques de creación:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para permitir que los usuarios puedan utilizar esta característica.
    1. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con el usuario B.Simon.
    1. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. [Configuración del inicio de sesión único en Prezi](#configure-prezi-sso), para configurar los valores de inicio de sesión único en la aplicación.
    1. [Creación de un usuario de prueba en Prezi](#create-a-prezi-test-user), para tener un homólogo de B.Simon en Prezi vinculado a la representación del usuario en Azure AD.
1. [Comprobación del inicio de sesión único](#test-sso), para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Para habilitar el inicio de sesión único de Azure AD en Azure Portal:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Prezi**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono **Editar** para editar los valores de **Configuración básica de SAML**.

   ![Edición de los valores de Configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

1. Seleccione **Establecer direcciones URL adicionales** y lleve a cabo el siguiente paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://prezi.com/login/sso/`.

1. Seleccione **Guardar**.

1. La aplicación Prezi espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![Atributos y notificaciones de usuario](common/default-attributes.png)

1. La aplicación Prezi espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | given_name | user.givenname |
    | family_name | user.surname |

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base 64)** . Seleccione **Descargar** para descargar el certificado y guárdelo en el equipo.

    ![El vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Prezi**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**. Vaya a **Usuarios** y, a continuación, seleccione **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del usuario, siga estos pasos:
   1. En el cuadro **Nombre**, escriba **B.Simon**.
   1. En el cuadro **Nombre de usuario**, escriba `username@companydomain.extension` (por ejemplo, `B.Simon@contoso.com`).
   1. Active la casilla de verificación **Mostrar contraseña**. Anote el valor que aparece en el cuadro de texto **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Prezi mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Prezi**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Vínculo Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios y haga clic en **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-prezi-sso"></a>Configuración del inicio de sesión único de Prezi

1. En otra ventana del explorador web, inicie sesión en Prezi con su cuenta de equipo y vaya a [Admin Console](https://prezi.com/organizations/manage) (Consola de administración).

1. En **Admin Console** (Consola de administración), seleccione la pestaña **Settings** (Configuración).

    ![Pestaña Settings](./media/prezi-tutorial/settings-image.png)

1. Vaya a la sección **Single Sign-On (SSO)** (Inicio de sesión único [SSO]) y active el conmutador para habilitar el inicio de sesión único.
    
    ![Conmutador de inicio de sesión único (SSO)](./media/prezi-tutorial/single-signon.png)

1. En la sección **Single sign-on (SSO)** (Inicio de sesión único [SSO]), siga estos pasos:

    ![Sección de inicio de sesión único (SSO)](./media/prezi-tutorial/configuration.png)

    1. En el cuadro de texto **Identifier or Issuer URL** (Identificador o dirección URL del emisor), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    1. En el cuadro de texto **SAML 2.0 Endpoint (HTTP)** (Punto de conexión SAML 2.0 [HTTP]), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    1. Abra el **certificado (Base 64)** descargado desde Azure Portal en el bloc de notas. Copie el contenido del certificado y péguelo en el cuadro de texto **Certificate (X.509)** (Certificado [X.509]).

    1. Seleccione **Guardar**.

### <a name="create-a-prezi-test-user"></a>Creación de un usuario de prueba de Prezi

En esta sección, se crea el usuario B.Simon en Prezi. Prezi admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si aún no existe un usuario en Prezi, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el panel de acceso.

Al seleccionar el icono de Prezi en el Panel de acceso, debería iniciar sesión automáticamente en la cuenta de Prezi para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
- [Pruebe Prezi con Azure AD](https://aad.portal.azure.com/)
- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
- [Protección de Prezi con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

