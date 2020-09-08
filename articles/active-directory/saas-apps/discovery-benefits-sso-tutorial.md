---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Discovery Benefits SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Discovery Benefits SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/03/2019
ms.author: jeedes
ms.openlocfilehash: 3beefb467f7875e6bc76765811c6525f5dab9393
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88536248"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-discovery-benefits-sso"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Discovery Benefits SSO

En este tutorial aprenderá a integrar Discovery Benefits SSO con Azure Active Directory (Azure AD). Al integrar Discovery Benefits SSO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Discovery Benefits SSO.
* Permitir que los usuarios inicien sesión automáticamente en Discovery Benefits SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Discovery Benefits SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Discovery Benefits SSO admite el inicio de sesión único iniciado por **IDP**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-discovery-benefits-sso-from-the-gallery"></a>Incorporación de Discovery Benefits SSO desde la galería

Para configurar la integración de Discovery Benefits SSO en Azure AD, tiene que agregar Discovery Benefits SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Discovery Benefits SSO** en el cuadro de búsqueda.
1. Seleccione **Discovery Benefits SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-discovery-benefits-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD para Discovery Benefits SSO

Configure y pruebe el inicio de sesión único de Azure AD con Discovery Benefits SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Discovery Benefits SSO.

Para configurar y probar el inicio de sesión único de Azure AD con Discovery Benefits SSO, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Discovery Benefits SSO](#configure-discovery-benefits-sso-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Discovery Benefits SSO](#create-discovery-benefits-sso-test-user)** , para tener un homólogo de B.Simon en Discovery Benefits SSO que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Discovery Benefits SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección  **Configuración básica de SAML** , la aplicación está preconfigurada en el modo iniciado por  **IDP**  y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón  **Guardar** .

1. La aplicación Discovery Benefits SSO espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

    a. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Identificador de usuario único (id. de nombre)** .

    ![Configuración del inicio de sesión único de Discovery Benefits SSO](./media/discovery-benefits-sso-tutorial/attribute01.png)

    ![Configuración del inicio de sesión único de Discovery Benefits SSO](./media/discovery-benefits-sso-tutorial/attribute02.png)

    b. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Administración de la transformación**.

    c. En el cuadro de texto **Transformación**, escriba el valor de **ToUppercase()** que se muestra para esa fila.

    d. En el cuadro de texto **Parámetro 1**, escriba el parámetro como `<Name Identifier value>`.

    e. Haga clic en **Agregar**.

    > [!NOTE]
    > Discovery Benefits SSO requiere que se pase un valor de cadena fijo en el campo **Identificador de usuario único (id. de nombre)** para que esta integración funcione. Actualmente, Azure AD no es compatible con esta característica por lo que, como solución alternativa, puede usar transformaciones **ToUpper** o **ToLower** de NameID para establecer un valor de cadena fijo tal como se mostró antes en la captura de pantalla.

    f. Hemos rellenado automáticamente las notificaciones adicionales que se requieren para la configuración del inicio de sesión único (`SSOInstance` y `SSOID`). Use el icono de **Editar** para asignar los valores en función de su organización.

    ![Configuración del inicio de sesión único de Discovery Benefits SSO](./media/discovery-benefits-sso-tutorial/attribute03.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Discovery Benefits SSO**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Discovery Benefits SSO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Discovery Benefits SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-discovery-benefits-sso-sso"></a>Configuración del inicio de sesión único de Discovery Benefits SSO

Para configurar el inicio de sesión único en **Discovery Benefits SSO**, es preciso enviar el **certificado (Base64)** descargado y las direcciones URL correspondientes copiadas de Azure Portal al [equipo de soporte técnico de Discovery Benefits SSO](mailto:Jsimpson@DiscoveryBenefits.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-discovery-benefits-sso-test-user"></a>Creación de un usuario de prueba de Discovery Benefits SSO

En esta sección, creará un usuario llamado Britta Simon en Discovery Benefits SSO. Trabaje con el  [equipo de soporte técnico de Discovery Benefits SSO](mailto:Jsimpson@DiscoveryBenefits.com) para agregar los usuarios en esa plataforma. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Discovery Benefits SSO en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Discovery Benefits SSO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Discovery Benefits SSO con Azure AD](https://aad.portal.azure.com/)

