---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Lenses.io | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Lenses.io.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 2a0d4a7c-a171-48c6-b1c1-f2bd728fb37f
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 07/02/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: c2b630111261be8e3615ab45e95633040e799551
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050988"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-the-lensesio-dataops-portal"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con el portal DataOps de Lenses.io

En este tutorial, obtendrá información sobre cómo integrar el portal DataOps de [Lenses.io](https://lenses.io/) con Azure Active Directory (Azure AD). Al integrar Lenses.io con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso al portal de Lenses.io.
* Permitir que los usuarios inicien sesión automáticamente en Lenses.io con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una instancia de un portal de Lenses. Puede implementar un portal de Lenses de [varias maneras](https://lenses.io/product/deployment/).
* Una [licencia](https://lenses.io/product/pricing/) de Lenses.io que admita el inicio de sesión único (SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Lenses.io admite el inicio de sesión único iniciado por **SP**.

* Una vez configurado Lenses.io, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-lensesio-from-the-gallery"></a>Incorporación de Lenses.io desde la galería

Para configurar la integración de Lenses.io en Azure AD, será preciso que agregue Lenses.io desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Lenses.io** en el cuadro de búsqueda.
1. Seleccione **Lenses.io** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-lensesio"></a>Configuración y prueba del inicio de sesión único de Azure AD para Lenses.io

Configure y pruebe el inicio de sesión único de Azure AD con el portal de Lenses.io mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Lenses.io.

Para configurar y probar el inicio de sesión único de Azure AD con Lenses.io, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba y un grupo de Azure AD](#create-an-azure-ad-test-user-and-group)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Lenses.io](#configure-lensesio-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de permisos de grupo de prueba de Lenses.io](#create-lensesio-test-group-permissions)** , para controlar qué usuario B. Simon debe acceder en Lenses.io (autorización).
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Lenses.io**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_LENSES_BASE_URL>`; por ejemplo, `https://lenses.my.company.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_LENSES_BASE_URL>`; por ejemplo, `https://lenses.my.company.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_LENSES_BASE_URL>/api/v2/auth/saml/callback?client_name=SAML2Client`
    p. ej. `https://lenses.my.company.com/api/v2/auth/saml/callback?client_name=SAML2Client`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL de inicio de sesión, la dirección URL de respuesta y el identificador reales, según la dirección URL base de la instancia del portal de Lenses. Puede encontrar más información en la [documentación del inicio de sesión único de Lenses.io](https://docs.lenses.io/install_setup/configuration/security.html#single-sign-on-sso-saml-2-0).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configuración de Lenses.io**, use el archivo XML anterior para configurar las lentes en el inicio de sesión único de Azure.

### <a name="create-an-azure-ad-test-user-and-group"></a>Creación de un usuario de prueba y un grupo de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal. También creará un grupo de prueba para B.Simon que se usará para controlar el acceso que B.Simon tiene en Lenses.
Para más información sobre cómo usa Lenses la asignación de pertenencia a grupos para la autorización, consulte la [documentación inicio de sesión único de Lenses](https://docs.lenses.io/install_setup/configuration/security.html#id3).

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

Para crear el grupo:
1. Vuelva a **Azure Active Directory** y seleccione **Grupos**
1. Seleccione **Nuevo grupo** en la parte superior de la pantalla.
1. En **Propiedades de grupo**, siga estos pasos:
   1. En el campo **Tipo de grupo**, seleccione `Security`.
   1. En el campo **Nombre del grupo**, escriba `LensesUsers`
   1. Haga clic en **Crear**.
1. Seleccione el grupo `LensesUsers` y anote el valor de **Identificador de objeto**; por ejemplo, `f8b5c1ec-45de-4abd-af5c-e874091fb5f7`. Este identificador se usará en Lenses para asignar los usuarios de ese grupo a los [permisos correctos](https://docs.lenses.io/install_setup/configuration/security.html#id3).  
   
Para asignar el grupo al usuario de prueba: 
1. Vuelva a **Azure Active Directory** y seleccione **Usuarios**.
1. Seleccione el usuario de prueba `B.Simon`.
1. Seleccione **Grupos**.
1. Seleccione **Agregar pertenencias** en la parte superior de la pantalla.
1. Busque `LensesUsers` y selecciónelo.
1. Haga clic en **Seleccionar**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon use el inicio de sesión único de Azure concediéndole acceso a Lenses.io.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Lenses.io**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-lensesio-sso"></a>Configuración del inicio de sesión único de Lenses.io

Para configurar el inicio de sesión único en el portal de **Lenses.io**, instale el archivo **XML de metadatos de federación** descargado en la instancia de Lenses.io y [configure Lenses.io para habilitar el inicio de sesión único](https://docs.lenses.io/install_setup/configuration/security.html#configure-lenses). 

### <a name="create-lensesio-test-group-permissions"></a>Creación de permisos del grupo de prueba de Lenses.io

En esta sección, creará un grupo en Lenses.io con el valor de **Identificador de objeto** del grupo `LensesUsers` que anotamos en la [sección de creación](#create-an-azure-ad-test-user-and-group) del usuario.
Asigne los permisos deseados que `B.Simon` debería tener en Lenses.io.
Puede encontrar más información en la sección sobre la [asignación de grupos de Lenses en Azure](https://docs.lenses.io/install_setup/configuration/security.html#azure-groups).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Lenses.io en el panel de acceso, debería iniciar sesión automáticamente en el portal de Lenses.io para el que ha configurado el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [ Configuración del inicio de sesión único en la instancia de Lenses.io](https://docs.lenses.io/install_setup/configuration/security.html#single-sign-on-sso-saml-2-0)

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Lenses.io con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Lenses.io con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
