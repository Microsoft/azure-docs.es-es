---
title: Uso avanzado de AuthN/AuthZ
description: Aprenda a personalizar la característica de autenticación y autorización en App Service para diferentes escenarios y obtener las notificaciones de usuario y los diferentes tokens.
ms.topic: article
ms.date: 07/08/2020
ms.custom: seodec18
ms.openlocfilehash: 2fa2e3463e057062ba743c2f6989aa571c85c983
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962475"
---
# <a name="advanced-usage-of-authentication-and-authorization-in-azure-app-service"></a>Uso avanzado de la autenticación y autorización en Azure App Service

En este artículo se muestra cómo personalizar la [autenticación y autorización integradas en App Service](overview-authentication-authorization.md) y cómo administrar la identidad desde la aplicación. 

Para comenzar inmediatamente, consulte uno de los siguientes tutoriales:

* [Tutorial: Autenticación y autorización de usuarios de extremo a extremo en Azure App Service](tutorial-auth-aad.md)
* [Configuración de la aplicación para usar el inicio de sesión de Azure Active Directory](configure-authentication-provider-aad.md)
* [Configuración de la aplicación para usar el inicio de sesión de Facebook](configure-authentication-provider-facebook.md)
* [Configuración de la aplicación para usar el inicio de sesión de Google](configure-authentication-provider-google.md)
* [Configuración de la aplicación para usar el inicio de sesión de la cuenta Microsoft](configure-authentication-provider-microsoft.md)
* [Configuración de la aplicación para usar el inicio de sesión de Twitter](configure-authentication-provider-twitter.md)
* [Configuración de la aplicación para iniciar sesión mediante un proveedor de OpenID Connect (versión preliminar)](configure-authentication-provider-openid-connect.md)

## <a name="use-multiple-sign-in-providers"></a>Uso de varios proveedores de inicio de sesión

La configuración del portal no ofrece una manera preparada para presentar varios proveedores de inicio de sesión a los usuarios (por ejemplo, Facebook y Twitter). Sin embargo, no es difícil agregar la funcionalidad a la aplicación. A continuación se describen los pasos necesarios:

Primero, en la página **Autenticación / Autorización** de Azure Portal, configure cada uno de los proveedores de identidades que desea habilitar.

En **Action to take when request is not authenticated** (Acción necesaria cuando la solicitud no está autenticada), seleccione **Allow Anonymous requests (no action)** (Permitir solicitudes anónimas (sin acción)).

En la página de inicio de sesión, en la barra de navegación o en cualquier otra ubicación de la aplicación, agregue un vínculo de inicio de sesión a cada uno de los proveedores que ha habilitado (`/.auth/login/<provider>`). Por ejemplo:

```html
<a href="/.auth/login/aad">Log in with Azure AD</a>
<a href="/.auth/login/microsoftaccount">Log in with Microsoft Account</a>
<a href="/.auth/login/facebook">Log in with Facebook</a>
<a href="/.auth/login/google">Log in with Google</a>
<a href="/.auth/login/twitter">Log in with Twitter</a>
```

Cuando el usuario haga clic en uno de los vínculos, la página de inicio de sesión correspondiente se abrirá para que el usuario inicie sesión.

Para redirigir al usuario después del inicio de sesión a una dirección URL personalizada, use el parámetro de cadena de consulta `post_login_redirect_url` (que no debe confundirse con el URI de redireccionamiento den su configuración de proveedor de identidades). Por ejemplo, para redirigir al usuario a `/Home/Index` después de iniciar sesión, utilice el siguiente código HTML:

```html
<a href="/.auth/login/<provider>?post_login_redirect_url=/Home/Index">Log in</a>
```

## <a name="validate-tokens-from-providers"></a>Validar los tokens de los proveedores

En un inicio de sesión que dirige el cliente, la aplicación consigue de forma manual que el cliente inicie sesión en el proveedor y envía el token de autenticación a App Service para su validación (consulte el [flujo de autenticación](overview-authentication-authorization.md#authentication-flow)). Esta validación en sí misma no le otorga acceso a los recursos de aplicación que quiere, pero una validación exitosa le dará un token de sesión que puede usar para obtener acceso a los recursos de la aplicación. 

Para validar el token del proveedor, la aplicación App Service debe configurarse primero con el proveedor que quiera usar. En el entorno de ejecución, después de recuperar el token de autenticación de su proveedor, publique el token en `/.auth/login/<provider>` para su validación. Por ejemplo: 

```
POST https://<appname>.azurewebsites.net/.auth/login/aad HTTP/1.1
Content-Type: application/json

{"id_token":"<token>","access_token":"<token>"}
```

El formato del token varía ligeramente según el proveedor. Consulte la tabla siguiente para más detalles:

| Valor del proveedor | Necesario en el cuerpo de la solicitud | Comentarios |
|-|-|-|
| `aad` | `{"access_token":"<access_token>"}` | |
| `microsoftaccount` | `{"access_token":"<token>"}` | La propiedad `expires_in` es opcional. <br/>Al solicitar el token de los servicios Live, solicite siempre el ámbito `wl.basic`. |
| `google` | `{"id_token":"<id_token>"}` | La propiedad `authorization_code` es opcional. Cuando se especifica, también puede ir acompañado opcionalmente por la propiedad `redirect_uri`. |
| `facebook`| `{"access_token":"<user_access_token>"}` | Use un [token de acceso de usuario](https://developers.facebook.com/docs/facebook-login/access-tokens) válido de Facebook. |
| `twitter` | `{"access_token":"<access_token>", "access_token_secret":"<acces_token_secret>"}` | |
| | | |

Si el token del proveedor se valida correctamente, la API regresa con un elemento `authenticationToken` en el cuerpo de la respuesta, que es su token de sesión. 

```json
{
    "authenticationToken": "...",
    "user": {
        "userId": "sid:..."
    }
}
```

Una vez que tenga este token de sesión, puede obtener acceso a los recursos de aplicación protegidos agregando el encabezado `X-ZUMO-AUTH` a sus solicitudes HTTP. Por ejemplo: 

```
GET https://<appname>.azurewebsites.net/api/products/1
X-ZUMO-AUTH: <authenticationToken_value>
```

## <a name="sign-out-of-a-session"></a>Cierre de sesión de una sesión

Los usuarios pueden iniciar un cierre de sesión mediante el envío de una solicitud `GET` al punto de conexión de la aplicación `/.auth/logout`. La solicitud `GET` hace lo siguiente:

- Borra las cookies de autenticación de la sesión actual.
- Elimina los tokens del usuario actual del almacén de tokens.
- Para Azure Active Directory y Google, realiza un cierre de sesión del servidor en el proveedor de identidades.

A continuación, se muestra un vínculo de cierre de sesión simple en una página web:

```html
<a href="/.auth/logout">Sign out</a>
```

De forma predeterminada, un cierre de sesión correcto redirige el cliente a la dirección URL `/.auth/logout/done`. Puede cambiar la página de redirección del inicio de sesión posterior agregando el parámetro de consulta `post_logout_redirect_uri`. Por ejemplo:

```
GET /.auth/logout?post_logout_redirect_uri=/index.html
```

Se recomienda [codificar](https://wikipedia.org/wiki/Percent-encoding) el valor de `post_logout_redirect_uri`.

Al utilizar direcciones URL completas, la dirección URL debe hospedarse en el mismo dominio o configurarse como una dirección URL de redirección externa permitida para la aplicación. En el ejemplo siguiente, para redirigir a `https://myexternalurl.com` que no se hospedado en el mismo dominio:

```
GET /.auth/logout?post_logout_redirect_uri=https%3A%2F%2Fmyexternalurl.com
```

Ejecute el siguiente comando en [Azure Cloud Shell](../cloud-shell/quickstart.md):

```azurecli-interactive
az webapp auth update --name <app_name> --resource-group <group_name> --allowed-external-redirect-urls "https://myexternalurl.com"
```

## <a name="preserve-url-fragments"></a>Conservación de los fragmentos de dirección URL

Después de que los usuarios inician sesión en la aplicación, normalmente desean redirigirse a la misma sección de la misma página, como `/wiki/Main_Page#SectionZ`. Sin embargo, dado que los [fragmentos de dirección URL](https://wikipedia.org/wiki/Fragment_identifier) (por ejemplo, `#SectionZ`) nunca se envían al servidor, no se conservan de forma predeterminada después de que el inicio de sesión de OAuth se complete y se redirija a la aplicación. Los usuarios obtienen una experiencia poco óptima cuando necesitan volver a navegar al delimitador deseado. Esta limitación se aplica a todas las soluciones de autenticación del servidor.

En la autenticación de App Service, puede conservar los fragmentos de dirección URL en el inicio de sesión de OAuth. Para ello, establezca un valor de aplicación denominado `WEBSITE_AUTH_PRESERVE_URL_FRAGMENT` en `true`. Puede hacerlo el [Azure Portal](https://portal.azure.com), o simplemente ejecute el siguiente comando en [Azure Cloud Shell](../cloud-shell/quickstart.md):

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group <group_name> --settings WEBSITE_AUTH_PRESERVE_URL_FRAGMENT="true"
```

## <a name="access-user-claims"></a>Acceso a las notificaciones de usuario

App Service pasa las notificaciones de usuario a la aplicación mediante encabezados especiales. Las solicitudes externas no están autorizadas para establecer estos encabezados, por lo que solo están presentes si App Service los establece. A continuación puede ver algunos encabezados de ejemplo:

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID

El código escrito en cualquier lenguaje o plataforma puede obtener la información que necesita de estos encabezados. Para las aplicaciones de ASP.NET 4.6, **ClaimsPrincipal** se establece automáticamente con los valores adecuados. Sin embargo, ASP.NET Core no proporciona un middleware de autenticación que se integre con las notificaciones de usuario de App Service. Para obtener una solución alternativa, vea [MaximeRouiller.Azure.AppService.EasyAuth](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth).

Si el [almacén de tokens](overview-authentication-authorization.md#token-store) está habilitado para la aplicación, también puede obtener detalles adicionales sobre el usuario autenticado mediante una llamada a `/.auth/me`. Los SDK del servidor de Mobile Apps proporcionan métodos de asistente para trabajar con estos datos. Para más información, consulte [Uso del SDK de Node.js de Azure Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk#howto-tables-getidentity) y [Trabajar con el SDK del servidor back-end de .NET para Azure Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk#user-info).

## <a name="retrieve-tokens-in-app-code"></a>Recuperación de los tokens en el código de aplicación

Desde el código de servidor, los tokens específicos del proveedor se inyectan en el encabezado de solicitud, por lo que puede acceder a ellos fácilmente. En la siguiente tabla se muestran los posibles nombres de encabezado de token:

| Proveedor | Nombres de encabezados |
|-|-|
| Azure Active Directory | `X-MS-TOKEN-AAD-ID-TOKEN` <br/> `X-MS-TOKEN-AAD-ACCESS-TOKEN` <br/> `X-MS-TOKEN-AAD-EXPIRES-ON`  <br/> `X-MS-TOKEN-AAD-REFRESH-TOKEN` |
| Token de Facebook | `X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN` <br/> `X-MS-TOKEN-FACEBOOK-EXPIRES-ON` |
| Google | `X-MS-TOKEN-GOOGLE-ID-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-ACCESS-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-EXPIRES-ON` <br/> `X-MS-TOKEN-GOOGLE-REFRESH-TOKEN` |
| Cuenta Microsoft | `X-MS-TOKEN-MICROSOFTACCOUNT-ACCESS-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-EXPIRES-ON` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-AUTHENTICATION-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-REFRESH-TOKEN` |
| Twitter | `X-MS-TOKEN-TWITTER-ACCESS-TOKEN` <br/> `X-MS-TOKEN-TWITTER-ACCESS-TOKEN-SECRET` |
|||

Desde el código de cliente (por ejemplo, una aplicación móvil o JavaScript en el explorador), envíe la solicitud `GET` HTTP a `/.auth/me` (el [almacén de tokens](overview-authentication-authorization.md#token-store) debe estar habilitado). El JSON devuelto tiene los tokens específicos del proveedor.

> [!NOTE]
> Los tokens de acceso son para acceder a recursos de proveedor, por lo que solo están presentes si se configura el proveedor con un secreto de cliente. Para ver cómo obtener tokens de actualización, consulte Tokens de acceso de actualización.

## <a name="refresh-identity-provider-tokens"></a>Actualización de los tokens de proveedor de identidad

Cuando el token de acceso de su proveedor (no el [token de sesión](#extend-session-token-expiration-grace-period)) expire, debe volver a autenticar al usuario antes de volver a usar ese token. Puede evitar la expiración del token mediante la realización de una llamada `GET` al punto de conexión `/.auth/refresh` de la aplicación. Cuando se llama, App Service actualiza automáticamente los tokens de acceso en el [almacén de tokens](overview-authentication-authorization.md#token-store) para el usuario autenticado. Las solicitudes posteriores para los tokens por código de aplicación obtienen los tokens actualizados. Sin embargo, para que la actualización de token funcione, el almacén de tokens debe contener [tokens de actualización](https://auth0.com/learn/refresh-tokens/) para el proveedor. Cada proveedor documenta la manera de obtener tokens de actualización, pero en la lista siguiente se muestra un breve resumen:

- **Google**: anexe un parámetro de cadena de consulta `access_type=offline` en su llamada API `/.auth/login/google`. Si usa el SDK de Mobile Apps, puede agregar el parámetro a una de las sobrecargas `LogicAsync` (vea [Google Refresh Tokens](https://developers.google.com/identity/protocols/OpenIDConnect#refresh-tokens) (Tokens de actualización de Google)).
- **Facebook**: no proporciona tokens de actualización. Los tokens de larga duración expiran en 60 días (vea [Facebook Expiration and Extension of Access Tokens](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension) (Expiración y extensión de tokens de acceso de Facebook)).
- **Twitter**: los tokens de acceso no expiran [vea [Twitter OAuth FAQ](https://developer.twitter.com/en/docs/basics/authentication/FAQ) (Preguntas más frecuentes sobre Twitter OAuth)].
- **Cuenta Microsoft**: cuando [defina la configuración de autenticación de Cuenta Microsoft](configure-authentication-provider-microsoft.md), seleccione el ámbito `wl.offline_access`.
- **Azure Active Directory**: en [https://resources.azure.com](https://resources.azure.com), siga estos pasos:
    1. En la parte superior de la página, seleccione **Lectura y escritura**.
    2. En el explorador de la izquierda, vaya a **subscriptions** > ** _\<subscription\_name_** > **resourceGroups** > **_ \<resource\_group\_name> _** > **providers** > **Microsoft.Web** > **sites** > **_ \<app\_name>_** > **config** > **authsettings**. 
    3. Haga clic en **Editar**.
    4. Modifique la siguiente propiedad. Reemplace _\<app\_id>_ por el identificador de aplicación de Azure Active Directory del servicio al que desea acceder.

        ```json
        "additionalLoginParams": ["response_type=code id_token", "resource=<app_id>"]
        ```

    5. Haga clic en **Put**. 

Una vez configurado el proveedor, puede [buscar el token de actualización y el tiempo de expiración para el token de acceso](#retrieve-tokens-in-app-code) en el almacén de tokens. 

Para actualizar el token de acceso en cualquier momento, simplemente llame a `/.auth/refresh` en cualquier lenguaje. El fragmento de código siguiente utiliza jQuery para actualizar los tokens de acceso de un cliente de JavaScript.

```javascript
function refreshTokens() {
  let refreshUrl = "/.auth/refresh";
  $.ajax(refreshUrl) .done(function() {
    console.log("Token refresh completed successfully.");
  }) .fail(function() {
    console.log("Token refresh failed. See application logs for details.");
  });
}
```

Si un usuario revoca los permisos concedidos a la aplicación, la llamada a `/.auth/me` puede producir un error con un respuesta `403 Forbidden`. Para diagnosticar errores, compruebe los registros de aplicación para más detalles.

## <a name="extend-session-token-expiration-grace-period"></a>Ampliación del período de gracia de expiración del token de sesión

La sesión autenticada expira después de 8 horas. Después de que expire una sesión autenticada, hay un período de gracia de 72 horas de forma predeterminada. Dentro de este período de gracia, puede actualizar el token de sesión con App Service sin volver a autenticar al usuario. Simplemente puede llamar a `/.auth/refresh` cuando el token de sesión deje de ser válido y no es necesario que realice un seguimiento usted mismo de la expiración del token. Una vez que transcurra el período de gracia de 72 horas, el usuario debe iniciar sesión de nuevo para obtener un token de sesión válido.

Si 72 horas no es tiempo suficiente para usted, puede ampliar este período de expiración. Extender la expiración durante un período prolongado podría tener implicaciones de seguridad importantes (por ejemplo, cuando un token de autenticación se pierde o roba). Por tanto, debe dejarla en el valor predeterminado de 72 horas o establecer el período de extensión en el valor más pequeño.

Para extender el período de expiración predeterminado, ejecute el siguiente comando en [Cloud Shell](../cloud-shell/overview.md).

```azurecli-interactive
az webapp auth update --resource-group <group_name> --name <app_name> --token-refresh-extension-hours <hours>
```

> [!NOTE]
> El período de gracia solo se aplica a la sesión autenticada de App Service, no a los tokens de los proveedores de identidades. No hay ningún período de gracia para los tokens del proveedor expirados. 
>

## <a name="limit-the-domain-of-sign-in-accounts"></a>Limitación del dominio de cuentas de inicio de sesión

Cuenta Microsoft y Azure Active Directory le permiten iniciar sesión desde varios dominios. Por ejemplo, Cuenta Microsoft permite cuentas de _outlook.com_, _live.com_ y _hotmail.com_. Azure AD permite cualquier número de dominios personalizados para las cuentas de inicio de sesión. Sin embargo, puede que quiera acelerar el proceso para que los usuarios accedan directamente a su propia página de inicio de sesión personalizada de Azure AD (por ejemplo, `contoso.com`). Para sugerir el nombre de dominio de las cuentas de inicio de sesión, siga estos pasos.

En [https://resources.azure.com](https://resources.azure.com), vaya a **subscriptions** > ** _\<subscription\_name_** > **resourceGroups** > **_ \<resource\_group\_name> _** > **providers** > **Microsoft.Web** > **sites** > **_ \<app\_name> _** > **config** > **authsettings**. 

Haga clic en **Editar**, modifique la propiedad siguiente y luego haga clic en **Put**. Asegúrese de reemplazar _\<domain\_name>_ por el dominio que desee.

```json
"additionalLoginParams": ["domain_hint=<domain_name>"]
```

Esta configuración anexa el parámetro de cadena de consulta `domain_hint` a la dirección URL de redireccionamiento de inicio de sesión. 

> [!IMPORTANT]
> Es posible que el cliente quite el parámetro `domain_hint` después de recibir la dirección URL de redireccionamiento y, a continuación, inicie sesión con otro dominio. Por lo tanto, aunque esta función es cómoda, no es una característica de seguridad.
>

## <a name="authorize-or-deny-users"></a>Autorización o denegación de usuarios

Si bien App Service se encarga del caso de autorización más sencillo (es decir, rechazar las solicitudes no autenticadas), es posible que la aplicación requiera un comportamiento de autorización más específico, como limitar el acceso a un solo grupo específico de usuarios. En algunos casos, debe escribir código de aplicación personalizado para permitir o denegar el acceso al usuario que inició sesión. En otros casos, App Service o el proveedor de identidades pueden ayudar sin necesidad de realizar cambios en el código.

- [Nivel de servidor](#server-level-windows-apps-only)
- [Nivel de proveedor de identidades](#identity-provider-level)
- [Nivel de aplicación](#application-level)

### <a name="server-level-windows-apps-only"></a>Nivel de servidor (solo aplicaciones de Windows)

En cualquier aplicación de Windows, puede definir el comportamiento de la autorización del servidor web de IIS editando el archivo *Web.config*. Las aplicaciones de Linux no usan IIS y no se pueden configurar a través de *Web. config*.

1. Vaya a `https://<app-name>.scm.azurewebsites.net/DebugConsole`.

1. En el explorador de archivos de App Service, vaya a *site/wwwroot*. Si no existe un archivo *Web.config*, créelo seleccionando **+**  > **Nuevo archivo**. 

1. Seleccione el lápiz de *Web. config* para editarlo. Agregue el código de configuración siguiente y haga clic en **Guardar**. Si ya existe *Web.config*, simplemente agregue el elemento `<authorization>` con todo lo que contiene. Agregue las cuentas que desea permitir al elemento `<allow>`.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
       <system.web>
          <authorization>
            <allow users="user1@contoso.com,user2@contoso.com"/>
            <deny users="*"/>
          </authorization>
       </system.web>
    </configuration>
    ```

### <a name="identity-provider-level"></a>Nivel de proveedor de identidades

El proveedor de identidades puede proporcionar cierta autorización llave en mano. Por ejemplo:

- Para [Azure App Service](configure-authentication-provider-aad.md), puede [administrar el acceso de nivel empresarial](../active-directory/manage-apps/what-is-access-management.md) directamente en Azure AD. Para obtener instrucciones, consulte [Cómo quitar el acceso de un usuario a una aplicación](../active-directory/manage-apps/methods-for-removing-user-access.md).
- En el caso de [Google](configure-authentication-provider-google.md), los proyectos de la API de Google que pertenecen a una [organización](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#organizations) se pueden configurar para permitir el acceso solo a los usuarios de su organización (consulte la [página de soporte técnico de **configuración de OAuth 2.0** de Google](https://support.google.com/cloud/answer/6158849?hl=en)).

### <a name="application-level"></a>Nivel de aplicación

Si alguno de los otros niveles no proporciona la autorización que necesita, o si no se admite la plataforma o el proveedor de identidades, debe escribir código personalizado para autorizar a los usuarios en función de las [notificaciones de usuario](#access-user-claims).

## <a name="configure-using-a-file-preview"></a><a name="config-file"> </a> Configuración con un archivo (versión preliminar)

La configuración de autenticación se puede configurar mediante un archivo que proporciona la implementación. Puede que sea necesario para ciertas funcionalidades que se encuentren en versión preliminar de la autenticación y autorización de App Service.

> [!IMPORTANT]
> Recuerde que la carga de la aplicación y, por lo tanto, este archivo, pueden moverse entre entornos, al igual que sucede con los [espacios](./deploy-staging-slots.md). Es probable que quiera anclar un registro de aplicación diferente a cada espacio, por lo que, en estos casos, debería seguir usando el método de configuración estándar en lugar de usar el archivo de configuración.

### <a name="enabling-file-based-configuration"></a>Habilitación de la configuración basada en archivos

> [!CAUTION]
> Durante la versión preliminar, al habilitar la configuración basada en archivos se deshabilitará la característica de administración de la autenticación y autorización de App Service de la aplicación a través de algunos clientes, como Azure Portal, la CLI de Azure y Azure PowerShell.

1. Cree un nuevo archivo JSON para la configuración en la raíz del proyecto (implementado en D:\home\site\wwwroot en la aplicación web o de funciones). Rellene la configuración deseada según la [referencia de configuración basada en archivos](#configuration-file-reference). Si modifica una configuración de Azure Resource Manager existente, asegúrese de traducir las propiedades capturadas en la colección de `authsettings` en el archivo de configuración.

2. Modifique la configuración existente, que se captura en las API de [Azure Resource Manager](../azure-resource-manager/management/overview.md) de `Microsoft.Web/sites/<siteName>/config/authsettings`. Para modificarla, puede usar una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) o una herramienta como [Azure Resource Explorer](https://resources.azure.com/). En la colección authsettings, tendrá que establecer tres propiedades (y puede quitar otras):

    1.  Establecer en `enabled` en "true"
    2.  Establecer en `isAuthFromFile` en "true"
    3.  Establecer `authFilePath` en el nombre del archivo (por ejemplo, "auth.json")

> [!NOTE]
> El formato de `authFilePath` varía entre las plataformas. En Windows, se admiten las rutas de acceso relativas y absolutas. Se recomienda la relativa. En el caso de Linux, actualmente solo se admiten rutas de acceso absolutas, por lo que el valor de configuración debe ser "/home/site/wwwroot/auth.json" o similar.

Una vez que haya realizado esta actualización de la configuración, el contenido del archivo se usará para definir el comportamiento de autenticación o autorización de App Service para ese sitio. Si alguna vez quiere volver a la configuración de Azure Resource Manager, puede hacerlo volviendo a establecer el elemento `isAuthFromFile` en "false".

### <a name="configuration-file-reference"></a>Referencia del archivo de configuración

Los secretos a los que se hará referencia desde el archivo de configuración deben almacenarse como [configuración de la aplicación](./configure-common.md#configure-app-settings). Puede asignar el nombre que quiera a la configuración. Simplemente tiene que asegurarse de que las referencias del archivo de configuración usan las mismas claves.

El código siguiente agota las posibles opciones de configuración del archivo:

```json
{
    "platform": {
        "enabled": <true|false>
    },
    "globalValidation": {
        "requireAuthentication": <true|false>,
        "unauthenticatedClientAction": "RedirectToLoginPage|AllowAnonymous|Return401|Return403",
        "redirectToProvider": "<default provider alias>",
        "excludedPaths": [
            "/path1",
            "/path2"
        ]
    },
    "identityProviders": {
        "azureActiveDirectory": {
            "enabled": <true|false>,
            "registration": {
                "openIdIssuer": "<issuer url>",
                "clientId": "<app id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_AAD_SECRET",
            },
            "login": {
                "loginParameters": [
                    "paramName1=value1",
                    "paramName2=value2"
                ]
            },
            "validation": {
                "allowedAudiences": [
                    "audience1",
                    "audience2"
                ]
            }
        },
        "facebook": {
            "enabled": <true|false>,
            "registration": {
                "appId": "<app id>",
                "appSecretSettingName": "APP_SETTING_CONTAINING_FACEBOOK_SECRET"
            },
            "graphApiVersion": "v3.3",
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            },
        },
        "gitHub": {
            "enabled": <true|false>,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_GITHUB_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            }
        },
        "google": {
            "enabled": true,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_GOOGLE_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            },
            "validation": {
                "allowedAudiences": [
                    "audience1",
                    "audience2"
                ]
            }
        },
        "twitter": {
            "enabled": <true|false>,
            "registration": {
                "consumerKey": "<consumer key>",
                "consumerSecretSettingName": "APP_SETTING_CONTAINING TWITTER_CONSUMER_SECRET"
            }
        },
        "openIdConnectProviders": {
            "provider name": {
                "enabled": <true|false>,
                "registration": {
                    "clientId": "<client id>",
                    "clientCredential": {
                        "secretSettingName": "<name of app setting containing client secret>"
                    },
                    "openIdConnectConfiguration": {
                        "authorizationEndpoint": "<url specifying authorization endpoint>",
                        "tokenEndpoint": "<url specifying token endpoint>",
                        "issuer": "<url specifying issuer>",
                        "certificationUri": "<url specifying jwks endpoint>",
                        "wellKnownOpenIdConfiguration": "<url specifying .well-known/open-id-configuration endpoint - if this property is set, the other properties of this object are ignored, and authorizationEndpoint, tokenEndpoint, issuer, and certificationUri are set to the corresponding values listed at this endpoint>"
                    }
                },
                "login": {
                    "nameClaimType": "<name of claim containing name>",
                    "scope": [
                        "openid",
                        "profile",
                        "email"
                    ],
                    "loginParameterNames": [
                        "paramName1=value1",
                        "paramName2=value2"
                    ],
                }
            },
            //...
        },
        "login": {
            "routes": {
                "logoutEndpoint": "<logout endpoint>"
            },
            "tokenStore": {
                "enabled": <true|false>,
                "tokenRefreshExtensionHours": "<double>",
                "fileSystem": {
                    "directory": "<directory to store the tokens in if using a file system token store (default)>"
                },
                "azureBlobStorage": {
                    "sasUrlSettingName": "<app setting name containing the sas url for the Azure Blob Storage if opting to use that for a token store>"
                }
            },
            "preserveUrlFragmentsForLogins": <true|false>,
            "allowedExternalRedirectUrls": [
                "https://uri1.azurewebsites.net/",
                "https://uri2.azurewebsites.net/"
            ],
            "cookieExpiration": {
                "convention": "FixedTime|IdentityProviderDerived",
                "timeToExpiration": "<timespan>"
            },
            "nonce": {
                "validateNonce": <true|false>,
                "nonceExpirationInterval": "<timespan>"
            }
        },
        "httpSettings": {
            "requireHttps": <true|false>,
            "routes": {
                "apiPrefix": "<api prefix>"
            },
            "forwardProxy": {
                "convention": "NoProxy|Standard|Custom",
                "customHostHeaderName": "<host header value>",
                "customProtoHeaderName": "<proto header value>"
            }
        }
    }
}
```

## <a name="pin-your-app-to-a-specific-authentication-runtime-version"></a>Anclaje de la aplicación a una versión específica en tiempo de ejecución de autenticación

Al habilitar la autenticación y la autorización, el middleware de la plataforma se inserta en la canalización de solicitudes HTTP, tal y como se describe en la [información general sobre las características](overview-authentication-authorization.md#how-it-works). Este middleware de la plataforma se actualiza periódicamente con nuevas características y mejoras, como parte de las actualizaciones habituales de la plataforma. La aplicación web o de funciones se ejecutará de forma predeterminada en la versión más reciente del middleware de la plataforma. Estas actualizaciones automáticas siempre son compatibles con las versiones anteriores. Sin embargo, en el caso excepcional de que la actualización automática provoque algún problema en tiempo de ejecución a la aplicación web o de funciones, se puede revertir temporalmente a la versión anterior del middleware. En este artículo se explica cómo anclar temporalmente una aplicación a una versión específica del middleware de autenticación.

### <a name="automatic-and-manual-version-updates"></a>Actualizaciones de versiones automáticas y manuales 

Para anclar la aplicación a una versión específica del middleware de la plataforma, establezca el valor `runtimeVersion` para la aplicación. La aplicación siempre se ejecuta en la versión más reciente, a menos que decida volver a anclarla explícitamente a una versión específica. El número de versiones admitidas a la vez es reducido. Si ancla la aplicación a una versión que ya no se admite, la aplicación usará la versión más reciente. Para ejecutar siempre la versión más reciente, establezca `runtimeVersion` en ~1. 

### <a name="view-and-update-the-current-runtime-version"></a>Visualización y actualización de la versión actual del entorno de ejecución

Puede cambiar la versión en tiempo de ejecución utilizada por la aplicación. La nueva versión en tiempo de ejecución surtirá efecto después de reiniciar la aplicación. 

#### <a name="view-the-current-runtime-version"></a>Visualización de la versión actual del runtime

Para ver la versión actual del middleware de autenticación de la plataforma, utilice la CLI de Azure o uno de los puntos de conexión HTTP de la versión built0 en la aplicación.

##### <a name="from-the-azure-cli"></a>Desde la CLI de Azure

En la CLI de Azure, consulte la versión actual del middleware mediante el comando [az webapp auth show](/cli/azure/webapp/auth?view=azure-cli-latest#az-webapp-auth-show).

```azurecli-interactive
az webapp auth show --name <my_app_name> \
--resource-group <my_resource_group>
```

En el código, reemplace `<my_app_name>` por el nombre de la aplicación. Reemplace también `<my_resource_group>` por el nombre del grupo de recursos para la aplicación.

Verá el campo `runtimeVersion` en la salida de la CLI. Será similar al siguiente ejemplo de salida, que se ha abreviado para mayor claridad: 
```output
{
  "additionalLoginParams": null,
  "allowedAudiences": null,
    ...
  "runtimeVersion": "1.3.2",
    ...
}
```

##### <a name="from-the-version-endpoint"></a>En el punto de conexión de la versión

También puede consultar el punto de conexión de /.auth/version en una aplicación para ver la versión actual del middleware en el que se ejecuta la aplicación. Será similar al siguiente ejemplo de salida:
```output
{
"version": "1.3.2"
}
```

#### <a name="update-the-current-runtime-version"></a>Actualización de la versión de tiempo de ejecución

En la CLI de Azure, puede actualizar el valor de `runtimeVersion` en la aplicación mediante el comando [az webapp auth update](/cli/azure/webapp/auth?view=azure-cli-latest#az-webapp-auth-update).

```azurecli-interactive
az webapp auth update --name <my_app_name> \
--resource-group <my_resource_group> \
--runtime-version <version>
```

Reemplace `<my_app_name>` por el nombre de la aplicación. Reemplace también `<my_resource_group>` por el nombre del grupo de recursos para la aplicación. Por último, reemplace `<version>` por una versión válida del tiempo de ejecución 1.x o `~1` en el caso de la versión más reciente. Encontrará las notas de la versión en las diferentes versiones en tiempo de ejecución [aquí] (https://github.com/Azure/app-service-announcements) ) para que pueda identificar la versión a la que debe anclar la aplicación.

Este comando se puede ejecutar desde [Azure Cloud Shell](../cloud-shell/overview.md), para lo que es preciso hacer clic en **Pruébelo** en el código de ejemplo anterior. También puede usar la [CLI de Azure localmente](/cli/azure/install-azure-cli) para ejecutar este comando después de ejecutar [az login](/cli/azure/reference-index#az-login) para iniciar sesión.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Autenticación y autorización de usuarios de un extremo a otro](tutorial-auth-aad.md)