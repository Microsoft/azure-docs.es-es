---
title: 'Inicio rápido: supervisión de aplicaciones de Azure Spring Cloud con registros, métricas y seguimiento'
description: Use el streaming de registro, el análisis de registros, las métricas y el seguimiento para supervisar las aplicaciones de ejemplo de Piggymetrics en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/04/2020
ms.custom: devx-track-java
ms.openlocfilehash: f9f03c355e1e619d004c8ec8c1cc2f91932db744
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89046840"
---
# <a name="quickstart-monitoring-azure-spring-cloud-apps-with-logs-metrics-and-tracing"></a>Inicio rápido: supervisión de aplicaciones de Azure Spring Cloud con registros, métricas y seguimiento

Con la funcionalidad de supervisión integrada de Azure Spring Cloud, puede depurar y supervisar problemas complejos. Azure Spring Cloud integra [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) con [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) de Azure. Esta integración proporciona una eficaz funcionalidad de registros, métricas y seguimiento distribuido desde Azure Portal. En los procedimientos siguientes se explica cómo usar el streaming de registro, Log Analytics, las métricas y el seguimiento distribuido con aplicaciones de PiggyMetrics implementadas.

## <a name="prerequisites"></a>Requisitos previos

Complete los pasos anteriores: 

* [Aprovisionamiento de una instancia de Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md)
* [Configure el servidor de configuración](spring-cloud-quickstart-setup-config-server.md)
* [Compile e implemente las aplicaciones](spring-cloud-quickstart-deploy-apps.md).

## <a name="logs"></a>Registros

Hay dos maneras de ver los registros en Azure Spring Cloud: **Streaming de registros** de registros en tiempo real por instancia de aplicación o **Log Analytics** para registros agregados con la funcionalidad de consulta avanzada.

### <a name="log-streaming"></a>Secuencias de registro

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

El streaming de registro se puede usar en la CLI de Azure con el siguiente comando.

```azurecli
az spring-cloud app logs -s <service instance name> -g <resource group name> -n gateway -f
```

Verá registros como este:

[ ![Streaming de registro desde la CLI de Azure](media/spring-cloud-quickstart-logs-metrics-tracing/logs-streaming-cli.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-streaming-cli.png#lightbox)

> [!TIP]
> Use `az spring-cloud app logs -h` para explorar más parámetros y funcionalidades de las secuencias de registro.

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

Para obtener los registros mediante Azure Toolkit for IntelliJ:

1. Seleccione **Azure Explorer** (Explorador de Azure) y, luego, **Spring Cloud**.

1. Haga clic con el botón derecho en la aplicación en ejecución.

1. Seleccione **Streaming Logs** (Registros de streaming) en la lista desplegable.

   ![Selección de los registros de streaming](media/spring-cloud-intellij-howto/streaming-logs.png)
    
1. Seleccione **Instancia**.

   ![Selección de la instancia](media/spring-cloud-intellij-howto/select-instance.png)
    
1. El registro de streaming se verá en la ventana de salida.

   ![Salida del registro de streaming](media/spring-cloud-intellij-howto/streaming-log-output.png)

---
### <a name="log-analytics"></a>Log Analytics

1. Vaya a la página **servicio | Información general** y seleccione **Registros** en la sección **Supervisión**. Haga clic en **Ejecutar** en una de las consultas de ejemplo de Azure Spring Cloud. 

   [ ![Entrada de Log Analytics](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png#lightbox)
    
1. Luego verá los registros filtrados. Para más instrucciones sobre la escritura de consultas, consulte la [documentación de Azure Log Analytics](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries).

   [ ![Consulta de Log Analytics](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query.png#lightbox)

## <a name="metrics"></a>Métricas

1. Vaya a la página **servicio | Información general** y seleccione **Métricas** en la sección **Supervisión**. Agregue la primera métrica, para lo que debe seleccionar `system.cpu.usage` en **Metric** (Métrica) y `Avg` en **Aggregation** (Agregación), con el fin de ver la escala de tiempo de la utilización general de la CPU.

   [ ![Entrada de Métricas](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu.png#lightbox)
    
1. Haga clic en **Agregar filtro** en la barra de herramientas superior y seleccione `App=Gateway` para ver la utilización de la CPU solo para la aplicación **gateway**.

   [ ![Usar un filtro en métricas](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter.png#lightbox)

1. Descarte el filtro que se ha creado antes, haga clic en **Apply Splitting** (Aplicar separación) y seleccione `App` en **Values** (Valores) para ver la utilización que las diferentes aplicaciones hacen de la CPU.

   [ ![Aplicar separación en métricas](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split.png#lightbox)

## <a name="distributed-tracing"></a>Seguimiento distribuido

1. Vaya a la página **servicio | Información general** y seleccione **Seguimiento distribuido** en la sección **Supervisión**. Luego, haga clic en la pestaña **View application map** (Ver mapa de la aplicación) de la derecha.

   [ ![Entrada de Seguimiento distribuido](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png#lightbox)

1. Ya puede ver el estado de las llamadas entre las aplicaciones de Piggymetrics. 

   [ ![Información general sobre el seguimiento distribuido](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview.png#lightbox)
    
1. Haga clic en el enlace entre **gateway** y **account-service** para ver más información, como las llamadas más lentas por métodos de HTTP.

   [ ![Seguimiento distribuido](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call.png#lightbox)
    
1. Por último, haga clic en **Investigate Performance** (Investigar el rendimiento) para explorar un análisis del rendimiento integrado más eficaz.

   [ ![Rendimiento del seguimiento distribuido](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance.png#lightbox)

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos del portal, para lo que debe ejecutar el siguiente comando en Cloud Shell:

```azurecli
az group delete --name <your resource group name; for example: hellospring-1558400876966-rg> --yes
```

En los pasos anteriores, también se establece el nombre del grupo de recursos predeterminado. Para borrar ese grupo predeterminado, ejecute el siguiente comando en Cloud Shell:

```azurecli
az configure --defaults group=
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la funcionalidad de supervisión estándar de Azure Spring Cloud, consulte:

> [!div class="nextstepaction"]
> [Servicios de diagnóstico](diagnostic-services.md)
> [Seguimiento distribuido](spring-cloud-tutorial-distributed-tracing.md)
> [Registros de secuencias en tiempo real](spring-cloud-howto-log-streaming.md)
