---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con RSA Archer Suite | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y RSA Archer Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/01/2020
ms.author: jeedes
ms.openlocfilehash: fa8b150e7b5e1bd5bfc5e05b3b00a13522b52f87
ms.sourcegitcommit: 206629373b7c2246e909297d69f4fe3728446af5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/06/2020
ms.locfileid: "89500590"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-rsa-archer-suite"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con RSA Archer Suite

En este tutorial, aprenderá a integrar RSA Archer Suite con Azure Active Directory (Azure AD). Al integrar RSA Archer Suite con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a RSA Archer Suite.
* Permitir que los usuarios inicien sesión automáticamente en RSA Archer Suite con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en RSA Archer Suite.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* RSA Archer Suite admite el inicio de sesión único iniciado por **SP**.
* RSA Archer Suite admite el aprovisionamiento de usuarios **Just-In-Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-rsa-archer-suite-from-the-gallery"></a>Adición de RSA Archer Suite desde la galería

Para configurar la integración de RSA Archer Suite en Azure AD, deberá agregar RSA Archer Suite desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **RSA Archer Suite** en el cuadro de búsqueda.
1. Seleccione **RSA Archer Suite** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-rsa-archer-suite"></a>Configuración y prueba del inicio de sesión único de Azure AD para RSA Archer Suite

Configure y pruebe el inicio de sesión único de Azure AD con RSA Archer Suite mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de RSA Archer Suite.

Para configurar y probar el inicio de sesión único de Azure AD con RSA Archer Suite, es preciso completar los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en RSA Archer Suite](#configure-rsa-archer-suite-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de RSA Archer Suite](#create-rsa-archer-suite-test-user)** : para tener un homólogo de B.Simon en RSA Archer Suite vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **RSA Archer Suite**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<BASE_URL>/default.aspx?IDP=<REALM_NAME>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba el valor: `RSAArcherSuite_TENANT_STRING`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualice el valor con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de RSA Archer Suite](mailto:archersupport@rsa.com) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación RSA Archer Suite espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación RSA Archer Suite espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | -------------- | --------- |
    | Nombre | user.givenname |
    | Apellidos | user.surname |
    | PhoneNumber | user.telephonenumber |
    | City | user.city |
    | Zipcode | user.postalcode |
    | State | user.state |
    | Calle | user.streetaddress |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar RSA Archer Suite**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a RSA Archer Suite mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **RSA Archer Suite**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-rsa-archer-suite-sso"></a>Configuración del inicio de sesión único de RSA Archer Suite

1. En otra ventana del explorador, inicie sesión en el sitio web de RSA Archer Suite como administrador.

1. En la página siguiente, realice estos pasos.

    ![Configuración del inicio de sesión único de RSA Archer Suite](./media/rsa-archer-suite-tutorial/configuring-saml-sso.png)

    a. Vaya a la pestaña **Single Sign-On** (Inicio de sesión único) y seleccione **SAML** en **Single Sign-On Mode** (Modo de inicio de sesión único) en el desplegable.

    b. Active la casilla **Allow manual bypass** (Permitir la omisión manual).

    c. Proporcione un nombre válido en el cuadro de texto **Instance Entity ID** (Identificador de entidad de la instancia).

    d. Pegue el valor de la **Huella digital** en el cuadro de texto **Certificate Thumbprint** (Huella digital del certificado).

    e. Haga clic en el botón **Select** (Seleccionar) y cargue el archivo **XML de metadatos de federación** descargado de Azure Portal.

    f. Haga clic en **Save** (Guardar) para guardar la configuración de inicio de sesión único. 

### <a name="create-rsa-archer-suite-test-user"></a>Creación de un usuario de prueba de RSA Archer Suite

En esta sección se crea una usuario llamado B.Simon en RSA Archer Suite. RSA Archer Suite admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe aún en RSA Archer Suite, se creará uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

1. Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de RSA Archer Suite, donde puede iniciar el flujo de inicio de sesión. 

2. Vaya directamente a la dirección URL de inicio de sesión de RSA Archer Suite e inicie el flujo de inicio de sesión desde allí.

3. Puede usar el Panel de acceso de Microsoft. Al hacer clic en el icono de RSA Archer Suite en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de RSA Archer Suite para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

 Una vez configurado RSA Archer Suite, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

