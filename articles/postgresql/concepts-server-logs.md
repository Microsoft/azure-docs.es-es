---
title: 'Registros de Azure Database for PostgreSQL: servidor único'
description: 'Se describe la configuración de registros, el almacenamiento y el análisis en Azure Database for PostgreSQL: servidor único.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 06/25/2020
ms.openlocfilehash: c5515c21d89b01576bb6aae8c9d480b439dc1507
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902696"
---
# <a name="logs-in-azure-database-for-postgresql---single-server"></a>Registros en Azure Database for PostgreSQL con un único servidor

Azure Database for PostgreSQL permite configurar y acceder a los registros estándar de Postgres. Los registros se pueden usar para identificar, solucionar y reparar errores de configuración y casos de rendimiento no óptimo. La información de registro que puede configurar y a la que puede acceder incluye errores, información de consultas, registros de vaciado automático, conexiones y puntos de control (no está disponible el acceso a los registros de transacciones).

El registro de auditoría está disponible a través de una extensión de Postgres, pgaudit. Para más información, visite el artículo sobre [conceptos de auditoría](concepts-audit.md).


## <a name="configure-logging"></a>registro 
Puede configurar el registro estándar de Postgres en el servidor mediante los parámetros de servidor relacionados con el registro. En cada servidor de Azure Database for PostgreSQL, `log_checkpoints` y `log_connections` están activados de forma predeterminada. Existen parámetros adicionales que puede ajustar según sus necesidades de registro: 

:::image type="content" source="./media/concepts-server-logs/log-parameters.png" alt-text="Azure Database for PostgreSQL: parámetros de registro":::

Para obtener más información acerca de los parámetros de registro de Postgres, visite las secciones [When To Log](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHEN) (Cuándo registrar) y [What To Log](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHAT) (Qué registrar) de la documentación de Postgres. La mayoría de los parámetros de registro de Postgres, pero no todos, se pueden configurar en Azure Database for PostgreSQL.

Para aprender a configurar los parámetros en Azure Database for PostgreSQL, consulte la [documentación del portal](howto-configure-server-parameters-using-portal.md) o la [documentación de la CLI](howto-configure-server-parameters-using-cli.md). 

> [!NOTE]
> La configuración de un volumen elevado de registros (por ejemplo, el registro de instrucciones) puede suponer una sobrecarga importante en el rendimiento. 

## <a name="access-log-files"></a>Acceso a los archivos .log
El formato de registro predeterminado en Azure Database for PostgreSQL es .log. Una línea de ejemplo de este registro tiene el siguiente aspecto:

```
2019-10-14 17:00:03 UTC-5d773cc3.3c-LOG: connection received: host=101.0.0.6 port=34331 pid=16216
```

Azure Database for PostgreSQL proporciona una ubicación de almacenamiento a corto plazo para los archivos .log. Cada hora o cada 100 MB, lo que suceda primero, comienza un nuevo archivo. Los registros se anexan al archivo actual a medida que se emiten desde Postgres.  

Puede establecer el período de retención de este almacenamiento de registros a corto plazo mediante el parámetro `log_retention_period`. El valor predeterminado es 3 días y el valor máximo 7 días. La ubicación de almacenamiento a corto plazo puede contener hasta 1 GB de archivos de registro. Una vez alcanzado 1 GB, se eliminarán los archivos más antiguos, con independencia del período de retención, para dejar espacio a los nuevos registros. 

Para la retención a largo plazo de registros y análisis de registros, puede descargar los archivos .log y moverlos a un servicio de terceros. Puede descargar los archivos mediante [Azure Portal](howto-configure-server-logs-in-portal.md) o la [CLI de Azure](howto-configure-server-logs-using-cli.md). Como alternativa, puede establecer la configuración de diagnóstico de Azure Monitor, que emite automáticamente los registros (en formato JSON) a ubicaciones a largo plazo. En la sección siguiente puede obtener más información sobre esta opción. 

Puede detener la generación de archivos .log si establece el parámetro `logging_collector` en OFF. Se recomienda desactivar la generación de archivos .log si se usa la configuración de diagnóstico de Azure Monitor. Esta configuración reducirá el impacto en el rendimiento del registro adicional.

## <a name="resource-logs"></a>Registros del recurso

Azure Database for PostgreSQL se integra con la configuración de diagnóstico de Azure Monitor. Esta configuración permite enviar los registros de Postgres en formato JSON a los registros de Azure Monitor para llevar a cabo análisis y creación de alertas, a Event Hubs para streaming y a Azure Storage para el archivado. 

> [!IMPORTANT]
> Esta característica de diagnóstico para registros de servidor solo está disponible en los [planes de tarifa](concepts-pricing-tiers.md) de uso general y optimizados para memoria.


### <a name="configure-diagnostic-settings"></a>Configuración de valores de diagnóstico

Puede habilitar la configuración de diagnóstico para el servidor de Postgres mediante Azure Portal, la CLI, API REST y PowerShell. La categoría de registro que se debe seleccionar es **PostgreSQLLogs** (hay otros registros que puede configurar si usa el [Almacén de consultas](concepts-query-store.md)).

Para habilitar los registros de recursos mediante Azure Portal:

   1. En el portal, vaya a *Configuración de diagnóstico*  en el menú de navegación del servidor de Postgres.
   2. Seleccione *Agregar configuración de diagnóstico*.
   3. Asigne un nombre a esta configuración. 
   4. Seleccione el punto de conexión preferido (cuenta de almacenamiento, centro de eventos, análisis de registros). 
   5. Seleccione el tipo de registro **PostgreSQLLogs**.
   7. Guarde la configuración.

Para habilitar los registros de recursos mediante PowerShell, la CLI o la API REST, consulte el artículo sobre la [configuración de diagnóstico](../azure-monitor/platform/diagnostic-settings.md).

### <a name="access-resource-logs"></a>Acceso a los registros de recursos

La forma de acceder a los registros depende del punto de conexión que elija. Si se trata de Azure Storage, consulte el artículo sobre la [cuenta de almacenamiento de registros](../azure-monitor/platform/resource-logs-collect-storage.md). Si se trata de Event Hubs, consulte el artículo [Transmisión de los registros de Azure](../azure-monitor/platform/resource-logs-stream-event-hubs.md).

Si se trata de los registros de Azure Monitor, los registros se envían al área de trabajo seleccionada. Los registros de Postgres usan el modo de recopilación **AzureDiagnostics**, por lo que se pueden consultar desde la tabla AzureDiagnostics. A continuación se describen los campos de la tabla. Obtenga más información acerca de las consultas y las alertas en [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

A continuación se muestran consultas que puede intentar iniciar. Puede configurar alertas basadas en las consultas.

Buscar todos los registros de Postgres de un servidor determinado del último día
```
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where Category == "PostgreSQLLogs"
| where TimeGenerated > ago(1d) 
```

Buscar todos los intentos de conexión que no son de localhost
```
AzureDiagnostics
| where Message contains "connection received" and Message !contains "host=127.0.0.1"
| where Category == "PostgreSQLLogs" and TimeGenerated > ago(6h)
```
La consulta anterior mostrará los resultados de las últimas 6 horas para cualquier servidor Postgres que inicie sesión en esta área de trabajo.

### <a name="log-format"></a>Formato de registro

En la tabla siguiente se describen los campos del tipo **PostgreSQLLogs**. En función del punto de conexión de salida que elija, pueden variar los campos incluidos y el orden en el que aparecen. 

|**Campo** | **Descripción** |
|---|---|
| TenantId | El identificador de inquilino |
| SourceSystem | `Azure` |
| TimeGenerated [UTC] | Marca de tiempo de cuando se grabó el registro en UTC |
| Tipo | Tipo del registro. Siempre `AzureDiagnostics` |
| SubscriptionId | GUID de la suscripción a la que pertenece el servidor |
| ResourceGroup | Nombre del grupo de recursos al que pertenece el servidor |
| ResourceProvider | Nombre del proveedor de recursos Siempre `MICROSOFT.DBFORPOSTGRESQL` |
| ResourceType | `Servers` |
| ResourceId | URI de recurso |
| Recurso | Nombre del servidor |
| Category | `PostgreSQLLogs` |
| OperationName | `LogEvent` |
| errorLevel | Ejemplo de nivel de registro: LOG, ERROR, NOTICE |
| Message | Mensaje de registro principal | 
| Domain | Versión del servidor, ejemplo: postgres-10 |
| Detail | Mensaje de registro secundario (si procede) |
| ColumnName | Nombre de la columna (si procede) |
| SchemaName | Nombre del esquema (si procede) |
| DatatypeName | Nombre del tipo de datos (si procede) |
| LogicalServerName | Nombre del servidor | 
| _ResourceId | URI de recurso |
| Prefijo | Prefijo de la línea de registro |


## <a name="next-steps"></a>Pasos siguientes
- Más información sobre el acceso a los registros desde [Azure Portal](howto-configure-server-logs-in-portal.md) o la [CLI de Azure](howto-configure-server-logs-using-cli.md).
- Más información sobre los [precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).
- Más información sobre los [registros de auditoría](concepts-audit.md)
