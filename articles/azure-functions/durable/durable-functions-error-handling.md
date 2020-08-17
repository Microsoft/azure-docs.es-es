---
title: 'Control de errores con Durable Functions: Azure'
description: Aprenda a controlar errores en la extensión Durable Functions para Azure Functions.
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: azfuncdf
ms.openlocfilehash: 6650322834d491d78470e2d8dbd24e2c6750ae39
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081702"
---
# <a name="handling-errors-in-durable-functions-azure-functions"></a>Control de errores con Durable Functions (Azure Functions)

Las orquestaciones de Durable Functions se implementan en código y pueden utilizar las características de control de errores integradas en el lenguaje de programación. Realmente no hay ningún concepto nuevo que deba conocer para agregar el control de errores y la compensación en las orquestaciones. No obstante, hay algunos comportamientos que deben tenerse en cuenta.

## <a name="errors-in-activity-functions"></a>Errores en funciones de actividad

Cualquier excepción que se produce en una función de actividad se devuelve a la función de orquestador en el búfer y se inicia como `FunctionFailedException`. Puede escribir código de control y compensación de errores que se adapte a sus necesidades en la función de orquestador.

Por ejemplo, considere la siguiente función de orquestador que transfiere fondos de una cuenta a otra:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TransferFunds")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var transferDetails = context.GetInput<TransferOperation>();

    await context.CallActivityAsync("DebitAccount",
        new
        {
            Account = transferDetails.SourceAccount,
            Amount = transferDetails.Amount
        });

    try
    {
        await context.CallActivityAsync("CreditAccount",
            new
            {
                Account = transferDetails.DestinationAccount,
                Amount = transferDetails.Amount
            });
    }
    catch (Exception)
    {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        await context.CallActivityAsync("CreditAccount",
            new
            {
                Account = transferDetails.SourceAccount,
                Amount = transferDetails.Amount
            });
    }
}
```

> [!NOTE]
> Los ejemplos de C# anteriores corresponden a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const transferDetails = context.df.getInput();

    yield context.df.callActivity("DebitAccount",
        {
            account = transferDetails.sourceAccount,
            amount = transferDetails.amount,
        }
    );

    try {
        yield context.df.callActivity("CreditAccount",
            {
                account = transferDetails.destinationAccount,
                amount = transferDetails.amount,
            }
        );
    }
    catch (error) {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        yield context.df.callActivity("CreditAccount",
            {
                account = transferDetails.sourceAccount,
                amount = transferDetails.amount,
            }
        );
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    transfer_details = context.get_input()

    yield context.call_activity('DebitAccount', {
         'account': transfer_details['sourceAccount'],
         'amount' : transfer_details['amount']
    })

    try:
        yield context.call_activity('CreditAccount', {
                'account': transfer_details['destinationAccount'],
                'amount': transfer_details['amount'],
            })
    except:
        yield context.call_activity('CreditAccount', {
            'account': transfer_details['sourceAccount'],
            'amount': transfer_details['amount']
        })

main = df.Orchestrator.create(orchestrator_function)
```

---

Si se produce un error en la primera llamada a la función **CreditAccount**, la función de orquestador lo compensa al devolver los fondos a la cuenta de origen.

## <a name="automatic-retry-on-failure"></a>Reintento automático en caso de error

Al llamar a funciones de actividad o funciones de suborquestación, puede especificar una directiva de reintentos automáticos. En el ejemplo siguiente se intenta llamar a una función hasta 3 veces y se espera 5 segundos entre cada reintento:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TimerOrchestratorWithRetry")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var retryOptions = new RetryOptions(
        firstRetryInterval: TimeSpan.FromSeconds(5),
        maxNumberOfAttempts: 3);

    await context.CallActivityWithRetryAsync("FlakyFunction", retryOptions, null);

    // ...
}
```

> [!NOTE]
> Los ejemplos de C# anteriores corresponden a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const firstRetryIntervalInMilliseconds = 5000;
    const maxNumberOfAttempts = 3;

    const retryOptions = 
        new df.RetryOptions(firstRetryIntervalInMilliseconds, maxNumberOfAttempts);

    yield context.df.callActivityWithRetry("FlakyFunction", retryOptions);

    // ...
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    first_retry_interval_in_milliseconds = 5000
    max_number_of_attempts = 3

    retry_options = df.RetryOptions(first_retry_interval_in_milliseconds, max_number_of_attempts)

    yield context.call_activity_with_retry('FlakyFunction', retry_options)

main = df.Orchestrator.create(orchestrator_function)
```

---

La llamada a la función de actividad en el ejemplo anterior toma un parámetro para configurar una directiva de reintento automático. Existen varias opciones para personalizar la directiva de reintentos automáticos:

* **Número máximo de intentos**: Número máximo de reintentos.
* **Intervalo para el primer reintento**: cantidad de tiempo de espera antes del primer reintento.
* **Backoff coefficient** (Coeficiente de retroceso): coeficiente que se usa para determinar la tasa de incremento del retroceso. De manera predeterminada, su valor es 1.
* **Max retry interval** (Intervalo de reintento máximo): cantidad máxima de tiempo de espera entre reintentos.
* **Retry timeout** (Tiempo de espera de reintento): cantidad máxima de tiempo durante el que realizar reintentos. El comportamiento predeterminado es realizar reintentos de manera indefinida.
* **Manipulador**: se puede especificar una devolución de llamada definida por el usuario para determinar si se debe reintentar función.

## <a name="function-timeouts"></a>Tiempos de espera de función

Es posible que desee abandonar una llamada de función dentro de una función de orquestador si tarda demasiado tiempo en completarse. Actualmente, la manera adecuada de hacerlo es mediante la creación de un [temporizador durable](durable-functions-timers.md) mediante `context.CreateTimer` (.NET), `context.df.createTimer` (JavaScript) o `context.create_timer` (Python), junto con `Task.WhenAny` (.NET), `context.df.Task.any` (JavaScript) o `context.task_any` (Python), como en el ejemplo siguiente:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TimerOrchestrator")]
public static async Task<bool> Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("FlakyFunction");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

> [!NOTE]
> Los ejemplos de C# anteriores corresponden a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const deadline = moment.utc(context.df.currentUtcDateTime).add(30, "s");

    const activityTask = context.df.callActivity("FlakyFunction");
    const timeoutTask = context.df.createTimer(deadline.toDate());

    const winner = yield context.df.Task.any([activityTask, timeoutTask]);
    if (winner === activityTask) {
        // success case
        timeoutTask.cancel();
        return true;
    } else {
        // timeout case
        return false;
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    deadline = context.current_utc_datetime + timedelta(seconds = 30)
    
    activity_task = context.call_activity('FlakyFunction')
    timeout_task = context.create_timer(deadline)

    winner = yield context.task_any(activity_task, timeout_task)
    if winner == activity_task:
        timeout_task.cancel()
        return True
    else:
        return False

main = df.Orchestrator.create(orchestrator_function)
```

---

> [!NOTE]
> Este mecanismo no finaliza realmente la ejecución de la función de actividad en curso. En su lugar, simplemente permite que la función de orquestador pase por alto el resultado y continúe. Consulte la documentación sobre [temporizadores](durable-functions-timers.md#usage-for-timeout) para más información.

## <a name="unhandled-exceptions"></a>Excepciones no controladas

Si se produce un error en una función de orquestador con una excepción no controlada, se registran los detalles de la excepción y la instancia se completa con el estado `Failed`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre las orquestaciones infinitas](durable-functions-eternal-orchestrations.md)

> [!div class="nextstepaction"]
> [Cómo diagnosticar problemas](durable-functions-diagnostics.md)
