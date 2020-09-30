---
title: 'Registros de auditoría en Azure Database for MySQL: Servidor flexible'
description: Se describen los registros de auditoría disponibles en el Servidor flexible de Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: e630ccec744c4edf0dc80d374ecc93526f960d44
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932194"
---
# <a name="track-database-activity-with-audit-logs-in-azure-database-for-mysql-flexible-server"></a>Seguimiento de la actividad de base de datos con Registros de auditoría en el Servidor flexible de Azure Database for MySQL

> [!IMPORTANT] 
> El Servidor flexible de Azure Database for MySQL está actualmente en versión preliminar pública.

El Servidor flexible de Azure Database for MySQL proporciona a los usuarios la capacidad de configurar registros de auditoría. Los registros de auditoría se pueden usar para realizar un seguimiento de la actividad de nivel de base de datos, incluidos los eventos de conexión, administración, DDL y DML. Estos tipos de registros se usan normalmente para fines de cumplimiento.

## <a name="configure-audit-logging"></a>Configuración del registro de auditoría

>[!IMPORTANT]
> Se recomienda registrar solo los tipos de evento y los usuarios necesarios con fines de auditoría para asegurarse de que el rendimiento del servidor no se ve afectado en gran medida.

Los registros de auditoría están deshabilitados de forma predeterminada. Para habilitarlos, establezca el parámetro del servidor `audit_log_enabled` en *Activado*. Para ello, se puede usar Azure Portal o la CLI de Azure <!-- add link to server parameter-->. 

Entre otros parámetros que se pueden ajustar para controlar el comportamiento del registro de auditoría se incluyen los siguientes:

- `audit_log_events`: controla los eventos que se registrarán. Consulte la tabla a continuación para ver los eventos de auditoría específicos.
- `audit_log_include_users`: Usuarios de MySQL que se incluirán para el registro. El valor predeterminado de este parámetro es estar vacío, lo que incluirá todos los usuarios en el registro. Este tiene una mayor prioridad que `audit_log_exclude_users`. La longitud máxima del parámetro es de 512 caracteres.
- `audit_log_exclude_users`: Los usuarios de MySQL que se excluirán del registro. La longitud máxima del parámetro es de 512 caracteres.

> [!NOTE]
> `audit_log_include_users` tiene mayor prioridad sobre `audit_log_exclude_users`. Por ejemplo, si `audit_log_include_users` = `demouser` y `audit_log_exclude_users` = `demouser`, el usuario se incluirá en los registros de auditoría porque `audit_log_include_users` tiene mayor prioridad.

| **Evento** | **Descripción** |
|---|---|
| `CONNECTION` | - Iniciación de la conexión (correcta o incorrecta) <br> - Reautenticación de usuario con un usuario o contraseña diferentes durante la sesión <br> - Terminación de la conexión |
| `DML_SELECT`| Consultas SELECT |
| `DML_NONSELECT` | Consultas INSERT/DELETE/UPDATE |
| `DML` | DML = DML_SELECT + DML_NONSELECT |
| `DDL` | Consultas como "DROP DATABASE" |
| `DCL` | Consultas como "GRANT PERMISSION" |
| `ADMIN` | Consultas como "SHOW STATUS" |
| `GENERAL` | Todos en DML_SELECT, DML_NONSELECT, DML, DDL, DCL y ADMIN |
| `TABLE_ACCESS` | - Solo está disponible para MySQL 5.7 <br> - Instrucciones de lectura de tablas como SELECT o INSERT INTO... SELECT <br> - Instrucciones de eliminación de tablas, como DELETE o TRUNCATE TABLE <br> - Instrucciones de inserción en tablas, como INSERT o REPLACE <br> - Instrucciones de actualización de tablas, como UPDATE |

## <a name="access-audit-logs"></a>Acceso a registros de auditoría

Los registros de auditoría están integrados con la configuración de diagnóstico de Azure Monitor. Una vez que haya habilitado los registros de auditoría en el servidor flexible de MySQL, puede enviarlos a los registros de Azure Monitor, Event Hubs o Azure Storage. Para obtener más información acerca de la configuración de diagnóstico, vea la [documentación de los registros de diagnóstico](../../azure-monitor/platform/platform-logs-overview.md). Para obtener más información sobre cómo habilitar la configuración de diagnóstico en Azure Portal, vea el [artículo sobre registros de auditoría en el portal](how-to-configure-audit-logs-portal.md#set-up-diagnostics).

En las secciones siguientes se describe la salida de los registros de auditoría de MySQL en función del tipo de evento. En función del método de salida que elija, pueden variar los campos incluidos y el orden en el que aparecen.

### <a name="connection"></a>Conexión

| **Propiedad** | **Descripción** |
|---|---|
| `TenantId` | El identificador de inquilino |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Marca de tiempo de cuando se grabó el registro en UTC |
| `Type` | Tipo del registro. Siempre `AzureDiagnostics` |
| `SubscriptionId` | GUID de la suscripción a la que pertenece el servidor |
| `ResourceGroup` | Nombre del grupo de recursos al que pertenece el servidor |
| `ResourceProvider` | Nombre del proveedor de recursos Siempre `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de recurso |
| `Resource` | Nombre del servidor |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Nombre del servidor |
| `event_class_s` | `connection_log` |
| `event_subclass_s` | `CONNECT`, `DISCONNECT`, `CHANGE USER` (solo disponible para MySQL 5.7) |
| `connection_id_d` | Identificador único de conexión generado por MySQL |
| `host_s` | En blanco |
| `ip_s` | Dirección IP del cliente que se conecta a MySQL |
| `user_s` | Nombre del usuario que ejecuta la consulta |
| `db_s` | Nombre de la base de datos a la que se conecta |
| `\_ResourceId` | URI de recurso |

### <a name="general"></a>General

El esquema siguiente se aplica a los tipos de evento GENERAL, DML_SELECT, DML_NONSELECT, DML, DDL, DCL y ADMIN.

> [!NOTE]
> Para `sql_text_s`, el registro se truncará si supera los 2048 caracteres.

| **Propiedad** | **Descripción** |
|---|---|
| `TenantId` | El identificador de inquilino |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Marca de tiempo de cuando se grabó el registro en UTC |
| `Type` | Tipo del registro. Siempre `AzureDiagnostics` |
| `SubscriptionId` | GUID de la suscripción a la que pertenece el servidor |
| `ResourceGroup` | Nombre del grupo de recursos al que pertenece el servidor |
| `ResourceProvider` | Nombre del proveedor de recursos Siempre `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de recurso |
| `Resource` | Nombre del servidor |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Nombre del servidor |
| `event_class_s` | `general_log` |
| `event_subclass_s` | `LOG`, `ERROR`, `RESULT` (solo disponible para MySQL 5.6) |
| `event_time` | Hora de inicio de la consulta en marca de tiempo UTC |
| `error_code_d` | Código de error si la consulta no es correcta. `0` significa que no hay error |
| `thread_id_d` | Id. del subproceso que ejecutó la consulta |
| `host_s` | En blanco |
| `ip_s` | Dirección IP del cliente que se conecta a MySQL |
| `user_s` | Nombre del usuario que ejecuta la consulta |
| `sql_text_s` | Texto de la consulta completa |
| `\_ResourceId` | URI de recurso |

### <a name="table-access"></a>Acceso a la tabla

> [!NOTE]
> Los registros de acceso a tablas solo se muestran en MySQL 5.7.<br>Para `sql_text_s`, el registro se truncará si supera los 2048 caracteres.

| **Propiedad** | **Descripción** |
|---|---|
| `TenantId` | El identificador de inquilino |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Marca de tiempo de cuando se grabó el registro en UTC |
| `Type` | Tipo del registro. Siempre `AzureDiagnostics` |
| `SubscriptionId` | GUID de la suscripción a la que pertenece el servidor |
| `ResourceGroup` | Nombre del grupo de recursos al que pertenece el servidor |
| `ResourceProvider` | Nombre del proveedor de recursos Siempre `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de recurso |
| `Resource` | Nombre del servidor |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Nombre del servidor |
| `event_class_s` | `table_access_log` |
| `event_subclass_s` | `READ`, `INSERT`, `UPDATE` o `DELETE` |
| `connection_id_d` | Identificador único de conexión generado por MySQL |
| `db_s` | Nombre de la base de datos a la que se accede |
| `table_s` | Nombre de la tabla a la que se accede |
| `sql_text_s` | Texto de la consulta completa |
| `\_ResourceId` | URI de recurso |

## <a name="analyze-logs-in-azure-monitor-logs"></a>Análisis de registros en los registros de Azure Monitor

Una vez que los registros de auditoría se canalizan a los registros de Azure Monitor a través de registros de diagnóstico, puede realizar un análisis en mayor profundidad de sus eventos auditados. A continuación encontrará algunas consultas de ejemplo que le ayudarán a ponerse en marcha. Asegúrese de que actualizar los datos siguientes con el nombre del servidor.

- Enumeración de los eventos GENERALES en un servidor determinado

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs' and event_class_s == "general_log"
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last 
    ```

- Enumeración de los eventos de CONEXIÓN en un servidor determinado

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs' and event_class_s == "connection_log"
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ```

- Resumen de los eventos auditados en un servidor determinado

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | summarize count() by event_class_s, event_subclass_s, user_s, ip_s
    ```

- Representación de una distribución de tipo de evento de auditoría en un servidor concreto

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart 
    ```

- Enumeración de los eventos auditados en todos los servidores de MySQL con registros de diagnóstico habilitados para los registros de auditoría

    ```kusto
    AzureDiagnostics
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ``` 

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre los [Registros de consultas lentos](concepts-slow-query-logs.md)
- Configuración de registros de consultas de auditoría en [Azure Portal](how-to-configure-audit-logs-portal.md)
<!-- - [How to configure audit logs in the Azure portal](howto-configure-audit-logs-portal.md)-->