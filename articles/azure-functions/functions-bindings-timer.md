---
title: Desencadenador de temporizador para Azure Functions
description: Descubra cómo utilizar desencadenadores de temporizador en Azure Functions.
author: craigshoemaker
ms.assetid: d2f013d1-f458-42ae-baf8-1810138118ac
ms.topic: reference
ms.date: 09/08/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 45f704afce28967237b2905ef068678ba05ae085
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88206640"
---
# <a name="timer-trigger-for-azure-functions"></a>Desencadenador de temporizador para Azure Functions 

En este artículo se explica cómo usar desencadenadores de temporizador en Azure Functions. Con un desencadenador de temporizador puede ejecutar una función de forma programada. 

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

Para información sobre cómo ejecutar manualmente una función desencadenada por un temporizador, consulte [Ejecución manual de una función no desencadenada por HTTP](./functions-manually-run-non-http.md).

## <a name="packages---functions-1x"></a>Paquetes: Functions 1.x

El desencadenador del temporizador se proporciona en el paquete NuGet [Microsoft.Azure.WebJobs.Extensions.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions), versión 2.x. El código fuente del paquete está en el repositorio [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions/Extensions/Timers/) de GitHub.

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="packages---functions-2x-and-higher"></a>Paquetes: Functions 2.x y versiones posteriores

El desencadenador del temporizador se proporciona en el paquete NuGet [Microsoft.Azure.WebJobs.Extensions.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions), versión 3.x. El código fuente del paquete está en el repositorio [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/) de GitHub.

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que se ejecuta cada vez que los minutos tienen un valor divisible entre cinco (por ejemplo, si la función se inicia a las 18:57:00, la próxima ejecución será a las 19:00:00): El objeto [`TimerInfo`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs) se pasa a la función.

```cs
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador de temporizador en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace. La función escribe un registro que indica si esta invocación de función se debe a una repetición de la programación no ejecutada. El objeto [`TimerInfo`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs) se pasa a la función.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Este es el código de script de C#:

```csharp
public static void Run(TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}" );  
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador de temporizador en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace. La función escribe un registro que indica si esta invocación de función se debe a una repetición de la programación no ejecutada. Un [objeto de temporizador](#usage) se pasa a la función.

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Este es el código de JavaScript:

```JavaScript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if (myTimer.IsPastDue)
    {
        context.log('Node is running late!');
    }
    context.log('Node timer trigger function ran!', timeStamp);   

    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se usa un enlace de desencadenador de temporizador cuya configuración se describe en el archivo *function.json*. La [función de Python](functions-reference-python.md) real que usa el enlace se describe en el archivo *__init__.py*. El objeto pasado a la función es de tipo [azure.functions.TimerRequest](/python/api/azure-functions/azure.functions.timerrequest). La lógica de la función escribe en los registros para indicar si la invocación actual se debe a una repetición de la programación no ejecutada. 

Estos son los datos de enlace del archivo *function.json*:

```json
{
    "name": "mytimer",
    "type": "timerTrigger",
    "direction": "in",
    "schedule": "0 */5 * * * *"
}
```

Este es el código de Python:

```python
import datetime
import logging

import azure.functions as func


def main(mytimer: func.TimerRequest) -> None:
    utc_timestamp = datetime.datetime.utcnow().replace(
        tzinfo=datetime.timezone.utc).isoformat()

    if mytimer.past_due:
        logging.info('The timer is past due!')

    logging.info('Python timer trigger function ran at %s', utc_timestamp)
```

# <a name="java"></a>[Java](#tab/java)

El la función de ejemplo siguiente se desencadena y se ejecuta cada cinco minutos. La anotación `@TimerTrigger` en la función define la programación con el mismo formato de cadena que las [expresiones CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression).

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [TimerTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs).

El constructor del atributo toma una expresión CRON o `TimeSpan`. Puede usar `TimeSpan` solamente si la aplicación de funciones se ejecuta en un plan de App Service. No se admite `TimeSpan` para las funciones premium de consumo o elásticas.

En el ejemplo siguiente se muestra una expresión CRON:

```csharp
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

La anotación `@TimerTrigger` en la función define la programación con el mismo formato de cadena que las [expresiones CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression).

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `TimerTrigger`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Debe establecerse en "timerTrigger". Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal.|
|**direction** | N/D | Debe establecerse en "in". Esta propiedad se establece automáticamente cuando se crea el desencadenador en Azure Portal. |
|**name** | N/D | Nombre de la variable que representa el objeto de temporizador en el código de la función. | 
|**schedule**|**ScheduleExpression**|Una [expresión CRON](#ncrontab-expressions) o un valor [TimeSpan](#timespan). `TimeSpan` solamente se puede usar para una aplicación de función que se ejecuta en un plan de App Service. Puede colocar la expresión de programación en una configuración de aplicación y establecer esta propiedad en el nombre de dicha configuración encapsulado en signos **%** , como en este ejemplo: "%ScheduleAppSetting%". |
|**runOnStartup**|**RunOnStartup**|Si `true`, la función se invoca cuando se inicia el entorno de ejecución. Por ejemplo, el entorno de ejecución se inicia cuando la aplicación de función se reactiva después de estar inactiva por inactividad, cuando la aplicación de función se reinicia debido a cambios de función y cuando la aplicación de función se escala horizontalmente. Por lo tanto, **runOnStartup** muy pocas veces, si acaso, se debe establecer en `true`, especialmente en producción. |
|**useMonitor**|**useMonitor**|Establezca esta propiedad en `true` o `false` para indicar si la programación se debe supervisar. La supervisión de la programación conserva las apariciones de programación para ayudar a garantizar que la programación se mantiene correctamente aunque las instancias de aplicación de función se reinicien. Si no se establece explícitamente, el valor predeterminado es `true` para las programaciones que tienen un intervalo de periodicidad superior o igual a 1 minuto. Para las programaciones que se desencadenan más de una vez por minuto, el valor predeterminado es `false`.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!CAUTION]
> No se recomienda establecer **runOnStartup** en `true` en producción. Con esta configuración, el código se ejecuta en momentos altamente impredecibles. En ciertas configuraciones de producción, estas ejecuciones adicionales pueden dar lugar a costos considerablemente mayores para las aplicaciones hospedadas en los planes de consumo. Por ejemplo, con **runOnStartup** habilitado, se invoca al desencadenador cada vez que se escala la aplicación de función. Asegúrese de comprender perfectamente el comportamiento de producción de las funciones antes de habilitar **runOnStartup** en producción.   

## <a name="usage"></a>Uso

Cuando se invoca una función de desencadenador de temporizador, se pasa a esta un objeto de temporizador. La siguiente función JSON es un ejemplo que representa el objeto de temporizador.

```json
{
    "Schedule":{
    },
    "ScheduleStatus": {
        "Last":"2016-10-04T10:15:00+00:00",
        "LastUpdated":"2016-10-04T10:16:00+00:00",
        "Next":"2016-10-04T10:20:00+00:00"
    },
    "IsPastDue":false
}
```

La propiedad `IsPastDue` es `true` cuando la invocación de función actual es posterior a la programada. Por ejemplo, un reinicio de aplicación de función podría provocar la ausencia de una invocación.

## <a name="ncrontab-expressions"></a>Expresiones NCRONTAB 

Azure Functions usa la biblioteca [NCronTab](https://github.com/atifaziz/NCrontab) para interpretar las expresiones NCRONTAB. Una expresión NCRONTAB es similar a una expresión CRON, excepto en que incluye un sexto campo adicional al comienzo para usarlo con una precisión de segundos:

`{second} {minute} {hour} {day} {month} {day-of-week}`

Cada campo puede tener uno de los siguientes tipos de valores:

|Tipo  |Ejemplo  |Cuándo se desencadena  |
|---------|---------|---------|
|Un valor específico |<nobr>"0 5 * * * *"</nobr>|A las hh:05:00, donde hh es cada hora (una vez por hora)|
|Todos los valores (`*`)|<nobr>"0 * 5 * * *"</nobr>|A las 5:mm: 00 cada día, donde mm es cada minuto de la hora (60 veces al día)|
|Un intervalo (operador `-`)|<nobr>"5-7 * * * * *"</nobr>|A las hh:mm:05, hh:mm:06 y hh:mm:07, donde hh:mm es cada minuto de cada hora (tres veces por minuto)|
|Un conjunto de valores (operador `,`)|<nobr>"5,8,10 * * * * *"</nobr>|A las hh:mm:05, hh:mm:08 y hh:mm:10, donde hh:mm es cada minuto de cada hora (tres veces por minuto)|
|Un valor de intervalo (operador `/`)|<nobr>"0 */5 * * * *"</nobr>|A las hh:00:00,las hh:05:00, las hh:10:00 y así sucesivamente hasta las hh:55:00, donde hh es cada hora (doce veces por hora)|

[!INCLUDE [functions-cron-expressions-months-days](../../includes/functions-cron-expressions-months-days.md)]

### <a name="ncrontab-examples"></a>Ejemplos de NCRONTAB

Estos son algunos ejemplos de expresiones NCRONTAB que puede usar para el desencadenador de temporizador en Azure Functions.

|Ejemplo|Cuándo se desencadena  |
|---------|---------|
|`"0 */5 * * * *"`|Una vez cada cinco minutos|
|`"0 0 * * * *"`|Una vez al principio de cada hora|
|`"0 0 */2 * * *"`|Una vez cada dos horas|
|`"0 0 9-17 * * *"`|Una vez cada hora de 9 a. m. a 5 p. m.|
|`"0 30 9 * * *"`|A las 9:30 a. m. todos los días|
|`"0 30 9 * * 1-5"`|A las 9:30 a. m. cada día de la semana|
|`"0 30 9 * Jan Mon"`|A las 9:30 a. m. todos los lunes en enero|


### <a name="ncrontab-time-zones"></a>Zonas horarias de NCRONTAB

Los números de una expresión CRON hacen referencia a una hora y fecha, no un intervalo de tiempo. Por ejemplo, un 5 en el campo `hour` hace referencia a las 5:00 a. m., no a cada 5 horas.

[!INCLUDE [functions-timezone](../../includes/functions-timezone.md)]

## <a name="timespan"></a>TimeSpan

 `TimeSpan` solamente se puede usar para una aplicación de función que se ejecuta en un plan de App Service.

A diferencia de una expresión CRON, un valor `TimeSpan` especifica el intervalo de tiempo entre cada invocación de función. Cuando una función se completa después de ejecutarse más tiempo del intervalo especificado, el temporizador vuelve a invocar inmediatamente a la función.

Expresado como una cadena, el formato `TimeSpan` es `hh:mm:ss` cuando `hh` es menor que 24. Si los dos primeros dígitos son 24 o un valor superior, el formato es `dd:hh:mm`. Estos son algunos ejemplos:

|Ejemplo |Cuándo se desencadena  |
|---------|---------|
|"01:00:00" | Cada hora        |
|"00:01:00"|Cada minuto         |
|"24:00:00" | Cada 24 días        |
|"1.00:00:00" | Todos los días        |

## <a name="scale-out"></a>Escalado horizontal

Si una aplicación de función se escala horizontalmente a varias instancias, una sola instancia de una función desencadenada por el temporizador se ejecuta en todas las instancias.

## <a name="function-apps-sharing-storage"></a>Aplicaciones de función que comparten almacenamiento

Si comparte cuentas de almacenamiento en aplicaciones de funciones que no se implementan en app service, es posible que tenga que asignar explícitamente un identificador de host a cada aplicación.

| Versión de Functions | Configuración                                              |
| ----------------- | ---------------------------------------------------- |
| 2.x (y versiones posteriores)  | Variable de entorno `AzureFunctionsWebHost__hostid` |
| 1.x               | `id` en *host.json*                                  |

Puede omitir el valor de identificación o establecer manualmente la configuración de identificación de cada aplicación de funciones en otro valor.

El desencadenador de temporizador utiliza un bloqueo de almacenamiento para asegurarse de que solo hay una instancia de temporizador cuando una aplicación de función se escala a varias instancias. Si dos aplicaciones de función comparten la misma configuración de identificación y cada una usa un desencadenador de temporizador, solo se ejecutará un temporizador.

## <a name="retry-behavior"></a>Comportamiento de reintento

A diferencia del desencadenador de cola, el desencadenador de temporizador no vuelve a realizar el intento después de que se produce un error en una función. Cuando se produce un error en una función, no se la llamará nuevamente hasta la próxima vez en la programación.

## <a name="troubleshooting"></a>Solución de problemas

Para obtener información sobre qué hacer cuando el desencadenador del temporizador no funcione según lo esperado, consulte [Investigating and reporting issues with timer triggered functions not firing](https://github.com/Azure/azure-functions-host/wiki/Investigating-and-reporting-issues-with-timer-triggered-functions-not-firing) (Investigación y notificación de problemas en que las funciones desencadenadas por el temporizador no se activan).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ir a una guía de inicio rápido que use un desencadenador de temporizador](functions-create-scheduled-function.md)

> [!div class="nextstepaction"]
> [Más información sobre desencadenadores y enlaces de Azure Functions](functions-triggers-bindings.md)
