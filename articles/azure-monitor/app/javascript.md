---
title: Azure Application Insights para aplicaciones web de JavaScript
description: Obtenga recuentos de sesiones y vistas de página, además de datos de cliente web y Aplicaciones de página única, y realice el seguimiento de los patrones de uso. Detecte problemas de rendimiento y excepciones en páginas web de JavaScript.
ms.topic: conceptual
ms.date: 08/06/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 3acb7379644b5bfcb22ed86b6bde7031095fef24
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88224860"
---
# <a name="application-insights-for-web-pages"></a>Application Insights para páginas web

Obtenga información sobre el rendimiento y la utilización de su página web o aplicación. Si agrega [Application Insights](app-insights-overview.md) al script de la página, obtendrá los intervalos de carga de la página y de las llamadas AJAX, recuentos y detalles sobre las excepciones del explorador y los errores de AJAX, así como usuarios y recuentos de sesiones. Todos estos datos se pueden segmentar por página, sistema operativo del cliente y versión del explorador, geolocalización y otras dimensiones. Puede establecer alertas sobre recuentos de errores o sobre cargas de página lentas. Y mediante la inserción de llamadas de seguimiento en el código de JavaScript, puede controlar cómo se utilizan las distintas características de la aplicación de la página web.

Puede utilizar Application Insights con cualquier página web, solo tiene que agregar un pequeño fragmento de JavaScript. Si el servicio web es [Java](java-get-started.md) o [ASP.NET](asp-net.md), puede usar los SDK del lado servidor junto con el SDK de JavaScript del lado cliente para obtener una descripción completa del rendimiento de la aplicación.

## <a name="adding-the-javascript-sdk"></a>Adición del SDK de JavaScript

1. En primer lugar, necesita un recurso de Application Insights. Si aún no tiene un recurso y una clave de instrumentación, siga las [instrucciones para crear un nuevo recurso](create-new-resource.md).
2. Copie la _clave de instrumentación_ (también conocida como "iKey") del recurso al que desea que se envíe la telemetría de JavaScript (del paso 1). Lo agregará al valor `instrumentationKey` del SDK de JavaScript de Application Insights.
3. Agregue el SDK de JavaScript de Application Insights a su página web o aplicación mediante una de las dos opciones siguientes:
    * [Configuración de npm](#npm-based-setup)
    * [Fragmento de código Javascript](#snippet-based-setup)

> [!IMPORTANT]
> Use solo un método para agregar el SDK de JavaScript a la aplicación. Si usa el programa de instalación de NPM, no use el fragmento de código y viceversa.

> [!NOTE]
> El programa de instalación de NPM instala el SDK de JavaScript como una dependencia del proyecto, habilitando IntelliSense, mientras que el fragmento de código captura el SDK en tiempo de ejecución. Ambos admiten las mismas características. Sin embargo, los desarrolladores que desean más eventos y configuración personalizados normalmente optan por el programa de instalación de NPM, mientras que los usuarios que buscan una habilitación rápida del análisis web integrado optan por el fragmento de código.

### <a name="npm-based-setup"></a>Configuración basada en npm

Instale a través de NPM.

```sh
npm i --save @microsoft/applicationinsights-web
```

> [!Note]
> **Con este paquete se incluyen typings**, por lo que **no** necesita instalar un paquete de typings independiente.
    
```js
import { ApplicationInsights } from '@microsoft/applicationinsights-web'

const appInsights = new ApplicationInsights({ config: {
  instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE'
  /* ...Other Configuration Options... */
} });
appInsights.loadAppInsights();
appInsights.trackPageView(); // Manually call trackPageView to establish the current user/session/pageview
```

### <a name="snippet-based-setup"></a>Configuración basada en fragmento de código

Si su aplicación no usa npm, puede instrumentar directamente las páginas web con Application Insights pegando este fragmento de código en la parte superior de cada una de las páginas. Preferiblemente, debe ser el primer script de la sección `<head>` para que pueda supervisar cualquier posible problema con todas las dependencias y, opcionalmente, cualquier error de JavaScript. Si usa una aplicación de servidor de Blazor, agregue el fragmento de código en la parte superior del archivo `_Host.cshtml` en la sección `<head>`.

Para ayudar a realizar un seguimiento de la versión del fragmento de código que la aplicación usa, a partir de la versión 2.5.5, el evento de vista de página incluirá una nueva etiqueta "ai.internal.snippet" que contendrá la versión de fragmento de código identificada.

El fragmento de código actual (que se muestra a continuación) se identificará como la versión "3".

```html
<script type="text/javascript">
!function(T,l,y){var S=T.location,u="script",k="instrumentationKey",D="ingestionendpoint",C="disableExceptionTracking",E="ai.device.",I="toLowerCase",b="crossOrigin",w="POST",e="appInsightsSDK",t=y.name||"appInsights";(y.name||T[e])&&(T[e]=t);var n=T[t]||function(d){var g=!1,f=!1,m={initialize:!0,queue:[],sv:"4",version:2,config:d};function v(e,t){var n={},a="Browser";return n[E+"id"]=a[I](),n[E+"type"]=a,n["ai.operation.name"]=S&&S.pathname||"_unknown_",n["ai.internal.sdkVersion"]="javascript:snippet_"+(m.sv||m.version),{time:function(){var e=new Date;function t(e){var t=""+e;return 1===t.length&&(t="0"+t),t}return e.getUTCFullYear()+"-"+t(1+e.getUTCMonth())+"-"+t(e.getUTCDate())+"T"+t(e.getUTCHours())+":"+t(e.getUTCMinutes())+":"+t(e.getUTCSeconds())+"."+((e.getUTCMilliseconds()/1e3).toFixed(3)+"").slice(2,5)+"Z"}(),iKey:e,name:"Microsoft.ApplicationInsights."+e.replace(/-/g,"")+"."+t,sampleRate:100,tags:n,data:{baseData:{ver:2}}}}var h=d.url||y.src;if(h){function a(e){var t,n,a,i,r,o,s,c,p,l,u;g=!0,m.queue=[],f||(f=!0,t=h,s=function(){var e={},t=d.connectionString;if(t)for(var n=t.split(";"),a=0;a<n.length;a++){var i=n[a].split("=");2===i.length&&(e[i[0][I]()]=i[1])}if(!e[D]){var r=e.endpointsuffix,o=r?e.location:null;e[D]="https://"+(o?o+".":"")+"dc."+(r||"services.visualstudio.com")}return e}(),c=s[k]||d[k]||"",p=s[D],l=p?p+"/v2/track":config.endpointUrl,(u=[]).push((n="SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details)",a=t,i=l,(o=(r=v(c,"Exception")).data).baseType="ExceptionData",o.baseData.exceptions=[{typeName:"SDKLoadFailed",message:n.replace(/\./g,"-"),hasFullStack:!1,stack:n+"\nSnippet failed to load ["+a+"] -- Telemetry is disabled\nHelp Link: https://go.microsoft.com/fwlink/?linkid=2128109\nHost: "+(S&&S.pathname||"_unknown_")+"\nEndpoint: "+i,parsedStack:[]}],r)),u.push(function(e,t,n,a){var i=v(c,"Message"),r=i.data;r.baseType="MessageData";var o=r.baseData;return o.message='AI (Internal): 99 message:"'+("SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details) ("+n+")").replace(/\"/g,"")+'"',o.properties={endpoint:a},i}(0,0,t,l)),function(e,t){if(JSON){var n=T.fetch;if(n&&!y.useXhr)n(t,{method:w,body:JSON.stringify(e),mode:"cors"});else if(XMLHttpRequest){var a=new XMLHttpRequest;a.open(w,t),a.setRequestHeader("Content-type","application/json"),a.send(JSON.stringify(e))}}}(u,l))}function i(e,t){f||setTimeout(function(){!t&&m.core||a()},500)}var e=function(){var n=l.createElement(u);n.src=h;var e=y[b];return!e&&""!==e||"undefined"==n[b]||(n[b]=e),n.onload=i,n.onerror=a,n.onreadystatechange=function(e,t){"loaded"!==n.readyState&&"complete"!==n.readyState||i(0,t)},n}();y.ld<0?l.getElementsByTagName("head")[0].appendChild(e):setTimeout(function(){l.getElementsByTagName(u)[0].parentNode.appendChild(e)},y.ld||0)}try{m.cookie=l.cookie}catch(p){}function t(e){for(;e.length;)!function(t){m[t]=function(){var e=arguments;g||m.queue.push(function(){m[t].apply(m,e)})}}(e.pop())}var n="track",r="TrackPage",o="TrackEvent";t([n+"Event",n+"PageView",n+"Exception",n+"Trace",n+"DependencyData",n+"Metric",n+"PageViewPerformance","start"+r,"stop"+r,"start"+o,"stop"+o,"addTelemetryInitializer","setAuthenticatedUserContext","clearAuthenticatedUserContext","flush"]),m.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4};var s=(d.extensionConfig||{}).ApplicationInsightsAnalytics||{};if(!0!==d[C]&&!0!==s[C]){method="onerror",t(["_"+method]);var c=T[method];T[method]=function(e,t,n,a,i){var r=c&&c(e,t,n,a,i);return!0!==r&&m["_"+method]({message:e,url:t,lineNumber:n,columnNumber:a,error:i}),r},d.autoExceptionInstrumented=!0}return m}(y.cfg);(T[t]=n).queue&&0===n.queue.length&&n.trackPageView({})}(window,document,{
src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js", // The SDK URL Source
//name: "appInsights", // Global SDK Instance name defaults to "appInsights" when not supplied
//ld: 0, // Defines the load delay (in ms) before attempting to load the sdk. -1 = block page load and add to head. (default) = 0ms load after timeout,
//useXhr: 1, // Use XHR instead of fetch to report failures (if available),
//crossOrigin: "anonymous", // When supplied this will add the provided value as the cross origin attribute on the script tag 
cfg: { // Application Insights Configuration
    instrumentationKey: "YOUR_INSTRUMENTATION_KEY_GOES_HERE"
    /* ...Other Configuration Options... */
}});
</script>
```

> [!NOTE]
> Para mejorar la legibilidad y reducir errores potenciales de JavaScript, todas las opciones de configuración posibles se muestran en una nueva línea en el código del fragmento de código anterior. Si no desea cambiar el valor de una línea comentada, se puede quitar.


#### <a name="reporting-script-load-failures"></a>Informes de errores de carga de script

Esta versión del fragmento detecta errores e informa de ellos al cargar el SDK desde la red CDN como una excepción en el portal de Azure Monitor (en errores &gt; excepciones &gt; explorador). Esta excepción proporciona visibilidad en los errores de este tipo para que tenga en cuenta que la aplicación no informa de la telemetría (u otras excepciones) según lo esperado. Esta señal es una medida importante para comprender que ha perdido la telemetría porque el SDK no se cargó o se inicializó, lo que puede dar lugar a:
- Informes incompletos del uso que los usuarios están haciendo (o intentan hacer ) del sitio;
- Falta de telemetría sobre el modo en que los usuarios finales usan su sitio;
- Ausencia de errores de JavaScript que podrían estar impidiendo que los usuarios finales usaran el sitio correctamente.

Para obtener detalles sobre esta excepción, vea la página de solución de problemas [Error de carga del SDK](javascript-sdk-load-failure.md).

Los informes de este error como una excepción en el portal no usan la opción de configuración ```disableExceptionTracking``` de la configuración de Application Insights y, por tanto, si se produce este error, el fragmento de código siempre lo notificará, aunque la compatibilidad con window.onerror esté deshabilitada.

Los informes de errores de carga del SDK NO se admiten específicamente en IE 8 (o versiones inferiores). Esto ayuda a disminuir el tamaño reducido del fragmento de código suponiendo que la mayoría de los entornos no usan exclusivamente IE 8 o una versión inferior. Si tiene este requisito y desea recibir estas excepciones, deberá incluir un polyfill de fetch o crear su propia versión de fragmento de código que use ```XDomainRequest``` en lugar de ```XMLHttpRequest```. Se recomienda usar el [código fuente del fragmento de código proporcionado](https://github.com/microsoft/ApplicationInsights-JS/blob/master/AISKU/snippet/snippet.js) como punto de partida.

> [!NOTE]
> Si usa una versión anterior del fragmento de código, se recomienda encarecidamente que actualice a la versión más reciente para que reciba estos problemas no notificados previamente.

#### <a name="snippet-configuration-options"></a>Opciones de configuración del fragmento de código

Ahora, todas las opciones de configuración se han trasladado al final del script para evitar la introducción accidental de errores de JavaScript que no solo harían que el SDK no pudiera cargarse, sino que también deshabilitaría la notificación del error.

Cada opción de configuración se muestra arriba en una nueva línea. Si no desea invalidar el valor predeterminado de un elemento mostrado como [opcional], puede quitar esa línea para minimizar el tamaño resultante de la página devuelta.

Las opciones de configuración disponibles son las siguientes: 

| Nombre | Tipo | Descripción
|------|------|----------------
| src | string **[obligatorio]** | Dirección URL completa desde la que se va a cargar el SDK. Este valor se usa para el atributo "src" de una etiqueta &lt;script /&gt; agregada dinámicamente. Puede usar la ubicación de la red CDN pública o la suya propia hospedada de forma privada.
| name | cadena *[opcional]* | El nombre global del SDK inicializado. El valor predeterminado es `appInsights`. Por lo tanto, ```window.appInsights``` será una referencia a la instancia inicializada. Nota: Si proporciona un valor de nombre o una instancia anterior parece estar asignada (mediante el nombre global appInsightsSDK), este valor de nombre también se definirá en el espacio de nombres global como ```window.appInsightsSDK=<name value>```. El código de inicialización del SDK requiere todo esto para asegurarse de que está inicializando y actualizando los métodos de proxy y esqueleto del fragmento de código correctos.
| ld | número en ms *[opcional]* | Define el retraso de carga que se debe esperar antes de intentar cargar el SDK. El valor predeterminado es 0 ms y cualquier valor negativo agregará una etiqueta de script inmediatamente a la región &lt;head&gt; de la página, que luego bloqueará el evento de carga de la página hasta que se cargue el script (o se produzca un error).
| useXhr | booleano *[opcional]* | Este valor solo se usa para notificar errores de carga del SDK. Los informes intentarán primero usar fetch() si está disponible y luego recurrirán a XHR. Si establece este valor en true, simplemente se omitirá la comprobación de fetch. El uso de este valor solo es necesario si la aplicación se usa en un entorno en el que fetch no enviaría los eventos de error.
| crossOrigin | cadena *[opcional]* | Al incluir este valor, la etiqueta de script agregada para descargar el SDK incluirá el atributo crossOrigin con este valor de cadena. Cuando no se define (que es la opción predeterminada), no se agrega ningún atributo crossOrigin. Los valores recomendados no se definen (que es la opción predeterminada); ""; o "anonymous" (para todos los valores válidos, consulte la documentación [Atributo HTML: `crossorigin`](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin))
| cfg | objeto **[obligatorio]** | Se trata de la configuración pasada al SDK de Application Insights durante la inicialización.

### <a name="sending-telemetry-to-the-azure-portal"></a>Envío de telemetría a Azure Portal

De forma predeterminada, el SDK de JavaScript de Application Insights recopila automáticamente un número de elementos de telemetría que son útiles para determinar el estado de la aplicación y la experiencia del usuario subyacente. Entre ellas se incluyen las siguientes:

- **Excepciones no detectadas** en la aplicación, incluida información sobre lo siguiente:
    - Seguimiento de la pila
    - Detalles de la excepción y mensaje que acompaña al error
    - Número de línea y columna del error
    - Dirección URL en la que se produjo el error
- **Solicitudes de dependencia de red** realizadas por las solicitudes **XHR** and **Fetch** (la recopilación de capturas está deshabilitada de forma predeterminada) de la aplicación, incluida información sobre lo siguiente:
    - Dirección URL del origen de dependencia
    - Comando y método usado para solicitar la dependencia
    - Duración de la solicitud
    - Código de resultado y estado correcto de la solicitud
    - Identificador (si existe) del usuario que realiza la solicitud
    - Contexto de correlación (si existe) en el que se realiza la solicitud
- **Información del usuario** (por ejemplo, ubicación, red, IP)
- **Información del dispositivo** (por ejemplo, explorador, sistema operativo, versión, idioma, modelo)
- **Información de la sesión**

### <a name="telemetry-initializers"></a>Inicializadores de telemetría
Los inicializadores de telemetría se usan para modificar el contenido de la telemetría recopilada antes de enviarla desde el explorador del usuario. También se pueden usar para impedir que se envíen determinados datos de telemetría, devolviendo `false`. Se pueden agregar varios inicializadores de telemetría a la instancia de Application Insights, que se ejecutarán en el orden en el que se han agregado.

El argumento de entrada en `addTelemetryInitializer` es una devolución de llamada que toma [`ITelemetryItem`](https://github.com/microsoft/ApplicationInsights-JS/blob/master/API-reference.md#addTelemetryInitializer) como argumento y devuelve `boolean` o `void`. Si devuelve `false`, no se envía el elemento de telemetría, sino que pasa al siguiente inicializador de telemetría, si existe, o se envía al punto de conexión de recopilación de telemetría.

Ejemplo de uso de inicializadores de telemetría:
```ts
var telemetryInitializer = (envelope) => {
  envelope.data.someField = 'This item passed through my telemetry initializer';
};
appInsights.addTelemetryInitializer(telemetryInitializer);
appInsights.trackTrace({message: 'This message will use a telemetry initializer'});

appInsights.addTelemetryInitializer(() => false); // Nothing is sent after this is executed
appInsights.trackTrace({message: 'this message will not be sent'}); // Not sent
```

## <a name="configuration"></a>Configuración
La mayoría de los campos de configuración tienen un nombre que permite establecer false como valor predeterminado. Todos los campos son opcionales, excepto `instrumentationKey`.

| Nombre | Valor predeterminado | Descripción |
|------|---------|-------------|
| instrumentationKey | null | **Obligatorio**<br>La clave de instrumentación que ha obtenido en Azure Portal. |
| accountId | null | Un identificador de cuenta opcional, si su aplicación agrupa a los usuarios en cuentas. Sin espacios, comas, puntos y comas, signos igual o barras verticales. |
| sessionRenewalMs | 1800000 | Se registra una sesión si el usuario está inactivo durante este período de tiempo en milisegundos. El valor predeterminado es 30 minutos. |
| sessionExpirationMs | 86400000 | Se registra una sesión si ha continuado durante este período de tiempo en milisegundos. El valor predeterminado es 24 horas. |
| maxBatchSizeInBytes | 10 000 | Tamaño máximo del lote de telemetría. Si un lote supera este límite, se envía inmediatamente y se inicia un nuevo lote. |
| maxBatchInterval | 15000 | Cuánto tiempo deben recopilar datos de telemetría en lote antes de enviarlos (milisegundos). |
| disableExceptionTracking | false | Si es true, las excepciones no se recopilan automáticamente. El valor predeterminado es False. |
| disableTelemetry | false | Si es true, no se recopila ni se envía la telemetría. El valor predeterminado es False. |
| enableDebug | false | Si es true, los datos de depuración **internos** se devuelven como una excepción **en lugar de** registrarse, independientemente de la configuración de registro del SDK. El valor predeterminado es False. <br>***Nota:*** Si se habilita este valor, se descartará la telemetría siempre que se produzca un error interno. Esto puede ser útil para identificar rápidamente problemas con la configuración o el uso del SDK. Si no desea perder telemetría durante la depuración, considere la posibilidad de usar `consoleLoggingLevel` o `telemetryLoggingLevel` en lugar de `enableDebug`. |
| loggingLevelConsole | 0 | Registra errores **internos** de Application Insights en la consola. <br>0: desactivado <br>1: solo errores críticos <br>2: todo (errores y advertencias) |
| loggingLevelTelemetry | 1 | Envía errores **internos** de Application Insights como telemetría. <br>0: desactivado <br>1: solo errores críticos <br>2: todo (errores y advertencias) |
| diagnosticLogInterval | 10 000 | (interno) Intervalo de sondeo (en milisegundos) para la cola de registro interno |
| samplingPercentage | 100 | Porcentaje de eventos que se enviarán. El valor predeterminado es 100, lo que significa que se envían todos los eventos. Establézcalo si desea conservar el límite de datos para aplicaciones a gran escala. |
| autoTrackPageVisitTime | false | Si es true, en una vista de página, se realiza un seguimiento del tiempo de visualización de la página instrumentada anterior, que se envía como telemetría, y se inicia un nuevo temporizador para la vista de página actual. El valor predeterminado es False. |
| disableAjaxTracking | false | Si es true, las llamadas Ajax no se recopilan automáticamente. El valor predeterminado es False. |
| disableFetchTracking | true | Si es true, las solicitudes Fetch no se recopilan automáticamente. El valor predeterminado es true. |
| overridePageViewDuration | false | Si es true, el comportamiento predeterminado de trackPageView se cambia para registrar el final del intervalo de duración de la vista de página cuando se llama a trackPageView. Si es false y no se proporciona ninguna duración personalizada a trackPageView, el rendimiento de la vista de página se calcula mediante la API de tiempos de navegación. El valor predeterminado es False. |
| maxAjaxCallsPerView | 500 | El valor predeterminado es 500: controla cuántas llamadas AJAX se supervisarán por vista de página. Establézcalo en -1 para supervisar todas las llamadas AJAX (ilimitadas) en la página. |
| disableDataLossAnalysis | true | Si es false, se comprobará al inicio en los búferes del remitente de telemetría interna si hay elementos que todavía no se hayan enviado. |
| disableCorrelationHeaders | false | Si es false, el SDK agregará dos encabezados ("Request-Id" y "Request-Context") a todas las solicitudes de dependencia para correlacionarlas con las solicitudes correspondientes en el lado servidor. El valor predeterminado es False. |
| correlationHeaderExcludedDomains |  | Deshabilite encabezados de correlación para dominios específicos. |
| correlationHeaderDomains |  | Habilite encabezados de correlación para dominios específicos. |
| disableFlushOnBeforeUnload | false | El valor predeterminado es false. Si es true, no se llamará al método Flush cuando se desencadene un evento onBeforeUnload. |
| enableSessionStorageBuffer | true | El valor predeterminado es true. Si es true, el búfer con toda la telemetría no enviada se guarda en el almacenamiento de la sesión. El búfer se restaura al cargar la página. |
| isCookieUseDisabled | false | El valor predeterminado es false. Si es true, el SDK no almacenará ni leerá ningún dato de las cookies.|
| cookieDomain | null | Dominio de cookies personalizado. Resulta útil si desea compartir cookies de Application Insights entre subdominios. |
| isRetryDisabled | false | El valor predeterminado es false. Si es false, se produce un reintento con los errores 206 (parcialmente correcto), 408 (tiempo de espera), 429 (demasiadas solicitudes), 500 (error interno del servidor), 503 (servicio no disponible) y 0 (sin conexión, solo si se detecta). |
| isStorageUseDisabled | false | Si es true, el SDK no almacenará ni leerá ningún dato del almacenamiento local o de la sesión. El valor predeterminado es False. |
| isBeaconApiDisabled | true | Si es false, el SDK enviará toda la telemetría mediante [Beacon API](https://www.w3.org/TR/beacon). |
| onunloadDisableBeacon | false | El valor predeterminado es false. Si la pestaña está cerrada, el SDK enviará la telemetría restante mediante la [API Beacon ](https://www.w3.org/TR/beacon). |
| sdkExtension | null | Establece el nombre de la extensión del SDK. Solo puede contener caracteres alfabéticos. El nombre de la extensión se agrega como prefijo a la etiqueta "ai.internal.sdkVersion" (por ejemplo, "ext_javascript:2.0.0"). El valor predeterminado es null. |
| isBrowserLinkTrackingEnabled | false | El valor predeterminado es False. Si es true, el SDK realizará un seguimiento de todas las solicitudes de [Vínculo con exploradores](/aspnet/core/client-side/using-browserlink). |
| appId | null | AppId se utiliza para la correlación entre las dependencias AJAX que se producen en el lado cliente con las solicitudes del lado servidor. Cuando Beacon API está habilitada, no se puede usar automáticamente, pero se puede establecer manualmente en la configuración. El valor predeterminado es null. |
| enableCorsCorrelation | false | Si es true, el SDK agregará dos encabezados ("Request-Id" y "Request-Context") a todas las solicitudes CORS para correlacionar dependencias AJAX salientes con las solicitudes correspondientes en el lado servidor. El valor predeterminado es false. |
| namePrefix | no definido | Un valor opcional que se usará como sufijo de nombre para el nombre de cookies y localStorage.
| enableAutoRouteTracking | false | Realice un seguimiento automático de los cambios de ruta en aplicaciones de página única (SPA). Si es true, cada cambio de ruta enviará una nueva vista de página a Application Insights. Los cambios de ruta hash (`example.com/foo#bar`) también se registran como nuevas vistas de página.
| enableRequestHeaderTracking | false | Si es true, se realiza un seguimiento de los encabezados de solicitud AJAX y Fetch. El valor predeterminado es false.
| enableResponseHeaderTracking | false | Si es true, se realiza un seguimiento de los encabezados de respuesta de solicitud AJAX y Fetch. El valor predeterminado es false.
| distributedTracingMode | `DistributedTracingModes.AI` | Establece del modo de seguimiento distribuido. Si se establece el modo AI_AND_W3C o el modo W3C, se generarán los encabezados de contexto de seguimiento W3C (traceparent/tracestate) y se incluirán en todas las solicitudes salientes. AI_AND_W3C se proporciona para la compatibilidad con versiones anteriores de cualquier servicio instrumentado de Application Insights heredado. Consulte el ejemplo [aquí](./correlation.md#enable-w3c-distributed-tracing-support-for-web-apps).
| enableAjaxErrorStatusText | false | El valor predeterminado es false. Si es true, incluye el texto de datos de error de respuesta en el evento de dependencia en solicitudes AJAX erróneas.
| enableAjaxPerfTracking | false | El valor predeterminado es false. Marca para habilitar la búsqueda y la inclusión de intervalos de window.performance de explorador adicionales en las métricas notificadas de `ajax` (XHR y Fetch).
| maxAjaxPerfLookupAttempts | 3 | El valor predeterminado es 3. Número máximo de veces que se deben buscar los intervalos de window.performance (si están disponibles). Esto es necesario, ya que no todos los exploradores rellenan window.performance antes de notificar el final de la solicitud de XHR y para las solicitudes de fetch se agregan después de su terminación.
| ajaxPerfLookupDelay | 25 | Su valor predeterminado es 25 ms. Cantidad de tiempo que hay que esperar antes de volver a intentar encontrar los intervalos de windows.performance para una solicitud `ajax`. El tiempo se da en milisegundos y se pasa directamente a setTimeout().
| enableUnhandledPromiseRejectionTracking | false | Si es true, los rechazos de promise no controlados se recopilarán automáticamente y se notificarán como un error de JavaScript. Cuando disableExceptionTracking es true (no realizar seguimiento de excepciones), el valor de configuración se omite y los rechazos de promise no controlados no se notifican.

## <a name="single-page-applications"></a>Aplicaciones de página única

De forma predeterminada, este SDK **no** controlará el cambio de ruta basado en el estado que se produce en las aplicaciones de página única. Para habilitar el seguimiento automático de cambios de ruta para una aplicación de página única, puede agregar `enableAutoRouteTracking: true` a la configuración de instalación.

Actualmente, ofrecemos un [complemento React](#react-extensions) independiente que puede inicializarse con este SDK. También llevará a cabo el seguimiento de cambios de ruta, así como la recopilación de [otros datos de telemetría específicos de React](https://github.com/microsoft/ApplicationInsights-JS/blob/17ef50442f73fd02a758fbd74134933d92607ecf/extensions/applicationinsights-react-js/README.md).

> [!NOTE]
> Use `enableAutoRouteTracking: true` solo si **no** está usando el complemento React. Ambos pueden enviar nuevos valores PageView cuando cambia la ruta. Si ambos están habilitados, es posible que se envíen valores de PageView duplicados.

## <a name="configuration-autotrackpagevisittime"></a>Configuración: autoTrackPageVisitTime

Al establecer `autoTrackPageVisitTime: true`, se realiza un seguimiento del tiempo que un usuario permanece en cada página. En cada instancia nueva de PageView, el tiempo que pasó el usuario en la página anterior (*previous*) se envía como una [métrica personalizada](../platform/metrics-custom-overview.md) denominada `PageVisitTime`. Esta métrica personalizada es visible en el [Explorador de métricas](../platform/metrics-getting-started.md) como "Métricas basadas en registros".

## <a name="react-extensions"></a>Extensiones de React

| Extensiones |
|---------------|
| [React](javascript-react-plugin.md)|
| [React Native](javascript-react-native-plugin.md)|

## <a name="correlation"></a>Correlación

La correlación entre el cliente y el servidor es compatible con:

- Solicitudes XHR/AJAX 
- Solicitudes Fetch 

La correlación entre el cliente y el servidor **no es compatible** con solicitudes `GET` ni `POST`.

### <a name="enable-cross-component-correlation-between-client-ajax-and-server-requests"></a>Habilitación de la correlación entre componentes entre las solicitudes de servidor y AJAX de cliente

Para habilitar la correlación `CORS`, el cliente debe enviar dos encabezados de solicitud adicionales `Request-Id` y `Request-Context`, y el lado servidor debe poder aceptar conexiones con los encabezados presentes. El envío de estos encabezados se habilita estableciendo `enableCorsCorrelation: true` dentro de la configuración del SDK de JavaScript. 

En función de la configuración de `Access-Control-Allow-Headers` en el lado servidor, a menudo es necesario extender la lista del lado servidor mediante la adición manual de `Request-Id` y `Request-Context`.

Access-Control-Allow-Headers: `Request-Id`, `Request-Context`, `<your header>`

Si alguno de los servidores de terceros con los que se comunica el cliente no puede aceptar los encabezados `Request-Id` y `Request-Context`, y no es posible actualizar su configuración, deberá colocarlos en una lista de exclusión a través de la propiedad de configuración `correlationHeaderExcludeDomains`. Esta propiedad admite caracteres comodín.

```javascript
// excerpt of the config section of the JavaScript SDK snippet with correlation
// between client-side AJAX and server requests enabled.
cfg: { // Application Insights Configuration
    instrumentationKey: "YOUR_INSTRUMENTATION_KEY_GOES_HERE"
    enableCorsCorrelation: true,
    correlationHeaderExcludedDomains: ['myapp.azurewebsites.net', '*.queue.core.windows.net']
    /* ...Other Configuration Options... */
}});
</script>

``` 

## <a name="explore-browserclient-side-data"></a>Exploración de datos del explorador o del lado cliente

Para ver datos del explorador o del lado cliente, vaya a **Métricas** y agregue métricas individuales que le interesen:

![Captura de pantalla de la página Métricas de Application Insights que muestra presentaciones gráficas de los datos de métricas de una aplicación web.](./media/javascript/page-view-load-time.png)

También puede ver los datos desde el SDK de JavaScript mediante la experiencia de explorador en el portal.

Seleccione **Explorador** y, a continuación, elija **Errores** o **Rendimiento**.

![Captura de pantalla de la página Explorador de Application Insights que muestra cómo agregar errores del explorador o el rendimiento del explorador a las métricas que se pueden ver de la aplicación web.](./media/javascript/browser.png)

### <a name="performance"></a>Rendimiento

![Captura de pantalla de la página Rendimiento de Application Insights muestra representaciones gráficas de las métricas Operaciones de una aplicación web.](./media/javascript/performance-operations.png)

### <a name="dependencies"></a>Dependencias

![Captura de pantalla de la página Rendimiento de Application Insights muestra representaciones gráficas de las métricas Dependencia de una aplicación web.](./media/javascript/performance-dependencies.png)

### <a name="analytics"></a>Análisis

Para consultar la telemetría recopilada por el SDK de JavaScript, seleccione el botón **Ver en registros (Analytics)** . Al agregar una instrucción `where` de `client_Type == "Browser"`, solo verá los datos del SDK de JavaScript y se excluirán todos los datos de telemetría del lado servidor recopilados por otros SDK.
 
```kusto
// average pageView duration by name
let timeGrain=5m;
let dataset=pageViews
// additional filters can be applied here
| where timestamp > ago(1d)
| where client_Type == "Browser" ;
// calculate average pageView duration for all pageViews
dataset
| summarize avg(duration) by bin(timestamp, timeGrain)
| extend pageView='Overall'
// render result in a chart
| render timechart
```

### <a name="source-map-support"></a>Compatibilidad con mapa de origen

La pila de llamadas compactada de los datos de telemetría de las excepciones se puede descompactar en Azure Portal. Todas las integraciones existentes en el panel Detalles de la excepción funcionarán con la pila de llamadas recién descompactada.

#### <a name="link-to-blob-storage-account"></a>Vínculo a la cuenta de Blob Storage

Puede vincular su recurso de Application Insights a su propio contenedor de Azure Blob Storage para desminificar las pilas de llamadas automáticamente. Para empezar, consulte el artículo sobre la [compatibilidad con mapa de origen automático](./source-map-support.md).

### <a name="drag-and-drop"></a>Arrastrar y colocar

1. Seleccione un elemento de telemetría de excepciones en Azure Portal para ver sus detalles de transacción completa.
2. Identifique qué mapas de origen corresponden a esta pila de llamadas. El mapa de origen debe coincidir con el archivo de origen de un marco de pila, pero con el sufijo `.map`.
3. Arrastre y coloque los mapas de origen en la pila de llamadas en Azure Portal ![Imagen animada que muestra cómo arrastrar y colocar archivos del mapa de origen de una carpeta de compilación a la ventana Pila de llamadas de Azure Portal.](https://i.imgur.com/Efue9nU.gif)

### <a name="application-insights-web-basic"></a>Versión web básica de Application Insights

Para una experiencia ligera, puede instalar la versión básica de Application Insights.
```
npm i --save @microsoft/applicationinsights-web-basic
```
Esta versión incluye las características y funcionalidades mínimas, que puede ir ampliando como desee. Por ejemplo, no realiza ninguna recopilación automática (excepciones no detectadas, AJAX, etc.). En esta versión no se incluyen las API para enviar determinados tipos de telemetría, como `trackTrace`, `trackException`, etc., por lo que tendrá que proporcionar su propio contenedor. La única API disponible es `track`. Aquí tiene un [ejemplo](https://github.com/Azure-Samples/applicationinsights-web-sample1/blob/master/testlightsku.html).

## <a name="examples"></a>Ejemplos

Para ver ejemplos ejecutables, consulte [Ejemplos del SDK de Application Insights para JavaScript](https://github.com/topics/applicationinsights-js-demo).

## <a name="upgrading-from-the-old-version-of-application-insights"></a>Actualizar desde la versión anterior de Application Insights

Cambios importantes en la versión del SDK V2:
- Para permitir mejores firmas de API, se han actualizado algunas de las llamadas API, como trackPageView o trackException. No se admite la ejecución en Internet Explorer 8 o en versiones anteriores del explorador.
- El sobre de telemetría presenta cambios en el nombre y la estructura del campo debido a actualizaciones del esquema de datos.
- `context.operation` se ha trasladado a `context.telemetryTrace`. También se han cambiado algunos campos (`operation.id` --> `telemetryTrace.traceID`).
  - Para actualizar manualmente el identificador de la vista de página actual (por ejemplo, en las aplicaciones SPA), puede hacerlo con `appInsights.properties.context.telemetryTrace.traceID = Util.generateW3CId()`.
    > [!NOTE]
    > Para mantener el identificador de seguimiento único, donde usó anteriormente `Util.newId()`, ahora use `Util.generateW3CId()`. En última instancia, terminan siendo el identificador de la operación.

Si usa el SDK de producción de Application Insights actual (1.0.20) y desea ver si el nuevo SDK funciona en tiempo de ejecución, actualice la dirección URL en función de su escenario de carga de SDK actual.

- Descarga mediante el escenario de CDN: actualice el fragmento de código que usa actualmente para que apunte a la dirección URL siguiente:
   ```
   "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js"
   ```

- Escenario de npm: llame a `downloadAndSetup` para descargar el script ApplicationInsights completo de la red CDN e inicialícelo con la clave de instrumentación:

   ```ts
   appInsights.downloadAndSetup({
     instrumentationKey: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx",
     url: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js"
     });
   ```

Pruebe en el entorno interno para comprobar que la telemetría de supervisión funciona según lo esperado. Si todo funciona, actualice las firmas de API correctamente a la versión del SDK V2 y realice la implementación en los entornos de producción.

## <a name="sdk-performanceoverhead"></a>Rendimiento y sobrecarga del SDK

Gracias a su tamaño comprimido con gzip de 36 KB y a los aproximadamente 15 ms que tarda en inicializarse, Application Insights agrega una cantidad insignificante de tiempo de carga a su sitio web. Con el fragmento de código, los componentes mínimos de la biblioteca se cargan rápidamente. Mientras tanto, el script completo se descarga en segundo plano.

Mientras el script se descarga de la red CDN, todo el seguimiento de la página se pone en cola. Una vez que se completa la inicialización asincrónica del script descargado, se realiza un seguimiento de todos los eventos que se pusieron en cola. Como resultado, no perderá telemetría durante todo el ciclo de vida de la página. Este proceso de instalación proporciona a la página un sistema de análisis sin interrupciones, invisible para los usuarios.

> Resumen:
> - ![versión de npm](https://badge.fury.io/js/%40microsoft%2Fapplicationinsights-web.svg)
> - ![tamaño comprimido con gzip](https://img.badgesize.io/https://js.monitor.azure.com/scripts/b/ai.2.min.js.svg?compression=gzip)
> - Tiempo de inicialización total de **15 ms**
> - **Cero** datos de telemetría perdidos durante el ciclo de vida de la página

## <a name="browser-support"></a>Compatibilidad con exploradores

![Chrome](https://raw.githubusercontent.com/alrra/browser-logos/master/src/chrome/chrome_48x48.png) | ![Firefox](https://raw.githubusercontent.com/alrra/browser-logos/master/src/firefox/firefox_48x48.png) | ![IE](https://raw.githubusercontent.com/alrra/browser-logos/master/src/edge/edge_48x48.png) | ![Opera](https://raw.githubusercontent.com/alrra/browser-logos/master/src/opera/opera_48x48.png) | ![Safari](https://raw.githubusercontent.com/alrra/browser-logos/master/src/safari/safari_48x48.png)
--- | --- | --- | --- | --- |
Chrome (última versión) ✔ |  Firefox (última versión) ✔ | Internet Explorer 9 (y posteriores) y Edge ✔<br>Compatible con IE 8 | Opera (última versión) ✔ | Safari (última versión) ✔ |

## <a name="es3ie8-compatibility"></a>Compatibilidad con ES3/IE8

Como un SDK, hay numerosos usuarios que no pueden controlar los exploradores que sus clientes usan. Por lo tanto, es necesario asegurarse de que este SDK continúa "funcionando" y no interrumpe la ejecución de JS cuando lo carga un explorador más antiguo. Aunque resultaría idóneo no admitir IE8 ni los exploradores de generaciones anteriores (ES3), hay numerosos usuarios y clientes de gran tamaño que siguen requiriendo que las páginas "funcionen" y, como se comentó, pueden o no controlar el explorador que sus usuarios finales deciden usar.

Esto NO significa que solamente se admita el conjunto común más bajo de características; simplemente indica que necesitamos mantener la compatibilidad con el código de ES3 y, al agregar nuevas características, deberá hacerse de manera que no interrumpan el análisis de JavaScript para ES3 y se agregarán como una característica opcional.

[Consulte GitHub para obtener todos los detalles sobre la compatibilidad con IE8](https://github.com/Microsoft/ApplicationInsights-JS#es3ie8-compatibility)

## <a name="open-source-sdk"></a>SDK de código abierto

El SDK de JavaScript de Application Insights es de código abierto. Para ver el código fuente o para contribuir al proyecto, visite el [repositorio de GitHub oficial](https://github.com/Microsoft/ApplicationInsights-JS). 

Para obtener las actualizaciones y correcciones de errores más recientes, [consulte las notas de la versión](./release-notes.md).

## <a name="next-steps"></a><a name="next"></a> Pasos siguientes
* [Seguir el uso](usage-overview.md)
* [Eventos y métricas personalizados](api-custom-events-metrics.md)
* [Compilación - Métrica - Aprendizaje](usage-overview.md)
* [Solución de problemas de errores de carga del SDK](javascript-sdk-load-failure.md)
