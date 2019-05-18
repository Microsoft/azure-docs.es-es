---
title: Obtener una vista previa en durable Functions características - Azure Functions
description: Obtenga información acerca de las características de vista previa para Durable Functions.
services: functions
author: cgillum
manager: jeconnoc
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.date: 04/23/2019
ms.author: azfuncdf
ms.openlocfilehash: 8ceb84ab9e9c41ff6a9cbde62571fb12ae67d790
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596085"
---
# <a name="durable-functions-20-preview-azure-functions"></a>Durable Functions 2.0 preview (Azure Functions)

*Durable Functions* es una extensión de [Azure Functions](../functions-overview.md) y [Azure WebJobs](../../app-service/web-sites-create-web-jobs.md) que le permite escribir funciones con estado en un entorno sin servidor. La extensión administra el estado, establece puntos de control y reinicia en su nombre. Si no todavía está familiarizado con Durable Functions, consulte el [documentación Descripción general de](durable-functions-overview.md).

Durable Functions es una característica de GA (disponible con carácter general) de Azure Functions, pero también contiene varias características secundarias que se encuentran actualmente en versión preliminar pública. En este artículo se describe las características más recientes de versión preliminar y entra en detalles sobre cómo funcionan y cómo puede comenzar a usarlos.

> [!NOTE]
> Estas características forman parte de una versión de Durable Functions 2.0, que es actualmente un **versión alfa de calidad** con varios cambios importantes. Azure funciones duraderas compila el paquete de extensión puede encontrarse en nuget.org con las versiones en forma de **2.0.0-alpha**. Estas compilaciones no son adecuadas para las cargas de trabajo de producción, y versiones posteriores pueden contener cambios adicionales.

## <a name="breaking-changes"></a>Cambios importantes

Se presentan varios cambios importantes en Durable Functions 2.0. No se esperan que las aplicaciones existentes para que sean compatibles con Durable Functions 2.0 sin cambios de código. En esta sección se enumera algunos de los cambios:

### <a name="dropping-net-framework-support"></a>Quitar la compatibilidad de .NET Framework

Se ha quitado la compatibilidad con .NET Framework (y, por tanto, las funciones 1.0) para Durable Functions 2.0. Es la razón principal habilitar los colaboradores que no sean Windows fácilmente compilar y probar los cambios que realicen a Durable Functions desde las plataformas macOS y Linux. La razón secundaria es ayudar a los desarrolladores a desplazarse a la última versión del runtime de Azure Functions.

### <a name="hostjson-schema"></a>Host.json schema

El fragmento de código siguiente muestra el nuevo esquema de host.json. Tenga en cuenta el cambio principal es la nueva `"storageProvider"` sección y el `"azureStorage"` sección debajo de ella. Este cambio se realizó para admitir [alternativos de proveedores de almacenamiento](durable-functions-preview.md#alternate-storage-providers).

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": <string>,
      "storageProvider": {
        "azureStorage": {
          "connectionStringName": <string>,
          "controlQueueBatchSize": <int?>,
          "partitionCount": <int?>,
          "controlQueueVisibilityTimeout": <hh:mm:ss?>,
          "workItemQueueVisibilityTimeout": <hh:mm:ss?>,
          "trackingStoreConnectionStringName": <string?>,
          "trackingStoreNamePrefix": <string?>,
          "maxQueuePollingInterval": <hh:mm:ss?>
        }
      },
      "maxConcurrentActivityFunctions": <int?>,
      "maxConcurrentOrchestratorFunctions": <int?>,
      "traceInputAndOutputs": <bool?>,
      "eventGridTopicEndpoint": <string?>,
      "eventGridKeySettingName": <string?>,
      "eventGridPublishRetryCount": <string?>,
      "eventGridPublishRetryInterval": <hh:mm:ss?>,
      "eventGridPublishRetryHttpStatus": <int[]?>,
      "eventgridPublishEventTypes": <string[]?>,
      "customLifeCycleNotificationHelperType"
      "extendedSessionsEnabled": <bool?>,
      "extendedSessionIdleTimeoutInSeconds": <int?>,
      "logReplayEvents": <bool?>
  }
}
```

A medida que Durable Functions 2.0 se estabiliza, se introducirán más cambios a la `durableTask` sección host.json. Para obtener más información sobre estos cambios, consulte [este problema de GitHub](https://github.com/Azure/azure-functions-durable-extension/issues/641).

### <a name="public-interface-changes"></a>Cambios en la interfaz pública

Los distintos objetos de "contexto" compatibles con Durable Functions tenían clases base abstractas destinadas a usarse en las pruebas unitarias. Como parte de Durable Functions 2.0, se han reemplazado con interfaces de estas clases base abstractas. Código de función que usa directamente los tipos concretos no se ven afectados.

La siguiente tabla representa los cambios principales:

| Tipo anterior | Nuevo tipo |
|----------|----------|
| DurableOrchestrationClientBase | IDurableOrchestrationClient |
| DurableOrchestrationContextBase | IDurableOrchestrationContext |
| DurableActivityContextBase | IDurableActivityContext |

En el caso donde una clase base abstracta contiene métodos virtuales, se han reemplazado por los métodos de extensión definidos en estos métodos virtuales `DurableContextExtensions`.

## <a name="entity-functions"></a>Funciones de la entidad

Las funciones de entidad definen las operaciones de lectura y actualización de pequeños fragmentos de estado, conocido como *entidades duraderas*. Al igual que las funciones de orquestador, las funciones de entidad son funciones con un tipo especial de desencadenador, *desencadenador entidad*. A diferencia de las funciones de orquestador, funciones de la entidad no tiene las restricciones de código específico. Funciones de la entidad administran también el estado de forma explícita en lugar de forma implícita que representa el estado a través del flujo de control.

El código siguiente es un ejemplo de una función de entidad simple que define un *contador* entidad. La función define tres operaciones `add`, `subtract`, y `reset`, cada uno de los que actualizar un valor entero, `currentValue`.

```csharp
[FunctionName("Counter")]
public static async Task Counter(
    [EntityTrigger] IDurableEntityContext ctx)
{
    int currentValue = ctx.GetState<int>();
    int operand = ctx.GetInput<int>();

    switch (ctx.OperationName)
    {
        case "add":
            currentValue += operand;
            break;
        case "subtract":
            currentValue -= operand;
            break;
        case "reset":
            await SendResetNotificationAsync();
            currentValue = 0;
            break;
    }

    ctx.SetState(currentValue);
}
```

Entidad *instancias* se accede a través de un identificador único, el *Id. de entidad*. Un identificador de entidad es simplemente un par de cadenas que identifica una instancia de entidad. Consta de:

1. un **nombre de entidad**: un nombre que identifica el tipo de la entidad (por ejemplo, "Contador")
2. un **clave de entidad**: una cadena que identifica la entidad entre todas las demás entidades del mismo nombre (por ejemplo, un GUID)

Por ejemplo, un *contador* función entidad podría utilizarse para mantener la puntuación en un juego en línea. Cada instancia del juego tendrá un identificador de entidad única, como `@Counter@Game1`, `@Counter@Game2`, y así sucesivamente.

### <a name="comparison-with-virtual-actors"></a>Comparación con los actores virtuales

El diseño de entidades duradero se ve influenciado por el [modelo de actor](https://en.wikipedia.org/wiki/Actor_model). Si ya está familiarizado con los actores, los conceptos de entidades duraderas deben ser familiares para usted. En concreto, son similares a las entidades duraderas [actores virtuales](https://research.microsoft.com/en-us/projects/orleans/) de muchas maneras:

* Entidades duraderas son direccionables a través de un *Id. de entidad*.
* Las operaciones de entidad duradero ejecutan en serie, una vez, para evitar condiciones de carrera.
* Entidades duraderas se crean automáticamente cuando se llama o se señala.
* Cuando no está ejecutando las operaciones, durable se descargan las entidades en modo silencioso desde la memoria.

Hay algunas diferencias importantes, sin embargo, que son cabe destacar:

* Entidades duraderas se modelan como funciones puras. Este diseño es diferente de la mayoría marcos orientada a objetos que representan actores con compatibilidad específica del idioma para las clases, propiedades y métodos.
* Dar prioridad entidades duraderas *durabilidad* sobre *latencia*y, por tanto, no puede ser adecuado para las aplicaciones con requisitos estrictos de latencia.
* Los mensajes enviados entre las entidades se entregan en orden y de forma confiable.
* Entidades duraderas pueden utilizarse junto con las orquestaciones durable y pueden servir como bloqueos distribuidos, que se describen más adelante en este artículo.
* Patrones de solicitud/respuesta en las entidades están limitadas a las orquestaciones. Para la comunicación de la entidad a la entidad, se permiten solo mensajes unidireccionales (también conocido como "señalización"), como se muestra en el modelo de actor original. Este comportamiento evita los interbloqueos distribuidos.

### <a name="durable-entity-apis"></a>API de Entity duradero

Compatibilidad con Entity implica varias API. En primer lugar, hay una nueva API para definir funciones de la entidad, como se indicó anteriormente, que especifica qué debe ocurrir cuando se invoca una operación en una entidad. Además, las API existentes para los clientes y las orquestaciones se han actualizado con nueva funcionalidad para la interacción con las entidades.

### <a name="implementing-entity-operations"></a>Implementación de operaciones de entidad

La ejecución de una operación en una entidad puede llamar a estos miembros del objeto de contexto (`IDurableEntityContext` en. NET):

* **OperationName**: Obtiene el nombre de la operación.
* **GetInput\<T >**: Obtiene la entrada para la operación.
* **GetState\<T >**: Obtiene el estado actual de la entidad.
* **SetState**: actualiza el estado de la entidad.
* **SignalEntity**: envía un mensaje unidireccional a una entidad.
* **Self**: Obtiene el identificador de la entidad.
* **Devolver**: devuelve un valor para el cliente o una orquestación que llamó a la operación.
* **IsNewlyConstructed**: devuelve `true` si la entidad no existía antes de la operación.
* **DestructOnExit**: elimina la entidad después de finalizar la operación.

Las operaciones tienen menos restricciones que las orquestaciones:

* Las operaciones pueden llamar a E/S externo, mediante las API sincrónicas o asincrónicas (se recomienda usar solo las asincrónicas).
* Las operaciones pueden ser no deterministas. Por ejemplo, resulta seguro llamar a `DateTime.UtcNow`, `Guid.NewGuid()` o `new Random()`.

### <a name="accessing-entities-from-clients"></a>Acceder a las entidades de clientes

Entidades duraderas se pueden invocar desde las funciones normales a través de la `orchestrationClient` enlace (`IDurableOrchestrationClient` en. NET). Se admiten los siguientes métodos:

* **ReadEntityStateAsync\<T >**: lee el estado de una entidad.
* **SignalEntityAsync**: envía un mensaje unidireccional a una entidad y espera a que poner en cola.

Estos métodos priorizan el rendimiento a través de coherencia: `ReadEntityStateAsync` puede devolver un valor obsoleto, y `SignalEntityAsync` puede devolver antes de que haya finalizado la operación. En cambio, una llamada a las entidades desde las orquestaciones (como se describe a continuación) es altamente coherente.

### <a name="accessing-entities-from-orchestrations"></a>Acceder a las entidades desde las orquestaciones

Las orquestaciones pueden acceder a las entidades mediante el objeto de contexto. Puede elegir entre una comunicación unidireccional (desencadenar y omitir) y la comunicación bidireccional (solicitud y respuesta). Los métodos respectivos son

* **SignalEntity**: envía un mensaje unidireccional a una entidad.
* **CallEntityAsync**: envía un mensaje a una entidad y espera una respuesta que indica que se ha completado la operación.
* **CallEntityAsync\<T >**: envía un mensaje a una entidad y espera una respuesta que contiene un resultado de tipo T.

Cuando se usa la comunicación bidireccional, las excepciones producidas durante la ejecución de la operación también se transmiten a la orquestación de llamada y se vuelve a iniciar. En cambio, cuando se usa fire and forget, no se observan las excepciones.

### <a name="locking-entities-from-orchestrations"></a>Bloqueo de las entidades desde las orquestaciones

Las orquestaciones pueden bloquear las entidades. Esta capacidad proporciona una manera sencilla de evitar las carreras no deseadas mediante *secciones críticas*.

El objeto de contexto proporciona los métodos siguientes:

* **LockAsync**: adquiere bloqueos en una o más entidades.
* **IsLocked**: devuelve true si actualmente se encuentra en una sección crítica, false en caso contrario.

La sección crítica se liberan los extremos y todos los bloqueos, cuando finaliza la orquestación. En. NET, `LockAsync` devuelve un `IDisposable` que finaliza la sección crítica cuando se elimina, que puede utilizarse junto con un `using` cláusula para obtener una representación sintáctica de la sección crítica.

Por ejemplo, considere la posibilidad de una orquestación que necesita para probar si dos jugadores deben estar disponibles y, a continuación, asignarlos a un juego. Esta tarea puede implementarse mediante una sección crítica como sigue:

```csharp
[FunctionName("Orchestrator")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext ctx)
{
    EntityId player1 = /* ... */;
    EntityId player2 = /* ... */;

    using (await ctx.LockAsync(player1, player2))
    {
        bool available1 = await ctx.CallEntityAsync<bool>(player1, "is-available");
        bool available2 = await ctx.CallEntityAsync<bool>(player2, "is-available");

        if (available1 && available2)
        {
            Guid gameId = ctx.NewGuid();

            await ctx.CallEntityAsync(player1, "assign-game", gameId);
            await ctx.CallEntityAsync(player2, "assign-game", gameId);
        }
    }
}
```

En la sección crítica, ambas entidades Reproductor están bloqueadas, lo que significa que no se ejecutan las operaciones que no sean las que se llaman desde dentro de la sección crítica). Este comportamiento impide que las carreras con operaciones en conflicto, como reproductores que se asigna a otro juego o firma desactivado.

Se imponen varias restricciones en las secciones críticas de cómo se pueden usar. Estas restricciones se sirven para evitar los interbloqueos y la reentrada.

* No se puede anidar las secciones críticas.
* Las secciones críticas no pueden crear suborchestrations.
* Las secciones críticas pueden llamar a solo las entidades que haya bloqueado.
* Secciones críticas no pueden llamar a la misma entidad utilizando varias llamadas paralelas.
* Las secciones críticas pueden señalar solo las entidades que no han bloqueado.

## <a name="alternate-storage-providers"></a>Proveedores de almacenamiento alternativo

Durable Task Framework admite varios proveedores de almacenamiento de hoy en día, incluidos [Azure Storage](https://github.com/Azure/durabletask/tree/master/src/DurableTask.AzureStorage), [Azure Service Bus](https://github.com/Azure/durabletask/tree/master/src/DurableTask.ServiceBus), un [emulador en memoria](https://github.com/Azure/durabletask/tree/master/src/DurableTask.Emulator)y un experimental [Redis](https://github.com/Azure/durabletask/tree/redis/src/DurableTask.Redis) proveedor. Sin embargo, hasta ahora, la extensión Durable Task para Azure Functions solo admite el proveedor de almacenamiento de Azure. A partir de Durable Functions 2.0, compatibilidad con proveedores de almacenamiento alternativo se agrega, empezando por el proveedor de Redis.

> [!NOTE]
> Durable Functions 2.0 sólo es compatible con proveedores de .NET Standard 2.0 compatible. En el momento de escribir este artículo, el proveedor de Azure Service Bus no admite .NET Standard 2.0 y, por lo tanto, no está disponible como un proveedor de almacenamiento alternativo.

### <a name="emulator"></a>Emulador

El [DurableTask.Emulator](https://www.nuget.org/packages/Microsoft.Azure.DurableTask.Emulator/) proveedor es una memoria local, el proveedor de almacenamiento no duradero adecuado para escenarios de pruebas locales. Puede configurarse con los mínimos siguientes **host.json** esquema:

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": <string>,
      "storageProvider": {
        "emulator": { }
      }
    }
  }
}
```

### <a name="redis-experimental"></a>Redis (experimental)

El [DurableTask.Redis](https://www.nuget.org/packages/Microsoft.Azure.DurableTask.Redis/) proveedor conserva todos los Estados de orquestación a un clúster de Redis configurado.

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": <string>,
      "storageProvider": {
        "redis": {
          "connectionStringName": <string>,
        }
      }
    }
  }
}
```

El `connectionStringName` debe hacer referencia al nombre de una variable de entorno o configuración de aplicación. Esa variable de entorno o configuración de la aplicación debe contener un valor de cadena de conexión de Redis en forma de *servidor: puerto*. Por ejemplo, `localhost:6379` para conectarse a un clúster de Redis local.

> [!NOTE]
> El proveedor de Redis es experimental actualmente y solo es compatible con aplicaciones de función que se ejecutan en un único nodo.
