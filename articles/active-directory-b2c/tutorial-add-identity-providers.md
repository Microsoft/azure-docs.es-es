---
title: 'Tutorial: Incorporación de proveedores de identidades a las aplicaciones'
titleSuffix: Azure AD B2C
description: Siga este tutorial para aprender a agregar proveedores de identidades a las aplicaciones en Azure Active Directory B2C mediante Azure Portal.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9f9abf9105da773ec5f8321c0f8e70e20516618c
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922156"
---
# <a name="tutorial-add-identity-providers-to-your-applications-in-azure-active-directory-b2c"></a>Tutorial: Incorporación de proveedores de identidades a las aplicaciones en Azure Active Directory B2C

En las aplicaciones, es posible que quiera permitir que los usuarios inicien sesión con distintos proveedores de identidades. Un *proveedor de identidades* crea, mantiene y administra la información de identidad al tiempo que proporciona servicios de autenticación a las aplicaciones. Puede agregar proveedores de identidades compatibles con Azure Active Directory B2C (Azure AD B2C) a los [flujos de usuario](user-flow-overview.md) mediante Azure Portal.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Crear las aplicaciones del proveedor de identidades
> * Agregar los proveedores de identidades a su inquilino, tanto en Facebook como en Azure Active Directory
> * Agregar los proveedores de identidades al flujo del usuario

Por lo general, solo se usa un proveedor de identidades en las aplicaciones, pero existe la opción de agregar más. En este tutorial se muestra cómo agregar un proveedor de identidades de Azure AD y un proveedor de identidades de Facebook a la aplicación. Agregar ambos tipos de proveedores de identidades a la aplicación es opcional. También puede agregar otros proveedores de identidades, como [Amazon](identity-provider-amazon.md), [GitHub](identity-provider-github.md), [Google](identity-provider-google.md), [LinkedIn](identity-provider-linkedin.md), [Microsoft](identity-provider-microsoft-account.md) o [Twitter](identity-provider-twitter.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

[Cree un flujo de usuario](tutorial-create-user-flows.md) para permitir a los usuarios registrarse e iniciar sesión en la aplicación.

## <a name="create-applications"></a>Crear aplicaciones

Las aplicaciones del proveedor de identidades proporcionó el identificador y la clave para habilitar la comunicación con el inquilino de Azure AD B2C. En esta sección del tutorial, puede crear una aplicación de Azure AD y una aplicación de Facebook desde la cual se obtienen los identificadores y las claves para agregar los proveedores de identidades al inquilino. Si va a agregar solo uno de los proveedores de identidades, solo debe crear la aplicación correspondiente a ese proveedor.

### <a name="create-an-azure-active-directory-application"></a>Creación de una aplicación de Azure Active Directory

Para habilitar el inicio de sesión de los usuarios de Azure AD, deberá registrar una aplicación dentro del inquilino de Azure AD. El inquilino de Azure AD no es el mismo que el usuario de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD. Para ello, seleccione el filtro **Directorio y suscripción** que se encuentra en el menú superior y elija el directorio que contiene el inquilino de Azure AD.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**.
1. Escriba un nombre para la aplicación. Por ejemplo, `Azure AD B2C App`.
1. Acepte la selección de **Solo las cuentas de este directorio organizativo** para esta aplicación.
1. Para la **URI de redirección**, acepte el valor de **Web**, escriba la siguiente URL en minúscula y reemplace `your-B2C-tenant-name` por el nombre del inquilino de Azure AD B2C.

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Por ejemplo, `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`.

    Todas las direcciones URL ahora deberían estar utilizando [b2clogin.com](b2clogin.md).

1. Seleccione **Registrar** y después anote el **Id. de aplicación (cliente)** que usará en un paso posterior.
1. En **Administrar**, en el menú de aplicaciones, seleccione **Certificados y secretos** y luego **Nuevo secreto de cliente**.
1. Escriba una **descripción** del secreto de cliente. Por ejemplo, `Azure AD B2C App Secret`.
1. Seleccione el período de expiración. Para esta aplicación, acepte la selección de **En 1 año**.
1. Seleccione **Agregar** y después anote el valor del nuevo secreto de cliente que usará en un paso posterior.

### <a name="create-a-facebook-application"></a>Creación de una aplicación de Facebook

Para usar una cuenta de Facebook como proveedor de identidades en Azure AD B2C, debe crear una aplicación en Facebook. Si aún no tiene una cuenta de Facebook, puede obtenerla en [https://www.facebook.com/](https://www.facebook.com/).

1. Inicie sesión en [Facebook for developers](https://developers.facebook.com/) con las credenciales de su cuenta de Facebook.
1. Si aún no lo ha hecho, debe registrarse como desarrollador de Facebook. Para ello, seleccione **Get Started** (Comenzar) en la esquina superior derecha de la página, acepte las directivas de Facebook y complete los pasos de registro.
1. Seleccione **Mis aplicaciones** y después **Crear una aplicación**.
1. Especifique el valor de **Display Name** (Nombre para mostrar) y un valor de **Contact Email** (Correo electrónico de contacto) válido.
1. Haga clic en **Create App ID** (Crear id. de aplicación). Es posible que deba aceptar las políticas de la plataforma Facebook y realizar una comprobación de seguridad en línea.
1. Seleccione **Settings** (Configuración)  > **Basic** (Básica).
1. Elija una **Categoría**, por ejemplo, `Business and Pages`. Facebook requiere este valor, pero Azure AD B2C no lo usa.
1. En la parte inferior de la página, seleccione **Add Platform** (Agregar plataforma) y, después, seleccione **Website** (Sitio web).
1. En **URL del sitio**, escriba `https://your-tenant-name.b2clogin.com/`, pero sustituya `your-tenant-name` por el nombre del inquilino.
1. Escriba una dirección URL en **Privacy Policy URL** (URL de directiva de privacidad), por ejemplo `http://www.contoso.com/`. La URL de la directiva de privacidad es una página que mantiene para proporcionar la información de privacidad de la aplicación.
1. Seleccione **Save changes** (Guardar los cambios).
1. En la parte superior de la página, anote el valor de **App ID** (Identificador de la aplicación).
1. Junto a **App Secret** (Secreto de la aplicación), seleccione **Show** (Mostrar) y anote su valor. Usará el identificador de la aplicación y el secreto de la aplicación para configurar Facebook como proveedor de identidades de su inquilino. El **secreto de la aplicación** es una credencial de seguridad importante que debe almacenar de forma segura.
1. Seleccione el signo más situado junto a **PRODUCTS** (PRODUCTOS) y, después, en **Facebook Login** (Inicio de sesión con Facebook), seleccione **Set up** (Configurar).
1. En **Facebook Login** (Inicio de sesión con Facebook) en el menú de la izquierda, seleccione **Settings** (Configuración).
1. En **Valid OAuth redirect URIs** (URI de redireccionamiento OAuth válidos), escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Reemplace `your-tenant-name` por el nombre del inquilino. Seleccione **Save Changes** (Guardar cambios) en la parte inferior de la página.
1. Para que su aplicación de Facebook esté disponible para Azure AD B2C, haga clic en el selector de **estados** situado en la parte superior derecha de la página, **actívelo** para hacer que la aplicación sea pública y, después, haga clic en **Confirmar**. En este momento el estado debería cambiar de **Desarrollo** a **Activo**.

## <a name="add-the-identity-providers"></a>Actualización de los proveedores de identidades

Después de crear la aplicación para el proveedor de identidades que quiere agregar, agregue el proveedor de identidades al inquilino.

### <a name="add-the-azure-active-directory-identity-provider"></a>Incorporación del proveedor de identidades de Azure Active Directory

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Nuevo proveedor de OpenID Connect**.
1. Escriba un **nombre**. Por ejemplo, escriba *Contoso Azure AD*.
1. En **URL de metadatos**, escriba la dirección URL siguiente y sustituya `your-AD-tenant-domain` por el nombre de dominio del inquilino de Azure AD:

    ```
    https://login.microsoftonline.com/your-AD-tenant-domain/.well-known/openid-configuration
    ```

    Por ejemplo, `https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration`.

1. En **Id. de cliente**, escriba el identificador de aplicación que ha anotado anteriormente.
1. En **Secreto de cliente**, escriba el secreto de cliente que ha anotado anteriormente.
1. Deje los valores predeterminados de **Ámbito**, **Tipo de respuesta** y **Modo de respuesta**.
1. (Opcional) Escriba un valor para **Domain_hint**. Por ejemplo, *ContosoAD*. Las [sugerencias de dominio](../active-directory/manage-apps/configure-authentication-for-federated-users-portal.md) son directivas que se incluyen en la solicitud de autenticación de una aplicación. Se pueden usar para enviar el usuario a su página de inicio de sesión del IdP federado. O también las puede usar una aplicación de varios inquilinos para enviar al usuario directamente a la página de inicio de sesión de Azure AD del inquilino.
1. En **Asignación de notificaciones del proveedor de identidades**, escriba los siguientes valores de asignación de notificaciones:

    * **Id. de usuario**: *oid*
    * **Nombre para mostrar**: *name*
    * **Nombre propio**: *given_name*
    * **Apellido**: *family_name*
    * **Correo electrónico**: *unique_name*

1. Seleccione **Guardar**.

### <a name="add-the-facebook-identity-provider"></a>Incorporación del proveedor de identidades de Facebook

1. Seleccione **Proveedores de identidades** y luego **Facebook**.
1. Escriba un **nombre**. Por ejemplo, *Facebook*.
1. En **Id. de cliente**, escriba el identificador de aplicación de Facebook que ha creado anteriormente.
1. En **Secreto de cliente**, escriba el secreto de aplicación que ha anotado.
1. Seleccione **Guardar**.

## <a name="update-the-user-flow"></a>Actualización del flujo de usuario

En el tutorial que completó como parte de los requisitos previos, creó un flujo de usuario para registro e inicio de sesión denominado *B2C_1_signupsignin1*. En esta sección, puede agregar los proveedores de identidades al flujo de usuario *B2C_1_signupsignin1*.

1. Seleccione **Flujos de usuario** y, luego, elija el flujo de usuario *B2C_1_signupsignin1*.
2. Seleccione **Proveedores de identidades**, seleccione los proveedores de identidades de **Facebook** y **Contoso Azure AD** que agregó.
3. Seleccione **Guardar**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. En la página Información general del flujo de usuario que creó, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *webapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione **Ejecutar flujo de usuario** y luego inicie sesión con un proveedor de identidades que haya agregado anteriormente.
1. Repita los pasos del 1 al 3 con los otros proveedores de identidades que agregó.

Si la operación de inicio de sesión se realiza correctamente, se le redirigirá a `https://jwt.ms`, en donde se muestra el token descodificado, similar a lo siguiente:

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "<key-ID>"
}.{
  "exp": 1562346892,
  "nbf": 1562343292,
  "ver": "1.0",
  "iss": "https://your-b2c-tenant.b2clogin.com/10000000-0000-0000-0000-000000000000/v2.0/",
  "sub": "20000000-0000-0000-0000-000000000000",
  "aud": "30000000-0000-0000-0000-000000000000",
  "nonce": "defaultNonce",
  "iat": 1562343292,
  "auth_time": 1562343292,
  "name": "User Name",
  "idp": "facebook.com",
  "postalCode": "12345",
  "tfp": "B2C_1_signupsignin1"
}.[Signature]
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo:

> [!div class="checklist"]
> * Crear las aplicaciones del proveedor de identidades
> * Agregar los proveedores de identidades al inquilino
> * Agregar los proveedores de identidades al flujo del usuario

A continuación, descubra cómo personalizar la interfaz de usuario de las páginas que se muestran a los usuarios como parte de su experiencia de identidad en sus aplicaciones:

> [!div class="nextstepaction"]
> [Personalización de la interfaz de usuario de las aplicaciones en Azure Active Directory B2C](tutorial-customize-ui.md)
