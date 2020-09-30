---
title: Uso del seguimiento distribuido con Azure Spring Cloud
description: Aprenda a usar el seguimiento distribuido de Spring Cloud mediante Azure Application Insights
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 10/06/2019
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 97926d5bdf3123ae50714d36ad0234872f67aa96
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90908294"
---
# <a name="use-distributed-tracing-with-azure-spring-cloud"></a>Uso del seguimiento distribuido con Azure Spring Cloud

Con las herramientas de seguimiento distribuido de Azure Spring Cloud, puede depurar y supervisar fácilmente problemas complejos. Azure Spring Cloud integra [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) con [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) de Azure. Esta integración proporciona una funcionalidad eficaz de seguimiento distribuido desde Azure Portal.

::: zone pivot="programming-language-csharp"
En este artículo, aprenderá a habilitar una aplicación Steeltoe de .NET Core para usar el seguimiento distribuido.

## <a name="prerequisites"></a>Requisitos previos

Para seguir estos procedimientos, necesita una aplicación Steeltoe que ya esté [preparada para su implementación en Azure Spring Cloud](spring-cloud-tutorial-prepare-app-deployment.md).

## <a name="dependencies"></a>Dependencias

Instalación de los siguientes paquetes NuGet

* [Steeltoe.Management.TracingCore](https://www.nuget.org/packages/Steeltoe.Management.TracingCore/)
* [Steeltoe.Management.ExporterCore](https://www.nuget.org/packages/Microsoft.Azure.SpringCloud.Client/)

## <a name="update-startupcs"></a>Actualización de Startup.cs

1. En el método `ConfigureServices`, llame a los métodos `AddDistributedTracing` y `AddZipkinExporter`.

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDistributedTracing(Configuration);
       services.AddZipkinExporter(Configuration);
   }
   ```

1. En el método `Configure`, llame al método `UseTracingExporter`.

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
        app.UseTracingExporter();
   }
   ```

## <a name="update-configuration"></a>actualizar la configuración

Agregue la siguiente configuración al origen de configuración que se usará cuando la aplicación se ejecute en Azure Spring Cloud:

1. Establezca `management.tracing.alwaysSample` en true.

2. Si desea ver los intervalos de seguimiento enviados entre el servidor Eureka, el servidor de configuración y las aplicaciones de usuario: establezca `management.tracing.egressIgnorePattern` en "/api/v2/spans|/v2/apps/. */permissions|/eureka/.* |/oauth/.*".

Por ejemplo, *appsettings.json* incluiría las siguientes propiedades:
 
```json
"management": {
    "tracing": {
      "alwaysSample": true,
      "egressIgnorePattern": "/api/v2/spans|/v2/apps/.*/permissions|/eureka/.*|/oauth/.*"
    }
  }
```

Para obtener más información sobre el seguimiento distribuido en las aplicaciones de .NET Core Steeltoe, consulte el artículo de [seguimiento distribuido](https://steeltoe.io/docs/3/tracing/distributed-tracing) en la documentación de Steeltoe.
::: zone-end
::: zone pivot="programming-language-java"
En este artículo aprenderá a:

> [!div class="checklist"]
> * Habilitar el seguimiento distribuido en Azure Portal.
> * Agregar Spring Cloud Sleuth a una aplicación.
> * Ver los mapas de dependencia de las aplicaciones de microservicio.
> * Buscar datos de seguimiento con distintos filtros.

## <a name="prerequisites"></a>Requisitos previos

Para realizar estos procedimientos, necesita un servicio de Azure Spring Cloud que ya esté aprovisionado y en ejecución. Complete el inicio rápido [Implementación de la primera aplicación de Azure Spring Cloud](spring-cloud-quickstart.md) para aprovisionar y ejecutar un servicio de Azure Spring Cloud.

## <a name="add-dependencies"></a>Adición de dependencias

1. Agregue la siguiente línea al archivo application.properties:

   ```xml
   spring.zipkin.sender.type = web
   ```

   Después de este cambio, el remitente de Zipkin puede realizar envíos a la web.

1. Omita este paso si ha seguido nuestra [guía para preparar una aplicación de Azure Spring Cloud](spring-cloud-tutorial-prepare-app-deployment.md). En caso contrario, vaya al entorno de desarrollo local y edite el archivo pom.xml para incluir la siguiente dependencia de Spring Cloud Sleuth:

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-sleuth</artifactId>
                <version>${spring-cloud-sleuth.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
    </dependencies>
    ```

1. Vuelva a compilar e implementar el servicio Azure Spring Cloud para reflejar estos cambios.

## <a name="modify-the-sample-rate"></a>Modificación de la frecuencia de muestreo

Puede cambiar la frecuencia con la que se recopilan los datos de telemetría. Para ello, debe modificar la frecuencia de muestreo. Por ejemplo, si quiere muestrear la mitad tan a menudo, abra el archivo application.properties y cambie la línea siguiente:

```xml
spring.sleuth.sampler.probability=0.5
```

Si ya ha creado e implementado una aplicación, puede modificar la frecuencia de muestreo. Para ello, agregue la línea anterior como una variable de entorno en la CLI de Azure o en Azure Portal.
::: zone-end

## <a name="enable-application-insights"></a>Habilitación de Application Insights

1. Vaya a la página del servicio Azure Spring Cloud en Azure Portal.
1. En la página **Monitoring** (Supervisión), seleccione **Distributed Tracing** (Seguimiento distribuido).
1. Seleccione **Editar configuración** para editar o agregar una configuración nueva.
1. Cree una consulta de Application Insights o seleccione una existente.
1. Elija la categoría de registro que quiere supervisar y especifique el tiempo de retención en días.
1. Seleccione **Aplicar** para aplicar el nuevo seguimiento.

## <a name="view-the-application-map"></a>Visualización del mapa de aplicación

Vuelva a la página **Distributed Tracing** (Seguimiento distribuido) y seleccione **View application map** (Ver mapa de aplicación). Revise la representación visual de la configuración de la aplicación y de la supervisión. Para aprender a usar el mapa de aplicación, consulte [Mapa de aplicación: Evaluación de prioridades de las aplicaciones distribuidas](https://docs.microsoft.com/azure/azure-monitor/app/app-map).

## <a name="use-search"></a>Uso de la búsqueda

Use la función de búsqueda para consultar otros elementos de telemetría concretos. En la página **Distributed Tracing** (Seguimiento distribuido), seleccione **Buscar**. Para más información sobre cómo usar la función de búsqueda, consulte [Uso de Búsqueda en Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/diagnostic-search).

## <a name="use-application-insights"></a>Uso de Application Insights

Además del mapa de aplicación y la función de búsqueda, Application Insights proporciona funcionalidades de supervisión. Busque en Azure Portal el nombre de la aplicación y, después, abra una página de Application Insights para buscar información de supervisión. Para más información sobre cómo usar estas herramientas, consulte [Consultas de registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/query-language).

## <a name="disable-application-insights"></a>Deshabilitación de Application Insights

1. Vaya a la página del servicio Azure Spring Cloud en Azure Portal.
1. En **Monitoring** (Supervisión), seleccione **Distributed Tracing** (Seguimiento distribuido).
1. Seleccione **Disable** (Deshabilitar) para deshabilitar Application Insights.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a habilitar y comprender el seguimiento distribuido en Azure Spring Cloud. Para más información sobre los servicios de enlace a una aplicación, consulte [Enlace de una base de datos de Azure Cosmos DB a una aplicación de Azure Spring Cloud](spring-cloud-tutorial-bind-cosmos.md).
