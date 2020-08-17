---
title: Reemplazo de los puntos de conexión de SDK predeterminados con Azure Application Insights
description: Modifique los puntos de conexión de SDK predeterminados de Azure Monitor Application Insights para regiones como Azure Government.
ms.topic: conceptual
ms.date: 07/26/2019
ms.custom: references_regions, devx-track-javascript
ms.openlocfilehash: 22f81b327c161df4f6605cfcb64b48b8d75beed3
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373137"
---
# <a name="application-insights-overriding-default-endpoints"></a>Reemplazo de puntos de conexión predeterminados de Application Insights

Para enviar datos de Application Insights en determinadas regiones, deberá reemplazar las direcciones de los puntos de conexión predeterminados. Cada SDK requiere modificaciones ligeramente distintas, las cuales se describen en este artículo. Dichos cambios requieren ajustar el código de ejemplo y reemplazar los valores de marcador de posición de `QuickPulse_Endpoint_Address`, `TelemetryChannel_Endpoint_Address` y `Profile_Query_Endpoint_address` con las direcciones de punto de conexión reales para la región específica. Al final de este artículo se incluyen vínculos a las direcciones de punto de conexión para las regiones en las que se requiere esta configuración.

> [!NOTE]
> Las [cadenas de conexión](./sdk-connection-string.md?tabs=net) son el nuevo método preferido de configuración de los puntos de conexión personalizados en Application Insights.

---

## <a name="sdk-code-changes"></a>Cambios de código del SDK

# <a name="net"></a>[.NET](#tab/net)

> [!NOTE]
> El archivo applicationinsights.config se sobrescribe automáticamente cada vez que se realiza una actualización de SDK. Después de realizar una actualización de SDK asegúrese de volver a escribir los valores de punto de conexión específicos para su región.

```xml
<ApplicationInsights>
  ...
  <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <QuickPulseServiceEndpoint>Custom_QuickPulse_Endpoint_Address</QuickPulseServiceEndpoint>
    </Add>
  </TelemetryModules>
    ...
  <TelemetryChannel>
     <EndpointAddress>TelemetryChannel_Endpoint_Address</EndpointAddress>
  </TelemetryChannel>
  ...
  <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
    <ProfileQueryEndpoint>Profile_Query_Endpoint_address</ProfileQueryEndpoint>
  </ApplicationIdProvider>
  ...
</ApplicationInsights>
```

# <a name="net-core"></a>[.NET Core](#tab/netcore)

Modifique el archivo appsettings.json en el proyecto como se indica a continuación para modificar el punto de conexión principal:

```json
"ApplicationInsights": {
    "InstrumentationKey": "instrumentationkey",
    "TelemetryChannel": {
      "EndpointAddress": "TelemetryChannel_Endpoint_Address"
    }
  }
```

Los valores de los puntos de conexión Live Metrics y Profile Query solo pueden establecerse mediante código. Para reemplazar los valores predeterminados de todos los valores de punto de conexión mediante código, realice los siguientes cambios en el método `ConfigureServices` del archivo `Startup.cs`:

```csharp
using Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId;
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse; //Place at top of Startup.cs file

   services.ConfigureTelemetryModule<QuickPulseTelemetryModule>((module, o) => module.QuickPulseServiceEndpoint="QuickPulse_Endpoint_Address");

   services.AddSingleton<IApplicationIdProvider, ApplicationInsightsApplicationIdProvider>(_ => new ApplicationInsightsApplicationIdProvider() { ProfileQueryEndpoint = "Profile_Query_Endpoint_address" });

   services.AddSingleton<ITelemetryChannel>(_ => new ServerTelemetryChannel() { EndpointAddress = "TelemetryChannel_Endpoint_Address" });

    //Place in the ConfigureServices method. Place this before services.AddApplicationInsightsTelemetry("instrumentation key"); if it's present
```

# <a name="azure-functions"></a>[Funciones de Azure](#tab/functions)

Para Azure Functions se recomienda usar [cadenas de conexión](./sdk-connection-string.md?tabs=net) establecidas en la configuración de la aplicación de la función. Para tener acceso a la configuración de la aplicación desde el panel funciones, seleccione **Configuración** > **configuración** > **Configuración de la aplicación**. 

Nombre: `APPLICATIONINSIGHTS_CONNECTION_STRING` Valor: `Connection String Value`

# <a name="java"></a>[Java](#tab/java)

Modifique el archivo applicationinsights.xml para cambiar la dirección predeterminada del punto de conexión.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
  <InstrumentationKey>ffffeeee-dddd-cccc-bbbb-aaaa99998888</InstrumentationKey>
  <TelemetryModules>
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
  </TelemetryModules>
  <TelemetryInitializers>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
  </TelemetryInitializers>
  <!--Add the following Channel value to modify the Endpoint address-->
  <Channel type="com.microsoft.applicationinsights.channel.concrete.inprocess.InProcessTelemetryChannel">
  <EndpointAddress>TelemetryChannel_Endpoint_Address</EndpointAddress>
  </Channel>
</ApplicationInsights>
```

### <a name="spring-boot"></a>Spring Boot

Modifique el archivo `application.properties` y agregue lo siguiente:

```yaml
azure.application-insights.channel.in-process.endpoint-address= TelemetryChannel_Endpoint_Address
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
var appInsights = require("applicationinsights");
appInsights.setup('INSTRUMENTATION_KEY');
appInsights.defaultClient.config.endpointUrl = "TelemetryChannel_Endpoint_Address"; // ingestion
appInsights.defaultClient.config.profileQueryEndpoint = "Profile_Query_Endpoint_address"; // appid/profile lookup
appInsights.defaultClient.config.quickPulseHost = "QuickPulse_Endpoint_Address"; //live metrics
appInsights.Configuration.start();
```

También se pueden configurar los puntos de conexión a través de variables de entorno:

```
Instrumentation Key: "APPINSIGHTS_INSTRUMENTATIONKEY"
Profile Endpoint: "Profile_Query_Endpoint_address"
Live Metrics Endpoint: "QuickPulse_Endpoint_Address"
```

# <a name="javascript"></a>[JavaScript](#tab/js)

```javascript
<script type="text/javascript">
    var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(e){function n(e){t[e]=function(){var n=arguments;t.queue.push(function(){t[e].apply(t,n)})}}var t={config:e};t.initialize=!0;var i=document,a=window;setTimeout(function(){var n=i.createElement("script");n.src=e.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",i.getElementsByTagName("script")[0].parentNode.appendChild(n)});try{t.cookie=i.cookie}catch(e){}t.queue=[],t.version=2;for(var r=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];r.length;)n("track"+r.pop());n("startTrackPage"),n("stopTrackPage");var s="Track"+r[0];if(n("start"+s),n("stop"+s),n("setAuthenticatedUserContext"),n("clearAuthenticatedUserContext"),n("flush"),!(!0===e.disableExceptionTracking||e.extensionConfig&&e.extensionConfig.ApplicationInsightsAnalytics&&!0===e.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){n("_"+(r="onerror"));var o=a[r];a[r]=function(e,n,i,a,s){var c=o&&o(e,n,i,a,s);return!0!==c&&t["_"+r]({message:e,url:n,lineNumber:i,columnNumber:a,error:s}),c},e.autoExceptionInstrumented=!0}return t}(
    {
      instrumentationKey:"INSTRUMENTATION_KEY",
      endpointUrl: "TelemetryChannel_Endpoint_Address"
    }
    );window[aiName]=aisdk,aisdk.queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
</script>
```

# <a name="python"></a>[Python](#tab/python)

Para obtener instrucciones sobre cómo modificar el punto de conexión de ingesta para el SDK de opencensus-python, consulte el [repositorio de opencensus-python.](https://github.com/census-instrumentation/opencensus-python/blob/af284a92b80bcbaf5db53e7e0813f96691b4c696/contrib/opencensus-ext-azure/opencensus/ext/azure/common/__init__.py)

---

## <a name="regions-that-require-endpoint-modification"></a>Regiones que requieren la modificación del punto de conexión

Actualmente, las únicas regiones que requieren modificaciones de punto de conexión son [Azure Government](../../azure-government/compare-azure-government-global-azure.md#application-insights) y [Azure China](/azure/china/resources-developer-guide).

|Region |  Nombre del punto de conexión | Value |
|-----------------|:------------|:-------------|
| Azure China | Canal de telemetría | `https://dc.applicationinsights.azure.cn/v2/track` |
| Azure China | QuickPulse (Live Metrics) |`https://live.applicationinsights.azure.cn/QuickPulseService.svc` |
| Azure China | Profile Query |`https://dc.applicationinsights.azure.cn/api/profiles/{0}/appId`  |
| Azure Government | Canal de telemetría |`https://dc.applicationinsights.us/v2/track` |
| Azure Government | QuickPulse (Live Metrics) |`https://quickpulse.applicationinsights.us/QuickPulseService.svc` |
| Azure Government | Profile Query |`https://dc.applicationinsights.us/api/profiles/{0}/appId` |

Si actualmente usa la [API REST de Application Insights](https://dev.applicationinsights.io/
), a la que normalmente se accede a través de "api.applicationinsights.io", deberá usar un punto de conexión local a su región:

|Region |  Nombre del punto de conexión | Value |
|-----------------|:------------|:-------------|
| Azure China | API DE REST | `api.applicationinsights.azure.cn` |
| Azure Government | API DE REST | `api.applicationinsights.us`|

> [!NOTE]
> La supervisión sin código basada en agentes o extensiones de Azure App Service **actualmente no se admite** en estas regiones. Tan pronto como esta funcionalidad esté disponible, este artículo se actualizará.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de las modificaciones personalizadas para Azure Government, consulte la guía detallada para [administración y supervisión de Azure](../../azure-government/compare-azure-government-global-azure.md#application-insights).
- Para obtener más información acerca de Azure China, consulte el [cuaderno de estrategias de Azure China](/azure/china/).
