---
title: Información general sobre Durable Functions en Azure
description: Introducción a la extensión Durable Functions de Azure Functions.
author: cgillum
ms.topic: overview
ms.date: 03/12/2020
ms.author: cgillum
ms.reviewer: azfuncdf
ms.openlocfilehash: 8fd670104a04229ed688b365de89e2ffc22b5429
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499388"
---
# <a name="what-are-durable-functions"></a>¿Qué es Durable Functions?

*Durable Functions* es una extensión de [Azure Functions](../functions-overview.md) que permite escribir funciones con estado en un entorno de proceso sin servidor. La extensión permite definir flujos de trabajo con estado mediante la escritura de [*funciones del orquestador*](durable-functions-orchestrations.md) y entidades con estado mediante la escritura de [*funciones de entidad*](durable-functions-entities.md) con el modelo de programación de Azure Functions. En segundo plano, la extensión administra automáticamente el estado, los puntos de comprobación y los reinicios, lo que le permite centrarse en la lógica de negocios.

## <a name="supported-languages"></a><a name="language-support"></a>

Durable Functions admite actualmente los siguientes idiomas:

* **C#** : tanto las [bibliotecas de clases precompiladas](../functions-dotnet-class-library.md) como [script C#](../functions-reference-csharp.md).
* **JavaScript**: solo es compatible con la versión 2.x del entorno de ejecución de Azure Functions. Requiere la versión 1.7.0 de la extensión Durable Functions, o una posterior. 
* **Python**: requiere la versión 1.8.5 de la extensión Durable Functions, o cualquier versión posterior. 
* **F#** : tanto las bibliotecas de clases precompiladas como script F#. El script F# solo es compatible con la versión 1.x del entorno de ejecución de Azure Functions.

Durable Functions tiene el objetivo de admitir todos los [idiomas de Azure Functions](../supported-languages.md). Consulte en la [lista de problemas de Durable Functions](https://github.com/Azure/azure-functions-durable-extension/issues) en qué punto se encuentra la compatibilidad con idiomas adicionales.

Al igual que con Azure Functions, existen plantillas que le ayudarán a desarrollar Durable Functions mediante [Visual Studio 2019](durable-functions-create-first-csharp.md), [Visual Studio Code](quickstart-js-vscode.md) y [Azure Portal](durable-functions-create-portal.md).

## <a name="application-patterns"></a>Patrones de aplicación

El caso de uso principal para Durable Functions es simplificar los requisitos de coordinación con estado complejos en las aplicaciones sin servidor. Las siguientes secciones describen patrones de aplicación típicos que se pueden beneficiar de Durable Functions:

* [Encadenamiento de funciones](#chaining)
* [Distribución ramificada de salida y de entrada](#fan-in-out)
* [API de HTTP asincrónico](#async-http)
* [Supervisión](#monitoring)
* [Interacción humana](#human)
* [Agregador (entidades con estado)](#aggregator)

### <a name="pattern-1-function-chaining"></a><a name="chaining"></a>Patrón 1: Diagrama de encadenamiento de funciones

En el modelo de encadenamiento de funciones, una secuencia de funciones se ejecuta en un orden específico. Con este patrón, la salida de una función se aplica a la entrada de otra función.

![Un diagrama del patrón de encadenamiento de funciones](./media/durable-functions-concepts/function-chaining.png)

Puede utilizar Durable Functions para implementar el patrón de encadenamiento de funciones de forma concisa, como se muestra en el siguiente ejemplo.

En este ejemplo, los valores `F1`, `F2`, `F3` y `F4` son los nombres de otras funciones de la aplicación de funciones. Puede implementar el flujo de control mediante construcciones de código imperativas normales. El código se ejecuta de arriba hacia abajo. y puede implicar semántica de flujo de control del lenguaje ya existente, como instrucciones condicionales y bucles. Puede incluir lógica de control de errores en bloques `try`/`catch`/`finally`.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Chaining")]
public static async Task<object> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    try
    {
        var x = await context.CallActivityAsync<object>("F1", null);
        var y = await context.CallActivityAsync<object>("F2", x);
        var z = await context.CallActivityAsync<object>("F3", y);
        return  await context.CallActivityAsync<object>("F4", z);
    }
    catch (Exception)
    {
        // Error handling or compensation goes here.
    }
}
```

Puede usar el parámetro `context` para invocar otras funciones por nombre, pasar parámetros y devolver la salida de una función. Cada vez que el código llama a `await`, el marco de Durable Functions establece puntos de control del progreso de la instancia actual de la función. Si el proceso o la máquina virtual se reciclan a mitad de la ejecución, la instancia de la función se reanuda desde la llamada a `await` anterior. Para obtener más información, consulte la siguiente sección, Patrón 2: Distribución ramificada de entrada y salida.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    try {
        const x = yield context.df.callActivity("F1");
        const y = yield context.df.callActivity("F2", x);
        const z = yield context.df.callActivity("F3", y);
        return    yield context.df.callActivity("F4", z);
    } catch (error) {
        // Error handling or compensation goes here.
    }
});
```

Puede usar el objeto `context.df` para invocar otras funciones por nombre, pasar parámetros y devolver la salida de una función. Cada vez que el código llama a `yield`, el marco de Durable Functions establece puntos de control del progreso de la instancia actual de la función. Si el proceso o la máquina virtual se reciclan a mitad de la ejecución, la instancia de la función se reanuda desde la llamada a `yield` anterior. Para obtener más información, consulte la siguiente sección, Patrón 2: Distribución ramificada de entrada y salida.

> [!NOTE]
> En JavaScript, el objeto `context` representa todo el [contexto de la función](../functions-reference-node.md#context-object). Acceda al contexto de Durable Functions mediante la propiedad `df` en el contexto principal.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    x = yield context.call_activity("F1", None)
    y = yield context.call_activity("F2", x)
    z = yield context.call_activity("F3", y)
    result = yield context.call_activity("F4", z)
    return result


main = df.Orchestrator.create(orchestrator_function)
```

Puede usar el objeto `context` para invocar otras funciones por nombre, pasar parámetros y devolver la salida de una función. Cada vez que el código llama a `yield`, el marco de Durable Functions establece puntos de control del progreso de la instancia actual de la función. Si el proceso o la máquina virtual se reciclan a mitad de la ejecución, la instancia de la función se reanuda desde la llamada a `yield` anterior. Para obtener más información, consulte la siguiente sección, Patrón 2: Distribución ramificada de entrada y salida.

> [!NOTE]
> El objeto `context` en Python representa el contexto de la orquestación. Acceda al contexto de Azure Functions principal mediante la propiedad `function_context` en el contexto de la orquestación.

---

### <a name="pattern-2-fan-outfan-in"></a><a name="fan-in-out"></a>Patrón 2: distribución ramificada de entrada y salida

En el patrón de distribución ramificada de salida y entrada, se ejecutan en paralelo varias funciones y después se espera a que todas finalicen. A menudo se realiza algún trabajo de agregación en los resultados devueltos de las funciones.

![Un diagrama del patrón de distribución ramificada de entrada y salida](./media/durable-functions-concepts/fan-out-fan-in.png)

Con las funciones normales, puedes realizar la distribución ramificada de salida al hacer que la función envíe varios mensajes a una cola. La distribución ramificada de entrada es mucho más compleja. Para realizarla en una función normal, escribe código para realizar un seguimiento de cuándo finalizan las funciones desencadenadas por la cola y después almacenar la salida de la función.

La extensión de Durable Functions controla este patrón con código relativamente sencillo:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("FanOutFanIn")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // Get a list of N work items to process in parallel.
    object[] workBatch = await context.CallActivityAsync<object[]>("F1", null);
    for (int i = 0; i < workBatch.Length; i++)
    {
        Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
        parallelTasks.Add(task);
    }

    await Task.WhenAll(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    int sum = parallelTasks.Sum(t => t.Result);
    await context.CallActivityAsync("F3", sum);
}
```

El trabajo de distribución ramificada se distribuye en varias instancias de la función `F2`. Se realiza un seguimiento del trabajo mediante una lista dinámica de las tareas. Se llama a la `Task.WhenAll` para esperar a que todas las funciones llamadas finalicen. Después, los resultados de la función `F2` se agregan desde la lista de tareas dinámica y se pasan a la función `F3`.

La creación automática de puntos de control que se produce en la llamada a `await` en `Task.WhenAll` garantiza que cualquier posible bloqueo o reinicio a mitad del proceso no requiera que se reinicien las tareas ya completadas.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const parallelTasks = [];

    // Get a list of N work items to process in parallel.
    const workBatch = yield context.df.callActivity("F1");
    for (let i = 0; i < workBatch.length; i++) {
        parallelTasks.push(context.df.callActivity("F2", workBatch[i]));
    }

    yield context.df.Task.all(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    const sum = parallelTasks.reduce((prev, curr) => prev + curr, 0);
    yield context.df.callActivity("F3", sum);
});
```

El trabajo de distribución ramificada se distribuye en varias instancias de la función `F2`. Se realiza un seguimiento del trabajo mediante una lista dinámica de las tareas. Se llama a `context.df.Task.all` API para esperar a que todas las funciones llamadas finalicen. Después, los resultados de la función `F2` se agregan desde la lista de tareas dinámica y se pasan a la función `F3`.

La creación automática de puntos de control que se produce en la llamada a `yield` en `context.df.Task.all` garantiza que cualquier posible bloqueo o reinicio a mitad del proceso no requiera que se reinicien las tareas ya completadas.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    # Get a list of N work items to process in parallel.
    work_batch = yield context.call_activity("F1", None)

    parallel_tasks = [ context.call_activity("F2", b) for b in work_batch ]
    
    outputs = yield context.task_all(parallel_tasks)

    # Aggregate all N outputs and send the result to F3.
    total = sum(outputs)
    yield context.call_activity("F3", total)


main = df.Orchestrator.create(orchestrator_function)
```

El trabajo de distribución ramificada se distribuye en varias instancias de la función `F2`. Se realiza un seguimiento del trabajo mediante una lista dinámica de las tareas. Se llama a `context.task_all` API para esperar a que todas las funciones llamadas finalicen. Después, los resultados de la función `F2` se agregan desde la lista de tareas dinámica y se pasan a la función `F3`.

La creación automática de puntos de control que se produce en la llamada a `yield` en `context.task_all` garantiza que cualquier posible bloqueo o reinicio a mitad del proceso no requiera que se reinicien las tareas ya completadas.

---

> [!NOTE]
> En raras ocasiones es posible que se produzca un bloqueo en la ventana después de que se complete una función de actividad, pero antes de que su finalización se guarde en el historial de la orquestación. Em ese caso, la función de actividad se volvería a ejecutar desde el principio en cuanto se recuperara el proceso.

### <a name="pattern-3-async-http-apis"></a><a name="async-http"></a>Patrón 3: Las API de HTTP asincrónico

El patrón de las API HTTP asincrónico soluciona el problema de coordinar el estado de las operaciones de larga duración con los clientes externos. Una forma habitual de implementar este patrón es que un punto de conexión HTTP desencadene la acción de larga duración. A continuación, el cliente se redirige a un punto de conexión de estado al que sondea para saber cuando finalice la operación.

![Un diagrama del modelo de API de HTTP](./media/durable-functions-concepts/async-http-api.png)

Durable Functions proporciona **compatibilidad integrada** con este patrón, lo que simplifica, o incluso elimina, el código que hay que escribir para interactuar con ejecuciones de funciones de larga duración. Por ejemplo, los ejemplos que se incluyen en el inicio rápido de Durable Functions ([C#](durable-functions-create-first-csharp.md) y [JavaScript](quickstart-js-vscode.md)) muestran un comando REST simple que se puede usar para iniciar nuevas instancias de funciones del orquestador. Tras iniciar una instancia, la extensión expone las API de HTTP de webhook que consultan el estado de la función del orquestador. 

En el ejemplo siguiente se muestran los comandos REST para iniciar un orquestador y consultar su estado. Para que el ejemplo se vea con mayor claridad se omiten algunos detalles del protocolo.

```
> curl -X POST https://myfunc.azurewebsites.net/api/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2019-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2019-03-16T21:20:57Z", ...}
```

Dado que el entorno en tiempo de ejecución de Durable Functions administra el estado, no es necesario que implemente su propio mecanismo de seguimiento del estado.

La extensión Durable Functions expone las API HTTP integradas que administran orquestaciones de larga duración. Este patrón también lo puede implementar usted mismo si utiliza sus propios desencadenadores de función (por ejemplo, HTTP, una cola o Azure Event Hubs) y el [enlace del cliente de orquestación](durable-functions-bindings.md#orchestration-client). Por ejemplo, puede utilizar un mensaje de cola para desencadenar la terminación. O bien puede usar un desencadenador de HTTP que esté protegido por una directiva de autenticación de Azure Active Directory, en lugar de las API HTTP integradas que utilizan una clave generada para la autenticación.

Para más información, consulte el artículo acerca de las [características de HTTP](durable-functions-http-features.md), en el que se explica cómo se pueden exponer procesos asincrónicos de larga duración a través de HTTP con la extensión Durable Functions.

### <a name="pattern-4-monitor"></a><a name="monitoring"></a>Patrón 4: Supervisión

El patrón de supervisión hace referencia a un proceso flexible y periódico en un flujo de trabajo. Un ejemplo es el sondeo hasta que se cumplen condiciones específicas. Puede usar un [desencadenador de temporizador](../functions-bindings-timer.md) normal para solucionar un escenario simple (como un trabajo de limpieza periódico), pero el intervalo es estático y resulta más difícil administrar los ciclos de vida de las instancias. Puede usar Durable Functions para crear intervalos de periodicidad flexible, administrar el ciclo de vida de las tareas y crear varios procesos de supervisión a partir de una única orquestación.

Un ejemplo del patrón de supervisión es invertir el escenario anterior de la API de HTTP asincrónica. En lugar de exponer un punto de conexión para que un cliente externo supervise una operación de larga duración, el monitor de larga duración consume el punto de conexión externo y espera algún cambio de estado.

![Un diagrama del patrón de supervisión](./media/durable-functions-concepts/monitor.png)

Con unas cuantas líneas de código, puede utilizar Durable Functions para crear varios monitores que observen puntos de conexión arbitrarios. Los monitores pueden finalizar la ejecución cuando se cumple una condición u otra función puede usar el cliente de orquestación durable para terminar los monitores. Puede cambiar el intervalo de `wait` del monitor según una condición específica (por ejemplo, retroceso exponencial). 

El código siguiente implementa un monitor básico:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("MonitorJobStatus")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform an action when a condition is met.
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform more work here, or let the orchestration end.
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const jobId = context.df.getInput();
    const pollingInterval = getPollingInterval();
    const expiryTime = getExpiryTime();

    while (moment.utc(context.df.currentUtcDateTime).isBefore(expiryTime)) {
        const jobStatus = yield context.df.callActivity("GetJobStatus", jobId);
        if (jobStatus === "Completed") {
            // Perform an action when a condition is met.
            yield context.df.callActivity("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        const nextCheck = moment.utc(context.df.currentUtcDateTime).add(pollingInterval, 's');
        yield context.df.createTimer(nextCheck.toDate());
    }

    // Perform more work here, or let the orchestration end.
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df
import json
from datetime import timedelta 


def orchestrator_function(context: df.DurableOrchestrationContext):
    job = json.loads(context.get_input())
    job_id = job["jobId"]
    polling_interval = job["pollingInterval"]
    expiry_time = job["expiryTime"]

    while context.current_utc_datetime < expiry_time:
        job_status = yield context.call_activity("GetJobStatus", job_id)
        if job_status == "Completed":
            # Perform an action when a condition is met.
            yield context.call_activity("SendAlert", job_id)
            break

        # Orchestration sleeps until this time.
        next_check = context.current_utc_datetime + timedelta(seconds=polling_interval)
        yield context.create_timer(next_check)

    # Perform more work here, or let the orchestration end.


main = df.Orchestrator.create(orchestrator_function)
```

---

Cuando se recibe una solicitud, se crea una nueva instancia de orquestación para ese identificador de trabajo. La instancia sondea un estado hasta que se cumple una condición y se cierra el bucle. Un temporizador durable controla el intervalo de sondeo. Después, se puede realizar trabajo adicional o puede finalizar la orquestación. Cuando el valor de `nextCheck` supera el de `expiryTime`, el monitor finaliza.

### <a name="pattern-5-human-interaction"></a><a name="human"></a>Patrón 5: Interacción humana

Muchos procesos automatizados implican algún tipo de interacción humana. La intervención humana en un proceso automatizado es más difícil, ya que las personas no tienen la misma alta disponibilidad y capacidad de respuesta que los servicios en la nube. Los procesos automatizados pueden permitir esta interacción mediante el uso de tiempos de expiración y la lógica de compensación.

Un proceso de aprobación es un ejemplo de un proceso empresarial que implica la interacción humana. Para un informe de gastos que supera un importe en dólares determinado puede ser necesaria la aprobación de un administrador. Si el administrador no aprueba el informe de gastos en un plazo de 72 horas (puede estar de vacaciones), se inicia un proceso de escalado para obtener la aprobación de otra persona (por ejemplo el jefe del administrador).

![Un diagrama del patrón de interacción humana](./media/durable-functions-concepts/approval.png)

Puede implementar el patrón en este ejemplo mediante una función de orquestador. El orquestador usa un [temporizador durable](durable-functions-timers.md) para solicitar la aprobación, y la escala si se agota el tiempo de espera. El orquestador espera a que ocurra un [evento externo](durable-functions-external-events.md), como una notificación generada por interacción humana.

Estos ejemplos crean un proceso de aprobación para hacer una demostración del patrón de interacción humana:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval", null);
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate", null);
        }
    }
}
```

Para crear el temporizador durable, llame a `context.CreateTimer`. `context.WaitForExternalEvent` recibe la notificación. Después, se realiza una llamada a `Task.WhenAny` para decidir si la aprobación se escala (primero se agota el tiempo de expiración) o se procesa (la aprobación antes se recibe de que se agote el tiempo de expiración).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require('moment');

module.exports = df.orchestrator(function*(context) {
    yield context.df.callActivity("RequestApproval");

    const dueTime = moment.utc(context.df.currentUtcDateTime).add(72, 'h');
    const durableTimeout = context.df.createTimer(dueTime.toDate());

    const approvalEvent = context.df.waitForExternalEvent("ApprovalEvent");
    if (approvalEvent === yield context.df.Task.any([approvalEvent, durableTimeout])) {
        durableTimeout.cancel();
        yield context.df.callActivity("ProcessApproval", approvalEvent.result);
    } else {
        yield context.df.callActivity("Escalate");
    }
});
```

Para crear el temporizador durable, llame a `context.df.createTimer`. `context.df.waitForExternalEvent` recibe la notificación. Después, se realiza una llamada a `context.df.Task.any` para decidir si la aprobación se escala (primero se agota el tiempo de expiración) o se procesa (la aprobación antes se recibe de que se agote el tiempo de expiración).

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df
import json
from datetime import timedelta 


def orchestrator_function(context: df.DurableOrchestrationContext):
    yield context.call_activity("RequestApproval", None)

    due_time = context.current_utc_datetime + timedelta(hours=72)
    durable_timeout_task = context.create_timer(due_time)
    approval_event_task = context.wait_for_external_event("ApprovalEvent")

    winning_task = yield context.task_any([approval_event_task, durable_timeout_task])

    if approval_event_task == winning_task:
        durable_timeout_task.cancel()
        yield context.call_activity("ProcessApproval", approval_event_task.result)
    else:
        yield context.call_activity("Escalate", None)


main = df.Orchestrator.create(orchestrator_function)
```

Para crear el temporizador durable, llame a `context.create_timer`. `context.wait_for_external_event` recibe la notificación. Después, se realiza una llamada a `context.task_any` para decidir si la aprobación se escala (primero se agota el tiempo de expiración) o se procesa (la aprobación antes se recibe de que se agote el tiempo de expiración).

---

Un cliente externo puede enviar la notificación de eventos a una función de orquestador en espera mediante las [API HTTP integradas](durable-functions-http-api.md#raise-event):

```bash
curl -d "true" http://localhost:7071/runtime/webhooks/durabletask/instances/{instanceId}/raiseEvent/ApprovalEvent -H "Content-Type: application/json"
```

También se pueden generar eventos mediante el cliente de orquestación durable desde otra función de la misma aplicación de funciones:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RaiseEventToOrchestration")]
public static async Task Run(
    [HttpTrigger] string instanceId,
    [DurableClient] IDurableOrchestrationClient client)
{
    bool isApproved = true;
    await client.RaiseEventAsync(instanceId, "ApprovalEvent", isApproved);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const isApproved = true;
    await client.raiseEvent(instanceId, "ApprovalEvent", isApproved);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df


async def main(client: str):
    durable_client = df.DurableOrchestrationClient(client)
    is_approved = True
    await durable_client.raise_event(instance_id, "ApprovalEvent", is_approved)
```

---

### <a name="pattern-6-aggregator-stateful-entities"></a><a name="aggregator"></a>Patrón 6: Agregador (entidades con estado)

El sexto patrón se trata de agregar datos de eventos durante un período de tiempo en una sola*entidad* direccionable. En este patrón, los datos que se agreguen pueden proceder de varios orígenes, pueden entregarse en lotes o pueden estar dispersos en largos períodos de tiempo. Es posible que el agregador tome medidas según datos de eventos a medida que llegan, y puede que los clientes externos necesiten consultar los datos agregados.

![Diagrama de agregador](./media/durable-functions-concepts/aggregator.png)

El aspecto más difícil de implementar este patrón con funciones normales sin estado es que el control de simultaneidad se convierte en un gran desafío. No solo es necesario preocuparse por varios subprocesos que modifican los mismos datos al mismo tiempo, también deberá preocuparse por garantizar que el agregador se ejecuta solo en una única VM a la vez.

Puede usar [entidades duraderas](durable-functions-entities.md) para implementar fácilmente este patrón como una sola función.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    int currentValue = ctx.GetState<int>();
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            int amount = ctx.GetInput<int>();
            ctx.SetState(currentValue + amount);
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(currentValue);
            break;
    }
}
```

Las entidades duraderas también pueden modelarse como clases en .NET. Este modelo puede ser útil si la lista de operaciones es fija y de gran tamaño. El ejemplo siguiente es una implementación equivalente de la entidad `Counter` que usa clases y métodos .NET.

```csharp
public class Counter
{
    [JsonProperty("value")]
    public int CurrentValue { get; set; }

    public void Add(int amount) => this.CurrentValue += amount;

    public void Reset() => this.CurrentValue = 0;

    public int Get() => this.CurrentValue;

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.entity(function(context) {
    const currentValue = context.df.getState(() => 0);
    switch (context.df.operationName) {
        case "add":
            const amount = context.df.getInput();
            context.df.setState(currentValue + amount);
            break;
        case "reset":
            context.df.setState(0);
            break;
        case "get":
            context.df.return(currentValue);
            break;
    }
});
```

# <a name="python"></a>[Python](#tab/python)

Las entidades duraderas no se admiten actualmente en Python.

---

Los clientes pueden poner en cola las *operaciones* (también conocidas como "señalización") de una función de entidad mediante el [enlace del cliente de la entidad](durable-functions-bindings.md#entity-client).

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static async Task Run(
    [EventHubTrigger("device-sensor-events")] EventData eventData,
    [DurableClient] IDurableOrchestrationClient entityClient)
{
    var metricType = (string)eventData.Properties["metric"];
    var delta = BitConverter.ToInt32(eventData.Body, eventData.Body.Offset);

    // The "Counter/{metricType}" entity is created on-demand.
    var entityId = new EntityId("Counter", metricType);
    await entityClient.SignalEntityAsync(entityId, "add", delta);
}
```

> [!NOTE]
> Los proxies generados dinámicamente también están disponibles en .NET para la señalización de entidades con seguridad de tipos. Y, además de la señalización, los clientes también pueden consultar el estado de cualquier función de entidad mediante [ métodos con seguridad de tipos](durable-functions-bindings.md#entity-client-usage) en el enlace del cliente de orquestación.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await context.df.signalEntity(entityId, "add", 1);
};
```

# <a name="python"></a>[Python](#tab/python)

Las entidades duraderas no se admiten actualmente en Python.

---

Las funciones de entidad están disponibles en [Durable Functions 2.0](durable-functions-versions.md) y versiones posteriores para C# y JavaScript.

## <a name="the-technology"></a>La tecnología

En segundo plano, la extensión Durable Functions se crea con [Durable Task Framework](https://github.com/Azure/durabletask), que es una biblioteca de código abierto que se encuentra en GitHub y se usa para crear flujos de trabajo en el código. Así como Azure Functions es la evolución sin servidor de Azure WebJobs, Durable Functions es la evolución sin servidor de Durable Task Framework. Microsoft y otras organizaciones utilizan Durable Task Framework ampliamente para automatizar los procesos críticos. Es una opción natural para el entorno de Azure Functions sin servidor.

## <a name="code-constraints"></a>Restricciones del código

Con el fin de proporcionar garantías de ejecución prolongada y confiable, las funciones del orquestador tienen un conjunto de reglas de codificación que deben seguirse. Para más información, consulte el artículo acerca de las [restricciones del código de las funciones de Orchestrator](durable-functions-code-constraints.md).

## <a name="billing"></a>Facturación

Durable Functions se factura igual que Azure Functions. Para más información, consulte los [precios de Azure Functions](https://azure.microsoft.com/pricing/details/functions/). Al ejecutar funciones de orquestador en el [plan de consumo](../functions-scale.md#consumption-plan) de Azure Functions, hay algunos comportamientos de facturación que deben tenerse en cuenta. Para más información acerca de estos comportamientos, consulte el artículo en el que se explica la [facturación de Durable Functions](durable-functions-billing.md).

## <a name="jump-right-in"></a>Comenzar de inmediato

Puede empezar a trabajar con Durable Functions en menos de 10 minutos completando uno de estos tutoriales de inicio rápido específicos del idioma:

* [C# con Visual Studio 2019](durable-functions-create-first-csharp.md)
* [JavaScript con Visual Studio Code](quickstart-js-vscode.md)
* [Python mediante Visual Studio Code](quickstart-python-vscode.md)

En ambos inicios rápidos, podrá crear y probar localmente una función durable de "hello world". Luego, publicará el código de función en Azure. La función que crea organiza y encadena llamadas a otras funciones.

## <a name="learn-more"></a>Más información

El siguiente vídeo resalta las ventajas de Durable Functions:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Durable-Functions-in-Azure-Functions/player] 

Para obtener una explicación más detallada tanto de Durable Functions como de su tecnología subyacente, vea el vídeo siguiente (en él se habla de .NET, pero los conceptos también se aplican a otros lenguajes admitidos):

> [!VIDEO https://channel9.msdn.com/Events/dotnetConf/2018/S204/player]

Durable Functions es una extensión avanzada para [Azure Functions](../functions-overview.md) que no es adecuada para todas las aplicaciones. Para obtener una comparación con otras tecnologías de orquestación de Azure, consulte [Comparativa entre Azure Functions y Azure Logic Apps](../functions-compare-logic-apps-ms-flow-webjobs.md#compare-azure-functions-and-azure-logic-apps).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tipos de funciones y características de Durable Functions](durable-functions-types-features-overview.md)
