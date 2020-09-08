---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Raketa | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Raketa.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/28/2020
ms.author: jeedes
ms.openlocfilehash: f5a01d74d22d6ea010d80ef1ef8ce1a53fbd4422
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548820"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-raketa"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Raketa

En este tutorial, aprenderá a integrar Raketa con Azure Active Directory (Azure AD). Al integrar Raketa con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Raketa.
* Permitir que los usuarios inicien sesión automáticamente en Raketa con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Raketa.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Raketa admite el inicio de sesión único iniciado por **SP**.
* Una vez que haya configurado Raketa, puede aplicar el control de sesión, que protege a su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-raketa-from-the-gallery"></a>Adición de Raketa desde la galería

Para configurar la integración de Raketa en Azure AD, es preciso agregar Raketa desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory** [1].

    ![rkt_1](./media/raketa-tutorial/azure-active-directory.png)

1. Vaya a **Aplicaciones empresariales** [2] y seleccione **Todas las aplicaciones** [3].

1. Para agregar una nueva aplicación, seleccione **Nueva aplicación** [4]. 

    ![rkt_2](./media/raketa-tutorial/new-app.png)

1. En la sección **Agregar desde la galería** [5], escriba **Raketa** en el cuadro de búsqueda [6].

1. Seleccione **Raketa** en el panel de resultados [7] y, a continuación, haga clic en el botón **Agregar** [8]. 

    ![rkt_3](./media/raketa-tutorial/add-btn.png)


## <a name="configure-and-test-azure-ad-single-sign-on-for-raketa"></a>Configuración y prueba del inicio de sesión único de Azure AD para Raketa

Configure y pruebe el inicio de sesión único de Azure AD con Raketa mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Raketa.

Para configurar y probar el inicio de sesión único de Azure AD con Raketa, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Raketa](#configure-raketa-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Raketa](#create-raketa-test-user)** : para tener un homólogo de B.Simon en Raketa que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Raketa**, busque la sección **Administrar** y seleccione **Inicio de sesión único** [9].

    ![rkt_4](./media/raketa-tutorial/manage-sso.png)

1. En la página **Seleccione un método de inicio de sesión único** [9], elija **SAML** [10].

    ![rkt_5](./media/raketa-tutorial/saml.png)

1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** [11] y modificar los ajustes.

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    1. En los cuadros de texto **Identificador (id. de entidad)** [12] y **URL de inicio de sesión** [14], escriba la dirección URL: `https://raketa.travel/`.

    1. En el cuadro de texto **URL de respuesta** [13], escriba una dirección URL con el siguiente patrón: `https://raketa.travel/sso/acs?clientId=<CLIENT_ID>`.  

    ![rkt_6](./media/raketa-tutorial/enter-urls.png)

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Actualícelo con la dirección URL de respuesta real. Póngase en contacto con el [equipo de soporte técnico al cliente de Raketa](mailto:help@raketa.travel) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** [15] para descargarlo y guardarlo en el equipo.

1. En la sección **Configurar Raketa**, copie las direcciones URL adecuadas según sus necesidades.

    1. URL de inicio de sesión [16]: la dirección URL de la página web de autorización, que se usa para redirigir a los usuarios al sistema de autenticación.

    1. Identificador de Azure AD [17]: identificador de Azure AD.

    1. URL de cierre de sesión [18]: la dirección URL de la página web, que se usa para redirigir a los usuarios después del cierre de sesión.

    ![rkt_7](./media/raketa-tutorial/copy-urls.png)


### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory** [1], **Usuarios** [19] y **Todos los usuarios** [20].

1. Seleccione **Nuevo usuario** [21] en la parte superior de la pantalla.

    ![rkt_8](./media/raketa-tutorial/new-user.png)

1. En las propiedades del **usuario**, siga estos pasos:

   1. En el campo **Nombre de usuario** [22], escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.

   1. En el campo **Nombre** [23], escriba `B.Simon`.

   1. Active la casilla **Mostrar contraseña** [25] y, después, anote el valor que se muestra en el cuadro **Contraseña** [24].

   1. Haga clic en **Crear** [26]. 

    ![rkt_9](./media/raketa-tutorial/create-user.png)


### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Raketa mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** [2] y **Todas las aplicaciones** [3].

1. En la lista de aplicaciones, seleccione **Raketa** [27].  

    ![rkt_10](./media/raketa-tutorial/add-raketa.png)

1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos** [28]. 

    ![rkt_11](./media/raketa-tutorial/users-groups.png)

1. Seleccione **Agregar usuario** [29]. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos** [30].

    ![rkt_12](./media/raketa-tutorial/add-user-raketa.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** [31] de la lista de usuarios y haga clic en el botón **Seleccionar** [32] de la parte inferior de la pantalla.

1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar** [33]. 

    ![rkt_13](./media/raketa-tutorial/assign-user.png)

## <a name="configure-raketa-sso"></a>Configuración del inicio de sesión único de Raketa

Para configurar el inicio de sesión único en **Raketa**, es preciso enviar el **certificado (Base 64)** descargado y las direcciones URL correspondientes copiadas de Azure Portal al [equipo de soporte técnico de Raketa](mailto:help@raketa.travel). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-raketa-test-user"></a>Creación de un usuario de prueba de Raketa

En esta sección creará un usuario llamado B.Simon en Raketa. Colabore con el [equipo de soporte técnico de Raketa](mailto:help@raketa.travel) para agregar los usuarios en la plataforma de Raketa. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Raketa en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Raketa para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Raketa con Azure AD](https://aad.portal.azure.com/).

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Raketa con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)