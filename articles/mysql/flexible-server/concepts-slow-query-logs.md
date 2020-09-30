---
title: Registros de consultas lentas en Azure Database for MySQL con servidor flexible
description: Se describen los registros de consultas lentas disponibles en Azure Database for MySQL con servidor flexible.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: 2ce10353d1d102667718f368dc55a5821f16e25d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932110"
---
# <a name="slow-query-logs-in-azure-database-for-mysql-flexible-server-preview"></a>Registros de consultas lentas en Azure Database for MySQL con servidor flexible (versión preliminar)

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con servidor flexible se encuentra en versión preliminar pública.

En Azure Database for MySQL con servidor flexible, los usuarios pueden configurar registros de consultas lentas y acceder a ellos. De forma predeterminada, los registros de consultas lentas están deshabilitados y se pueden habilitar para ayudar a identificar los cuellos de botella de rendimiento durante la solución de problemas.

Para obtener más información sobre el registro de consultas lentas de MySQL, vea la [sección sobre el registro de consultas lentas](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) en la documentación del motor de MySQL.

## <a name="configure-slow-query-logging"></a>Configuración del registro de consultas lentas 
De forma predeterminada, los registros de consultas lentas están deshabilitados. Para habilitarlos, establezca el parámetro `slow_query_log` del servidor en *Activado*. Para ello, se puede usar Azure Portal o la CLI de Azure <!-- add link to server parameter-->. 

Entre los parámetros que se pueden ajustar para controlar el comportamiento de los registros de consultas lentas se incluyen los siguientes:

- **long_query_time**: registra una consulta si tarda más de `long_query_time` (en segundos) en completarse. El valor predeterminado es 10 segundos.
- **log_slow_admin_statements**: determina si se registran las instrucciones administrativas (por ejemplo, `ALTER_TABLE` y `ANALYZE_TABLE`).
- **log_queries_not_using_indexes**: determina si se registran las consultas que no usan índices.
- **log_throttle_queries_not_using_indexes**: limita el número de consultas que no son de índice que se pueden escribir en el registro de consultas lentas. Este parámetro surte efecto cuando `log_queries_not_using_indexes` se establece en *Activado*.

> [!IMPORTANT]
> Si las tablas no están indexadas y se establecen los parámetros `log_queries_not_using_indexes` y `log_throttle_queries_not_using_indexes` en **Activado**, el rendimiento de MySQL puede verse afectado, ya que todas las consultas que se ejecuten en estas tablas no indexadas se escribirán en el registro de consultas lentas.

Consulte la [documentación rel registro de consultas lentas](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) de MySQL para obtener una descripción completa de los parámetros de registro de consultas lentas.

## <a name="access-slow-query-logs"></a>Acceso a registros de consultas lentas

Los registros de consultas lentas están integrados con la configuración de diagnóstico de Azure Monitor. Una vez que haya habilitado los registros de auditoría en el servidor flexible de MySQL, puede enviarlos a los registros de Azure Monitor, Event Hubs o Azure Storage. Para obtener más información acerca de la configuración de diagnóstico, vea la [documentación de los registros de diagnóstico](../../azure-monitor/platform/platform-logs-overview.md). Para obtener más información sobre cómo habilitar la configuración de diagnóstico en Azure Portal, vea el [artículo sobre registros de consultas lentas en el portal](how-to-configure-slow-query-logs-portal.md#set-up-diagnostics).

En la tabla siguiente se describe la salida del registro de consultas lentas. En función del método de salida que elija, pueden variar los campos incluidos y el orden en el que aparecen.

| **Propiedad** | **Descripción** |
|---|---|
| `TenantId` | El identificador de inquilino |
| `SourceSystem` | `Azure` |
| `TimeGenerated` [UTC] | Marca de tiempo de cuando se grabó el registro en UTC |
| `Type` | Tipo del registro. Siempre `AzureDiagnostics` |
| `SubscriptionId` | GUID de la suscripción a la que pertenece el servidor |
| `ResourceGroup` | Nombre del grupo de recursos al que pertenece el servidor |
| `ResourceProvider` | Nombre del proveedor de recursos Siempre `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de recurso |
| `Resource` | Nombre del servidor |
| `Category` | `MySqlSlowLogs` |
| `OperationName` | `LogEvent` |
| `Logical_server_name_s` | Nombre del servidor |
| `start_time_t` [UTC] | Hora de inicio de la consulta |
| `query_time_s` | Tiempo total en segundos que tardó en ejecutarse la consulta |
| `lock_time_s` | Tiempo total en segundos durante el que se bloqueó la consulta |
| `user_host_s` | Nombre de usuario |
| `rows_sent_s` | Número de filas enviadas |
| `rows_examined_s` | Número de filas examinadas |
| `last_insert_id_s` | [last_insert_id](https://dev.mysql.com/doc/refman/5.7/en/information-functions.html#function_last-insert-id) |
| `insert_id_s` | Id. de inserción |
| `sql_text_s` | Consulta completa |
| `server_id_s` | ID del servidor |
| `thread_id_s` | Id. de subproceso |
| `\_ResourceId` | URI de recurso |

> [!Note]
> Para `sql_text_s`, el registro se truncará si supera los 2048 caracteres.

## <a name="analyze-logs-in-azure-monitor-logs"></a>Análisis de registros en los registros de Azure Monitor

Una vez que los registros de consultas lentos se canalizan a los registros de Azure Monitor a través de registros de diagnóstico, puede realizar un análisis en mayor profundidad de sus consultas lentas. A continuación encontrará algunas consultas de ejemplo que le ayudarán a ponerse en marcha. Asegúrese de que actualizar los datos siguientes con el nombre del servidor.

- Consultas que duran más de 10 segundos en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```

- Lista de cinco consultas más largas en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | order by query_time_d desc
    | take 5
    ```

- Resumir las consultas lentas por tiempo de consulta mínimo, máximo, medio y desviación estándar en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count(), min(query_time_d), max(query_time_d), avg(query_time_d), stdev(query_time_d), percentile(query_time_d, 95) by LogicalServerName_s
    ```

- Representar en un gráfico la distribución de consultas lentas en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart
    ```

- Mostrar las consultas durante más de 10 segundos en todos los servidores de MySQL con Registros de diagnóstico habilitado

    ```Kusto
    AzureDiagnostics
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```    
    
## <a name="next-steps"></a>Pasos siguientes
- Más información sobre los [registros de auditoría](concepts-audit-logs.md)
- Configuración de registros de consultas lentas en [Azure Portal](how-to-configure-slow-query-logs-portal.md)
<!-- - [How to configure slow query logs from the Azure CLI](howto-configure-server-logs-in-cli.md). -->