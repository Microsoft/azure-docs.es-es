---
title: 'Suborquestaciones de Durable Functions: Azure Functions'
description: Introducción a la característica de orquestación Durable Functions de Azure.
author: cgillum
ms.topic: overview
ms.date: 09/08/2019
ms.author: azfuncdf
ms.openlocfilehash: 5eec15871279f3ca38c726fcd1ef1b21d0d38699
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750189"
---
# <a name="durable-orchestrations"></a>Orquestaciones de Durable Functions

Durable Functions es una extensión de [Azure Functions](../functions-overview.md). Puede usar una *función de orquestador* para organizar la ejecución de otras funciones de Durable Functions dentro de una aplicación de función. Las funciones de orquestador tienen las siguientes características:

* Las funciones de orquestador definen flujos de trabajo de función con código de procedimiento. No se necesitan esquemas ni diseñadores.
* Las funciones de orquestador pueden llamar a otras funciones duraderas de forma sincrónica y asincrónica. La salida de las funciones llamadas puede guardarse de forma confiable en variables locales.
* Las funciones de orquestador son duraderas y confiables. El progreso de la ejecución se controla automáticamente cuando la función está en espera o está procesándose. El estado local nunca se pierde si el proceso se recicla o se reinicia la máquina virtual.
* Las funciones de orquestador pueden ser de ejecución prolongada. La duración total de una *instancia de orquestación* puede ser de segundos, días o meses, o no terminar nunca.

En este artículo se ofrece información general sobre las funciones de orquestador y cómo pueden ayudarle a resolver diversos desafíos del desarrollo de aplicaciones. Si aún no está familiarizado con los tipos de funciones disponibles en una aplicación de Durable Functions, lea primero el artículo [Tipos de Durable Functions](durable-functions-types-features-overview.md).

## <a name="orchestration-identity"></a>Identidad de orquestación

Cada *instancia* de una orquestación tiene un identificador (también conocido como *identificador de instancia*). De forma predeterminada, cada identificador de instancia es un GUID generado automáticamente. Sin embargo, los identificadores de instancia también pueden ser un valor de cadena generado por el usuario. Cada identificador de instancia de orquestación debe ser único dentro de una [central de tareas](durable-functions-task-hubs.md).

A continuación se muestran algunas reglas acerca de los identificadores de instancia:

* Deben tener entre 1 y 256 caracteres.
* No deben comenzar por `@`.
* No deben contener caracteres `/`, `\`, `?` ni `#`.
* No deben contener caracteres de control.

> [!NOTE]
> En general, se recomienda usar identificadores de instancia generados automáticamente siempre que sea posible. Los identificadores de instancia generados por el usuario están diseñados para escenarios en los que hay una asignación de uno a uno entre una instancia de orquestación y alguna entidad externa específica de la aplicación, como un pedido de compra o un documento.

El identificador de instancia de una orquestación es un parámetro necesario en la mayoría de las [operaciones de administración de instancias](durable-functions-instance-management.md). También son importantes para el diagnóstico, como la [búsqueda de datos de seguimiento de orquestación ](durable-functions-diagnostics.md#application-insights) en Application Insights con fines de solución de problemas o análisis. Por esta razón, se recomienda guardar los identificadores de instancia generados en una ubicación externa (por ejemplo, una base de datos o en registros de aplicación), donde se puede hacer referencia a ellos fácilmente más adelante.

## <a name="reliability"></a>Confiabilidad

Las funciones del orquestador mantienen de forma confiable su estado de ejecución mediante el patrón de diseño [origen de eventos](/azure/architecture/patterns/event-sourcing). En lugar de almacenar directamente el estado actual de una orquestación, Durable Task Framework utiliza un almacén solo de anexión para registrar la serie completa de acciones que la orquestación de función realiza. Un almacén de solo anexar tiene muchas ventajas en comparación con el "volcado" del estado de tiempo de ejecución completo. Entre las ventajas se incluyen un mayor rendimiento, escalabilidad y capacidad de respuesta. También obtiene coherencia ocasional para los datos transaccionales y registros e historiales de auditoría completa. Los registros de auditoría son compatibles con las acciones de compensación confiables.

Durable Functions usa el aprovisionamiento de eventos transparente. En segundo plano, el operador `await` (C#) o `yield` (JavaScript y Python) en una función del orquestador devuelve el control del subproceso del orquestador al distribuidor de Durable Task Framework. El distribuidor, a continuación, confirma las nuevas acciones que la función de orquestador programó (como llamar a una o más funciones secundarias o programar un temporizador durable) al almacenamiento. Esta acción de confirmación transparente se anexa al historial de ejecución de la instancia de orquestación. El historial se guarda en una tabla de almacenamiento. La acción de confirmación, a continuación, agrega mensajes a una cola para programar el trabajo real. En este momento, la función del orquestador se puede descargar de la memoria.

Cuando una función de orquestación recibe más trabajo para realizar (por ejemplo, se recibe un mensaje de respuesta o expira un temporizador durable), el orquestador se activa y vuelve a ejecutar toda la función desde el principio con el fin de recompliar el estado local. Durante esta reproducción, si el código intenta realizar una llamada a una función (o realizar cualquier otro trabajo asincrónico), Durable Task Framework consulta el historial de ejecución de la orquestación actual. Si descubre que la [función de actividad](durable-functions-types-features-overview.md#activity-functions) ya se ha ejecutado y ofrece un resultado, reproduce el resultado de esa función y el código del orquestador sigue ejecutándose. La reproducción continúa hasta que finaliza el código de función o hasta que se programa un nuevo trabajo asincrónico.

> [!NOTE]
> Para que el patrón de reproducción funcione correctamente y de forma confiable, el código de la función de orquestador debe ser *determinista*. Para más información sobre las restricciones de código para las funciones de orquestador, vea el tema sobre las [restricciones de código de la función de orquestador](durable-functions-code-constraints.md).

> [!NOTE]
> Si una función de orquestador emite mensajes de registro, el comportamiento de reproducción puede provocar la emisión de mensajes de registro duplicados. Consulte el tema [Registro](durable-functions-diagnostics.md#app-logging) para más información sobre por qué se produce este comportamiento y cómo solucionarlo.

## <a name="orchestration-history"></a>Historial de orquestación

El comportamiento del origen de eventos de Durable Task Framework está estrechamente relacionado con el código de la función de orquestador que se escriba. Supongamos que tiene una función de orquestador de encadenamiento de actividad, como la siguiente función de orquestador:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("E1_HelloSequence")]
public static async Task<List<string>> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var outputs = new List<string>();

    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Tokyo"));
    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Seattle"));
    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "London"));

    // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
    return outputs;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const output = [];
    output.push(yield context.df.callActivity("E1_SayHello", "Tokyo"));
    output.push(yield context.df.callActivity("E1_SayHello", "Seattle"));
    output.push(yield context.df.callActivity("E1_SayHello", "London"));

    // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
    return output;
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    result1 = yield context.call_activity('SayHello', "Tokyo")
    result2 = yield context.call_activity('SayHello', "Seattle")
    result3 = yield context.call_activity('SayHello', "London")
    return [result1, result2, result3]

main = df.Orchestrator.create(orchestrator_function)
```
---

En cada instrucción `await` (C#) o `yield` (JavaScript y Python), Durable Task Framework establece los puntos de control del estado de ejecución de la función en un back-end de almacenamiento duradero (normalmente, en Azure Table Storage). Este estado es lo que se conoce como *historial de orquestación*.

### <a name="history-table"></a>Tabla del historial

Por lo general, Durable Task Framework hace lo siguiente en cada punto de control:

1. Guarda el historial de ejecución en las tablas de Azure Storage.
2. Pone en cola los mensajes para las funciones que el orquestador desea invocar.
3. Pone en cola los mensajes para el orquestador en sí,&mdash; por ejemplo, los de temporizador durable.

Una vez completado el punto de control, la función de orquestador se puede quitar de la memoria hasta que tenga más trabajo.

> [!NOTE]
> Azure Storage no proporciona garantías para la transacción de guardado de datos en Table Storage y las colas. Para controlar los errores, el proveedor de almacenamiento de Durable Functions usa patrones de *posible coherencia*. Estos patrones garantizan la ausencia de pérdida de datos en caso de bloqueo o pérdida de conectividad en medio de un punto de control.

Tras la finalización, el historial de la función mostrada anteriormente tiene un aspecto similar a la siguiente tabla en Azure Table Storage (abreviado con fines ilustrativos):

| PartitionKey (InstanceId)                     | EventType             | Timestamp               | Entrada | Nombre             | Resultado                                                    | Estado |
|----------------------------------|-----------------------|----------|--------------------------|-------|------------------|-----------------------------------------------------------|
| eaee885b | ExecutionStarted      | 2017-05-05T18:45:28.852Z | null  | E1_HelloSequence |                                                           |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:32.362Z |       |                  |                                                           |                     |
| eaee885b | TaskScheduled         | 2017-05-05T18:45:32.670Z |       | E1_SayHello      |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:32.670Z |       |                  |                                                           |                     |
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.201Z |       |                  | """Hello Tokyo!"""                                        |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:34.232Z |       |                  |                                                           |                     |
| eaee885b | TaskScheduled         | 2017-05-05T18:45:34.435Z |       | E1_SayHello      |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:34.435Z |       |                  |                                                           |                     |
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.763Z |       |                  | """Hello Seattle!"""                                      |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:34.857Z |       |                  |                                                           |                     |
| eaee885b | TaskScheduled         | 2017-05-05T18:45:34.857Z |       | E1_SayHello      |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:34.857Z |       |                  |                                                           |                     |
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.919Z |       |                  | """Hello London!"""                                       |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:35.032Z |       |                  |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:35.044Z |       |                  |                                                           |                     |
| eaee885b | ExecutionCompleted    | 2017-05-05T18:45:35.044Z |       |                  | "[""Hello Tokyo!"",""Hello Seattle!"",""Hello London!""]" | Completed           |

Notas sobre los valores de las columnas:

* **PartitionKey**: contiene el identificador de instancia de la orquestación.
* **EventType**: representa el tipo de evento. Puede ser uno de los siguientes tipos:
  * **OrchestrationStarted**: la función de orquestador se reanuda desde una instrucción await o se ejecuta por primera vez. La columna `Timestamp` se usa para rellenar el valor determinista para las API `CurrentUtcDateTime` (.NET), `currentUtcDateTime` (JavaScript) y `current_utc_datetime` (Python).
  * **ExecutionStarted**: la función de orquestador comenzó a ejecutarse por primera vez. Este evento también contiene la entrada de función en la columna `Input`.
  * **TaskScheduled**: se programó una función de actividad. El nombre de la función de actividad se captura en la columna `Name`.
  * **TaskCompleted**: función de actividad completada. El resultado de la función está en la columna `Result`.
  * **TimerCreated**: se ha creado un temporizador durable. La columna `FireAt` contiene la hora UTC programada a la que expira el temporizador.
  * **TimerFired**: se ha desencadenado un temporizador durable.
  * **EventRaised**: se envió un evento externo a la instancia de orquestación. La columna `Name` captura el nombre del evento y `Input`, la carga del evento.
  * **OrchestratorCompleted**: la función de orquestador esperada.
  * **ContinueAsNew**: la función de orquestador se ha completado y se ha reiniciado con el nuevo estado. La columna `Result` contiene el valor, que se utiliza como entrada en la instancia reiniciada.
  * **ExecutionCompleted**: la función de orquestador se ejecutó hasta finalizar o que se produjera un error. Las salidas de la función o los detalles del error se almacenan en la columna `Result`.
* **Marca de tiempo**: marca de tiempo UTC del evento del historial.
* **Name**: nombre de la aplicación de función invocada.
* **Entrada**: entrada de la función con formato JSON.
* **Result**: resultado de la función; es decir, el valor devuelto.

> [!WARNING]
> Aunque es útil como herramienta de depuración, no confíe en esta tabla. Puede cambiar con el progreso de la extensión Durable Functions.

Cada vez que se reanuda la función desde una instrucción `await` (C#) o `yield` (JavaScript y Python), Durable Task Framework vuelve a ejecutar la función de orquestador desde el principio. Con cada nueva ejecución, consulta el historial de ejecución para determinar si se ha realizado la operación asincrónica actual.  En caso afirmativo, el marco reproduce la salida de esa operación inmediatamente y pasa a la siguiente instrucción `await` (C#) o `yield` (JavaScript y Python). Este proceso continuará hasta que el historial entero se haya completado. Una vez que se haya reproducido el historial actual, las variables locales se restaurarán a sus valores anteriores.

## <a name="features-and-patterns"></a>Características y patrones

Las secciones siguientes describen las características y los patrones de las funciones de orquestador.

### <a name="sub-orchestrations"></a>Suborquestaciones

Las funciones de orquestador pueden llamar a las funciones de actividad, pero pueden llamar también a otras funciones de orquestador. Por ejemplo, puede crear una orquestación mayor a partir de una biblioteca de funciones de orquestador. También puede ejecutar varias instancias de una función de orquestador en paralelo.

Para más información y ejemplos, vea el artículo [Suborquestaciones](durable-functions-sub-orchestrations.md).

### <a name="durable-timers"></a>Temporizadores durables

Las orquestaciones pueden programar *temporizadores duraderos* para implementar retrasos o configurar el control de tiempo de espera en acciones asincrónicas. Use temporizadores durables en funciones de orquestador en lugar de `Thread.Sleep` y `Task.Delay` (C#), o `setTimeout()` y `setInterval()` (JavaScript), o `time.sleep()` (Python).

Para más información y ejemplos, consulte el artículo [Temporizadores en Durable Functions](durable-functions-timers.md).

### <a name="external-events"></a>Eventos externos

Las funciones de orquestador pueden esperar por los eventos externos para actualizar una instancia de orquestación. Esta característica de Durable Functions suele ser útil para controlar las interacciones humanas u otras devoluciones de llamada externas.

Para más información y ejemplos, consulte el artículo [Eventos externos](durable-functions-external-events.md).

### <a name="error-handling"></a>Control de errores

Las funciones de orquestador pueden usar las características de control de errores del lenguaje de programación. Los patrones existentes como `try`/`catch` se admiten en el código de orquestación.

Las funciones de orquestador también pueden agregar directivas de reintento a las funciones de suborquestador o actividad a las que llaman. Si una actividad o una función de suborquestador produce un error con una excepción, la directiva de reintentos especificada puede retrasar automáticamente la ejecución y volver a intentarla un número especificado de veces.

> [!NOTE]
> Si hay una excepción no controlada en una función de orquestador, la instancia de orquestación se completará con un estado `Failed`. Una instancia de orquestación no se puede reintentar una vez que se ha producido un error.

Para más información y ejemplos, vea el artículo [Control de errores](durable-functions-error-handling.md).

### <a name="critical-sections-durable-functions-2x-currently-net-only"></a>Secciones críticas (Durable Functions 2.x, actualmente solo .NET)

Las instancias de orquestación tienen un único subproceso, por lo que no es necesario preocuparse por las condiciones de carrera *dentro* de una orquestación. Sin embargo, se pueden producir condiciones de carrera cuando las orquestaciones interactúan con sistemas externos. Para mitigar las condiciones de carrera al interactuar con sistemas externos, las funciones de orquestador pueden definir *secciones críticas* con un método `LockAsync` de .NET.

En el código de ejemplo siguiente se muestra una función de orquestador que define una sección crítica. Entra en la sección crítica mediante el método `LockAsync`. Este método requiere pasar una o varias referencias a una [entidad duradera](durable-functions-entities.md), que administra de forma duradera el estado de bloqueo. Solo una única instancia de esta orquestación puede ejecutar el código de la sección crítica a la vez.

```csharp
[FunctionName("Synchronize")]
public static async Task Synchronize(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var lockId = new EntityId("LockEntity", "MyLockIdentifier");
    using (await context.LockAsync(lockId))
    {
        // critical section - only one orchestration can enter at a time
    }
}
```

`LockAsync` adquiere los bloqueos duraderos y devuelve un resultado `IDisposable` que finaliza la sección crítica cuando se desecha. Este resultado `IDisposable` se puede usar junto con un bloque `using` para obtener una representación sintáctica de la sección crítica. Cuando una función de orquestador entra en una sección crítica, solo una instancia puede ejecutar ese bloque de código. Todas las demás instancias que intenten entrar en la sección crítica se bloquearán hasta que la instancia anterior salga.

La característica de sección crítica también es útil para coordinar los cambios en las entidades duraderas. Para más información acerca de las secciones críticas, consulte el tema [Coordinación de entidades duraderas](durable-functions-entities.md#entity-coordination).

> [!NOTE]
> Las secciones críticas están disponibles en Durable Functions 2.0 y versiones posteriores. Actualmente, solo las orquestaciones de .NET implementan esta característica.

### <a name="calling-http-endpoints-durable-functions-2x"></a>Llamada a puntos de conexión HTTP (Durable Functions 2.x)

No se permite la E/S de las funciones de orquestador, según se describe en las [restricciones de código de las funciones de orquestador](durable-functions-code-constraints.md). La solución alternativa habitual para esta limitación es ajustar cualquier código que necesite realizar operaciones de E/S en una función de actividad. Las orquestaciones que interactúan con sistemas externos usan con frecuencia funciones de actividad para realizar llamadas HTTP y devolver el resultado a la orquestación.

# <a name="c"></a>[C#](#tab/csharp)

Para simplificar este patrón común, las funciones de orquestador pueden usar el método `CallHttpAsync` de .NET para invocar directamente a las API HTTP.

```csharp
[FunctionName("CheckSiteAvailable")]
public static async Task CheckSiteAvailable(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    Uri url = context.GetInput<Uri>();

    // Makes an HTTP GET request to the specified endpoint
    DurableHttpResponse response = 
        await context.CallHttpAsync(HttpMethod.Get, url);

    if (response.StatusCode >= 400)
    {
        // handling of error codes goes here
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const url = context.df.getInput();
    var res = yield context.df.callHttp("GET", url);
    if (res.statusCode >= 400) {
        // handling of error codes goes here
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    url = context.get_input()
    res = yield context.call_http('GET', url)
    if res.status_code >= 400:
        # handing of error code goes here
```
---

Además de admitir patrones de solicitud y respuesta básicos, el método admite el control automático de los patrones de sondeo asincrónicos HTTP 202 y, además, admite la autenticación de servicios externos mediante [identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md).

Para más información y ejemplos detallados, consulte el artículo [Características de HTTP](durable-functions-http-features.md).

> [!NOTE]
> La llamada a puntos de conexión HTTP directamente desde las funciones de orquestador está disponible en Durable Functions 2.0 y versiones posteriores.

### <a name="passing-multiple-parameters"></a>Paso de varios parámetros

No es posible pasar varios parámetros a una función de actividad directamente. Se recomienda pasar una matriz de objetos o de objetos compuestos.

# <a name="c"></a>[C#](#tab/csharp)

En .NET también puede usar objetos [ValueTuples](/dotnet/csharp/tuples). El ejemplo siguiente utiliza las nuevas características de [ValueTuples](/dotnet/csharp/tuples) agregado con [C# 7](/dotnet/csharp/whats-new/csharp-7#tuples):

```csharp
[FunctionName("GetCourseRecommendations")]
public static async Task<object> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string major = "ComputerScience";
    int universityYear = context.GetInput<int>();

    object courseRecommendations = await context.CallActivityAsync<object>(
        "CourseRecommendations",
        (major, universityYear));
    return courseRecommendations;
}

[FunctionName("CourseRecommendations")]
public static async Task<object> Mapper([ActivityTrigger] IDurableActivityContext inputs)
{
    // parse input for student's major and year in university
    (string Major, int UniversityYear) studentInfo = inputs.GetInput<(string, int)>();

    // retrieve and return course recommendations by major and university year
    return new
    {
        major = studentInfo.Major,
        universityYear = studentInfo.UniversityYear,
        recommendedCourses = new []
        {
            "Introduction to .NET Programming",
            "Introduction to Linux",
            "Becoming an Entrepreneur"
        }
    };
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

#### <a name="orchestrator"></a>Orquestador

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const location = {
        city: "Seattle",
        state: "WA"
    };
    const weather = yield context.df.callActivity("GetWeather", location);

    // ...
};
```

#### <a name="getweather-activity"></a>`GetWeather` Actividad

```javascript
module.exports = async function (context, location) {
    const {city, state} = location; // destructure properties into variables

    // ...
};
```

# <a name="python"></a>[Python](#tab/python)

#### <a name="orchestrator"></a>Orquestador

```python
from collections import namedtuple
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    Location = namedtuple('Location', ['city', 'state'])
    location = Location(city='Seattle', state= 'WA')

    weather = yield context.call_activity("GetWeather", location)

    # ...

```
#### <a name="getweather-activity"></a>`GetWeather` Actividad

```python
from collections import namedtuple

Location = namedtuple('Location', ['city', 'state'])

def main(location: Location) -> str:
    city, state = location
    return f"Hello {city}, {state}!"
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Restricciones de código del orquestador](durable-functions-code-constraints.md)
