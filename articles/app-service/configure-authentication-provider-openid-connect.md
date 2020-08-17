---
title: Configuración de un proveedor de OpenID Connect (versión preliminar)
description: Aprenda a configurar un proveedor de OpenID Connect como proveedor de identidades para una aplicación de App Service o Azure Functions.
ms.topic: article
ms.date: 07/08/2020
ms.reviewer: mahender
ms.openlocfilehash: 89164061a968e37f928f8c21f5323c418e85361f
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87413925"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-login-using-an-openid-connect-provider-preview"></a>Configuración de la aplicación de App Service o Azure Functions para iniciar sesión mediante un proveedor de OpenID Connect (versión preliminar)

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este artículo se muestra cómo configurar Azure App Service o Azure Functions para usar un proveedor de autenticación personalizado que cumpla con las [especificaciones de OpenID Connect](https://openid.net/connect/). OpenID Connect (OIDC) es un estándar del sector que usan muchos proveedores de identidades (IDP). No es necesario comprender los detalles de las especificaciones para configurar la aplicación para que use un IDP que cumpla dichas especificaciones.

Puede configurar la aplicación para que use uno o varios proveedores de OIDC. Debe asignar un nombre único a cada uno en la configuración y solo uno puede actuar como destino de redireccionamiento predeterminado.

> [!CAUTION]
> Al habilitar un proveedor de OpenID Connect, se deshabilitará la característica de administración de la autenticación y autorización de App Service de la aplicación a través de algunos clientes, como Azure Portal, la CLI de Azure y Azure PowerShell. La característica se basa en una nueva superficie de API que, durante la versión preliminar, aún no se ha tomado en cuenta para todas las experiencias de administración.

## <a name="register-your-application-with-the-identity-provider"></a><a name="register"> </a>Registro de la aplicación con el proveedor de identidades

El proveedor le solicitará que registre con él los detalles de la aplicación. Consulte las instrucciones correspondientes a ese proveedor. Deberá recopilar **id. de cliente** y un **secreto de cliente** para la aplicación.

> [!IMPORTANT]
> El secreto de aplicación es una credencial de seguridad importante, No comparta este secreto con nadie ni lo distribuya en una aplicación cliente.
>

> [!NOTE]
> Algunos proveedores pueden requerir pasos adicionales para su configuración y para usar los valores que proporcionan. Por ejemplo, Apple proporciona una clave privada que no se usa como secreto de cliente de OIDC y, en su lugar, se debe usar para crear un token JWT que se trata como el secreto que se proporciona en la configuración de la aplicación (consulte la sección "Creación del secreto de cliente" de la [documentación para iniciar sesión con Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens))
>

Agregue el secreto de cliente como una [configuración de aplicación](./configure-common.md#configure-app-settings), con un nombre de configuración de su elección. Anote este nombre para usarlo más adelante.

Además, necesitará los metadatos de OpenID Connect para el proveedor. A menudo se expone a través de un [documento de metadatos de configuración](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig), que es la dirección URL del emisor del proveedor con el sufijo `/.well-known/openid-configuration`. Obtenga esta dirección URL de configuración.

Si no puede usar un documento de metadatos de configuración, tendrá que recopilar los siguientes valores por separado:

- La URL del emisor (a veces se muestra como `issuer`)
- El [punto de conexión de autorización de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-3.1) (a veces se muestra como `authorization_endpoint`)
- El [punto de conexión de token de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-3.2) (a veces se muestra como `token_endpoint`)
- La URL del documento de [conjunto de claves web JSON de OAuth 2.0](https://tools.ietf.org/html/rfc8414#section-2) (a veces se muestra como `jwks_uri`)

## <a name="add-provider-information-to-your-application"></a><a name="configure"> </a>Adición de información de un proveedor a su aplicación

> [!NOTE]
> La configuración necesaria está en un nuevo formato de API, que actualmente solo es compatible con la [configuración basada en archivos (versión preliminar)](.\app-service-authentication-how-to.md#config-file). Debe seguir los pasos que se indican a continuación para usar este tipo de archivo.

Esta sección le guiará a través de la actualización de la configuración para incluir su nuevo IDP. La siguiente es una configuración de ejemplo.

1. En el objeto `identityProviders`, agregue un objeto `openIdConnectProviders` si todavía no existe ninguno.
1. Agregue una clave para el nuevo proveedor en el objeto `openIdConnectProviders`. Este es un nombre descriptivo que se usa para hacer referencia al proveedor en el resto de la configuración. Por ejemplo, si quiere exigir que todas las solicitudes se autentiquen con este proveedor, debe establecer `globalValidation.unauthenticatedClientAction` en "RedirectToLoginPage" y `globalValidation.unauthenticatedClientAction` en este mismo nombre descriptivo.
1. Asigne un objeto a esa clave con un objeto `registration` dentro y, opcionalmente, un objeto `login`:
    
    ```json
    "myCustomIDP" : {
       "registration" : {},
       "login": {
             "nameClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
             "scope": [],
             "loginParameterNames": [],
       }
    }
    ```

1. En el objeto de registro, establezca `clientId` en el id. de cliente que recopiló, establezca `clientCredential.secretSettingName` en el nombre de la configuración de la aplicación en la que almacenó el secreto de cliente y cree un objeto `openIdConnectConfiguration`:

    ```json
    "registration": {
      "clientId": "bd96cf8a-3f2b-4806-b180-d3c5fd11a2be",
      "clientCredential": {
         "secretSettingName": "IDP_CLIENT_SECRET"
      },
      "openIdConnectConfiguration" : {}
    }
    ```

1. En el objeto `openIdConnectConfiguration`, especifique los metadatos de OpenID Connect que recopiló anteriormente. En función de la información recopilada, tendrá dos opciones para especificarla:

    - Establezca la propiedad `wellKnownOpenIdConfiguration` en la URL de metadatos de configuración que recopiló anteriormente.
    - También puede establecer los cuatro valores individuales que recopiló de la siguiente manera:
        - Establezca `issuer` en la URL del emisor
        - Establezca `authorizationEndpoint` en el punto de conexión de autorización
        - Establezca `tokenEndpoint` en el punto de conexión del token
        - Establezca `certificationUri` en la URL del documento del conjunto de claves web JSON

    Estas dos opciones se excluyen mutuamente.

Una vez establecida esta configuración, está listo para usar el proveedor de OpenID Connect para autenticarse en la aplicación.

Una configuración de ejemplo podría ser similar a la siguiente (se usó el inicio de sesión con Apple como ejemplo, en el que el valor de APPLE_GENERATED_CLIENT_SECRET apunta a un token JWT generado según la [documentación de Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)):

```json
{
    "platform": {
        "enabled": true
    },
    "globalValidation": {
        "redirectToProvider": "apple",
        "unauthenticatedClientAction": "RedirectToLoginPage"
    },
    "identityProviders": {
        "openIdConnectProviders": {
            "apple": {
                "registration": {
                    "clientId": "com.contoso.example.client",
                    "clientCredential": {
                        "secretSettingName": "APPLE_GENERATED_CLIENT_SECRET"
                    },
                    "openIdConnectConfiguration": {
                        "wellKnownOpenIdConfiguration": "https://appleid.apple.com/.well-known/openid-configuration"
                    }
                },
                "login": {
                    "nameClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
                    "scope": [],
                    "loginParameterNames": []
                }
            }
        }
    },
    "login": {
        "tokenStore": {
            "enabled": true
        }
    }     
}
```

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
