---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Meraki Dashboard | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Meraki Dashboard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: jeedes
ms.openlocfilehash: f4a4c38cf079c22dbd2b8eda5e68cc3f147b1fc0
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88535018"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-meraki-dashboard"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Meraki Dashboard

En este tutorial, aprenderá a integrar Meraki Dashboard con Azure Active Directory (Azure AD). Al integrar Meraki Dashboard con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Meraki Dashboard.
* Permitir que los usuarios inicien sesión automáticamente en Meraki Dashboard con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Meraki Dashboard.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* El panel de Meraki Dashboard admite el inicio de sesión único iniciado por **IDP**
* Una vez configurado Meraki Dashboard, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-meraki-dashboard-from-the-gallery"></a>Incorporación de Meraki Dashboard desde la galería

Para configurar la integración de Meraki Dashboard en Azure AD, deberá agregar Meraki Dashboard desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Meraki Dashboard** en el cuadro de búsqueda.
1. Seleccione **Meraki Dashboard** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-meraki-dashboard"></a>Configuración y prueba del inicio de sesión único de Azure AD para Meraki Dashboard

Configure y pruebe el inicio de sesión único de Azure AD con Meraki Dashboard con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Meraki Dashboard.

Para configurar y probar el inicio de sesión único de Azure AD con Meraki Dashboard, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Meraki Dashboard](#configure-meraki-dashboard-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Meraki Dashboard](#create-meraki-dashboard-test-user)** , para tener un homólogo de B.Simon en Meraki Dashboard que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Meraki Dashboard**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:
     
    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://n27.meraki.com/saml/login/m9ZEgb/< UNIQUE ID >`.

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Actualice este valor con la dirección URL de respuesta real, que se explica más adelante en el tutorial.

1. Haga clic en el botón **Save** (Guardar).

1. La aplicación Meraki Dashboard espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Meraki Dashboard espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen|
    | ---------------| --------- |
    | `https://dashboard.meraki.com/saml/attributes/username` | user.userprincipalname |
    | `https://dashboard.meraki.com/saml/attributes/role` | user.assignedroles |

    > [!NOTE]
    > Para aprender a configurar roles en Azure AD, consulte [este vínculo](https://docs.microsoft.com/azure/active-directory/develop/active-directory-enterprise-app-role-management).

1. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

1. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** y guárdelo en el equipo.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

1. En la sección **Configurar Meraki Dashboard**, copie el valor de la dirección URL de cierre de sesión y guárdelo en el equipo.

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

En esta sección, va a permitir que B.Simon acceda a Meraki Dashboard mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Meraki Dashboard**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-meraki-dashboard-sso"></a>Configuración del inicio de sesión único de Meraki Dashboard

1. En otra ventana del explorador web, inicie sesión en el sitio web de Meraki Dashboard como administrador.

1. Vaya a **Organization** (Organización)  -> **Configuration** (Configuración).

    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure1.png)

1. En Autenticación, cambie **SAML SSO** (SSO de SAML) a **SAML SSO enabled** (SSO de SAML habilitado).

    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure2.png)

1. Haga clic en **Add a SAML IdP** (Agregar un IdP de SAML).

    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure3.png)

1. Pegue el valor de **Thumprint** (Huella digital), que ha copiado de Azure Portal en **X.590 cert SHA1 fingerprint** (Huella digital de SHA1 de certificado X.590). A continuación, haga clic en **Save**(Guardar). Después de guardar, se mostrará la dirección URL del consumidor. Copie el valor de Consumer URL (URL de cliente) y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** en Azure Portal.

    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure4.png)

### <a name="create-meraki-dashboard-test-user"></a>Creación de un usuario de prueba de Meraki Dashboard

1. En otra ventana del explorador web, inicie sesión en el sitio web de Meraki Dashboard como administrador.

1. Vaya a **Organization** (Organización)  -> **Administrators** (Administradores).

    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/user1.png)

1. En la sección roles de administrador de SAML, haga clic en el botón **Add SAML role** (Agregar rol de SAML).

    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/user2.png)

1. Escriba el rol **meraki_full_admin**, marque **Organization access** (Acceso de la organización) como **Full** (Completo) y haga clic en **Create role** (Crear rol). Repita el proceso para **meraki_readonly_admin** y, esta vez, marque **Organization access** (Acceso de la organización) como **Read-only** (Solo lectura).
 
    ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/user3.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Meraki Dashboard en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Meraki Dashboard para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Meraki Dashboard con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Meraki Dashboard con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

