---
title: Configuración de identidad de Azure Active Directory para Azure API for FHIR
description: Conozca los principios de identidad, autenticación y autorización para los servidores de FHIR de Azure.
services: healthcare-apis
author: caitlinv39
ms.reviewer: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: cavoeg
ms.openlocfilehash: cdb73670996341e9219230bb277e087009266f32
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846027"
---
# <a name="azure-active-directory-identity-configuration-for-azure-api-for-fhir"></a>Configuración de identidad de Azure Active Directory para Azure API for FHIR

Una parte importante al trabajar con los datos del sector sanitario es asegurarse de que sean seguros y de que los usuarios o las aplicaciones no autorizados no puedan acceder a ellos. Los servidores de FHIR usan [OAuth 2.0](https://oauth.net/2/) para garantizar esta seguridad de datos. El servicio [Azure API for FHIR](https://azure.microsoft.com/services/azure-api-for-fhir/) se protege mediante [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/), que es un ejemplo de un proveedor de identidades de OAuth 2.0. En este artículo se proporciona información general sobre la autorización de un servidor de FHIR y los pasos necesarios para obtener un token para acceder a dicho servidor. Aunque estos pasos se aplicarán a cualquier servidor de FHIR y a cualquier proveedor de identidades, en este artículo le guiaremos a través de Azure API for FHIR como el servidor de FHIR y Azure AD como nuestro proveedor de identidades.

## <a name="access-control-overview"></a>Introducción al control de acceso

Para que una aplicación cliente acceda a Azure API for FHIR, debe presentar un token de acceso. El token de acceso es una colección de propiedades (notificaciones) codificada [Base64](https://en.wikipedia.org/wiki/Base64) y firmada que proporcionan información sobre la identidad del cliente y los roles y privilegios que se conceden a este.

Hay varias maneras de obtener un token, pero a Azure API for FHIR no le importa cómo se obtiene, siempre que sea un token adecuadamente firmado con las notificaciones correctas. 

Con el [flujo de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/v1-protocols-oauth-code) como ejemplo, el acceso a un servidor de FHIR pasa por los cuatro pasos siguientes:

![Authorization de FHIR](media/azure-ad-hcapi/fhir-authorization.png)

1. El cliente envía una solicitud al punto de conexión `/authorize` de Azure AD. Azure AD redirigirá al cliente a una página de inicio de sesión donde el usuario se autenticará mediante las credenciales apropiadas (por ejemplo, el nombre de usuario y la contraseña o la autenticación en dos fases). Vea los detalles sobre la [obtención de un código de autorización](https://docs.microsoft.com/azure/active-directory/develop/v1-protocols-oauth-code#request-an-authorization-code). Después de que la autenticación se haya realizado correctamente, se devuelve un *código de autorización* al cliente. Azure AD solo permitirá que se devuelva este código de autorización a una dirección URL de respuesta registrada configurada en el registro de la aplicación cliente (consulte la información que figura a continuación).
1. La aplicación cliente intercambia el código de autorización para un *token de acceso* en el punto de conexión `/token` de Azure AD. Al solicitar un token, es posible que la aplicación cliente tenga que proporcionar un secreto de cliente (la contraseña de aplicaciones). Consulte los detalles sobre la [obtención de un token de acceso](https://docs.microsoft.com/azure/active-directory/develop/v1-protocols-oauth-code#use-the-authorization-code-to-request-an-access-token).
1. El cliente realiza una solicitud a Azure API for FHIR, por ejemplo `GET /Patient`, para buscar todos los pacientes. Al efectuar la solicitud, incluye el token de acceso en un encabezado de solicitud HTTP, por ejemplo `Authorization: Bearer eyJ0e...`, donde `eyJ0e...` representa el token de acceso con codificación Base64.
1. Azure API for FHIR valida que el token contenga las notificaciones adecuadas (propiedades en el token). Si todo concuerda, se completará la solicitud y se devolverá un conjunto de FHIR con los resultados al cliente.

Es importante tener en cuenta que Azure API for FHIR no está implicado en la validación de las credenciales de usuario y no emite el token. Azure AD lleva a cabo la autenticación y la creación del token. Azure API for FHIR simplemente valida que el token esté firmado correctamente (que sea auténtico) y que tenga las notificaciones adecuadas.

## <a name="structure-of-an-access-token"></a>Estructura de un token de acceso

El desarrollo de aplicaciones de FHIR a menudo implica la depuración de problemas de acceso. Si a un cliente se le deniega el acceso a Azure API for FHIR, es útil entender la estructura del token de acceso y cómo se puede descodificar para inspeccionar el contenido (las notificaciones) del token. 

Normalmente, los servidores de FHIR esperan un método [JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token) (JWT, a veces pronunciado "jot"). Consta de tres partes:

1. Un encabezado, que podría ser similar a lo siguiente:
    ```json
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```
1. La carga útil (las notificaciones), por ejemplo:
    ```json
    {
     "oid": "123",
     "iss": "https://issuerurl",
     "iat": 1422779638,
     "roles": [
        "admin"
      ]
    }
    ```
1. Una firma, que se calcula concatenando el contenido con codificación Base64 del encabezado y la carga útil, y calculando un hash criptográfico de ellos en función del algoritmo (`alg`) especificado en el encabezado. Un servidor podrá obtener claves públicas del proveedor de identidades y validar que este token lo emitió un proveedor de identidades específico y no se ha manipulado.

El token completo consta de las versiones con codificación Base64 (realmente codificadas en URL como Base64) de esos tres segmentos. Los tres segmentos se concatenan y se separan con `.` (punto).

A continuación se muestra un token de ejemplo:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJvaWQiOiIxMjMiLCAiaXNzIjoiaHR0cHM6Ly9pc3N1ZXJ1cmwiLCJpYXQiOjE0MjI3Nzk2MzgsInJvbGVzIjpbImFkbWluIl19.gzSraSYS8EXBxLN_oWnFSRgCzcmJmMjLiuyu5CSpyHI
```

El token se puede descodificar e inspeccionar con herramientas como [https://jwt.ms](https://jwt.ms). El resultado de descodificar el token es:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}.{
  "oid": "123",
  "iss": "https://issuerurl",
  "iat": 1422779638,
  "roles": [
    "admin"
  ]
}.[Signature]
```

## <a name="obtaining-an-access-token"></a>Obtención de un token de acceso

Como se mencionó anteriormente, hay varias maneras de obtener un token desde Azure AD. Esas formas se describen en detalle en la [documentación para desarrolladores de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/).

Azure AD tiene dos versiones diferentes de los puntos de conexión OAuth 2.0, a las que se hace referencia como `v1.0` y `v2.0`. Ambas versiones son puntos de conexión de OAuth 2.0 y las designaciones `v1.0` y `v2.0` hacen referencia a las diferencias en el modo en que Azure AD implementa es estándar. 

Al usar un servidor de FHIR, puede usar los puntos de conexión `v1.0` o `v2.0`. La elección puede depender de las bibliotecas de autenticación que se usan en la aplicación cliente.

Las secciones pertinentes de la documentación de Azure AD son:

* Punto de conexión `v1.0`:
    * [Flujo de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/v1-protocols-oauth-code).
    * [Flujo de credenciales de cliente](https://docs.microsoft.com/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow).
* Punto de conexión `v2.0`:
    * [Flujo de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).
    * [Flujo de credenciales de cliente](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow).

Hay otras variaciones (por ejemplo, en nombre del flujo) para obtener un token. Vea la documentación de Azure AD para más detalles. Al usar Azure API for FHIR, también hay algunos métodos abreviados para obtener un token de acceso (con fines de depuración) [mediante la CLI de Azure](get-healthcare-apis-access-token-cli.md).

## <a name="next-steps"></a>Pasos siguientes

En este documento, aprendió algunos de los conceptos básicos implicados en la protección del acceso a Azure API for FHIR mediante Azure AD. Para aprender cómo implementar una instancia de Azure API for FHIR, continúe con el inicio rápido de implementación.

>[!div class="nextstepaction"]
>[Implementación de Azure API for FHIR](fhir-paas-portal-quickstart.md)