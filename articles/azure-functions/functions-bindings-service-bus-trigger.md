---
title: Desencadenador de Azure Service Bus para Azure Functions
description: Aprenda a ejecutar una función de Azure cuando se crean mensajes de Azure Service Bus.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 72f12ece3c939ee8fbaecbec377d18c5bb3b0a49
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294618"
---
# <a name="azure-service-bus-trigger-for-azure-functions"></a>Desencadenador de Azure Service Bus para Azure Functions

Utilice el desencadenador de Service Bus para responder a mensajes de una cola o tema de Service Bus. A partir de la versión 3.1.0 de la extensión, puede desencadenar en una cola o un tema habilitados para la sesión.

Para obtener información sobre los detalles de instalación y configuración, vea la [información general](functions-bindings-service-bus-output.md).

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que lee [metadatos de mensaje](#message-metadata) y registra un mensaje de cola de Service Bus:

```cs
[FunctionName("ServiceBusQueueTriggerCSharp")]                    
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
    string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    ILogger log)
{
    log.LogInformation($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
    log.LogInformation($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.LogInformation($"DeliveryCount={deliveryCount}");
    log.LogInformation($"MessageId={messageId}");
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador de Service Bus en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función lee [metadatos de mensaje](#message-metadata) y registra un mensaje de cola de Service Bus.

Estos son los datos de enlace del archivo *function.json*:

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

Este es el código de script de C#:

```cs
using System;

public static void Run(string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    TraceWriter log)
{
    log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");

    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"DeliveryCount={deliveryCount}");
    log.Info($"MessageId={messageId}");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador de Service Bus en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función lee [metadatos de mensaje](#message-metadata) y registra un mensaje de cola de Service Bus. 

Estos son los datos de enlace del archivo *function.json*:

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

Este es el código del script de JavaScript:

```javascript
module.exports = function(context, myQueueItem) {
    context.log('Node.js ServiceBus queue trigger function processed message', myQueueItem);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('DeliveryCount =', context.bindingData.deliveryCount);
    context.log('MessageId =', context.bindingData.messageId);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra cómo leer un mensaje de la cola de Service Bus mediante un desencadenador.

Un enlace de Service Bus se define en *function.json*, donde *type* se establece en `serviceBusTrigger`.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "msg",
      "type": "serviceBusTrigger",
      "direction": "in",
      "queueName": "inputqueue",
      "connection": "AzureServiceBusConnectionString"
    }
  ]
}
```

El código de *_\_init_\_.py* declara un parámetro como `func.ServiceBusMessage`, que permite leer el mensaje de la cola en la función.

```python
import azure.functions as func

import logging
import json

def main(msg: func.ServiceBusMessage):
    logging.info('Python ServiceBus queue trigger processed message.')

    result = json.dumps({
        'message_id': msg.message_id,
        'body': msg.get_body().decode('utf-8'),
        'content_type': msg.content_type,
        'expiration_time': msg.expiration_time,
        'label': msg.label,
        'partition_key': msg.partition_key,
        'reply_to': msg.reply_to,
        'reply_to_session_id': msg.reply_to_session_id,
        'scheduled_enqueue_time': msg.scheduled_enqueue_time,
        'session_id': msg.session_id,
        'time_to_live': msg.time_to_live,
        'to': msg.to,
        'user_properties': msg.user_properties,
        'metadata' : msg.metadata
    })

    logging.info(result)
```

# <a name="java"></a>[Java](#tab/java)

La siguiente función de Java usa la anotación `@ServiceBusQueueTrigger` de la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime) para describir la configuración de un desencadenador de cola de Service Bus. La función toma el mensaje colocado en la cola y lo agrega a los registros.

```java
@FunctionName("sbprocessor")
 public void serviceBusProcess(
    @ServiceBusQueueTrigger(name = "msg",
                             queueName = "myqueuename",
                             connection = "myconnvarname") String message,
   final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
```

Las funciones de Java también pueden desencadenarse cuando se agrega un mensaje a un tema de Service Bus. En el ejemplo siguiente se usa la anotación `@ServiceBusTopicTrigger` para describir la configuración del desencadenador.

```java
@FunctionName("sbtopicprocessor")
    public void run(
        @ServiceBusTopicTrigger(
            name = "message",
            topicName = "mytopicname",
            subscriptionName = "mysubscription",
            connection = "ServiceBusConnection"
        ) String message,
        final ExecutionContext context
    ) {
        context.getLogger().info(message);
    }
```

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

Para las [bibliotecas de clases de C#](functions-dotnet-class-library.md), utilice los siguientes atributos para configurar un desencadenador de Service Bus:

* [ServiceBusTriggerAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusTriggerAttribute.cs)

  El constructor del atributo toma el nombre de la cola o del tema y la suscripción. En la versión 1.x de Azure Functions, también puede especificar los derechos de acceso de la conexión. Si no los especifica, el valor predeterminado es `Manage`. Para más información, consulte la sección [Desencadenador: configuración](#configuration).

  A continuación, tiene un ejemplo que muestra el atributo que se usa con un parámetro de cadena:

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue")] string myQueueItem, ILogger log)
  {
      ...
  }
  ```

  Dado que la propiedad `Connection` no está definida, Functions busca una configuración de la aplicación denominada `AzureWebJobsServiceBus`, que es el nombre predeterminado de la cadena de conexión de Service Bus. También puede establecer la propiedad `Connection` para especificar el nombre de una configuración de aplicación que contiene la cadena de conexión de Service Bus que se va a usar, tal como se muestra en el ejemplo siguiente:

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
      string myQueueItem, ILogger log)
  {
      ...
  }
  ```

  Para obtener un ejemplo completo, consulte el [Desencadenador: ejemplo](#example).

* [ServiceBusAccountAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAccountAttribute.cs)

  Proporciona otra manera de especificar la cuenta de Service Bus que se debe usar. El constructor toma el nombre de una configuración de aplicación que contiene una cadena de conexión de Service Bus. El atributo se puede aplicar en el nivel de clase, método o parámetro. En el ejemplo siguiente se muestran el nivel de clase y de método:

  ```csharp
  [ServiceBusAccount("ClassLevelServiceBusAppSetting")]
  public static class AzureFunctions
  {
      [ServiceBusAccount("MethodLevelServiceBusAppSetting")]
      [FunctionName("ServiceBusQueueTriggerCSharp")]
      public static void Run(
          [ServiceBusTrigger("myqueue", AccessRights.Manage)] 
          string myQueueItem, ILogger log)
  {
      ...
  }
  ```

La cuenta de Service Bus que se debe usar se determina en el orden siguiente:

* La propiedad `Connection` del atributo `ServiceBusTrigger`.
* El atributo `ServiceBusAccount` aplicado al mismo parámetro que el atributo `ServiceBusTrigger`.
* El atributo `ServiceBusAccount` aplicado a la función.
* El atributo `ServiceBusAccount` aplicado a la clase.
* La configuración de aplicación "AzureWebJobsServiceBus".

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

La anotación `ServiceBusQueueTrigger` permite crear una función que se ejecuta cuando se crea un mensaje de la cola de Service Bus. Las opciones de configuración disponibles incluyen el nombre de la cola y el nombre de la cadena de conexión.

La anotación `ServiceBusTopicTrigger` permite designar un tema y una suscripción para establecer qué datos desencadenan la función.

Consulte el [ejemplo](#example) de desencadenador para más detalles.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `ServiceBusTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Debe establecerse en "serviceBusTrigger". Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal.|
|**direction** | N/D | Debe establecerse en "in". Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal. |
|**name** | N/D | Nombre de la variable que representa el mensaje de cola o tema en el código de la función. |
|**queueName**|**QueueName**|Nombre de la cola que se debe supervisar.  Se establece únicamente si se supervisa una cola, no un tema.
|**topicName**|**TopicName**|Nombre del tema que se debe supervisar. Se establece únicamente si se supervisa un tema, no una cola.|
|**subscriptionName**|**SubscriptionName**|Nombre de la suscripción que se debe supervisar. Se establece únicamente si se supervisa un tema, no una cola.|
|**connection**|**Connection**|Nombre de una configuración de aplicación que contiene la cadena de conexión de Service Bus que se usará para este enlace. Si el nombre de la configuración de aplicación comienza con "AzureWebJobs", puede especificar solo el resto del nombre. Por ejemplo, si establece `connection` en "MyServiceBus", el entorno de ejecución de Functions busca una configuración de aplicación llamada "AzureWebJobsMyServiceBus". Si deja el valor de `connection` vacío, el entorno de ejecución de Functions usa la cadena de conexión de Service Bus predeterminada en la configuración de aplicación que se denomina "AzureWebJobsServiceBus".<br><br>Para obtener la cadena de conexión, siga los pasos mostrados en [Obtención de las credenciales de administración](../service-bus-messaging/service-bus-quickstart-portal.md#get-the-connection-string). La cadena de conexión debe ser para un espacio de nombres de Service Bus y no estar limitada a una cola o un tema concretos. |
|**accessRights**|**Acceder**|Derechos de acceso para la cadena de conexión. Los valores disponibles son `manage` y `listen`. El valor predeterminado es `manage`, lo que indica que `connection` tiene el permiso **Administrar**. Si usa una cadena de conexión que no tiene el permiso **Administrar**, establezca `accessRights` en "listen". De lo contrario, el runtime de Functions puede intentar realizar operaciones que requieran derechos de administración y no conseguirlo. En la versión 2.x y posteriores de Azure Functions, esta propiedad no está disponible porque la versión más reciente del SDK de Service Bus no admite operaciones de administración.|
|**isSessionsEnabled**|**IsSessionsEnabled**|`true` si se conecta a una cola o suscripción [compatible con sesiones](../service-bus-messaging/message-sessions.md). En caso contrario, `false`, que es el valor predeterminado.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

Los siguientes tipos de parámetro están disponibles para el mensaje de cola o tema:

* `string`: si el mensaje es texto.
* `byte[]`: útil para datos binarios.
* Un tipo personalizado: si el mensaje contiene el archivo JSON, Azure Functions intenta deserializar los datos JSON.
* `BrokeredMessage`: proporciona el mensaje deserializado con el método [BrokeredMessage.GetBody\<T>()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody?view=azure-dotnet#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1).
* [`MessageReceiver`](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver?view=azure-dotnet): se usa para recibir y confirmar mensajes del contenedor de mensajes (necesario cuando [`autoComplete`](functions-bindings-service-bus-output.md#hostjson-settings) está establecido en `false`).

Estos tipos de parámetro son para la versión 1.x de Azure Functions; para la versión 2.x y versiones posteriores, use [`Message`](/dotnet/api/microsoft.azure.servicebus.message) en lugar de `BrokeredMessage`.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Los siguientes tipos de parámetro están disponibles para el mensaje de cola o tema:

* `string`: si el mensaje es texto.
* `byte[]`: útil para datos binarios.
* Un tipo personalizado: si el mensaje contiene el archivo JSON, Azure Functions intenta deserializar los datos JSON.
* `BrokeredMessage`: proporciona el mensaje deserializado con el método [BrokeredMessage.GetBody\<T>()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody?view=azure-dotnet#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1).

Estos parámetros son para la versión de Azure Functions 1.x; para 2.x y versiones posteriores, use [`Message`](/dotnet/api/microsoft.azure.servicebus.message) en lugar de `BrokeredMessage`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Puede acceder al mensaje de cola o tema mediante el uso de `context.bindings.<name from function.json>`. El mensaje de Service Bus se pasa a la función como una cadena o como un objeto JSON.

# <a name="python"></a>[Python](#tab/python)

El mensaje de cola está disponible para la función mediante un parámetro de tipo `func.ServiceBusMessage`. El mensaje de Service Bus se pasa a la función como una cadena o como un objeto JSON.

# <a name="java"></a>[Java](#tab/java)

El mensaje de Service Bus entrante está disponible mediante un parámetro `ServiceBusQueueMessage` o `ServiceBusTopicMessage`.

[Consulte el ejemplo para más información](#example).

---

## <a name="poison-messages"></a>Mensajes dudosos

El control de mensajes dudosos no se puede controlar ni configurar en Azure Functions. Service Bus controla este tipo de mensajes.

## <a name="peeklock-behavior"></a>Comportamiento de PeekLock

El entorno de ejecución de Functions recibe un mensaje en [modo PeekLock](../service-bus-messaging/service-bus-performance-improvements.md#receive-mode). Llama a `Complete` en el mensaje si la función finaliza correctamente, o llama a `Abandon` si se produce un error en la función. Si la ejecución de la función dura más que el tiempo de espera de `PeekLock`, el bloqueo se renovará automáticamente siempre que la función esté en ejecución. 

El `maxAutoRenewDuration` se puede configurar en *host.json*, que se asigna a [OnMessageOptions.MaxAutoRenewDuration](/dotnet/api/microsoft.azure.servicebus.messagehandleroptions.maxautorenewduration?view=azure-dotnet). El valor predeterminado de esta opción es de 5 minutos según la documentación de Service Bus, aunque que puede aumentar el límite de tiempo de Functions desde el valor predeterminado de 5 minutos a 10 minutos. Sin embargo puede que no quiera hacer eso con las funciones de Service Bus, ya que podría superar el límite de renovación de este.

## <a name="message-metadata"></a>Metadatos del mensaje

El desencadenador de Service Bus proporciona varias [propiedades de metadatos](./functions-bindings-expressions-patterns.md#trigger-metadata). Estas propiedades pueden usarse como parte de expresiones de enlace en otros enlaces o como parámetros del código. Estas propiedades forman parte de la clase [Message](/dotnet/api/microsoft.azure.servicebus.message?view=azure-dotnet).

|Propiedad|Tipo|Descripción|
|--------|----|-----------|
|`ContentType`|`string`|Un identificador de tipo de contenido que usa el remitente y el receptor para la lógica específica de la aplicación.|
|`CorrelationId`|`string`|Identificador de correlación.|
|`DeadLetterSource`|`string`|Origen de los mensajes fallidos.|
|`DeliveryCount`|`Int32`|Número total de entregas.|
|`EnqueuedTimeUtc`|`DateTime`|Hora de puesta en la cola en UTC.|
|`ExpiresAtUtc`|`DateTime`|Hora de expiración en UTC.|
|`Label`|`string`|Etiqueta específica de la aplicación.|
|`MessageId`|`string`|Se trata de un valor definido por el usuario que Service Bus puede utilizar para identificar mensajes duplicados, si está habilitado.|
|`MessageReceiver`|`MessageReceiver`|Receptor de mensajes de Service Bus. Se puede usar para abandonar o completar el mensaje, o bien para colocarlo en la cola mensajes fallidos.|
|`MessageSession`|`MessageSession`|Receptor de mensajes específico para las colas y los temas habilitados para la sesión.|
|`ReplyTo`|`string`|La respuesta a la dirección de cola.|
|`SequenceNumber`|`long`|El número único que Service Bus asigna a un mensaje.|
|`To`|`string`|Dirección de envío.|
|`UserProperties`|`IDictionary<string, object>`|Propiedades establecidas por el remitente.|

Consulte los [ejemplos de código](#example) que utilizan estas propiedades más arriba en este artículo.

## <a name="next-steps"></a>Pasos siguientes

- [Envío de mensajes de Azure Service Bus desde Azure Functions (enlace de salida)](./functions-bindings-service-bus-output.md)
