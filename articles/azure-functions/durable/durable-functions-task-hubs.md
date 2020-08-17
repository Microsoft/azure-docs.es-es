---
title: 'Centrales de tareas en Durable Functions: Azure'
description: Aprenda qué son las centrales de tareas en la extensión Durable Functions para Azure Functions. Aprenda a configurar centrales de tareas.
author: cgillum
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: azfuncdf
ms.openlocfilehash: 26234039c77601bc1d29beeebd3fcb8461d6d6c9
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432703"
---
# <a name="task-hubs-in-durable-functions-azure-functions"></a>Centrales de tareas en Durable Functions (Azure Functions)

Una *central de tareas* en [Durable Functions](durable-functions-overview.md) es un contenedor lógico para los recursos de Azure Storage que se usan para las orquestaciones. Las funciones de orquestador y actividad solo pueden interactuar entre sí si pertenecen a la misma central de tareas.

Si varias aplicaciones de función comparten una cuenta de almacenamiento, se *debe* configurar cada una de ellas con un nombre de central de tareas independiente. Una cuenta de almacenamiento puede contener varias centrales de tareas. El siguiente diagrama muestra una central de tareas por cada aplicación de función en las cuentas de almacenamiento compartidas y dedicadas.

![Diagrama de almacenamiento que muestra las cuentas de almacenamiento compartidas y dedicadas.](./media/durable-functions-task-hubs/task-hubs-storage.png)

## <a name="azure-storage-resources"></a>Recursos de Azure Storage

Una central de tareas consta de los siguientes recursos de almacenamiento:

* Una o más colas de control.
* Una cola de elementos de trabajo.
* Una tabla de historial.
* Una tabla de instancias.
* Un contenedor de almacenamiento que contiene uno o varios blobs de concesión.
* Un contenedor de almacenamiento que contiene cargas de mensajes grandes, si procede.

Todos estos recursos se crean automáticamente en la cuenta de Azure Storage predeterminada cuando se ejecutan funciones de orquestador, de entidad o de actividad o cuando se programa su ejecución. El artículo [Rendimiento y escala](durable-functions-perf-and-scale.md) explica cómo se utilizan estos recursos.

## <a name="task-hub-names"></a>Nombres de las centrales de tareas

Las centrales de tareas se identifican mediante un nombre que se ajusta a estas reglas:

* Solo contiene caracteres alfanuméricos.
* Comienza con una letra.
* Tiene una longitud mínima de 3 caracteres, con una longitud máxima de 45 caracteres.

El nombre de la central de tareas se declara en el archivo *host.json*, como se muestra en el ejemplo siguiente:

### <a name="hostjson-functions-20"></a>host.json (Functions 2.0)

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub"
    }
  }
}
```

### <a name="hostjson-functions-1x"></a>host.json (Functions 1.x)

```json
{
  "durableTask": {
    "hubName": "MyTaskHub"
  }
}
```

Las centrales de tareas también se pueden configurar mediante la configuración de la aplicación, tal como se muestra en el siguiente archivo de ejemplo `host.json`:

### <a name="hostjson-functions-10"></a>host.json (Functions 1.0)

```json
{
  "durableTask": {
    "hubName": "%MyTaskHub%"
  }
}
```

### <a name="hostjson-functions-20"></a>host.json (Functions 2.0)

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": "%MyTaskHub%"
    }
  }
}
```

El nombre de la central de tareas se establecerá en el valor de la configuración `MyTaskHub` de la aplicación. El siguiente valor de `local.settings.json` muestra cómo definir la configuración `MyTaskHub` como `samplehubname`:

```json
{
  "IsEncrypted": false,
  "Values": {
    "MyTaskHub" : "samplehubname"
  }
}
```

En el siguiente código se muestra cómo escribir una función que usa el [enlace del cliente de orquestación](durable-functions-bindings.md#orchestration-client) para trabajar con una central de tareas definida como configuración de la aplicación:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("HttpStart")]
public static async Task<HttpResponseMessage> Run(
    [HttpTrigger(AuthorizationLevel.Function, methods: "post", Route = "orchestrators/{functionName}")] HttpRequestMessage req,
    [DurableClient(TaskHub = "%MyTaskHub%")] IDurableOrchestrationClient starter,
    string functionName,
    ILogger log)
{
    // Function input comes from the request content.
    object eventData = await req.Content.ReadAsAsync<object>();
    string instanceId = await starter.StartNewAsync(functionName, eventData);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    return starter.CreateCheckStatusResponse(req, instanceId);
}
```

> [!NOTE]
> El ejemplo de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La propiedad de la central de tareas del archivo `function.json` se establece mediante la configuración de aplicación:

```json
{
    "name": "input",
    "taskHub": "%MyTaskHub%",
    "type": "orchestrationClient",
    "direction": "in"
}
```

# <a name="python"></a>[Python](#tab/python)

La propiedad de la central de tareas del archivo `function.json` se establece mediante la configuración de aplicación:

```json
{
    "name": "input",
    "taskHub": "%MyTaskHub%",
    "type": "orchestrationClient",
    "direction": "in"
}
```

---

Los nombres de la central de tareas deben empezar por una letra y estar formados únicamente por letras y números. Si no se especifica, se usará un nombre de central de tareas predeterminado, tal y como se muestra en la tabla siguiente:

| Versión de la extensión de Durable | Nombre de central de tareas predeterminado |
| - | - |
| 2.x | Cuando se implementa en Azure, el nombre de la central de tareas se deriva del nombre de la _aplicación de funciones_. Cuando se ejecuta fuera de Azure, el nombre predeterminado de la central de tareas es `TestHubName`. |
| 1.x | El nombre predeterminado de la central de tareas para todos los entornos es `DurableFunctionsHub`. |

Para obtener más información sobre las diferencias entre versiones de extensión, consulte el artículo [Versiones de Durable Functions](durable-functions-versions.md).

> [!NOTE]
> El nombre es lo que diferencia una central de tareas de otra cuando hay varias de ellas en una cuenta de almacenamiento compartido. Si tiene varias aplicaciones de función que comparten una cuenta de almacenamiento, deberá configurar explícitamente nombres diferentes para cada central de tareas en el archivo *host.json*. En caso contrario, las diversas aplicaciones de funciones competirán entre sí por los mensajes, lo cual podría provocar un comportamiento indefinido, como, por ejemplo, orquestaciones que se "atascan" inesperadamente en el estado `Pending` o `Running`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo administrar el control de versiones de orquestación](durable-functions-versioning.md)
