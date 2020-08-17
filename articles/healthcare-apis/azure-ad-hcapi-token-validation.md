---
title: Validación del token de acceso de Azure API for FHIR
description: Este artículo le guía durante el proceso de validación del token y ofrece sugerencias sobre cómo solucionar problemas de acceso
services: healthcare-apis
author: caitlinv39
ms.reviewer: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: cavoeg
ms.openlocfilehash: 426ec0c2d6cc274aa0b6829eb4a30fd29b9ba8e2
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844667"
---
# <a name="azure-api-for-fhir-access-token-validation"></a>Validación del token de acceso de Azure API for FHIR

La forma en que Azure API for FHIR valida el token de acceso depende de la implementación y la configuración. En este artículo se le guía a lo largo de los pasos de validación, lo que puede ser útil para solucionar problemas de acceso.

## <a name="validate-token-has-no-issues-with-identity-provider"></a>El token de validación no tiene problemas con el proveedor de identidades

El primer paso de la validación del token es comprobar que ha sido emitido por el proveedor de identidades correcto y que no se ha modificado. El servidor de FHIR se configura para usar un proveedor de identidades concreto conocido como autoridad `Authority`. El servidor de FHIR recupera información sobre el proveedor de identidades del punto de conexión `/.well-known/openid-configuration`. Si se usa Azure AD, la dirección URL completa sería:

```
GET https://login.microsoftonline.com/<TENANT-ID>/.well-known/openid-configuration
```

donde `<TENANT-ID>` es el inquilino concreto de Azure AD (ya sea un identificador de inquilino o un nombre de dominio).

Azure AD devuelve un documento como el siguiente al servidor de FHIR.

```json
{
    "authorization_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/authorize",
    "token_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/token",
    "token_endpoint_auth_methods_supported": [
        "client_secret_post",
        "private_key_jwt",
        "client_secret_basic"
    ],
    "jwks_uri": "https://login.microsoftonline.com/common/discovery/keys",
    "response_modes_supported": [
        "query",
        "fragment",
        "form_post"
    ],
    "subject_types_supported": [
        "pairwise"
    ],
    "id_token_signing_alg_values_supported": [
        "RS256"
    ],
    "http_logout_supported": true,
    "frontchannel_logout_supported": true,
    "end_session_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/logout",
    "response_types_supported": [
        "code",
        "id_token",
        "code id_token",
        "token id_token",
        "token"
    ],
    "scopes_supported": [
        "openid"
    ],
    "issuer": "https://sts.windows.net/<TENANT-ID>/",
    "claims_supported": [
        "sub",
        "iss",
        "cloud_instance_name",
        "cloud_instance_host_name",
        "cloud_graph_host_name",
        "msgraph_host",
        "aud",
        "exp",
        "iat",
        "auth_time",
        "acr",
        "amr",
        "nonce",
        "email",
        "given_name",
        "family_name",
        "nickname"
    ],
    "microsoft_multi_refresh_token": true,
    "check_session_iframe": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/checksession",
    "userinfo_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/openid/userinfo",
    "tenant_region_scope": "WW",
    "cloud_instance_name": "microsoftonline.com",
    "cloud_graph_host_name": "graph.windows.net",
    "msgraph_host": "graph.microsoft.com",
    "rbac_url": "https://pas.windows.net"
}
``` 
Las propiedades importantes para el servidor de FHIR son `jwks_uri`, que indica al servidor dónde capturar las claves de cifrado necesarias para validar la firma del token, y `issuer`, que indica al servidor qué va a haber en la notificación del emisor (`iss`) de los tokens emitidos por este servidor. El servidor de FHIR puede usar esto para validar que está recibiendo un token auténtico.

## <a name="validate-claims-of-the-token"></a>Validación de notificaciones del token

Una vez que el servidor ha comprobado la autenticidad del token, el servidor de FHIR pasa a validar que el cliente tenga las notificaciones necesarias para acceder al token.

Si se usa Azure API for FHIR, el servidor valida que:

1. El token tenga el derecho `Audience` (notificación `aud`).
1. El usuario o la entidad de seguridad para los que se ha emitido el token tenga acceso al plano de datos del servidor de FHIR. La notificación `oid` del token contiene un identificador de objeto de identidad que identifica de forma única al usuario o a la entidad de seguridad.

Se recomienda que el servicio FHIR se [configure para usar RBAC de Azure](configure-azure-rbac.md) para la administración de las asignaciones de roles del plano de datos. Pero también puede [configurar RBAC local](configure-local-rbac.md) si el servicio FHIR usa un inquilino de Azure Active Directory externo o secundario. 

Si se usa el servidor de FHIR de Microsoft de OSS para Azure, el servidor valida que:

1. El token tenga el derecho `Audience` (notificación `aud`).
1. El token tenga un rol en la notificación `roles` a la que se permite el acceso al servidor de FHIR.

Vea detalles sobre cómo [definir roles en el servidor de FHIR](https://github.com/microsoft/fhir-server/blob/master/docs/Roles.md).

Un servidor de FHIR también puede validar que un token de acceso tenga los ámbitos (en la notificación del token `scp`) para acceder a la sección de FHIR API a la que un cliente está intentando acceder. Actualmente, Azure API for FHIR y el servidor de FHIR para Azure no validan ámbitos de token.

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo realizar la validación del token, puede completar el tutorial para crear una aplicación de JavaScript y leer los datos de FHIR.

>[!div class="nextstepaction"]
>[Tutorial de aplicación web](tutorial-web-app-fhir-server.md)