---
title: 'Enlaces para Durable Functions: Azure'
description: Aprenda a utilizar desencadenadores y enlaces en la extensión Durable Functions para Azure Functions.
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: azfuncdf
ms.openlocfilehash: 899bc3fdc94b8232acd3edf3e0cbab3c481ff8f2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081855"
---
# <a name="bindings-for-durable-functions-azure-functions"></a>Enlaces para Durable Functions (Azure Functions)

La extensión [Durable Functions](durable-functions-overview.md) presenta dos nuevos enlaces de desencadenador que controlan la ejecución de las funciones del orquestador y de actividad. También presenta a un enlace de salida que actúa como un cliente para el tiempo de ejecución de Durable Functions.

## <a name="orchestration-trigger"></a>Desencadenador de orquestación

El desencadenador de orquestación permite crear [funciones de orquestador durables](durable-functions-types-features-overview.md#orchestrator-functions). Este desencadenador admite iniciar nuevas instancias de función del orquestador y reanudar las instancias existentes de función del orquestador que están en "espera" de una tarea.

Al utilizar las herramientas de Visual Studio para Azure Functions, el desencadenador de orquestación se configura usando el atributo .NET [OrchestrationTriggerAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.orchestrationtriggerattribute?view=azure-dotnet).

Al escribir funciones de orquestador en lenguajes de scripting (por ejemplo, JavaScript o scripting en C#), el desencadenador de orquestación se define mediante el objeto JSON siguiente en la matriz `bindings` del archivo *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "orchestration": "<Optional - name of the orchestration>",
    "type": "orchestrationTrigger",
    "direction": "in"
}
```

* `orchestration` es el nombre de la orquestación que los clientes tienen que usar cuando quieren iniciar nuevas instancias de esta función de orquestador. Esta propiedad es opcional. Si no se especifica, se utiliza el nombre de la función.

Internamente este enlace de desencadenador sondea una serie de colas de la cuenta de almacenamiento predeterminado para la aplicación de la función. Estas colas son detalles de implementación internos de la extensión, esta es la razón de que no se configuren explícitamente en las propiedades de enlace.

### <a name="trigger-behavior"></a>Comportamiento de un desencadenador

Estas son algunas notas acerca del desencadenador de orquestación:

* **Subprocesamiento único**: un único subproceso de distribución se usa para toda la ejecución de función del orquestador en una instancia de host. Por esta razón, es importante asegurarse de que el código de función del orquestador es eficaz y no realiza ninguna E/S. También es importante asegurarse de que este subproceso no realizar ningún trabajo asincrónico excepto cuando se espera en tipos de tareas específicos de Durable Functions.
* **Control de mensajes dudosos**: no hay compatibilidad con mensajes dudosos en los desencadenadores de orquestación.
* **Visibilidad de los mensajes**: los mensajes de desencadenador de orquestación se quitan de la cola y se mantienen invisibles durante un tiempo configurable. La visibilidad de estos mensajes se renueva automáticamente siempre que la aplicación de la función se esté ejecutando y se mantenga correcta.
* **Valores devueltos**: los valores devueltos se serializan en JSON y se conservan en la tabla de historial de orquestación en Azure Table Storage. Estos valores devueltos pueden ser consultados por el enlace de cliente de orquestación que se describe más adelante.

> [!WARNING]
> Las funciones del orquestador nunca deben usar ningún enlace de entrada o salida que no sea el enlace de desencadenador de orquestación. Si lo hacen, existe la posibilidad de que se produzcan problemas con la extensión Durable Task porque esos enlaces pueden no seguir las reglas de E/S y subprocesamiento único. Si desea usar otros enlaces, agréguelos a una función de actividad llamada desde la función de orquestador.

> [!WARNING]
> Las funciones del orquestador de JavaScript nunca se deberían declarar `async`.

### <a name="trigger-usage-net"></a>Uso del desencadenador (.NET)

El enlace de desencadenador de orquestación admite entradas y salidas. Estas son algunas cosas que hay que saber acerca del control de entradas y salidas:

* **entradas**: las funciones de orquestación de .NET admiten solamente `DurableOrchestrationContext` como tipo de parámetro. No se admite deserialización de entradas directamente en la firma de función. El código debe usar el método `GetInput<T>` (.NET) o `getInput` (JavaScript) para capturar entradas de la función de orquestador. Estas entradas tienen que ser tipos serializables con JSON.
* **salidas**: los desencadenadores de orquestación admiten valores de salida, así como de entrada. El valor devuelto de la función se utiliza para asignar el valor de salida y tiene que ser serializable con JSON. Si una función .NET devuelve `Task` o `void`, un valor `null` se guardará como salida.

### <a name="trigger-sample"></a>Ejemplo de desencadenador

El código de ejemplo siguiente muestra el aspecto que podría tener la función del orquestador más simple "Hola mundo":

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static string Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    return $"Hello {name}!";
}
```
> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

#### <a name="javascript-functions-20-only"></a>JavaScript (solo Functions 2.0)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    return `Hello ${name}!`;
});
```

> [!NOTE]
> El objeto `context` de JavaScript no representa a DurableOrchestrationContext sino al [contexto de la función en su conjunto](../functions-reference-node.md#context-object). Puede acceder a los métodos de la orquestación a través de la propiedad `context` del objeto `df`.

> [!NOTE]
> Los orquestadores de JavaScript deben usar `return`. La biblioteca `durable-functions` se encarga de llamar al método `context.done`.

La mayoría de las funciones del orquestador llaman a funciones de actividad, por lo que aquí tenemos un ejemplo de "Hola mundo" que muestra cómo llamar a una función de actividad:

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static async Task<string> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    string result = await context.CallActivityAsync<string>("SayHello", name);
    return result;
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

#### <a name="javascript-functions-20-only"></a>JavaScript (solo Functions 2.0)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    const result = yield context.df.callActivity("SayHello", name);
    return result;
});
```

## <a name="activity-trigger"></a>Desencadenador de actividad

El desencadenador de actividad le permite crear funciones que las funciones del orquestador llaman, conocidas como [funciones de actividad](durable-functions-types-features-overview.md#activity-functions).

Si usa Visual Studio, el desencadenador de actividad se configura mediante el atributo `ActivityTriggerAttribute`de .NET.

Si utiliza VS Code o Azure Portal para el desarrollo, el desencadenador de actividad está definido por el objeto JSON siguiente en la matriz `bindings` de *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "activity": "<Optional - name of the activity>",
    "type": "activityTrigger",
    "direction": "in"
}
```

* `activity`es el nombre de la actividad. Este valor es el nombre que utilizan las funciones del orquestador para invocar esta función de actividad. Esta propiedad es opcional. Si no se especifica, se utiliza el nombre de la función.

Internamente este enlace de desencadenador sondea una cola de la cuenta de almacenamiento predeterminado para la aplicación de la función. Esta cola es un detalle de implementación interno de la extensión, esta es la razón de que no se configure explícitamente en las propiedades de enlace.

### <a name="trigger-behavior"></a>Comportamiento de un desencadenador

Estas son algunas notas acerca del desencadenador de actividad:

* **Subprocesamiento**: a diferencia del desencadenador de orquestación, los desencadenadores de actividad no tienen ninguna restricción en relación a los subprocesos o E/S. Se puede tratar como funciones normales.
* **Control de mensajes dudosos**: no hay compatibilidad con mensajes dudosos en los desencadenadores de actividad.
* **Visibilidad de los mensajes**: los mensajes de desencadenador de actividad se quitan de la cola y se mantienen invisibles durante un tiempo configurable. La visibilidad de estos mensajes se renueva automáticamente siempre que la aplicación de la función se esté ejecutando y se mantenga correcta.
* **Valores devueltos**: los valores devueltos se serializan en JSON y se conservan en la tabla de historial de orquestación en Azure Table Storage.

> [!WARNING]
> El back-end de almacenamiento para las funciones de actividad es un detalle de implementación y el código de usuario no debe interactuar directamente con estas entidades de almacenamiento.

### <a name="trigger-usage-net"></a>Uso del desencadenador (.NET)

El enlace de desencadenador de actividad admite entradas y salidas como el desencadenador de orquestación. Estas son algunas cosas que hay que saber acerca del control de entradas y salidas:

* **entradas**: las funciones de actividad de .NET usan de forma nativa `DurableActivityContext` como tipo de parámetro. De forma alternativa, una función de actividad puede declararse con cualquier tipo de parámetro que se puede serializar con JSON. Cuando usa `DurableActivityContext`, puede llamar a `GetInput<T>` para capturar y deserializar la entrada de la función de actividad.
* **salidas**: las funciones de actividad admiten valores de salida, así como de entrada. El valor devuelto de la función se utiliza para asignar el valor de salida y tiene que ser serializable con JSON. Si una función .NET devuelve `Task` o `void`, un valor `null` se guardará como salida.
* **metadatos**: las funciones de actividad de .NET pueden enlazar a un parámetro `string instanceId` para obtener el identificador de instancia de la orquestación primaria.

### <a name="trigger-sample"></a>Ejemplo de desencadenador

El ejemplo de código siguiente muestra el aspecto que podría tener una función de actividad simple "Hola mundo":

#### <a name="c"></a>C#

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] IDurableActivityContext helloContext)
{
    string name = helloContext.GetInput<string>();
    return $"Hello {name}!";
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableActivityContext` en lugar de `IDurableActivityContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

El tipo de parámetro predeterminado para el enlace de .NET `ActivityTriggerAttribute` es `IDurableActivityContext`. Sin embargo, los desencadenadores de actividad de .NET también admiten enlazar directamente con tipos serializables con JSON (incluidos los tipos primitivos), por lo que la misma función podría simplificarse como sigue:

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] string name)
{
    return $"Hello {name}!";
}
```

#### <a name="javascript-functions-20-only"></a>JavaScript (solo Functions 2.0)

```javascript
module.exports = async function(context) {
    return `Hello ${context.bindings.name}!`;
};
```

Los enlaces de JavaScript también se pueden pasar como parámetros adicionales, por lo que la misma función podría simplificarse como sigue:

```javascript
module.exports = async function(context, name) {
    return `Hello ${name}!`;
};
```


### <a name="using-input-and-output-bindings"></a>Uso de enlaces de entrada y de salida

Puede usar los enlaces de entrada y salida normales además del enlace de desencadenador de actividad. Por ejemplo, puede tomar la entrada del enlace de actividad y enviar un mensaje a un objeto EventHub mediante el enlace de salida de EventHub:

```json
{
  "bindings": [
    {
      "name": "message",
      "type": "activityTrigger",
      "direction": "in"
    },
    {
      "type": "eventHub",
      "name": "outputEventHubMessage",
      "connection": "EventhubConnectionSetting",
      "eventHubName": "eh_messages",
      "direction": "out"
  }
  ]
}
```

```javascript
module.exports = async function (context) {
    context.bindings.outputEventHubMessage = context.bindings.message;
};
```

## <a name="orchestration-client"></a>Cliente de orquestación

El enlace del cliente de orquestación le permite escribir funciones que interactúan con las funciones del orquestador. Estas funciones se conocen a veces como [funciones de cliente](durable-functions-types-features-overview.md#client-functions). Por ejemplo, puede actuar en las instancias de orquestación de las siguientes formas:

* Iniciarlas.
* Consultar su estado.
* Finalizarlas.
* Enviarles eventos mientras se están ejecutando.
* Purgar del historial de instancias.

Si usa Visual Studio, puede enlazar con el cliente de orquestación mediante el atributo `OrchestrationClientAttribute` de .NET para Durable Functions 1.0. A partir de Durable Functions 2.0, puede enlazar con el cliente de orquestación mediante el atributo `DurableClientAttribute` de .NET.

Si utiliza lenguajes de scripting (por ejemplo, archivos *.csx* o *.js*) para desarrollar, el desencadenador de orquestación está definido por el objeto JSON siguiente en la matriz `bindings` de *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "orchestrationClient",
    "direction": "in"
}
```

* `taskHub`: se usa en escenarios donde varias aplicaciones de función comparten la misma cuenta de almacenamiento, pero tienen que estar aisladas entre sí. Si no se especifica, se usa el valor predeterminado `host.json`. Este valor tiene que coincidir con el valor usado por las funciones de orquestador de destino.
* `connectionName`: nombre de una configuración de aplicación que contiene una cadena de conexión de cuenta de almacenamiento. La cuenta de almacenamiento representada por esta cadena de conexión tiene que ser la misma que utilizan las funciones del orquestador de destino. Si no se especifica, se utiliza la cadena de conexión de cuenta de almacenamiento predeterminada de la aplicación de función.

> [!NOTE]
> En la mayoría de los casos, se recomienda omitir estas propiedades y confiar en el comportamiento predeterminado.

### <a name="client-usage"></a>Uso del cliente

En las funciones de .NET, habitualmente se enlaza con `IDurableOrchestrationClient`, lo que proporciona acceso completo a todas las API de cliente de orquestación compatibles con Durable Functions. En las versiones anteriores de Durable Functions 2.x, se enlaza con la clase `DurableOrchestrationClient`. En JavaScript, el objeto que se devuelve desde `getClient` expone las mismas API. Las API en el objeto de cliente incluyen:

* `StartNewAsync`
* `GetStatusAsync`
* `TerminateAsync`
* `RaiseEventAsync`
* `PurgeInstanceHistoryAsync`
* `CreateCheckStatusResponse`
* `CreateHttpManagementPayload`

Como alternativa, las funciones de .NET pueden enlazar con `IAsyncCollector<T>`, donde `T` es `StartOrchestrationArgs` o `JObject`.

Para obtener más información sobre estas operaciones, vea la documentación de la API `IDurableOrchestrationClient`.

### <a name="client-sample-visual-studio-development"></a>Ejemplo de cliente (Desarrollo de Visual Studio)

Aquí tenemos un ejemplo de una función desencadenada por la cola que inicia una orquestación "HelloWorld".

```csharp
[FunctionName("QueueStart")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableOrchestrationClient starter)
{
    // Orchestration input comes from the queue message content.
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> El código de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar el atributo `OrchestrationClient` en lugar del atributo `DurableClient`, además de usar el tipo de parámetro `DurableOrchestrationClient` en lugar de `IDurableOrchestrationClient`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

### <a name="client-sample-not-visual-studio"></a>Ejemplo de cliente (no de Visual Studio)

Si no está usando Visual Studio para el desarrollo, puede crear el siguiente archivo *function.json*. En este ejemplo se muestra cómo configurar una función desencadenada por la cola que utiliza el enlace del cliente de orquestación durable:

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "queueTrigger",
      "queueName": "durable-function-trigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ]
}
```

> [!NOTE]
> El JSON anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `orchestrationClient` en lugar de `durableClient` como tipo de desencadenador. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

A continuación se encuentran ejemplos específicos del idioma que inician nuevas instancias de función de orquestador.

#### <a name="c-script-sample"></a>Ejemplo de script de C#

En el ejemplo siguiente se muestra cómo usar el enlace de cliente de orquestación durable para iniciar una nueva instancia de función desde una función de C# desencadenada por cola:

```csharp
#r "Microsoft.Azure.WebJobs.Extensions.DurableTask"

using Microsoft.Azure.WebJobs.Extensions.DurableTask;

public static Task Run(string input, IDurableOrchestrationClient starter)
{
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar el tipo de parámetro `DurableOrchestrationClient` en lugar de `IDurableOrchestrationClient`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

#### <a name="javascript-sample"></a>Ejemplo de JavaScript

El ejemplo siguiente muestra cómo utilizar el enlace de cliente de orquestación durable para iniciar una nueva instancia de función desde una función de JavaScript:

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    return instanceId = await client.startNew("HelloWorld", undefined, context.bindings.input);
};
```

Pueden encontrar más información acerca de cómo iniciar instancias en [Administración de instancias](durable-functions-instance-management.md).

## <a name="entity-trigger"></a>Desencadenador de entidad

Los desencadenadores de entidad permiten crear [funciones de entidad](durable-functions-entities.md). Este desencadenador admite el procesamiento de eventos para una instancia de entidad específica.

Cuando se usa Visual Studio Tools para Azure Functions, el desencadenador de entidad se configura con el atributo `EntityTriggerAttribute` de .NET.

> [!NOTE]
> Los desencadenadores de entidad están disponibles a partir de Durable Functions 2.x.

Internamente este enlace de desencadenador sondea una serie de colas de la cuenta de almacenamiento predeterminado para la aplicación de la función. Estas colas son detalles de implementación internos de la extensión, esta es la razón de que no se configuren explícitamente en las propiedades de enlace.

### <a name="trigger-behavior"></a>Comportamiento de un desencadenador

Estas son algunas notas sobre el desencadenador de entidad:

* **De un único subproceso**: un único subproceso de distribuidor se utiliza para procesar las operaciones de una entidad determinada. Si se envían varios mensajes a una sola entidad de manera simultánea, las operaciones se procesarán una a una.
* **Control de mensajes dudosos**: no hay compatibilidad con mensajes dudosos en los desencadenadores de entidad.
* **Visibilidad de los mensajes**: los mensajes de desencadenador de entidad se quitan de la cola y se mantienen invisibles durante un tiempo configurable. La visibilidad de estos mensajes se renueva automáticamente siempre que la aplicación de la función se esté ejecutando y se mantenga correcta.
* **Valores devueltos**: las funciones de entidad no admiten valores devueltos. Hay API específicas que se pueden usar para guardar el estado o pasar valores devuelta a las orquestaciones.

Los cambios de estado realizados en una entidad durante su ejecución se conservarán automáticamente una vez completada la ejecución.

### <a name="trigger-usage-net"></a>Uso del desencadenador (.NET)

Cada función de entidad tiene un tipo de parámetro de `IDurableEntityContext`, que tiene los miembros siguientes:

* **EntityName**: el nombre de la entidad que se está ejecutando actualmente.
* **EntityKey**: la clave de la entidad que se está ejecutando actualmente.
* **EntityId**: el identificador de la entidad que se está ejecutando actualmente.
* **OperationName**: el nombre de la operación actual.
* **HasState**: indica si la entidad existe, es decir, tiene algún estado. 
* **GetState\<TState>()** : obtiene el estado actual de la entidad. Si no existe, se crea y se inicializa en `default<TState>`. El parámetro `TState` debe ser un tipo primitivo o serializable de JSON. 
* **GetState\<TState>(initfunction)** : obtiene el estado actual de la entidad. Si aún no existe, se crea llamando al parámetro `initfunction` que se proporciona. El parámetro `TState` debe ser un tipo primitivo o serializable de JSON. 
* **SetState(arg)** : crea o actualiza el estado de la entidad. El parámetro `arg` debe ser un objeto serializable de JSON o primitivo.
* **DeleteState()** : elimina el estado de la entidad. 
* **GetInput\<TInput>()** : obtiene la entrada de la operación actual. El parámetro de tipo `TInput` debe ser un tipo primitivo o serializable de JSON.
* **Return(arg)** : devuelve un valor a la orquestación que ha llamado a la operación. El parámetro `arg` debe ser un objeto primitivo o serializable de JSON.
* **SignalEntity(EntityId, scheduledTimeUtc, operation, input)** : envía un mensaje unidireccional a una entidad. El parámetro `operation` debe ser una cadena que no sea NULL, el parámetro `scheduledTimeUtc` opcional debe ser una fecha y hora UTC en la que invocar la operación y el parámetro `input` debe ser un objeto primitivo o serializable de JSON.
* **CreateNewOrchestration(orchestratorFunctionName, input)** : inicia una nueva orquestación. El parámetro `input` debe ser un objeto primitivo o serializable de JSON.

Se puede acceder al objeto `IDurableEntityContext` pasado a la función de entidad mediante la propiedad async-local `Entity.Current`. Este enfoque es práctico cuando se usa el modelo de programación basado en clases.

### <a name="trigger-sample-c-function-based-syntax"></a>Ejemplo de desencadenador (sintaxis basada en función de C#)

El código siguiente es un ejemplo de una entidad *Counter* simple implementada como una función duradera. Esta función define tres operaciones (`add`, `reset` y `get`), cada una de las cuales opera en un valor de estado entero.

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            ctx.SetState(ctx.GetState<int>() + ctx.GetInput<int>());
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(ctx.GetState<int>()));
            break;
    }
}
```

Para más información sobre la sintaxis basada en funciones y cómo utilizarla, consulte [Sintaxis basada en funciones](durable-functions-dotnet-entities.md#function-based-syntax).

### <a name="trigger-sample-c-class-based-syntax"></a>Ejemplo de desencadenador (sintaxis basada en clase de C#)

El ejemplo siguiente es una implementación equivalente de la entidad `Counter` que usa clases y métodos.

```csharp
[JsonObject(MemberSerialization.OptIn)]
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

El estado de esta entidad es un objeto de tipo `Counter`, que contiene un campo que almacena el valor actual del contador. Para conservar este objeto en el almacenamiento, la biblioteca [Json.NET](https://www.newtonsoft.com/json) lo serializa y deserializa. 

Para más información sobre la sintaxis basada en clases y cómo utilizarla, consulte [Definición de las clases de entidad](durable-functions-dotnet-entities.md#defining-entity-classes).

> [!NOTE]
> El método de punto de entrada de la función con el atributo `[FunctionName]` se *debe* declarar `static` cuando se usan clases de entidad. Los métodos de punto de entrada no estáticos pueden dar lugar a la inicialización de varios objetos y, potencialmente, a otros comportamientos no definidos.

Las clases de entidad tienen mecanismos especiales para interactuar con los enlaces y la inserción de dependencias de .NET. Para más información, consulte [Construcción de entidades](durable-functions-dotnet-entities.md#entity-construction).

### <a name="trigger-sample-javascript"></a>Ejemplo de desencadenador (JavaScript)

El código siguiente es un ejemplo de una entidad *Counter* simple implementada como una función durable escrita en JavaScript. Esta función define tres operaciones (`add`, `reset` y `get`), cada una de las cuales opera en un valor de estado entero.

**function.json**
```json
{
  "bindings": [
    {
      "name": "context",
      "type": "entityTrigger",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

**index.js**
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

> [!NOTE]
> Las entidades durables están disponibles en JavaScript a partir de la versión **1.3.0** del paquete de npm `durable-functions`.

## <a name="entity-client"></a>Cliente de entidad

El enlace de cliente de entidad permite desencadenar de manera asincrónica las [funciones de entidad](#entity-trigger). Estas funciones se conocen a veces como [funciones de cliente](durable-functions-types-features-overview.md#client-functions).

Si usa Visual Studio, puede enlazar con el cliente de entidad mediante el atributo `DurableClientAttribute` de .NET.

> [!NOTE]
> `[DurableClientAttribute]` también se puede usar para enlazar con el [cliente de orquestación](#orchestration-client).

Si utiliza lenguajes de scripting (por ejemplo, archivos *.csx* o *.js*) para desarrollar, el desencadenador de entidad está definido por el objeto JSON siguiente en la matriz `bindings` de *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "durableClient",
    "direction": "in"
}
```

* `taskHub`: se usa en escenarios donde varias aplicaciones de función comparten la misma cuenta de almacenamiento, pero tienen que estar aisladas entre sí. Si no se especifica, se usa el valor predeterminado `host.json`. Este valor tiene que coincidir con el valor usado por las funciones de entidad de destino.
* `connectionName`: nombre de una configuración de aplicación que contiene una cadena de conexión de cuenta de almacenamiento. La cuenta de almacenamiento representada por esta cadena de conexión tiene que ser la misma que utilizan las funciones de entidad de destino. Si no se especifica, se utiliza la cadena de conexión de cuenta de almacenamiento predeterminada de la aplicación de función.

> [!NOTE]
> En la mayoría de los casos, se recomienda omitir las propiedades opcionales y confiar en el comportamiento predeterminado.

### <a name="entity-client-usage"></a>Uso del cliente de entidad

En funciones de .NET, habitualmente se enlaza a `IDurableEntityClient`, lo que proporciona acceso completo a todas las API de cliente compatibles con las entidades duraderas. También puede enlazar a la interfaz `IDurableOrchestrationClient`, que proporciona acceso a las API de cliente, tanto para las entidades como para las orquestaciones. Las API en el objeto de cliente incluyen:

* **ReadEntityStateAsync\<T>** : lee el estado de una entidad. Devuelve una respuesta que indica si la entidad de destino existe y, si es así, cuál es su estado.
* **SignalEntityAsync**: envía un mensaje unidireccional a una entidad y espera a que se ponga en cola.
* **ListEntitiesAsync**: consulta el estado de varias entidades. Las entidades se pueden consultar por *nombre* y *hora de la última operación*.

No es necesario crear la entidad de destino antes de enviar una señal: el estado de la entidad se puede crear desde dentro de la función de entidad que controla la señal.

> [!NOTE]
> Es importante comprender que las "señales" enviadas desde el cliente simplemente se ponen en cola y se procesan de forma asincrónica en un momento posterior. En concreto, `SignalEntityAsync` normalmente se devuelve antes de que la entidad inicie incluso la operación y no es posible recuperar el valor devuelto ni observar las excepciones. Si se requieren garantías más seguras (por ejemplo, para flujos de trabajo), se deben usar las *funciones del orquestador*, que pueden esperar a que se completen las operaciones de entidad, y pueden procesar los valores devueltos y observar las excepciones.

### <a name="example-client-signals-entity-directly---c"></a>Ejemplo: el cliente señala la entidad directamente (C#)

Este es un ejemplo de una función desencadenada por la cola que invoca una entidad "Counter".

```csharp
[FunctionName("AddFromQueue")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableEntityClient client)
{
    // Entity operation input comes from the queue message content.
    var entityId = new EntityId(nameof(Counter), "myCounter");
    int amount = int.Parse(input);
    return client.SignalEntityAsync(entityId, "Add", amount);
}
```

### <a name="example-client-signals-entity-via-interface---c"></a>Ejemplo: el cliente señala la entidad mediante la interfaz (C#)

Siempre que sea posible, se recomienda [acceder a las entidades mediante interfaces](durable-functions-dotnet-entities.md#accessing-entities-through-interfaces), ya que así se proporciona una comprobación de más tipos. Por ejemplo, supongamos que la entidad `Counter` mencionada anteriormente implementó una interfaz `ICounter`, definida de la siguiente manera:

```csharp
public interface ICounter
{
    void Add(int amount);
    void Reset();
    Task<int> Get();
}

public class Counter : ICounter
{
    // ...
}
```

Después, el código de cliente puede usar `SignalEntityAsync<ICounter>` para generar un proxy con seguridad de tipos:

```csharp
[FunctionName("UserDeleteAvailable")]
public static async Task AddValueClient(
    [QueueTrigger("my-queue")] string message,
    [DurableClient] IDurableEntityClient client)
{
    var target = new EntityId(nameof(Counter), "myCounter");
    int amount = int.Parse(message);
    await client.SignalEntityAsync<ICounter>(target, proxy => proxy.Add(amount));
}
```

El parámetro `proxy` es una instancia generada dinámicamente de `ICounter`, que traslada internamente la llamada a `Add` a la llamada equivalente (sin tipo) a `SignalEntityAsync`.

> [!NOTE]
> Las API `SignalEntityAsync` representan operaciones unidireccionales. Si una interfaz de identidad devuelve `Task<T>`, el valor del parámetro `T` siempre será NULL o `default`.

Especialmente, no tiene sentido señalar la operación `Get`, ya que no se devuelve ningún valor. En cambio, los clientes pueden usar `ReadStateAsync` para acceder al estado del contador directamente o pueden iniciar una función de orquestador que llame a la operación `Get`.

### <a name="example-client-signals-entity---javascript"></a>Ejemplo: el cliente señala la entidad (JavaScript)

Este es un ejemplo de una función desencadenada por la cola que señala una entidad "Counter" en JavaScript.

**function.json**
```json
{
    "bindings": [
      {
        "name": "input",
        "type": "queueTrigger",
        "queueName": "durable-entity-trigger",
        "direction": "in",
      },
      {
        "name": "starter",
        "type": "durableClient",
        "direction": "in"
      }
    ],
    "disabled": false
  }
```

**index.js**
```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await context.df.signalEntity(entityId, "add", 1);
};
```

> [!NOTE]
> Las entidades durables están disponibles en JavaScript a partir de la versión **1.3.0** del paquete de npm `durable-functions`.

<a name="host-json"></a>
## <a name="hostjson-settings"></a>configuración de host.json

[!INCLUDE [durabletask](../../../includes/functions-host-json-durabletask.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de la API HTTP integrada para la administración de instancias](durable-functions-http-api.md)
