---
title: Punto de conexión de UserInfo de la Plataforma de identidad de Microsoft | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre el punto de conexión de UserInfo en la Plataforma de identidad de Microsoft.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/22/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 59e811b7813ef94682896e0f95e971ca0094ef65
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119647"
---
# <a name="microsoft-identity-platform-userinfo-endpoint"></a>Punto de conexión de UserInfo de la Plataforma de identidad de Microsoft

El punto de conexión de UserInfo forma parte del [estándar de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo) (OIDC), diseñado para devolver notificaciones sobre el usuario que se ha autenticado.  En la Plataforma de identidad de Microsoft, el punto de conexión de UserInfo se hospeda en Microsoft Graph (https://graph.microsoft.com/oidc/userinfo) ). 

## <a name="find-the-well-known-configuration-endpoint"></a>Búsqueda del punto de conexión de configuración .well-known.

Se puede descubrir mediante programación el punto de conexión de UserInfo con el documento de detección de OpenID Connect, en `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`. Se muestra en el campo `userinfo_endpoint`, y este patrón se puede usar en las nubes para ayudar a apuntar al punto de conexión correcto.  No se recomienda codificar de forma rígida el punto de conexión de UserInfo en la aplicación: use el documento de detección de OIDC para buscar este punto de conexión en runtime.

Como parte de la especificación de OpenID Connect, a menudo las [bibliotecas compatibles con OIDC](https://openid.net/developers/certified/) llaman automáticamente al punto de conexión de UserInfo para obtener información sobre el usuario.  Sin hospedar un punto de conexión de este tipo, la Plataforma de identidad de Microsoft no sería compatible con los estándares y algunas bibliotecas no funcionarían correctamente.  Desde la [lista de notificaciones identificadas en el estándar OIDC](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) se producen las notificaciones de nombre, la notificación de asunto y el correo electrónico cuando están disponibles y se les da su consentimiento.  

## <a name="consider-use-an-id-token-instead"></a>Considere: Usar un token de id. en su lugar.

La información disponible en el token de id. que la aplicación puede recibir es un superconjunto de la información que puede obtener del punto de conexión de UserInfo.  Dado que se puede obtener un token de id. al mismo tiempo que se obtiene un token para llamar al punto de conexión de UserInfo, se recomienda usar ese token de id. para obtener información sobre el usuario en lugar de llamar al punto de conexión de UserInfo.  El uso del token de id. eliminará de una a dos solicitudes de red del inicio de la aplicación, lo que reduce la latencia de dicha aplicación.

Si necesita más detalles sobre el usuario, debe llamar a [Microsoft Graph`/user` API](/graph/api/user-get) para obtener información como el número de la oficina o el puesto.   También se pueden usar [notificaciones opcionales](active-directory-optional-claims.md) para incluir información adicional del usuario en sus tokens de id. y de acceso.

## <a name="calling-the-userinfo-endpoint"></a>Llamada al punto de conexión de UserInfo

UserInfo es una API estándar de token de portador de OAuth, a la que se llama como a cualquier otra Microsoft Graph API mediante el token de acceso recibido al obtener un token para Microsoft Graph. Devuelve una respuesta JSON que contiene notificaciones sobre el usuario.

### <a name="permissions"></a>Permisos

Use los siguientes [permisos de OIDC](v2-permissions-and-consent.md#openid-connect-scopes) para llamar a la API UserInfo. `openid` es necesario, y los ámbitos `profile` y `email` garantizan que se proporcione información adicional en la respuesta.

|Tipo de permiso      | Permisos    |
|:--------------------|:---------------------------------------------------------|
|Delegado (cuenta profesional o educativa). | openid (necesario), perfil, correo electrónico |
|Delegado (cuenta personal de Microsoft) | openid (necesario), perfil, correo electrónico |
|Application | No aplicable |

> [!TIP]
> Copie esta dirección URL en el explorador a fin de obtener un token para el punto de conexión de UserInfo, así como un [token de id.](id-tokens.md), y reemplace el id. de cliente y el URI de redirección por los suyos. Tenga en cuenta que solo solicita ámbitos para los ámbitos de OpenID o Graph, y nada más.  Esto es necesario, ya que no se pueden solicitar permisos para dos recursos diferentes en la misma solicitud de token.
>
> `https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<yourClientID>&response_type=token+id_token&redirect_uri=<YourRedirectUri>&scope=user.read+openid+profile+email&response_mode=fragment&state=12345&nonce=678910`
>
> Se puede usar este token de acceso en la sección siguiente.

Como con cualquier otro token de Microsoft Graph, es posible que el token que reciba aquí no sea un JWT. Si ha iniciado sesión con un usuario de una cuenta de Microsoft, será un formato de token cifrado. Esto se debe a que Microsoft Graph tiene un patrón especial de emisión de tokens. Esto no afecta a su capacidad de usar el token de acceso para llamar al punto de conexión de UserInfo.

### <a name="calling-the-api"></a>Llamada a la API

La API UserInfo admite GET y POST, según la especificación de OIDC.

```http
GET or POST /oidc/userinfo HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6Il…
```

### <a name="userinfo-response"></a>Respuesta de UserInfo

```jsonc
{
    "sub": "OLu859SGc2Sr9ZsqbkG-QbeLgJlb41KcdiPoLYNpSFA",
    "name": "Mikah Ollenburg", // names all require the “profile” scope.
    "family_name": " Ollenburg",
    "given_name": "Mikah",
    "email": "mikoll@contoso.com" //requires the “email” scope.
}
```

Las notificaciones que se enumeran aquí, incluidas `sub`, son las mismas que la aplicación verá en el [token de id.](id-tokens.md) emitido en la aplicación.  

## <a name="notes-and-caveats-on-the-userinfo-endpoint"></a>Notas y advertencias sobre el punto de conexión de UserInfo

* Si se quiere llamar a este punto de conexión de UserInfo, se debe usar el punto de conexión v2.0.  Si se usa el punto de conexión v1.0, obtendrá un token para el punto de conexión v1.0 de UserInfo, hospedado en login.microsoftonline.com.  Se recomienda que todas las bibliotecas y aplicaciones compatibles con OIDC usen el punto de conexión v2.0 para garantizar la compatibilidad.
* La respuesta del punto de conexión de UserInfo no se puede personalizar.  Si quiere personalizar las notificaciones, use [asignación de notificaciones]( active-directory-claims-mapping.md) para editar la información devuelta en los tokens.
* La respuesta del punto de conexión de UserInfo no se puede agregar.  Si quiere obtener notificaciones adicionales sobre el usuario, use [notificaciones opcionales]( active-directory-optional-claims.md) para agregar notificaciones nuevas a los tokens.

## <a name="next-steps"></a>Pasos siguientes

* [Revisar el contenido de los tokens de id.](id-tokens.md)
* [Personalizar el contenido de un token de id. mediante notificaciones opcionales](active-directory-optional-claims.md)
* [Solicitar un token de acceso y un token de id. mediante el protocolo OAuth2](v2-protocols-oidc.md)