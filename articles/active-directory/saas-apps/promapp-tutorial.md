---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Nintex Promapp | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Nintex Promapp.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/30/2020
ms.author: jeedes
ms.openlocfilehash: 3b4102b4e8dcbd5aa04e354cec0ab91bf139572c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553485"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-nintex-promapp"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Nintex Promapp

En este tutorial, aprenderá a integrar Nintex Promapp Azure Active Directory (Azure AD). Al integrar Nintex Promapp con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Nintex Promapp.
* Permitir que los usuarios inicien sesión automáticamente en Nintex Promapp con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en Nintex Promapp.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Nintex Promapp admite el inicio de sesión único iniciado por **SP e IDP**.
* Nintex Promapp admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-nintex-promapp-from-the-gallery"></a>Incorporación de Nintex Promapp desde la galería

Para configurar la integración de Nintex Promapp en Azure AD, deberá agregar Nintex Promapp desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Nintex Promapp** en el cuadro de búsqueda.
1. Seleccione **Nintex Promapp** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-nintex-promapp"></a>Configuración y prueba del inicio de sesión único de Azure AD para Nintex Promapp

Configure y pruebe el inicio de sesión único de Azure AD con Nintex Promapp mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Nintex Promapp.

Para configurar y probar el inicio de sesión único de Azure AD con Nintex Promapp, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Nintex Promapp](#configure-nintex-promapp-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en Nintex Promapp](#create-nintex-promapp-test-user)** , para tener un homólogo de B.Simon en Nintex Promapp que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Nintex Promapp**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    1. En el cuadro **Identificador**, escriba una dirección URL con este patrón:

        ```https
        https://go.promapp.com/TENANTNAME/
        https://au.promapp.com/TENANTNAME/
        https://us.promapp.com/TENANTNAME/
        https://eu.promapp.com/TENANTNAME/
        https://ca.promapp.com/TENANTNAME/
        ```

       > [!NOTE]
       > La integración de Azure AD con Nintex Promapp está configurada actualmente solo para la autenticación iniciada por el servicio (es decir, si va a una dirección URL de Nintex Promapp se inicia el proceso de autenticación). Sin embargo, el campo **Dirección URL de respuesta** es obligatorio.

    1. En el cuadro **Dirección URL de respuesta**, escriba una dirección URL con este patrón:

       `https://<DOMAINNAME>.promapp.com/TENANTNAME/saml/authenticate.aspx`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://<DOMAINNAME>.promapp.com/TENANTNAME/saml/authenticate`.

    > [!NOTE]
    > Estos valores son marcadores de posición. Debe usar los valores reales de identificador, dirección URL de respuesta o dirección URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Nintex Promapp](https://www.promapp.com/about-us/contact-us/) para obtener los valores. También puede consultar los patrones que se muestran en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Nintex Promapp**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a Nintex Promapp mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Nintex Promapp**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-nintex-promapp-sso"></a>Configuración del inicio de sesión único de Nintex Promapp

1. Inicie sesión en el sitio de la empresa Nintex Promapp como administrador.

2. En el menú de la parte superior de la ventana, seleccione **Admin** (Administrador):

    ![Seleccionar Admin (Administrador)][12]

3. Seleccione **Configure** (Configurar):

    ![Seleccionar Configure (Configurar)][13]

4. En el cuadro de diálogo **Security** (Seguridad), siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Security (Seguridad)][14]

    1. Pegue el valor de **Dirección URL de inicio de sesión**  que copió de Azure Portal en el cuadro **SSO-Login URL** (URL de inicio de sesión único).

    1. En la lista **SSO - Single Sign-on Mode** (Modo Inicio de sesión único), seleccione **Optional** (Opcional). Seleccione **Guardar**.

       > [!NOTE]
       > El modo opcional es solo para pruebas. Una vez que esté satisfecho con la configuración, seleccione **Required** (Re requiere) en la lista **SSO - Single Sign-on Mode** (Modo Inicio de sesión único) para forzar a todos los usuarios a autenticarse en Azure AD.

    1. En el Bloc de notas, abra el certificado que descargó en la sección anterior. Copie el contenido del certificado sin la primera línea ( **---BEGIN CERTIFICATE---** ) o la última línea ( **---END CERTIFICATE---** ). Pegue el contenido del certificado en el cuadro **SSO-certificado x.509** y, a continuación, seleccione **Save** (Guardar).

### <a name="create-nintex-promapp-test-user"></a>Creación de un usuario de prueba en Nintex Promapp

En esta sección se crea un usuario llamado B.Simon en Nintex Promapp. Nintex Promapp admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe ya en Nintex Promapp, se crea después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Nintex Promapp en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Nintex Promapp para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Nintex Promapp con Azure AD](https://aad.portal.azure.com/)

<!--Image references-->

[12]: ./media/promapp-tutorial/tutorial_promapp_05.png
[13]: ./media/promapp-tutorial/tutorial_promapp_06.png
[14]: ./media/promapp-tutorial/tutorial_promapp_07.png