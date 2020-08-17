---
title: 'Control de eventos externos con Durable Functions: Azure'
description: Aprenda a controlar eventos externos en la extensión Durable Functions para Azure Functions.
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: azfuncdf
ms.openlocfilehash: 3cd04c93d508bd06c4ddd2e05074084202b9fc60
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87014946"
---
# <a name="handling-external-events-in-durable-functions-azure-functions"></a>Control de eventos externos con Durable Functions (Azure Functions)

Las funciones de orquestador tienen la capacidad de esperar y escuchar eventos externos. Esta característica de [Durable Functions](durable-functions-overview.md) suele ser útil para controlar las interacciones humanas u otros desencadenadores externos.

> [!NOTE]
> Los eventos externos son operaciones asincrónicas unidireccionales. No resultan adecuados para situaciones donde el cliente que envía el evento necesita una respuesta sincrónica de la función de orquestador.

## <a name="wait-for-events"></a>Espera de eventos

Los métodos [WaitForExternalEvent](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_WaitForExternalEvent_) (.NET) y `waitForExternalEvent` (JavaScript) y `wait_for_external_event` (Python) del [enlace de desencadenador de orquestación](durable-functions-bindings.md#orchestration-trigger) permiten que una función de orquestador espere y escuche un evento externo de manera asincrónica. La función de orquestador de escucha declara el *nombre* del evento y la *forma de los datos* que espera recibir.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("BudgetApproval")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool approved = await context.WaitForExternalEvent<bool>("Approval");
    if (approved)
    {
        // approval granted - do the approved action
    }
    else
    {
        // approval denied - send a notification
    }
}
```

> [!NOTE]
> El código de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const approved = yield context.df.waitForExternalEvent("Approval");
    if (approved) {
        // approval granted - do the approved action
    } else {
        // approval denied - send a notification
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    approved = context.wait_for_external_event('Approval')
    if approved:
        # approval granted - do the approved action
    else:
        # approval denied - send a notification

main = df.Orchestrator.create(orchestrator_function)
```

---

El ejemplo anterior escucha un evento único específico y toma medidas cuando se recibe.

Puede escuchar varios eventos al mismo tiempo, al igual que en el ejemplo siguiente, que espera una de tres notificaciones de eventos posibles.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Select")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var event1 = context.WaitForExternalEvent<float>("Event1");
    var event2 = context.WaitForExternalEvent<bool>("Event2");
    var event3 = context.WaitForExternalEvent<int>("Event3");

    var winner = await Task.WhenAny(event1, event2, event3);
    if (winner == event1)
    {
        // ...
    }
    else if (winner == event2)
    {
        // ...
    }
    else if (winner == event3)
    {
        // ...
    }
}
```

> [!NOTE]
> El código de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const event1 = context.df.waitForExternalEvent("Event1");
    const event2 = context.df.waitForExternalEvent("Event2");
    const event3 = context.df.waitForExternalEvent("Event3");

    const winner = yield context.df.Task.any([event1, event2, event3]);
    if (winner === event1) {
        // ...
    } else if (winner === event2) {
        // ...
    } else if (winner === event3) {
        // ...
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    event1 = context.wait_for_external_event('Event1')
    event2 = context.wait_for_external_event('Event2')
    event3 = context.wait_for_external_event('Event3')

    winner = context.task_any([event1, event2, event3])
    if winner == event1:
        # ...
    elif winner == event2:
        # ...
    elif winner == event3:
        # ...

main = df.Orchestrator.create(orchestrator_function)
```

---

El ejemplo anterior escucha *cualquiera* de varios eventos posibles. También es posible esperar *todos* los eventos.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("NewBuildingPermit")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string applicationId = context.GetInput<string>();

    var gate1 = context.WaitForExternalEvent("CityPlanningApproval");
    var gate2 = context.WaitForExternalEvent("FireDeptApproval");
    var gate3 = context.WaitForExternalEvent("BuildingDeptApproval");

    // all three departments must grant approval before a permit can be issued
    await Task.WhenAll(gate1, gate2, gate3);

    await context.CallActivityAsync("IssueBuildingPermit", applicationId);
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

En .NET, si la carga del evento no se puede convertir al tipo `T` esperado, se produce una excepción.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const applicationId = context.df.getInput();

    const gate1 = context.df.waitForExternalEvent("CityPlanningApproval");
    const gate2 = context.df.waitForExternalEvent("FireDeptApproval");
    const gate3 = context.df.waitForExternalEvent("BuildingDeptApproval");

    // all three departments must grant approval before a permit can be issued
    yield context.df.Task.all([gate1, gate2, gate3]);

    yield context.df.callActivity("IssueBuildingPermit", applicationId);
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    application_id = context.get_input()
    
    gate1 = context.wait_for_external_event('CityPlanningApproval')
    gate2 = context.wait_for_external_event('FireDeptApproval')
    gate3 = context.wait_for_external_event('BuildingDeptApproval')

    yield context.task_all([gate1, gate2, gate3])
    yield context.call_activity('IssueBuildingPermit', application_id)

main = df.Orchestrator.create(orchestrator_function)
```

---

`WaitForExternalEvent` espera indefinidamente alguna entrada.  La aplicación de función puede descargarse con seguridad mientras espera. En el momento en que un evento llega a esta instancia de orquestación, esta se activa automáticamente y procesa de inmediato el evento.

> [!NOTE]
> Si la aplicación de función usa el plan de consumo, no se aplican costos de facturación mientras una función de orquestador espera una tarea de `WaitForExternalEvent` (.NET), `waitForExternalEvent` (JavaScript) o `wait_for_external_event` (Python), independientemente de cuánto tiempo espere.

## <a name="send-events"></a>Envío de eventos

Puede usar los métodos [RaiseEventAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_RaiseEventAsync_) (.NET) o `raiseEventAsync` (JavaScript) para enviar un evento externo a una orquestación. Estos métodos los expone el enlace del [cliente de orquestación](durable-functions-bindings.md#orchestration-client). También puede usar la [API de HTTP de generación de evento](durable-functions-http-api.md#raise-event) integrada para enviar un evento externo a una orquestación.

Un evento generado incluye valores de *instance ID*, *eventName* y *eventData* como parámetros. Las funciones de Orchestrator controlan estos eventos con las API `WaitForExternalEvent` (.NET) o `waitForExternalEvent` (JavaScript). El valor de *eventName* debe coincidir con los extremos de envío y recepción para que se procese el evento. Los datos del evento también se deben poder serializar con JSON.

De manera interna, los mecanismos de "generación de eventos" ponen en cola un mensaje que recoge la función de orquestador en espera. Si la instancia no está esperando el *nombre de evento* especificado, el mensaje del evento se agrega a una cola en memoria. Si la instancia de orquestación inicia posteriormente la escucha de ese *nombre de evento*, se comprobará si hay mensajes de eventos en la cola.

> [!NOTE]
> Si no hay ninguna instancia de orquestación con el *identificador de instancia* especificado, se descartará el mensaje del evento.

A continuación se muestra una función desencadenada por la cola de ejemplo que envía un evento de aprobación a una instancia de la función de orquestador. El identificador de la instancia de orquestación procede del cuerpo del mensaje de la cola.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ApprovalQueueProcessor")]
public static async Task Run(
    [QueueTrigger("approval-queue")] string instanceId,
    [DurableClient] IDurableOrchestrationClient client)
{
    await client.RaiseEventAsync(instanceId, "Approval", true);
}
```

> [!NOTE]
> El código de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar el atributo `OrchestrationClient` en lugar del atributo `DurableClient`, además de usar el tipo de parámetro `DurableOrchestrationClient` en lugar de `IDurableOrchestrationClient`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);
    await client.raiseEvent(instanceId, "Approval", true);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(instance_id:str, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    await client.raise_event(instance_id, 'Approval', True)
```

---

Internamente, `RaiseEventAsync` (.NET), `raiseEvent` (JavaScript) o `raise_event` (Python) pone en cola un mensaje que la función de orquestador en espera selecciona. Si la instancia no está esperando el *nombre de evento* especificado, el mensaje del evento se agrega a una cola en memoria. Si la instancia de orquestación inicia posteriormente la escucha de ese *nombre de evento*, se comprobará si hay mensajes de eventos en la cola.

> [!NOTE]
> Si no hay ninguna instancia de orquestación con el *identificador de instancia* especificado, se descartará el mensaje del evento.

### <a name="http"></a>HTTP

A continuación, se muestra una solicitud HTTP que genera un evento de "Aprobación" para una instancia de orquestación. 

```http
POST /runtime/webhooks/durabletask/instances/MyInstanceId/raiseEvent/Approval&code=XXX
Content-Type: application/json

"true"
```

En este caso, el id. de la instancia está codificado de forma rígida como *MyInstanceId*.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga información sobre cómo implementar el control de errores](durable-functions-error-handling.md)

> [!div class="nextstepaction"]
> [Ejecutar un ejemplo que espera interacción humana](durable-functions-phone-verification.md)