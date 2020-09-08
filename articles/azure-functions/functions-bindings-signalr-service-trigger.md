---
title: Enlace del desencadenador de SignalR Service para Azure Functions
description: Obtenga información sobre cómo enviar mensajes de SignalR Service desde Azure Functions.
author: chenyl
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 05/11/2020
ms.author: chenyl
ms.openlocfilehash: e2651afbcdc3bae71bb531aa0e821f83264c295d
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212597"
---
# <a name="signalr-service-trigger-binding-for-azure-functions"></a>Enlace del desencadenador de SignalR Service para Azure Functions

Use el enlace del desencadenador de *SignalR* para responder a mensajes enviados desde Azure SignalR Service. Cuando se desencadena la función, los mensajes pasados a la función se analizan como un objeto JSON.

Para obtener información sobre los detalles de instalación y configuración, vea la [información general](functions-bindings-signalr-service.md).

## <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra una función que recibe un mensaje mediante el enlace del desencadenador y registra el mensaje.

# <a name="c"></a>[C#](#tab/csharp)

El enlace del desencadenador de SignalR Service para C# tiene dos modelos de programación. Modelo basado en clases y modelo tradicional. El modelo basado en clases puede ofrecer una experiencia coherente de programación del lado servidor de SignalR. El modelo tradicional proporciona más flexibilidad con otros enlaces de función.

### <a name="with-class-based-model"></a>Modelo basado en clases

Vea [Modelo basado en clases](../azure-signalr/signalr-concept-serverless-development-config.md#class-based-model) para más información.

```cs
public class SignalRTestHub : ServerlessHub
{
    [FunctionName("SignalRTest")]
    public async Task SendMessage([SignalRTrigger]InvocationContext invocationContext, string message, ILogger logger)
    {
        logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
    }
}
```

### <a name="with-traditional-model"></a>Modelo tradicional

El modelo tradicional obedece la convención de Azure Functions desarrollada por C#. Si no está familiarizado con ella, puede consultar diversos [documentos](./functions-dotnet-class-library.md).

```cs
[FunctionName("SignalRTest")]
public static async Task Run([SignalRTrigger("SignalRTest", "messages", "SendMessage", parameterNames: new string[] {"message"})]InvocationContext invocationContext, string message, ILogger logger)
{
    logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
}
```

#### <a name="use-attribute-signalrparameter-to-simplify-parameternames"></a>Uso del atributo `[SignalRParameter]` para simplificar `ParameterNames`

Dado que es complicado usar `ParameterNames`, se proporciona `SignalRParameter` para lograr el mismo propósito.

```cs
[FunctionName("SignalRTest")]
public static async Task Run([SignalRTrigger("SignalRTest", "messages", "SendMessage")]InvocationContext invocationContext, [SignalRParameter]string message, ILogger logger)
{
    logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Estos son los datos de enlace del archivo *function.json*:

Function.json de ejemplo:

```json
{
    "type": "signalRTrigger",
    "name": "invocation",
    "hubName": "SignalRTest",
    "category": "messages",
    "event": "SendMessage",
    "parameterNames": [
        "message"
    ],
    "direction": "in"
}
```

Este es el código de script de C#:

```cs
#r "Microsoft.Azure.WebJobs.Extensions.SignalRService"
using System;
using Microsoft.Azure.WebJobs.Extensions.SignalRService;
using Microsoft.Extensions.Logging;

public static void Run(InvocationContext invocation, string message, ILogger logger)
{
    logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Estos son los datos de enlace del archivo *function.json*:

Function.json de ejemplo:

```json
{
    "type": "signalRTrigger",
    "name": "invocation",
    "hubName": "SignalRTest",
    "category": "messages",
    "event": "SendMessage",
    "parameterNames": [
        "message"
    ],
    "direction": "in"
}
```

Este es el código de JavaScript:

```javascript
module.exports = function (context, invocation) {
    context.log(`Receive ${context.bindingData.message} from ${invocation.ConnectionId}.`)
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

Estos son los datos de enlace del archivo *function.json*:

Function.json de ejemplo:

```json
{
    "type": "signalRTrigger",
    "name": "invocation",
    "hubName": "SignalRTest",
    "category": "messages",
    "event": "SendMessage",
    "parameterNames": [
        "message"
    ],
    "direction": "in"
}
```

Este es el código de Python:

```python
import logging
import json
import azure.functions as func

def main(invocation) -> None:
    invocation_json = json.loads(invocation)
    logging.info("Receive {0} from {1}".format(invocation_json['Arguments'][0], invocation_json['ConnectionId']))
```

---

## <a name="configuration"></a>Configuración

### <a name="signalrtrigger"></a>SignalRTrigger

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `SignalRTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type**| N/D | Se debe establecer en `SignalRTrigger`.|
|**direction**| N/D | Se debe establecer en `in`.|
|**name**| N/D | Nombre de variable utilizado en el código de función para el objeto de contexto de invocación del desencadenador. |
|**hubName**|**HubName**| Este valor debe establecerse en el nombre del centro de SignalR para la función que se va a desencadenar.|
|**category**|**Categoría**| Este valor debe establecerse como la categoría de mensajes para la función que se va a desencadenar. La categoría puede tener uno de estos valores: <ul><li>**conexiones**: que incluye eventos *conectados* y *desconectados*.</li><li>**mensajes**: que incluye todos los demás eventos, excepto los de la categoría *conexiones*.</li></ul> |
|**event**|**Evento**| Este valor debe establecerse como el evento de mensajes para la función que se va a desencadenar. En la categoría *mensajes*, el evento es el *objetivo* en el [mensaje de invocación](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocation-message-encoding) que envían los clientes. En la categoría *conexiones*, solo se usan *conectados* y *desconectados*. |
|**parameterNames**|**parameterNames**| (Opcional) Lista de nombres que se enlaza a los parámetros. |
|**connectionStringSetting**|**ConnectionStringSetting**| El nombre de la configuración de la aplicación que contiene la cadena de conexión de SignalR Service (el valor predeterminado es "AzureSignalRConnectionString") |

## <a name="payload"></a>Carga

El tipo de entrada del desencadenador se declara como `InvocationContext` o como un tipo personalizado. Si elige `InvocationContext`, obtiene acceso completo al contenido de la solicitud. En un tipo personalizado, el runtime intenta analizar el cuerpo de la solicitud JSON para establecer las propiedades del objeto.

### <a name="invocationcontext"></a>InvocationContext

InvocationContext incluye todo el contenido del mensaje enviado desde SignalR Service.

|Propiedad en InvocationContext | Descripción|
|------------------------------|------------|
|Argumentos| Disponible para la categoría *mensajes*. Contiene *argumentos* en el [mensaje de invocación](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocation-message-encoding).|
|Error| Disponible para evento *desconectado*. Puede estar vacío si la conexión se cerró sin errores o puede contener los mensajes de error.|
|Hub| El nombre del centro al que pertenece el mensaje.|
|Category| Categoría del mensaje.|
|Evento| Evento del mensaje.|
|ConnectionId| El identificador de conexión del cliente que envía el mensaje.|
|UserId| La identidad del usuario del cliente que envía el mensaje.|
|encabezados| los encabezados de la solicitud.|
|Consultar| La consulta de la solicitud cuando los clientes se conectan al servicio.|
|Notificaciones| Las notificaciones del cliente.|

## <a name="using-parameternames"></a>Usar `ParameterNames`

La propiedad `ParameterNames` en `SignalRTrigger` permite enlazar argumentos de mensajes de invocación con los parámetros de funciones. Esto proporciona una manera más práctica de acceder a los argumentos de `InvocationContext`.

Supongamos que tiene un cliente de SignalR de JavaScript que intenta invocar el método `broadcast` en Azure Functions con dos argumentos.

```javascript
await connection.invoke("broadcast", message1, message2);
```

Puede acceder a estos dos argumentos desde el parámetro, así como asignar un tipo de parámetro para ellos mediante `ParameterNames`.

### <a name="remarks"></a>Observaciones

Al enlazar parámetros, el orden es importante. Si está utilizando `ParameterNames`, el orden de `ParameterNames` coincide con el orden de los argumentos que se invocan en el cliente. Si usa el atributo `[SignalRParameter]` en C#, el orden de los argumentos en los métodos de Azure Functions coincide con el orden de los argumentos en los clientes.

`ParameterNames` y el atributo `[SignalRParameter]` **no se pueden** usar al mismo tiempo o se producirá una excepción.

## <a name="send-messages-to-signalr-service-trigger-binding"></a>Envío de mensajes al enlace del desencadenador de SignalR Service

Azure Functions genera una dirección URL para el enlace del desencadenador de SignalR Service con el formato siguiente:

```http
https://<APP_NAME>.azurewebsites.net/runtime/webhooks/signalr?code=<API_KEY>
```

Azure Functions genera la `API_KEY`. La `API_KEY` se puede obtener de Azure Portal mientras usa el enlace del desencadenador de SignalR Service.
:::image type="content" source="media/functions-bindings-signalr-service/signalr-keys.png" alt-text="Clave de API":::

Debe establecer esta dirección URL en `UrlTemplate` en la configuración ascendente de SignalR Service.

## <a name="next-steps"></a>Pasos siguientes

* [Desarrollo y configuración de Azure Functions con Azure SignalR Service](../azure-signalr/signalr-concept-serverless-development-config.md)
* [Ejemplo de enlace del desencadenador de SignalR Service](https://github.com/Azure/azure-functions-signalrservice-extension/tree/dev/samples/bidirectional-chat)
