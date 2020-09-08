---
title: Monitor Azure Functions
description: Obtenga información acerca de cómo usar Azure Application Insights con Azure Functions para supervisar la ejecución de funciones.
ms.assetid: 501722c3-f2f7-4224-a220-6d59da08a320
ms.topic: conceptual
ms.date: 04/04/2019
ms.custom: devx-track-csharp, fasttrack-edit
ms.openlocfilehash: 239d1da028a06d4272ed9b22b624413394aa142f
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212993"
---
# <a name="monitor-azure-functions"></a>Monitor Azure Functions

[Azure Functions](functions-overview.md) ofrece integración incorporada con [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) para supervisar las funciones. En este artículo se muestra cómo configurar Azure Functions para enviar archivos de registro generados por el sistema a Application Insights.

Se recomienda usar Application Insights porque recopila datos de errores, de rendimiento y de registros. Detecta automáticamente anomalías en el rendimiento e incluye herramientas de análisis eficaces que ayudan a diagnosticar problemas y comprender cómo se usan las funciones. Está diseñado para ayudarle a mejorar continuamente el rendimiento y la facilidad de uso. Incluso puede usar Application Insights durante el desarrollo local de proyectos de aplicación de función. Para más información, consulte [¿Qué es Application Insights?](../azure-monitor/app/app-insights-overview.md)

Dado que la instrumentación de Application Insights necesaria está integrada en Azure Functions, todo lo que necesita es una clave de instrumentación válida para conectar la aplicación de función a un recurso de Application Insights. La clave de instrumentación debe agregarse a la configuración de la aplicación cuando se cree el recurso de la aplicación de funciones en Azure. Si la aplicación de funciones aún no tiene esta clave, puede [establecerla de forma manual](#enable-application-insights-integration).  

## <a name="application-insights-pricing-and-limits"></a>Precios y límites de Application Insights

Puede probar la integración de Application Insights con Azure Functions de forma gratuita. Hay un límite diario para la cantidad de datos que se puede procesar de forma gratuita. Podría alcanzar este límite durante las pruebas. Azure proporciona notificaciones del portal y por correo electrónico cuando se aproxima al límite diario. Si pasa por alto esas alertas y llega al límite, los nuevos registros no aparecerán en las consultas de Application Insights. Tenga en cuenta el límite para evitar el tiempo que puede perder, de forma innecesaria, solucionando problemas. Para más información, consulte [Administración de precios y volúmenes de datos de Application Insights](../azure-monitor/app/pricing.md).

> [!IMPORTANT]
> Application Insights tiene una característica de [muestreo](../azure-monitor/app/sampling.md) que le puede ayudar a impedir que se recopilen demasiados datos de telemetría sobre las ejecuciones completadas en los momentos de picos de carga. El muestreo está habilitado de forma predeterminada. Si parece que faltan datos, es posible que simplemente deba ajustar la configuración de muestreo para adaptarla a su escenario de supervisión en particular. Para más información, consulte [Configuración del muestreo](#configure-sampling).

La lista completa de características de Application Insights disponibles para la aplicación de función se detalla en [Características compatibles de Application Insights para Azure Functions](../azure-monitor/app/azure-functions-supported-features.md).

## <a name="view-telemetry-in-monitor-tab"></a>Visualización de datos de telemetría en la pestaña Supervisar

Con la [integración de Application Insights habilitada](#enable-application-insights-integration), puede ver los datos de telemetría en la pestaña **Supervisar**.

1. En la página de la aplicación de función, seleccione una función que se haya ejecutado por lo menos una vez después de que se configurase Application Insights. A continuación, seleccione **Supervisar** en el panel izquierdo. Seleccione **Actualizar** periódicamente hasta que aparezca la lista de las invocaciones de funciones.

   ![Lista de invocaciones](media/functions-monitoring/monitor-tab-ai-invocations.png)

    > [!NOTE]
    > La lista puede tardar hasta cinco minutos en aparecer, mientras el cliente de telemetría agrupa los datos en lotes para su transmisión al servidor. Este retraso no se aplica a [Live Metrics Stream](../azure-monitor/app/live-stream.md). Este servicio se conecta al host de Functions cuando se carga la página, por lo que los registros se transmiten directamente a la página.

1. Para ver los registros de una invocación de función determinada, seleccione el vínculo de la columna **Fecha (UTC)** para esa invocación. La salida del registro para esa invocación aparece en una nueva página.

   ![Detalles de la invocación](media/functions-monitoring/invocation-details-ai.png)

1. Elija **Ejecutar en Application Insights** para ver el origen de la consulta que recupera los datos de registro de Azure Monitor en el registro de Azure. Si esta es la primera vez que usa Azure Log Analytics en la suscripción, se le pedirá que lo habilite.

1. Después de habilitar Log Analytics, se muestra la siguiente consulta. Puede ver que los resultados de la consulta se limitan a los últimos 30 días (`where timestamp > ago(30d)`). Además, los resultados no muestran más de 20 filas (`take 20`). Por el contrario, la lista de detalles de invocación de la función es para los últimos 30 días sin límite.

   ![Lista de invocación del análisis de Application Insights](media/functions-monitoring/ai-analytics-invocation-list.png)

Para más información, consulte la sección [Consultar datos de telemetría](#query-telemetry-data) que se encuentra más adelante en este artículo.

## <a name="view-telemetry-in-application-insights"></a>Visualización de la telemetría en Application Insights

Para abrir Application Insights desde una aplicación de funciones en Azure Portal, seleccione **Application Insights** en **Configuración** en la página de la izquierda. Si esta es la primera vez que usa Application Insights con la suscripción, se le pedirá que lo habilite: seleccione **Activar Application Insights** y, a continuación, **Aplicar** en la página siguiente.

![Abra Application Insights desde la página Información general de la aplicación de función](media/functions-monitoring/ai-link.png)

Para obtener más información acerca de Application Insights, consulte [Documentación de Application Insights](/azure/application-insights/). En esta sección se muestran algunos ejemplos de cómo ver los datos de Application Insights. Si ya está familiarizado con Application Insights, puede ir directamente a [las secciones sobre cómo configurar y personalizar los datos de telemetría](#configure-categories-and-log-levels).

![Pestaña Información general de Application Insights](media/functions-monitoring/metrics-explorer.png)

Las siguientes áreas de Application Insights pueden ser útiles al evaluar el comportamiento, el rendimiento y los errores de las funciones:

| Investigación | Descripción |
| ---- | ----------- |
| **[Errores](../azure-monitor/app/asp-net-exceptions.md)** |  Cree alertas y gráficos basados en errores de funciones y excepciones del servidor. **Nombre de la operación** es el nombre de la función. Los errores en las dependencias no se muestran, a no ser que se implemente telemetría personalizada para las dependencias. |
| **[Rendimiento](../azure-monitor/app/performance-counters.md)** | Analice los problemas de rendimiento mediante la visualización del uso de recursos y el rendimiento por **instancias de rol de nube**. Estos datos pueden ser útiles para la depuración de escenarios en los que las funciones están dificultando los recursos subyacentes. |
| **[Métricas](../azure-monitor/platform/metrics-charts.md)** | Cree gráficos y alertas basados en métricas. Las métricas incluyen el número de invocaciones de función, el tiempo de ejecución y las tasas de éxito. |
| **[Live Metrics    ](../azure-monitor/app/live-stream.md)** | Vea los datos de métricas a medida que se crean casi en tiempo real. |

## <a name="query-telemetry-data"></a>Consultar datos de telemetría

El [Análisis de Application Insights](../azure-monitor/log-query/log-query-overview.md) proporciona acceso a todos los datos de telemetría en forma de tabla en una base de datos. Analytics ofrece un lenguaje de consulta para extraer, manipular y visualizar los datos. 

Elija **Registros** para explorar o consultar eventos registrados.

![Ejemplo de Analytics](media/functions-monitoring/analytics-traces.png)

Este es un ejemplo de consulta que muestra la distribución de solicitudes por trabajador durante los últimos 30 minutos.

<pre>
requests
| where timestamp > ago(30m) 
| summarize count() by cloud_RoleInstance, bin(timestamp, 1m)
| render timechart
</pre>

Las tablas que están disponibles se muestran en la pestaña **Esquema** de la izquierda. Encontrará los datos generados por las invocaciones de función en las tablas siguientes:

| Tabla | Descripción |
| ----- | ----------- |
| **traces** | Registros creados por el tiempo de ejecución y por el código de la función. |
| **requests** | Una solicitud por cada invocación de función. |
| **exceptions** | Cualquier excepción que produzca el tiempo de ejecución. |
| **customMetrics** | Recuento de invocaciones correctas y erróneas, tasa de éxito y duración. |
| **customEvents** | Eventos registrados por el tiempo de ejecución, por ejemplo: solicitudes HTTP que desencadenan una función. |
| **performanceCounters** | Información acerca del rendimiento de los servidores en los que se ejecutan las funciones. |

El resto de las tablas son para pruebas de disponibilidad, y telemetría de explorador y de cliente. Puede implementar telemetría personalizada para agregarles datos.

Dentro de cada tabla, algunos de los datos específicos de Functions están en un campo `customDimensions`.  Por ejemplo, la consulta siguiente recupera todos los seguimientos que tienen el nivel de registro `Error`.

<pre>
traces 
| where customDimensions.LogLevel == "Error"
</pre>

El tiempo de ejecución proporciona los campos `customDimensions.LogLevel` y `customDimensions.Category`. Puede proporcionar campos adicionales en los registros que escriba en el código de función. Consulte [Registro estructurado](#structured-logging) más adelante en este artículo.

## <a name="configure-categories-and-log-levels"></a>Configurar categorías y niveles de registro

Puede usar Application Insights sin ninguna configuración personalizada. La configuración predeterminada puede dar lugar a grandes volúmenes de datos. Si usa una suscripción de Visual Studio Azure, es posible que alcance su límite de datos para Application Insights. Más adelante en este artículo verá cómo configurar y personalizar los datos que envían las funciones a Application Insights. Para una aplicación de función, el registro se configura en el archivo [host.json].

### <a name="categories"></a>Categorías

El registrador de Azure Functions incluye una *categoría* para cada registro. La categoría indica qué parte del código de tiempo de ejecución o del código de la función escribió el registro. En el siguiente gráfico se describen las categorías principales de registros que crea el runtime. 

| Category | Descripción |
| ----- | ----- | 
| Host.Results | Estos registros se muestran como **requests** en Application Insights. e indican el éxito o fracaso de una función. Todos estos registros se escriben en el nivel `Information`. Si filtra por `Warning` o superior, no verá ninguno de estos datos. |
| Host.Aggregator | Estos registros proporcionan los recuentos y promedios de las invocaciones de función en un período de tiempo [configurable](#configure-the-aggregator). El período predeterminado es 30 segundos o 1000 resultados, lo que ocurra primero. Los registros están disponibles en la tabla **customMetrics** de Application Insights. Algunos ejemplos son el número de ejecuciones, la tasa de éxito y la duración. Todos estos registros se escriben en el nivel `Information`. Si filtra por `Warning` o superior, no verá ninguno de estos datos. |

Todos los registros de categorías diferentes a las que están disponibles en la tabla **traces** en Application Insights.

Todos los registros con categorías que comienzan por `Host` los escribe el tiempo de ejecución de Functions. Los registros **Función iniciada** y **Función concluida** tienen la categoría `Host.Executor`. Para las ejecuciones correctas, estos registros son de nivel `Information`. Las excepciones se registran en el nivel `Error`. El tiempo de ejecución también crea registros del nivel `Warning`; por ejemplo, poner en cola los mensajes enviados a la cola de dudosos.

El tiempo de ejecución de Functions crea registros con una categoría que empieza con "Host". En la versión 1.x, los registros `function started`, `function executed` y `function completed` tienen la categoría `Host.Executor`. A partir de la versión 2.x, estos registros tienen la categoría `Function.<YOUR_FUNCTION_NAME>`.

Si escribe registros en el código de la función, la categoría es `Function.<YOUR_FUNCTION_NAME>.User` y puede ser cualquier nivel de registro. En la versión 1.x del runtime de Functions, la categoría es `Function`.

### <a name="log-levels"></a>Niveles de registro

El registrador de Azure Functions también incluye un *nivel de registro* con cada registro. [LogLevel](/dotnet/api/microsoft.extensions.logging.loglevel) es una enumeración y el código entero indica la importancia relativa:

|LogLevel    |Código|
|------------|---|
|Seguimiento       | 0 |
|Depurar       | 1 |
|Información | 2 |
|Advertencia     | 3 |
|Error       | 4 |
|Crítico    | 5 |
|None        | 6 |

El nivel de registro `None` se explica en la sección siguiente. 

### <a name="log-configuration-in-hostjson"></a>Configuración del registro en host.json

El archivo [host.json] configura la cantidad de registro que envía una aplicación de la función a Application Insights. Para cada categoría, debe indicar el nivel de registro mínimo para enviar. Hay dos ejemplos: el primero tiene como destino la [versión 2.x y posteriores](functions-versions.md#version-2x) del entorno de ejecución de Functions (con .NET Core) y el segundo ejemplo es para el entorno de ejecución de la versión 1.x.

### <a name="version-2x-and-higher"></a>Versión 2.x y posteriores

La versión v2.x y las versiones posteriores del entorno de ejecución de Functions usan la [jerarquía de filtros de registro de .NET Core](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1#log-filtering). 

```json
{
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error",
      "Function": "Error",
      "Host.Aggregator": "Trace"
    }
  }
}
```

### <a name="version-1x"></a>Versión 1.x

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host.Results": "Error",
        "Function": "Error",
        "Host.Aggregator": "Trace"
      }
    }
  }
}
```

En el ejemplo siguiente se configuran las reglas siguientes:

* Para los registros con la categoría `Host.Results` o `Function`, envíe solo el nivel `Error` y superior a Application Insights. Los registros del nivel `Warning` e inferiores se omiten.
* Para los registros con la categoría `Host.Aggregator`, envíe todos los registros a Application Insights. El nivel de registro `Trace` es el mismo que algunos registradores llaman `Verbose`, pero usa `Trace` en el archivo [host.json].
* Para el resto de registros, envíe solo el nivel `Information` y superior a Application Insights.

El valor de categoría de [host.json] controla el registro de todas las categorías que comienzan con el mismo valor. `Host` en [host.json] controla el registro de `Host.General`, `Host.Executor`, `Host.Results` y así sucesivamente.

Si [host.json] incluye varias categorías que comienzan con la misma cadena, las más largas se asignan primero. Imagine que quiere que todo el tiempo de ejecución excepto `Host.Aggregator` se registre en el nivel `Error`, pero que `Host.Aggregator` lo haga en el nivel `Information`:

### <a name="version-2x-and-later"></a>Versión 2.x y posterior

```json
{
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "default": "Information",
      "Host": "Error",
      "Function": "Error",
      "Host.Aggregator": "Information"
    }
  }
}
```

### <a name="version-1x"></a>Versión 1.x 

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host": "Error",
        "Function": "Error",
        "Host.Aggregator": "Information"
      }
    }
  }
}
```

Para eliminar todos los registros de una categoría, puede usar el nivel de registro `None`. No hay ningún registro escrito con esa categoría y no existe ningún nivel de registro por encima de él.

## <a name="configure-the-aggregator"></a>Configurar el agregador

Como se indicó en la sección anterior, el tiempo de ejecución agrega datos acerca de las ejecuciones de la función durante un período de tiempo. El período predeterminado es 30 segundos o 1000 ejecuciones, lo que ocurra primero. Puede configurar este valor en el archivo [host.json].  Este es un ejemplo:

```json
{
    "aggregator": {
      "batchSize": 1000,
      "flushTimeout": "00:00:30"
    }
}
```

## <a name="configure-sampling"></a>Configurar el muestreo

Application Insights tiene una característica de [muestreo](../azure-monitor/app/sampling.md) que le puede ayudar a impedir que se recopilen demasiados datos de telemetría sobre las ejecuciones completadas en los momentos de picos de carga. Cuando tasa de ejecuciones entrantes supera un umbral especificado, Application Insights empieza a omitir aleatoriamente algunas de las ejecuciones entrantes. La configuración predeterminada para el número máximo de ejecuciones por segundo es de 20 (cinco en la versión 1.x). Puede configurar el muestreo en [host.json](./functions-host-json.md#applicationinsights).  Este es un ejemplo:

### <a name="version-2x-and-later"></a>Versión 2.x y posterior

```json
{
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond" : 20,
        "excludedTypes": "Request"
      }
    }
  }
}
```

En la versión 2. x, puede excluir determinados tipos de telemetría del muestreo. En el ejemplo anterior, los datos de tipo `Request` se excluyen del muestreo. Esto garantiza el registro de *todas* las ejecuciones de funciones (solicitudes) mientras que otros tipos de telemetría siguen sujetos al muestreo.

### <a name="version-1x"></a>Versión 1.x 

```json
{
  "applicationInsights": {
    "sampling": {
      "isEnabled": true,
      "maxTelemetryItemsPerSecond" : 5
    }
  }
}
```

## <a name="write-logs-in-c-functions"></a>Escribir registros en funciones de C#

Puede escribir registros en el código de función que aparecen como seguimientos de Application Insights.

### <a name="ilogger"></a>ILogger

Use un parámetro [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger) en las funciones en lugar de un parámetro `TraceWriter`. Los registros creados con `TraceWriter` se dirigen a Application Insights, pero `ILogger` le permite hacer un [registro estructurado](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).

Con un objeto `ILogger`, llamará a los [métodos de extensión en ILogger](/dotnet/api/microsoft.extensions.logging.loggerextensions#methods) `Log<level>` para crear registros. El código siguiente escribe registros `Information` con la categoría "Function.<SU_NOMBRE_DE_FUNCIÓN>.User".

```cs
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger logger)
{
    logger.LogInformation("Request for item with key={itemKey}.", id);
```

### <a name="structured-logging"></a>Registro estructurado

El orden de los marcadores de posición, no sus nombres, determina qué parámetros se usan en el mensaje del registro. Supongamos que tiene el siguiente código:

```csharp
string partitionKey = "partitionKey";
string rowKey = "rowKey";
logger.LogInformation("partitionKey={partitionKey}, rowKey={rowKey}", partitionKey, rowKey);
```

Si mantiene la misma cadena de mensaje e invierte el orden de los parámetros, el texto del mensaje resultante tendrá los valores en los lugares incorrectos.

Los marcadores de posición se controlan de esta forma para que pueda hacer el registro estructurado. Application Insights almacena los pares de nombre-valor del parámetro y la cadena de mensaje. Como resultado, los argumentos del mensaje se convierten en campos que puede consultar.

Si la llamada de método del registrador tiene un aspecto similar al ejemplo anterior, puede consultar el campo `customDimensions.prop__rowKey`. Se agrega el prefijo `prop__` para asegurar que no haya ninguna colisión entre los campos que agrega el tiempo de ejecución y los campos que agrega el código de la función.

También puede consultar la cadena del mensaje original haciendo referencia al campo `customDimensions.prop__{OriginalFormat}`.  

Esta es una representación de JSON de ejemplo de los datos `customDimensions`:

```json
{
  "customDimensions": {
    "prop__{OriginalFormat}":"C# Queue trigger function processed: {message}",
    "Category":"Function",
    "LogLevel":"Information",
    "prop__message":"c9519cbf-b1e6-4b9b-bf24-cb7d10b1bb89"
  }
}
```

### <a name="custom-metrics-logging"></a>Registro de métricas personalizadas

En funciones de secuencia de comandos de C#, puede usar el método de extensión `LogMetric` en `ILogger` para crear métricas personalizadas en Application Insights. Este es un ejemplo de una llamada de método:

```csharp
logger.LogMetric("TestMetric", 1234);
```

Este código es una alternativa a llamar a `TrackMetric` con la API de Application Insights para .NET.

## <a name="write-logs-in-javascript-functions"></a>Escribir registros en las funciones de JavaScript

En las funciones de Node.js, use `context.log` para escribir los registros. El registro estructurado no está habilitado.

```
context.log('JavaScript HTTP trigger function processed a request.' + context.invocationId);
```

### <a name="custom-metrics-logging"></a>Registro de métricas personalizadas

Cuando ejecuta la [versión 1.x](functions-versions.md#creating-1x-apps) del sistema en tiempo de ejecución de Functions, las funciones de Node.js pueden usar el método `context.log.metric` para crear métricas personalizadas en Application Insights. Este método no se admite actualmente en la versión 2.x y posteriores. Este es un ejemplo de una llamada de método:

```javascript
context.log.metric("TestMetric", 1234);
```

Este código es una alternativa a llamar a `trackMetric` con el SDK de Node.js de Application Insights.

## <a name="log-custom-telemetry-in-c-functions"></a>Registrar telemetría personalizada en funciones de C#

Hay una versión específica de funciones del SDK de Application Insights que puede usar para enviar datos de telemetría personalizados desde las funciones a Application Insights: [Microsoft.Azure.WebJobs.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights). Use el comando siguiente desde el símbolo del sistema para instalar este paquete:

# <a name="command"></a>[Comando](#tab/cmd)

```cmd
dotnet add package Microsoft.Azure.WebJobs.Logging.ApplicationInsights --version <VERSION>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -Version <VERSION>
```

---

En este comando, reemplace `<VERSION>` por una versión de este paquete que admita la versión instalada de [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs/). 

En el siguiente ejemplo de C# se usa la [API de telemetría personalizada](../azure-monitor/app/api-custom-events-metrics.md). El ejemplo es para una biblioteca de clases. NET, pero el código de Application Insights es el mismo para la secuencia de comandos de C#.

### <a name="version-2x-and-later"></a>Versión 2.x y posterior

La versión 2.x y versiones posteriores del entorno de ejecución utilizan las características más recientes de Application Insights para correlacionar automáticamente la telemetría con la operación actual. No es necesario establecer manualmente los campos de operación `Id`, `ParentId` o `Name`.

```cs
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
using System.Linq;

namespace functionapp0915
{
    public class HttpTrigger2
    {
        private readonly TelemetryClient telemetryClient;

        /// Using dependency injection will guarantee that you use the same configuration for telemetry collected automatically and manually.
        public HttpTrigger2(TelemetryConfiguration telemetryConfiguration)
        {
            this.telemetryClient = new TelemetryClient(telemetryConfiguration);
        }

        [FunctionName("HttpTrigger2")]
        public Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = null)]
            HttpRequest req, ExecutionContext context, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            DateTime start = DateTime.UtcNow;

            // Parse query parameter
            string name = req.Query
                .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
                .Value;

            // Track an Event
            var evt = new EventTelemetry("Function called");
            evt.Context.User.Id = name;
            this.telemetryClient.TrackEvent(evt);

            // Track a Metric
            var metric = new MetricTelemetry("Test Metric", DateTime.Now.Millisecond);
            metric.Context.User.Id = name;
            this.telemetryClient.TrackMetric(metric);

            // Track a Dependency
            var dependency = new DependencyTelemetry
            {
                Name = "GET api/planets/1/",
                Target = "swapi.co",
                Data = "https://swapi.co/api/planets/1/",
                Timestamp = start,
                Duration = DateTime.UtcNow - start,
                Success = true
            };
            dependency.Context.User.Id = name;
            this.telemetryClient.TrackDependency(dependency);

            return Task.FromResult<IActionResult>(new OkResult());
        }
    }
}
```

[GetMetric](../azure-monitor/app/api-custom-events-metrics.md#getmetric) es la API recomendada actualmente para crear una métrica.

### <a name="version-1x"></a>Versión 1.x

```cs
using System;
using System.Net;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Azure.WebJobs;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Linq;

namespace functionapp0915
{
    public static class HttpTrigger2
    {
        private static string key = TelemetryConfiguration.Active.InstrumentationKey = 
            System.Environment.GetEnvironmentVariable(
                "APPINSIGHTS_INSTRUMENTATIONKEY", EnvironmentVariableTarget.Process);

        private static TelemetryClient telemetryClient = 
            new TelemetryClient() { InstrumentationKey = key };

        [FunctionName("HttpTrigger2")]
        public static async Task<HttpResponseMessage> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
            HttpRequestMessage req, ExecutionContext context, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            DateTime start = DateTime.UtcNow;

            // Parse query parameter
            string name = req.GetQueryNameValuePairs()
                .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
                .Value;

            // Get request body
            dynamic data = await req.Content.ReadAsAsync<object>();

            // Set name to query string or body data
            name = name ?? data?.name;
         
            // Track an Event
            var evt = new EventTelemetry("Function called");
            UpdateTelemetryContext(evt.Context, context, name);
            telemetryClient.TrackEvent(evt);
            
            // Track a Metric
            var metric = new MetricTelemetry("Test Metric", DateTime.Now.Millisecond);
            UpdateTelemetryContext(metric.Context, context, name);
            telemetryClient.TrackMetric(metric);
            
            // Track a Dependency
            var dependency = new DependencyTelemetry
                {
                    Name = "GET api/planets/1/",
                    Target = "swapi.co",
                    Data = "https://swapi.co/api/planets/1/",
                    Timestamp = start,
                    Duration = DateTime.UtcNow - start,
                    Success = true
                };
            UpdateTelemetryContext(dependency.Context, context, name);
            telemetryClient.TrackDependency(dependency);
        }
        
        // Correlate all telemetry with the current Function invocation
        private static void UpdateTelemetryContext(TelemetryContext context, ExecutionContext functionContext, string userName)
        {
            context.Operation.Id = functionContext.InvocationId.ToString();
            context.Operation.ParentId = functionContext.InvocationId.ToString();
            context.Operation.Name = functionContext.FunctionName;
            context.User.Id = userName;
        }
    }    
}
```

No llame a `TrackRequest` ni a `StartOperation<RequestTelemetry>`, ya que verá solicitudes duplicadas de invocaciones de función.  El tiempo de ejecución de Functions realiza un seguimiento automático de las solicitudes.

No establezca `telemetryClient.Context.Operation.Id`. Esta configuración global provoca una correlación incorrecta cuando muchas funciones se ejecuten de manera simultánea. En su lugar, cree una instancia de telemetría nueva (`DependencyTelemetry`, `EventTelemetry`) y modifique su propiedad `Context`. Luego, pase la instancia de telemetría al método `Track` correspondiente en `TelemetryClient` (`TrackDependency()`, `TrackEvent()`, `TrackMetric()`). Este método garantiza que la telemetría tenga los detalles de correlación correctos para la invocación actual de la función.

## <a name="log-custom-telemetry-in-javascript-functions"></a>Registrar telemetría personalizada en funciones de JavaScript

A continuación, se muestran ejemplos de fragmentos de código que envían telemetría personalizada con el [SDK de Node.js para Application Insights](https://github.com/microsoft/applicationinsights-node.js):

### <a name="version-2x-and-later"></a>Versión 2.x y posterior

```javascript
const appInsights = require("applicationinsights");
appInsights.setup();
const client = appInsights.defaultClient;

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    // Use this with 'tagOverrides' to correlate custom telemetry to the parent function invocation.
    var operationIdOverride = {"ai.operation.id":context.traceContext.traceparent};

    client.trackEvent({name: "my custom event", tagOverrides:operationIdOverride, properties: {customProperty2: "custom property value"}});
    client.trackException({exception: new Error("handled exceptions can be logged with this method"), tagOverrides:operationIdOverride});
    client.trackMetric({name: "custom metric", value: 3, tagOverrides:operationIdOverride});
    client.trackTrace({message: "trace message", tagOverrides:operationIdOverride});
    client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL", tagOverrides:operationIdOverride});
    client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true, tagOverrides:operationIdOverride});

    context.done();
};
```

### <a name="version-1x"></a>Versión 1.x

```javascript
const appInsights = require("applicationinsights");
appInsights.setup();
const client = appInsights.defaultClient;

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    // Use this with 'tagOverrides' to correlate custom telemetry to the parent function invocation.
    var operationIdOverride = {"ai.operation.id":context.operationId};

    client.trackEvent({name: "my custom event", tagOverrides:operationIdOverride, properties: {customProperty2: "custom property value"}});
    client.trackException({exception: new Error("handled exceptions can be logged with this method"), tagOverrides:operationIdOverride});
    client.trackMetric({name: "custom metric", value: 3, tagOverrides:operationIdOverride});
    client.trackTrace({message: "trace message", tagOverrides:operationIdOverride});
    client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL", tagOverrides:operationIdOverride});
    client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true, tagOverrides:operationIdOverride});

    context.done();
};
```

El parámetro `tagOverrides` establece `operation_Id` en el identificador de invocación de la función. Esta configuración le permite correlacionar toda la telemetría personalizada y generada automáticamente para una determinada invocación de función.

## <a name="dependencies"></a>Dependencias

Functions v2 recopila automáticamente las dependencias de solicitudes HTTP, ServiceBus, EventHub y SQL.

Puede escribir código personalizado para mostrar las dependencias. Para ver ejemplos, consulte el código de ejemplo en la [sección de telemetría personalizada de C#](#log-custom-telemetry-in-c-functions). El código de ejemplo da como resultado un *mapa de aplicación* de Application Insights, que tiene un aspecto como el de la siguiente imagen:

![Mapa de aplicación](./media/functions-monitoring/app-map.png)

> [!NOTE]
> Las dependencias se escriben en el nivel de Información. Si filtra por Advertencia o superior, no verá ninguno de estos datos. Además, la recopilación automática de dependencias se produce en un ámbito que no es de usuario. Por lo tanto, asegúrese de que el nivel esté establecido en, al menos, **Información** fuera del ámbito de usuario en host.json (es decir, fuera de la clave Function.<NOMBRE_DE_LA_FUNCIÓN>.User) si desea que se capturen esas dependencias.

## <a name="enable-application-insights-integration"></a>Habilitación de la integración de Application Insights

Para que una aplicación de función envíe datos a Application Insights tiene que conocer la clave de instrumentación de un recurso de Application Insights. La clave tiene que estar en un valor de configuración de la aplicación llamado **APPINSIGHTS_INSTRUMENTATIONKEY**.

Al crear la función de aplicaciones [en Azure Portal](functions-create-first-azure-function.md), desde la línea de comandos mediante [Azure Functions Core Tools](functions-create-first-azure-function-azure-cli.md) o [Visual Studio Code](functions-create-first-function-vs-code.md), la integración de Application Insights está habilitada de forma predeterminada. El recurso de Application Insights tiene el mismo nombre que la aplicación de función y se crea en la misma región o en la región más cercana.

### <a name="new-function-app-in-the-portal"></a>Nueva aplicación de función en el portal

Para revisar el recurso de Application Insights que se está creando, selecciónelo para expandir la ventana **Application Insights**. Puede cambiar el valor de **Nuevo nombre de recurso**  o elegir otro valor en **Ubicación** en la [ubicación geográfica de Azure](https://azure.microsoft.com/global-infrastructure/geographies/) donde quiera almacenar los datos.

![Habilitar Application Insights al crear una aplicación de función](media/functions-monitoring/enable-ai-new-function-app.png)

Cuando elige **Crear**, se crea un recurso de Application Insights con la aplicación de función, que tiene `APPINSIGHTS_INSTRUMENTATIONKEY` establecido en la configuración de la aplicación. Todo está listo para funcionar.

<a id="manually-connect-an-app-insights-resource"></a>
### <a name="add-to-an-existing-function-app"></a>Adición a una aplicación de función existente 

Al crear una aplicación de funciones mediante [Visual Studio](functions-create-your-first-function-visual-studio.md), debe crear el recurso de Application Insights. A continuación, puede agregar la clave de instrumentación de ese recurso como una [configuración de la aplicación](functions-how-to-use-azure-function-app-settings.md#settings) en la aplicación de funciones.

[!INCLUDE [functions-connect-new-app-insights.md](../../includes/functions-connect-new-app-insights.md)]

Las primeras versiones de Functions usaban la supervisión integrada, que ya no se recomienda. Al habilitar la integración de Application Insights para una aplicación de función tal, también debe [deshabilitar el registro integrado](#disable-built-in-logging).  

## <a name="report-issues"></a>Informar de problemas

Para notificar un problema con la integración de Application Insights en Functions o para realizar una solicitud o sugerencia [cree un problema en GitHub](https://github.com/Azure/Azure-Functions/issues/new).

## <a name="streaming-logs"></a>Registros de transmisión

A menudo, cuando desarrolla una aplicación, quiere ver lo que se escribe en los registros casi en tiempo real cuando se ejecuta en Azure.

Hay dos maneras de ver una secuencia de los archivos de registro que generan las ejecuciones de la función.

* **Streaming integrado de registros**: la plataforma de App Service le permite ver una secuencia de los archivos de registro de aplicaciones. Esto equivale a la salida que se ve al depurar las funciones durante el [desarrollo local](functions-develop-local.md) y cuando se usa la pestaña **Prueba** del portal. Se muestra toda la información basada en el registro. Para más información, consulte [Registros de Stream](../app-service/troubleshoot-diagnostic-logs.md#stream-logs). Este método de streaming solo admite una instancia y no se puede usar con una aplicación que se ejecuta en Linux en un plan de consumo.

* **Live Metrics Stream**: cuando la aplicación de función está [conectada a Application Insights](#enable-application-insights-integration), puede ver los datos de registro y otras métricas casi en tiempo real en Azure Portal mediante [Live Metrics Stream](../azure-monitor/app/live-stream.md). Use este método cuando supervise las funciones que se ejecutan en varias instancias o en Linux en un plan de consumo. En este método se usan [datos muestreados](#configure-sampling).

Las secuencias de registro se pueden ver tanto en el portal como en la mayoría de los entornos de desarrollo locales. 

### <a name="portal"></a>Portal

Puede ver ambos tipos de secuencias de registro en el portal.

#### <a name="built-in-log-streaming"></a>Streaming integrado de registros

Para ver los registros de secuencias en el portal, seleccione la pestaña **Características de la plataforma** en la aplicación de función. Luego, en **Supervisión**, elija **Secuencias de registro**.

![Habilitar los registros de secuencias en el portal](./media/functions-monitoring/enable-streaming-logs-portal.png)

Esto conecta la aplicación con el servicio de secuencias de registro, y los registros de aplicación se muestran en la ventana. Puede alternar entre **Registros de aplicaciones** y **Registros de servidor web**.  

![Ver los registros de secuencias en el portal](./media/functions-monitoring/streaming-logs-window.png)

#### <a name="live-metrics-stream"></a>Secuencia de métricas en directo

Para ver Live Metrics Stream de su aplicación, seleccione la pestaña **Información general** de la aplicación de funciones. Cuando haya habilitado Application Insights, verá un vínculo a **Application Insights** en **Características configuradas**. Este vínculo lo llevará a la página Application Insights correspondiente a la aplicación.

En Application Insights, seleccione **Live Metrics Stream**. Las [entradas de los registros muestreados](#configure-sampling) se muestran en **Telemetría de ejemplo**.

![Vista de Live Metrics Stream en el portal](./media/functions-monitoring/live-metrics-stream.png) 

### <a name="visual-studio-code"></a>Visual Studio Code

[!INCLUDE [functions-enable-log-stream-vs-code](../../includes/functions-enable-log-stream-vs-code.md)]

### <a name="core-tools"></a>Core Tools

[!INCLUDE [functions-streaming-logs-core-tools](../../includes/functions-streaming-logs-core-tools.md)]

### <a name="azure-cli"></a>Azure CLI

Puede habilitar los registros de secuencias con la [CLI de Azure](/cli/azure/install-azure-cli). Use los siguientes comandos para iniciar sesión, elija su suscripción y transmita los archivos de registro:

```azurecli
az login
az account list
az account set --subscription <subscriptionNameOrId>
az webapp log tail --resource-group <RESOURCE_GROUP_NAME> --name <FUNCTION_APP_NAME>
```

### <a name="azure-powershell"></a>Azure PowerShell

Puede habilitar los registros de secuencias con [Azure PowerShell](/powershell/azure/). Para PowerShell, use el comando [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) para habilitar el registro en la aplicación de funciones, como se muestra en el siguiente fragmento de código: 

:::code language="powershell" source="~/powershell_scripts/app-service/monitor-with-logs/monitor-with-logs.ps1" range="19-20":::

Para obtener más información, vea el [código de ejemplo completo](../app-service/scripts/powershell-monitor.md#sample-script). 

## <a name="scale-controller-logs-preview"></a>Registros del controlador de escala (versión preliminar)

Esta característica se encuentra en su versión preliminar. 

El [controlador de escala de Azure Functions](./functions-scale.md#runtime-scaling) supervisa las instancias del host de Azure Functions en el que se ejecuta la aplicación. Este controlador toma decisiones sobre cuándo agregar o quitar instancias en función del rendimiento actual. Puede hacer que el controlador de escala emita registros en Application Insights o en Blob Storage para comprender mejor las decisiones que está tomando el controlador de escala para la aplicación de funciones.

Para habilitar esta característica, agregue una nueva configuración de la aplicación denominada `SCALE_CONTROLLER_LOGGING_ENABLED`. El valor de esta configuración debe tener el formato `<DESTINATION>:<VERBOSITY>`, en función de lo siguiente:

[!INCLUDE [functions-scale-controller-logging](../../includes/functions-scale-controller-logging.md)]

Por ejemplo, el siguiente comando de la CLI de Azure activa el registro detallado del controlador de escala para Application Insights:

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings SCALE_CONTROLLER_LOGGING_ENABLED=AppInsights:Verbose
```

En este ejemplo, reemplace `<FUNCTION_APP_NAME>` y `<RESOURCE_GROUP_NAME>` por el nombre de la aplicación de funciones y el nombre del grupo de recursos, respectivamente. 

El siguiente comando de la CLI de Azure deshabilita el registro estableciendo el nivel de detalle en `None`:

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings SCALE_CONTROLLER_LOGGING_ENABLED=AppInsights:None
```

También puede deshabilitar el registro si quita la configuración de `SCALE_CONTROLLER_LOGGING_ENABLED` con el siguiente comando de la CLI de Azure:

```azurecli-interactive
az functionapp config appsettings delete --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--setting-names SCALE_CONTROLLER_LOGGING_ENABLED
```

## <a name="disable-built-in-logging"></a>Deshabilitar el registro integrado

Cuando habilite Application Insights, deshabilite el registro integrado que usa Azure Storage. El registro integrado es útil para realizar pruebas con cargas de trabajo ligeras, pero no está diseñado para su uso en producción de alta carga. Para la supervisión de producción, se recomienda Application Insights. Si el registro integrado se usa en producción, el registro resultante podría estar incompleto debido a la limitación de Azure Storage.

Para deshabilitar el registro integrado, elimine la configuración de la aplicación `AzureWebJobsDashboard`. Para obtener información acerca de cómo eliminar la configuración de la aplicación en Azure Portal, consulte la sección **Application settings** (Configuración de la aplicación) en [How to manage a function app](functions-how-to-use-azure-function-app-settings.md#settings) (Cómo administrar una aplicación de función). Antes de eliminar el valor de configuración de la aplicación, asegúrese de que no haya ninguna función existente en la misma aplicación de función que utilice dicho valor para los desencadenadores o enlaces de Azure Storage.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los siguientes recursos:

* [Application Insights](/azure/application-insights/)
* [Registro de ASP.NET Core](/aspnet/core/fundamentals/logging/)

[host.json]: functions-host-json.md
