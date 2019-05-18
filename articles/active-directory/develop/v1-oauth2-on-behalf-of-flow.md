---
title: Autenticación entre servicios de Azure Active Directory con la especificación provisional On-Behalf-Of de OAuth 2.0 | Microsoft Docs
description: En este artículo se describe cómo usar los mensajes HTTP para implementar la autenticación entre servicios con el flujo On-Behalf-Of de OAuth 2.0.
services: active-directory
documentationcenter: .net
author: navyasric
manager: CelesteDG
editor: ''
ms.assetid: 09f6f318-e88b-4024-9ee1-e7f09fb19a82
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/06/2017
ms.author: ryanwi
ms.reviewer: hirsin, nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: a2983980786fc706d103c0147a0776f2ff8c2d4f
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545470"
---
# <a name="service-to-service-calls-that-use-delegated-user-identity-in-the-on-behalf-of-flow"></a>Llamadas entre servicios que usan la identidad de usuarios delegada en el flujo de On-Behalf-Of

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

El flujo de On-Behalf-Of (OBO) de OAuth 2.0 permite que una aplicación que invoca un servicio o una API web pase la autenticación de usuario a otro servicio o API web. El flujo de OBO propaga la identidad y los permisos del usuario delegado a través de la cadena de solicitud. Para que el servicio de nivel intermedio realice solicitudes autenticadas al servicio de bajada, debe proteger un token de acceso de Azure Active Directory (Azure AD) en nombre del usuario.

> [!IMPORTANT]
> Desde mayo de 2018, no se puede usar el parámetro `id_token` para el flujo de On-Behalf-Of.  Las aplicaciones de una sola página han de pasar un token de acceso a un cliente confidencial de nivel intermedio para ejecutar flujos de OBO. Para obtener más información acerca de los clientes que pueden realizar llamadas On-Behalf-Of, consulte las [limitaciones](#client-limitations).

## <a name="on-behalf-of-flow-diagram"></a>Diagrama del flujo de On-Behalf-Of

El flujo de OBO se inicia una vez que el usuario se ha autenticado en una aplicación que usa el [flujo de concesión del código de autorización de OAuth 2.0](v1-protocols-oauth-code.md). En ese momento, la aplicación envía un token de acceso (token A) a la API web de nivel intermedio (API A) que contiene las solicitudes y consentimientos del usuario para acceder a la API A. A continuación, la API A realiza una solicitud autenticada a la API web de bajada (API B).

Estos pasos constituyen el flujo de On-Behalf-Of: ![flujo de On-Behalf-Of de OAuth 2.0](./media/v1-oauth2-on-behalf-of-flow/active-directory-protocols-oauth-on-behalf-of-flow.png)

1. La aplicación cliente realiza una solicitud a la API A con el token A.
1. La API A se autentica en el punto de conexión de emisión de tokens de Azure AD y solicita un token para obtener acceso a la API B.
1. El punto de conexión de emisión de tokens de Azure AD valida las credenciales de la API A con el token A y emite el token de acceso para la API B (token B).
1. La solicitud a la API B contiene el token B en el encabezado de autorización.
1. La API B devuelve los datos del recurso protegido.

>[!NOTE]
>La notificación de audiencia en un token de acceso que se usa para solicitar un token para un servicio de bajada debe ser el identificador del servicio que realiza la solicitud de OBO. El token también debe estar firmado con la clave de firma global de Azure Active Directory (que es el valor predeterminado para las aplicaciones registradas a través de **Registros de aplicaciones** del portal).

## <a name="register-the-application-and-service-in-azure-ad"></a>Registro de la aplicación y el servicio en Azure AD

Registre el servicio de nivel intermedio y la aplicación cliente en Azure AD.

### <a name="register-the-middle-tier-service"></a>Registro del servicio de nivel intermedio

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
1. En la barra superior, seleccione su cuenta y, en la lista **Directorio**, seleccione un inquilino de Active Directory para la aplicación.
1. Haga clic en **Más servicios** en el panel izquierdo y elija **Azure Active Directory**.
1. Seleccione **Registros de aplicaciones** y, después, haga clic en **Nuevo registro de aplicaciones**.
1. Escriba un nombre descriptivo para la aplicación y seleccione el tipo de aplicación.
    1. Según el tipo de aplicación, establezca la dirección URL de redireccionamiento o de inicio de sesión en la URL base.
    1. Haga clic en **Crear** para crear la aplicación.
1. Genere un secreto de cliente antes de salir de Azure Portal.
   1. Todavía en Azure Portal, elija la aplicación y seleccione **Configuración**.
   1. Seleccione **Claves** en el menú de configuración y agregue una clave con una duración de un año o dos años.
   1. Al guardar esta página, Azure Portal muestra el valor de clave. Copie y guarde el valor de clave en una ubicación segura.

      > [!IMPORTANT]
      > Necesitará la clave para configurar las opciones de la aplicación en la implementación. Este valor de clave no se volverá a mostrar y no se puede recuperar de ninguna otra manera. Regístrelo en cuanto esté visible en Azure Portal.

### <a name="register-the-client-application"></a>Registro del tipo de aplicación cliente

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
1. En la barra superior, seleccione su cuenta y, en la lista **Directorio**, seleccione un inquilino de Active Directory para la aplicación.
1. Haga clic en **Más servicios** en el panel izquierdo y elija **Azure Active Directory**.
1. Seleccione **Registros de aplicaciones** y, después, haga clic en **Nuevo registro de aplicaciones**.
1. Escriba un nombre descriptivo para la aplicación y seleccione el tipo de aplicación.
   1. Según el tipo de aplicación, establezca la dirección URL de redireccionamiento o de inicio de sesión en la URL base.
   1. Haga clic en **Crear** para crear la aplicación.
1. Configure permisos para la aplicación.
   1. En el menú Configuración, elija la sección **Permisos necesarios** y, después, seleccione **Agregar** y **Seleccionar una API**.
   1. Escriba el nombre del servicio de nivel intermedio en el campo de texto.
   1. Elija **Seleccionar permisos** y, después, seleccione **Access service name** (Acceder al nombre del servicio).

### <a name="configure-known-client-applications"></a>Configuración de aplicaciones cliente conocidas

En este caso, el servicio de nivel intermedio debe obtener el consentimiento del usuario para obtener acceso a la API de bajada sin necesidad de interacción de un usuario. La opción para conceder acceso a la API de bajada debe presentarse inicialmente como parte del paso de autorización durante la autenticación.

Siga estos pasos para enlazar explícitamente el registro de la aplicación cliente de Azure AD con el registro del servicio de nivel intermedio. Esta operación combina el consentimiento requerido por el cliente y el nivel intermedio en un único cuadro de diálogo.

1. Vaya al registro del servicio de nivel intermedio y haga clic en **Manifiesto** para abrir el editor de manifiestos.
1. Busque la propiedad de matriz `knownClientApplications` y agregue el identificador de cliente de la aplicación cliente como un elemento.
1. Seleccione **Guardar** para guardar el manifiesto.

## <a name="service-to-service-access-token-request"></a>Solicitud de token de acceso entre servicios

Para solicitar un token de acceso, realice una solicitud HTTP POST al punto de conexión específico del inquilino de Azure AD con los parámetros siguientes:

```
https://login.microsoftonline.com/<tenant>/oauth2/token
```

La aplicación cliente está protegida mediante un secreto compartido o un certificado.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Primer caso: solicitud de token de acceso con un secreto compartido

Cuando se utiliza un secreto compartido, una solicitud de token de acceso entre servicios contiene los parámetros siguientes:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| grant_type |requerido | Tipo de la solicitud de token. Una solicitud OBO usa JSON Web Token, por lo que el valor debe ser **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| Aserción |requerido | Valor del token de acceso usado en la solicitud. |
| client_id |requerido | Identificador de aplicación asignado al servicio de llamada durante el registro con Azure AD. Para buscar el identificador de la aplicación en Azure Portal, seleccione **Active Directory**, elija el directorio y, por último, seleccione el nombre de la aplicación. |
| client_secret |requerido | La clave registrada para el servicio de llamada de Azure AD. Este valor debe haberse anotado en el momento del registro. |
| Recurso |requerido | URI del identificador de la aplicación del servicio de recepción (recurso seguro). Para buscar el URI del identificador de la aplicación en Azure Portal, seleccione **Active Directory** y elija el directorio. Seleccione el nombre de la aplicación, elija **Todas las opciones** y, después, seleccione **Propiedades**. |
| requested_token_use |requerido | Especifica cómo se debe procesar la solicitud. En el "flujo en nombre de", el valor debe ser **on_behalf_of**. |
| ámbito |requerido | Lista de ámbitos separados por un espacio para la solicitud de token. Para OpenID Connect, el ámbito **openid** debe especificarse.|

#### <a name="example"></a>Ejemplo

El siguiente elemento HTTP POST solicita un token de acceso para la API web https://graph.windows.net. El parámetro `client_id` permite identificar el servicio que solicita el token de acceso.

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_secret=0Y1W%2BY3yYb3d9N8vSjvm8WrGzVZaAaHbHHcGbcgG%2BoI%3D
&resource=https%3A%2F%2Fgraph.windows.net
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=openid
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Segundo caso: solicitud de token de acceso con un certificado

Una solicitud de token de acceso entre servicios con un certificado contiene los parámetros siguientes:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| grant_type |requerido | Tipo de la solicitud de token. Una solicitud OBO usa un token de acceso JWT, por lo que el valor debe ser **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| Aserción |requerido | Valor del token usado en la solicitud. |
| client_id |requerido | Identificador de aplicación asignado al servicio de llamada durante el registro con Azure AD. Para buscar el identificador de la aplicación en Azure Portal, seleccione **Active Directory**, elija el directorio y, por último, seleccione el nombre de la aplicación. |
| client_assertion_type |requerido |El valor debe ser `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| client_assertion |requerido | Debe crear una instancia de JSON Web Token y firmar con el certificado que ha registrado como credenciales de la aplicación. Consulte el artículo acerca de las [credenciales de certificado](active-directory-certificate-credentials.md) para obtener información sobre el formato de la aserción y de cómo registrar el certificado.|
| Recurso |requerido | URI del identificador de la aplicación del servicio de recepción (recurso seguro). Para buscar el URI del identificador de la aplicación en Azure Portal, seleccione **Active Directory** y elija el directorio. Seleccione el nombre de la aplicación, elija **Todas las opciones** y, después, seleccione **Propiedades**. |
| requested_token_use |requerido | Especifica cómo se debe procesar la solicitud. En el "flujo en nombre de", el valor debe ser **on_behalf_of**. |
| ámbito |requerido | Lista de ámbitos separados por un espacio para la solicitud de token. Para OpenID Connect, el ámbito **openid** debe especificarse.|

Estos parámetros son casi iguales a los de la solicitud con un secreto compartido, salvo que el parámetro `client_secret parameter` se sustituye por los parámetros `client_assertion_type` y `client_assertion`.

#### <a name="example"></a>Ejemplo

El siguiente elemento HTTP POST solicita un token de acceso para la API web https://graph.windows.net con un certificado. El parámetro `client_id` permite identificar el servicio que solicita el token de acceso.

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&resource=https%3A%2F%2Fgraph.windows.net
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=openid
```

## <a name="service-to-service-access-token-response"></a>Respuesta de token de acceso entre servicios

Una respuesta correcta es una respuesta de OAuth 2.0 de JSON con los parámetros siguientes:

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| token_type |Indica el valor de tipo de token. El único tipo que admite Azure AD es el **portador**. Para más información sobre los tokens de portador, consulte [OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |
| ámbito |Ámbito de acceso concedido en el token. |
| expires_in |Período de validez del token de acceso (en segundos). |
| expires_on |La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha de expiración. Este valor se utiliza para determinar la duración de los tokens almacenados en caché. |
| Recurso |URI del identificador de la aplicación del servicio de recepción (recurso seguro). |
| access_token |El token de acceso solicitado. El servicio de llamada puede usar este token para autenticarse en el servicio de recepción. |
| ID_token |Token de identificador solicitado. El servicio de llamada puede usar este token para verificar la identidad del usuario y comenzar una sesión con el usuario. |
| refresh_token |Token de actualización para el token de acceso solicitado. El servicio de llamada puede usar este token para solicitar otro token de acceso después de que expire el token de acceso actual. |

### <a name="success-response-example"></a>Ejemplo de respuesta correcta

En el ejemplo siguiente se muestra una respuesta correcta a una solicitud de un token de acceso para la API web https://graph.windows.net.

```
{
    "token_type":"Bearer",
    "scope":"User.Read",
    "expires_in":"43482",
    "ext_expires_in":"302683",
    "expires_on":"1493466951",
    "not_before":"1493423168",
    "resource":"https://graph.windows.net",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0IiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiLyIsImlhdCI6MTQ5MzQyMzE2OCwibmJmIjoxNDkzNDIzMTY4LCJleHAiOjE0OTM0NjY5NTEsImFjciI6IjEiLCJhaW8iOiJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJhcHBpZGFjciI6IjEiLCJlX2V4cCI6MzAyNjgzLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJwdWlkIjoiMTAwMzNGRkZBMTJFRDdGRSIsInNjcCI6IlVzZXIuUmVhZCIsInN1YiI6IjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLCJ0aWQiOiIyNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IiLCJ1bmlxdWVfbmFtZSI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXBuIjoibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJ4Q3dmemhhLVAwV0pRT0x4Q0dnS0FBIiwidmVyIjoiMS4wIn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ",
    "refresh_token":"AQABAAAAAABnfiG-mA6NTae7CdWW7QfdjKGu9-t1scy_TDEmLi4eLQMjJGt_nAoVu6A4oSu1KsRiz8XyQIPKQxSGfbf2FoSK-hm2K8TYzbJuswYusQpJaHUQnSqEvdaCeFuqXHBv84wjFhuanzF9dQZB_Ng5za9xKlUENrNtlq9XuLNVKzxEyeUM7JyxzdY7JiEphWImwgOYf6II316d0Z6-H3oYsFezf4Xsjz-MOBYEov0P64UaB5nJMvDyApV-NWpgklLASfNoSPGb67Bc02aFRZrm4kLk-xTl6eKE6hSo0XU2z2t70stFJDxvNQobnvNHrAmBaHWPAcC3FGwFnBOojpZB2tzG1gLEbmdROVDp8kHEYAwnRK947Py12fJNKExUdN0njmXrKxNZ_fEM33LHW1Tf4kMX_GvNmbWHtBnIyG0w5emb-b54ef5AwV5_tGUeivTCCysgucEc-S7G8Cz0xNJ_BOiM_4bAv9iFmrm9STkltpz0-Tftg8WKmaJiC0xXj6uTf4ZkX79mJJIuuM7XP4ARIcLpkktyg2Iym9jcZqymRkGH2Rm9sxBwC4eeZXM7M5a7TJ-5CqOdfuE3sBPq40RdEWMFLcrAzFvP0VDR8NKHIrPR1AcUruat9DETmTNJukdlJN3O41nWdZOVoJM-uKN3uz2wQ2Ld1z0Mb9_6YfMox9KTJNzRzcL52r4V_y3kB6ekaOZ9wQ3HxGBQ4zFt-2U0mSszIAA",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8yNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IvIiwiaWF0IjoxNDkzNDIzMTY4LCJuYmYiOjE0OTM0MjMxNjgsImV4cCI6MTQ5MzQ2Njk1MSwiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXRpIjoieEN3ZnpoYS1QMFdKUU9MeENHZ0tBQSIsInZlciI6IjEuMCJ9."
}
```

### <a name="error-response-example"></a>Ejemplo de respuesta de error

El punto de conexión del token de Azure AD devuelve una respuesta de error cuando intenta adquirir un token de acceso para una API de bajada que se establece con una directiva de acceso condicional (por ejemplo, la autenticación multifactor). El servicio de nivel intermedio debe exponer el error a la aplicación cliente para que esta pueda proporcionar la interacción del usuario necesaria para cumplir la directiva de acceso condicional.

```
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>Usar el token de acceso para obtener acceso al recurso protegido

El servicio de nivel intermedio puede usar el token de acceso adquirido para realizar solicitudes autenticadas a la API web de bajada mediante el establecimiento del token en el encabezado `Authorization`.

### <a name="example"></a>Ejemplo

```
GET /me?api-version=2013-11-08 HTTP/1.1
Host: graph.windows.net
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0IiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiLyIsImlhdCI6MTQ5MzQyMzE2OCwibmJmIjoxNDkzNDIzMTY4LCJleHAiOjE0OTM0NjY5NTEsImFjciI6IjEiLCJhaW8iOiJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJhcHBpZGFjciI6IjEiLCJlX2V4cCI6MzAyNjgzLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJwdWlkIjoiMTAwMzNGRkZBMTJFRDdGRSIsInNjcCI6IlVzZXIuUmVhZCIsInN1YiI6IjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLCJ0aWQiOiIyNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IiLCJ1bmlxdWVfbmFtZSI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXBuIjoibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJ4Q3dmemhhLVAwV0pRT0x4Q0dnS0FBIiwidmVyIjoiMS4wIn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>Aserciones de SAML obtenidas con un flujo de OBO de OAuth2.0

Algunos servicios web basados en OAuth necesitan tener acceso a otras API de servicio web que aceptan aserciones de SAML en flujos no interactivos. Azure Active Directory puede proporcionar una aserción de SAML en respuesta a un flujo de On-Behalf-Of que usa un servicio web basado en SAML como un recurso de destino.

>[!NOTE]
>Se trata de una extensión no estándar del flujo de On-Behalf-Of de OAuth 2.0 que permite a una aplicación basada en OAuth2 acceder a puntos de conexión de API de servicio web que consumen tokens SAML.

> [!TIP]
> Cuando llama a un servicio web SAML protegido desde una aplicación web front-end, solo puede llamar a la API e iniciar un flujo de autenticación interactiva normal que usará la sesión existente del usuario. Solo debe considerar el uso de un flujo de OBO cuando una llamada entre servicios requiere que un token SAML proporcione el contexto del usuario.

### <a name="obtain-a-saml-token-by-using-an-obo-request-with-a-shared-secret"></a>Obtener un token SAML mediante una solicitud OBO con un secreto compartido

Una solicitud de servicio a servicio para una aserción SAML contiene los siguientes parámetros:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| grant_type |requerido | Tipo de la solicitud de token. En el caso de una solicitud que usa un JWT, el valor debe ser **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| Aserción |requerido | Valor del token de acceso usado en la solicitud.|
| client_id |requerido | Identificador de aplicación asignado al servicio de llamada durante el registro con Azure AD. Para buscar el identificador de la aplicación en Azure Portal, seleccione **Active Directory**, elija el directorio y, por último, seleccione el nombre de la aplicación. |
| client_secret |requerido | La clave registrada para el servicio de llamada de Azure AD. Este valor debe haberse anotado en el momento del registro. |
| Recurso |requerido | URI del identificador de la aplicación del servicio de recepción (recurso seguro). Este es el recurso que será la audiencia del token SAML. Para buscar el URI del identificador de la aplicación en Azure Portal, seleccione **Active Directory** y elija el directorio. Seleccione el nombre de la aplicación, elija **Todas las opciones** y, después, seleccione **Propiedades**. |
| requested_token_use |requerido | Especifica cómo se debe procesar la solicitud. En el "flujo en nombre de", el valor debe ser **on_behalf_of**. |
| requested_token_type | requerido | Especifica el tipo de token solicitado. El valor puede ser **urn:ietf:params:oauth:token-type:saml2** o **urn:ietf:params:oauth:token-type:saml1**, en función de los requisitos del recurso al que se accede. |

La respuesta contiene un token SAML codificado con UTF8 y Base64url.

- **SubjectConfirmationData para una aserción SAML procedente de una llamada OBO**: si la aplicación de destino requiere un valor de destinatario en **SubjectConfirmationData**, el valor debe ser una dirección URL de respuesta que no sea de caracteres comodín en la configuración de la aplicación de recursos.
- **El nodo SubjectConfirmationData**: este no puede contener un atributo **InResponseTo**, ya que no forma parte de una respuesta SAML. La aplicación que recibe el token SAML debe tener la capacidad de aceptar la aserción SAML sin el atributo **InResponseTo**.

- **Consentimiento**: debe otorgarse un consentimiento con el fin de recibir un token SAML que contenga datos de usuario en un flujo de OAuth. Para obtener información sobre los permisos y el consentimiento del administrador, consulte [Permisos y consentimiento en el punto de conexión v1.0 de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/v1-permissions-and-consent).

### <a name="response-with-saml-assertion"></a>Respuesta con aserción SAML

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| token_type |Indica el valor de tipo de token. El único tipo que admite Azure AD es el **portador**. Para más información sobre los tokens de portador, consulte [The OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |
| ámbito |Ámbito de acceso concedido en el token. |
| expires_in |Período de validez del token de acceso (en segundos). |
| expires_on |La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha de expiración. Este valor se utiliza para determinar la duración de los tokens almacenados en caché. |
| Recurso |URI del identificador de la aplicación del servicio de recepción (recurso seguro). |
| access_token |Parámetro que devuelve la aserción de SAML. |
| refresh_token |El token de actualización. El servicio de llamada puede usar este token para solicitar otro token de acceso después de que expire la aserción SAML actual. |

- token_type: Portador
- expires_in: 3296
- ext_expires_in: 0
- expires_on: 1529627844
- resource: `https://api.contoso.com`
- access_token: \<Aserción SAML\>
- issued_token_type: urn:ietf:params:oauth:token-type:saml2
- refresh_token: \<Token de actualización\>

## <a name="client-limitations"></a>Limitaciones del cliente

Los clientes públicos con direcciones URL de respuesta con caracteres comodín no pueden utilizar el parámetro `id_token` con flujos OBO. Pero un cliente confidencial todavía puede canjear los tokens de **acceso** adquiridos a través del flujo de concesión implícita, aunque el cliente público tenga registrado un URI de redireccionamiento con caracteres comodín.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el protocolo OAuth 2.0 y conozca otra manera de realizar la autenticación entre servicios que usa las credenciales del cliente:

* [Autenticación entre servicios mediante la concesión de credenciales de cliente de OAuth 2.0 en Azure AD](v1-oauth2-client-creds-grant-flow.md)
* [OAuth 2.0 en Azure AD](v1-protocols-oauth-code.md)
