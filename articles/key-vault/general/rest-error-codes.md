---
title: 'Códigos de error de la API de REST: Azure Key Vault'
description: Una operación en un servicio web Azure Key Vault podría devolver estos códigos de error.
keywords: ''
services: machine-learning
author: msmbaldwin
ms.custom: seodec18
ms.author: mbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: reference
ms.date: 12/16/2019
ms.openlocfilehash: 95fb9798bc62236d785c911ec75db8d6e11f6e97
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685721"
---
# <a name="azure-key-vault-rest-api-error-codes"></a>Códigos de error de la API de REST de Azure Key Vault
 
Una operación en un servicio web Azure Key Vault podría devolver los siguientes códigos de error.
 
## <a name="http-401-unauthenticated-request"></a>HTTP 401: Solicitud no autenticada

401 significa que la solicitud no está autenticada para Key Vault. 

Una solicitud se autentica si:

- El almacén de claves conoce la identidad del autor de llamada; y
- Se permite al autor de llamada intentar obtener acceso a recursos de Key Vault. 

Hay varios motivos diferentes por los que una solicitud puede devolver 401.

### <a name="no-authentication-token-attached-to-the-request"></a>No hay ningún token de autenticación asociado a la solicitud. 

A continuación se muestra una solicitud PUT de ejemplo, que establece el valor de un secreto:

``` 
PUT https://putreqexample.vault.azure.net//secrets/DatabaseRotatingPassword?api-version=7.0 HTTP/1.1
x-ms-client-request-id: 03d275a2-52a4-4bed-82c8-6fe15165affb
accept-language: en-US
Authorization: Bearer     eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSIsImtpZCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSJ9.eyJhdWQiOiJodHRwczovL3ZhdWx0LmF6dXJlLm5ldCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpYXQiOjE1NDg2OTc1MTMsIm5iZiI6MTU0ODY5NzUxMywiZXhwIjoxNTQ4NzAxNDEzLCJhaW8iOiI0MkpnWUhoODVqaVBnZHF5ZlRGZE5TdHY3bGUvQkFBPSIsImFwcGlkIjoiZmFkN2Q1YjMtNjlkNi00YjQ4LTkyNTktOGQxMjEyNGUxY2YxIiwiYXBwaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3LyIsIm9pZCI6IjM5NzVhZWVkLTdkMDgtNDUzYi1iNmY0LTQ0NWYzMjY5ODA5MSIsInN1YiI6IjM5NzVhZWVkLTdkMDgtNDUzYi1iNmY0LTQ0NWYzMjY5ODA5MSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInV0aSI6IjItZ3JoUmtlSWs2QmVZLUxuNDJtQUEiLCJ2ZXIiOiIxLjAifQ.fgubiz1MKqTJTXI8dHIV7t9Fle6FdHrkaGYKcBeVRX1WtLVuk1QVxzIFDlZKLXJ7QPNs0KWpeiWQI9IWIRK-8wO38yCqKTfDlfHOiNWGOpkKddlG729KFqakVf2w0GPyGPFCONRDAR5wjQarN9Bt8I8YbHwZQz_M1hztlnv-Lmsk1jBmech9ujD9-lTMBmSfFFbHcqquev119V7sneI-zxBZLf8C0pIDkaXf1t8y6Xr8CUJDMdlWLslCf3pBCNIOy65_TyGvy4Z4AJryTPBarNBPwOkNAtjCfZ4BDc2KqUZM5QN_VK4foP64sVzUL6mSr0Gh7lQJIL5b1qIpJxjxyQ
User-Agent: FxVersion/4.7.3324.0 OSName/Windows OSVersion/6.2.9200.0 Microsoft.Azure.KeyVault.KeyVaultClient/3.0.3.0
Content-Type: application/json; charset=utf-8
Host: putreqexample.vault.azure.net
Content-Length: 31

{
   "value": "m*gBJ7$Zuoz)"
}
```

El encabezado de "autorización" es el token de acceso necesario con cada llamada a Key Vault para operaciones del plano de datos. Si falta el encabezado, la respuesta debe ser 401.

### <a name="the-token-lacks-the-correct-resource-associated-with-it"></a>Al token le falta el recurso correcto asociado a él. 

Al solicitarse un token de acceso del punto de conexión de Azure para OAUTH, un parámetro llamado "recurso" es obligatorio. El valor es importante para el proveedor de tokens, ya que determina el ámbito del token para su uso previsto. El recurso para que **todos** los tokens obtengan acceso a una instancia de Key Vault es *https:\//vault.keyvault.net* (sin barra diagonal final).

### <a name="the-token-is-expired"></a>El token ha expirado

Los tokens están codificados en base64 y los valores se pueden descodificar en sitios web como [http://jwt.calebb.net](http://jwt.calebb.net). Este es el token anterior descodificado:

```
    {
 typ: "JWT",
 alg: "RS256",
 x5t: "nbCwW11w3XkB-xUaXwKRSLjMHGQ",
 kid: "nbCwW11w3XkB-xUaXwKRSLjMHGQ"
}.
{
 aud: "https://vault.azure.net",
 iss: "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
 iat: 1548697513,
 nbf: 1548697513,
 exp: 1548701413,
 aio: "42JgYHh85jiPgdqyfTFdNStv7le/BAA=",
 appid: "fad7d5b3-69d6-4b48-9259-8d12124e1cf1",
 appidacr: "1",
 idp: "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
 oid: "3975aeed-7d08-453b-b6f4-445f32698091",
 sub: "3975aeed-7d08-453b-b6f4-445f32698091",
 tid: "72f988bf-86f1-41af-91ab-2d7cd011db47",
 uti: "2-grhRkeIk6BeY-Ln42mAA",
 ver: "1.0"
}.
[signature]
```

Podemos ver muchas partes importantes en este token:

- aud (audiencia): recurso del token. Tenga en cuenta que es `https://vault.azure.net`. Este token NO funcionará para ningún recurso que no coincida de forma explícita con este valor, como grafo.
- iat (emitido a las): número de tics desde el inicio de la época en que se emitió el token.
- nbf (no antes de): número de tics desde el inicio de la época en que se valida este token.
- exp (expiración): número de tics desde el inicio de la época en que expira este token.
- appid (id. de la aplicación): GUID del identificador de aplicación que realiza esta solicitud.
- tid (id. de inquilino): GUID del identificador de inquilino de la entidad de seguridad que realiza esta solicitud.

Es importante que se identifiquen correctamente todos los valores en el token para que funcione la solicitud. Si todo es correcto, la solicitud no dará como resultado 401.

### <a name="troubleshooting-401"></a>Solución del 401

Los 401 deben investigarse desde la perspectiva de la generación de tokens, antes de que se realice la solicitud al almacén de claves. En general, el código se usa para solicitar el token. Una vez que se recibe el token, se pasa a la solicitud de Key Vault. Si el código se ejecuta localmente, puede usar Fiddler para capturar la solicitud o respuesta a `https://login.microsoftonline.com`. Una solicitud tiene este aspecto:

``` 
POST https://login.microsoftonline.com/<key vault tenant ID>/oauth2/token HTTP/1.1
Accept: application/json
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Host: login.microsoftonline.com
Content-Length: 192

resource=https%3A%2F%2Fvault.azure.net&client_id=<registered-app-ID>&client_secret=<registered-app-secret>&client_info=1&grant_type=client_credentials
```

La siguiente información proporcionada por el usuario debe ser correcta:

- Id. de inquilino del almacén de claves
- Valor de recurso establecido en https%3A%2F%2Fvault.azure.net (dirección URL codificada)
- Id. de cliente
- Secreto del cliente

Asegúrese de que el resto de la solicitud es casi idéntico.

Si solo puede obtener el token de acceso de la respuesta, puede descodificarlo (como se muestra anteriormente) para garantizar el id. de inquilino, el id. de cliente (id. de la aplicación) y el recurso.

## <a name="http-403-insufficient-permissions"></a>HTTP 403: Permisos insuficientes

HTTP 403 significa que la solicitud se autenticó (conoce la identidad solicitante), pero la identidad n tiene permiso para obtener acceso al recurso solicitado. Hay dos causas:

- No hay ninguna directiva de acceso para la identidad.
- La dirección IP del recurso solicitante no está en la lista de permitidos en la configuración de firewall del almacén de claves.

HTTP 403 suele producirse si la aplicación del cliente no usa el id. de cliente que el cliente piensa que es. Normalmente, esto significa que las directivas de acceso no se han configurado correctamente para la identidad de llamada real.

### <a name="troubleshooting-403"></a>Solución del 403

Primero, active el registro. Para obtener instrucciones sobre cómo hacerlo, consulte [Registro de Azure Key Vault](logging.md).

Una vez activado el registro, puede determinar si el 403 se debe a la directiva de acceso o a la directiva de firewall.

#### <a name="error-due-to-firewall-policy"></a>Error por la directiva de firewall

"La dirección de cliente (00.00.00.00) no está autorizada y el autor de llamada no es un servicio de confianza"

Hay una lista limitada de "servicios de confianza de Azure". Los sitios web de Azure **no** son un servicio de Azure de confianza. Para obtener más información, consulte la entrada de blog sobre el [acceso del firewall de Key Vault por parte de Azure App Services](https://azidentity.azurewebsites.net/post/2019/01/03/key-vault-firewall-access-by-azure-app-services).

Debe agregar la dirección IP del sitio web de Azure a Key Vault para que funcione.

Si se debe a la directiva de acceso: busque el id. de objeto de la solicitud y asegúrese de que este coincide con el objeto al que el usuario intenta asignar la directiva de acceso. A menudo habrá varios objetos en ADD con el mismo nombre, de modo que elegir el correcto es muy importante. Al eliminar y volver a agregar la directiva de acceso, es posible ver si existen varios objetos con el mismo nombre.

Además, la mayoría de las directivas de acceso no requieren el uso de la "aplicación autorizada" como se muestra en el portal. Las aplicaciones autorizadas se usan en escenarios de autenticación "en nombre de", que son poco comunes. 


## <a name="http-429-too-many-requests"></a>HTTP 429: Demasiadas solicitudes

La limitación se produce si el número de solicitudes supera el valor máximo indicado para el período de tiempo. Si se produce la limitación, la respuesta de Key Vault será HTTP 429. Hay valores máximos indicados para tipos de solicitudes realizadas. Por ejemplo: la creación de una clave HSM de 2048 bits es de cinco solicitudes por 10 segundos, pero todas las demás transacciones HSM tienen un límite de 1000 solicitudes en 10 segundos. Por lo tanto, es importante entender qué tipos de llamadas se realizan al determinar la causa de la limitación.
En general, las solicitudes a Key Vault están limitadas a 2000 solicitudes en 10 segundos. Las excepciones son operaciones clave, como se documenta en [Límites de servicio de Key Vault](service-limits.md)

### <a name="troubleshooting-429"></a>Solución del 429
La limitación se soluciona mediante estas técnicas:

- Reduzca el número de solicitudes realizadas a Key Vault determinando si hay patrones para un recurso solicitado e intentando almacenarlos en memoria caché en la aplicación que efectúa la llamada. 

- Cuando se produzca la limitación de Key Vault, adapte el código solicitante a fin de usar un retroceso exponencial para el reintento. El algoritmo se explica aquí: [Cómo limitar su aplicación](overview-throttling.md#how-to-throttle-your-app-in-response-to-service-limits)

- Si el número de solicitudes no se puede reducir mediante el almacenamiento en memoria caché y el retroceso cronometrado no funciona, considere la posibilidad de dividir las claves en varias instancias de Key Vault. El límite de servicio de una sola suscripción es cinco veces el límite individual de Key Vault. Si se usan más de cinco instancias de Key Vault, se debe considerar la utilización de varias suscripciones. 

Aquí encontrará instrucciones detalladas, incluida la solicitud de aumento de los límites: [Guía de las limitaciones de Key Vault](overview-throttling.md)


