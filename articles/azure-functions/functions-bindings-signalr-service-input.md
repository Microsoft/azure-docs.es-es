---
title: Enlace de entrada de SignalR Service para Azure Functions
description: Aprenda a devolver una dirección URL de punto de conexión de servicio de SignalR y un token de acceso en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 02/20/2020
ms.author: cshoe
ms.openlocfilehash: 326115a2a6cf29fcf211cdbd918edd0994fe45ed
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212115"
---
# <a name="signalr-service-input-binding-for-azure-functions"></a>Enlace de entrada de SignalR Service para Azure Functions

Para que un cliente pueda conectarse a Azure SignalR Service, antes debe recuperar la dirección URL del punto de conexión del servicio y un token de acceso válido. El enlace de entrada *SignalRConnectionInfo* genera la dirección URL del punto de conexión de SignalR Service y un token válido que se usan para conectarse al servicio. Dado que el token tiene una temporal y que se puede usar para autenticar un usuario concreto en una conexión, no debe almacenarlo en la caché ni compartirlo con los clientes. Los clientes pueden usar un desencadenador HTTP que use este enlace para recuperar la información de conexión.

Para más información sobre cómo se utiliza este enlace para crear una función "negotiate" que pueda consumir un SDK de cliente de SignalR, consulte el [artículo Desarrollo y configuración de Azure Functions](../azure-signalr/signalr-concept-serverless-development-config.md) en la documentación sobre conceptos de SignalR Service.

Para obtener información sobre los detalles de instalación y configuración, vea la [información general](functions-bindings-signalr-service.md).

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el siguiente ejemplo se muestra una [función de C#](functions-dotnet-class-library.md) que adquiere la información de la conexión de SignalR mediante el enlace de entrada y la devuelve a través de HTTP.

```cs
[FunctionName("negotiate")]
public static SignalRConnectionInfo Negotiate(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req,
    [SignalRConnectionInfo(HubName = "chat")]SignalRConnectionInfo connectionInfo)
{
    return connectionInfo;
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de entrada de la información de la conexión de SignalR en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace para devolver la información de conexión.

Estos son los datos de enlace del archivo *function.json*:

Function.json de ejemplo:

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Este es el código de script de C#:

```cs
#r "Microsoft.Azure.WebJobs.Extensions.SignalRService"
using Microsoft.Azure.WebJobs.Extensions.SignalRService;

public static SignalRConnectionInfo Run(HttpRequest req, SignalRConnectionInfo connectionInfo)
{
    return connectionInfo;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de entrada de la información de la conexión de SignalR en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace para devolver la información de la conexión.

Estos son los datos de enlace del archivo *function.json*:

Function.json de ejemplo:

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Este es el código de JavaScript:

```javascript
module.exports = async function (context, req, connectionInfo) {
    context.res.body = connectionInfo;
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de entrada de la información de la conexión de SignalR en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa el enlace para devolver la información de conexión.

Estos son los datos de enlace del archivo *function.json*:

Function.json de ejemplo:

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Este es el código de Python:

```python
def main(req: func.HttpRequest, connectionInfoJson: str) -> func.HttpResponse:
    return func.HttpResponse(
        connectionInfoJson,
        status_code=200,
        headers={
            'Content-type': 'application/json'
        }
    )
```

# <a name="java"></a>[Java](#tab/java)

En el siguiente ejemplo se muestra una [función de Java](functions-reference-java.md) que adquiere la información de la conexión de SignalR mediante el enlace de entrada y la devuelve a través de HTTP.

```java
@FunctionName("negotiate")
public SignalRConnectionInfo negotiate(
        @HttpTrigger(
            name = "req",
            methods = { HttpMethod.POST },
            authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> req,
        @SignalRConnectionInfoInput(
            name = "connectionInfo",
            hubName = "chat") SignalRConnectionInfo connectionInfo) {
    return connectionInfo;
}
```

---

## <a name="authenticated-tokens"></a>Tokens autenticados

Si la función la desencadena un cliente autenticado, puede agregar una notificación del identificador de usuario al token generado. Puede agregar fácilmente la autenticación a una aplicación de función mediante [Autenticación de App Service](../app-service/overview-authentication-authorization.md).

Autenticación de App Service establece encabezados HTTP denominados `x-ms-client-principal-id` y `x-ms-client-principal-name` que contienen el identificador y el nombre de la entidad de seguridad de cliente del usuario autenticado, respectivamente.

# <a name="c"></a>[C#](#tab/csharp)

Puede establecer la propiedad `UserId` del enlace en el valor de cualquier encabezado mediante una [expresión de enlace](./functions-bindings-expressions-patterns.md): `{headers.x-ms-client-principal-id}` o `{headers.x-ms-client-principal-name}`.

```cs
[FunctionName("negotiate")]
public static SignalRConnectionInfo Negotiate(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req, 
    [SignalRConnectionInfo
        (HubName = "chat", UserId = "{headers.x-ms-client-principal-id}")]
        SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier claim set to the authenticated user
    return connectionInfo;
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Puede establecer la propiedad `userId` del enlace en el valor de cualquier encabezado mediante una [expresión de enlace](./functions-bindings-expressions-patterns.md): `{headers.x-ms-client-principal-id}` o `{headers.x-ms-client-principal-name}`.

Function.json de ejemplo:

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Este es el código de script de C#:

```cs
#r "Microsoft.Azure.WebJobs.Extensions.SignalRService"
using Microsoft.Azure.WebJobs.Extensions.SignalRService;

public static SignalRConnectionInfo Run(HttpRequest req, SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier
    // claim set to the authenticated user
    return connectionInfo;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Puede establecer la propiedad `userId` del enlace en el valor de cualquier encabezado mediante una [expresión de enlace](./functions-bindings-expressions-patterns.md): `{headers.x-ms-client-principal-id}` o `{headers.x-ms-client-principal-name}`.

Function.json de ejemplo:

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Este es el código de JavaScript:

```javascript
module.exports = async function (context, req, connectionInfo) {
    // connectionInfo contains an access key token with a name identifier
    // claim set to the authenticated user
    context.res.body = connectionInfo;
};
```

# <a name="python"></a>[Python](#tab/python)

Puede establecer la propiedad `userId` del enlace en el valor de cualquier encabezado mediante una [expresión de enlace](./functions-bindings-expressions-patterns.md): `{headers.x-ms-client-principal-id}` o `{headers.x-ms-client-principal-name}`.

Function.json de ejemplo:

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Este es el código de Python:

```python
def main(req: func.HttpRequest, connectionInfoJson: str) -> func.HttpResponse:
    # connectionInfo contains an access key token with a name identifier
    # claim set to the authenticated user
    return func.HttpResponse(
        connectionInfoJson,
        status_code=200,
        headers={
            'Content-type': 'application/json'
        }
    )
```

# <a name="java"></a>[Java](#tab/java)

Puede establecer la propiedad `userId` del enlace en el valor de cualquier encabezado mediante una [expresión de enlace](./functions-bindings-expressions-patterns.md): `{headers.x-ms-client-principal-id}` o `{headers.x-ms-client-principal-name}`.

```java
@FunctionName("negotiate")
public SignalRConnectionInfo negotiate(
        @HttpTrigger(
            name = "req",
            methods = { HttpMethod.POST },
            authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> req,
        @SignalRConnectionInfoInput(
            name = "connectionInfo",
            hubName = "chat",
            userId = "{headers.x-ms-client-principal-id}") SignalRConnectionInfo connectionInfo) {
    return connectionInfo;
}
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Envío de mensajes de SignalR Service (enlace de salida)](./functions-bindings-signalr-service-output.md) 
