---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Netvision Compas | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Netvision Compas.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/16/2020
ms.author: jeedes
ms.openlocfilehash: 8f92ff60dacd78687207c7523504182521700a7b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554505"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netvision-compas"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Netvision Compas

En este tutorial, aprenderá a integrar Netvision Compas con Azure Active Directory (Azure AD). Al integrar Netvision Compas con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Netvision Compas.
* Permitir que los usuarios inicien sesión automáticamente en Netvision Compas con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Netvision Compas.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Netvision Compas admite el inicio de sesión único iniciado por **SP e IDP**.
* Una vez configurado Netvision Compas, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).


## <a name="adding-netvision-compas-from-the-gallery"></a>Adición de Netvision Compas desde la galería

Para configurar la integración de Netvision Compas en Azure AD, será preciso que agregue Netvision Compas desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Netvision Compas** en el cuadro de búsqueda.
1. Seleccione **Netvision Compas** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-netvision-compas"></a>Configuración y prueba del inicio de sesión único de Azure AD para Netvision Compas

Configure y pruebe el inicio de sesión único de Azure AD con Netvision Compas mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Netvision Compas.

Para configurar y probar el inicio de sesión único de Azure AD con Netvision Compas, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Netvision Compas](#configure-netvision-compas-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Netvision Compas](#configure-netvision-compas-test-user)** , para tener un homólogo de B.Simon en Netvision Compas vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Netvision Compas**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<TENANT>.compas.cloud/Identity/Saml20`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<TENANT>.compas.cloud/Identity/Auth/AssertionConsumerService`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<TENANT>.compas.cloud/Identity/Auth/AssertionConsumerService`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico para clientes de Netvision Compas](mailto:contact@net.vision) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el archivo de metadatos y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)



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

En esta sección va a permitir que B.Simon acceda a Netvision Compas mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Netvision Compas**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-netvision-compas-sso"></a>Configuración del inicio de sesión único de Netvision Compas

En esta sección, habilitará el inicio de sesión único de SAML en **Netvision Compas**.
1. Inicie sesión en **Netvision Compas** con una cuenta administrativa y acceda al área de administración.

    ![Área de administración](media/netvision-compas-tutorial/admin.png)

1. Localice el área **System** (Sistema) y seleccione **Identity Providers** (Proveedores de identidades).

    ![IdP de administración](media/netvision-compas-tutorial/admin-idps.png)

1. Seleccione la acción **Add** (Agregar) para registrar Azure AD como un nuevo IdP.

    ![Adición de IdP](media/netvision-compas-tutorial/idps-add.png)

1. Seleccione **SAML** en **Provider type** (Tipo de proveedor).
1. Escriba valores significativos en los campos **Display name** (Nombre para mostrar) y **Description** (Descripción).
1. Asigne usuarios de **Netvision Compas** al IdP seleccionándolos en la lista **Available users** (Usuarios disponibles) y, a continuación, haciendo clic en el botón **Add selected** (Agregar selección). También se pueden asignar usuarios al IdP mientras se sigue el procedimiento de aprovisionamiento.
1. En la opción **Metadata** (Metadatos) de SAML, haga clic en el botón **Choose File** (Elegir archivo) y seleccione el archivo de metadatos guardado anteriormente en el equipo.
1. Haga clic en **Save**(Guardar).

    ![Edición de IdP](media/netvision-compas-tutorial/idp-edit.png)


### <a name="configure-netvision-compas-test-user"></a>Configuración de un usuario de prueba de Netvision Compas

En esta sección, configurará un usuario existente en **Netvision Compas** para que use Azure AD para en el inicio de sesión único.
1. Siga el procedimiento de aprovisionamiento de usuarios de **Netvision Compas**, tal como se define en la empresa, o edite una cuenta de usuario existente.
1. Al definir el perfil del usuario, asegúrese de que la dirección que figura en **Email (Personal)** (Correo electrónico [personal]) coincide con el nombre de usuario de Azure AD: username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.

    ![Edit user](media/netvision-compas-tutorial/user-config.png)

Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD.

### <a name="using-the-access-panel-idp-initiated"></a>Uso del Panel de acceso (iniciado por IdP)

Al hacer clic en el icono de Netvision Compas del panel de acceso, debería iniciar sesión automáticamente en la instancia de Netvision Compas para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

### <a name="directly-accessing-netvision-compas-sp-initiated"></a>Acceso directo a Netvision Compas (iniciado por el proveedor de servicios).

1. Acceda a la dirección URL de **Netvision Compas**. Por ejemplo, `https://tenant.compas.cloud`.
1. Escriba el nombre de usuario de **Netvision Compas** y seleccione **Next** (Siguiente).

    ![Usuario de inicio de sesión](media/netvision-compas-tutorial/login-user.png)

1. **(opcional)** Si se asignan varios IdP al usuario en **Netvision Compas**, se presenta una lista de los IdP disponibles. Seleccione el proveedor de identidades Azure AD configurado previamente en **Netvision Compas**.

    ![Selección de inicio de sesión](media/netvision-compas-tutorial/login-choose.png)

1. Se le redirigirá a Azure AD para realizar la autenticación. Una vez que se haya autenticado correctamente, iniciará sesión automáticamente en la versión de **Netvision Compas** para la que configuró el inicio de sesión único.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Netvision Compas con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
