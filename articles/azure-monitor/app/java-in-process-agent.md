---
title: 'Supervisión de aplicaciones Java en cualquier entorno: Application Insights de Azure Monitor'
description: Supervisión del rendimiento de aplicaciones para aplicaciones Java que se ejecutan en cualquier entorno sin instrumentar la aplicación. Seguimiento distribuido y mapa de aplicación.
ms.topic: conceptual
ms.date: 03/29/2020
ms.openlocfilehash: e1442d1b1fb1bf8fbef82354b8aa1d2354640aa9
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87902089"
---
# <a name="java-codeless-application-monitoring-azure-monitor-application-insights---public-preview"></a>Supervisión de aplicaciones sin código de Java con Azure Monitor Application Insights: versión preliminar pública

La supervisión de aplicaciones Java sin código destaca por su simplicidad. No hace falta que realice ningún cambio en el código, ya que el agente de Java se puede habilitar mediante un par de cambios de configuración.

 El agente de Java funciona en cualquier entorno y le permite supervisar todas sus aplicaciones Java. En otras palabras, tanto si ejecuta las aplicaciones Java en máquinas virtuales, en el entorno local, en AKS, en Windows o en Linux (usted decide), el agente de Java 3.0 las supervisará.

Ya no es necesario que agregue el SDK de Java de Application Insights a la aplicación, ya que el agente 3.0 recopila solicitudes, dependencias y registros por su cuenta automáticamente.

Así mismo, puede enviar telemetría personalizada desde la aplicación. El agente 3.0 realizará un seguimiento de esta y la correlacionará con la telemetría recopilada automáticamente.

## <a name="quickstart"></a>Guía de inicio rápido

**1. Descargue el agente.**

Descargue [applicationinsights-agent-3.0.0-PREVIEW.5.jar](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.0-PREVIEW.5/applicationinsights-agent-3.0.0-PREVIEW.5.jar).

**2. Apunte JVM al agente.**

Agregue `-javaagent:path/to/applicationinsights-agent-3.0.0-PREVIEW.5.jar` a los argumentos de JVM de la aplicación.

Los argumentos típicos de JVM son `-Xmx512m` y `-XX:+UseG1GC`. Por lo tanto, si sabe dónde debe agregarlos, lo mismo se aplica para este.

Para obtener ayuda adicional con la configuración de los argumentos de JVM de la aplicación, consulte [Versión preliminar 3.0: sugerencias para actualizar los argumentos de JVM](./java-standalone-arguments.md).

**3. Apunte el agente al recurso de Application Insights.**

Si aún no tiene ningún recurso de Application Insights, puede crear uno nuevo siguiendo los pasos descritos en la [guía de creación de recursos](./create-new-resource.md).

Apunte el agente hacia el recurso de Application Insights, ya sea estableciendo una variable de entorno:

```
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=00000000-0000-0000-0000-000000000000
```

O creando un archivo de configuración denominado `ApplicationInsights.json` y colocándolo en el mismo directorio que `applicationinsights-agent-3.0.0-PREVIEW.5.jar`, con el siguiente contenido:

```json
{
  "instrumentationSettings": {
    "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000"
  }
}
```

Puede encontrar la cadena de conexión en el recurso de Application Insights:

:::image type="content" source="media/java-ipa/connection-string.png" alt-text="Cadena de conexión a Application Insights":::

**4. ¡Ya está!**

Ahora, inicie la aplicación y vaya a su recurso de Application Insights en Azure Portal para ver los datos de supervisión.

> [!NOTE]
> Los datos de supervisión pueden tardar un par de minutos en aparecer en el portal.


## <a name="configuration-options"></a>Opciones de configuración

En el archivo `ApplicationInsights.json`, también puede configurar lo siguiente:

* Nombre del rol en la nube
* Instancia de rol en la nube
* Captura del registro de aplicaciones
* Métricas JMX
* Micrómetro
* Latido
* muestreo
* Proxy HTTP
* Diagnóstico automático

Consulte los detalles en [Versión preliminar pública 3.0: opciones de configuración](./java-standalone-config.md).

## <a name="autocollected-requests-dependencies-logs-and-metrics"></a>Solicitudes, dependencias, registros y métricas recopilados automáticamente

### <a name="requests"></a>Requests

* Consumidores de JMS
* Consumidores de Kafka
* Netty/WebFlux
* Servlets
* Programación de Spring

### <a name="dependencies-with-distributed-trace-propagation"></a>Dependencias con propagación de seguimiento distribuido

* Apache HttpClient y HttpAsyncClient
* gRPC
* java.net.HttpURLConnection
* JMS
* Kafka
* Cliente Netty
* OkHttp

### <a name="other-dependencies"></a>Otras dependencias

* Cassandra
* JDBC
* MongoDB (asincrónico y sincrónico)
* Redis (Lettuce y Jedis)

### <a name="logs"></a>Registros

* java.util.logging
* Log4j
* SLF4J/Logback

### <a name="metrics"></a>Métricas

* Micrometer (incluidas las métricas del actuador de Spring Boot)
* Métricas JMX

## <a name="sending-custom-telemetry-from-your-application"></a>Envío de telemetría personalizada desde su aplicación

Nuestro objetivo con la versión 3.0 y las posteriores es permitirle enviar telemetría personalizada mediante API estándar.

Se admiten Micrometer, OpenTelemetry API y los marcos de registro conocidos. Application Insights Java 3.0 capturará automáticamente la telemetría y la correlacionará con la telemetría recopilada automáticamente.

### <a name="supported-custom-telemetry"></a>Telemetría personalizada admitida

En la tabla siguiente se representan los tipos de telemetría personalizados admitidos actualmente que se pueden habilitar para complementar el agente Java 3.0. En resumen, las métricas personalizadas se admiten a través de Micrometer, las excepciones personalizadas y los seguimientos se pueden habilitar a través de las plataformas de registro, y se admite cualquier tipo de telemetría personalizada a través del [SDK de Application Insights Java 2.x](#sending-custom-telemetry-using-application-insights-java-sdk-2x). 

|                     | Micrómetro | Log4j, logback, JUL | SDK 2.x |
|---------------------|------------|---------------------|---------|
| **Eventos personalizados**   |            |                     |  Sí    |
| **Métricas personalizadas**  |  Sí       |                     |  Sí    |
| **Dependencias**    |            |                     |  Sí    |
| **Excepciones**      |            |  Sí                |  Sí    |
| **Vistas de página**      |            |                     |  Sí    |
| **Solicitudes**        |            |                     |  Sí    |
| **Traces**          |            |  Sí                |  Sí    |

No tenemos previsto publicar ningún SDK con Application Insights 3.0 en este momento.

Application Insights Java 3.0 ya escucha la telemetría que se envía al SDK de Java de Application Insights 2.x. Esta funcionalidad es una parte importante de la historia de actualización de los usuarios existentes de 2.x y llena una brecha importante en la compatibilidad con la telemetría personalizada hasta que OpenTelemetry API ofrezca disponibilidad general.

## <a name="sending-custom-telemetry-using-application-insights-java-sdk-2x"></a>Envío de telemetría personalizada con el SDK de Java de Application Insights 2.x

Agregue `applicationinsights-core-2.6.0.jar` a la aplicación (todas las versiones 2.x son compatibles con Application Insights Java 3.0, pero merece la pena usar la más reciente si es posible):

```xml
  <dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>applicationinsights-core</artifactId>
    <version>2.6.0</version>
  </dependency>
```

Cree un elemento TelemetryClient:

  ```java
private static final TelemetryClient telemetryClient = new TelemetryClient();
```

Úselo para enviar telemetría personalizada.

### <a name="events"></a>Eventos

  ```java
telemetryClient.trackEvent("WinGame");
```
### <a name="metrics"></a>Métricas

Puede enviar telemetría de métricas a través de [Micrometer](https://micrometer.io):

```java
  Counter counter = Metrics.counter("test_counter");
  counter.increment();
```

También puede usar el SDK de Java de Application Insights 2.x:

```java
  telemetryClient.trackMetric("queueLength", 42.0);
```

### <a name="dependencies"></a>Dependencias

```java
  boolean success = false;
  long startTime = System.currentTimeMillis();
  try {
      success = dependency.call();
  } finally {
      long endTime = System.currentTimeMillis();
      RemoteDependencyTelemetry telemetry = new RemoteDependencyTelemetry();
      telemetry.setTimestamp(new Date(startTime));
      telemetry.setDuration(new Duration(endTime - startTime));
      telemetryClient.trackDependency(telemetry);
  }
```

### <a name="logs"></a>Registros
Puede enviar telemetría de registro personalizada a través de su marco de registro favorito.

También puede usar el SDK de Java de Application Insights 2.x:

```java
  telemetryClient.trackTrace(message, SeverityLevel.Warning, properties);
```

### <a name="exceptions"></a>Excepciones
Puede enviar telemetría de excepción personalizada a través de su marco de registro favorito.

También puede usar el SDK de Java de Application Insights 2.x:

```java
  try {
      ...
  } catch (Exception e) {
      telemetryClient.trackException(e);
  }
```

## <a name="upgrading-from-application-insights-java-sdk-2x"></a>Actualización del SDK de Java de Application Insights 2.x

Si ya usa el SDK de Java de Application Insights 2.x en la aplicación, no es necesario que lo quite. El agente de Java 3.0 lo detectará, y capturará y correlacionará cualquier telemetría personalizada que envíe a través del SDK de Java 2.x, al mismo tiempo que suprimirá cualquier recopilación automática que realice el SDK de Java 2.x para evitar la captura duplicada.

> [!NOTE]
> Nota: Los elementos Telemetryinitializer y TelemetryProcessors del SDK de Java 2.x no se ejecutarán cuando se use el agente 3.0.