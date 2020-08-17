---
title: Análisis de registros y métricas en Azure Spring Cloud | Microsoft Docs
description: Aprenda a analizar datos de diagnóstico en Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 01/06/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 56f392210aac6045a9dc8cc3522d36092162f26c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086122"
---
# <a name="analyze-logs-and-metrics-with-diagnostics-settings"></a>Análisis de registros y métricas con la configuración de diagnóstico

Mediante la funcionalidad de diagnóstico de Azure Spring Cloud puede analizar los registros y las métricas con uno de los siguientes servicios:

* Use Azure Log Analytics, donde los datos se escriben en Azure Storage. Se produce un retraso al exportar registros a Log Analytics.
* Guarde los registros en una cuenta de almacenamiento para auditarlos o para inspeccionarlos manualmente. Puede especificar el tiempo de retención (en días).
* Transmita los registros al centro de eventos para la ingesta en un servicio de terceros o una solución de análisis personalizada.

Elija la categoría de registro y la categoría de métrica que desea supervisar.

> [!TIP]
> ¿Solo desea transmitir en secuencias los registros? Consulte este [comando de la CLI de Azure](https://docs.microsoft.com/cli/azure/ext/spring-cloud/spring-cloud/app?view=azure-cli-latest#ext-spring-cloud-az-spring-cloud-app-logs).

## <a name="logs"></a>Registros

|Log | Descripción |
|----|----|
| **ApplicationConsole** | Registro de la consola de todas las aplicaciones del cliente. |
| **SystemLogs** | Actualmente, solo el [servidor de configuración de Spring Cloud](https://cloud.spring.io/spring-cloud-config/reference/html/#_spring_cloud_config_server) mantiene registros en esta categoría. |

## <a name="metrics"></a>Métricas

Para obtener una lista completa de las métricas, consulte la sección [Métricas de Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-concept-metrics#user-metrics-options).

Para empezar, habilite uno de estos servicios para recibir los datos. Para más información sobre la configuración de Log Analytics, consulte [Introducción a los análisis de registros de Azure Monitor](../azure-monitor/log-query/get-started-portal.md).

## <a name="configure-diagnostics-settings"></a>Configuración de valores de diagnóstico

1. En Azure Portal, vaya a la instancia de Azure Spring Cloud.
1. Seleccione la opción **Configuración de diagnóstico** y después, **Agregar configuración de diagnóstico**.
1. Escriba un nombre para la configuración y elija dónde desea enviar los registros. También puede seleccionar cualquier combinación de las tres opciones siguientes:
    * **Archivar en una cuenta de almacenamiento**
    * **Transmisión a un centro de eventos**
    * **Enviar a Log Analytics**

1. Elija la categoría de registro y de métrica que desea supervisar y especifique el tiempo de retención (en días). El tiempo de retención solo se aplica a la cuenta de almacenamiento.
1. Seleccione **Guardar**.

> [!NOTE]
> 1. Puede transcurrir un intervalo de hasta 15 minutos desde que se emiten los registros o las métricas hasta que aparecen en la cuenta de almacenamiento, el centro de eventos o Log Analytics.
> 1. Si se elimina o se mueve la instancia de Azure Spring Cloud, la operación no se realizará en cascada a los recursos de la **configuración de diagnósticos**. Los recursos de la **administración de diagnósticos** se deben eliminar manualmente antes de la operación en su elemento primario, es decir, la instancia de Azure Spring Cloud. De lo contrario, si se aprovisiona una nueva instancia de Azure Spring Cloud con el mismo id. de recurso que la eliminada o si la instancia de Azure Spring Cloud se vuelve a trasladar, los recursos de la **configuración de diagnósticos** anteriores siguen extendiéndola.

## <a name="view-the-logs-and-metrics"></a>Visualización de registros y métricas
Existen varios métodos para ver los registros y las métricas, tal y como se describe en los apartados siguientes.

### <a name="use-the-logs-blade"></a>Uso de la hoja Registros

1. En Azure Portal, vaya a la instancia de Azure Spring Cloud.
1. Para abrir el panel **Búsqueda de registros**, seleccione **Registros**.
1. En el cuadro de texto **Tablas**
   * Para ver los registros, escriba una consulta simple como la siguiente:

    ```sql
    AppPlatformLogsforSpring
    | limit 50
    ```
   * Para ver las métricas, escriba una consulta simple como la siguiente:

    ```sql
    AzureMetrics
    | limit 50
    ```
1. Para ver el resultado de la búsqueda, seleccione **Ejecutar**.

### <a name="use-log-analytics"></a>Uso de Log Analytics

1. En Azure Portal, en el panel izquierdo, seleccione **Log Analytics**.
1. Seleccione el área de trabajo de Log Analytics que eligió al agregar la configuración de diagnóstico.
1. Para abrir el panel **Búsqueda de registros**, seleccione **Registros**.
1. En el cuadro de texto **Tablas**,
   * Para ver los registros, escriba una consulta simple como la siguiente:

    ```sql
    AppPlatformLogsforSpring
    | limit 50
    ```
    * Para ver las métricas, escriba una consulta simple como la siguiente:

    ```sql
    AzureMetrics
    | limit 50
    ```

1. Para ver el resultado de la búsqueda, seleccione **Ejecutar**.
1. Puede buscar los registros de la aplicación o instancia específicas con una condición de filtro:

    ```sql
    AppPlatformLogsforSpring
    | where ServiceName == "YourServiceName" and AppName == "YourAppName" and InstanceName == "YourInstanceName"
    | limit 50
    ```
> [!NOTE]
> `==` distingue mayúsculas de minúsculas, pero `=~` no.

Para más información sobre el lenguaje de consulta que se utiliza en Log Analytics, consulte [Consultas de registros de Azure Monitor](../azure-monitor/log-query/query-language.md).

### <a name="use-your-storage-account"></a>Uso de la cuenta de almacenamiento

1. En Azure Portal, busque **Cuentas de almacenamiento** en el panel de navegación de la izquierda o en el cuadro de búsqueda.
1. Seleccione la cuenta de almacenamiento que eligió al agregar la configuración de diagnóstico.
1. Para abrir el panel **Contenedor de blobs**, seleccione **Blobs**.
1. Para revisar los registros de aplicaciones, busque un contenedor llamado **insights-logs-applicationconsole**.
1. Para revisar las métricas de aplicaciones, busque un contenedor llamado **insights-metrics-pt1m**.

Para más información sobre cómo enviar información de diagnóstico a una cuenta de almacenamiento, consulte [Almacenamiento y visualización de los datos de diagnóstico en Azure Storage](../storage/common/storage-introduction.md).

### <a name="use-your-event-hub"></a>Uso del centro de eventos

1. En Azure Portal, busque **Event Hubs** en el panel de navegación de la izquierda o en el cuadro de búsqueda.

1. Busque y seleccione el centro de eventos que eligió al agregar la configuración de diagnóstico.
1. Para abrir el panel **Event Hub List** (Lista de centros de eventos), seleccione **Centros de eventos**.
1. Para revisar los registros de aplicaciones, busque un centro de eventos llamado **insights-logs-applicationconsole**.
1. Para revisar las métricas de aplicaciones, busque un centro de eventos llamado **insights-metrics-pt1m**.

Para más información sobre cómo enviar información de diagnóstico a un centro de eventos, consulte [Transmisión de datos de Azure Diagnostics en la ruta de acceso activa mediante Event Hubs](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostics-extension-stream-event-hubs).

## <a name="analyze-the-logs"></a>Análisis de los registros

Azure Log Analytics se ejecuta con un motor Kusto para que pueda consultar los registros para el análisis. Revise el [tutorial de Log Analytics](../azure-monitor/log-query/get-started-portal.md) para una introducción rápida a la consulta de registros con Kusto.

Los registros de aplicaciones proporcionan información crítica y registros detallados sobre el estado de la aplicación, el rendimiento y mucho más. En las siguientes secciones se muestran algunas consultas sencillas que le ayudarán a comprender los estados actual y anterior de la aplicación.

### <a name="show-application-logs-from-azure-spring-cloud"></a>Visualización de los registros de aplicaciones de Azure Spring Cloud

Para revisar una lista de los registros de aplicaciones de Azure Spring Cloud ordenados por tiempo donde los más recientes se muestran en primer lugar, ejecute la siguiente consulta:

```sql
AppPlatformLogsforSpring
| project TimeGenerated , ServiceName , AppName , InstanceName , Log
| sort by TimeGenerated desc
```

### <a name="show-logs-entries-containing-errors-or-exceptions"></a>Visualización de las entradas de registro con errores o excepciones

Para revisar las entradas de registro sin ordenar que mencionan un error o una excepción, ejecute la siguiente consulta:

```sql
AppPlatformLogsforSpring
| project TimeGenerated , ServiceName , AppName , InstanceName , Log
| where Log contains "error" or Log contains "exception"
```

Use esta consulta para buscar errores o modificar los términos de la consulta para encontrar códigos de error específicos o excepciones.

### <a name="show-the-number-of-errors-and-exceptions-reported-by-your-application-over-the-last-hour"></a>Visualización del número de errores y excepciones que ha notificado la aplicación durante la última hora

Para crear un gráfico circular que muestre el número de errores y excepciones registrados por la aplicación en la última hora, ejecute la siguiente consulta:

```sql
AppPlatformLogsforSpring
| where TimeGenerated > ago(1h)
| where Log contains "error" or Log contains "exception"
| summarize count_per_app = count() by AppName
| sort by count_per_app desc
| render piechart
```

### <a name="learn-more-about-querying-application-logs"></a>Más información sobre la consulta de registros de aplicaciones

Azure Monitor proporciona buen soporte técnico para consultar los registros de aplicaciones mediante Log Analytics. Para más información sobre este servicio, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/log-query/get-started-queries.md). Para más información sobre la compilación de consultas para analizar los registros de aplicaciones, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

### <a name="how-to-convert-multi-line-java-stack-traces-into-a-single-line"></a>¿Cómo se convierten los seguimientos de la pila de Java de varias líneas en una única línea?

Hay una solución alternativa para convertir los seguimientos de la pila de varias líneas en una única línea. Puede modificar la salida del registro de Java para volver a dar formato a los mensajes de seguimiento de la pila, reemplazando los caracteres de nueva línea por un token. Si usa la biblioteca Logback de Java, puede volver a dar formato a los mensajes de seguimiento de la pila con la adición de `%replace(%ex){'[\r\n]+', '\\n'}%nopex` como se indica a continuación:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                level: %level, message: "%logger{36}: %msg", exceptions: "%replace(%ex){'[\r\n]+', '\\n'}%nopex"%n
            </pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```
Y después puede reemplazar el token por caracteres de nueva línea en Log Analytics como se indica a continuación:

```sql
AppPlatformLogsforSpring
| extend Log = array_strcat(split(Log, '\\n'), '\n')
```
Es posible que pueda usar la misma estrategia para otras bibliotecas de registro de Java.