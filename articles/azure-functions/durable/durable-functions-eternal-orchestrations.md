---
title: 'Orquestaciones infinitas en Durable Functions: Azure'
description: Aprenda a implementar orquestaciones infinitas mediante la extensión Durable Functions para Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: azfuncdf
ms.openlocfilehash: 34c70f4305ebb2c45757d982ab558aea6450003f
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86506373"
---
# <a name="eternal-orchestrations-in-durable-functions-azure-functions"></a>Orquestaciones infinitas en Durable Functions (Azure Functions)

Las *orquestaciones infinitas* son funciones de orquestador que nunca terminan. Son útiles si desea usar [Durable Functions](durable-functions-overview.md) con agregadores y con cualquier escenario que requiera un bucle infinito.

## <a name="orchestration-history"></a>Historial de orquestación

Como se explica en el tema del [historial de orquestación](durable-functions-orchestrations.md#orchestration-history), Durable Task Framework realiza un seguimiento del historial de cada función de orquestación. Este historial crecerá continuamente siempre y cuando la función de orquestador siga programando nuevo trabajo. Si la función de orquestador entra en un bucle infinito y programa trabajo continuamente, este historial podría alcanzar un tamaño crítico y provocar problemas de rendimiento considerables. El concepto de *orquestación infinita* se diseñó para mitigar este tipo de problemas en aplicaciones que necesitan bucles infinitos.

## <a name="resetting-and-restarting"></a>Restablecimiento y reinicio

En lugar de utilizar bucles infinitos, las funciones de orquestador restablecen su estado mediante una llamada a los métodos `ContinueAsNew` (.NET), `continueAsNew` (JavaScript) o `continue_as_new` (Python) del [enlace de desencadenador de orquestación](durable-functions-bindings.md#orchestration-trigger). Este método toma un único parámetro serializable con JSON, que se convierte en la nueva entrada para la siguiente generación de función de orquestador.

Cuando se llama a `ContinueAsNew`, la instancia pone en cola un mensaje para sí misma antes de cerrarse. El mensaje reinicia la instancia con el nuevo valor de entrada. Se conserva el mismo identificador de instancia, pero el historial de la función de orquestador se trunca eficazmente.

> [!NOTE]
> Durable Task Framework mantiene el mismo identificador de instancia, pero internamente crea un nuevo *identificador de ejecución* para la función de orquestador que `ContinueAsNew` restablece. Por lo general, este identificador de ejecución no se expone externamente, pero puede ser útil conocerlo al depurar la ejecución de la orquestación.

## <a name="periodic-work-example"></a>Ejemplo de trabajo periódico

Un caso práctico de orquestaciones infinitas es el del código que se necesita para realizar trabajo periódico indefinidamente.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Periodic_Cleanup_Loop")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("DoCleanup", null);

    // sleep for one hour between cleanups
    DateTime nextCleanup = context.CurrentUtcDateTime.AddHours(1);
    await context.CreateTimer(nextCleanup, CancellationToken.None);

    context.ContinueAsNew(null);
}
```

> [!NOTE]
> El ejemplo de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    yield context.df.callActivity("DoCleanup");

    // sleep for one hour between cleanups
    const nextCleanup = moment.utc(context.df.currentUtcDateTime).add(1, "h");
    yield context.df.createTimer(nextCleanup.toDate());

    context.df.continueAsNew(undefined);
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    yield context.call_activity("DoCleanup")

    # sleep for one hour between cleanups
    next_cleanup = context.current_utc_datetime + timedelta(hours = 1)
    yield context.create_timer(next_cleanup)

    context.continue_as_new(None)

main = df.Orchestrator.create(orchestrator_function)
```

---

La diferencia entre este ejemplo y una función desencadenada por temporizador es que aquí los tiempos del desencadenador de limpieza no se basan en una programación. Por ejemplo, una programación CRON que ejecuta una función cada hora lo hará a la 1:00, 2:00, 3:00, etc. y potencialmente podría encontrarse con problemas de superposición. Sin embargo, en este ejemplo, si la limpieza tarda 30 minutos, se programará a la 1:00, 2:30, 4:00, etc., de forma que no habrá posibilidad alguna de superposición.

## <a name="starting-an-eternal-orchestration"></a>Inicio de una orquestación infinita

Utilice los métodos `StartNewAsync` (.NET), `startNew` (JavaScript) o `start_new` (Python) para iniciar una orquestación infinita, igual que haría con cualquier otra función de orquestación.  

> [!NOTE]
> Si debe asegurarse de que se ejecuta una orquestación infinita singleton, es importante mantener el mismo `id` de instancia al iniciar la orquestación. Para más información, consulte el artículo sobre la [administración de instancias](durable-functions-instance-management.md).

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Trigger_Eternal_Orchestration")]
public static async Task<HttpResponseMessage> OrchestrationTrigger(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequestMessage request,
    [DurableClient] IDurableOrchestrationClient client)
{
    string instanceId = "StaticId";

    await client.StartNewAsync("Periodic_Cleanup_Loop", instanceId); 
    return client.CreateCheckStatusResponse(request, instanceId);
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar el atributo `OrchestrationClient` en lugar del atributo `DurableClient`, además de usar el tipo de parámetro `DurableOrchestrationClient` en lugar de `IDurableOrchestrationClient`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context, req) {
    const client = df.getClient(context);
    const instanceId = "StaticId";
    
    // null is used as the input, since there is no input in "Periodic_Cleanup_Loop".
    await client.startNew("Periodic_Cleanup_Loop", instanceId, null);

    context.log(`Started orchestration with ID = '${instanceId}'.`);
    return client.createCheckStatusResponse(context.bindingData.req, instanceId);
};
```
# <a name="python"></a>[Python](#tab/python)

```python
async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    instance_id = 'StaticId'

    await client.start_new('Periodic_Cleanup_Loop', instance_id, None)

    logging.info(f"Started orchestration with ID = '{instance_id}'.")
    return client.create_check_status_response(req, instance_id)

```

---

## <a name="exit-from-an-eternal-orchestration"></a>Salir de una orquestación infinita

Si en algún momento fuera necesario completar una función de orquestador, lo único que debe hacer es *no* llamar a `ContinueAsNew` y dejar que la función se cierre.

Si una función de orquestador se encuentra en un bucle infinito y debe detenerse, use los métodos `TerminateAsync` (.NET), `terminate` (JavaScript) o `terminate` (Python) del [enlace del cliente de orquestación](durable-functions-bindings.md#orchestration-client) para detenerla. Para más información, consulte el artículo sobre la [administración de instancias](durable-functions-instance-management.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información acerca de cómo implementar orquestaciones singleton](durable-functions-singletons.md)
