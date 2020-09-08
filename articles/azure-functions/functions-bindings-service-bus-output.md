---
title: Enlaces de salida de Azure Service Bus para Azure Functions
description: Obtenga información sobre cómo enviar mensajes de Azure Service Bus desde Azure Functions.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 3fc8f205bff52fad6e55b7aa6692ec80ae5e954a
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212163"
---
# <a name="azure-service-bus-output-binding-for-azure-functions"></a>Enlace de salida de Azure Service Bus para Azure Functions

Use el enlace de salida de Azure Service Bus para enviar mensajes de cola o tema.

Para obtener información sobre los detalles de instalación y configuración, consulte [Introducción](functions-bindings-service-bus-output.md).

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que envía un mensaje de cola de Service Bus:

```cs
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue", Connection = "ServiceBusConnection")]
public static string ServiceBusOutput([HttpTrigger] dynamic input, ILogger log)
{
    log.LogInformation($"C# function processed: {input.Text}");
    return input.Text;
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de salida de Service Bus en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función usa un desencadenador de temporizador para enviar un mensaje de cola cada 15 segundos.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

Este es el código de script de C# que crea un único mensaje:

```cs
public static void Run(TimerInfo myTimer, ILogger log, out string outputSbQueue)
{
    string message = $"Service Bus queue message created at: {DateTime.Now}";
    log.LogInformation(message); 
    outputSbQueue = message;
}
```

Este es el código de script de C# que crea varios mensajes:

```cs
public static async Task Run(TimerInfo myTimer, ILogger log, IAsyncCollector<string> outputSbQueue)
{
    string message = $"Service Bus queue messages created at: {DateTime.Now}";
    log.LogInformation(message); 
    await outputSbQueue.AddAsync("1 " + message);
    await outputSbQueue.AddAsync("2 " + message);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de salida de Service Bus en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función usa un desencadenador de temporizador para enviar un mensaje de cola cada 15 segundos.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

Este es el código de script de JavaScript que crea un único mensaje:

```javascript
module.exports = function (context, myTimer) {
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueue = message;
    context.done();
};
```

Este es el código de script de JavaScript que crea varios mensajes:

```javascript
module.exports = function (context, myTimer) {
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueue = [];
    context.bindings.outputSbQueue.push("1 " + message);
    context.bindings.outputSbQueue.push("2 " + message);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra cómo escribir en una cola de Service Bus en Python.

Una definición de enlace de Service Bus se define en *function.json*, donde *type* se establece en `serviceBus`.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "serviceBus",
      "direction": "out",
      "connection": "AzureServiceBusConnectionString",
      "name": "msg",
      "queueName": "outqueue"
    }
  ]
}
```

En *_\_init_\_.py*, puede escribir un mensaje en la cola pasando un valor al método `set`.

```python
import azure.functions as func

def main(req: func.HttpRequest, msg: func.Out[str]) -> func.HttpResponse:

    input_msg = req.params.get('message')

    msg.set(input_msg)

    return 'OK'
```

# <a name="java"></a>[Java](#tab/java)

En el ejemplo siguiente se muestra una función de Java que envía un mensaje a una cola de Service Bus `myqueue` cuando una solicitud HTTP la desencadena.

```java
@FunctionName("httpToServiceBusQueue")
@ServiceBusQueueOutput(name = "message", queueName = "myqueue", connection = "AzureServiceBusConnection")
public String pushToQueue(
  @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
  final String message,
  @HttpOutput(name = "response") final OutputBinding<T> result ) {
      result.setValue(message + " has been sent.");
      return message;
 }
```

 En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@QueueOutput` en los parámetros de función cuyo valor se escribiría en una cola de Service Bus.  El parámetro type debe ser `OutputBinding<T>`, donde T es cualquier tipo nativo de Java de un POJO.

Las funciones de Java también pueden escribir en un tema de Service Bus. En el ejemplo siguiente se usa la anotación `@ServiceBusTopicOutput` para describir la configuración del enlace de salida. 

```java
@FunctionName("sbtopicsend")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> request,
            @ServiceBusTopicOutput(name = "message", topicName = "mytopicname", subscriptionName = "mysubscription", connection = "ServiceBusConnection") OutputBinding<String> message,
            final ExecutionContext context) {
        
        String name = request.getBody().orElse("Azure Functions");

        message.setValue(name);
        return request.createResponseBuilder(HttpStatus.OK).body("Hello, " + name).build();
        
    }
```

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [ServiceBusAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAttribute.cs).

El constructor del atributo toma el nombre de la cola o del tema y la suscripción. También puede especificar los derechos de acceso de la conexión. En la sección [Salida: configuración](#configuration) se explica cómo elegir la configuración de derechos de acceso. Este es un ejemplo que muestra el atributo aplicado al valor devuelto de la función:

```csharp
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue")]
public static string Run([HttpTrigger] dynamic input, ILogger log)
{
    ...
}
```

Puede establecer la propiedad `Connection` para especificar el nombre de una configuración de aplicación que contiene la cadena de conexión de Service Bus que se va a usar, tal como se muestra en el ejemplo siguiente:

```csharp
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue", Connection = "ServiceBusConnection")]
public static string Run([HttpTrigger] dynamic input, ILogger log)
{
    ...
}
```

Para obtener un ejemplo completo, consulte [Salida: ejemplo](#example).

Puede usar el atributo `ServiceBusAccount` para especificar la cuenta de Service Bus en el nivel de clase, método o parámetro.  Para obtener más información, consulte [Desencadenador: atributos](functions-bindings-service-bus-trigger.md#attributes-and-annotations).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

Las anotaciones `ServiceBusQueueOutput` y `ServiceBusTopicOutput` están disponibles para escribir un mensaje como salida de la función. El parámetro decorado con estas anotaciones se debe declarar como `OutputBinding<T>`, donde `T` es el tipo correspondiente al tipo del mensaje.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `ServiceBus`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Debe establecerse en "serviceBus". Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal.|
|**direction** | N/D | Debe establecerse en "out". Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal. |
|**name** | N/D | Nombre de la variable que representa el mensaje de cola o tema en el código de la función. Se establece en "$return" para hacer referencia al valor devuelto de la función. |
|**queueName**|**QueueName**|Nombre de la cola.  Se establece únicamente si se envían mensajes de cola, no de tema.
|**topicName**|**TopicName**|Nombre del tema. Se establece únicamente si se envían mensajes de tema, no de cola.|
|**connection**|**Connection**|Nombre de una configuración de aplicación que contiene la cadena de conexión de Service Bus que se usará para este enlace. Si el nombre de la configuración de aplicación comienza con "AzureWebJobs", puede especificar solo el resto del nombre. Por ejemplo, si establece `connection` en "MyServiceBus", el entorno de ejecución de Functions busca una configuración de aplicación llamada "AzureWebJobsMyServiceBus". Si deja el valor de `connection` vacío, el entorno de ejecución de Functions usa la cadena de conexión de Service Bus predeterminada en la configuración de aplicación que se denomina "AzureWebJobsServiceBus".<br><br>Para obtener la cadena de conexión, siga los pasos mostrados en [Obtención de las credenciales de administración](../service-bus-messaging/service-bus-quickstart-portal.md#get-the-connection-string). La cadena de conexión debe ser para un espacio de nombres de Service Bus y no estar limitada a una cola o un tema concretos.|
|**accessRights** (solo v1)|**Acceder**|Derechos de acceso para la cadena de conexión. Los valores disponibles son `manage` y `listen`. El valor predeterminado es `manage`, lo que indica que `connection` tiene el permiso **Administrar**. Si usa una cadena de conexión que no tiene el permiso **Administrar**, establezca `accessRights` en "listen". De lo contrario, el runtime de Functions puede intentar realizar operaciones que requieran derechos de administración y no conseguirlo. En la versión 2.x y posteriores de Azure Functions, esta propiedad no está disponible porque la versión más reciente del SDK de Service Bus no admite operaciones de administración.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

En Azure Functions 1.x, el entorno de ejecución crea la cola si no existe y se estableció `accessRights` en `manage`. En la versión 2.x y posteriores de Functions, ya debe existir la cola o el tema. Si especifica una cola o un tema que no existe, la función generará un error. 

# <a name="c"></a>[C#](#tab/csharp)

Puede usar los siguientes tipos de parámetro para el enlace de salida:

* `out T paramName` - `T` puede ser cualquier tipo que se pueda serializar con JSON. Si el valor del parámetro es null cuando finaliza la función, Functions crea el mensaje con un objeto null.
* `out string`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje.
* `out byte[]`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje.
* `out BrokeredMessage`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje (en el caso de Functions 1.x)
* `out Message`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje (en el caso de Functions 2.x y versiones posteriores)
* `ICollector<T>` o `IAsyncCollector<T>` (para métodos asincrónicos): para crear varios mensajes. Se crea un mensaje al llamar al método `Add` .

Cuando trabaje con funciones de C#:

* Las funciones asincrónicas necesitan un valor devuelto o `IAsyncCollector`, en lugar de un parámetro `out`.

* Para tener acceso al identificador de sesión, enlace a un tipo [`Message`](/dotnet/api/microsoft.azure.servicebus.message) y use la propiedad `sessionId`.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Puede usar los siguientes tipos de parámetro para el enlace de salida:

* `out T paramName` - `T` puede ser cualquier tipo que se pueda serializar con JSON. Si el valor del parámetro es null cuando finaliza la función, Functions crea el mensaje con un objeto null.
* `out string`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje.
* `out byte[]`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje.
* `out BrokeredMessage`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje (en el caso de Functions 1.x)
* `out Message`: si el valor del parámetro es null cuando finaliza la función, Functions no crea ningún mensaje (en el caso de Functions 2.x y versiones posteriores)
* `ICollector<T>` o `IAsyncCollector<T>`: para crear varios mensajes. Se crea un mensaje al llamar al método `Add` .

Cuando trabaje con funciones de C#:

* Las funciones asincrónicas necesitan un valor devuelto o `IAsyncCollector`, en lugar de un parámetro `out`.

* Para tener acceso al identificador de sesión, enlace a un tipo [`Message`](/dotnet/api/microsoft.azure.servicebus.message) y use la propiedad `sessionId`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Puede acceder a la cola o al tema mediante el uso de `context.bindings.<name from function.json>`. Puede asignar una cadena, una matriz de bytes o un objeto de JavaScript (deserializado en JSON) a `context.binding.<name>`.

# <a name="python"></a>[Python](#tab/python)

Use el [SDK de Azure Service Bus](../service-bus-messaging/index.yml) en lugar del enlace de salida integrado.

# <a name="java"></a>[Java](#tab/java)

Use el [SDK de Azure Service Bus](../service-bus-messaging/index.yml) en lugar del enlace de salida integrado.

---

## <a name="exceptions-and-return-codes"></a>Excepciones y códigos de retorno

| Enlace | Referencia |
|---|---|
| Azure Service Bus | [Códigos de error de Service Bus](../service-bus-messaging/service-bus-messaging-exceptions.md) |
| Azure Service Bus | [Límites de Service Bus](../service-bus-messaging/service-bus-quotas.md) |

<a name="host-json"></a>  

## <a name="hostjson-settings"></a>configuración de host.json

En esta sección se describen las opciones de configuración globales disponibles para este enlace en las versiones 2.x y posteriores. El siguiente archivo host.json de ejemplo contiene solo la configuración para este enlace. Para más información acerca de las opciones de configuración globales, consulte [Referencia de host.json para Azure Functions](functions-host-json.md).

> [!NOTE]
> Para obtener una referencia de host.json en Functions 1.x, consulte la [referencia de host.json para Azure Functions, versión 1.x](functions-host-json-v1.md).

```json
{
    "version": "2.0",
    "extensions": {
        "serviceBus": {
            "prefetchCount": 100,
            "messageHandlerOptions": {
                "autoComplete": true,
                "maxConcurrentCalls": 32,
                "maxAutoRenewDuration": "00:05:00"
            },
            "sessionHandlerOptions": {
                "autoComplete": false,
                "messageWaitTimeout": "00:00:30",
                "maxAutoRenewDuration": "00:55:00",
                "maxConcurrentSessions": 16
            }
        }
    }
}
```

Si `isSessionsEnabled` se ha establecido en `true`, se respetará `sessionHandlerOptions`.  Si `isSessionsEnabled` se ha establecido en `false`, se respetará `messageHandlerOptions`.

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|prefetchCount|0|Obtiene o establece el número de mensajes que el destinatario del mensaje puede solicitar simultáneamente.|
|maxAutoRenewDuration|00:05:00|Duración máxima dentro de la cual el bloqueo de mensajes se renovará automáticamente.|
|autoComplete|true|Si el desencadenador debe llamar automáticamente a Complete después del procesamiento o si el código de la función llamará manualmente a Complete.<br><br>La configuración en `false` solo se admite en C#.<br><br>Si se establece en `true`, el desencadenador completa automáticamente el mensaje si la ejecución de la función se completa correctamente y abandona el mensaje en caso contrario.<br><br>Cuando se establece en `false`, usted es responsable de llamar a los métodos [MessageReceiver](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver?view=azure-dotnet) para completar, abandonar o cerrar el mensaje. Si se produce una excepción (y no se llama a ninguno de los métodos `MessageReceiver`), se mantiene el bloqueo. Una vez que el bloqueo expira, el mensaje se vuelve a poner en cola con la `DeliveryCount` incrementada y el bloqueo se renueva automáticamente.<br><br>En las funciones que no son C#, las excepciones en la función dan como resultado las llamadas en tiempo de ejecución `abandonAsync` en segundo plano. Si no se produce ninguna excepción, se llama a `completeAsync` en segundo plano. |
|maxConcurrentCalls|16|Número máximo de llamadas simultáneas a la devolución de llamada que el bombeo de mensajes debe iniciar por instancia con escala. De forma predeterminada, el entorno de ejecución de Functions procesa simultáneamente varios mensajes.|
|maxConcurrentSessions|2000|Número máximo de sesiones que se puede administrar simultáneamente por instancia con escala.|

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando se crea un mensaje de cola o tema de Service Bus (Desencadenador)](./functions-bindings-service-bus-trigger.md)
