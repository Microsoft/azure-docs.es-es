---
title: 'Encadenamiento de funciones en Durable Functions: Azure'
description: Obtenga información acerca de cómo ejecutar un ejemplo de Durable Functions que ejecute una secuencia de funciones.
author: cgillum
ms.topic: conceptual
ms.date: 11/29/2019
ms.author: azfuncdf
ms.openlocfilehash: b117fca23b26919f3c404dd32ba64c0c89d66ae7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87033571"
---
# <a name="function-chaining-in-durable-functions---hello-sequence-sample"></a>Encadenamiento de funciones en Durable Functions: Hello Sequence de ejemplo

El encadenamiento de funciones hace referencia al patrón de ejecución de una secuencia de funciones en un orden concreto. A menudo la salida de una función tiene que aplicarse a la entrada de otra función. Este artículo describe la secuencia de encadenamiento que se crea al completar el inicio rápido de Durable Functions ([ C# ](durable-functions-create-first-csharp.md) o [JavaScript](quickstart-js-vscode.md)). Para más información sobre Durable Functions, consulte [Durable Functions overview](durable-functions-overview.md) (Información general de Durable Functions).

[!INCLUDE [durable-functions-prerequisites](../../../includes/durable-functions-prerequisites.md)]

## <a name="the-functions"></a>Funciones

En este artículo se explican las funciones siguientes en la aplicación de ejemplo:

* `E1_HelloSequence`: una [función de orquestador](durable-functions-bindings.md#orchestration-trigger) que llama a `E1_SayHello` varias veces en una secuencia. Almacena las salidas de las llamadas de `E1_SayHello` y registra los resultados.
* `E1_SayHello`: una [función de actividad](durable-functions-bindings.md#activity-trigger) que antepone una cadena con "Hello".
* `HttpStart`: una función desencadenada por HTTP que inicia una instancia del orquestador.

### <a name="e1_hellosequence-orchestrator-function"></a>Función de orquestador E1_HelloSequence

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs?range=13-25)]

Todas las funciones de orquestación de C# deben tener un parámetro de tipo `DurableOrchestrationContext`, que existe en el ensamblado `Microsoft.Azure.WebJobs.Extensions.DurableTask`. Este objeto de contexto le permite llamar a otras funciones de *actividad* y pasar parámetros de entrada con su método `CallActivityAsync`.

El código llama a `E1_SayHello` tres veces en secuencia con distintos valores de parámetro. El valor devuelto de cada llamada se agrega a la lista `outputs`, que se devuelve al final de la función.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

> [!NOTE]
> Durable Functions de JavaScript solo está disponible para el runtime de Functions 2.0.

#### <a name="functionjson"></a>function.json

Si utiliza Visual Studio Code o Azure Portal para el desarrollo, aquí tiene el contenido del archivo *function.json* para la función de orquestador. La mayoría de los archivos *function.json* de orquestador presentan un aspecto prácticamente idéntico a este.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E1_HelloSequence/function.json)]

Lo importante es el tipo de enlace `orchestrationTrigger`. Todas las funciones de orquestador deben usar este tipo de desencadenador.

> [!WARNING]
> Para cumplir la regla de "ninguna E/S" de las funciones de orquestador, no use ningún enlace de entrada o salida al utilizar el enlace de desencadenador `orchestrationTrigger`.  Si se necesitan otros enlaces de entrada o salida, deberían utilizarse en el contexto de funciones `activityTrigger` en su lugar, a las que llama el orquestador. Para más información, consulte el artículo sobre las [restricciones de código de las funciones de orquestador](durable-functions-code-constraints.md).

#### <a name="indexjs"></a>index.js

Esta es la función:

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E1_HelloSequence/index.js)]

Todas las funciones de orquestación de JavaScript tienen que incluir el módulo [`durable-functions`](https://www.npmjs.com/package/durable-functions). Se trata de una biblioteca que le permite escribir Durable Functions en JavaScript. Hay tres diferencias importantes entre una función de orquestación y otras funciones de JavaScript:

1. La función es una [función de generador.](/scripting/javascript/advanced/iterators-and-generators-javascript)
2. La función se ajusta en una llamada al método `orchestrator` del módulo `durable-functions` (aquí `df`).
3. La función debe ser sincrónica. Dado que el método "orchestrator" se encarga de llamar a "context.done", la función debería simplemente devolver "return".

El objeto `context` contiene un objeto de contexto de orquestación `df` duradero que le permite llamar a otras funciones de *actividad* y pasar parámetros de entrada mediante su método `callActivity`. El código llama a `E1_SayHello` tres veces en secuencia con valores de parámetro diferentes, utilizando `yield` para indicar que la ejecución debe esperar en las llamadas de función de actividad asincrónica que se van a devolver. El valor devuelto de cada llamada se agrega a la matriz `outputs`, que se devuelve al final de la función.

---

### <a name="e1_sayhello-activity-function"></a>Función de actividad E1_SayHello

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs?range=27-32)]

Las actividades usan el atributo `ActivityTrigger`. Use el `IDurableActivityContext` proporcionado para realizar acciones relacionadas con la actividad, como el acceso al valor de entrada mediante `GetInput<T>`.

La implementación de `E1_SayHello` es una operación de formato de cadena relativamente sencilla.

En lugar de enlazar a un `IDurableActivityContext`, puede enlazar directamente con el tipo que se pasa a la función de actividad. Por ejemplo:

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs?range=34-38)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

#### <a name="e1_sayhellofunctionjson"></a>E1_SayHello/function.json

El archivo *function.json* para la función de actividad `E1_SayHello` es similar al de `E1_HelloSequence` salvo en que usa un tipo de enlace `activityTrigger` en lugar de un tipo de enlace `orchestrationTrigger`.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E1_SayHello/function.json)]

> [!NOTE]
> Cualquier función llamada por una función de orquestación debe utilizar el enlace `activityTrigger`.

La implementación de `E1_SayHello` es una operación de formato de cadena relativamente sencilla.

#### <a name="e1_sayhelloindexjs"></a>E1_SayHello/index.js

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E1_SayHello/index.js)]

A diferencia de una función de orquestación de JavaScript, una función de actividad no necesita una configuración especial. La entrada que le pasa la función del orquestador se encuentra en el objeto `context.bindings` bajo el nombre del enlace `activityTrigger`, en este caso `context.bindings.name`. El nombre de enlace puede establecerse como un parámetro de la función exportada y se puede acceder a él directamente, que es lo que hace el código de ejemplo.

---

### <a name="httpstart-client-function"></a>Función de cliente HttpStart

Puede iniciar una instancia de la función de orquestador mediante una función de cliente. Usará la función desencadenada por HTTP `HttpStart`para iniciar instancias de `E1_HelloSequence`.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpStart.cs?range=13-30)]

Para interactuar con los orquestadores, la función tiene que incluir un enlace de entrada `DurableClient`. El cliente se utiliza para iniciar una orquestación. También puede ayudarle a devolver una respuesta HTTP que contiene las direcciones URL para comprobar el estado de la nueva orquestación.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

#### <a name="httpstartfunctionjson"></a>HttpStart/function.json

[!code-json[Main](~/samples-durable-functions/samples/javascript/HttpStart/function.json?highlight=16-20)]

Para interactuar con los orquestadores, la función tiene que incluir un enlace de entrada `durableClient`.

#### <a name="httpstartindexjs"></a>HttpStart/index.js

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/HttpStart/index.js)]

Para obtener un objeto `DurableOrchestrationClient`, use `df.getClient`. El cliente se utiliza para iniciar una orquestación. También puede ayudarle a devolver una respuesta HTTP que contiene las direcciones URL para comprobar el estado de la nueva orquestación.

---

## <a name="run-the-sample"></a>Ejecución del ejemplo

Para ejecutar la orquestación de `E1_HelloSequence`, envíe la siguiente solicitud POST HTTP a la función `HttpStart`.

```
POST http://{host}/orchestrators/E1_HelloSequence
```

> [!NOTE]
> El fragmento de código HTTP anterior da por supuesto que hay una entrada en el archivo `host.json` que elimina el prefijo `api/` predeterminado de todas las direcciones URL de las funciones de activación de HTTP. Puede encontrar el marcador para esta configuración en el archivo `host.json` en los ejemplos.

Por ejemplo, si ejecuta el ejemplo en una aplicación de función llamada "myfunctionapp", reemplace "{host}" por "myfunctionapp.azurewebsites.net".

El resultado es una respuesta HTTP 202, similar a la siguiente (acortada para simplificar):

```
HTTP/1.1 202 Accepted
Content-Length: 719
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/96924899c16d43b08a536de376ac786b?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

(...trimmed...)
```

En este punto, la orquestación se pone en cola y comienza a ejecutarse inmediatamente. La dirección URL en el encabezado `Location` puede usarse para comprobar el estado de la ejecución.

```
GET http://{host}/runtime/webhooks/durabletask/instances/96924899c16d43b08a536de376ac786b?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

El resultado es el estado de la orquestación. Se ejecuta y se completa con rapidez, por lo que se muestra con el estado *Completed* (Completado) con una respuesta similar a la siguiente (acortada para simplificar):

```
HTTP/1.1 200 OK
Content-Length: 179
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":null,"output":["Hello Tokyo!","Hello Seattle!","Hello London!"],"createdTime":"2017-06-29T05:24:57Z","lastUpdatedTime":"2017-06-29T05:24:59Z"}
```

Como puede ver, el valor de `runtimeStatus` de la instancia es *Completed* (Completado) y `output` contiene el resultado serializado con JSON de la ejecución de la función de orquestador.

> [!NOTE]
> Puede implementar una lógica de inicio similar para otros tipos de desencadenadores, como `queueTrigger`, `eventHubTrigger` o `timerTrigger`.

Observe los registros de ejecución de la función. La función `E1_HelloSequence` se ha iniciado y completado varias veces debido al comportamiento de reproducción descrito en el tema sobre la [confiabilidad de la orquestación](durable-functions-orchestrations.md#reliability). Por otro lado, solo ha habido tres de ejecuciones de `E1_SayHello`, puesto que esas ejecuciones de función no se reproducen.

## <a name="next-steps"></a>Pasos siguientes

En este ejemplo se ha demostrado una orquestación de encadenamiento de función simple. En el ejemplo siguiente se muestra cómo implementar el de distribución ramificada de salida y entrada.

> [!div class="nextstepaction"]
> [Ejecución del ejemplo de distribución ramificada de salida y de entrada](durable-functions-cloud-backup.md)
