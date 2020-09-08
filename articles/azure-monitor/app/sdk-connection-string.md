---
title: Cadenas de conexión en Azure Application Insights | Microsoft Docs
description: Uso de cadenas de conexión
ms.topic: conceptual
author: timothymothra
ms.author: tilee
ms.date: 01/17/2020
ms.custom: devx-track-javascript, devx-track-csharp
ms.reviewer: mbullwin
ms.openlocfilehash: 944257c93e00dca77507f26db15f7bf45fbb387e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936425"
---
# <a name="connection-strings"></a>Cadenas de conexión

## <a name="overview"></a>Información general

Las cadenas de conexión proporcionan a los usuarios de Application Insights una única opción de configuración, lo que elimina la necesidad de varias configuraciones de proxy. Resultan de gran utilidad para los servidores web de la intranet y los entornos de nube híbrida o soberana que buscan enviar datos al servicio de supervisión.

Los pares clave-valor proporcionan a los usuarios una manera fácil de definir una combinación de sufijo y prefijo para cada servicio o producto de Application Insights (AI).

> [!IMPORTANT]
> No se recomienda establecer la cadena de conexión y la clave de instrumentación. En caso de que un usuario establezca ambas, tendrá prioridad la que se haya establecido en último lugar. 


## <a name="scenario-overview"></a>Información general de escenario 

Escenarios de cliente en los que se visualiza dónde tiene esto el mayor efecto:

- Excepciones de firewall o redirecciones de proxy 

    En los casos en los que es necesario supervisar el servidor web de la intranet, nuestra solución anterior solicitaba a los clientes que agregaran puntos de conexión de servicio individuales a la configuración. Para más información, consulte [esta página](../faq.md#can-i-monitor-an-intranet-web-server). 
    Las cadenas de conexión ofrecen una mejor alternativa, ya que reducen esta labor a una única configuración. Una corrección de sufijos y prefijos simple permite el rellenado y la redirección automáticos de todos los puntos de conexión a los servicios adecuados. 

- Entornos de nube soberana o híbrida

    Los usuarios pueden enviar datos a una [región de Azure Government](../../azure-government/compare-azure-government-global-azure.md#application-insights) definida.
    Las cadenas de conexión permiten definir la configuración de punto de conexión de los servidores de la intranet o la configuración de la nube híbrida. 

## <a name="getting-started"></a>Introducción

### <a name="finding-my-connection-string"></a>¿Dónde está la cadena de conexión?

La cadena de conexión se muestra en la hoja de información general del recurso de Application Insights.

![cadena de conexión en la hoja de información general](media/overview-dashboard/overview-connection-string.png)

### <a name="schema"></a>Schema

#### <a name="max-length"></a>Longitud máxima

La conexión tiene una longitud máxima admitida de 4096 caracteres.

#### <a name="key-value-pairs"></a>Pares de clave-valor

La cadena de conexión consta de una lista de valores de configuración representados como pares de clave-valor separados por punto y coma: `key1=value1;key2=value2;key3=value3`

#### <a name="syntax"></a>Sintaxis

- `InstrumentationKey` (por ejemplo: 00000000-0000-0000-0000-000000000000) La cadena de conexión es un campo **obligatorio**.
- `Authorization` (por ejemplo: ikey) (Este valor es opcional porque actualmente no se admite la autorización mediante ikey.)
- `EndpointSuffix` (por ejemplo: applicationinsights.azure.cn) Al establecer el sufijo del punto de conexión, se indicará al SDK a qué nube de Azure se conectará. El SDK ensamblará el resto del punto de conexión para los servicios individuales.
- Puntos de conexión explícitos.
  Cualquier servicio se puede invalidar explícitamente en la cadena de conexión.
   - `IngestionEndpoint` (por ejemplo: `https://dc.applicationinsights.azure.com`)
   - `LiveEndpoint` (por ejemplo: `https://live.applicationinsights.azure.com`)
   - `ProfilerEndpoint` (por ejemplo: `https://profiler.applicationinsights.azure.com`)
   - `SnapshotEndpoint` (por ejemplo: `https://snapshot.applicationinsights.azure.com`)

#### <a name="endpoint-schema"></a>Esquema del punto de conexión

`<prefix>.<suffix>`
- Prefijo: define un servicio. 
- Sufijo: define el nombre de dominio común.

##### <a name="valid-suffixes"></a>Sufijos válidos

Esta es una lista de sufijos válidos. 
- applicationinsights.azure.cn
- applicationinsights.us


Consulte también: https://docs.microsoft.com/azure/azure-monitor/app/custom-endpoints#regions-that-require-endpoint-modification


##### <a name="valid-prefixes"></a>Prefijos válidos

- [Ingesta de telemetría](./app-insights-overview.md): `dc`
- [Live Metrics](./live-stream.md): `live`
- [Profiler](./profiler-overview.md): `profiler`
- [Instantánea](./snapshot-debugger.md): `snapshot`



## <a name="connection-string-examples"></a>Ejemplos de cadena de conexión


### <a name="minimal-valid-connection-string"></a>Cadena de conexión válida mínima

`InstrumentationKey=00000000-0000-0000-0000-000000000000;`

En este ejemplo, solo se ha establecido la clave de instrumentación.

- El esquema de autorización tiene como valor predeterminado "ikey". 
- Clave de instrumentación: 00000000-0000-0000-0000-000000000000
- Los identificadores URI de servicio regional se basan en los [valores predeterminados del SDK](https://github.com/microsoft/ApplicationInsights-dotnet/blob/e50d569cebf485e72e98f4a08a0bc0e30cdf42bc/BASE/src/Microsoft.ApplicationInsights/Extensibility/Implementation/Endpoints/Constants.cs#L6) y se conectarán a Azure público global:
   - Ingesta: `https://dc.services.visualstudio.com/`
   - Live Metrics: `https://rt.services.visualstudio.com/`
   - Profiler: `https://agent.azureserviceprofiler.net/`
   - Depurador: `https://agent.azureserviceprofiler.net/`



### <a name="connection-string-with-endpoint-suffix"></a>Cadena de conexión con sufijo de punto de conexión

`InstrumentationKey=00000000-0000-0000-0000-000000000000;EndpointSuffix=ai.contoso.com;`

En este ejemplo, esta cadena de conexión especifica el sufijo de punto de conexión, y el SDK construirá los puntos de conexión de servicio.

- El esquema de autorización tiene como valor predeterminado "ikey". 
- Clave de instrumentación: 00000000-0000-0000-0000-000000000000
- Los identificadores URI de servicio regional se basan en el sufijo de punto de conexión proporcionado: 
   - Ingesta: `https://dc.ai.contoso.com`
   - Live Metrics: `https://live.ai.contoso.com`
   - Profiler: `https://profiler.ai.contoso.com`
   - Depurador: `https://snapshot.ai.contoso.com`  



### <a name="connection-string-with-explicit-endpoint-overrides"></a>Cadena de conexión con invalidaciones explícitas del punto de conexión 

`InstrumentationKey=00000000-0000-0000-0000-000000000000;IngestionEndpoint=https://custom.com:111/;LiveEndpoint=https://custom.com:222/;ProfilerEndpoint=https://custom.com:333/;SnapshotEndpoint=https://custom.com:444/;`

En este ejemplo, esta cadena de conexión especifica invalidaciones explícitas para cada servicio. El SDK usará los puntos de conexión exactos proporcionados sin modificaciones.

- El esquema de autorización tiene como valor predeterminado "ikey". 
- Clave de instrumentación: 00000000-0000-0000-0000-000000000000
- Los identificadores URI de servicio regional se basan en los valores de invalidación explícitos: 
   - Ingesta: `https://custom.com:111/`
   - Live Metrics: `https://custom.com:222/`
   - Profiler: `https://custom.com:333/`
   - Depurador: `https://custom.com:444/`  


## <a name="how-to-set-a-connection-string"></a>Cómo establecer una cadena de conexión

Se admiten cadenas de conexión en las siguientes versiones del SDK:
- .NET y .NET Core v2.12.0
- Java v2.5.1 y Java 3.0
- JavaScript v2.3.0
- NodeJS v1.5.0
- Python v1.0.0

Una cadena de conexión se puede establecer por el código, la variable de entorno o el archivo de configuración.



### <a name="environment-variable"></a>Variable de entorno

- Cadena de conexión: `APPLICATIONINSIGHTS_CONNECTION_STRING`

# <a name="netnetcore"></a>[.NET/.NetCore](#tab/net)

TelemetryConfiguration.ConnectionString: https://github.com/microsoft/ApplicationInsights-dotnet/blob/add45ceed35a817dc7202ec07d3df1672d1f610d/BASE/src/Microsoft.ApplicationInsights/Extensibility/TelemetryConfiguration.cs#L271-L274

.NET establecido explícitamente:
```csharp
var configuration = new TelemetryConfiguration
{
    ConnectionString = "InstrumentationKey=00000000-0000-0000-0000-000000000000;"
};
```

Archivo de configuración .NET:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
    <ConnectionString>InstrumentationKey=00000000-0000-0000-0000-000000000000</ConnectionString>
</ApplicationInsights>
```

NetCore establecido explícitamente:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    var options = new ApplicationInsightsServiceOptions { ConnectionString = "InstrumentationKey=00000000-0000-0000-0000-000000000000;" };
    services.AddApplicationInsightsTelemetry(options: options);
}
```

NetCore config.json: 

```json
{
  "ApplicationInsights": {
    "ConnectionString" : "InstrumentationKey=00000000-0000-0000-0000-000000000000;"
    }
  }
```


# <a name="java"></a>[Java](#tab/java)


Java (v2.5.x) establecido explícitamente:
```java
TelemetryConfiguration.getActive().setConnectionString("InstrumentationKey=00000000-0000-0000-0000-000000000000");
```

ApplicationInsights.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
    <ConnectionString>InstrumentationKey=00000000-0000-0000-0000-000000000000;</ConnectionString>
</ApplicationInsights>
```

# <a name="javascript"></a>[JavaScript](#tab/js)

Importante: JavaScript no admite el uso de variables de entorno.

Uso del fragmento de código:

```javascript
<script type="text/javascript">
var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(e){function n(e){t[e]=function(){var n=arguments;t.queue.push(function(){t[e].apply(t,n)})}}var t={config:e};t.initialize=!0;var i=document,a=window;setTimeout(function(){var n=i.createElement("script");n.src=e.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",i.getElementsByTagName("script")[0].parentNode.appendChild(n)});try{t.cookie=i.cookie}catch(e){}t.queue=[],t.version=2;for(var r=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];r.length;)n("track"+r.pop());n("startTrackPage"),n("stopTrackPage");var s="Track"+r[0];if(n("start"+s),n("stop"+s),n("addTelemetryInitializer"),n("setAuthenticatedUserContext"),n("clearAuthenticatedUserContext"),n("flush"),t.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4},!(!0===e.disableExceptionTracking||e.extensionConfig&&e.extensionConfig.ApplicationInsightsAnalytics&&!0===e.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){n("_"+(r="onerror"));var o=a[r];a[r]=function(e,n,i,a,s){var c=o&&o(e,n,i,a,s);return!0!==c&&t["_"+r]({message:e,url:n,lineNumber:i,columnNumber:a,error:s}),c},e.autoExceptionInstrumented=!0}return t}(
{
  connectionString:"InstrumentationKey=00000000-0000-0000-0000-000000000000;"
}
);window[aiName]=aisdk,aisdk.queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
</script>
```


Instalación manual:
```javascript
import { ApplicationInsights } from '@microsoft/applicationinsights-web'

const appInsights = new ApplicationInsights({ config: {
  connectionString: 'InstrumentationKey=00000000-0000-0000-0000-000000000000;'
  /* ...Other Configuration Options... */
} });
appInsights.loadAppInsights();
appInsights.trackPageView();
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("InstrumentationKey=00000000-0000-0000-0000-000000000000;");
appInsights.start();
```

# <a name="python"></a>[Python](#tab/python)

Se recomienda que los usuarios establezcan la variable de entorno.

Para establecer explícitamente la cadena de conexión:

```python
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

tracer = Tracer(exporter=AzureExporter(connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000'), sampler=ProbabilitySampler(1.0))
```


## <a name="next-steps"></a>Pasos siguientes

Comience en el tiempo de ejecución con:

* [Aplicaciones hospedadas en IIS en máquina virtual de Azure y conjunto de escalado de máquinas virtuales de Azure](./azure-vm-vmss-apps.md)
* [Servidor IIS](./monitor-performance-live-website-now.md)
* [Azure Web Apps](./azure-web-apps.md)

Comience en el tiempo de desarrollo con:

* [ASP.NET](./asp-net.md)
* [ASP.NET Core](./asp-net-core.md)
* [Java](./java-get-started.md)
* [Node.js](./nodejs.md)
* [Python](./opencensus-python.md)

