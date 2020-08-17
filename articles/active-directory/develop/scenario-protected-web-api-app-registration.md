---
title: Registro de aplicaciones de API web protegida | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo crear una API web protegida y la información que necesita para registrar la aplicación.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: c1fab15cade2ce23e053bc73028e6420692c3d8a
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86518281"
---
# <a name="protected-web-api-app-registration"></a>API web protegida: Registro de aplicación

En este artículo se explican los detalles específicos de registro de la aplicación para una API web protegida.

Para conocer los pasos comunes para registrar una aplicación, consulte [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](quickstart-register-app.md).

## <a name="accepted-token-version"></a>Versión del token aceptada

El punto de conexión de la plataforma de identidad de Microsoft puede emitir tokens v1.0 y v2.0. Para obtener más información sobre estos tokens, consulte [Tokens de acceso](access-tokens.md).

La versión del token que puede aceptar la API depende de los **tipos de cuenta admitidos** que haya seleccionado al crear el registro de aplicación de API web en Azure Portal.

- Si el valor de **tipos de cuentas admitidos** es **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype, Xbox, Outlook.com)** , la versión del token será v2.0.
- En caso contrario, la versión del token aceptada puede ser la v1.0.

Después de crear la aplicación, puede determinar o cambiar la versión del token aceptada siguiendo estos pasos:

1. En Azure Portal, seleccione la aplicación y, luego, seleccione **Manifiesto**.
1. Busque la propiedad **accessTokenAcceptedVersion** en el manifiesto.
1. El valor especifica a Azure Active Directory (Azure AD) qué versión del token acepta la API web.
    - Si el valor es 2, la API web acepta tokens v 2.0.
    - Si el valor es **NULL**, la API web acepta tokens v1.0.
1. Si ha cambiado la versión del token, seleccione **Guardar**.

> [!NOTE]
> La API web especifica qué versión del token acepta. Cuando un cliente solicita un token para la API web desde el punto de conexión de la plataforma de identidad de Microsoft (v2.0), el cliente obtiene un token que indica qué versión del token acepta la API web.

## <a name="no-redirect-uri"></a>URI sin redireccionamiento

Las API web no necesitan registrar ningún URI de redirección porque no hay ningún usuario con la sesión iniciada de forma interactiva.

## <a name="exposed-api"></a>API expuesta

Otra configuración específica de las API web es la API expuesta y los ámbitos o roles de aplicación expuestos.

### <a name="application-id-uri-and-scopes"></a>URI y ámbitos del identificador de aplicación

Normalmente, los ámbitos tienen el formato `resourceURI/scopeName`. En Microsoft Graph, los ámbitos tienen accesos directos. Por ejemplo, `User.Read` es un acceso directo de `https://graph.microsoft.com/user.read`.

Durante el registro de la aplicación, debe definir estos parámetros:

- URI del recurso
- Uno o varios ámbitos
- Uno o varios roles de aplicación

De forma predeterminada, el portal de registro de aplicación recomienda usar el URI de recurso `api://{clientId}`. Este URI es único pero no es legible para los humanos. Si cambia el URI, asegúrese de que el nuevo valor sea único. El portal de registro de aplicaciones se asegurará de que use un [dominio de editor configurado](howto-configure-publisher-domain.md)

Para las aplicaciones cliente, los ámbitos se muestran como *permisos delegados* y los roles de aplicación se muestran como *permisos de aplicación* para la API web.

También aparecen ámbitos en la ventana de consentimiento que se presenta a los usuarios de la aplicación. Por lo tanto, deberá proporcionar las cadenas correspondientes que describen el ámbito:

- Como las ve un usuario.
- Como las ve un administrador de inquilinos, que puede otorgar consentimiento de administrador.

Un usuario no puede dar su consentimiento a los roles de aplicación (ya que los usa una aplicación que llama a la API web en su propio nombre). Un administrador de inquilinos tendrá que dar su consentimiento a las aplicaciones cliente de la API web que exponen roles de aplicación. Consulte el [consentimiento del administrador](v2-admin-consent.md) para obtener más información

### <a name="exposing-delegated-permissions-scopes"></a>Exponer permisos delegados (ámbitos)

1. Seleccione **Exponer una API** en el registro de la aplicación.
1. Seleccione **Agregar un ámbito**.
1. Si se le pregunta, seleccione **Guardar y continuar** para aceptar el URI de identificador de aplicación propuesto (`api://{clientId}`).
1. Especifique estos valores:
    - Seleccione **Nombre de ámbito** y escriba **access_as_user**.
    - Seleccione **¿Quién puede dar el consentimiento?** y asegúrese de que esté seleccionado **Administradores y usuarios**.
    - En **Nombre para mostrar del consentimiento del administrador**, escriba **Access TodoListService as a user** (Acceder a TodoListService como usuario).
    - Seleccione **Descripción del consentimiento del administrador** y escriba **Accesses the TodoListService Web API as a user** (Accede a la API web de TodoListService como usuario).
    - Seleccione **Nombre para mostrar del consentimiento del usuario**  y escriba **Access TodoListService as a user** (Acceder a TodoListService como usuario).
    - En **Descripción del consentimiento del usuario**, escriba **Accesses the TodoListService Web API as a user** (Accede a la API web de TodoListService como usuario).
    - Mantenga el valor de **Estado** establecido en **Habilitado**.
 1. Seleccione la opción **Agregar un ámbito**.

### <a name="if-your-web-api-is-called-by-a-daemon-app"></a>Si una aplicación de demonio llama a la API web

En esta sección, aprenderá a registrar su API web protegida para que las aplicaciones de demonio puedan invocarla de forma segura.

- Solo se declaran y exponen *permisos de aplicación* ya que las aplicaciones de demonio no interactúan con los usuarios. Los permisos delegados no tendrían sentido.
- Los administradores de inquilinos pueden exigir que Azure AD emita tokens de API web solo para las aplicaciones que se han registrado para acceder a uno de los permisos de aplicación de la API.

#### <a name="exposing-application-permissions-app-roles"></a>Exposición de permisos de aplicación (roles de aplicación)

Para exponer permisos de aplicación, debe editar el manifiesto.

1. En el registro de aplicación para la aplicación, seleccione **Manifiesto**.
1. Para editar el manifiesto, busque el valor `appRoles` y agregue roles de aplicación. En el siguiente bloque JSON de ejemplo se proporcionan las definiciones del rol.
1. Deje `allowedMemberTypes` establecido solo en `"Application"`.
1. Asegúrese de que `id` sea un GUID único.
1. Asegúrese de que `displayName` y `value` no contengan espacios.
1. Guarde el manifiesto.

En el ejemplo siguiente se muestra el contenido de `appRoles`, donde el valor de `id` puede ser cualquier GUID único.

```json
"appRoles": [
    {
    "allowedMemberTypes": [ "Application" ],
    "description": "Accesses the TodoListService-Cert as an application.",
    "displayName": "access_as_application",
    "id": "ccf784a6-fd0c-45f2-9c08-2f9d162a0628",
    "isEnabled": true,
    "lang": null,
    "origin": "Application",
    "value": "access_as_application"
    }
],
```

#### <a name="ensuring-that-azure-ad-issues-tokens-for-your-web-api-to-only-allowed-clients"></a>Asegurarse de que Azure AD emite tokens para la API web solo para los clientes permitidos

La API web comprueba el rol de aplicación. Este rol es una manera de que un desarrollador de software exponga permisos de aplicación. También puede configurar Azure AD para que emita tokens de API solo a las aplicaciones que el administrador de inquilinos apruebe para el acceso a API.

Para agregar esta seguridad mejorada:

1. Vaya a la página **Información general** del registro de aplicaciones.
1. En **Aplicación administrada en directorio local**, seleccione el vínculo con el nombre de la aplicación. Puede que la etiqueta de esta selección esté truncada. Por ejemplo, puede que vea **Aplicación administrada en...**

   > [!NOTE]
   >
   > Al seleccionar este vínculo, irá a la página de **información general sobre aplicaciones empresariales**. Esta página está asociada a la entidad de servicio de la aplicación del inquilino en el que la creó. Para volver a la página de registro de aplicaciones, use el botón Atrás del explorador.

1. Seleccione la página **Propiedades** de la sección **Administrar** de las páginas de la aplicación empresarial.
1. Para que Azure AD permita el acceso a la API web solo desde determinados clientes, establezca **¿Asignación de usuarios?** en **Sí**.

   > [!IMPORTANT]
   >
   > Si establece **¿Asignación de usuarios?** en **Sí**, Azure AD comprueba las asignaciones de roles de aplicación de un cliente cuando solicita un token de acceso a la API web. Si el cliente no tiene asignado ningún rol de aplicación, Azure AD devolverá el mensaje de error "invalid_client: AADSTS501051: A la aplicación \<application name\> no se le asigna un rol para la \<web API\>".
   >
   > Si mantiene **¿Asignación de usuarios?** establecido en **No**, Azure AD no comprobará las asignaciones de rol de la aplicación cuando un cliente solicite un token de acceso para la API web. Cualquier cliente de demonio (es decir, cualquier cliente que use el flujo de credenciales de cliente) puede obtener un token de acceso para la API con solo especificar su público. Cualquier aplicación puede acceder a la API sin tener que solicitar permiso para ello.
   >
   > Sin embargo, como se ha explicado en la sección anterior, la API web siempre puede comprobar que la aplicación tenga el rol correcto, que esté autorizado por el administrador de inquilinos. Para realizar esta verificación, la API valida que el token de acceso tiene una notificación de roles con un valor correcto. En el ejemplo JSON anterior, el valor es `access_as_application`.

1. Seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración del código de la aplicación](scenario-protected-web-api-app-configuration.md)
