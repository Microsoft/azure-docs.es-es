---
title: Uso del valor devuelto de una función de Azure
description: Aprenda a administrar valores devueltos de Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 01/14/2019
ms.author: cshoe
ms.openlocfilehash: 1dd9fabbe235e45290e607f861b67466d33319ce
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212174"
---
# <a name="using-the-azure-function-return-value"></a>Uso del valor devuelto de una función de Azure

En este artículo se explica cómo funcionan los valores devueltos dentro de una función.

En los lenguajes que tienen un valor devuelto, puede enlazar el [enlace de salida](./functions-triggers-bindings.md#binding-direction) de una función al valor devuelto:

* En una biblioteca de clases C#, aplique el atributo de enlace de salida para el valor devuelto del método.
* En Java, aplique la anotación de enlace de salida al método de función.
* En otros lenguajes, establezca la propiedad `name` de *function.json* en `$return`.

Si hay varios enlaces de salida, use el valor devuelto para solo uno de ellos.

En C# y script de C#, los parámetros `out` y los [objetos de recopilador](functions-reference-csharp.md#writing-multiple-output-values) son formas alternativas de enviar datos a un enlace de salida.

# <a name="c"></a>[C#](#tab/csharp)

Este es el código de C# que utiliza el valor devuelto de un enlace de salida, seguido de un ejemplo asincrónico:

```cs
[FunctionName("QueueTrigger")]
[return: Blob("output-container/{id}")]
public static string Run([QueueTrigger("inputqueue")]WorkItem input, ILogger log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.LogInformation($"C# script processed queue message. Item={json}");
    return json;
}
```

```cs
[FunctionName("QueueTrigger")]
[return: Blob("output-container/{id}")]
public static Task<string> Run([QueueTrigger("inputqueue")]WorkItem input, ILogger log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.LogInformation($"C# script processed queue message. Item={json}");
    return Task.FromResult(json);
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Este es el enlace de salida del archivo *function.json*:

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```

Este es el código de script de C#, seguido de un ejemplo asincrónico:

```cs
public static string Run(WorkItem input, ILogger log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.LogInformation($"C# script processed queue message. Item={json}");
    return json;
}
```

```cs
public static Task<string> Run(WorkItem input, ILogger log)
{
    string json = string.Format("{{ \"id\": \"{0}\" }}", input.Id);
    log.LogInformation($"C# script processed queue message. Item={json}");
    return Task.FromResult(json);
}
```

# <a name="f"></a>[F#](#tab/fsharp)

Este es el enlace de salida del archivo *function.json*:

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```

Este es el código de F#:

```fsharp
let Run(input: WorkItem, log: ILogger) =
    let json = String.Format("{{ \"id\": \"{0}\" }}", input.Id)   
    log.LogInformation(sprintf "F# script processed queue message '%s'" json)
    json
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Este es el enlace de salida del archivo *function.json*:

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```

En JavaScript, el valor devuelto va en el segundo parámetro para `context.done`:

```javascript
module.exports = function (context, input) {
    var json = JSON.stringify(input);
    context.log('Node.js script processed queue message', json);
    context.done(null, json);
}
```

# <a name="python"></a>[Python](#tab/python)

Este es el enlace de salida del archivo *function.json*:

```json
{
    "name": "$return",
    "type": "blob",
    "direction": "out",
    "path": "output-container/{id}"
}
```
Este es el código de Python:

```python
def main(input: azure.functions.InputStream) -> str:
    return json.dumps({
        'name': input.name,
        'length': input.length,
        'content': input.read().decode('utf-8')
    })
```

# <a name="java"></a>[Java](#tab/java)

Este es el código de Java que usa el valor devuelto para un enlace de salida:

```java
@FunctionName("QueueTrigger")
@StorageAccount("AzureWebJobsStorage")
@BlobOutput(name = "output", path = "output-container/{id}")
public static String run(
  @QueueTrigger(name = "input", queueName = "inputqueue") WorkItem input,
  final ExecutionContext context
) {
  String json = String.format("{ \"id\": \"%s\" }", input.id);
  context.getLogger().info("Java processed queue message. Item=" + json);
  return json;
}
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Control de errores de enlace de Azure Functions](./functions-bindings-errors.md)
