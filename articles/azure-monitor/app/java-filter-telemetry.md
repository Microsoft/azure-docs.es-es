---
title: Filtrado de la telemetría de Azure Application Insights en la aplicación web Java
description: Reduzca el tráfico de telemetría mediante el filtrado de los eventos que no necesita supervisar.
ms.topic: conceptual
ms.date: 3/14/2019
ms.custom: devx-track-java
ms.openlocfilehash: 825c807d9af542e8776e3b6361b8f6b6dd08f164
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372185"
---
# <a name="filter-telemetry-in-your-java-web-app"></a>Filtrado de telemetría en la aplicación web de Java

Los filtros proporcionan una manera de seleccionar la telemetría que su [aplicación web de Java envía a Application Insights](java-get-started.md). Hay algunos filtros de serie que puede utilizar y también puede escribir sus propios filtros personalizados.

Los filtros de serie incluyen:

* El nivel de gravedad del seguimiento
* Direcciones URL específicas, palabras clave o códigos de respuesta
* Respuestas rápidas, es decir, las solicitudes a las que la aplicación responde rápidamente
* Nombres de eventos específicos

> [!NOTE]
> Los filtros sesgan las métricas de la aplicación. Por ejemplo, puede decidir que, para diagnosticar respuestas lentas, hay que establecer un filtro para descartar los tiempos de respuesta rápidos. Pero debe tener en cuenta que los tiempos de respuesta promedio notificados por Application Insights serán más lentos que la velocidad real y que el recuento de las solicitudes será menor que el recuento real.
> Si puede resultar un problema, use el [muestreo](./sampling.md) en su lugar.

## <a name="setting-filters"></a>Establecimiento de filtros

En ApplicationInsights.xml, agregue una sección `TelemetryProcessors` como la de este ejemplo:


```XML

    <ApplicationInsights>
      <TelemetryProcessors>

        <BuiltInProcessors>
           <Processor type="TraceTelemetryFilter">
                  <Add name="FromSeverityLevel" value="ERROR"/>
           </Processor>

           <Processor type="RequestTelemetryFilter">
                  <Add name="MinimumDurationInMS" value="100"/>
                  <Add name="NotNeededResponseCodes" value="200-400"/>
           </Processor>

           <Processor type="PageViewTelemetryFilter">
                  <Add name="DurationThresholdInMS" value="100"/>
                  <Add name="NotNeededNames" value="home,index"/>
                  <Add name="NotNeededUrls" value=".jpg,.css"/>
           </Processor>

           <Processor type="TelemetryEventFilter">
                  <!-- Names of events we don't want to see -->
                  <Add name="NotNeededNames" value="Start,Stop,Pause"/>
           </Processor>

           <!-- Exclude telemetry from availability tests and bots -->
           <Processor type="SyntheticSourceFilter">
                <!-- Optional: specify which synthetic sources,
                     comma-separated
                     - default is all synthetics -->
                <Add name="NotNeededSources" value="Application Insights Availability Monitoring,BingPreview"
           </Processor>

        </BuiltInProcessors>

        <CustomProcessors>
          <Processor type="com.fabrikam.MyFilter">
            <Add name="Successful" value="false"/>
          </Processor>
        </CustomProcessors>

      </TelemetryProcessors>
    </ApplicationInsights>

```




[Inspeccione el conjunto completo de procesadores integrados](https://github.com/Microsoft/ApplicationInsights-Java/tree/master/core/src/main/java/com/microsoft/applicationinsights/internal/processor).

## <a name="built-in-filters"></a>Filtros integrados

### <a name="metric-telemetry-filter"></a>Filtro de telemetría de métricas

```XML

           <Processor type="MetricTelemetryFilter">
                  <Add name="NotNeeded" value="metric1,metric2"/>
           </Processor>
```

* `NotNeeded`: lista de nombres de métricas personalizados separada por comas.


### <a name="page-view-telemetry-filter"></a>Filtro de telemetría de vista de página

```XML

           <Processor type="PageViewTelemetryFilter">
                  <Add name="DurationThresholdInMS" value="500"/>
                  <Add name="NotNeededNames" value="page1,page2"/>
                  <Add name="NotNeededUrls" value="url1,url2"/>
           </Processor>
```

* `DurationThresholdInMS`: la duración se refiere al tiempo dedicado a cargar la página. Si está establecido el tiempo, no se notifican las páginas que se cargan más rápido que en este momento.
* `NotNeededNames`: lista de nombres de página separados por comas.
* `NotNeededUrls`: lista de fragmentos de URL separados por comas. Por ejemplo, `"home"` filtra todas las páginas que tienen "inicio" en la dirección URL.


### <a name="request-telemetry-filter"></a>Filtro de telemetría de solicitudes


```XML

           <Processor type="RequestTelemetryFilter">
                  <Add name="MinimumDurationInMS" value="500"/>
                  <Add name="NotNeededResponseCodes" value="page1,page2"/>
                  <Add name="NotNeededUrls" value="url1,url2"/>
           </Processor>
```



### <a name="synthetic-source-filter"></a>Filtro de origen sintético

Filtra toda la telemetría con valores en la propiedad SyntheticSource. Incluye solicitudes de bots, spiders y pruebas de disponibilidad.

Filtre la telemetría para todas las solicitudes sintéticas:


```XML

           <Processor type="SyntheticSourceFilter" />
```

Filtre la telemetría para orígenes sintéticos específicos:


```XML

           <Processor type="SyntheticSourceFilter" >
                  <Add name="NotNeeded" value="source1,source2"/>
           </Processor>
```

* `NotNeeded`: Lista de nombres de origen sintético separados por comas.

### <a name="telemetry-event-filter"></a>Filtro de eventos de telemetría

Filtra los eventos personalizados (registrados mediante [TrackEvent()](./api-custom-events-metrics.md#trackevent)).


```XML

           <Processor type="TelemetryEventFilter" >
                  <Add name="NotNeededNames" value="event1, event2"/>
           </Processor>
```


* `NotNeededNames`: lista de nombres de eventos separados por comas.


### <a name="trace-telemetry-filter"></a>Filtro de telemetría de seguimiento

Filtra los seguimientos de registros (registrados mediante [TrackTrace()](./api-custom-events-metrics.md#tracktrace) o un [recopilador de plataforma de registro](java-trace-logs.md)).

```XML

           <Processor type="TraceTelemetryFilter">
                  <Add name="FromSeverityLevel" value="ERROR"/>
           </Processor>
```

* Los valores válidos de `FromSeverityLevel` son:
  *  OFF             - Filtra TODOS los seguimientos
  *  TRACE           - Sin filtrado. es igual al nivel de seguimiento
  *  INFO            - Filtra el nivel TRACE
  *  WARN            - Filtra el nivel TRACE e INFO
  *  ERROR           - Filtra WARN, INFO, TRACE
  *  CRITICAL        - Filtra todo menos CRITICAL


## <a name="custom-filters"></a>Filtros personalizados

### <a name="1-code-your-filter"></a>1. Codificación del filtro

En el código, cree una clase que implemente `TelemetryProcessor`:

```Java

    package com.fabrikam.MyFilter;
    import com.microsoft.applicationinsights.extensibility.TelemetryProcessor;
    import com.microsoft.applicationinsights.telemetry.Telemetry;

    public class SuccessFilter implements TelemetryProcessor {

       /* Any parameters that are required to support the filter.*/
       private final String successful;

       /* Initializers for the parameters, named "setParameterName" */
       public void setNotNeeded(String successful)
       {
          this.successful = successful;
       }

       /* This method is called for each item of telemetry to be sent.
          Return false to discard it.
          Return true to allow other processors to inspect it. */
       @Override
       public boolean process(Telemetry telemetry) {
        if (telemetry == null) { return true; }
        if (telemetry instanceof RequestTelemetry)
        {
            RequestTelemetry requestTelemetry = (RequestTelemetry)telemetry;
            return request.getSuccess() == successful;
        }
        return true;
       }
    }

```


### <a name="2-invoke-your-filter-in-the-configuration-file"></a>2. Invocación del filtro en el archivo de configuración

En ApplicationInsights.xml:

```XML


    <ApplicationInsights>
      <TelemetryProcessors>
        <CustomProcessors>
          <Processor type="com.fabrikam.SuccessFilter">
            <Add name="Successful" value="false"/>
          </Processor>
        </CustomProcessors>
      </TelemetryProcessors>
    </ApplicationInsights>

```

### <a name="3-invoke-your-filter-java-spring"></a>3. Invocación del filtro (Java Spring)

En el caso de las aplicaciones basadas en el marco Spring, los procesadores de telemetría personalizados deben registrarse en la clase principal de la aplicación como un bean. A continuación, cuando se inicie la aplicación, se conectarán automáticamente.

```Java
@Bean
public TelemetryProcessor successFilter() {
      return new SuccessFilter();
}
```

Deberá crear sus propios parámetros de filtro en `application.properties` y aprovechar el marco de configuración externalizado de Spring Boot para pasar los parámetros al filtro personalizado. 


## <a name="troubleshooting"></a>Solución de problemas

*El filtro no funciona.*

* Compruebe que ha proporcionado valores de parámetro válidos. Por ejemplo, las duraciones deben ser números enteros. Unos valores no válidos hará que el filtro que ignore. Si el filtro personalizado produce una excepción desde un constructor o el método set, se omitirá.

## <a name="next-steps"></a>Pasos siguientes

* [Muestreo](./sampling.md): considere la posibilidad del muestreo como una alternativa que no sesga las métricas.

