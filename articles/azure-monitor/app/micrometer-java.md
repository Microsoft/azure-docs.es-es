---
title: Cómo usar Micrometer con el SDK de Azure Application Insights para Java
description: Guía paso a paso sobre cómo usar Micrometer con las aplicaciones Spring Boot y no Spring Boot de Application Insights.
ms.topic: conceptual
author: lgayhardt
ms.custom: devx-track-java
ms.author: lagayhar
ms.date: 11/01/2018
ms.openlocfilehash: 2fbdf4e267e0f9a479a89d6f31e72f09d9e11bb0
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322574"
---
# <a name="how-to-use-micrometer-with-azure-application-insights-java-sdk"></a>Cómo usar Micrometer con el SDK de Azure Application Insights para Java
La aplicación Micrometer supervisa las métricas de medidas para código de aplicación basado en JVM y le permite exportar los datos a los sistemas de supervisión que prefiera. Este artículo le mostrará cómo usar Micrometer con Application Insights tanto para las aplicaciones Spring Boot como para las que no sean Spring Boot.

## <a name="using-spring-boot-15x"></a>Uso de Spring Boot 1.5x
Agregue las siguientes dependencias al archivo pom.xml o al archivo build.gradle: 
* [Application Insights spring-boot-starter](https://github.com/Microsoft/ApplicationInsights-Java/tree/master/azure-application-insights-spring-boot-starter) 2.5.0 o posterior
* Micrometer Azure Registry 1.1.0 o posterior
* [Micrometer Spring Legacy](https://micrometer.io/docs/ref/spring/1.5) 1.1.0 o posterior (modifica el código de configuración automática en la plataforma de Spring).
* [Recurso ApplicationInsights](./create-new-resource.md)

Pasos

1. Actualice el archivo pom.xml de la aplicación Spring Boot y agréguele las siguientes dependencias:

    ```XML
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-spring-boot-starter</artifactId>
        <version>2.5.0</version>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-spring-legacy</artifactId>
        <version>1.1.0</version>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-azure-monitor</artifactId>
        <version>1.1.0</version>
    </dependency>

    ```
2. Actualice el archivo application.properties o yml con la clave de Application Insights Instrumentation mediante la siguiente propiedad:

     `azure.application-insights.instrumentation-key=<your-instrumentation-key-here>`
1. Compile la aplicación y ejecútela.
2. Las opciones anteriores deberían conseguir que empezara a ejecutar las métricas agregadas previamente y recopiladas de forma automática para Azure Monitor. Para más información sobre cómo ajustar el iniciador de Spring Boot de Application Insights, consulte el archivo [Léame en GitHub](https://github.com/Microsoft/ApplicationInsights-Java/blob/master/azure-application-insights-spring-boot-starter/README.md).

## <a name="using-spring-2x"></a>Uso de Spring 2.x

Agregue las siguientes dependencias al archivo pom.xml o al archivo build.gradle:

* Application Insights Spring-boot-starter 2.1.2 o posterior
* Azure-spring-boot-metrics-starters 2.0.7 o posterior
* [Recurso de Application Insights](./create-new-resource.md)

Pasos:

1. Actualice el archivo pom.xml de la aplicación de Spring Boot y agréguele la siguiente dependencia:

    ```XML
    <dependency> 
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-spring-boot-metrics-starter</artifactId>
          <version>2.0.7</version>
    </dependency>
    ```
1. Actualice el archivo application.properties o yml con la clave de Application Insights Instrumentation mediante la siguiente propiedad:

     `azure.application-insights.instrumentation-key=<your-instrumentation-key-here>`
3. Compile la aplicación y ejecútela.
4. Las opciones anteriores deberían conseguir que empezara a ejecutar las métricas agregadas previamente y recopiladas de forma automática para Azure Monitor. Para más información sobre cómo ajustar el iniciador de Spring Boot de Application Insights, consulte el archivo [Léame en GitHub](https://github.com/Microsoft/azure-spring-boot/releases/latest).

Métricas predeterminadas:

*    Las métricas se configuran automáticamente para Tomcat, JVM, métricas de Logback, métricas de Log4J, métricas de tiempo de actividad, métricas de procesador y FileDescriptorMetrics.
*    Por ejemplo, si Netflix Hystrix está presente en la ruta de acceso de la clase, también obtenemos esas métricas. 
*    Las siguientes métricas pueden estar disponibles al agregar los beans (o componentes software) respectivos. 
        - CacheMetrics (CaffeineCache, EhCache2, GuavaCache, HazelcastCache y JCache)     
        - DataBaseTableMetrics 
        - HibernateMetrics 
        - JettyMetrics 
        - Métricas de OkHttp3 
        - Métricas de Kafka 

 

Cómo desactivar la recopilación automática de las métricas: 
 
- Métricas de JVM: 
    - Management.Metrics.binders.jvm.Enabled=False 
- Métricas de Logback: 
    - management.metrics.binders.logback.enabled=false
- Métricas de tiempo de actividad: 
    - management.metrics.binders.uptime.enabled=false 
- Métricas de procesador:
    -  management.metrics.binders.processor.enabled=false 
- FileDescriptorMetrics:
    - management.metrics.binders.files.enabled=false 
- Métricas de Hystrix si la biblioteca está en la ruta de clase: 
    - management.metrics.binders.hystrix.enabled=false 
- Métricas de AspectJ si la biblioteca está en la ruta de clase: 
    - spring.aop.enabled=false 

> [!NOTE]
> Especifique las propiedades anteriores en el archivo application.properties o application.yml de la aplicación Spring Boot

## <a name="use-micrometer-with-non-spring-boot-web-applications"></a>Uso de Micrometer con aplicaciones web que no son Spring Boot

Agregue las siguientes dependencias al archivo pom.xml o al archivo build.gradle:

* Application Insights Web Auto 2.5.0 o posterior
* Micrometer Azure Registry 1.1.0 o posterior
* [Recurso de Application Insights](./create-new-resource.md)

Pasos:

1. Agregue las siguientes dependencias en el archivo pom.xml o build.gradle:

    ```XML
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-azure-monitor</artifactId>
            <version>1.1.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>applicationinsights-web-auto</artifactId>
            <version>2.5.0</version>
        </dependency>
     ```

2. Coloque el archivo `ApplicationInsights.xml` en la carpeta de recursos:

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">
    
       <!-- The key from the portal: -->
       <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>
    
       <!-- HTTP request component (not required for bare API) -->
       <TelemetryModules>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
       </TelemetryModules>
    
       <!-- Events correlation (not required for bare API) -->
       <!-- These initializers add context data to each event -->
       <TelemetryInitializers>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
       </TelemetryInitializers>
    
    </ApplicationInsights>
    ```

3. Clase de Servlet de ejemplo (emite una métrica de temporizador):

    ```Java
        @WebServlet("/hello")
        public class TimedDemo extends HttpServlet {
    
          private static final long serialVersionUID = -4751096228274971485L;
    
          @Override
          @Timed(value = "hello.world")
          protected void doGet(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {
    
            response.getWriter().println("Hello World!");
            MeterRegistry registry = (MeterRegistry) getServletContext().getAttribute("AzureMonitorMeterRegistry");
    
        //create new Timer metric
            Timer sampleTimer = registry.timer("timer");
            Stream<Integer> infiniteStream = Stream.iterate(0, i -> i+1);
            infiniteStream.limit(10).forEach(integer -> {
              try {
                Thread.sleep(1000);
                sampleTimer.record(integer, TimeUnit.MILLISECONDS);
              } catch (Exception e) {}
               });
          }
          @Override
          public void init() throws ServletException {
            System.out.println("Servlet " + this.getServletName() + " has started");
          }
          @Override
          public void destroy() {
            System.out.println("Servlet " + this.getServletName() + " has stopped");
          }
    
        }
    
    ```

4. Clase de configuración de ejemplo:

    ```Java
         @WebListener
         public class MeterRegistryConfiguration implements ServletContextListener {
     
           @Override
           public void contextInitialized(ServletContextEvent servletContextEvent) {
     
         // Create AzureMonitorMeterRegistry
           private final AzureMonitorConfig config = new AzureMonitorConfig() {
             @Override
             public String get(String key) {
                 return null;
             }
            @Override
               public Duration step() {
                 return Duration.ofSeconds(60);}
     
             @Override
             public boolean enabled() {
                 return false;
             }
         };
     
      MeterRegistry azureMeterRegistry = AzureMonitorMeterRegistry.builder(config);
     
             //set the config to be used elsewhere
             servletContextEvent.getServletContext().setAttribute("AzureMonitorMeterRegistry", azureMeterRegistry);
     
           }
     
           @Override
           public void contextDestroyed(ServletContextEvent servletContextEvent) {
     
           }
         }
    ```

Para más información acerca de las métricas, consulte la [documentación de Micrometer](https://micrometer.io/docs/).

Otro ejemplo de código sobre cómo crear distintos tipos de métricas puede encontrarse en [el repositorio de GitHub de Micrometer oficial](https://github.com/micrometer-metrics/micrometer/tree/master/samples/micrometer-samples-core/src/main/java/io/micrometer/core/samples).

## <a name="how-to-bind-additional-metrics-collection"></a>Cómo enlazar la recopilación de métricas adicionales

### <a name="springbootspring"></a>SpringBoot/Spring

Cree un bean de la categoría de métrica respectiva. Por ejemplo, supongamos que necesitamos las métricas de caché de Guava:

```Java
    @Bean
    GuavaCacheMetrics guavaCacheMetrics() {
        Return new GuavaCacheMetrics();
    }
```
Hay varias que no están habilitadas de forma predeterminada, pero se pueden enlazar del modo anterior. Para obtener una lista completa, consulte [el repositorio de GitHub de Micrometer oficial](https://github.com/micrometer-metrics/micrometer/tree/master/micrometer-core/src/main/java/io/micrometer/core/instrument/binder ).

### <a name="non-spring-apps"></a>Aplicaciones que no sean Spring
Agregue el siguiente código de enlace al archivo de configuración:
```Java 
    New GuavaCacheMetrics().bind(registry);
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre Micrometer, vea la [documentación de Micrometer](https://micrometer.io/docs) oficial.
* Para obtener información sobre Spring en Azure, vea la [documentación de Spring en Azure](/java/azure/spring-framework/?view=azure-java-stable) oficial.

