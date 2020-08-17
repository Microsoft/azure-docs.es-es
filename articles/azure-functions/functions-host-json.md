---
title: Referencia de host.json para Azure Functions 2.x
description: Documentación de referencia para el archivo host.json de Azure Functions con el entorno en tiempo de ejecución de la versión 2.
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 629f579642185c5600586473d1280d9b26f4cba3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87055296"
---
# <a name="hostjson-reference-for-azure-functions-2x-and-later"></a>Referencia de host.json para Azure Functions 2.x y versiones posteriores 

> [!div class="op_single_selector" title1="Seleccione la versión del entorno de ejecución de Azure Functions que usa: "]
> * [Versión 1](functions-host-json-v1.md)
> * [Versión 2 y posteriores](functions-host-json.md)

El archivo de metadatos *host.json* contiene las opciones de configuración global que afectan a todas las funciones de dicha aplicación de función. En este artículo se enumeran los valores que están disponibles a partir de la versión 2.x del entorno en tiempo de ejecución de Azure Functions.  

> [!NOTE]
> Este artículo trata sobre Azure Functions 2.x y versiones posteriores.  Para obtener una referencia de host.json en Functions 1.x, consulte la [referencia de host.json para Azure Functions, versión 1.x](functions-host-json-v1.md).

Otras opciones de configuración de la aplicación de funciones se administran en la [configuración de la aplicación](functions-app-settings.md) (para aplicaciones implementadas) o en el archivo [local.settings.json](functions-run-local.md#local-settings-file) (para desarrollo local).

Las configuraciones de host.json relacionadas con los enlaces se aplican por igual a cada función de la aplicación de funciones. 

También puede [invalidar o aplicar la configuración por entorno](#override-hostjson-values) mediante la configuración de la aplicación.

## <a name="sample-hostjson-file"></a>Archivo host.json de ejemplo

El siguiente archivo *host.json* de ejemplo para la versión 2.x y posteriores tiene especificadas todas las opciones posibles (salvo aquellas que son solo para uso interno).

```json
{
    "version": "2.0",
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "extensions": {
        "cosmosDb": {},
        "durableTask": {},
        "eventHubs": {},
        "http": {},
        "queues": {},
        "sendGrid": {},
        "serviceBus": {}
    },
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[1.*, 2.0.0)"
    },
    "functions": [ "QueueProcessor", "GitHubWebHook" ],
    "functionTimeout": "00:05:00",
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    },
    "logging": {
        "fileLoggingMode": "debugOnly",
        "logLevel": {
          "Function.MyFunction": "Information",
          "default": "None"
        },
        "applicationInsights": {
            "samplingSettings": {
              "isEnabled": true,
              "maxTelemetryItemsPerSecond" : 20,
              "evaluationInterval": "01:00:00",
              "initialSamplingPercentage": 100.0, 
              "samplingPercentageIncreaseTimeout" : "00:00:01",
              "samplingPercentageDecreaseTimeout" : "00:00:01",
              "minSamplingPercentage": 0.1,
              "maxSamplingPercentage": 100.0,
              "movingAverageRatio": 1.0,
              "excludedTypes" : "Dependency;Event",
              "includedTypes" : "PageView;Trace"
            },
            "enableLiveMetrics": true,
            "enableDependencyTracking": true,
            "enablePerformanceCountersCollection": true,            
            "httpAutoCollectionOptions": {
                "enableHttpTriggerExtendedInfoCollection": true,
                "enableW3CDistributedTracing": true,
                "enableResponseHeaderInjection": true
            },
            "snapshotConfiguration": {
                "agentEndpoint": null,
                "captureSnapshotMemoryWeight": 0.5,
                "failedRequestLimit": 3,
                "handleUntrackedExceptions": true,
                "isEnabled": true,
                "isEnabledInDeveloperMode": false,
                "isEnabledWhenProfiling": true,
                "isExceptionSnappointsEnabled": false,
                "isLowPrioritySnapshotUploader": true,
                "maximumCollectionPlanSize": 50,
                "maximumSnapshotsRequired": 3,
                "problemCounterResetInterval": "24:00:00",
                "provideAnonymousTelemetry": true,
                "reconnectInterval": "00:15:00",
                "shadowCopyFolder": null,
                "shareUploaderProcess": true,
                "snapshotInLowPriorityThread": true,
                "snapshotsPerDayLimit": 30,
                "snapshotsPerTenMinutesLimit": 1,
                "tempFolder": null,
                "thresholdForSnapshotting": 1,
                "uploaderProxy": null
            }
        }
    },
    "managedDependency": {
        "enabled": true
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "watchDirectories": [ "Shared", "Test" ]
}
```

Las siguientes secciones de este artículo explican cada propiedad de nivel superior. Todas son opcionales, a menos que se indique lo contrario.

## <a name="aggregator"></a>aggregator

[!INCLUDE [aggregator](../../includes/functions-host-json-aggregator.md)]

## <a name="applicationinsights"></a>applicationInsights

Esta configuración es un elemento secundario de [logging](#logging).

Controla las opciones de Application Insights, incluidas las [opciones de muestreo](./functions-monitoring.md#configure-sampling).

Para obtener la estructura JSON completa, consulte el [archivo host.json de ejemplo](#sample-hostjson-file) anterior.

> [!NOTE]
> Los muestreos de registros pueden provocar que algunas ejecuciones no aparezcan en la hoja de supervisión de Application Insights. Para evitar el muestreo de registros, agregue `excludedTypes: "Request"` al valor `samplingSettings`.

| Propiedad | Valor predeterminado | Descripción |
| --------- | --------- | --------- | 
| samplingSettings | N/D | Consulte [applicationInsights.samplingSettings](#applicationinsightssamplingsettings). |
| enableLiveMetrics | true | Habilita la colección de Live Metrics. |
| enableDependencyTracking | true | Habilita el seguimiento de dependencias. |
| enablePerformanceCountersCollection | true | Habilita la colección de contadores de rendimiento Kudu. |
| liveMetricsInitializationDelay | 00:00:15 | Solo para uso interno. |
| httpAutoCollectionOptions | N/D | Consulte [applicationInsights.httpAutoCollectionOptions](#applicationinsightshttpautocollectionoptions). |
| snapshotConfiguration | N/D | Consulte [applicationInsights.snapshotConfiguration](#applicationinsightssnapshotconfiguration). |

### <a name="applicationinsightssamplingsettings"></a>applicationInsights.samplingSettings

|Propiedad | Valor predeterminado | Descripción |
| --------- | --------- | --------- | 
| isEnabled | true | Habilita o deshabilita el muestreo. | 
| maxTelemetryItemsPerSecond | 20 | Número de destino de los elementos de telemetría registrados por segundo en cada host de servidor. Si la aplicación se ejecuta en muchos hosts, reduzca este valor para que permanezca dentro de la tasa general de tráfico de destino. | 
| evaluationInterval | 01:00:00 | Intervalo en el que se vuelve a evaluar la velocidad actual de telemetría. La evaluación se realiza como una media móvil. Se recomienda acortar este intervalo si la telemetría experimenta ráfagas repentinas. |
| initialSamplingPercentage| 1.0 | Porcentaje de muestreo inicial aplicado al inicio del proceso de muestreo para modificar dinámicamente el porcentaje. No lo reduzca durante la depuración. |
| samplingPercentageIncreaseTimeout | 00:00:01 | Cuando cambia el valor de porcentaje de muestreo, esta propiedad determina la próxima vez que se permite que Application Insights vuelva a aumentar el porcentaje de muestreo para capturar más datos. |
| samplingPercentageDecreaseTimeout | 00:00:01 | Cuando cambia el valor de porcentaje de muestreo, esta propiedad determina la próxima vez que se permite que Application Insights vuelva a reducir el porcentaje de muestreo para capturar menos datos. |
| minSamplingPercentage | 0,1 | A medida que el porcentaje de muestreo va variando, esta propiedad determina el porcentaje de muestreo mínimo permitido. |
| maxSamplingPercentage | 0,1 | A medida que el porcentaje de muestreo va variando, esta propiedad determina el porcentaje de muestreo máximo permitido. |
| movingAverageRatio | 1.0 | En el cálculo de la media móvil, peso asignado al valor más reciente. Use un valor igual o menor que 1. Los valores menores hacen que el algoritmo reaccione con menor agilidad a los cambios repentinos. |
| excludedTypes | null | Una lista delimitada por puntos y coma de tipos que no desea que se muestreen. Los tipos reconocidos son: `Dependency`, `Event`, `Exception`, `PageView`, `Request` y `Trace`. Todas las instancias de los tipos especificados se transmiten; los tipos no especificados se muestrean. |
| includedTypes | null | Una lista delimitada por puntos y coma de tipos que desea que se muestreen (con listas vacías se muestrean todos los tipos). El tipo que figura en `excludedTypes` invalida los tipos que se enumeran aquí. Los tipos reconocidos son: `Dependency`, `Event`, `Exception`, `PageView`, `Request` y `Trace`. Las instancias de los tipos especificados se muestrean; los tipos no especificados ni implícitos se transmiten sin muestrear. |

### <a name="applicationinsightshttpautocollectionoptions"></a>applicationInsights.httpAutoCollectionOptions

|Propiedad | Valor predeterminado | Descripción |
| --------- | --------- | --------- | 
| enableHttpTriggerExtendedInfoCollection | true | Habilita o deshabilita la información de la solicitud HTTP extendida para los desencadenadores HTTP: encabezados de correlación de solicitud entrantes, compatibilidad con varias claves de instrumentación, método HTTP, ruta de acceso y respuesta. |
| enableW3CDistributedTracing | true | Habilita o deshabilita la compatibilidad del protocolo de seguimiento distribuido de W3C (y activa el esquema de correlación heredado). Está habilitado de forma predeterminada si el valor de `enableHttpTriggerExtendedInfoCollection` es true. Si `enableHttpTriggerExtendedInfoCollection` es false, esta marca se aplica solo a las solicitudes salientes, no a las entrantes. |
| enableResponseHeaderInjection | true | Habilita o deshabilita la inserción de encabezados de correlación de varios componentes en respuestas. La habilitación de la inyección permite a Application Insights construir un mapa de aplicación cuando se usan varias claves de instrumentación. Está habilitado de forma predeterminada si el valor de `enableHttpTriggerExtendedInfoCollection` es true. Esta configuración no se aplica si el valor de `enableHttpTriggerExtendedInfoCollection` es false. |

### <a name="applicationinsightssnapshotconfiguration"></a>applicationInsights.snapshotConfiguration

Para obtener más información sobre las instantáneas, vea los artículos sobre cómo [depurar instantáneas cuando se producen excepciones en aplicaciones de .NET](../azure-monitor/app/snapshot-debugger.md) y [solucionar problemas de habilitación del servicio Snapshot Debugger de Application Insights o ver instantáneas](../azure-monitor/app/snapshot-debugger-troubleshoot.md).

|Propiedad | Valor predeterminado | Descripción |
| --------- | --------- | --------- | 
| agentEndpoint | null | Punto de conexión utilizado para conectarse al servicio Snapshot Debugger de Application Insights. Si el valor es NULL, se utiliza un punto de conexión predeterminado. |
| captureSnapshotMemoryWeight | 0.5 | El peso dado al tamaño de memoria de proceso actual al comprobar si hay suficiente memoria para realizar una instantánea. El valor esperado es mayor que una fracción propia 0 (0 < CaptureSnapshotMemoryWeight < 1). |
| failedRequestLimit | 3 | Límite en el número de solicitudes con error para solicitar instantáneas antes de que se deshabilite el procesador de telemetría.|
| handleUntrackedExceptions | true | Habilita o deshabilita el seguimiento de las excepciones de las que la telemetría de Application Insights no realiza un seguimiento. |
| isEnabled | true | Habilita o deshabilita la recopilación de instantáneas. | 
| isEnabledInDeveloperMode | false | Habilita o deshabilita la recopilación de instantáneas en el modo de desarrollador. |
| isEnabledWhenProfiling | true | Habilita o deshabilita la creación de instantáneas incluso si Application Insights Profiler está recopilando una sesión de generación de perfiles detallada. |
| isExceptionSnappointsEnabled | false | Habilita o deshabilita el filtrado de excepciones. |
| isLowPrioritySnapshotUploader | true | Determina si se debe ejecutar el proceso SnapshotUploader por debajo de la prioridad normal. |
| maximumCollectionPlanSize | 50 | El número máximo de problemas de los que podemos realizar un seguimiento en cualquier momento en un intervalo de 1 a 9999. |
| maximumSnapshotsRequired | 3 | Número máximo de instantáneas recopiladas para un único problema, en un intervalo de 1 a 999. Un problema se puede considerar como una instrucción throw individual en la aplicación. Una vez que el número de instantáneas recopiladas para un problema alcanza este valor, no se recopilarán más instantáneas para ese problema hasta que se restablezcan los contadores de problemas (vea `problemCounterResetInterval`) y se vuelva a alcanzar el límite de `thresholdForSnapshotting`. |
| problemCounterResetInterval | 24:00:00 | Frecuencia con la que se restablecen los contadores de problemas en un intervalo de 1 minuto a 7 días. Cuando se alcanza este intervalo, todos los recuentos de problemas se restablecen a cero. Problemas que ya han alcanzado el umbral para realizar instantáneas, pero que aún no han generado el número de instantáneas en `maximumSnapshotsRequired`; permanecen activas. |
| provideAnonymousTelemetry | true | Determina si se va a enviar la telemetría de uso y de errores anónimos a Microsoft. Esta telemetría se puede usar si se pone en contacto con Microsoft para ayudar a solucionar problemas con Snapshot Debugger. También se usa para supervisar los patrones de uso. |
| reconnectInterval | 00:15:00 | La frecuencia con la que se vuelve a conectar al punto de conexión de Snapshot Debugger. El intervalo permitido es de 1 minuto a 1 día. |
| shadowCopyFolder | null | Especifica la carpeta que se va a usar para los archivos binarios de copias sombra. Si no se establece, se prueban las carpetas especificadas por las siguientes variables de entorno en orden: Fabric_Folder_App_Temp, LOCALAPPDATA, APPDATA, TEMP. |
| shareUploaderProcess | true | Si es true, solo una instancia de SnapshotUploader recopilará y cargará las instantáneas de varias aplicaciones que compartan InstrumentationKey. Si se establece en false, el valor de SnapshotUploader será único para cada tupla (processName, InstrumentationKey). |
| snapshotInLowPriorityThread | true | Determina si se procesan o no las instantáneas en un subproceso de prioridad de E/S inferior. La creación de una instantánea es una operación rápida, pero para cargar una instantánea en el servicio Snapshot Debugger, primero debe escribirse en el disco como un minivolcado. Eso sucede en el proceso SnapshotUploader. Al establecer este valor en true, se usa la E/S de baja prioridad para escribir el minivolcado, que no compite por recursos con la aplicación. Si se establece este valor en false, se acelera la creación del minivolcado a costa de ralentizar la aplicación. |
| snapshotsPerDayLimit | 30 | Número máximo de instantáneas permitidas en 1 día (24 horas). Este límite también se aplica en el lado del servicio de Application Insights. Las cargas se limitan a 50 por día y por aplicación (es decir, por clave de instrumentación). Este valor ayuda a evitar la creación de instantáneas adicionales que finalmente se rechazarán durante la carga. Un valor de cero quita el límite por completo, lo que no se recomienda. |
| snapshotsPerTenMinutesLimit | 1 | Número máximo de instantáneas permitidas en 10 minutos. Aunque no haya ningún límite superior en este valor, tenga cuidado al aumentarlo en las cargas de trabajo de producción, ya que podría afectar al rendimiento de la aplicación. La creación de una instantánea es rápida, pero la creación de un minivolcado de la instantánea y su carga en el servicio Snapshot Debugger es una operación mucho más lenta que compite con la aplicación por los recursos (CPU y E/S). |
| tempFolder | null | Especifica la carpeta donde se escribirán los minivolcados y los archivos de registro del cargador. Si no se establece, se usa *%TEMP%\Dumps*. |
| thresholdForSnapshotting | 1 | Número de veces que Application Insights necesita ver una excepción antes de solicitar instantáneas. |
| uploaderProxy | null | Invalida el servidor proxy usado en el proceso del cargador de instantáneas. Es posible que tenga que usar esta opción si la aplicación se conecta a Internet a través de un servidor proxy. Snapshot Collector se ejecuta en el proceso de la aplicación y usará la misma configuración de proxy. Sin embargo, el cargador de instantáneas se ejecuta como un proceso independiente y puede que tenga que configurar el servidor proxy manualmente. Si este valor es NULL, Snapshot Collector intentará detectar automáticamente la dirección del proxy examinando System.Net.WebRequest.DefaultWebProxy y pasando el valor al cargador de instantáneas. Si este valor no es NULL, no se usa la detección automática y el servidor proxy especificado aquí se usará en el cargador de instantáneas. |

## <a name="cosmosdb"></a>cosmosDb

Las opciones de configuración se pueden encontrar en los [desencadenadores y enlaces de Cosmos DB](functions-bindings-cosmosdb-v2-output.md#host-json).

## <a name="durabletask"></a>durableTask

Las opciones de configuración se puede encontrar en los [enlaces para Durable Functions](durable/durable-functions-bindings.md#host-json).

## <a name="eventhub"></a>eventHub

Las opciones de configuración se pueden encontrar en [desencadenadores y enlaces del centro de eventos](functions-bindings-event-hubs-trigger.md#host-json). 

## <a name="extensions"></a>extensions

Propiedad que devuelve un objeto que contiene todas las configuraciones específicas de enlace, como [http](#http) y [eventHub](#eventhub).

## <a name="extensionbundle"></a>extensionBundle 

Las agrupaciones de extensiones permiten agregar un conjunto compatible de extensiones de enlace de Functions a la aplicación de funciones. Para obtener más información, consulte [Agrupaciones de extensiones para el desarrollo local](functions-bindings-register.md#extension-bundles).

[!INCLUDE [functions-extension-bundles-json](../../includes/functions-extension-bundles-json.md)]

## <a name="functions"></a>functions

Lista de las funciones que el host de trabajo ejecuta. Una matriz vacía significa ejecutar todas las funciones. Su uso está previsto solo cuando se [ejecuta localmente](functions-run-local.md). En cambio, en las aplicaciones de función de Azure, siga los pasos de [Deshabilitamiento de funciones en Azure Functions](disable-function.md) para deshabilitar funciones específicas en lugar de usar esta configuración.

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

Indica la duración del tiempo de espera para todas las funciones. Sigue el formato de cadena TimeSpan. 

| Tipo de plan | Predeterminado (min) | Máximo (min) |
| -- | -- | -- |
| Consumo | 5 | 10 |
| Premium<sup>1</sup> | 30 | -1 (sin enlazar)<sup>2</sup> |
| Dedicado (App Service) | 30 | -1 (sin enlazar)<sup>2</sup> |

<sup>1</sup> La ejecución del plan Premium solo está garantizada durante 60 minutos, pero sin enlazar técnicamente.   
<sup>2</sup> Un valor de `-1` indica una ejecución sin enlazar, pero se recomienda mantener un límite superior fijo.

```json
{
    "functionTimeout": "00:05:00"
}
```

## <a name="healthmonitor"></a>healthMonitor

Configuración del [monitor de estado de host](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Host-Health-Monitor).

```
{
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    }
}
```

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------| 
|enabled|true|Especifica si está habilitada la característica. | 
|healthCheckInterval|10 segundos|El intervalo de tiempo entre las comprobaciones periódicas de mantenimiento en segundo plano. | 
|healthCheckWindow|2 minutes|Una ventana de tiempo deslizante usada en combinación con el valor `healthCheckThreshold`.| 
|healthCheckThreshold|6|Número máximo de veces que puede producirse un error en la comprobación de mantenimiento antes de que se inicie un reciclaje del host.| 
|counterThreshold|0.80|El umbral en el que un contador de rendimiento se considerará incorrecto.| 

## <a name="http"></a>http

Las opciones de configuración se pueden encontrar en los [desencadenadores y enlaces http](functions-bindings-http-webhook-output.md#hostjson-settings).

## <a name="logging"></a>logging

Controla los comportamientos de registro de la aplicación de función, Application Insights incluido.

```json
"logging": {
    "fileLoggingMode": "debugOnly",
    "logLevel": {
      "Function.MyFunction": "Information",
      "default": "None"
    },
    "console": {
        ...
    },
    "applicationInsights": {
        ...
    }
}
```

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|fileLoggingMode|debugOnly|Define qué nivel de registro de archivos está habilitado.  Las opciones son `never`, `always`, `debugOnly`. |
|logLevel|N/D|Objeto que define el filtrado por categoría de registro para las funciones de la aplicación. En las versiones 2.x y posteriores se sigue el diseño de filtrado por categoría de registro de ASP.NET Core. Esta configuración permite el filtrado del registro de funciones específicas. Para más información, consulte [Filtrado del registro](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1#log-filtering) en la documentación de ASP.NET Core. |
|console|N/D| Configuración del registro de [consola](#console). |
|applicationInsights|N/D| Configuración de [applicationInsights](#applicationinsights). |

## <a name="console"></a>console

Esta configuración es un elemento secundario de [logging](#logging). Controla el registro de la consola cuando no está en modo de depuración.

```json
{
    "logging": {
    ...
        "console": {
          "isEnabled": "false"
        },
    ...
    }
}
```

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------| 
|isEnabled|false|Habilita o deshabilita el registro de la consola.| 

## <a name="manageddependency"></a>managedDependency

La dependencia administrada es una característica en versión preliminar que actualmente solo se admite con funciones basadas en PowerShell. Permite que el servicio administre de forma automática las dependencias. Cuando la propiedad `enabled` está establecida en `true`, se procesa el archivo `requirements.psd1`. Las dependencias se actualizarán cuando se publique alguna versión secundaria. Para obtener más información, lea [Dependencia administrada](functions-reference-powershell.md#dependency-management) en el artículo de PowerShell.

```json
{
    "managedDependency": {
        "enabled": true
    }
}
```

## <a name="queues"></a>queues

Las opciones de configuración se pueden encontrar en los [desencadenadores y enlaces de la cola de Storage](functions-bindings-storage-queue-output.md#host-json).  

## <a name="sendgrid"></a>sendGrid

Las opciones de configuración se pueden encontrar en los [desencadenadores y enlaces de SendGrid](functions-bindings-sendgrid.md#host-json).

## <a name="servicebus"></a>serviceBus

Las opciones de configuración se pueden encontrar en los [desencadenadores y enlaces de Service Bus](functions-bindings-service-bus-output.md#host-json).

## <a name="singleton"></a>singleton

Opciones de configuración para el comportamiento de bloqueo Singleton. Para más información, consulte [problema de compatibilidad de GitHub con singleton](https://github.com/Azure/azure-webjobs-sdk-script/issues/912).

```json
{
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    }
}
```

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------| 
|lockPeriod|00:00:15|Período durante el cual se producen los bloqueos de nivel de función. Los bloqueos se renuevan automáticamente.| 
|listenerLockPeriod|00:01:00|Período durante el cual se producen los bloqueos de agente de escucha.| 
|listenerLockRecoveryPollingInterval|00:01:00|Intervalo de tiempo que se utiliza para la recuperación de bloqueos del agente de escucha si no se pudo adquirir un bloqueo de agente de escucha durante el inicio.| 
|lockAcquisitionTimeout|00:01:00|Cantidad máxima de tiempo que el entorno en tiempo de ejecución intentará adquirir un bloqueo.| 
|lockAcquisitionPollingInterval|N/D|Intervalo entre intentos de adquisición de bloqueo.| 

## <a name="version"></a>version

Este valor indica la versión de esquema de host.json. La cadena de versión `"version": "2.0"` es necesaria para una aplicación de funciones que tenga como destino la versión v2 o posterior del entorno de ejecución. No hay cambios en el esquema host.json entre las versiones 2 y 3.

## <a name="watchdirectories"></a>watchDirectories

Conjunto de [directorios de código compartido](functions-reference-csharp.md#watched-directories) en los que se deben supervisar los cambios.  Garantiza que cuando se cambie el código en estos directorios, las funciones recibirán los cambios.

```json
{
    "watchDirectories": [ "Shared" ]
}
```

## <a name="override-hostjson-values"></a>Invalidación de valores de host.json

Puede haber instancias en las que quiera configurar o modificar valores específicos en un archivo host.json para un entorno específico, sin cambiar el propio archivo host.json.  Puede invalidar valores específicos de host.json para crear un valor equivalente como una configuración de aplicación. Cuando el entorno de ejecución encuentra una configuración de aplicación en el formato `AzureFunctionsJobHost__path__to__setting`, invalida la configuración de host.json equivalente que se encuentra en `path.to.setting` en el archivo JSON. Cuando se expresa como una configuración de aplicación, el punto (`.`), que se utilizaba para indicar la jerarquía JSON, se reemplaza por un carácter de subrayado doble (`__`). 

Por ejemplo, suponga que desea deshabilitar el muestreo de Application Insights cuando se ejecuta localmente. Si ha cambiado el archivo host.json local para deshabilitar Application Insights, este cambio podría insertarse en la aplicación de producción durante la implementación. La manera más segura de hacerlo es crear una configuración de aplicación como `"AzureFunctionsJobHost__logging__applicationInsights__samplingSettings__isEnabled":"false"` en el archivo `local.settings.json`. Puede verlo en el siguiente archivo `local.settings.json`, que no se publica:

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "{storage-account-connection-string}",
        "FUNCTIONS_WORKER_RUNTIME": "{language-runtime}",
        "AzureFunctionsJobHost__logging__applicationInsights__samplingSettings__isEnabled":"false"
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Vea cómo actualizar el archivo host.json](functions-reference.md#fileupdate)

> [!div class="nextstepaction"]
> [Consulte las opciones de configuración global en las variables de entorno](functions-app-settings.md)
