---
title: API de Application Insights para eventos y métricas personalizados | Microsoft Docs
description: Inserte unas cuantas líneas de código en su aplicación de dispositivo o de escritorio, página o servicio web, para realizar el seguimiento del uso y diagnosticar problemas.
ms.topic: conceptual
ms.date: 05/11/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 430ec96006ed8f564ea5bbd0a28beca858ebe1ab
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87366879"
---
# <a name="application-insights-api-for-custom-events-and-metrics"></a>API de Application Insights para eventos y métricas personalizados

Inserte unas cuantas líneas de código en la aplicación para averiguar qué uso hacen de ella los usuarios o para ayudarle a diagnosticar problemas. Puede enviar datos de telemetría desde aplicaciones de escritorio y de dispositivo y desde clientes y servidores web. Use la API de telemetría principal de [Azure Application Insights](./app-insights-overview.md) para enviar métricas y eventos personalizados, así como sus propias versiones de telemetría estándar. Esta API es la misma que usan los recopiladores de datos estándar de Application Insights.

## <a name="api-summary"></a>API summary

La API central es uniforme en todas las plataformas, excepto por algunas pequeñas variaciones como `GetMetric` (solo .NET).

| Método | Se usa para |
| --- | --- |
| [`TrackPageView`](#page-views) |Páginas, pantallas, hojas o formularios. |
| [`TrackEvent`](#trackevent) |Acciones de usuario y otros eventos. Se usa para realizar el seguimiento del comportamiento de los usuarios o para supervisar el rendimiento. |
| [`GetMetric`](#getmetric) |Cero y métricas multidimensionales, agregación configurada de forma centralizada, solo en C#. |
| [`TrackMetric`](#trackmetric) |Las medidas de rendimiento, como las longitudes de cola, no están relacionadas con eventos específicos. |
| [`TrackException`](#trackexception) |Excepciones de registro para diagnóstico. Permite realizar el seguimiento del lugar donde se producen en relación con otros eventos y examinar los seguimientos de pila. |
| [`TrackRequest`](#trackrequest) |Registro de la frecuencia y duración de las solicitudes de servidor para el análisis de rendimiento. |
| [`TrackTrace`](#tracktrace) |Mensajes del registro de diagnóstico de recursos. También puede capturar registros de terceros. |
| [`TrackDependency`](#trackdependency) |Registro de la duración y frecuencia de las llamadas a componentes externos de los que depende la aplicación. |

Puede [adjuntar propiedades y métricas](#properties) a la mayoría de estas llamadas de telemetría.

## <a name="before-you-start"></a><a name="prep"></a>Antes de comenzar

Si aún no tiene una referencia en el SDK de Application Insights:

* Agregue el SDK de Application Insights a su proyecto:

  * [Proyecto de ASP.NET](./asp-net.md)
  * [Proyecto de ASP.NET Core](./asp-net-core.md)
  * [Proyecto de Java](./java-get-started.md)
  * [Proyecto de Node.js](./nodejs.md)
  * [JavaScript en cada página web](./javascript.md) 
* En el código de servidor web o de dispositivo, incluya:

    *C#:* `using Microsoft.ApplicationInsights;`

    *Visual Basic:* `Imports Microsoft.ApplicationInsights`

    *Java:* `import com.microsoft.applicationinsights.TelemetryClient;`

    *Node.js:* `var applicationInsights = require("applicationinsights");`

## <a name="get-a-telemetryclient-instance"></a>Obtención de una instancia de TelemetryClient

Obtenga una instancia de `TelemetryClient` (excepto en JavaScript en páginas web):

En el caso de las aplicaciones [ASP.NET Core](asp-net-core.md#how-can-i-track-telemetry-thats-not-automatically-collected) y aplicaciones que [no son HTTP o de trabajo para .NET/.NET Core](worker-service.md#how-can-i-track-telemetry-thats-not-automatically-collected), se recomienda obtener una instancia de `TelemetryClient` del contenedor de inserción de dependencias, tal como se explica en la documentación correspondiente.

Si usa AzureFunctions v2 o superior o Azure WebJobs v3 o superior, siga este documento: https://docs.microsoft.com/azure/azure-functions/functions-monitoring#version-2x-and-higher

*C#*

```csharp
private TelemetryClient telemetry = new TelemetryClient();
```
Los usuarios que reciban mensajes que indiquen que el método está obsoleto pueden visitar [microsoft/ApplicationInsights-dotnet#1152](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1152) para obtener más detalles.

*Visual Basic*

```vb
Private Dim telemetry As New TelemetryClient
```

*Java*

```java
private TelemetryClient telemetry = new TelemetryClient();
``` 

*Node.js*

```javascript
var telemetry = applicationInsights.defaultClient;
```

TelemetryClient es seguro para subprocesos.

Para proyectos ASP.NET y Java, las solicitudes HTTP entrantes se capturan automáticamente. Puede que quiera crear instancias adicionales de TelemetryClient para otro módulo de la aplicación. Por ejemplo, puede tener una instancia de TelemetryClient en la clase de middleware para notificar eventos de la lógica de negocios. Puede establecer propiedades como UserId y DeviceId para identificar la máquina. Esta información se adjunta a todos los eventos enviados por la instancia.

*C#*

```csharp
TelemetryClient.Context.User.Id = "...";
TelemetryClient.Context.Device.Id = "...";
```

*Java*

```java
telemetry.getContext().getUser().setId("...");
telemetry.getContext().getDevice().setId("...");
```

En proyectos de Node.js, puede usar `new applicationInsights.TelemetryClient(instrumentationKey?)` para crear una nueva instancia, pero esto solo se recomienda para escenarios que requieren configuración aislada del singleton `defaultClient`.

## <a name="trackevent"></a>TrackEvent

En Application Insights, un *evento personalizado* es un punto de datos que se puede mostrar en el [Explorador de métricas](../platform/metrics-charts.md) como recuento agregado, y como repeticiones individuales en [Búsqueda de diagnóstico](./diagnostic-search.md). (No está relacionado con MVC ni con "eventos" de otro marco).

Inserte llamadas a `TrackEvent` en el código para contabilizar diversos eventos: la frecuencia con la que los usuarios eligen una determinada característica, con la que logran unos determinados objetivos o con la que cometen determinados tipos de errores.

Por ejemplo, en una aplicación de juego, envíe un evento cada vez que un usuario gane el juego:

*JavaScript*

```javascript
appInsights.trackEvent({name:"WinGame"});
```

*C#*

```csharp
telemetry.TrackEvent("WinGame");
```

*Visual Basic*

```vb
telemetry.TrackEvent("WinGame")
```

*Java*

```java
telemetry.trackEvent("WinGame");
```

*Node.js*

```javascript
telemetry.trackEvent({name: "WinGame"});
```

### <a name="custom-events-in-analytics"></a>Eventos personalizados en Analytics

La telemetría está disponible en la tabla `customEvents` de [Analytics de Application Insights](../log-query/log-query-overview.md). Cada fila representa una llamada a `trackEvent(..)` en la aplicación.

Si el [muestreo](./sampling.md) está en uso, en la propiedad itemCount se muestra un valor mayor que 1. Por ejemplo, itemCount==10 significa que de cada 10 llamadas a trackEvent(), el proceso de muestreo solo transmite una. Para obtener un recuento correcto de eventos personalizados, debería usar código como `customEvents | summarize sum(itemCount)`.

## <a name="getmetric"></a>GetMetric

Para obtener información sobre cómo usar eficazmente la llamada a GetMetric() para capturar métricas previamente agregadas de forma local para aplicaciones .NET y .NET Core, visite la documentación de [GetMetric](./get-metric.md).

## <a name="trackmetric"></a>TrackMetric

> [!NOTE]
> Microsoft.ApplicationInsights.TelemetryClient.TrackMetric no es el método preferido para el envío de métricas. Las métricas deben agregarse previamente siempre durante un tiempo antes de enviarse. Use una de las sobrecargas GetMetric(..) para obtener un objeto de métrica para acceder a funcionalidades de agregación previa del SDK. Si va a implementar su propia lógica de agregación previa, puede usar el método TrackMetric() para enviar los agregados resultantes. Si su aplicación requiere el envío de un elemento de telemetría independiente en cada ocasión sin agregación a lo largo del tiempo, es probable que en su caso haya que usar la telemetría de eventos; consulte TelemetryClient.TrackEvent (Microsoft.ApplicationInsights.DataContracts.EventTelemetry).

Application Insights puede crear gráficos de métricas que no estén conectadas a eventos concretos. Por ejemplo, puede supervisar una longitud de cola a intervalos regulares. En el caso de las métricas, las mediciones individuales tienen menos interés que las variaciones y tendencias; por tanto, los gráficos estadísticos resultan útiles.

Para enviar las métricas a Application Insights, puede usar la API `TrackMetric(..)`. Hay dos formas de enviar una métrica:

* Valor único. Cada vez que realiza una medición en la aplicación, envía el valor correspondiente a Application Insights. Por ejemplo, suponga que cuenta con una métrica que describe el número de elementos de un contenedor. Durante un período determinado, primero coloca tres elementos en el contenedor y seguidamente retira dos. En consecuencia, llamaría a `TrackMetric` dos veces: la primera, para pasar el valor `3` y a continuación el valor `-2`. Application Insights almacena ambos valores en su nombre.

* Agregación. Al trabajar con métricas, las mediciones individuales pocas veces resultan de interés. En su lugar, lo importante son los resúmenes de acontecimientos durante un período determinado. Los resúmenes de este tipo se denominan _agregaciones_. En el ejemplo anterior, la suma de las métricas agregadas del período correspondiente es de `1` y el recuento de los valores de las métricas es de `2`. Al usar el método de agregación, solo invocará `TrackMetric` una vez por período y enviará los valores agregados. Este es el método que se recomienda usar, ya que puede reducir significativamente los costos y la sobrecarga de rendimiento gracias a que envía menos puntos de datos a Application Insights, a pesar de seguir recopilado toda la información pertinente.

### <a name="examples"></a>Ejemplos

#### <a name="single-values"></a>Valores únicos

Para enviar un único valor de métrica:

*JavaScript*

 ```javascript
appInsights.trackMetric("queueLength", 42.0);
 ```

*C#*

```csharp
var sample = new MetricTelemetry();
sample.Name = "metric name";
sample.Value = 42.3;
telemetryClient.TrackMetric(sample);
```

*Java*

```java
telemetry.trackMetric("queueLength", 42.0);
```

*Node.js*

 ```javascript
telemetry.trackMetric({name: "queueLength", value: 42.0});
 ```

### <a name="custom-metrics-in-analytics"></a>Métricas personalizadas en Analytics

La telemetría está disponible en la tabla `customMetrics` de [Analytics de Application Insights](../log-query/log-query-overview.md). Cada fila representa una llamada a `trackMetric(..)` en la aplicación.

* `valueSum`: es la suma de las medidas. Para obtener el valor medio, divídalo por `valueCount`.
* `valueCount`: el número de medidas que se agregaron en esta llamada a `trackMetric(..)`.

## <a name="page-views"></a>Vistas de página

En una aplicación de dispositivo o de página web, se envían datos de telemetría de la vista de página de forma predeterminada cuando se carga cada pantalla o página. Sin embargo, puede cambiar esta frecuencia para que se realice el seguimiento de las vistas de página en momentos diferentes o adicionales. Por ejemplo, en una aplicación que muestra pestañas u hojas, quizás quiera realizar el seguimiento de una página siempre que el usuario abra una nueva hoja.

Los datos de usuario y de sesión se envían como propiedades junto con las vistas de página, por lo que los gráficos de usuario y de sesión se activan cuando hay datos de telemetría de vistas de página.

### <a name="custom-page-views"></a>Vistas de página personalizadas

*JavaScript*

```javascript
appInsights.trackPageView("tab1");
```

*C#*

```csharp
telemetry.TrackPageView("GameReviewPage");
```

*Visual Basic*

```vb
telemetry.TrackPageView("GameReviewPage")
```

*Java*

```java
telemetry.trackPageView("GameReviewPage");
```

Si tiene varias fichas dentro de páginas HTML diferentes, puede especificar también la dirección URL:

```javascript
appInsights.trackPageView("tab1", "http://fabrikam.com/page1.htm");
```

### <a name="timing-page-views"></a>Cronometrar las vistas de página

De forma predeterminada, los tiempos notificados como **Tiempo de carga de la vista de página** se miden desde que el explorador envía la solicitud hasta que se llama al evento de carga de página del explorador.

En su lugar, puede:

* Establecer una duración explícita en la llamada [trackPageView](https://github.com/microsoft/ApplicationInsights-JS/blob/17ef50442f73fd02a758fbd74134933d92607ecf/legacy/API.md#trackpageview): `appInsights.trackPageView("tab1", null, null, null, durationInMilliseconds);`.
* Usar las llamadas para cronometrar la vista de página `startTrackPage` y `stopTrackPage`.

*JavaScript*

```javascript
// To start timing a page:
appInsights.startTrackPage("Page1");

...

// To stop timing and log the page:
appInsights.stopTrackPage("Page1", url, properties, measurements);
```

El nombre que use como primer parámetro asocia las llamadas inicial y final. El valor predeterminado es el nombre de la página actual.

Las duraciones de carga de página resultantes que se muestran en el Explorador de métricas se derivan del intervalo que media entre las llamadas inicial y final. Depende del usuario qué intervalo se cronometra realmente.

### <a name="page-telemetry-in-analytics"></a>Telemetría de páginas en Analytics

En [Analytics](../log-query/log-query-overview.md) hay dos tablas en las que se muestran datos de operaciones de explorador:

* La tabla `pageViews` contiene datos sobre la URL y el título de la página.
* La tabla `browserTimings` contiene datos sobre el rendimiento del cliente, como el tiempo que se tarda en procesar los datos entrantes.

Para averiguar cuánto tarda el explorador en procesar diferentes páginas:

```kusto
browserTimings
| summarize avg(networkDuration), avg(processingDuration), avg(totalDuration) by name
```

Para descubrir la popularidad de distintos exploradores:

```kusto
pageViews
| summarize count() by client_Browser
```

Para asociar las vistas de página a las llamadas AJAX, realice una combinación con dependencias:

```kusto
pageViews
| join (dependencies) on operation_Id 
```

## <a name="trackrequest"></a>TrackRequest

El SDK del servidor usa TrackRequest para registrar las solicitudes de HTTP.

También puede llamarlo usted mismo si quiere simular solicitudes en un contexto en el que no se está ejecutando el módulo de servicio web.

Sin embargo, lo que se recomienda para enviar telemetría de solicitudes es que la solicitud actúe como <a href="#operation-context">contexto de operación</a>.

## <a name="operation-context"></a>Contexto de operación

Puede correlacionar los elementos de telemetría juntos mediante su asociación con el contexto de la operación. El módulo de seguimiento de solicitud estándar realiza esta operación para excepciones y otros eventos enviados al procesar una solicitud HTTP. En [Búsqueda](./diagnostic-search.md) y [Análisis](../log-query/log-query-overview.md), puede encontrar fácilmente cualquier evento asociado a la solicitud mediante su identificador de operación.

Para más información sobre la correlación, vea [Correlación de telemetría en Application Insights](./correlation.md).

Al realizar el seguimiento de la telemetría manualmente, la forma más fácil de garantizar la correlación de telemetría es mediante el uso de este patrón:

*C#*

```csharp
// Establish an operation context and associated telemetry item:
using (var operation = telemetryClient.StartOperation<RequestTelemetry>("operationName"))
{
    // Telemetry sent in here will use the same operation ID.
    ...
    telemetryClient.TrackTrace(...); // or other Track* calls
    ...

    // Set properties of containing telemetry item--for example:
    operation.Telemetry.ResponseCode = "200";

    // Optional: explicitly send telemetry item:
    telemetryClient.StopOperation(operation);

} // When operation is disposed, telemetry item is sent.
```

Junto con la configuración de un contexto de la operación, `StartOperation` crea un elemento de telemetría del tipo que especifique. Envía el elemento de telemetría al eliminar la operación, o si llama explícitamente a `StopOperation`. Si usa `RequestTelemetry` como tipo de telemetría, su duración se establece en el intervalo cronometrado entre el inicio y la detención.

Los elementos de telemetría notificados dentro de un ámbito de operación se convierten en "elementos secundarios" de dicha operación. Los contextos de operación pueden estar anidados.

En la Búsqueda, el contexto de la operación se utiliza para crear la lista de **Elementos relacionados**:

![Elementos relacionados](./media/api-custom-events-metrics/21.png)

Consulte [Seguimiento de las operaciones personalizadas con el SDK de .NET para Application Insights](./custom-operations-tracking.md) para más información sobre el seguimiento de las operaciones personalizadas.

### <a name="requests-in-analytics"></a>Solicitudes en Analytics

En [Analytics de Application Insights](../log-query/log-query-overview.md), las solicitudes aparecen en la tabla `requests`.

Si el [muestreo](./sampling.md) está en uso, en la propiedad de itemCount se mostrará un valor superior a 1. Por ejemplo, itemCount==10 significa que de cada 10 llamadas a trackRequest(), el proceso de muestreo solo transmite una. Para obtener un recuento correcto de solicitudes y la duración media segmentada por nombres de solicitudes, use código como el siguiente:

```kusto
requests
| summarize count = sum(itemCount), avgduration = avg(duration) by name
```

## <a name="trackexception"></a>TrackException

Enviar excepciones a Application Insights:

* Para [contarlas](../platform/metrics-charts.md), como indicación de la frecuencia de un problema.
* Para [examinar los casos individuales](./diagnostic-search.md).

Los informes incluyen los seguimientos de la pila.

*C#*

```csharp
try
{
    ...
}
catch (Exception ex)
{
    telemetry.TrackException(ex);
}
```

*Java*

```java
try {
    ...
} catch (Exception ex) {
    telemetry.trackException(ex);
}
```

*JavaScript*

```javascript
try
{
    ...
}
catch (ex)
{
    appInsights.trackException(ex);
}
```

*Node.js*

```javascript
try
{
    ...
}
catch (ex)
{
    telemetry.trackException({exception: ex});
}
```

Los SDK capturan muchas excepciones automáticamente, por lo que no siempre es necesario llamar explícitamente a TrackException.

* ASP.NET: [escritura de código para detectar excepciones](./asp-net-exceptions.md).
* Java EE: [las excepciones se detectan automáticamente](./java-get-started.md#exceptions-and-request-failures).
* JavaScript: las excepciones se detectan automáticamente. Si desea deshabilitar la colección automática, agregue una línea al fragmento de código que se inserta en las páginas web:

```javascript
({
    instrumentationKey: "your key",
    disableExceptionTracking: true
})
```

### <a name="exceptions-in-analytics"></a>Excepciones en Analytics

En [Analytics de Application Insights](../log-query/log-query-overview.md), las excepciones aparecen en la tabla `exceptions`.

Si el [muestreo](./sampling.md) está en uso, en la propiedad `itemCount` se muestra un valor mayor que 1. Por ejemplo, itemCount==10 significa que de cada 10 llamadas a trackException(), el proceso de muestreo solo transmite una. Para obtener un recuento correcto de excepciones segmentadas por tipo de excepción, use código como el siguiente:

```kusto
exceptions
| summarize sum(itemCount) by type
```

La mayor parte de la información importante sobre pilas ya se extrajo en variables independientes, pero puede desmontar la estructura `details` para obtener más. Puesto que se trata de una estructura dinámica, debería convertir el resultado al tipo que espere. Por ejemplo:

```kusto
exceptions
| extend method2 = tostring(details[0].parsedStack[1].method)
```

Para asociar las excepciones a sus respectivas solicitudes, use una combinación:

```kusto
exceptions
| join (requests) on operation_Id
```

## <a name="tracktrace"></a>TrackTrace

Use TrackTrace para ayudar a diagnosticar problemas mediante el envío de una ''ruta de exploración'' a Application Insights. Puede enviar fragmentos de datos de diagnóstico e inspeccionarlos en [Búsqueda de diagnóstico](./diagnostic-search.md).

Los [adaptadores de registro](./asp-net-trace-logs.md) de .NET usan esta API para enviar registros de terceros al portal.

En Java, para [registradores estándar como Log4J o Logback](./java-trace-logs.md), utilice los appenders de Log4j o Logback de Application Insights para enviar registros de terceros al portal.

*C#*

```csharp
telemetry.TrackTrace(message, SeverityLevel.Warning, properties);
```

*Java*

```java
telemetry.trackTrace(message, SeverityLevel.Warning, properties);
```

*Node.js*

```javascript
telemetry.trackTrace({
    message: message,
    severity: applicationInsights.Contracts.SeverityLevel.Warning,
    properties: properties
});
```

*JavaScript del lado cliente/explorador*

```javascript
trackTrace(message: string, properties?: {[string]:string}, severityLevel?: SeverityLevel)
```

Registre un evento de diagnóstico, como la entrada o la salida de un método.

 Parámetro | Descripción
---|---
`message` | Datos de diagnóstico. Puede ser mucho más largo que un nombre.
`properties` | Asignación de cadena a cadena: Datos adicionales que se usan para [filtrar excepciones](#properties) en el portal. El valor predeterminado es vacío.
`severityLevel` | Valores admitidos: [SeverityLevel.ts](https://github.com/microsoft/ApplicationInsights-JS/blob/17ef50442f73fd02a758fbd74134933d92607ecf/shared/AppInsightsCommon/src/Interfaces/Contracts/Generated/SeverityLevel.ts)

Puede buscar en el contenido del mensaje, pero (a diferencia de los valores de propiedad) no puede filtrar por él.

El límite de tamaño en `message` es mucho mayor que el límite en propiedades.
Una ventaja de TrackTrace es que puede colocar datos relativamente largos en el mensaje. Por ejemplo, aquí puede codificar datos POST.  

Además, puede agregar un nivel de gravedad al mensaje. Y, al igual que con otra telemetría, puede agregar valores de propiedad para ayudar a filtrar o buscar distintos conjuntos de seguimientos. Por ejemplo:

*C#*

```csharp
var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
telemetry.TrackTrace("Slow database response",
                SeverityLevel.Warning,
                new Dictionary<string,string> { {"database", db.ID} });
```

*Java*

```java
Map<String, Integer> properties = new HashMap<>();
properties.put("Database", db.ID);
telemetry.trackTrace("Slow Database response", SeverityLevel.Warning, properties);
```

En [Búsqueda](./diagnostic-search.md), puede filtrar fácilmente todos los mensajes de un determinado nivel de gravedad relativos a una determinada base de datos.

### <a name="traces-in-analytics"></a>Seguimientos en Analytics

En [Analytics de Application Insights](../log-query/log-query-overview.md), las llamadas a TrackTrace aparecen en la tabla `traces`.

Si el [muestreo](./sampling.md) está en uso, en la propiedad itemCount se muestra un valor mayor que 1. Por ejemplo, itemCount==10 significa que de cada 10 llamadas a `trackTrace()`, el proceso de muestreo solo transmite una. Para obtener un recuento correcto de llamadas de seguimiento, debería codificar por tanto como `traces | summarize sum(itemCount)`.

## <a name="trackdependency"></a>TrackDependency

Utilice la llamada de TrackDependency para realizar un seguimiento de los tiempos de respuesta y las tasas de éxito de las llamadas a un fragmento de código externo. Los resultados se muestran en los gráficos de dependencia del portal. Es necesario agregar el fragmento de código siguiente siempre que se realice una llamada de dependencia.

*C#*

```csharp
var success = false;
var startTime = DateTime.UtcNow;
var timer = System.Diagnostics.Stopwatch.StartNew();
try
{
    success = dependency.Call();
}
catch(Exception ex) 
{
    success = false;
    telemetry.TrackException(ex);
    throw new Exception("Operation went wrong", ex);
}
finally
{
    timer.Stop();
    telemetry.TrackDependency("DependencyType", "myDependency", "myCall", startTime, timer.Elapsed, success);
}
```

*Java*

```java
boolean success = false;
Instant startTime = Instant.now();
try {
    success = dependency.call();
}
finally {
    Instant endTime = Instant.now();
    Duration delta = Duration.between(startTime, endTime);
    RemoteDependencyTelemetry dependencyTelemetry = new RemoteDependencyTelemetry("My Dependency", "myCall", delta, success);
    RemoteDependencyTelemetry.setTimeStamp(startTime);
    RemoteDependencyTelemetry.trackDependency(dependencyTelemetry);
}
```

*Node.js*

```javascript
var success = false;
var startTime = new Date().getTime();
try
{
    success = dependency.Call();
}
finally
{
    var elapsed = new Date() - startTime;
    telemetry.trackDependency({
        dependencyTypeName: "myDependency",
        name: "myCall",
        duration: elapsed,
        success: success
    });
}
```

Recuerde que los SDK del servidor incluyen un [módulo de dependencia](./asp-net-dependencies.md) que detecta y realiza automáticamente el seguimiento de ciertas llamadas de dependencia; por ejemplo, a bases de datos y API de REST. Debe instalar un agente en el servidor para que el módulo funcione. 

En Java, es posible realizar el seguimiento de ciertas llamadas de dependencia automáticamente mediante el [agente de Java](./java-agent.md).

Utilizará esta llamada si desea hacer un seguimiento de las llamadas no captadas por el seguimiento automatizado, o bien si no desea instalar el agente.

Para desactivar el módulo de seguimiento de dependencias estándar en C#, edite [ApplicationInsights.config](./configuration-with-applicationinsights-config.md) y elimine la referencia a `DependencyCollector.DependencyTrackingTelemetryModule`. En Java, no instale al agente de Java si no quiere recopilar dependencias estándar automáticamente.

### <a name="dependencies-in-analytics"></a>Dependencias en Analytics

En [Analytics de Application Insights](../log-query/log-query-overview.md), las llamadas de trackDependency aparecen en la tabla `dependencies`.

Si el [muestreo](./sampling.md) está en uso, en la propiedad itemCount se muestra un valor mayor que 1. Por ejemplo, itemCount==10 significa que de cada 10 llamadas a trackDependency(), el proceso de muestreo solo transmite una. Para obtener un recuento correcto de dependencias segmentadas por componente de destino, use código como el siguiente:

```kusto
dependencies
| summarize sum(itemCount) by target
```

Para asociar las dependencias a sus respectivas solicitudes, use una combinación:

```kusto
dependencies
| join (requests) on operation_Id
```

## <a name="flushing-data"></a>Datos de vaciado

Normalmente, el SDK envía datos a intervalos fijos (normalmente 30 segundos) o cuando el búfer está lleno (normalmente 500 elementos). Sin embargo, en algunos casos puede que desee vaciar el búfer: por ejemplo, si usa el SDK en una aplicación que se apaga.

*C#*

 ```csharp
telemetry.Flush();
// Allow some time for flushing before shutdown.
System.Threading.Thread.Sleep(5000);
```

*Java*

```java
telemetry.flush();
//Allow some time for flushing before shutting down
Thread.sleep(5000);
```

*Node.js*

```javascript
telemetry.flush();
```

La función es asincrónica para el [canal de telemetría del servidor](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/).

Lo ideal es que se utilice el método flush() en la actividad de cierre de la aplicación.

## <a name="authenticated-users"></a>Usuarios autenticados

En una aplicación web, los usuarios se [identifican por cookies](./usage-segmentation.md#the-users-sessions-and-events-segmentation-tool) (de manera predeterminada). Se puede contar al usuario más de una vez si accede a la aplicación desde un equipo o explorador diferente, o si elimina las cookies.

Si los usuarios inician sesión en su aplicación, puede obtener un recuento más preciso estableciendo el identificador del usuario autenticado en el código del explorador:

*JavaScript*

```javascript
// Called when my app has identified the user.
function Authenticated(signInId) {
    var validatedId = signInId.replace(/[,;=| ]+/g, "_");
    appInsights.setAuthenticatedUserContext(validatedId);
    ...
}
```

En una aplicación MVC web de ASP.NET, por ejemplo:

*Razor*

```cshtml
@if (Request.IsAuthenticated)
{
    <script>
        appInsights.setAuthenticatedUserContext("@User.Identity.Name
            .Replace("\\", "\\\\")"
            .replace(/[,;=| ]+/g, "_"));
    </script>
}
```

No es necesario usar el nombre de inicio de sesión real del usuario. Solo tiene que ser un identificador único para ese usuario. No debe incluir espacios ni ninguno de los caracteres `,;=|`.

El identificador de usuario también se establece en una cookie de sesión y se envía al servidor. Si está instalado el SDK del servidor, el identificador de usuario autenticado se envía como parte de las propiedades de contexto tanto de la telemetría del cliente como del servidor. A continuación, puede filtrar y buscar en ella.

Si su aplicación agrupa a los usuarios en cuentas, también puede pasar un identificador de la cuenta (con las mismas restricciones de caracteres).

```javascript
appInsights.setAuthenticatedUserContext(validatedId, accountId);
```

En el [Explorador de métricas](../platform/metrics-charts.md), puede crear un gráfico que cuente los **Usuarios autenticados** y las **Cuentas de usuario**.

También puede [Buscar](./diagnostic-search.md) puntos de datos de cliente con cuentas y nombres de usuario específicos.

## <a name="filtering-searching-and-segmenting-your-data-by-using-properties"></a><a name="properties"></a>Filtrado, búsqueda y segmentación de los datos mediante el uso de propiedades

Puede asociar propiedades y medidas a los eventos (y también a las métricas, vistas de página, excepciones y otros datos de telemetría).

*propiedades* son valores de cadena que se pueden usar para filtrar los datos de telemetría en los informes de uso. Por ejemplo, si su aplicación proporciona varios juegos, puede adjuntar el nombre del juego a cada evento para así poder ver cuáles son los juegos más populares.

Hay un límite de aproximadamente 8192 en la longitud de cadena. (Si quiere enviar fragmentos grandes de datos, use el parámetro de mensaje de TrackTrace).

*métricas* son valores numéricos que se pueden presentar de forma gráfica. Por ejemplo, puede que quiera ver si hay un aumento gradual en las puntuaciones que alcanzan sus jugadores. Los gráficos se pueden segmentar por las propiedades enviadas con el evento, así que puede separar o apilar los gráficos para diferentes juegos.

Para que valores de métricas se muestren correctamente, deben ser mayores o iguales que 0.

Hay algunos [límites en el número de propiedades, valores de propiedad y métricas](#limits) que puede usar.

*JavaScript*

```javascript
appInsights.trackEvent
    ("WinGame",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );

appInsights.trackPageView
    ("page name", "http://fabrikam.com/pageurl.html",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );
```

*C#*

```csharp
// Set up some properties and metrics:
var properties = new Dictionary <string, string>
    {{"game", currentGame.Name}, {"difficulty", currentGame.Difficulty}};
var metrics = new Dictionary <string, double>
    {{"Score", currentGame.Score}, {"Opponents", currentGame.OpponentCount}};

// Send the event:
telemetry.TrackEvent("WinGame", properties, metrics);
```

*Node.js*

```javascript
// Set up some properties and metrics:
var properties = {"game": currentGame.Name, "difficulty": currentGame.Difficulty};
var metrics = {"Score": currentGame.Score, "Opponents": currentGame.OpponentCount};

// Send the event:
telemetry.trackEvent({name: "WinGame", properties: properties, measurements: metrics});
```

*Visual Basic*

```vb
' Set up some properties:
Dim properties = New Dictionary (Of String, String)
properties.Add("game", currentGame.Name)
properties.Add("difficulty", currentGame.Difficulty)

Dim metrics = New Dictionary (Of String, Double)
metrics.Add("Score", currentGame.Score)
metrics.Add("Opponents", currentGame.OpponentCount)

' Send the event:
telemetry.TrackEvent("WinGame", properties, metrics)
```

*Java*

```java
Map<String, String> properties = new HashMap<String, String>();
properties.put("game", currentGame.getName());
properties.put("difficulty", currentGame.getDifficulty());

Map<String, Double> metrics = new HashMap<String, Double>();
metrics.put("Score", currentGame.getScore());
metrics.put("Opponents", currentGame.getOpponentCount());

telemetry.trackEvent("WinGame", properties, metrics);
```

> [!NOTE]
> Tenga cuidado de no registrar información de identificación personal en las propiedades.
>
>

### <a name="alternative-way-to-set-properties-and-metrics"></a>Método alternativo para establecer propiedades y métricas

Si le resulta más cómodo, puede recopilar los parámetros de un evento en un objeto independiente:

```csharp
var event = new EventTelemetry();

event.Name = "WinGame";
event.Metrics["processingTime"] = stopwatch.Elapsed.TotalMilliseconds;
event.Properties["game"] = currentGame.Name;
event.Properties["difficulty"] = currentGame.Difficulty;
event.Metrics["Score"] = currentGame.Score;
event.Metrics["Opponents"] = currentGame.Opponents.Length;

telemetry.TrackEvent(event);
```

> [!WARNING]
> No vuelva a usar la misma instancia de elemento de telemetría (`event` en este ejemplo) para llamar a Track*() varias veces. Esto puede hacer que se envíe la telemetría con una configuración incorrecta.
>
>

### <a name="custom-measurements-and-properties-in-analytics"></a>Mediciones y propiedades personalizadas en Analytics

En [Analytics](../log-query/log-query-overview.md), las métricas y propiedades personalizadas aparecen en los atributos `customMeasurements` y `customDimensions` de cada registro de telemetría.

Por ejemplo, si agregó una propiedad llamada "game" a la telemetría de solicitudes, esta consulta cuenta el número de apariciones de diferentes valores de "game" y muestra la media de la métrica personalizada "score":

```kusto
requests
| summarize sum(itemCount), avg(todouble(customMeasurements.score)) by tostring(customDimensions.game)
```

Tenga en lo siguiente:

* Al extraer un valor de los elementos de JSON customDimensions o customMeasurements, es de tipo dinámico, por lo que debe convertirlo a `tostring` o `todouble`.
* Para tener en cuenta la posibilidad de [muestreo](./sampling.md), debería usar `sum(itemCount)`, no `count()`.

## <a name="timing-events"></a><a name="timed"></a> Eventos de temporización

Seguro que en ocasiones le gustaría representar el tiempo que se tarda en realizar alguna acción. Por ejemplo, puede que quiera saber cuánto tiempo tardan los usuarios en considerar las opciones de un juego. Puede usar el parámetro de medida para ello.

*C#*

```csharp
var stopwatch = System.Diagnostics.Stopwatch.StartNew();

// ... perform the timed action ...

stopwatch.Stop();

var metrics = new Dictionary <string, double>
    {{"processingTime", stopwatch.Elapsed.TotalMilliseconds}};

// Set up some properties:
var properties = new Dictionary <string, string>
    {{"signalSource", currentSignalSource.Name}};

// Send the event:
telemetry.TrackEvent("SignalProcessed", properties, metrics);
```

*Java*

```java
long startTime = System.currentTimeMillis();

// Perform timed action

long endTime = System.currentTimeMillis();
Map<String, Double> metrics = new HashMap<>();
metrics.put("ProcessingTime", (double)endTime-startTime);

// Setup some properties
Map<String, String> properties = new HashMap<>();
properties.put("signalSource", currentSignalSource.getName());

// Send the event
telemetry.trackEvent("SignalProcessed", properties, metrics);
```

## <a name="default-properties-for-custom-telemetry"></a><a name="defaults"></a>Propiedades predeterminadas para la telemetría personalizada

Si quiere establecer valores de propiedad predeterminados para algunos de los eventos personalizados que escriba, puede hacerlo en una instancia de TelemetryClient. Se adjuntarán a cada elemento de telemetría enviado desde ese cliente.

*C#*

```csharp
using Microsoft.ApplicationInsights.DataContracts;

var gameTelemetry = new TelemetryClient();
gameTelemetry.Context.GlobalProperties["Game"] = currentGame.Name;
// Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame");
```

*Visual Basic*

```vb
Dim gameTelemetry = New TelemetryClient()
gameTelemetry.Context.GlobalProperties("Game") = currentGame.Name
' Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame")
```

*Java*

```java
import com.microsoft.applicationinsights.TelemetryClient;
import com.microsoft.applicationinsights.TelemetryContext;
...


TelemetryClient gameTelemetry = new TelemetryClient();
TelemetryContext context = gameTelemetry.getContext();
context.getProperties().put("Game", currentGame.Name);

gameTelemetry.TrackEvent("WinGame");
```

*Node.js*

```javascript
var gameTelemetry = new applicationInsights.TelemetryClient();
gameTelemetry.commonProperties["Game"] = currentGame.Name;

gameTelemetry.TrackEvent({name: "WinGame"});
```

Las llamadas de telemetría individuales pueden invalidar los valores predeterminados en los diccionarios de propiedad.

*Para los clientes web de JavaScript*, use los inicializadores de telemetría de JavaScript.

*Para agregar propiedades a toda la telemetría*, incluidos los datos de los módulos de recopilación estándar, [implemente `ITelemetryInitializer`](./api-filtering-sampling.md#add-properties).

## <a name="sampling-filtering-and-processing-telemetry"></a>Muestreo, filtrado y procesamiento de telemetría

Puede escribir código para procesar la telemetría antes de que se envíe desde el SDK. El procesamiento incluye los datos enviados desde los módulos de telemetría estándar, como la recopilación de solicitudes HTTP y de dependencias.

[Agregue propiedades](./api-filtering-sampling.md#add-properties) a la telemetría mediante la implementación de `ITelemetryInitializer`. Por ejemplo, puede agregar números de versión o valores calculados a partir de otras propiedades.

El [filtrado](./api-filtering-sampling.md#filtering) puede modificar o descartar la telemetría antes de que se envíe desde el SDK, mediante la implementación de `ITelemetryProcessor`. Puede controlar qué se envía y qué se descarta, pero debe tener en cuenta el efecto en las métricas. Según la forma en que se descarten los elementos, podría perder la capacidad de navegar entre elementos relacionados.

El [muestreo](./api-filtering-sampling.md) es una solución empaquetada para reducir el volumen de datos enviado desde la aplicación al portal. Lo hace sin que las métricas mostradas resulten afectadas. Y sin repercutir tampoco sobre capacidad para diagnosticar problemas navegando entre elementos relacionados, como excepciones, solicitudes y vistas de página.

[Más información](./api-filtering-sampling.md).

## <a name="disabling-telemetry"></a>Deshabilitación de la telemetría

Para *iniciar y detener dinámicamente* la recopilación y la transmisión de telemetría:

*C#*

```csharp
using  Microsoft.ApplicationInsights.Extensibility;

TelemetryConfiguration.Active.DisableTelemetry = true;
```

*Java*

```java
telemetry.getConfiguration().setTrackingDisabled(true);
```

Para *deshabilitar los recopiladores estándar seleccionados* (por ejemplo, contadores de rendimiento, solicitudes HTTP o dependencias), elimine o convierta en comentarios las líneas correspondientes en [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). Puede hacer esto, por ejemplo, si quiere enviar sus propios datos de TrackRequest.

*Node.js*

```javascript
telemetry.config.disableAppInsights = true;
```

Para *deshabilitar los recopiladores estándar seleccionados* (por ejemplo, los contadores de rendimiento, las solicitudes HTTP o las dependencias) en el tiempo de inicialización, encadene métodos de configuración a su código de inicialización de SDK:

```javascript
applicationInsights.setup()
    .setAutoCollectRequests(false)
    .setAutoCollectPerformance(false)
    .setAutoCollectExceptions(false)
    .setAutoCollectDependencies(false)
    .setAutoCollectConsole(false)
    .start();
```

Para deshabilitar estos recopiladores después de la inicialización, utilice el objeto de configuración: `applicationInsights.Configuration.setAutoCollectRequests(false)`

## <a name="developer-mode"></a><a name="debug"></a>Modo de programador

Durante la depuración, resulta útil enviar los datos de telemetría por la canalización para así poder ver los resultados inmediatamente. También puede recibir mensajes adicionales que le ayuden a realizar el seguimiento de los posibles problemas con la telemetría. Desactívelo en producción, ya que puede ralentizar la aplicación.

*C#*

```csharp
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = true;
```

*Visual Basic*

```vb
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = True
```

*Node.js*

Para Node.js, puede habilitar el modo de desarrollador habilitando el registro interno a través de `setInternalLogging` y estableciendo `maxBatchSize` en 0, lo que hace que la telemetría se envíe tan pronto como se recopile.

```js
applicationInsights.setup("ikey")
  .setInternalLogging(true, true)
  .start()
applicationInsights.defaultClient.config.maxBatchSize = 0;
```

## <a name="setting-the-instrumentation-key-for-selected-custom-telemetry"></a><a name="ikey"></a> Establecimiento de la clave de instrumentación para datos de telemetría personalizados seleccionados

*C#*

```csharp
var telemetry = new TelemetryClient();
telemetry.InstrumentationKey = "---my key---";
// ...
```

## <a name="dynamic-instrumentation-key"></a><a name="dynamic-ikey"></a> Copia de la clave de instrumentación

Para evitar la mezcla de telemetría de entornos de desarrollo, pruebas y producción, puede [crear recursos separados de Application Insights](./create-new-resource.md) y cambiar sus claves en función del entorno.

En lugar de obtener la clave de instrumentación del archivo de configuración, puede establecerla en el código. Establezca la clave en un método de inicialización, como global.aspx.cs en un servicio de ASP.NET:

*C#*

```csharp
protected void Application_Start()
{
    Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey =
        // - for example -
        WebConfigurationManager.Settings["ikey"];
    ...
}
```

*JavaScript*

```javascript
appInsights.config.instrumentationKey = myKey;
```

En una página web, podría configurarla a partir del estado del servidor web, en lugar de codificarla literalmente en el script. Por ejemplo, en una página web generada en una aplicación ASP.NET:

*JavaScript en Razor*

```cshtml
<script type="text/javascript">
// Standard Application Insights webpage script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
    // Generate from server property:
    @Microsoft.ApplicationInsights.Extensibility.
        TelemetryConfiguration.Active.InstrumentationKey;
}) // ...
```

```java
    String instrumentationKey = "00000000-0000-0000-0000-000000000000";

    if (instrumentationKey != null)
    {
        TelemetryConfiguration.getActive().setInstrumentationKey(instrumentationKey);
    }
```

## <a name="telemetrycontext"></a>TelemetryContext

TelemetryClient tiene una propiedad de Context, que contiene valores que se envían junto con todos los datos de telemetría. Normalmente, se establecen mediante los módulos de telemetría estándar, pero también los puede establecer usted mismo. Por ejemplo:

```csharp
telemetry.Context.Operation.Name = "MyOperationName";
```

Si establece cualquiera de estos valores manualmente, considere la posibilidad de quitar la línea pertinente de [ApplicationInsights.config](./configuration-with-applicationinsights-config.md), de modo que no se confundan sus valores con los valores estándar.

* **Component**: la aplicación y su versión.
* **Device**: datos sobre el dispositivo donde se ejecuta la aplicación. (En aplicaciones web, se trata del servidor o el dispositivo de cliente desde el que se envía la telemetría).
* **InstrumentationKey**: el recurso de Application Insights en Azure donde aparece la telemetría. Normalmente, se selecciona de ApplicationInsights.config.
* **Ubicación**: la ubicación geográfica del dispositivo.
* **Operation**: en las aplicaciones web, es la solicitud HTTP actual. En otros tipos de aplicaciones, puede establecer este valor para agrupar los eventos juntos.
  * **Identificador**: valor generado que correlaciona distintos eventos, de modo que cuando usted inspeccione cualquier evento en Búsqueda de diagnóstico, puede encontrar elementos relacionados.
  * **Name**: un identificador, generalmente la dirección URL de la solicitud HTTP.
  * **SyntheticSource**: si no es un valor nulo ni está vacío, esta cadena indica que el origen de la solicitud se ha identificado como un robot o una prueba web. De forma predeterminada, se excluye de cálculos en el Explorador de métricas.
* **Propiedades**: propiedades que se envían con todos los datos de telemetría. Se pueden invalidar en llamadas de seguimiento* individuales.
* **Sesión**: sesión del usuario. El identificador se establece en un valor generado, que cambia cuando el usuario lleva un tiempo sin estar activo.
* **Usuario**: información del usuario.

## <a name="limits"></a>límites

[!INCLUDE [application-insights-limits](../../../includes/application-insights-limits.md)]

Para evitar llegar al límite de velocidad de datos, utilice el [muestreo](./sampling.md).

Para determinar cuánto tiempo se conservan los datos, consulte el artículo sobre [retención de datos y privacidad](./data-retention-privacy.md).

## <a name="reference-docs"></a>Documentos de referencia

* [Referencia de ASP.NET](/dotnet/api/overview/azure/insights?view=azure-dotnet)
* [Referencia de Java](/java/api/overview/azure/appinsights?view=azure-java-stable/)
* [Referencia de JavaScript](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md)

## <a name="sdk-code"></a>Código del SDK

* [SDK básico de ASP.NET](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [ASP.NET](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [Paquetes de Windows Server](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [SDK de Java](https://github.com/Microsoft/ApplicationInsights-Java)
* [SDK de Node.js](https://github.com/Microsoft/ApplicationInsights-Node.js)
* [SDK de JavaScript](https://github.com/Microsoft/ApplicationInsights-JS)

## <a name="questions"></a>Preguntas

* *¿Qué excepciones pueden iniciar las llamadas de seguimiento_()?*

    Ninguno. No es necesario agruparlas en cláusulas try-catch. Si el SDK encuentra problemas, registrará los mensajes en la salida de la consola de depuración, y, si los mensajes pasan, en la Búsqueda de diagnóstico.
* *¿Hay una API de REST para obtener datos desde el portal?*

    Sí, la [API de acceso a datos](https://dev.applicationinsights.io/). Otras maneras de extraer datos son [exportar desde Analytics a Power BI](./export-power-bi.md) y la [exportación continua](./export-telemetry.md).

## <a name="next-steps"></a><a name="next"></a>Pasos siguientes

* [Búsqueda de eventos y registros](./diagnostic-search.md)
* [Solución de problemas](../faq.md)

