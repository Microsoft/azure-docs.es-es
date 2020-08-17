---
title: Enlace de salida de Azure Event Grid para Azure Functions
description: Aprenda a enviar un evento de Event Grid en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: fasttrack-edit, devx-track-python
ms.openlocfilehash: a471d793cc45cc8542aa933ef8ab0a79c42e7e1f
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848424"
---
# <a name="azure-event-grid-output-binding-for-azure-functions"></a>Enlace de salida de Azure Event Grid para Azure Functions

Use el enlace de salida de Event Hubs para escribir eventos en un tema personalizado. Debe tener una [clave de acceso válida para el tema personalizado](../event-grid/security-authenticate-publishing-clients.md).

Para obtener información sobre los detalles de instalación y configuración, consulte la [información general](./functions-bindings-event-grid.md).

> [!NOTE]
> El enlace de salida de Event Grid no es compatible con las firmas de acceso compartido (tokens de SAS). Debe usar la clave de acceso del tema.

> [!IMPORTANT]
> El enlace de salida de Event Grid solo está disponible para Functions 2.x y versiones posteriores.

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que escribe un mensaje en un tema personalizado de Event Grid usando el valor devuelto del método como resultado:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static EventGridEvent Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    return new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

En el siguiente ejemplo se muestra cómo usar la interfaz `IAsyncCollector` para enviar un lote de mensajes.

```csharp
[FunctionName("EventGridAsyncOutput")]
public static async Task Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    [EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]IAsyncCollector<EventGridEvent> outputEvents,
    ILogger log)
{
    for (var i = 0; i < 3; i++)
    {
        var myEvent = new EventGridEvent("message-id-" + i, "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
        await outputEvents.AddAsync(myEvent);
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestran los datos del enlace de salida de Event Grid en el archivo *function.json*.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Este es el código de script de C# que crea un evento:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out EventGridEvent outputEvent, ILogger log)
{
    outputEvent = new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

Este es el código de script de C# que crea varios eventos:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ICollector<EventGridEvent> outputEvent, ILogger log)
{
    outputEvent.Add(new EventGridEvent("message-id-1", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
    outputEvent.Add(new EventGridEvent("message-id-2", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestran los datos del enlace de salida de Event Grid en el archivo *function.json*.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Este es el código de JavaScript que crea un único evento:

```javascript
module.exports = async function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = {
        id: 'message-id',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    };
    context.done();
};
```

Este es el código de JavaScript que crea varios eventos:

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = [];

    context.bindings.outputEvent.push({
        id: 'message-id-1',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.bindings.outputEvent.push({
        id: 'message-id-2',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa el enlace. Después, envía un evento al tema personalizado, tal y como se especifica en `topicEndpointUri`.

Estos son los datos de enlace del archivo *function.json*:

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    },
    {
      "type": "eventGrid",
      "name": "outputEvent",
      "topicEndpointUri": "MyEventGridTopicUriSetting",
      "topicKeySetting": "MyEventGridTopicKeySetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Este es el ejemplo de Python para enviar un evento a un tema personalizado estableciendo `EventGridOutputEvent`:

```python
import logging
import azure.functions as func
import datetime


def main(eventGridEvent: func.EventGridEvent, 
         outputEvent: func.Out[func.EventGridOutputEvent]) -> None:

    logging.log("eventGridEvent: ", eventGridEvent)

    outputEvent.set(
        func.EventGridOutputEvent(
            id="test-id",
            data={"tag1": "value1", "tag2": "value2"},
            subject="test-subject",
            event_type="test-event-1",
            event_time=datetime.datetime.utcnow(),
            data_version="1.0"))
```

# <a name="java"></a>[Java](#tab/java)

El enlace de salida Event Grid no está disponible para Java.

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [EventGridAttribute](https://github.com/Azure/azure-functions-eventgrid-extension/blob/dev/src/EventGridExtension/OutputBinding/EventGridAttribute.cs).

El constructor del atributo toma el nombre de un valor de la aplicación que contiene el nombre del tema personalizado y el nombre de un valor de la aplicación que contiene la clave del tema. Para obtener más información sobre estas configuraciones, vea [Salida: configuración](#configuration). Este es un ejemplo de atributo `EventGrid`:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Para ver un ejemplo completo, consulte el [ejemplo](#example).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

El enlace de salida de Event Grid no está disponible para Python.

# <a name="java"></a>[Java](#tab/java)

El enlace de salida Event Grid no está disponible para Java.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `EventGrid`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Debe establecerse en "eventGrid". |
|**direction** | N/D | Debe establecerse en "out". Este parámetro se establece automáticamente cuando se crea el enlace en Azure Portal. |
|**name** | N/D | Nombre de la variable que se usa en el código de la función que representa el evento. |
|**topicEndpointUri** |**TopicEndpointUri** | Nombre de un valor de aplicación que contiene el URI para el tema personalizado, como `MyTopicEndpointUri`. |
|**topicKeySetting** |**TopicKeySetting** | Nombre de un valor de aplicación que contiene una clave de acceso para el tema personalizado. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Asegúrese de establecer el valor de la propiedad de configuración `TopicEndpointUri` en el nombre de un valor de la aplicación que contiene el URI del tema personalizado. No especifique el URI del tema personalizado directamente en esta propiedad.

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

Envíe mensajes mediante un parámetro de método, como `out EventGridEvent paramName`. Para escribir varios mensajes, puede usar `ICollector<EventGridEvent>` o `IAsyncCollector<EventGridEvent>` en lugar de `out EventGridEvent`.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Envíe mensajes mediante un parámetro de método, como `out EventGridEvent paramName`. En script de C#, `paramName` es el valor especificado en la propiedad `name` de *function.json*. Para escribir varios mensajes, puede usar `ICollector<EventGridEvent>` o `IAsyncCollector<EventGridEvent>` en lugar de `out EventGridEvent`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Acceda al evento de salida mediante `context.bindings.<name>`, donde `<name>` es el valor especificado en la propiedad `name` de *function.json*.

# <a name="python"></a>[Python](#tab/python)

El enlace de salida de Event Grid no está disponible para Python.

# <a name="java"></a>[Java](#tab/java)

El enlace de salida Event Grid no está disponible para Java.

---

## <a name="next-steps"></a>Pasos siguientes

* [Envío de un evento de Event Grid](./functions-bindings-event-grid-trigger.md)
