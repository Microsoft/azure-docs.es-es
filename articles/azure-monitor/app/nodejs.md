---
title: Supervisión de servicios de Node.js con Azure Application Insights | Microsoft Docs
description: Supervise el rendimiento y diagnostique problemas en servicios de Node.js con Application Insights.
ms.topic: conceptual
ms.date: 06/01/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: c6a1a030829f128c4369e99efcd56a416390afc6
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371624"
---
# <a name="monitor-your-nodejs-services-and-apps-with-application-insights"></a>Supervisión de servicios y aplicaciones de Node.js con Application Insights

[Application Insights](./app-insights-overview.md) supervisa los componentes y los servicios back-end después de implementarlos para ayudarle a detectar y diagnosticar rápidamente problemas de rendimiento, entre otros. Puede usar Application Insights para los servicios de Node.js que se hospedan en el centro de datos, en máquinas virtuales de Azure y en aplicaciones web, e incluso en otras nubes públicas.

Para recibir, almacenar y explorar los datos de supervisión, incluya el SDK en su código y luego configure el recurso correspondiente de Application Insights en Azure. El SDK envía datos a ese recurso para realizar el análisis y la exploración posteriormente.

El SDK de Node.js puede supervisar automáticamente las solicitudes HTTP entrantes y salientes, excepciones y varias métricas del sistema. A partir de la versión 0.20, el SDK también puede supervisar algunos [paquetes de terceros](https://github.com/microsoft/node-diagnostic-channel/tree/master/src/diagnostic-channel-publishers#currently-supported-modules) comunes, como MongoDB, MySQL y Redis. Todos los eventos relacionados con una solicitud HTTP entrante se correlacionan para una solución de problemas más rápida.

Puede usar TelemetryClient API para instrumentar y supervisar manualmente aspectos adicionales de la aplicación y el sistema. TelemetryClient API se describe con más detalle más adelante en este artículo.

## <a name="get-started"></a>Introducción

Realice las tareas siguientes para configurar la supervisión para una aplicación o servicio.

### <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que tiene una suscripción de Azure o bien [obtenga una nueva de forma gratuita][azure-free-offer]. Si su organización ya tiene una suscripción de Azure, un administrador puede seguir [estas instrucciones][add-aad-user] y agregarle a ella.

[azure-free-offer]: https://azure.microsoft.com/free/
[add-aad-user]: ../../active-directory/fundamentals/add-users-azure-active-directory.md

### <a name="set-up-an-application-insights-resource"></a><a name="resource"></a> Configuración de un recurso de Application Insights

1. Inicie sesión en [Azure Portal][portal].
2. [Creación de recursos en Application Insights](create-new-resource.md)

### <a name="set-up-the-nodejs-sdk"></a><a name="sdk"></a> Configuración del SDK de Node.js

Incluya el SDK en la aplicación para que pueda recopilar datos.

1. Copie la clave de instrumentación del recurso (también conocida como *ikey*) desde su recurso recién creado. Application Insights usa el valor de ikey para asignar datos a los recursos de Azure. Para que el SDK pueda usar el valor de ikey, se debe especificar en una variable de entorno o en el código.  

   ![Copia de la clave de instrumentación](./media/nodejs/instrumentation-key-001.png)

2. Agregue la biblioteca del SDK de Node.js a las dependencias de la aplicación a través de package.json. En la carpeta raíz de la aplicación, ejecute:

   ```bash
   npm install applicationinsights --save
   ```

    > [!NOTE]
    > Si usa TypeScript, no instale paquetes "typings" independientes. El paquete NPM contiene un archivo typings integrado.

3. Cargue explícitamente la biblioteca en el código. Dado que el SDK inserta la instrumentación en muchas otras bibliotecas, cargue la biblioteca tan pronto como sea posible, incluso antes que otras instrucciones `require`.

   ```javascript
   let appInsights = require('applicationinsights');
   ```
4.  También puede proporcionar una clave ikey mediante la variable de entorno `APPINSIGHTS_INSTRUMENTATIONKEY` en lugar de pasarla manualmente a `setup()` o `new appInsights.TelemetryClient()`. Esta práctica permite conservar los valores de ikey fuera del código fuente confirmado y también para especificar distintos valores de ikey para los diferentes entornos. Para realizar la configuración manualmente, llame a `appInsights.setup('[your ikey]');`.

    Para conocer opciones de configuración adicionales, consulte las secciones siguientes.

    Puede probar el SDK sin enviar telemetría estableciendo `appInsights.defaultClient.config.disableAppInsights = true`.

5. Comience a recopilar y enviar datos automáticamente mediante una llamada a `appInsights.start();`.

### <a name="monitor-your-app"></a><a name="monitor"></a> Supervisión de la aplicación

El SDK recopila automáticamente datos de telemetría sobre el entorno de ejecución de Node.js y algunos módulos de terceros comunes. Utilice la aplicación para generar algunos datos.

A continuación, en [Azure Portal][portal] vaya al recurso de Application Insights que creó antes. En la **Escala de tiempo con información general**, busque los primeros puntos de datos. Para ver datos más detallados, seleccione diferentes componentes en los gráficos.

Para ver la topología detectada para la aplicación, puede usar el [mapa de aplicación](app-map.md).

#### <a name="no-data"></a>Sin datos

Dado que el SDK crea lotes de datos para enviar, puede haber un retraso hasta que se muestran los elementos en el portal. Si no ve los datos en el recurso, pruebe algunas de las soluciones siguientes:

* Seguir usando la aplicación. Realizar más acciones para generar más datos de telemetría.
* Haga clic en **Actualizar** en la vista de recursos del portal. Los gráficos se actualizan periódicamente por sí mismos, pero si se actualizan de forma manual, se actualizan de inmediato.
* Compruebe que los [puertos de salida necesarios](./ip-addresses.md) estén abiertos.
* Use [Buscar](./diagnostic-search.md) para buscar eventos específicos.
* Consulte las [preguntas más frecuentes][FAQ].

## <a name="basic-usage"></a>Uso básico

Para la recopilación integrada de solicitudes HTTP, eventos populares de bibliotecas de terceros, excepciones no controladas y métricas del sistema, use:

```javascript

let appInsights = require("applicationinsights");
appInsights.setup("[your ikey]").start();

```

> [!NOTE]
> Si la clave de instrumentación se establece en la variable de entorno `APPINSIGHTS_INSTRUMENTATIONKEY`, se puede llamar a `.setup()` sin argumentos. Esto facilita el uso de diferentes claves ikey para diferentes entornos.

Cargue la biblioteca de Application Insights, `require("applicationinsights")`, tan pronto como sea posible en los scripts antes de cargar otros paquetes. Esto es necesario para que la biblioteca de Application Insights pueda preparar los paquetes posteriores con fines de seguimiento. Si encuentra conflictos con otras bibliotecas que realizan una preparación similar, intente cargar la biblioteca de Application Insights después de esas otras.

Debido a la forma en la que JavaScript controla las devoluciones de llamada, se requiere trabajo adicional para realizar el seguimiento de una solicitud entre las dependencias externas y las devoluciones de llamada posteriores. De forma predeterminada, este seguimiento adicional está habilitado; para deshabilitarlo, llame a `setAutoDependencyCorrelation(false)` tal y como se describe en la sección de [configuración](#sdk-configuration) que aparece más abajo.

## <a name="migrating-from-versions-prior-to-022"></a>Migración desde versiones anteriores a la 0.22

Hay cambios importantes entre las versiones anteriores y posteriores a la 0.22. Estos cambios están diseñados para ofrecer coherencia con otros SDK de Application Insights y permitir la extensibilidad futura.

En general, puede realizar la migración de las siguientes formas:

- Reemplace las referencias a `appInsights.client` por `appInsights.defaultClient`.
- Reemplace las referencias a `appInsights.getClient()` por `new appInsights.TelemetryClient()`.
- Reemplace todos los argumentos por métodos client.track* con un único objeto que contenga propiedades con nombre como argumentos. Vea las sugerencias de tipos integrados del IDE o [TelemetryTypes](https://github.com/Microsoft/ApplicationInsights-node.js/tree/develop/Declarations/Contracts/TelemetryTypes) para el objeto de excepción para cada tipo de telemetría.

Si accede a las funciones de configuración del SDK sin encadenarlas a `appInsights.setup()`, ahora puede encontrar estas funciones en `appInsights.Configurations` (por ejemplo, `appInsights.Configuration.setAutoCollectDependencies(true)`). Revise los cambios realizados en la configuración predeterminada en la siguiente sección.

## <a name="sdk-configuration"></a>Configuración del SDK

El objeto `appInsights` proporciona varios métodos de configuración. Estos métodos se enumeran en el siguiente fragmento de código con sus valores predeterminados.

```javascript
let appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>")
    .setAutoDependencyCorrelation(true)
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true, true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .setAutoCollectConsole(true)
    .setUseDiskRetryCaching(true)
    .setSendLiveMetrics(false)
    .setDistributedTracingMode(appInsights.DistributedTracingModes.AI)
    .start();
```

Para correlacionar totalmente los eventos en un servicio, asegúrese de establecer `.setAutoDependencyCorrelation(true)`. Con esta opción establecida, el SDK puede realizar el seguimiento del contexto a través de las devoluciones de llamada asincrónicas en Node.js.

Revise las descripciones en las sugerencias de tipos integrados del IDE, o bien consulte [applicationinsights.ts](https://github.com/microsoft/ApplicationInsights-node.js/blob/develop/applicationinsights.ts) para obtener información detallada sobre qué controlan y sobre los argumentos secundarios opcionales.

> [!NOTE]
>  De forma predeterminada, `setAutoCollectConsole` está configurado para *excluir* las llamadas a `console.log` (y a otros métodos de consola). Solo se recopilarán las llamadas a registradores de terceros admitidos (por ejemplo, Winston y Bunyan). Puede cambiar este comportamiento para incluir las llamadas a métodos `console` mediante `setAutoCollectConsole(true, true)`.

### <a name="sampling"></a>muestreo

De forma predeterminada, el SDK enviará todos los datos recopilados al servicio de Application Insights. Si recopila una gran cantidad de datos, es posible que quiera habilitar el muestreo para reducir la cantidad de datos enviados. Para ello, especifique el campo `samplingPercentage` en el objeto `config` de un cliente. Si establece `samplingPercentage` en 100 (el valor predeterminado), se enviarán todos los datos, y si lo establece en 0, no se enviará nada.

Si usa la correlación automática, todos los datos asociados a una única solicitud se incluirán o excluirán como una unidad.

Agregue el siguiente fragmento de código para habilitar el muestreo:

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>");
appInsights.defaultClient.config.samplingPercentage = 33; // 33% of all telemetry will be sent to Application Insights
appInsights.start();
```

### <a name="multiple-roles-for-multi-components-applications"></a>Varios roles para aplicaciones de varios componentes

Si la aplicación consta de varios componentes que quiere instrumentar con la misma clave de instrumentación, pero quiere seguir viendo estos componentes como unidades independientes en el portal, como si usaran claves de instrumentación independientes (por ejemplo, como nodos independientes en el mapa de aplicación), puede que necesite configurar manualmente el campo RoleName para distinguir la telemetría de un componente de otros componentes que envían datos a su recurso de Application Insights.

Use el siguiente código para configurar el campo RoleName:

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>");
appInsights.defaultClient.context.tags[appInsights.defaultClient.context.keys.cloudRole] = "MyRoleName";
appInsights.start();
```

### <a name="automatic-third-party-instrumentation"></a>Instrumentación de terceros automática

Para realizar un seguimiento del contexto a través de llamadas asincrónicas, es necesario hacer algunos cambios en las bibliotecas de terceros, como MongoDB y Redis. De forma predeterminada, Application Insights usará [`diagnostic-channel-publishers`](https://github.com/Microsoft/node-diagnostic-channel/tree/master/src/diagnostic-channel-publishers) para modificar de forma dinámica algunas de estas bibliotecas. Se puede deshabilitar si se establece la variable de entorno `APPLICATION_INSIGHTS_NO_DIAGNOSTIC_CHANNEL`.

> [!NOTE]
> Al establecer esa variable de entorno, es posible que los eventos dejen de estar asociados correctamente a la operación adecuada.

 Las modificaciones dinámicas individuales se pueden deshabilitar si se establece la variable de entorno `APPLICATION_INSIGHTS_NO_PATCH_MODULES` en una lista de paquetes separados por comas, por ejemplo, `APPLICATION_INSIGHTS_NO_PATCH_MODULES=console,redis` para evitar la modificación de los paquetes `console` y `redis`.

Actualmente hay nueve paquetes instrumentados: `bunyan`, `console`, `mongodb`, `mongodb-core`, `mysql`, `redis`, `winston`, `pg` y `pg-pool`. Consulte el [archivo Léame de diagnostic-channel-publishers](https://github.com/Microsoft/node-diagnostic-channel/blob/master/src/diagnostic-channel-publishers/README.md) para obtener información sobre la versión revisada de estos paquetes.

Las revisiones de `bunyan`, `winston` y `console` generarán eventos de seguimiento de Application Insights en función de si `setAutoCollectConsole` está habilitado. El resto generará eventos de dependencia de Application Insights en función de si `setAutoCollectDependencies` está habilitado.

### <a name="live-metrics"></a>Live Metrics

Para habilitar el envío de métricas en tiempo real desde la aplicación a Azure, use `setSendLiveMetrics(true)`. Actualmente no se admite el filtrado de métricas en tiempo real en el portal.

### <a name="extended-metrics"></a>Métricas extendidas

> [!NOTE]
> La capacidad de enviar métricas nativas extendidas se agregó en la versión 1.4.0.

Para habilitar el envío de métricas nativas extendidas desde su aplicación a Azure, instale el paquete de métricas nativas independiente. El SDK se cargará automáticamente cuando se instale y empezará a recopilar métricas nativas de Node.js.

```bash
npm install applicationinsights-native-metrics
```

Actualmente, el paquete de métricas nativas recopila automáticamente tiempo de CPU de recolección de elementos no utilizados, tics de bucle de eventos e información de uso de montones:

- **Recolección de elementos no utilizados**: cantidad de tiempo de CPU que se dedica a cada tipo de recolección de elementos no utilizados y número de repeticiones de cada tipo.
- **Bucle de eventos**: número de tics que se han producido y tiempo de CPU que se ha dedicado en total.
- **Montón frente a no montón**: cantidad de uso de memoria de la aplicación se dedica o no al montón.

### <a name="distributed-tracing-modes"></a>Modos de seguimiento distribuido

De forma predeterminada, el SDK enviará los encabezados que entienden otras aplicaciones o servicios instrumentados con un SDK de Application Insights. Opcionalmente, puede habilitar el envío y la recepción de encabezados de [contexto de seguimiento de W3C](https://github.com/w3c/trace-context), además de los encabezados de IA existentes, por lo que no se interrumpirá la correlación con ninguno de los servicios heredados existentes. La habilitación de encabezados de W3C permitirá que la aplicación se correlacione con otros servicios no instrumentados con Application Insights, pero que adopte este estándar de W3C.

```Javascript
const appInsights = require("applicationinsights");
appInsights
  .setup("<your ikey>")
  .setDistributedTracingMode(appInsights.DistributedTracingModes.AI_AND_W3C)
  .start()
```

## <a name="telemetryclient-api"></a>TelemetryClient API

Para obtener una descripción completa de TelemetryClient API, consulte [API de Application Insights para eventos y métricas personalizados](./api-custom-events-metrics.md).

Puede realizar el seguimiento de cualquier solicitud, evento, métrica o excepción mediante el SDK de Node.js de Application Insights. El ejemplo de código siguiente muestra algunas de las API que puede usar:

```javascript
let appInsights = require("applicationinsights");
appInsights.setup().start(); // assuming ikey in env var. start() can be omitted to disable any non-custom data
let client = appInsights.defaultClient;
client.trackEvent({name: "my custom event", properties: {customProperty: "custom property value"}});
client.trackException({exception: new Error("handled exceptions can be logged with this method")});
client.trackMetric({name: "custom metric", value: 3});
client.trackTrace({message: "trace message"});
client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL"});
client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});

let http = require("http");
http.createServer( (req, res) => {
  client.trackNodeHttpRequest({request: req, response: res}); // Place at the beginning of your request handler
});
```

### <a name="track-your-dependencies"></a>Seguimiento de las dependencias

Utilice el código siguiente para realizar el seguimiento de las dependencias:

```javascript
let appInsights = require("applicationinsights");
let client = new appInsights.TelemetryClient();

var success = false;
let startTime = Date.now();
// execute dependency call here....
let duration = Date.now() - startTime;
success = true;

client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:duration, resultCode:0, success: true, dependencyTypeName: "ZSQL"});;
```

A continuación se muestra una utilidad de ejemplo que usa `trackMetric` para medir el tiempo que tarda la programación del bucle de eventos:  

```javascript
function startMeasuringEventLoop() {
  var startTime = process.hrtime();
  var sampleSum = 0;
  var sampleCount = 0;

  // Measure event loop scheduling delay
  setInterval(() => {
    var elapsed = process.hrtime(startTime);
    startTime = process.hrtime();
    sampleSum += elapsed[0] * 1e9 + elapsed[1];
    sampleCount++;
  }, 0);

  // Report custom metric every second
  setInterval(() => {
    var samples = sampleSum;
    var count = sampleCount;
    sampleSum = 0;
    sampleCount = 0;

    if (count > 0) {
      var avgNs = samples / count;
      var avgMs = Math.round(avgNs / 1e6);
      client.trackMetric({name: "Event Loop Delay", value: avgMs});
    }
  }, 1000);
}
```

### <a name="add-a-custom-property-to-all-events"></a>Incorporación de una propiedad personalizada a todos los eventos

Utilice el código siguiente para agregar una propiedad personalizada a todos los eventos:

```javascript
appInsights.defaultClient.commonProperties = {
  environment: process.env.SOME_ENV_VARIABLE
};
```

### <a name="track-http-get-requests"></a>Seguimiento de solicitudes HTTP GET

Use el código siguiente para realizar un seguimiento manual de las solicitudes HTTP GET:

> [!NOTE]
> Se hace un seguimiento de todas las solicitudes de forma predeterminada. Para deshabilitar la recopilación automática, llame a .setAutoCollectRequests(false) antes de llamar a start().

```javascript
appInsights.defaultClient.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});
```

También puede realizar un seguimiento de las solicitudes mediante el método `trackNodeHttpRequest`:

```javascript
var server = http.createServer((req, res) => {
  if ( req.method === "GET" ) {
      appInsights.defaultClient.trackNodeHttpRequest({request:req, response:res});
  }
  // other work here....
  res.end();
});
```

### <a name="track-server-startup-time"></a>Seguimiento de la hora de inicio del servidor

Utilice el código siguiente para realizar un seguimiento de la hora de inicio del servidor:

```javascript
let start = Date.now();
server.on("listening", () => {
  let duration = Date.now() - start;
  appInsights.defaultClient.trackMetric({name: "server startup time", value: duration});
});
```

### <a name="preprocess-data-with-telemetry-processors"></a>Preprocesamiento de datos con procesadores de telemetría

Puede procesar y filtrar los datos recopilados antes de enviarlos para su retención mediante el uso de *procesadores de telemetría*. Los procesadores de telemetría se llaman de uno en uno en el orden en el que se han agregado antes de que el elemento de telemetría se envíe a la nube.

```javascript
public addTelemetryProcessor(telemetryProcessor: (envelope: Contracts.Envelope, context: { http.RequestOptions, http.ClientRequest, http.ClientResponse, correlationContext }) => boolean)
```

Si un procesador de telemetría devuelve false, no se enviará el elemento de telemetría.

Todos los procesadores de telemetría reciben los datos de telemetría y su sobre para inspeccionarlos y modificarlos. También reciben un objeto de contexto. En el caso de la telemetría de seguimiento manual, el contenido de este objeto lo define el parámetro `contextObjects` cuando se llama a un método de seguimiento. Para la telemetría recopilada automáticamente, este objeto se llena con la información de solicitud disponible y el contenido de la solicitud persistente, tal como lo proporciona `appInsights.getCorrelationContext()` (si está habilitada la correlación automática de dependencias).

El tipo de TypeScript de un procesador de telemetría es:

```javascript
telemetryProcessor: (envelope: ContractsModule.Contracts.Envelope, context: { http.RequestOptions, http.ClientRequest, http.ClientResponse, correlationContext }) => boolean;
```

Por ejemplo, un procesador que quita los datos de seguimiento de pilas de las excepciones podría escribirse y agregarse como se indica a continuación:

```javascript
function removeStackTraces ( envelope, context ) {
  if (envelope.data.baseType === "Microsoft.ApplicationInsights.ExceptionData") {
    var data = envelope.data.baseData;
    if (data.exceptions && data.exceptions.length > 0) {
      for (var i = 0; i < data.exceptions.length; i++) {
        var exception = data.exceptions[i];
        exception.parsedStack = null;
        exception.hasFullStack = false;
      }
    }
  }
  return true;
}

appInsights.defaultClient.addTelemetryProcessor(removeStackTraces);
```

## <a name="use-multiple-instrumentation-keys"></a>Uso de múltiples claves de instrumentación

Puede crear varios recursos de Application Insights y enviar distintos datos a cada uno de ellos con sus respectivas claves de instrumentación ("ikey").

 Por ejemplo:

```javascript
let appInsights = require("applicationinsights");

// configure auto-collection under one ikey
appInsights.setup("_ikey-A_").start();

// track some events manually under another ikey
let otherClient = new appInsights.TelemetryClient("_ikey-B_");
otherClient.trackEvent({name: "my custom event"});
```

## <a name="advanced-configuration-options"></a>Opciones de configuración avanzada

El objeto de cliente contiene una propiedad `config` con muchos valores de configuración opcionales para escenarios avanzados. Estos valores pueden establecerse de la siguiente manera:

```javascript
client.config.PROPERTYNAME = VALUE;
```

Las siguientes propiedades son específicas del cliente, por lo que puede configurar `appInsights.defaultClient` de forma independiente con respecto a los clientes creados con `new appInsights.TelemetryClient()`.

| Propiedad                        | Descripción                                                                                                |
| ------------------------------- |------------------------------------------------------------------------------------------------------------|
| instrumentationKey              | Identificador para el recurso de Application Insights.                                                      |
| endpointUrl                     | Punto de conexión de ingesta al que se van a enviar las cargas de telemetría.                                                      |
| quickPulseHost                  | Host de Live Metrics Stream al que se va a enviar la telemetría de métricas en tiempo real.                                            |
| proxyHttpUrl                    | Servidor proxy para el tráfico HTTP del SDK (opcional, el valor predeterminado se extrae de la variable de entorno `http_proxy`).     |
| proxyHttpsUrl                   | Servidor proxy para el tráfico HTTPS del SDK (opcional, el valor predeterminado se extrae de la variable de entorno `https_proxy`).   |
| httpAgent                       | Agente http.Agent que se usará para el tráfico HTTP del SDK (opcional, valor predeterminado sin definir).                                   |
| httpsAgent                      | Agente https.Agent que se usará para el tráfico HTTPS del SDK (opcional, valor predeterminado sin definir).                                 |
| maxBatchSize                    | Número máximo de elementos de telemetría que se incluirán en una carga en el punto de conexión de ingesta (el valor predeterminado es `250`).   |
| maxBatchIntervalMs              | Cantidad máxima de tiempo que hay que esperar para que una carga alcance el tamaño máximo de lote, maxBatchSize (el valor predeterminado es `15000`).               |
| disableAppInsights              | Marca que indica si la transmisión de telemetría está deshabilitada (el valor predeterminado es `false`).                                 |
| samplingPercentage              | Porcentaje de elementos de telemetría de los que se realiza un seguimiento y que se deben transmitir (el valor predeterminado es `100`).                      |
| correlationIdRetryIntervalMs    | Tiempo de espera antes de volver a intentar recuperar el identificador para la correlación entre componentes (el valor predeterminado es `30000`).     |
| correlationHeaderExcludedDomains| Lista de dominios que se excluirán de la inyección de encabezados de correlación entre componentes (para ver el valor predeterminado, consulte [Config.ts](https://github.com/Microsoft/ApplicationInsights-node.js/blob/develop/Library/Config.ts)).|

## <a name="next-steps"></a>Pasos siguientes

* [Navegación y paneles en el portal de Application Insights](./overview-dashboard.md)
* [Escritura de consultas de Analytics sobre los datos de telemetría](../log-query/get-started-portal.md)

<!--references-->

[portal]: https://portal.azure.com/
[FAQ]: ../faq.md

