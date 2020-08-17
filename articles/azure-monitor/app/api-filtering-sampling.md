---
title: Filtro y preprocesamiento en el SDK de Application Insights | Microsoft Docs
description: Escriba procesadores e inicializadores de telemetría para que el SDK filtre o agregue propiedades a los datos antes de enviar la telemetría al portal de Application Insights.
ms.topic: conceptual
ms.date: 11/23/2016
ms.custom: devx-track-javascript
ms.openlocfilehash: eec3cf44eb516ce20db564e1bed32e5741bfd02a
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87366762"
---
# <a name="filter-and-preprocess-telemetry-in-the-application-insights-sdk"></a>Filtro y preprocesamiento de la telemetría en el SDK de Application Insights

Puede escribir y configurar complementos para el SDK de Application Insights con el fin de personalizar cómo se enriquece y se procesa la telemetría antes de enviarla al servicio de Application Insights.

* [muestreo](sampling.md) reduce el volumen de la telemetría sin que ello influya en las estadísticas. Mantiene juntos los puntos de datos relacionados para que pueda navegar entre ellos a la hora de diagnosticar un problema. En el portal, se multiplican los recuentos totales para compensar el muestreo.
* El filtro con procesadores de telemetría permite filtrar la telemetría en el SDK antes de enviarla al servidor. Por ejemplo, para reducir el volumen de la telemetría puede excluir las solicitudes de robots. El filtro constituye un enfoque más básico que el muestreo para reducir el tráfico. Le permite tener un mayor control sobre lo que se transmite, aunque afecta a las estadísticas. Por ejemplo, puede filtrar todas las solicitudes correctas.
* [Los inicializadores de telemetría agregan o modifican propiedades](#add-properties) a cualquier telemetría enviada desde la aplicación, lo que incluye la telemetría de los módulos estándar. Por ejemplo, puede agregar valores calculados o números de versión para filtrar los datos en el portal.
* [API del SDK](./api-custom-events-metrics.md) se usa para enviar métricas y eventos personalizados.

Antes de comenzar:

* Instale el SDK adecuado para su aplicación: [ASP.NET](asp-net.md), [ASP.NET Core](asp-net-core.md), [No HTTP/Trabajo para .NET/.NET Core](worker-service.md) o [JavaScript](javascript.md).

<a name="filtering"></a>

## <a name="filtering"></a>Filtros

Esta técnica le ofrece un control directo sobre lo que se incluirá en la transmisión de telemetría o lo que se excluirá de ella. El filtrado se puede usar para impedir el envío de elementos de telemetría a Application Insights. Puede utilizar el filtrado junto con el muestreo o por separado.

Para filtrar la telemetría, escriba un procesador de telemetría y regístrelo con `TelemetryConfiguration`. Toda la telemetría pasa por el procesador. Puede optar por no incluirla en la transmisión o por proporcionársela al procesador siguiente en la cadena. Esto incluye la telemetría de los módulos estándar, como el recopilador de solicitudes HTTP y el recopilador de dependencia, así como la telemetría de la que haya realizado un seguimiento. Por ejemplo, puede filtrar para dejar fuera la telemetría acerca de las solicitudes de robots o las llamadas de dependencia correctas.

> [!WARNING]
> El filtrado de la telemetría enviada desde el SDK usando procesadores puede sesgar las estadísticas que se ven en el portal y dificultar el seguimiento de elementos relacionados.
>
> En su lugar, puede efectuar un [muestreo](./sampling.md).
>
>

### <a name="create-a-telemetry-processor-c"></a>Creación de un procesador de telemetría (C#)

1. Para crear un filtro, implemente `ITelemetryProcessor`.

    Los procesadores de telemetría forman una cadena de procesamiento. Al crear instancias de un procesador de telemetría, se le proporciona una referencia al procesador siguiente en la cadena. Cuando un punto de datos de telemetría se pasa al método de proceso, realiza su trabajo y, a continuación, llama (o no) al procesador de telemetría siguiente en la cadena.

    ```csharp
    using Microsoft.ApplicationInsights.Channel;
    using Microsoft.ApplicationInsights.Extensibility;

    public class SuccessfulDependencyFilter : ITelemetryProcessor
    {
        private ITelemetryProcessor Next { get; set; }

        // next will point to the next TelemetryProcessor in the chain.
        public SuccessfulDependencyFilter(ITelemetryProcessor next)
        {
            this.Next = next;
        }

        public void Process(ITelemetry item)
        {
            // To filter out an item, return without calling the next processor.
            if (!OKtoSend(item)) { return; }

            this.Next.Process(item);
        }

        // Example: replace with your own criteria.
        private bool OKtoSend (ITelemetry item)
        {
            var dependency = item as DependencyTelemetry;
            if (dependency == null) return true;

            return dependency.Success != true;
        }
    }
    ```

2. Agregue el procesador.

**Aplicaciones** ASP.NET

Inserte este fragmento en ApplicationInsights.config:

```xml
<TelemetryProcessors>
  <Add Type="WebApplication9.SuccessfulDependencyFilter, WebApplication9">
     <!-- Set public property -->
     <MyParamFromConfigFile>2-beta</MyParamFromConfigFile>
  </Add>
</TelemetryProcessors>
```

Puede pasar valores de cadena desde el archivo .config proporcionando propiedades con nombre públicas en la clase.

> [!WARNING]
> Tenga cuidado para que el nombre de tipo y los nombres de propiedad del archivo .config coincidan con los nombres de clase y propiedad del código. Si el archivo .config hace referencia a un tipo o propiedad que no existe, el SDK puede producir un error de forma silenciosa al enviar cualquier telemetría.
>

Alternativamente, se puede inicializar el filtro en el código. En una clase de inicialización adecuada (por ejemplo, AppStart de `Global.asax.cs`), inserte el procesador en la cadena:

```csharp
var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
builder.Use((next) => new SuccessfulDependencyFilter(next));

// If you have more processors:
builder.Use((next) => new AnotherProcessor(next));

builder.Build();
```

Los clientes de telemetría creados a partir de este punto usarán sus procesadores.

**Aplicaciones Core/ Servicio de trabajo** de ASP.NET

> [!NOTE]
> La adición de un procesador mediante `ApplicationInsights.config` o `TelemetryConfiguration.Active` no es válida para las aplicaciones de ASP.NET Core o si usa el SDK de Microsoft.ApplicationInsights.WorkerService.

Para las aplicaciones escritas mediante [ASP.NET Core](asp-net-core.md#adding-telemetry-processors) o [WorkerService](worker-service.md#adding-telemetry-processors), la adición de un nuevo procesador de telemetría se realiza mediante el método de extensión `AddApplicationInsightsTelemetryProcessor` en `IServiceCollection`, como se muestra. Se llama a este método en el método `ConfigureServices` de su clase `Startup.cs`.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // ...
        services.AddApplicationInsightsTelemetry();
        services.AddApplicationInsightsTelemetryProcessor<SuccessfulDependencyFilter>();

        // If you have more processors:
        services.AddApplicationInsightsTelemetryProcessor<AnotherProcessor>();
    }
```

### <a name="example-filters"></a>Filtros de ejemplo

#### <a name="synthetic-requests"></a>Solicitudes sintéticas

Filtrar las pruebas web y robots. Aunque el Explorador de métricas proporciona la opción para filtrar y dejar fuera los orígenes sintéticos, esta opción reduce el tráfico y el tamaño de ingesta filtrándolos en el SDK.

```csharp
public void Process(ITelemetry item)
{
  if (!string.IsNullOrEmpty(item.Context.Operation.SyntheticSource)) {return;}

  // Send everything else:
  this.Next.Process(item);
}
```

#### <a name="failed-authentication"></a>Error de autenticación

Filtrar solicitudes con una respuesta "401".

```csharp
public void Process(ITelemetry item)
{
    var request = item as RequestTelemetry;

    if (request != null &&
    request.ResponseCode.Equals("401", StringComparison.OrdinalIgnoreCase))
    {
        // To filter out an item, return without calling the next processor.
        return;
    }

    // Send everything else
    this.Next.Process(item);
}
```

#### <a name="filter-out-fast-remote-dependency-calls"></a>Filtrar las llamadas de dependencia rápida y remota

Si quiere diagnosticar solo las llamadas que son lentas, descarte las rápidas.

> [!NOTE]
> Este filtrado sesgará las estadísticas que se ven en el portal.
>
>

```csharp
public void Process(ITelemetry item)
{
    var request = item as DependencyTelemetry;

    if (request != null && request.Duration.TotalMilliseconds < 100)
    {
        return;
    }
    this.Next.Process(item);
}
```

#### <a name="diagnose-dependency-issues"></a>Diagnóstico de problemas de dependencia

[este blog](https://azure.microsoft.com/blog/implement-an-application-insights-telemetry-processor/) se describe un proyecto para diagnosticar problemas de dependencia enviando automáticamente pings regulares a las dependencias.

<a name="add-properties"></a>

### <a name="javascript-web-applications"></a>Aplicaciones web de JavaScript

**Filtro con ITelemetryInitializer**

1. Cree una función de devolución de llamada del inicializador de telemetría. La función de devolución de llamada toma `ITelemetryItem` como parámetro, que es el evento que se está procesando. La devolución de `false` desde esta devolución de llamada da lugar a que se filtre el elemento de telemetría.

   ```JS
   var filteringFunction = (envelope) => {
     if (envelope.data.someField === 'tobefilteredout') {
         return false;
     }
  
     return true;
   };
   ```

2. Agregue la devolución de llamada del inicializador de telemetría:

   ```JS
   appInsights.addTelemetryInitializer(filteringFunction);
   ```

## <a name="addmodify-properties-itelemetryinitializer"></a>Agregar o modificar propiedades: ITelemetryInitializer

Use los inicializadores de telemetría para enriquecer la telemetría con información adicional o para invalidar las propiedades de telemetría establecidas por los módulos de telemetría estándar.

Por ejemplo, Application Insights para un paquete web recopila telemetría acerca de las solicitudes HTTP. De forma predeterminada, marca como errónea cualquier solicitud con un código de respuesta >=400. Pero si quiere tratar 400 como correcto, puede proporcionar un inicializador de telemetría que establezca la propiedad de éxito.

Si se proporciona un inicializador de telemetría, se llama cada vez que se llama a cualquiera de los métodos Track*(). Esto incluye los métodos `Track()` llamados por los módulos de telemetría estándar. Por convención, estos módulos no establecen ninguna propiedad que ya haya sido establecida por un inicializador. Se llama a los inicializadores de telemetría antes de llamar a los procesadores de telemetría. Por tanto, cualquier enriquecimiento realizado por los inicializadores es visible para los procesadores.

**Defina su inicializador**

*C#*

```csharp
using System;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MvcWebRole.Telemetry
{
  /*
   * Custom TelemetryInitializer that overrides the default SDK
   * behavior of treating response codes >= 400 as failed requests
   *
   */
  public class MyTelemetryInitializer : ITelemetryInitializer
  {
    public void Initialize(ITelemetry telemetry)
    {
        var requestTelemetry = telemetry as RequestTelemetry;
        // Is this a TrackRequest() ?
        if (requestTelemetry == null) return;
        int code;
        bool parsed = Int32.TryParse(requestTelemetry.ResponseCode, out code);
        if (!parsed) return;
        if (code >= 400 && code < 500)
        {
            // If we set the Success property, the SDK won't change it:
            requestTelemetry.Success = true;

            // Allow us to filter these requests in the portal:
            requestTelemetry.Properties["Overridden400s"] = "true";
        }
        // else leave the SDK to set the Success property
    }
  }
}
```

**Aplicaciones ASP.NET: Carga del inicializador**

En ApplicationInsights.config:

```xml
<ApplicationInsights>
  <TelemetryInitializers>
    <!-- Fully qualified type name, assembly name: -->
    <Add Type="MvcWebRole.Telemetry.MyTelemetryInitializer, MvcWebRole"/>
    ...
  </TelemetryInitializers>
</ApplicationInsights>
```

Alternativamente, se pueden crear instancias del inicializador en el código, por ejemplo en Global.aspx.cs:

```csharp
protected void Application_Start()
{
    // ...
    TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
}
```

Obtenga más información de [este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole).

**Aplicaciones Core/ Servicio de trabajo de ASP.NET: Carga del inicializador**

> [!NOTE]
> La adición de un inicializador mediante `ApplicationInsights.config` o `TelemetryConfiguration.Active` no es válida para las aplicaciones de ASP.NET Core o si usa el SDK de Microsoft.ApplicationInsights.WorkerService.

Para las aplicaciones escritas mediante [ASP.NET Core](asp-net-core.md#adding-telemetryinitializers) o [WorkerService](worker-service.md#adding-telemetryinitializers), la adición de un nuevo inicializador de telemetría se realiza agregándolo al contenedor de inserción de dependencias, como se muestra. Esto se hace en el método `Startup.ConfigureServices`.

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyTelemetryInitializer>();
}
```
### <a name="javascript-telemetry-initializers"></a>Inicializadores de telemetría de JavaScript
*JavaScript*

Inserte un inicializador de telemetría inmediatamente después del código de inicialización que obtuvo del portal:

```JS
<script type="text/javascript">
    // ... initialization code
    ...({
        instrumentationKey: "your instrumentation key"
    });
    window.appInsights = appInsights;


    // Adding telemetry initializer.
    // This is called whenever a new telemetry item
    // is created.

    appInsights.queue.push(function () {
        appInsights.context.addTelemetryInitializer(function (envelope) {
            var telemetryItem = envelope.data.baseData;

            // To check the telemetry items type - for example PageView:
            if (envelope.name == Microsoft.ApplicationInsights.Telemetry.PageView.envelopeType) {
                // this statement removes url from all page view documents
                telemetryItem.url = "URL CENSORED";
            }

            // To set custom properties:
            telemetryItem.properties = telemetryItem.properties || {};
            telemetryItem.properties["globalProperty"] = "boo";

            // To set custom metrics:
            telemetryItem.measurements = telemetryItem.measurements || {};
            telemetryItem.measurements["globalMetric"] = 100;
        });
    });

    // End of inserted code.

    appInsights.trackPageView();
</script>
```

Para obtener un resumen de las propiedades no personalizadas disponibles en el elemento de telemetría, consulte [Modelo de exportación de datos de Application Insights](./export-data-model.md).

Puede agregar tantos inicializadores como desee. Se llaman en el orden en que se agregan.

### <a name="opencensus-python-telemetry-processors"></a>Procesadores de telemetría de Python para OpenCensus

Los procesadores de telemetría de Python para OpenCensus son simplemente funciones de devolución de llamada utilizadas para procesar la telemetría antes de que se exporten. La función de devolución de llamada debe aceptar un tipo de datos de [sobre](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/protocol.py#L86) como parámetro. Para filtrar la telemetría para que no se exporte, asegúrese de que la función de devolución de llamada devuelva `False`. Puede ver el esquema de los tipos de datos de Azure Monitor en los sobres [en GitHub](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/protocol.py).

> [!NOTE]
> `cloud_RoleName` se puede modificar cambiando el atributo `ai.cloud.role` del campo `tags`.

```python
def callback_function(envelope):
    envelope.tags['ai.cloud.role'] = 'new_role_name'
```

```python
# Example for log exporter
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)

# Callback function to append '_hello' to each log message telemetry
def callback_function(envelope):
    envelope.data.baseData.message += '_hello'
    return True

handler = AzureLogHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>')
handler.add_telemetry_processor(callback_function)
logger.addHandler(handler)
logger.warning('Hello, World!')
```
```python
# Example for trace exporter
import requests

from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace import config_integration
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['requests'])

# Callback function to add os_type: linux to span properties
def callback_function(envelope):
    envelope.data.baseData.properties['os_type'] = 'linux'
    return True

exporter = AzureExporter(
    connection_string='InstrumentationKey=<your-instrumentation-key-here>'
)
exporter.add_telemetry_processor(callback_function)
tracer = Tracer(exporter=exporter, sampler=ProbabilitySampler(1.0))
with tracer.span(name='parent'):
response = requests.get(url='https://www.wikipedia.org/wiki/Rabbit')
```

```python
# Example for metrics exporter
import time

from opencensus.ext.azure import metrics_exporter
from opencensus.stats import aggregation as aggregation_module
from opencensus.stats import measure as measure_module
from opencensus.stats import stats as stats_module
from opencensus.stats import view as view_module
from opencensus.tags import tag_map as tag_map_module

stats = stats_module.stats
view_manager = stats.view_manager
stats_recorder = stats.stats_recorder

CARROTS_MEASURE = measure_module.MeasureInt("carrots",
                                            "number of carrots",
                                            "carrots")
CARROTS_VIEW = view_module.View("carrots_view",
                                "number of carrots",
                                [],
                                CARROTS_MEASURE,
                                aggregation_module.CountAggregation())

# Callback function to only export the metric if value is greater than 0
def callback_function(envelope):
    return envelope.data.baseData.metrics[0].value > 0

def main():
    # Enable metrics
    # Set the interval in seconds in which you want to send metrics
    exporter = metrics_exporter.new_metrics_exporter(connection_string='InstrumentationKey=<your-instrumentation-key-here>')
    exporter.add_telemetry_processor(callback_function)
    view_manager.register_exporter(exporter)

    view_manager.register_view(CARROTS_VIEW)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    mmap.measure_int_put(CARROTS_MEASURE, 1000)
    mmap.record(tmap)
    # Default export interval is every 15.0s
    # Your application should run for at least this amount
    # of time so the exporter will meet this interval
    # Sleep can fulfill this
    time.sleep(60)

    print("Done recording metrics")

if __name__ == "__main__":
    main()
```
Puede agregar tantos procesadores como quiera. Se llaman en el orden en que se agregan. En el caso de que un procesador produzca una excepción, esta no afectará a los procesadores siguientes.

### <a name="example-telemetryinitializers"></a>TelemetryInitializers de ejemplo

#### <a name="add-a-custom-property"></a>Adición de una propiedad personalizada

El siguiente inicializador de ejemplo agrega una propiedad personalizada a cada una de las telemetrías de las que se ha realizado un seguimiento.

```csharp
public void Initialize(ITelemetry item)
{
  var itemProperties = item as ISupportProperties;
  if(itemProperties != null && !itemProperties.Properties.ContainsKey("customProp"))
    {
        itemProperties.Properties["customProp"] = "customValue";
    }
}
```

#### <a name="add-a-cloud-role-name"></a>Adición de un nombre de rol en la nube

El siguiente inicializador de ejemplo establece el nombre de rol en la nube a cada una de las telemetrías de las que se ha realizado un seguimiento.

```csharp
public void Initialize(ITelemetry telemetry)
{
    if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
    {
        telemetry.Context.Cloud.RoleName = "MyCloudRoleName";
    }
}
```

#### <a name="add-information-from-httpcontext"></a>Adición de información desde HttpContext

El inicializador de ejemplo siguiente lee los datos de [`HttpContext`](/aspnet/core/fundamentals/http-context?view=aspnetcore-3.1) y lo anexa a una instancia de `RequestTelemetry`. `IHttpContextAccessor` se proporciona automáticamente a través de la inserción de dependencias del constructor.

```csharp
public class HttpContextRequestTelemetryInitializer : ITelemetryInitializer
{
    private readonly IHttpContextAccessor httpContextAccessor;

    public HttpContextRequestTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    {
        this.httpContextAccessor =
            httpContextAccessor ??
            throw new ArgumentNullException(nameof(httpContextAccessor));
    }

    public void Initialize(ITelemetry telemetry)
    {
        var requestTelemetry = telemetry as RequestTelemetry;
        if (requestTelemetry == null) return;

        var claims = this.httpContextAccessor.HttpContext.User.Claims;
        Claim oidClaim = claims.FirstOrDefault(claim => claim.Type == "oid");
        requestTelemetry.Properties.Add("UserOid", oidClaim?.Value);
    }
}
```

## <a name="itelemetryprocessor-and-itelemetryinitializer"></a>ITelemetryProcessor e ITelemetryInitializer

¿Cuál es la diferencia entre los procesadores de telemetría y los inicializadores de telemetría?

* Hay algunas superposiciones en lo que puede hacer con ellos. Ambos se pueden usar para agregar o modificar las propiedades de la telemetría, aunque se recomienda usar inicializadores con esa finalidad.
* Los inicializadores de telemetría siempre se ejecutan antes que los procesadores de telemetría.
* Se puede llamar a los inicializadores de telemetría más de una vez. Por convención, no establecen ninguna propiedad que ya se haya establecido.
* Los procesadores de telemetría permiten reemplazar o descartar por completo un elemento de telemetría.
* Se garantiza que se llamará a todos los inicializadores de telemetría registrados para cada elemento de telemetría. En el caso de los procesadores de telemetría, el SDK garantiza la llamada al primer procesador de telemetría. Los procesadores de telemetría anteriores deciden si se llama o no al resto de los procesadores.
* Use los inicializadores de la telemetría para enriquecerla con propiedades adicionales o invalidar una existente. Use un procesador de telemetría para filtrar la telemetría.

## <a name="troubleshoot-applicationinsightsconfig"></a>Solución de problemas de ApplicationInsights.config

* Compruebe que el nombre del tipo completo y el nombre del ensamblado sean correctos.
* Compruebe que el archivo applicationinsights.config esté en el directorio de salida y que contenga todos los cambios recientes.

## <a name="reference-docs"></a>Documentos de referencia

* [Información general acerca de la API](./api-custom-events-metrics.md)
* [Referencia de ASP.NET](/previous-versions/azure/dn817570(v=azure.100))

## <a name="sdk-code"></a>Código del SDK

* [SDK básico de ASP.NET](https://github.com/Microsoft/ApplicationInsights-aspnetcore)
* [SDK de ASP.NET](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [SDK de JavaScript](https://github.com/Microsoft/ApplicationInsights-JS)

## <a name="next-steps"></a><a name="next"></a>Pasos siguientes
* [Búsqueda de eventos y registros](./diagnostic-search.md)
* [Muestreo](./sampling.md)
* [Solución de problemas](../faq.md)

