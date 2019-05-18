---
title: Autenticación entre servicios de Azure AD mediante OAuth2.0 | Microsoft Docs
description: Este artículo describe cómo utilizar los mensajes HTTP para implementar la autenticación entre servicios mediante el flujo de concesión de credenciales de cliente de OAuth2.0.
services: active-directory
documentationcenter: .net
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: a7f939d9-532d-4b6d-b6d3-95520207965d
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/08/2017
ms.author: ryanwi
ms.reviewer: nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9d734db7fbedaf3e3f3cd71c31f9391a2237f5b4
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545260"
---
# <a name="service-to-service-calls-using-client-credentials-shared-secret-or-certificate"></a>Llamadas entre servicios mediante las credenciales del cliente (secreto compartido o certificado)

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

El flujo de concesión de credenciales de cliente de OAuth 2.0 permite a un servicio web (*cliente confidencial*) usar sus propias credenciales para autenticarse al llamar a otro servicio web, en lugar de suplantar a un usuario. En este escenario, el cliente es normalmente un servicio web de nivel intermedio, un servicio demonio o un sitio web. Para conseguir un mayor nivel de control, Azure AD también permite al servicio que realiza la llamada usar un certificado (en lugar de un secreto compartido) como credencial.

## <a name="client-credentials-grant-flow-diagram"></a>Diagrama de flujo de concesión de credenciales de cliente
El diagrama siguiente explica cómo funciona el flujo de concesión de credenciales de cliente en Azure Active Directory (Azure AD).

![Flujo de concesión de credenciales de cliente de OAuth2.0](./media/v1-oauth2-client-creds-grant-flow/active-directory-protocols-oauth-client-credentials-grant-flow.jpg)

1. La aplicación cliente se autentica en el punto de conexión de emisión de tokens de Azure AD y solicita un token de acceso.
2. El punto de conexión de emisión de tokens de Azure AD emite el token de acceso.
3. El token de acceso se utiliza para autenticar los recursos protegidos.
4. Los datos de los recursos protegidos se devuelven a la aplicación cliente.

## <a name="register-the-services-in-azure-ad"></a>Registro de los servicios en Azure AD
Registre el servicio de llamada y el servicio de recepción en Azure Active Directory (Azure AD). Para obtener más información, consulte [Integración de aplicaciones con Azure Active Directory](quickstart-v1-integrate-apps-with-azure-ad.md).

## <a name="request-an-access-token"></a>Solicitar un token de acceso
Para solicitar un token de acceso, utilice una solicitud HTTP POST al punto de conexión específico del inquilino de Azure AD.

```
https://login.microsoftonline.com/<tenant id>/oauth2/token
```

## <a name="service-to-service-access-token-request"></a>Solicitud de token de acceso entre servicios
Se pueden dar dos casos en función de si la aplicación cliente elige un secreto compartido o un certificado para su protección.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Primer caso: solicitud de token de acceso con un secreto compartido
Cuando se utiliza un secreto compartido, una solicitud de token de acceso entre servicios contiene los parámetros siguientes:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| grant_type |requerido |Especifica el tipo de concesión solicitado. En un flujo de concesión de credenciales de cliente, el valor debe ser **client_credentials**. |
| client_id |requerido |Especifica el identificador de cliente de Azure AD del servicio web que realiza la llamada. Para buscar el identificador del cliente de la aplicación que realiza la llamada, en [Azure Portal](https://portal.azure.com), haga clic en **Azure Active Directory** y **Registros de aplicaciones**, y haga clic en la aplicación. El identificador client_id es el *identificador de aplicación* |
| client_secret |requerido |Escriba una clave registrada para el servicio web de llamada o la aplicación de demonio de Azure AD. Para crear una clave, en Azure Portal, haga clic en **Azure Active Directory** y en **Registros de aplicaciones**, haga clic en la aplicación, en **Configuración** y en **Claves** y agregue una clave.  Cuando lo proporcione, codifique como dirección URL este secreto. |
| Recurso |requerido |Escriba el URI del identificador de la aplicación del servicio web de recepción. Para buscar el URI del identificador de la aplicación, en Azure Portal, haga clic en **Azure Active Directory** y en **Registros de aplicaciones**, haga clic en la aplicación de servicio y, a continuación, en **Configuración** y en **Propiedades**. |

#### <a name="example"></a>Ejemplo
El siguiente elemento HTTP POST solicita un [token de acceso](access-tokens.md) para el servicio web https://service.contoso.com/. El parámetro `client_id` permite identificar el servicio web que solicita el token de acceso.

```
POST /contoso.com/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=625bc9f6-3bf6-4b6d-94ba-e97cf07a22de&client_secret=qkDwDJlDfig2IpeuUZYKH1Wb8q1V0ju6sILxQQqhJ+s=&resource=https%3A%2F%2Fservice.contoso.com%2F
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Segundo caso: solicitud de token de acceso con un certificado
Una solicitud de token de acceso entre servicios con un certificado contiene los parámetros siguientes:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| grant_type |requerido |Especifica el tipo de respuesta solicitado. En un flujo de concesión de credenciales de cliente, el valor debe ser **client_credentials**. |
| client_id |requerido |Especifica el identificador de cliente de Azure AD del servicio web que realiza la llamada. Para buscar el identificador del cliente de la aplicación que realiza la llamada, en [Azure Portal](https://portal.azure.com), haga clic en **Azure Active Directory** y **Registros de aplicaciones**, y haga clic en la aplicación. El identificador client_id es el *identificador de aplicación* |
| client_assertion_type |requerido |El valor debe ser `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| client_assertion |requerido | Aserción (un JSON Web Token) que debe crear y firmar con el certificado que ha registrado como credenciales de la aplicación. Lea el artículo sobre las [credenciales de certificado](active-directory-certificate-credentials.md) para obtener información sobre cómo registrar el certificado y el formato de la aserción.|
| resource | requerido |Escriba el URI del identificador de la aplicación del servicio web de recepción. Para buscar el URI del identificador de la aplicación, en Azure Portal, haga clic en **Azure Active Directory** y en **Registros de aplicaciones**, haga clic en la aplicación de servicio y, a continuación, en **Configuración** y en **Propiedades**. |

Tenga en cuenta que los parámetros son casi iguales que en el caso de solicitud con un secreto compartido, salvo que el parámetro client_secret se sustituye por dos parámetros: client_assertion_type y client_assertion.

#### <a name="example"></a>Ejemplo
El siguiente elemento HTTP POST solicita un token de acceso para el servicio web https://service.contoso.com/ con un certificado. El parámetro `client_id` permite identificar el servicio web que solicita el token de acceso.

```
POST /<tenant_id>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

resource=https%3A%2F%contoso.onmicrosoft.com%2Ffc7664b4-cdd6-43e1-9365-c2e1c4e1b3bf&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg&grant_type=client_credentials
```

### <a name="service-to-service-access-token-response"></a>Respuesta de token de acceso entre servicios

Una respuesta correcta contiene una respuesta de OAuth 2.0 de JSON con los siguientes parámetros:

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| access_token |El token de acceso solicitado. El servicio web de llamada puede usar este token para autenticarse en el servicio web de recepción. |
| token_type |Indica el valor de tipo de token. El único tipo que admite Azure AD es el **portador**. Para más información sobre los tokens de portador, consulte [OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |
| expires_in |Durante cuánto tiempo es válido el token de acceso (en segundos). |
| expires_on |La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha de expiración. Este valor se utiliza para determinar la duración de los tokens almacenados en caché. |
| not_before |La hora a partir de la cual el token de acceso pasa a ser utilizable. La fecha se representa como el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha de expiración del token.|
| Recurso |El URI del identificador de la aplicación del servicio web de recepción. |

#### <a name="example-of-response"></a>Ejemplo de respuesta
En el ejemplo siguiente se muestra una respuesta correcta a una solicitud de un token de acceso a un servicio web.

```
{
"access_token":"eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0ODI2NywibmJmIjoxMzg4NDQ4MjY3LCJleHAiOjEzODg0NTIxNjcsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6ImE5OTE5MTYyLTkyMTctNDlkYS1hZTIyLWYxMTM3YzI1Y2RlYSIsInN1YiI6ImE5OTE5MTYyLTkyMTctNDlkYS1hZTIyLWYxMTM3YzI1Y2RlYSIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZS8iLCJhcHBpZCI6ImQxN2QxNWJjLWM1NzYtNDFlNS05MjdmLWRiNWYzMGRkNThmMSIsImFwcGlkYWNyIjoiMSJ9.aqtfJ7G37CpKV901Vm9sGiQhde0WMg6luYJR4wuNR2ffaQsVPPpKirM5rbc6o5CmW1OtmaAIdwDcL6i9ZT9ooIIicSRrjCYMYWHX08ip-tj-uWUihGztI02xKdWiycItpWiHxapQm0a8Ti1CWRjJghORC1B1-fah_yWx6Cjuf4QE8xJcu-ZHX0pVZNPX22PHYV5Km-vPTq2HtIqdboKyZy3Y4y3geOrRIFElZYoqjqSv5q9Jgtj5ERsNQIjefpyxW3EwPtFqMcDm4ebiAEpoEWRN4QYOMxnC9OUBeG9oLA0lTfmhgHLAtvJogJcYFzwngTsVo6HznsvPWy7UP3MINA",
"token_type":"Bearer",
"expires_in":"3599",
"expires_on":"1388452167",
"resource":"https://service.contoso.com/"
}
```

## <a name="see-also"></a>Vea también
* [OAuth 2.0 en Azure AD](v1-protocols-oauth-code.md)
* [Ejemplo en C# de la llamada entre servicios con un secreto compartido](https://github.com/Azure-Samples/active-directory-dotnet-daemon) y [Ejemplo en C# de la llamada entre servicios con un certificado](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential)
