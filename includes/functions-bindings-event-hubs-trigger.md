---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 03/05/2019
ms.author: cshoe
ms.openlocfilehash: d8c6b79dca97de3dd46eb9c677f2c94191f276b0
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89304087"
---
Use el desencadenador de funciones para responder a un evento enviado a una secuencia de eventos del centro de eventos. Debe tener acceso de lectura al centro de eventos subyacente para configurar el desencadenador. Cuando esta función se desencadena, el mensaje que se pasa a la función se escribe como una cadena.

## <a name="scaling"></a>Ampliación

Cada instancia de una función de desencadenador de eventos está respaldada por una única instancia de [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor). El desencadenador (con tecnología de Event Hubs) garantiza que solo una instancia de [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) puede obtener una concesión en una partición determinada.

Por ejemplo, considere una instancia de Event Hubs con las siguientes características:

* 10 particiones.
* 1000 eventos distribuidos uniformemente en todas las particiones, con 100 mensajes en cada partición.

Cuando se habilita la función por primera vez, solo hay una instancia de la función. Vamos a llamar a esta instancia de función `Function_0`. Laq función `Function_0` tiene una sola instancia de [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) que contiene una concesión en las diez particiones. Esta instancia lee eventos de las particiones 0-9. A partir de este punto, se producirá una de las siguientes acciones:

* **No se necesitan nuevas instancias de función**: `Function_0` puede procesar los 1000 eventos antes de que la lógica de escalado de Azure Functions surta efecto. En este caso, `Function_0` procesa los 1000 mensajes.

* **Se agrega una instancia de función adicional**: Si la lógica de escalado de Azure Functions determina que `Function_0` tiene más mensajes de los que puede procesar, se crea una nueva instancia de la aplicación de función (`Function_1`). Esta nueva función también tiene asociada una instancia de [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor). Como la instancia de Event Hubs subyacente detecta que una nueva instancia de host está tratando de leer mensajes, efectúa un equilibrio de carga de las particiones entre las instancias de host. Por ejemplo, las particiones 0-4 pueden asignarse a `Function_0` y las particiones 5-9, a `Function_1`.

* **Se agregan N instancias de función más**: Si la lógica de escalado de Azure Functions determina que tanto `Function_0` como `Function_1` tienen más mensajes de los que pueden procesar, se crean más instancias de aplicaciones de función de `Functions_N`.  Se van creando aplicaciones hasta llegar a un punto en el que `N` es mayor que el número de particiones de centro de eventos. En nuestro ejemplo, Event Hubs vuelve a equilibrar la carga de las particiones, en este caso, entre las instancias `Function_0`...`Functions_9`.

Cuando se produce el escalado, `N` instancias es un número mayor que el número de particiones del centro de eventos. Este patrón se usa para garantizar que va a haber instancias de [EventProcessorHost](https://docs.microsoft.com/dotnet/api/microsoft.azure.eventhubs.processor) disponibles para obtener bloqueos de las particiones a medida que estén disponibles en otras instancias. Solo se le cobra por los recursos usados cuando se ejecuta la instancia de la función. En otras palabras, no se le cobrará por este aprovisionamiento en exceso.

Cuando se completa la ejecución de todas las funciones con o sin errores, se agregan puntos de comprobación a la cuenta de almacenamiento asociada. Cuando estos puntos de conexión se agregan correctamente, los 1000 mensajes ya no se vuelven a recuperar.

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](../articles/azure-functions/functions-dotnet-class-library.md) que registra el cuerpo del mensaje del desencadenador del centro de eventos.

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, ILogger log)
{
    log.LogInformation($"C# function triggered to process a message: {myEventHubMessage}");
}
```

Para acceder a los [metadatos del evento](#event-metadata) en el código de función, cree un enlace a un objeto [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) (requiere el uso de una instrucción para `Microsoft.Azure.EventHubs`). También puede acceder a las mismas propiedades mediante el uso de expresiones de enlace en la firma del método.  El ejemplo siguiente se muestran las dos maneras de obtener los mismos datos:

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run(
    [EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData myEventHubMessage,
    DateTime enqueuedTimeUtc,
    Int64 sequenceNumber,
    string offset,
    ILogger log)
{
    log.LogInformation($"Event: {Encoding.UTF8.GetString(myEventHubMessage.Body)}");
    // Metadata accessed by binding to EventData
    log.LogInformation($"EnqueuedTimeUtc={myEventHubMessage.SystemProperties.EnqueuedTimeUtc}");
    log.LogInformation($"SequenceNumber={myEventHubMessage.SystemProperties.SequenceNumber}");
    log.LogInformation($"Offset={myEventHubMessage.SystemProperties.Offset}");
    // Metadata accessed by using binding expressions in method parameters
    log.LogInformation($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.LogInformation($"SequenceNumber={sequenceNumber}");
    log.LogInformation($"Offset={offset}");
}
```

Para recibir eventos en un lote, convierta `string` o `EventData` en una matriz.  

> [!NOTE]
> Cuando se reciben en un lote, no se puede establecer un enlace a parámetros de método como en el ejemplo anterior con `DateTime enqueuedTimeUtc` y debe recibirlos desde cada objeto `EventData`.  

```cs
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData[] eventHubMessages, ILogger log)
{
    foreach (var message in eventHubMessages)
    {
        log.LogInformation($"C# function triggered to process a message: {Encoding.UTF8.GetString(message.Body)}");
        log.LogInformation($"EnqueuedTimeUtc={message.SystemProperties.EnqueuedTimeUtc}");
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador de centro de eventos en un archivo *function.json* y una [función de script de C#](../articles/azure-functions/functions-reference-csharp.md) que usa el enlace. La función registra el cuerpo del mensaje del desencadenador de centro de eventos.

Los ejemplos siguientes muestran datos de enlace de Event Hubs en el archivo *function.json*.

### <a name="version-2x-and-higher"></a>Versión 2.x y posteriores

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

### <a name="version-1x"></a>Versión 1.x

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Este es el código de script de C#:

```cs
using System;

public static void Run(string myEventHubMessage, TraceWriter log)
{
    log.Info($"C# function triggered to process a message: {myEventHubMessage}");
}
```

Para acceder a los [metadatos del evento](#event-metadata) en el código de función, cree un enlace a un objeto [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) (requiere el uso de una instrucción para `Microsoft.Azure.EventHubs`). También puede acceder a las mismas propiedades mediante el uso de expresiones de enlace en la firma del método.  El ejemplo siguiente se muestran las dos maneras de obtener los mismos datos:

```cs
#r "Microsoft.Azure.EventHubs"

using System.Text;
using System;
using Microsoft.ServiceBus.Messaging;
using Microsoft.Azure.EventHubs;

public static void Run(EventData myEventHubMessage,
    DateTime enqueuedTimeUtc,
    Int64 sequenceNumber,
    string offset,
    TraceWriter log)
{
    log.Info($"Event: {Encoding.UTF8.GetString(myEventHubMessage.Body)}");
    log.Info($"EnqueuedTimeUtc={myEventHubMessage.SystemProperties.EnqueuedTimeUtc}");
    log.Info($"SequenceNumber={myEventHubMessage.SystemProperties.SequenceNumber}");
    log.Info($"Offset={myEventHubMessage.SystemProperties.Offset}");

    // Metadata accessed by using binding expressions
    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"SequenceNumber={sequenceNumber}");
    log.Info($"Offset={offset}");
}
```

Para recibir eventos en un lote, convierta `string` o `EventData` en una matriz:

```cs
public static void Run(string[] eventHubMessages, TraceWriter log)
{
    foreach (var message in eventHubMessages)
    {
        log.Info($"C# function triggered to process a message: {message}");
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador de centro de eventos en un archivo *function.json* y una [función de JavaScript](../articles/azure-functions/functions-reference-node.md) que usa el enlace. La función lee los [metadatos del evento](#event-metadata) y registra el mensaje.

Los ejemplos siguientes muestran datos de enlace de Event Hubs en el archivo *function.json*.

### <a name="version-2x-and-higher"></a>Versión 2.x y posteriores

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

### <a name="version-1x"></a>Versión 1.x

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Este es el código de JavaScript:

```javascript
module.exports = function (context, myEventHubMessage) {
    context.log('Function triggered to process a message: ', myEventHubMessage);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('SequenceNumber =', context.bindingData.sequenceNumber);
    context.log('Offset =', context.bindingData.offset);

    context.done();
};
```

Para recibir eventos en un lote, establezca `cardinality` en `many` en el archivo *function.json*, como se muestra en los ejemplos siguientes.

### <a name="version-2x-and-higher"></a>Versión 2.x y posteriores

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

### <a name="version-1x"></a>Versión 1.x

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "path": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Este es el código de JavaScript:

```javascript
module.exports = function (context, eventHubMessages) {
    context.log(`JavaScript eventhub trigger function called for message array ${eventHubMessages}`);

    eventHubMessages.forEach((message, index) => {
        context.log(`Processed message ${message}`);
        context.log(`EnqueuedTimeUtc = ${context.bindingData.enqueuedTimeUtcArray[index]}`);
        context.log(`SequenceNumber = ${context.bindingData.sequenceNumberArray[index]}`);
        context.log(`Offset = ${context.bindingData.offsetArray[index]}`);
    });

    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de desencadenador de centro de eventos en un archivo *function.json* y una [función de Python](../articles/azure-functions/functions-reference-python.md) que usa el enlace. La función lee los [metadatos del evento](#event-metadata) y registra el mensaje.

Los ejemplos siguientes muestran datos de enlace de Event Hubs en el archivo *function.json*.

```json
{
  "type": "eventHubTrigger",
  "name": "event",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Este es el código de Python:

```python
import logging
import azure.functions as func


def main(event: func.EventHubEvent):
    logging.info('Function triggered to process a message: ', event.get_body())
    logging.info('  EnqueuedTimeUtc =', event.enqueued_time)
    logging.info('  SequenceNumber =', event.sequence_number)
    logging.info('  Offset =', event.offset)

    # Metadata
    for key in event.metadata:
        logging.info(f'Metadata: {key} = ', event.metadata[key])
```

# <a name="java"></a>[Java](#tab/java)

En el ejemplo siguiente se muestra un enlace del centro de eventos que registra el cuerpo del mensaje del desencadenador de centro de eventos.

```java
@FunctionName("ehprocessor")
public void eventHubProcessor(
  @EventHubTrigger(name = "msg",
                  eventHubName = "myeventhubname",
                  connection = "myconnvarname") String message,
       final ExecutionContext context )
       {
          context.getLogger().info(message);
 }
```

 En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `EventHubTrigger` en los parámetros cuyo valor provendría del centro de eventos. Los parámetros con estas anotaciones hacen que la función se ejecuta cuando llega un evento.  Esta anotación se puede usar con tipos nativos de Java, POJO o valores que aceptan valores NULL mediante `Optional<T>`.

 ---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](../articles/azure-functions/functions-dotnet-class-library.md), use el atributo [EventHubTriggerAttribute](https://github.com/Azure/azure-functions-eventhubs-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs/EventHubTriggerAttribute.cs).

El constructor del atributo toma el nombre del centro de eventos, el nombre del grupo de consumidores y el nombre de una configuración de aplicación que contenga la cadena de conexión. Para obtener más información sobre estas configuraciones, vea la sección [Configuración de desencadenador](#configuration). Este es un ejemplo de atributo `EventHubTriggerAttribute`:

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, ILogger log)
{
    ...
}
```

Para un ejemplo completo, consulte [Desencadenador: ejemplo de C#](#example).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

En la [biblioteca en tiempo de ejecución de funciones](https://docs.microsoft.com/java/api/overview/azure/functions/runtime) de Java, utilice la anotación [EventHubTrigger](https://docs.microsoft.com/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger) en los parámetros cuyo valor provendría del centro de eventos. Los parámetros con estas anotaciones hacen que la función se ejecuta cuando llega un evento. Esta anotación se puede usar con tipos nativos de Java, POJO o valores que aceptan valores NULL mediante `Optional<T>`.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `EventHubTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Se debe establecer en `eventHubTrigger`. Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal.|
|**direction** | N/D | Se debe establecer en `in`. Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal. |
|**name** | N/D | Nombre de la variable que representa el elemento de evento en el código de la función. |
|**path** |**EventHubName** | Solo Functions 1.x. El nombre del centro de eventos. Cuando el nombre del centro de eventos también está presente en la cadena de conexión, ese valor reemplaza esta propiedad en tiempo de ejecución. |
|**eventHubName** |**EventHubName** | Functions 2.x y versiones posteriores. El nombre del centro de eventos. Cuando el nombre del centro de eventos también está presente en la cadena de conexión, ese valor reemplaza esta propiedad en tiempo de ejecución. Se puede hacer referencia a él desde la [configuración de la aplicación](../articles/azure-functions/functions-bindings-expressions-patterns.md#binding-expressions---app-settings) `%eventHubName%` |
|**consumerGroup** |**ConsumerGroup** | Una propiedad opcional que establece el [grupo de consumidores](../articles/event-hubs/event-hubs-features.md#event-consumers) que se usará para suscribirse a los eventos del centro. Si se pasa por alto, se utilizará el grupo de consumidores `$Default`. |
|**cardinalidad** | N/D | Se utiliza para los lenguajes distintos de C#. Defínalo como `many` para permitir el procesamiento por lotes.  Si se omite o se define como `one`, se pasa un único mensaje a la función.<br><br>En C#, esta propiedad se asigna automáticamente siempre que el desencadenador tenga una matriz para el tipo.|
|**connection** |**Connection** | El nombre de una configuración de aplicación que contenga la cadena de conexión para el espacio de nombres del centro de eventos. Copie esta cadena de conexión haciendo clic en el botón **Información de conexión** del [espacio de nombres](../articles/event-hubs/event-hubs-create.md#create-an-event-hubs-namespace), no del propio centro de eventos. Esta cadena de conexión debe tener al menos permisos de lectura para activar el desencadenador.|

[!INCLUDE [app settings to local.settings.json](../articles/azure-functions/../../includes/functions-app-settings-local.md)]

## <a name="event-metadata"></a>Metadatos de evento

El desencadenador de Event Hubs proporciona varias [propiedades de metadatos](../articles/azure-functions/./functions-bindings-expressions-patterns.md). Se pueden usar propiedades de metadatos como parte de expresiones de enlace en otros enlaces o como parámetros del código. Las propiedades proceden de la clase [EventData](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata).

|Propiedad|Tipo|Descripción|
|--------|----|-----------|
|`PartitionContext`|[PartitionContext](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.partitioncontext)|Instancia de `PartitionContext`.|
|`EnqueuedTimeUtc`|`DateTime`|Hora de puesta en la cola en UTC.|
|`Offset`|`string`|El desplazamiento de los datos relacionados con el flujo de partición de Event Hubs. El desplazamiento es un marcador o identificador del flujo de Event Hubs. El identificador es único dentro de una partición del flujo de Event Hubs.|
|`PartitionKey`|`string`|La partición a la que se deben enviar los datos del evento.|
|`Properties`|`IDictionary<String,Object>`|Las propiedades de usuario de los datos del evento.|
|`SequenceNumber`|`Int64`|El número de secuencia de registro del evento.|
|`SystemProperties`|`IDictionary<String,Object>`|Las propiedades del sistema, incluidos los datos del evento.|

Consulte los [ejemplos de código](#example) que utilizan estas propiedades más arriba en este artículo.

## <a name="hostjson-properties"></a>Propiedades de host.json
<a name="host-json"></a>

El archivo [host.json](../articles/azure-functions/functions-host-json.md#eventhub) contiene opciones de configuración que controlan el comportamiento de Event Hubs. La configuración varía en función de la versión de Azure Functions.

[!INCLUDE [functions-host-json-event-hubs](../articles/azure-functions/../../includes/functions-host-json-event-hubs.md)]