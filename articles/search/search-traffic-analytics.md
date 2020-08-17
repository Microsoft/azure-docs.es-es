---
title: Telemetría para análisis del tráfico de búsqueda
titleSuffix: Azure Cognitive Search
description: Habilite el análisis del tráfico de búsqueda para Azure Cognitive Search, recopile datos de telemetría y eventos iniciados por el usuario mediante Application Insights y, después, analice los resultados en un informe de Power BI.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/18/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 6ab32a2ccb4c7eb79309798c2b53d326723ad6ea
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420080"
---
# <a name="collect-telemetry-data-for-search-traffic-analytics"></a>Recopilación de datos de telemetría para análisis del tráfico de búsqueda

El análisis del tráfico de búsqueda es un patrón para recopilar datos de telemetría sobre las interacciones de los usuarios con su aplicación de Azure Cognitive Search, como eventos de clic iniciados por el usuario y entradas de teclado. Con esta información, puede determinar la eficacia de la solución de búsqueda, incluidos los términos de búsqueda más populares, la tasa de clics y las entradas de consulta que producen cero resultados.

Este patrón depende de [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) (una característica de [Azure monitor](https://docs.microsoft.com/azure/azure-monitor/)) para recopilar datos de usuario. Requiere que agregue la instrumentación a su código de cliente, tal como se describe en este artículo. Por último, necesitará una herramienta de informes para analizar los datos. Se recomienda Power BI, pero puede utilizar el panel de la aplicación o cualquier herramienta que se conecte a Application Insights.

> [!NOTE]
> El patrón que se describe en este artículo es para escenarios avanzados y datos de la secuencia de clics generados por el código que agrega al cliente. Por el contrario, los registros de servicio son fáciles de configurar, proporcionan una gama de métricas y se pueden realizar en el portal sin necesidad de código. Se recomienda habilitar el registro para todos los escenarios. Para obtener más información, consulte [Recopilación y análisis de datos de registro](search-monitor-logs.md).

## <a name="identify-relevant-search-data"></a>Identificación de los datos de búsqueda pertinentes

Para contar con métricas útiles para el análisis del tráfico de búsqueda, es necesario registrar algunas de las señales de los usuarios de la aplicación de búsqueda. Estas señales indican el contenido en el que los usuarios están interesados y que consideran relevantes. Para el análisis del tráfico de búsqueda, se incluyen:

+ Eventos de búsqueda generados por el usuario: Solo las consultas de búsqueda iniciadas por un usuario son interesantes. Las solicitudes de búsqueda usadas para rellenar las facetas, el contenido adicional o cualquier información interna, no son importantes y sesgan y desvían los resultados.

+ Eventos de clic generados por el usuario: En una página de resultados de la búsqueda, un evento de clic generalmente significa que un documento es un resultado pertinente para una consulta de búsqueda específica.

Al vincular los eventos de búsqueda y clic con un id. de correlación, comprenderá mejor el rendimiento de la funcionalidad de búsqueda de su aplicación.

## <a name="add-search-traffic-analytics"></a>Incorporación de los análisis del tráfico de búsqueda

En la página del [portal](https://portal.azure.com) para el servicio Azure Cognitive Search, la página de análisis del tráfico de búsqueda contiene una hoja de referencia para seguir este modelo de telemetría. Desde esta página, puede seleccionar o crear un recurso de Application Insights, obtener la clave de instrumentación, copiar fragmentos de código que se puedan adaptar a la solución y descargar un informe de Power BI que se base en el esquema que se refleja en el patrón.

![Página Análisis del tráfico de búsqueda en el portal](media/search-traffic-analytics/azuresearch-trafficanalytics.png "Página Análisis del tráfico de búsqueda en el portal")

## <a name="1---set-up-application-insights"></a>1\. Configuración de Application Insights

Seleccione un recurso existente de Application Insights o [cree uno](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource) si aún no lo tiene. Si usa la página Análisis de tráfico de búsqueda, puede copiar la clave de instrumentación que necesita la aplicación para conectarse a Application Insights.

Una vez que tenga un recurso de Application Insights, puede seguir las [instrucciones para los lenguajes y las plataformas admitidos](https://docs.microsoft.com/azure/azure-monitor/app/platforms) para registrar la aplicación. El registro consiste simplemente en agregar la clave de instrumentación de Application Insights al código, con lo que se configura la asociación. Puede encontrar la clave en el portal o en la página Análisis de tráfico de búsqueda al seleccionar un recurso existente.

En los pasos siguientes se refleja un acceso directo que funciona en algunos tipos de proyecto de Visual Studio. Crea un recurso y registra la aplicación en unos pocos clics.

1. Para el desarrollo en Visual Studio y ASP.NET, abra la solución y seleccione **Proyecto** > **Agregar telemetría de Application Insights**.

1. Haga clic en **Introducción**.

1. Registre su aplicación proporcionando un cuenta de Microsoft, una suscripción a Azure y un recurso de Application Insights (el valor predeterminado es un nuevo recurso). Haga clic en **Registrar**.

En este punto, la aplicación está configurada para la supervisión de aplicaciones, lo que significa que se realiza un seguimiento de todas las cargas de páginas con las métricas predeterminadas. Para obtener más información sobre los pasos anteriores, consulte [Habilitación de la telemetría de Application Insights del lado servidor](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-application-insights-server-side-telemetry-visual-studio).

## <a name="2---add-instrumentation"></a>2- Incorporación de la instrumentación

En este paso se instrumenta su propia aplicación de búsqueda con el recurso de Application Insights creado en el paso anterior. Este proceso tiene cuatro pasos, empezando por la creación de un cliente de telemetría.

### <a name="step-1-create-a-telemetry-client"></a>Paso 1: Creación de un cliente de telemetría

Cree un objeto que envíe eventos a Application Insights. Puede agregar instrumentación al código de aplicación del lado servidor o al código del lado cliente que se ejecuta en un explorador, expresado aquí como variantes de C# y JavaScript (para otros lenguajes, consulte la lista completa de [plataformas y marcos compatibles](https://docs.microsoft.com/azure/application-insights/app-insights-platforms). Elija el enfoque que le proporcione la profundidad deseada de información.

Los datos de telemetría del lado servidor capturan métricas en la capa de aplicación, por ejemplo, en aplicaciones que se ejecutan como servicio web en la nube o como aplicación local en una red corporativa. Los datos de telemetría del lado servidor capturan los eventos de búsqueda y de clics, la posición de un documento en los resultados y la información de consulta, pero el ámbito de la colección de datos se limitará a la información que haya disponible en esa capa.

En el cliente, es posible que tenga código adicional que manipule las entradas de consulta, agregue navegación o incluya el contexto (por ejemplo, las consultas iniciadas desde una página principal en lugar de la página de un producto). Si esto describe su solución, puede optar por la instrumentación del lado cliente para que la telemetría refleje los detalles adicionales. La forma en que se recopilan estos detalles adicionales va más allá del ámbito de este patrón, pero puede revisar [Application Insights para páginas web](https://docs.microsoft.com/azure/azure-monitor/app/javascript#explore-browserclient-side-data) para obtener más instrucciones. 

**Uso de C#**

En C#, **InstrumentationKey** se encuentra en la configuración de la aplicación, como appsettings.json si el proyecto es ASP.NET. Consulte las instrucciones de registro si no está seguro de la ubicación de la clave.

```csharp
private static TelemetryClient _telemetryClient;

// Add a constructor that accepts a telemetry client:
public HomeController(TelemetryClient telemetry)
{
    _telemetryClient = telemetry;
}
```

**Uso de JavaScript**

```javascript
<script type="text/javascript">var appInsights=window.appInsights||function(config){function r(config){t[config]=function(){var i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var t={config:config},u=document,e=window,o="script",s=u.createElement(o),i,f;s.src=config.url||"//az416426.vo.msecnd.net/scripts/a/ai.0.js";u.getElementsByTagName(o)[0].parentNode.appendChild(s);try{t.cookie=u.cookie}catch(h){}for(t.queue=[],i=["Event","Exception","Metric","PageView","Trace","Dependency"];i.length;)r("track"+i.pop());return r("setAuthenticatedUserContext"),r("clearAuthenticatedUserContext"),config.disableExceptionTracking||(i="onerror",r("_"+i),f=e[i],e[i]=function(config,r,u,e,o){var s=f&&f(config,r,u,e,o);return s!==!0&&t["_"+i](config,r,u,e,o),s}),t}
({
instrumentationKey: "<YOUR INSTRUMENTATION KEY>"
});
window.appInsights=appInsights;
</script>
```

### <a name="step-2-request-a-search-id-for-correlation"></a>Paso 2: Solicitud de un id. de búsqueda para la correlación

Para correlacionar las solicitudes de búsqueda con clics, es necesario tener un identificador de correlación que relacione estos dos eventos distintos. Azure Cognitive Search proporciona un identificador de búsqueda cuando lo solicita con un encabezado HTTP.

Tener el identificador de búsqueda permite la correlación de las métricas emitidas por Azure Cognitive Search para la solicitud en sí, con las métricas personalizadas que está registrando en Application Insights.  

**Uso de C#**

```csharp
// This sample uses the .NET SDK https://www.nuget.org/packages/Microsoft.Azure.Search

var client = new SearchIndexClient(<SearchServiceName>, <IndexName>, new SearchCredentials(<QueryKey>)

// Use HTTP headers so that you can get the search ID from the response
var headers = new Dictionary<string, List<string>>() { { "x-ms-azs-return-searchid", new List<string>() { "true" } } };
var response = await client.Documents.SearchWithHttpMessagesAsync(searchText: searchText, searchParameters: parameters, customHeaders: headers);
string searchId = string.Empty;
if (response.Response.Headers.TryGetValues("x-ms-azs-searchid", out IEnumerable<string> headerValues))
{
    searchId = headerValues.FirstOrDefault();
}
```

**Uso de JavaScript (llamando a las API de REST)**

```javascript
request.setRequestHeader("x-ms-azs-return-searchid", "true");
request.setRequestHeader("Access-Control-Expose-Headers", "x-ms-azs-searchid");
var searchId = request.getResponseHeader('x-ms-azs-searchid');
```

### <a name="step-3-log-search-events"></a>Paso 3: Registro de eventos de búsqueda

Cada vez que un usuario emite una solicitud de búsqueda, debe registrarla como un evento de búsqueda con el esquema siguiente en un evento personalizado de Application Insights. Recuerde registrar solo las consultas de búsqueda generadas por el usuario.

+ **SearchServiceName**: (cadena) nombre del servicio de búsqueda
+ **SearchId**: (guid) identificador único de la consulta de búsqueda (se incluye en la respuesta de la búsqueda)
+ **IndexName**: (cadena) índice del servicio de búsqueda que se va a consultar
+ **QueryTerms**: (cadena) términos de búsqueda especificados por el usuario
+ **ResultCount**: (int) número de documentos devueltos (se incluye en la respuesta de la búsqueda)
+ **ScoringProfile**: (cadena) nombre del perfil de puntuación usado, si existe

> [!NOTE]
> Solicite el recuento de consultas generadas por el usuario agregando $count=true a la consulta de búsqueda. Para más información, consulte [Búsqueda de documentos (REST)](/rest/api/searchservice/search-documents#counttrue--false).
>

**Uso de C#**

```csharp
var properties = new Dictionary <string, string> 
{
    {"SearchServiceName", <service name>},
    {"SearchId", <search Id>},
    {"IndexName", <index name>},
    {"QueryTerms", <search terms>},
    {"ResultCount", <results count>},
    {"ScoringProfile", <scoring profile used>}
};
_telemetryClient.TrackEvent("Search", properties);
```

**Uso de JavaScript**

```javascript
appInsights.trackEvent("Search", {
  SearchServiceName: <service name>,
  SearchId: <search id>,
  IndexName: <index name>,
  QueryTerms: <search terms>,
  ResultCount: <results count>,
  ScoringProfile: <scoring profile used>
});
```

### <a name="step-4-log-click-events"></a>Paso 4: Registro de eventos de clic

Cada vez que un usuario hace clic en un documento, es una señal de que debe registrarse para fines de análisis de búsqueda. Utilice eventos personalizados de Application Insights para registrar estos eventos con el siguiente esquema:

+ **ServiceName**: (cadena) nombre del servicio de búsqueda
+ **SearchId**: (guid) identificador único de la consulta de búsqueda relacionada
+ **DocId**: (cadena) identificador del documento
+ **Position**: (int) clasificación del documento en la página de resultados de la búsqueda

> [!NOTE]
> La posición hace referencia al orden cardinal en la aplicación. Puede establecer este número, siempre que en todo momento sea el mismo, para permitir la comparación.
>

**Uso de C#**

```csharp
var properties = new Dictionary <string, string> 
{
    {"SearchServiceName", <service name>},
    {"SearchId", <search id>},
    {"ClickedDocId", <clicked document id>},
    {"Rank", <clicked document position>}
};
_telemetryClient.TrackEvent("Click", properties);
```

**Uso de JavaScript**

```javascript
appInsights.trackEvent("Click", {
    SearchServiceName: <service name>,
    SearchId: <search id>,
    ClickedDocId: <clicked document id>,
    Rank: <clicked document position>
});
```

## <a name="3---analyze-in-power-bi"></a>3- Análisis en Power BI

Una vez que haya instrumentado la aplicación y comprobado que esta se ha conectado correctamente a Application Insights, debe descargar una plantilla de informe predefinida para analizar los datos en Power BI Desktop. El informe contiene tablas y gráficos predefinidos útiles para analizar los datos adicionales que se capturaron para los análisis de tráfico de búsqueda.

1. En el panel de navegación izquierdo de Azure Cognitive Search, en **Configuración**, haga clic en **Análisis de tráfico de búsqueda**.

1. En la página **Análisis de tráfico de búsqueda**, en el paso 3, haga clic en **Obtener Power BI Desktop** para instalar Power BI.

   ![Obtención de informes de Power BI](./media/search-traffic-analytics/get-use-power-bi.png "Obtención de informes de Power BI")

1. En la misma página, haga clic en **Descargar informe de Power BI**.

1. El informe se abrirá en Power BI Desktop y se le pedirá que se conecte a Application Insights y proporcione las credenciales. Puede encontrar la información de conexión en las páginas de Azure Portal para el recurso de Application Insights. Para obtener las credenciales, proporcione el mismo nombre de usuario y la misma contraseña que usa para iniciar sesión en el portal.

   ![Conexión a Application Insights](./media/search-traffic-analytics/connect-to-app-insights.png "Conexión a Application Insights")

1. Haga clic en **Cargar**.

El informe contiene gráficos y tablas que le ayudarán a tomar decisiones más informadas para mejorar la relevancia y el rendimiento de las búsquedas.

Las métricas incluyen los siguientes elementos:

+ Busque los pares de términos y documentos más populares: términos resultantes en el mismo documento en el que se hizo clic, ordenados por clics.
+ Búsquedas sin clics: términos de las consultas principales que no registran ningún clic

En la captura de pantalla siguiente se muestra el aspecto que podría tener un informe integrado si hubiese usado todos los elementos del esquema.

![Panel de Power BI para Azure Cognitive Search](./media/search-traffic-analytics/azuresearch-powerbi-dashboard.png "Panel de Power BI para Azure Cognitive Search")

## <a name="next-steps"></a>Pasos siguientes

Instrumente la aplicación de búsqueda para obtener datos eficaces y reveladores sobre el servicio de búsqueda.

Puede encontrar más información en [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) y visitar la [página de precios](https://azure.microsoft.com/pricing/details/application-insights/) para conocer más sobre los distintos niveles de servicio.

Obtenga más información sobre cómo crear informes increíbles. Consulte [Introducción a Power BI Desktop](https://docs.microsoft.com/power-bi/fundamentals/desktop-getting-started) para más información.
