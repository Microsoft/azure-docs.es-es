---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Coggle | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Coggle.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 4193278b-414a-4e1e-8216-aa3848622fd4
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 06/30/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 429571e2dae625664e482482f4245ed7e6325b0a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093754"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-coggle"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Coggle

En este tutorial, aprenderá a integrar Coggle con Azure Active Directory (Azure AD). Al integrar Coggle con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Coggle.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Coggle con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Coggle.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Coggle admite el inicio de sesión único iniciado por **SP e IDP**
* Coggle admite el aprovisionamiento de usuarios **Just-In-Time**

* Una vez que haya configurado Coggle, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-coggle-from-the-gallery"></a>Adición de Coggle desde la galería

Para configurar la integración de Coggle en Azure AD, tendrá que agregar Coggle desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Coggle** en el cuadro de búsqueda.
1. Seleccione **Coggle** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-coggle"></a>Configuración y prueba del inicio de sesión único de Azure AD para Coggle

Configure y pruebe el inicio de sesión único de Azure AD con Coggle mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Coggle.

Para configurar y probar el inicio de sesión único de Azure AD con Coggle, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Coggle](#configure-coggle-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Coggle](#create-coggle-test-user)** , para tener un homólogo de B.Simon en Coggle que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Coggle**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://coggle.it/<TENANT_NAME>/login`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de Coggle](mailto:hello@Coggle.it) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. Haga clic en **Save**(Guardar).

1. La aplicación Coggle espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Coggle espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen|
    | ---------------| --------- |
    | firstName | user.givenname |
    | lastName | user.surname |
    | email | user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configuración de Coggle**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Coggle mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Coggle**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-coggle-sso"></a>Configuración del inicio de sesión único de Coggle

1. En otra ventana del explorador, inicie sesión en el sitio de la compañía de Coggle como administrador.

2. Haga clic en **Coggle** y seleccione **My Settings** (Mi configuración).

    ![Configuración de Coggle](./media/Coggle-tutorial/configure1.png)

3. Haga clic en el siguiente **logotipo** y seleccione **Autenticación**.

    ![Configuración de Coggle](./media/Coggle-tutorial/configure2.png)

4. Haga clic en **Edit SAML Config** (Editar configuración de SAML).

    ![Configuración de Coggle](./media/Coggle-tutorial/configure3.png)

5. En la página de diálogo **SAML Integration** (Integración de SAML), siga estos pasos:

    ![Configuración de Coggle](./media/Coggle-tutorial/configure4.png)

    a. En el cuadro de texto **Entrypoint (ID Provider SSO URL)** (Punto de entrada [Dirección URL del inicio de sesión único del proveedor de identidades)], pegue el valor de la **dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    b. Abra en el Bloc de notas el archivo del **certificado (Base64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **Certificate** (Certificado).

    c. Haga clic en **Guardar**.

### <a name="create-coggle-test-user"></a>Creación de un usuario de prueba de Coggle

En esta sección, se crea un usuario llamado B.Simon en Coggle. Coggle admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario ya no existe en Coggle, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Coggle en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Coggle para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Coggle con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Coggle con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
