---
title: 'Guía del desarrollador de entidades duraderas en .NET: Azure Functions'
description: Aprenda a trabajar con entidades duraderas en .NET con la extensión Durable Functions para Azure Functions.
author: sebastianburckhardt
ms.topic: conceptual
ms.date: 10/06/2019
ms.author: azfuncdf
ms.openlocfilehash: d480b8db69b34eda7ca1ea8e1b2755179f9c673f
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88055180"
---
# <a name="developers-guide-to-durable-entities-in-net"></a>Guía del desarrollador de entidades duraderas en .NET

En este artículo, se describen de forma detallada las interfaces disponibles para desarrollar entidades duraderas con .NET, y se incluyen ejemplos y consejos generales. 

Las funciones de entidad proporcionan a los desarrolladores de aplicaciones sin servidor una manera cómoda de organizar el estado de la aplicación como una colección de entidades específicas. Para más información sobre los conceptos subyacentes, consulte el artículo [Entidades duraderas: Conceptos](durable-functions-entities.md).

Actualmente se ofrecen dos API para definir las entidades:

- La **sintaxis basada en clases** representa entidades y operaciones como clases y métodos. Esta sintaxis genera código fácilmente legible y permite invocar operaciones con comprobación de tipos mediante interfaces. 

- La **sintaxis basada en funciones** es una interfaz de nivel inferior que representa a las entidades como funciones. Proporciona un control preciso sobre cómo se distribuyen las operaciones de entidad y cómo se administra el estado de la entidad.  

Este artículo se centra principalmente en la sintaxis basada en clases, ya que se considera que es la más adecuada para la mayoría de las aplicaciones. Sin embargo, la [sintaxis basada en funciones](#function-based-syntax) puede ser adecuada para las aplicaciones que quieran definir o administrar sus propias abstracciones para el estado y las operaciones de la entidad. Además, puede ser conveniente para implementar bibliotecas que necesitan tipos genéricos que actualmente no se admiten con la sintaxis basada en clases. 

> [!NOTE]
> La sintaxis basada en clases es simplemente una capa encima de la sintaxis basada en funciones, por lo que ambas variantes se pueden usar indistintamente en la misma aplicación. 
 
## <a name="defining-entity-classes"></a>Definición de clases de entidad

El ejemplo siguiente es una implementación de una entidad `Counter` que almacena un valor único de tipo entero y ofrece cuatro operaciones: `Add`, `Reset`, `Get` y `Delete`.

```csharp
[JsonObject(MemberSerialization.OptIn)]
public class Counter
{
    [JsonProperty("value")]
    public int Value { get; set; }

    public void Add(int amount) 
    {
        this.Value += amount;
    }

    public Task Reset() 
    {
        this.Value = 0;
        return Task.CompletedTask;
    }

    public Task<int> Get() 
    {
        return Task.FromResult(this.Value);
    }

    public void Delete() 
    {
        Entity.Current.DeleteState();
    }

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}
```

La función `Run` contiene el texto reutilizable necesario para usar la sintaxis basada en clases. Debe ser una función de Azure *estática*. Se ejecuta una vez por cada mensaje de operación que procesa la entidad. Cuando se llama a `DispatchAsync<T>` y la entidad no está aún en memoria, construye un objeto de tipo `T` y rellena sus campos con el último código de JSON guardado encontrado en el almacenamiento (si existe). A continuación, invoca el método con el nombre coincidente.

> [!NOTE]
> El estado de una entidad basada en clases se **crear implícitamente** antes de que la entidad procese una operación y se puede **eliminar** explícitamente en una operación mediante la llamada a `Entity.Current.DeleteState()`.

### <a name="class-requirements"></a>Requisitos de las clases
 
Las clases de entidad son POCO (objetos CLR estándar sin formato) que no requieren superclases, interfaces ni atributos especiales. Pero:

- La clase debe poder construirse (consulte [Construcción de entidades](#entity-construction)).
- La clase debe ser código de JSON serializable (consulte [Serialización de entidades](#entity-serialization)).

Además, cualquier método que se vaya a invocar como una operación debe cumplir requisitos adicionales:

- Una operación debe tener como máximo un argumento y no debe tener ninguna sobrecarga ni argumentos de tipo genérico.
- Una operación diseñada para llamarse desde una orquestación mediante una interfaz debe devolver `Task` o `Task<T>`.
- Los argumentos y los valores devueltos deben ser valores u objetos serializables.

### <a name="what-can-operations-do"></a>¿Qué pueden hacer las operaciones?

Todas las operaciones de entidad pueden leer y actualizar el estado de la entidad, y los cambios en el estado se conservan automáticamente en el almacenamiento. Además, las operaciones pueden realizar cálculos de E/S externos y de otro tipo dentro de los límites generales comunes a todas las funciones de Azure.

Las operaciones también tienen acceso a la funcionalidad proporcionada por el contexto de `Entity.Current`:

* `EntityName`: el nombre de la entidad que se ejecuta actualmente.
* `EntityKey`: la clave de la entidad que se ejecuta actualmente.
* `EntityId`: el identificador de la entidad que se ejecuta actualmente (incluye el nombre y la clave).
* `SignalEntity`: envía un mensaje unidireccional a una entidad.
* `CreateNewOrchestration`: inicia una nueva orquestación.
* `DeleteState`: elimina el estado de esta entidad.

Por ejemplo, se puede modificar la entidad de contador para que inicie una orquestación cuando el contador llegue a 100 y pase el identificador de la entidad como argumento de entrada:

```csharp
    public void Add(int amount) 
    {
        if (this.Value < 100 && this.Value + amount >= 100)
        {
            Entity.Current.StartNewOrchestration("MilestoneReached", Entity.Current.EntityId);
        }
        this.Value += amount;      
    }
```

## <a name="accessing-entities-directly"></a>Acceso directo a las entidades

Se puede acceder directamente a las entidades basadas en clases mediante nombres de cadena explícitos para la entidad y sus operaciones. A continuación se proporcionan algunos ejemplos; para una explicación más detallada de los conceptos subyacentes (por ejemplo, la diferencia entre señales y llamadas), consulte el análisis en [Acceso a entidades](durable-functions-entities.md#access-entities). 

> [!NOTE]
> Siempre que sea posible, se recomienda [acceder a las entidades mediante interfaces](#accessing-entities-through-interfaces), ya que así se proporciona una comprobación de más tipos.

### <a name="example-client-signals-entity"></a>Ejemplo: El cliente señala la entidad

La siguiente función http de Azure implementa una operación DELETE mediante convenciones de REST. Envía una señal de eliminación a la entidad de contador cuya clave se pasa en la ruta de acceso a la dirección URL.

```csharp
[FunctionName("DeleteCounter")]
public static async Task<HttpResponseMessage> DeleteCounter(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    await client.SignalEntityAsync(entityId, "Delete");    
    return req.CreateResponse(HttpStatusCode.Accepted);
}
```

### <a name="example-client-reads-entity-state"></a>Ejemplo: El cliente lee el estado de la entidad

La siguiente función http de Azure implementa una operación GET mediante convenciones de REST. Lee el estado actual de la entidad de contador cuya clave se pasa en la ruta de acceso a la dirección URL.

```csharp
[FunctionName("GetCounter")]
public static async Task<HttpResponseMessage> GetCounter(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    var state = await client.ReadEntityStateAsync<Counter>(entityId); 
    return req.CreateResponse(state);
}
```

> [!NOTE]
> El objeto devuelto por `ReadEntityStateAsync` es simplemente una copia local, es decir, una instantánea del estado de la entidad de un momento anterior en el tiempo. En concreto, puede ser obsoleto, y la modificación de este objeto no tiene efecto sobre la entidad real. 

### <a name="example-orchestration-first-signals-then-calls-entity"></a>Ejemplo: La orquestación primero señala y luego llama a la entidad

La siguiente orquestación señala una entidad de contador para incrementarla y, luego, llama a la misma entidad para leer su último valor.

```csharp
[FunctionName("IncrementThenGet")]
public static async Task<int> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId("Counter", "myCounter");

    // One-way signal to the entity - does not await a response
    context.SignalEntity(entityId, "Add", 1);

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await context.CallEntityAsync<int>(entityId, "Get");

    return currentValue;
}
```

## <a name="accessing-entities-through-interfaces"></a>Acceso a las entidades mediante interfaces

Se pueden usar interfaces para acceder a las entidades mediante los objetos proxy generados. Este enfoque garantiza que el nombre y el tipo de argumento de una operación coinciden con lo que se implementa. Siempre que sea posible, se recomienda usar interfaces para acceder a las entidades.

Por ejemplo, se puede modificar el ejemplo de contador de la siguiente manera:

```csharp
public interface ICounter
{
    void Add(int amount);
    Task Reset();
    Task<int> Get();
    void Delete();
}

public class Counter : ICounter
{
    ...
}
```

Las clases de entidad y las interfaces de entidad son similares a los granos y las interfaces de granos popularizados por [Orleans ](https://www.microsoft.com/research/project/orleans-virtual-actors/). Para más información sobre las similitudes y las diferencias entre las entidades duraderas y Orleans, consulte [Comparación con actores virtuales](durable-functions-entities.md#comparison-with-virtual-actors).

Además de proporcionar comprobación de tipos, las interfaces son útiles para separar mejor los problemas dentro de la aplicación. Por ejemplo, puesto que una entidad puede implementar varias interfaces, una sola entidad puede atender a varios roles. Además, como varias entidades pueden implementar una interfaz, se pueden implementar patrones de comunicación generales como bibliotecas reutilizables.

### <a name="example-client-signals-entity-through-interface"></a>Ejemplo: El cliente señala la entidad mediante la interfaz

El código de cliente puede usar `SignalEntityAsync<TEntityInterface>` para enviar señales a las entidades que implementan `TEntityInterface`. Por ejemplo:

```csharp
[FunctionName("DeleteCounter")]
public static async Task<HttpResponseMessage> DeleteCounter(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    await client.SignalEntityAsync<ICounter>(entityId, proxy => proxy.Delete());    
    return req.CreateResponse(HttpStatusCode.Accepted);
}
```

En el ejemplo anterior, el parámetro `proxy` es una instancia generada dinámicamente de `ICounter`, que convierte internamente la llamada a `Delete` en una señal.

> [!NOTE]
> Las API de `SignalEntityAsync` solo se pueden usar para operaciones unidireccionales. Aunque una operación devuelva `Task<T>`, el valor del parámetro `T` siempre será null o `default`, no el resultado real.
Por ejemplo, no tiene sentido señalar la operación `Get`, ya que no se devuelve ningún valor. En cambio, los clientes pueden usar `ReadStateAsync` para acceder al estado del contador directamente o pueden iniciar una función de orquestador que llame a la operación `Get`. 

### <a name="example-orchestration-first-signals-then-calls-entity-through-proxy"></a>Ejemplo: La orquestación primero señala y luego llama a la entidad mediante un proxy

Para llamar a una entidad o señalarla desde una orquestación, se puede usar `CreateEntityProxy`, junto con el tipo de interfaz, para generar un proxy para la entidad. Este proxy se puede usar luego para llamar a las operaciones o señalarlas:

```csharp
[FunctionName("IncrementThenGet")]
public static async Task<int> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId("Counter", "myCounter");
    var proxy = context.CreateEntityProxy<ICounter>(entityId);

    // One-way signal to the entity - does not await a response
    proxy.Add(1);

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await proxy.Get();

    return currentValue;
}
```

De manera implícita, se señalan las operaciones que devuelven `void` y se llama a todas las operaciones que devuelvan `Task` o `Task<T>`. Se puede cambiar este comportamiento predeterminado y las operaciones de señalización, incluso si devuelven tareas, usando el método `SignalEntity<IInterfaceType>` de forma explícita.

### <a name="shorter-option-for-specifying-the-target"></a>Opción más corta para especificar el destino

Al llamar a una entidad o señalarla mediante una interfaz, el primer argumento debe especificar la entidad de destino. El destino se puede especificar indicando el identificador de la entidad, o bien, en casos en los que solo haya una clase que implemente la entidad, solo la clave de la entidad:

```csharp
context.SignalEntity<ICounter>(new EntityId(nameof(Counter), "myCounter"), ...);
context.SignalEntity<ICounter>("myCounter", ...);
```

Si solo se especifica la clave de la entidad y no se puede encontrar una implementación única en tiempo de ejecución, se produce la excepción `InvalidOperationException`. 

### <a name="restrictions-on-entity-interfaces"></a>Restricciones sobre las interfaces de entidad

Como de costumbre, todos los parámetros y tipos de valor devuelto deben ser serializables con JSON. De lo contrario, se inician excepciones de serialización en tiempo de ejecución.

También se aplican algunas reglas adicionales:
* Las interfaces de entidad solo deben definir métodos.
* Las interfaces de entidad no deben contener parámetros genéricos.
* Los métodos de la interfaz de entidad no deben tener más de un parámetro.
* Los métodos de la interfaz de entidad deben devolver `void`, `Task` o `Task<T>`. 

Si se infringe alguna de estas reglas, se produce una excepción `InvalidOperationException` en tiempo de ejecución cuando la interfaz se usa como un argumento de tipo para `SignalEntity` o `CreateProxy`. En el mensaje de excepción se explica qué regla se infringió.

> [!NOTE]
> Los métodos de interfaz que devuelven `void` solo se pueden señalar (unidireccional), no llamar (bidireccionales). Los métodos de interfaz que devuelven `Task` o `Task<T>` se pueden llamar o señalar. Si se llaman, devuelven el resultado de la operación o vuelven a iniciar las excepciones producidas por esta. Sin embargo, cuando se señalan, no devuelven el resultado ni la excepción reales de la operación, sino solo el valor predeterminado.

## <a name="entity-serialization"></a>Serialización de entidades

Puesto que el estado de una entidad se conserva de forma duradera, la clase de entidad debe ser serializable. El entorno de ejecución de Durable Functions emplea la biblioteca [Json.NET](https://www.newtonsoft.com/json) con este fin; esta biblioteca admite una serie de directivas y atributos para controlar el proceso de serialización y deserialización. Los tipos de datos de C# más usados (incluidas las matrices y los tipos de colección) ya son serializables y se pueden usar fácilmente para definir el estado de las entidades duraderas.

Por ejemplo, Json.NET puede serializar y deserializar fácilmente la siguiente clase:

```csharp
[JsonObject(MemberSerialization = MemberSerialization.OptIn)]
public class User
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("yearOfBirth")]
    public int YearOfBirth { get; set; }

    [JsonProperty("timestamp")]
    public DateTime Timestamp { get; set; }

    [JsonProperty("contacts")]
    public Dictionary<Guid, Contact> Contacts { get; set; } = new Dictionary<Guid, Contact>();

    [JsonObject(MemberSerialization = MemberSerialization.OptOut)]
    public struct Contact
    {
        public string Name;
        public string Number;
    }

    ...
}
```

### <a name="serialization-attributes"></a>Atributos de serialización

En el ejemplo anterior, se decidió incluir varios atributos para que la serialización subyacente sea más visible:
- Se anotó la clase con `[JsonObject(MemberSerialization.OptIn)]` para recordarnos que la clase debe ser serializable y para conservar solo los miembros que están marcados explícitamente como propiedades JSON.
-  Se anotaron los campos que se van a conservar con `[JsonProperty("name")]` para recordarnos que un campo forma parte del estado de la entidad conservada y para especificar el nombre de la propiedad que se va a usar en la representación JSON.

Sin embargo, estos atributos no son necesarios; se permiten otras convenciones o atributos siempre que funcionen con Json.NET. Por ejemplo, se pueden usar atributos `[DataContract]` o no usar ningún atributo en absoluto:

```csharp
[DataContract]
public class Counter
{
    [DataMember]
    public int Value { get; set; }
    ...
}

public class Counter
{
    public int Value;
    ...
}
```

De forma predeterminada, el nombre de la clase *no* se almacena como parte de la representación JSON: es decir, se usa `TypeNameHandling.None` como configuración predeterminada. Este comportamiento predeterminado se puede invalidar mediante los atributos `JsonObject` o `JsonProperty`.

### <a name="making-changes-to-class-definitions"></a>Realización de cambios en las definiciones de clase

Es necesario tener cuidado al realizar cambios en una definición de clase después de que se haya ejecutado una aplicación, ya que el objeto JSON almacenado ya no puede coincidir con la nueva definición de clase. Aun así, a menudo es posible tratar correctamente con el cambio de los formatos de datos, siempre y cuando se entienda el proceso de deserialización que usa `JsonConvert.PopulateObject`.

A continuación se muestran algunos ejemplos de cambios y su efecto:

1. Si se agrega una nueva propiedad, que no está presente en el código de JSON almacenado, se adopta su valor predeterminado.
1. Si se quita una propiedad, que se encuentra en el código de JSON almacenado, se pierde el contenido anterior.
1. Si se cambia el nombre de una propiedad, el efecto es el mismo que si se quita el antiguo y se agrega uno nuevo.
1. Si se cambia el tipo de una propiedad, de forma que ya no se puede deserializar del código JSON almacenado, se produce una excepción.
1. Si se cambia el tipo de una propiedad, pero aún se puede deserializar del código JSON almacenado, así se hará.

Hay muchas opciones disponibles para personalizar el comportamiento de Json.NET. Por ejemplo, para forzar una excepción si el código de JSON almacenado contiene un campo que no existe en la clase, especifique el atributo `JsonObject(MissingMemberHandling = MissingMemberHandling.Error)`. También es posible escribir código personalizado para la deserialización que pueda leer código de JSON almacenado en formatos arbitrarios.

## <a name="entity-construction"></a>Construcción de entidades

En ocasiones se quiere ejercer más control sobre cómo se construyen los objetos de entidad. Ahora se van a describir varias opciones para cambiar el comportamiento predeterminado al construir objetos de entidad. 

### <a name="custom-initialization-on-first-access"></a>Inicialización personalizada en el primer acceso

En ocasiones, es necesario realizar una inicialización especial antes de enviar una operación a una entidad a la que nunca se ha accedido, o que se ha eliminado. Para especificar este comportamiento, se puede agregar una condicional delante de `DispatchAsync`:

```csharp
[FunctionName(nameof(Counter))]
public static Task Run([EntityTrigger] IDurableEntityContext ctx)
{
    if (!ctx.HasState)
    {
        ctx.SetState(...);
    }
    return ctx.DispatchAsync<Counter>();
}
```

### <a name="bindings-in-entity-classes"></a>Enlaces en clases de entidad

A diferencia de las funciones normales, los métodos de clase de entidad no tienen acceso directo a enlaces de entrada y salida. En su lugar, los datos de enlace se deben capturar en la declaración de la función de punto de entrada y, a continuación, se deben pasar al método `DispatchAsync<T>`. Cualquier objeto pasado a `DispatchAsync<T>` se pasará automáticamente al constructor de clases de la entidad como un argumento.

En el ejemplo siguiente se muestra cómo se puede poner a disposición de una entidad basada en clases una referencia a `CloudBlobContainer` desde el [enlace de entrada del blob](../functions-bindings-storage-blob-input.md).

```csharp
public class BlobBackedEntity
{
    [JsonIgnore]
    private readonly CloudBlobContainer container;

    public BlobBackedEntity(CloudBlobContainer container)
    {
        this.container = container;
    }

    // ... entity methods can use this.container in their implementations ...
    
    [FunctionName(nameof(BlobBackedEntity))]
    public static Task Run(
        [EntityTrigger] IDurableEntityContext context,
        [Blob("my-container", FileAccess.Read)] CloudBlobContainer container)
    {
        // passing the binding object as a parameter makes it available to the
        // entity class constructor
        return context.DispatchAsync<BlobBackedEntity>(container);
    }
}
```

Para más información sobre los enlaces en Azure Functions, consulte la documentación [Desencadenadores y enlaces de Azure Functions](../functions-triggers-bindings.md).

### <a name="dependency-injection-in-entity-classes"></a>Inserción de dependencias en las clases de entidad

Las clases de entidad admiten la [inserción de dependencias de Azure Functions](../functions-dotnet-dependency-injection.md). En el ejemplo siguiente se muestra cómo registrar un servicio `IHttpClientFactory` en una entidad basada en clases.

```csharp
[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();
        }
    }
}
```

En el fragmento de código siguiente se muestra cómo incorporar el servicio insertado en la clase de la entidad.

```csharp
public class HttpEntity
{
    [JsonIgnore]
    private readonly HttpClient client;

    public class HttpEntity(IHttpClientFactory factory)
    {
        this.client = factory.CreateClient();
    }

    public Task<int> GetAsync(string url)
    {
        using (var response = await this.client.GetAsync(url))
        {
            return (int)response.StatusCode;
        }
    }

    [FunctionName(nameof(HttpEntity))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<HttpEntity>();
}
```

> [!NOTE]
> Para evitar problemas con la serialización, asegúrese de excluir de esta los campos destinados a almacenar los valores insertados.

> [!NOTE]
> A diferencia de cuando se usa la inserción de constructores en Azure Functions para .NET normal, el método de punto de entrada de las funciones se *debe* declarar `static` para las entidades basadas en clases. Declarar un punto de entrada de función no estático puede producir conflictos entre el inicializador de objetos de Azure Functions normal y el inicializador de objetos de las entidades duraderas.

## <a name="function-based-syntax"></a>Sintaxis basada en funciones

Hasta ahora, nos hemos centrado en la sintaxis basada en clases, dado que se espera que sea más adecuada para la mayoría de las aplicaciones. Sin embargo, la sintaxis basada en funciones puede ser adecuada para las aplicaciones que quieran definir o administrar sus propias abstracciones en las operaciones y el estado de la entidad. Además, puede ser conveniente al implementar bibliotecas que necesitan tipos genéricos que actualmente no se admiten con la sintaxis basada en clases. 

Con la sintaxis basada en funciones, la función de entidad administra explícitamente el envío de operaciones y el estado de la entidad. Por ejemplo, en el código siguiente se muestra la entidad *Counter* implementada mediante la sintaxis basada en funciones.  

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
            ctx.Return(ctx.GetState<int>());
            break;
        case "delete":
            ctx.DeleteState();
            break;
    }
}
```

### <a name="the-entity-context-object"></a>Objeto de contexto de la entidad

Se puede acceder a la funcionalidad específica de la entidad mediante un objeto de contexto de tipo `IDurableEntityContext`. Este objeto de contexto está disponible como parámetro de la función de entidad y mediante la propiedad local asincrónica `Entity.Current`.

Los miembros siguientes proporcionan información sobre la operación actual y nos permiten especificar un valor devuelto. 

* `EntityName`: el nombre de la entidad que se ejecuta actualmente.
* `EntityKey`: la clave de la entidad que se ejecuta actualmente.
* `EntityId`: el identificador de la entidad que se ejecuta actualmente (incluye el nombre y la clave).
* `OperationName`: el nombre de la operación actual.
* `GetInput<TInput>()`: obtiene la entrada de la operación actual.
* `Return(arg)`: devuelve un valor a la orquestación que llamó a la operación.

Los miembros siguientes administran el estado de la entidad (crear, leer, actualizar, eliminar). 

* `HasState`: indica si la entidad existe, es decir, tiene algún estado. 
* `GetState<TState>()`: obtiene el estado actual de la entidad. Si no existe, se crea.
* `SetState(arg)`: crea o actualiza el estado de la entidad.
* `DeleteState()`: elimina el estado de la entidad, si existe. 

Si el estado que devuelve `GetState` es un objeto, se puede modificar directamente con el código de aplicación. No es necesario volver a llamar a `SetState` al final (pero no pasa nada por hacerlo). Si se llama varias veces a `GetState<TState>`, se debe usar el mismo tipo.

Por último, se usan los siguientes miembros para señalar otras entidades o iniciar nuevas orquestaciones:

* `SignalEntity(EntityId, operation, input)`: envía un mensaje unidireccional a una entidad.
* `CreateNewOrchestration(orchestratorFunctionName, input)`: inicia una nueva orquestación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los conceptos de entidad](durable-functions-entities.md)
