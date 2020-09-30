---
title: Autenticación en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre las distintas formas en que una aplicación o un servicio pueden autenticarse en Communication Services.
author: matthewrobertson
manager: jken
services: azure-communication-services
ms.author: marobert
ms.date: 07/24/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: 928737608ae3e3e44b352724713a284ff9a45da9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932324"
---
# <a name="authenticate-to-azure-communication-services"></a>Autenticación en Azure Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

En este artículo se proporciona información sobre la autenticación de clientes en Azure Communication Services mediante *claves de acceso* y *tokens de acceso de usuario*. Se debe autenticar cada interacción del cliente con Azure Communication Services.

En la tabla siguiente se describen las opciones de autenticación que son compatibles con las bibliotecas cliente de Azure Communication Services:

| Biblioteca de cliente | Clave de acceso    | Tokens de acceso de usuario |
| -------------- | ------------- | ------------------ |
| Administración | Compatible     | No compatible      |
| SMS            | Compatible     | No compatible      |
| Chat           | No compatible | Compatible          |
| Llamar        | No compatible | Compatible          |

Cada opción de autorización se describe brevemente a continuación:

- Autenticación con **clave de acceso** para operaciones de SMS y administración. La autenticación con clave de acceso es adecuada para las aplicaciones que se ejecutan en un entorno de servicio de confianza. Para autenticar con una clave de acceso, un cliente genera un [código de autenticación de método basado en hash (HMAC)](https://en.wikipedia.org/wiki/HMAC) e incluye en el encabezado `Authorization` de cada solicitud HTTP. Para obtener más información, vea [Autenticación con una clave de acceso](#authenticate-with-an-access-key).
- Autenticación con **token de acceso de usuario** para chat y llamadas. Los tokens de acceso de usuario permiten que las aplicaciones cliente se autentiquen directamente en Azure Communication Services. Estos tokens se generan en un servicio de aprovisionamiento de token del lado servidor que cree. A continuación, se proporcionan a los dispositivos cliente que usan el token para inicializar las bibliotecas cliente de chat y llamadas. Para obtener más información, vea [Autenticación con un token de acceso de usuario](#authenticate-with-a-user-access-token).

## <a name="authenticate-with-an-access-key"></a>Autenticación con una clave de acceso

La autenticación con clave de acceso usa una clave secreta compartida para generar un HMAC para cada solicitud HTTP calculada mediante el algoritmo SHA256 y la envía en el encabezado `Authorization` mediante el esquema `HMAC-SHA256`.

```
Authorization: "HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=<hmac-sha256-signature>"
```

Las bibliotecas cliente de Azure Communication Services que usan la autenticación con clave de acceso se deben inicializar con la cadena de conexión del recurso. Si no está usando una biblioteca cliente, puede generar HMAC mediante programación con la clave de acceso del recurso. Para obtener más información sobre las cadenas de conexión, visite el [inicio rápido del aprovisionamiento de recursos](../quickstarts/create-communication-resource.md).

### <a name="sign-an-http-request"></a>Envío de una solicitud HTTP

Si no usa una biblioteca cliente para hacer solicitudes HTTP a las API REST de Azure Communication Services, deberá crear mediante programación HMAC para cada solicitud HTTP. En los pasos siguientes se describe cómo construir el encabezado de autorización:

1. Especifique la marca de tiempo de la hora universal coordinada (UTC) para la solicitud en el encabezado `x-ms-date` o en el encabezado `Date` HTTP estándar. El servicio lo valida para protegerse frente a determinados ataques de seguridad, incluidos los ataques de reproducción.
1. Aplique un algoritmo hash al cuerpo de la solicitud HTTP mediante el algoritmo SHA256 y, a continuación, páselo con la solicitud, a través del encabezado `x-ms-content-sha256`.
1. Construya la cadena que se va a firmar mediante la concatenación del verbo HTTP (por ejemplo, `GET` o `PUT`), la ruta de acceso de la solicitud HTTP y los valores de los encabezados HTTP `Date`, `Host` y `x-ms-content-sha256` en el formato siguiente:
    ```
    VERB + "\n"
    URLPathAndQuery + "\n"
    DateHeaderValue + ";" + HostHeaderValue + ";" + ContentHashHeaderValue
    ```
1. Genere una firma HMAC-256 de la cadena con codificación UTF-8 que creó en el paso anterior. A continuación, codifique los resultados como Base64. Tenga en cuenta que también debe descodificar en Base64 la clave de la cuenta de almacenamiento. Utilice el formato siguiente (mostrado como pseudocódigo):
    ```
    Signature=Base64(HMAC-SHA256(UTF8(StringToSign), Base64.decode(<your_azure_storage_account_shared_key>)))
    ```
1. Especifique el encabezado de autorización del modo siguiente:
    ```
    Authorization="HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=<hmac-sha256-signature>"  
    ```
    Donde `<hmac-sha256-signature>` es el HMAC calculado en el paso anterior.

## <a name="authenticate-with-a-user-access-token"></a>Autenticación con un token de acceso de usuario

Los tokens de acceso de usuario permiten que las aplicaciones cliente se autentiquen directamente en Azure Communication Services. Para ello, debe configurar un servicio de confianza que autentique a los usuarios de la aplicación y emita tokens de acceso de usuario con la biblioteca cliente de administración. Visite la documentación conceptual sobre la [arquitectura de cliente y servidor](./client-and-server-architecture.md) para obtener más información sobre nuestras consideraciones de arquitectura.

La clase `CommunicationClientCredential` contiene la lógica para proporcionar credenciales de token de acceso de usuario a las bibliotecas cliente y administrar su ciclo de vida.

### <a name="initialize-the-client-libraries"></a>Inicialización del objeto cliente

Para inicializar las bibliotecas cliente de Azure Communication Services que requieren la autenticación de token de acceso de usuario, primero debe crear una instancia de la clase `CommunicationClientCredential` y, a continuación, usarla para inicializar un cliente de API.

En los fragmentos de código siguientes se muestra cómo inicializar la biblioteca cliente de chat con un token de acceso de usuario:

#### <a name="c"></a>[C#](#tab/csharp)

```csharp
// user access tokens should be created by a trusted service using the Administration client library
var token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance
var userCredential = new CommunicationUserCredential(token);

// initialize the chat client library with the credential
var chatClient = new ChatClient(ENDPOINT_URL, userCredential);
```

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// user access tokens should be created by a trusted service using the Administration client library
const token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance with the AzureCommunicationUserCredential class
const userCredential = new AzureCommunicationUserCredential(token);

// initialize the chat client library with the credential
let chatClient = new ChatClient(ENDPOINT_URL, userCredential);
```

#### <a name="swift"></a>[Swift](#tab/swift)

```swift
// user access tokens should be created by a trusted service using the Administration client library
let token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance
let userCredential = try CommunicationUserCredential(token: token)

// initialize the chat client library with the credential
let chatClient = try CommunicationChatClient(credential: userCredential, endpoint: ENDPOINT_URL)
```

#### <a name="java"></a>[Java](#tab/java)

```java
// user access tokens should be created by a trusted service using the Administration client library
String token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance
CommunicationUserCredential userCredential = new CommunicationUserCredential(token);

// Initialize the chat client
final ChatClientBuilder builder = new ChatClientBuilder();
builder.endpoint(ENDPOINT_URL)
    .credential(userCredential)
    .httpClient(HTTP_CLIENT);
ChatClient chatClient = builder.buildClient();
```

---

### <a name="refreshing-user-access-tokens"></a>Actualización de tokens de acceso de usuario

Los tokens de acceso de usuario son credenciales de corta duración que deben volver a emitirse para evitar que los usuarios experimenten interrupciones del servicio. El constructor `CommunicationUserCredential` acepta una función de devolución de llamada de actualización que le permite actualizar los tokens de acceso de usuario antes de que expiren. Debe usar esta devolución de llamada para capturar un nuevo token de acceso de usuario desde el servicio de confianza.

#### <a name="c"></a>[C#](#tab/csharp)

```csharp
var userCredential = new CommunicationUserCredential(
    initialToken: token,
    refreshProactively: true,
    tokenRefresher: cancellationToken => fetchNewTokenForCurrentUser(cancellationToken)
);
```

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const userCredential = new AzureCommunicationUserCredential({
  tokenRefresher: async () => fetchNewTokenForCurrentUser(),
  refreshProactively: true,
  initialToken: token
});
```

#### <a name="swift"></a>[Swift](#tab/swift)

```swift
 let userCredential = try CommunicationUserCredential(initialToken: token, refreshProactively: true) { |completionHandler|
   let updatedToken = fetchTokenForCurrentUser()
   completionHandler(updatedToken, nil)
 }
```

#### <a name="java"></a>[Java](#tab/java)

```java
TokenRefresher tokenRefresher = new TokenRefresher() {
    @Override
    Future<String> getFetchTokenFuture() {
        return fetchNewTokenForCurrentUser();
    }
}

CommunicationUserCredential credential = new CommunicationUserCredential(tokenRefresher, token, true);
```
---

La opción `refreshProactively` le permite decidir cómo va a administrar el ciclo de vida del token. De forma predeterminada, cuando un token está obsoleto, la devolución de llamada bloqueará las solicitudes de API e intentará actualizarlas. Cuando `refreshProactively` se establece en `true`, la devolución de llamada se programa y se ejecuta de forma asincrónica antes de que expire el token.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de tokens de acceso de usuario](../quickstarts/access-tokens.md)

Para más información, consulte los siguientes artículos.
- [Información sobre la arquitectura de cliente y servidor](../concepts/client-and-server-architecture.md)
