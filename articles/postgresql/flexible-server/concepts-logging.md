---
title: 'Registros en Azure Database for PostgreSQL: Servidor flexible'
description: 'Se describe la configuración, el almacenamiento y el análisis de registros en Azure Database for PostgreSQL: Servidor flexible'
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 852bce26e348c817b609d5ff837641741afe4461
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932473"
---
# <a name="logs-in-azure-database-for-postgresql---flexible-server"></a>Registros en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar

Azure Database for PostgreSQL permite configurar los registros estándar de Postgres y acceder a ellos. Los registros se pueden usar para identificar, solucionar y reparar errores de configuración y casos de rendimiento no óptimo. La información de registro que puede configurar y a la que puede acceder incluye errores, información de consultas, registros de vaciado automático, conexiones y puntos de control (no está disponible el acceso a los registros de transacciones).

El registro de auditoría está disponible a través de una extensión de Postgres, `pgaudit`. Para más información, visite el artículo sobre [conceptos de auditoría](concepts-audit.md).

## <a name="configure-logging"></a>registro

Puede configurar el registro estándar de Postgres en el servidor mediante los parámetros de servidor relacionados con el registro. Para obtener más información acerca de los parámetros de registro de Postgres, visite las secciones [When To Log](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHEN) (Cuándo registrar) y [What To Log](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHAT) (Qué registrar) de la documentación de Postgres. La mayoría de los parámetros de registro de Postgres, pero no todos, se pueden configurar en Azure Database for PostgreSQL.

Para aprender a configurar los parámetros en Azure Database for PostgreSQL, consulte la [documentación del portal](howto-configure-server-parameters-using-portal.md) o la [documentación de la CLI](howto-configure-server-parameters-using-cli.md).

> [!NOTE]
> La configuración de un volumen elevado de registros (por ejemplo, el registro de instrucciones) puede suponer una sobrecarga importante en el rendimiento. 

## <a name="accessing-logs"></a>Acceso a los registros

Azure Database for PostgreSQL se integra con la configuración de diagnóstico de Azure Monitor. Esta configuración permite enviar los registros de Postgres en formato JSON a los registros de Azure Monitor para llevar a cabo análisis y creación de alertas, a Event Hubs para streaming y a Azure Storage para el archivado. 

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

- Más información acerca de cómo [Configurar registros y acceder a ellos](howto-configure-and-access-logs.md).
- Más información sobre los [precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).
- Más información sobre los [registros de auditoría](concepts-audit.md)
