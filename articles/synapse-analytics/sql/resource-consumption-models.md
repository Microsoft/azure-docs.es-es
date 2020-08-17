---
title: Consumo de recursos de Synapse SQL
description: Más información sobre los modelos de consumo de Synapse SQL en Azure Synapse Analytics.
services: synapse analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.openlocfilehash: 086c853b21301074cf35f13db52db1fdb145518c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489578"
---
# <a name="synapse-sql-resource-consumption"></a>Consumo de recursos de Synapse SQL

En este artículo se describen los modelos de consumo de recursos de Synapse SQL (versión preliminar).

## <a name="sql-on-demand"></a>SQL a petición

SQL a petición es un servicio de pago por consulta que no requiere que se elija el tamaño correcto. El sistema se ajusta automáticamente en función de sus requisitos, lo que le libera de administrar la infraestructura y elegir el tamaño adecuado para la solución.

## <a name="sql-pool---data-warehouse-units-dwus-and-compute-data-warehouse-units-cdwus"></a>Grupo de SQL: unidades de almacenamiento de datos (DWU) y unidades de almacenamiento de datos de proceso (cDWU)

Se incluyen recomendaciones acerca de cómo elegir el número ideal de unidades de almacenamiento de datos (DWU) para optimizar el precio y el rendimiento y cómo cambiar el número de unidades.

### <a name="data-warehouse-units"></a>Unidades de almacenamiento de datos

Un grupo de Synapse SQL representa una colección de recursos analíticos que se aprovisionan. Los recursos analíticos se definen como una combinación de CPU, memoria y E/S. Estos tres recursos se agrupan en unidades de escalado de proceso denominadas Unidades de almacenamiento de datos (DWU). Una DWU representa una medida abstracta y normalizada de recursos de proceso y rendimiento. Un cambio en el nivel de servicio modifica el número de DWU que están disponibles en el sistema, lo que a su vez ajusta el rendimiento y el costo del sistema.

Para obtener un mayor rendimiento, puede aumentar el número de unidades de almacenamiento de datos. Para obtener un menor rendimiento, reduzca las unidades de almacenamiento de datos. Los costos de almacenamiento y de proceso se facturan por separado, por lo que cambiar las unidades de almacenamiento de datos no afecta a los costos de almacenamiento.

El rendimiento de las unidades de almacenamiento de datos se basa en estas métricas de carga de trabajo de almacenamiento de datos:

- Con qué rapidez una consulta de almacenamiento de datos estándar puede examinar un gran número de filas y, después, realizar una agregación compleja. Esta es una operación de gran consumo de E/S y de CPU.
- Con qué rapidez el almacenamiento de datos puede ingerir datos de Azure Storage Blob o de Azure Data Lake. Esta es una operación de gran consumo de red y CPU.
- Con qué rapidez el comando T-SQL [`CREATE TABLE AS SELECT`](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) puede copiar una tabla. Esta operación implica la lectura de datos del almacenamiento, su distribución entre los nodos del dispositivo y su nueva escritura en el almacenamiento. Esta es una operación de gran consumo de CPU, E/S y red.

Aumentar las DWU:

- Cambia linealmente el rendimiento del sistema para exámenes, agregaciones e instrucciones CTAS.
- Aumenta el número de lectores y escritores para las operaciones de carga de PolyBase.
- Aumenta el número máximo de consultas simultáneas y ranuras de simultaneidad.

### <a name="service-level-objective"></a>Objetivo de nivel de servicio

El objetivo de nivel de servicio (SLO) es la opción de escalabilidad que determina el nivel de costo y el rendimiento del almacenamiento de datos. Los niveles de servicio de Gen2 se miden en unidades de almacenamiento de datos de proceso (cDWU); por ejemplo, DW2000c. Los niveles de servicio de Gen1 se miden en DWU; por ejemplo, DW2000.

El objetivo de nivel de servicio (SLO) es la opción de escalabilidad que determina el nivel de costo y el rendimiento del almacenamiento de datos. Los niveles de servicio del grupo SQL Gen2 se miden en unidades de almacenamiento de datos (DWU); por ejemplo, DW2000c.

> [!NOTE]
> Azure SQL Data Warehouse Gen2 ha agregado recientemente funcionalidades de escalado adicionales compatibles con niveles de proceso tan bajos como 100 cDWU. Los almacenes de datos existentes actualmente en Gen1 que requieren los niveles de proceso más bajos ahora pueden actualizarse a Gen2 en las regiones que están actualmente disponibles sin ningún costo adicional.  Si esto no se admite aún en su región, aún puede actualizar a una región admitida. Para obtener más información, vea [Actualización a Gen2](../sql-data-warehouse/upgrade-to-latest-generation.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

En T-SQL, el valor de SERVICE_OBJECTIVE determina el nivel de servicio y el nivel de rendimiento del grupo SQL.

```sql
CREATE DATABASE mySQLDW
(Edition = 'Datawarehouse'
 ,SERVICE_OBJECTIVE = 'DW1000c'
)
;
```

### <a name="performance-tiers-and-data-warehouse-units"></a>Niveles de rendimiento y unidades de almacenamiento de datos

Cada nivel de rendimiento usa una unidad de medida ligeramente diferente para sus unidades de almacenamiento de datos. Esta diferencia se refleja en la factura, ya que la unidad de escala se traduce directamente en la facturación.

- Los almacenamientos de datos de Gen1 se miden en unidades de almacenamiento de datos (DWU).
- Los almacenamientos de datos de Gen2 se miden en unidades de almacenamiento de datos de proceso (cDWU).

Tanto las DWU como las cDWU admiten el escalado vertical y la reducción vertical del proceso, así como pausar el proceso cuando no es necesario usar el almacén de datos. Estas operaciones son a petición. El nivel Gen2 usa una memoria caché basada en disco local en los nodos de proceso para mejorar el rendimiento. Al escalar o pausar el sistema, se invalida la memoria caché y es necesario un período de calentamiento de la memoria caché para conseguir un rendimiento óptimo.  

A medida que aumente unidades de almacenamiento de datos, también se aumentan linealmente los recursos informáticos. Gen2 proporciona el mejor rendimiento de consultas y la mayor escalabilidad. Los sistemas Gen2 también aprovechan al máximo el uso de la memoria caché.

#### <a name="capacity-limits"></a>Límites de capacidad

Cada servidor SQL Server (por ejemplo, myserver.database.windows.net) tiene una cuota de [unidad de transacción de base de datos (DTU)](../../azure-sql/database/service-tiers-dtu.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) que permite un número específico de unidades de almacenamiento de datos. Para más información, consulte los [límites de capacidad de administración de cargas de trabajo](../sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#workload-management).

### <a name="assess-the-number-of-data-warehouse-units-you-need"></a>Evaluación del número de unidades del almacenamiento de datos necesarias

El número ideal de unidades de almacenamiento de datos depende en gran medida de la carga de trabajo y la cantidad de datos que cargó en el sistema.

Pasos para encontrar la mejor DWU para la carga de trabajo:

1. Comience por seleccionar una DWU más pequeña.
2. Supervise el rendimiento de su aplicación a medida que prueba cargas de datos en el sistema, observando el número de DWU seleccionadas en comparación con el rendimiento que observe.
3. Identifique los requisitos adicionales para períodos de máxima actividad periódicos. Puede que las cargas de trabajo que muestran picos y aumentos de actividad significativos se deban escalar con frecuencia.

Un grupo de SQL es un sistema de escalado horizontal que puede aprovisionar grandes cantidades de procesos y consultar cantidades considerables de datos. Para ver sus verdaderas capacidades de escalado, especialmente en DWU más grandes, se recomienda escalar el conjunto de datos para asegurar que tiene suficientes datos como para alimentar las CPU. Para probar la escala, se recomienda usar al menos 1 TB.

> [!NOTE]
>
> El rendimiento de las consultas solo aumenta con más paralelización si el trabajo se puede dividir entre nodos de proceso. Si ve que el escalado no cambia el rendimiento, es posible que deba ajustar el diseño de las tablas o de las consultas. Para obtener instrucciones para el ajuste de consultas, vea [Manage user queries](../overview-cheat-sheet.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (Administración de consultas de usuarios).

### <a name="permissions"></a>Permisos

Para cambiar las unidades de almacenamiento de datos es necesario disponer de los permisos descritos en [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

Los roles integrados de Azure, como Colaborador de SQL DB y Colaborador de SQL Server, pueden cambiar la configuración de DWU.

#### <a name="view-current-dwu-settings"></a>Ver la configuración de DWU actual

Para ver la configuración actual de DWU:

1. Abra el Explorador de objetos de SQL Server en Visual Studio.
2. Conéctese a la base de datos maestra asociada al servidor SQL lógico.
3. Seleccione en la vista de administración dinámica sys.database_service_objectives. Este es un ejemplo:

```sql
SELECT  db.name [Database]
,        ds.edition [Edition]
,        ds.service_objective [Service Objective]
FROM    sys.database_service_objectives   AS ds
JOIN    sys.databases                     AS db ON ds.database_id = db.database_id
;
```

### <a name="change-data-warehouse-units"></a>Cambiar unidades de almacenamiento de datos

#### <a name="azure-portal"></a>Azure portal

Para cambiar DWU:

1. Abra [Azure Portal](https://portal.azure.com), abra la base de datos y haga clic en **Escalar**.

2. En **Escalar**, mueva el control deslizante izquierdo o derecho para cambiar el valor de DWU.

3. Haga clic en **Save**(Guardar). Aparece un mensaje de confirmación. Haga clic en **Sí** para confirmar o **No** para cancelar.

#### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para cambiar las DWU, use el cmdlet de PowerShell [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). En el ejemplo siguiente se establece el objetivo de nivel de servicio en DW1000 para la base de datos MySQLDW que se hospeda en el servidor MyServer.

```Powershell
Set-AzSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer" -RequestedServiceObjectiveName "DW1000c"
```

Para obtener más información, vea [Cmdlets de PowerShell y API REST para SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-reference-powershell-cmdlets.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

#### <a name="t-sql"></a>T-SQL

Con T-SQL, puede ver la configuración actual de DWU, modificarla y comprobar el progreso.

Para cambiar las DWU:

1. Conéctese a la base de datos maestra asociada al servidor.
2. Use la instrucción TSQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest). En el ejemplo siguiente se establece el objetivo de nivel de servicio en DW1000c para la base de datos MySQLDW.

```Sql
ALTER DATABASE MySQLDW
MODIFY (SERVICE_OBJECTIVE = 'DW1000c')
;
```

#### <a name="rest-apis"></a>API de REST

Para cambiar las DWU, utilice la API REST [Create or Update Database](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) (Creación o actualización de base de datos). En el ejemplo siguiente se establece el objetivo de nivel de servicio en DW1000c para la base de datos MySQLDW, que se hospeda en el servidor MyServer. El servidor está en un grupo de recursos de Azure denominado ResourceGroup1.

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}?api-version=2014-04-01-preview HTTP/1.1
Content-Type: application/json; charset=UTF-8

{
    "properties": {
        "requestedServiceObjectiveName": DW1000
    }
}
```

Para obtener más ejemplos de API REST, consulte [API REST para Azure SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="check-status-of-dwu-changes"></a>Comprobar el estado de los cambios de DWU

Los cambios de DWU pueden tardar varios minutos en completarse. Si está realizando una operación de escalado automáticamente, considere implementar la lógica para asegurarse de que ciertas operaciones se completaron antes de pasar a realizar otra acción.

La comprobación del estado de la base de datos a través de varios puntos de conexión le permitirá implementar correctamente la automatización. El portal le proporcionará una notificación tras la finalización de una operación y el estado actual de las bases de datos, pero no permitirá la comprobación programática del estado.

No se puede comprobar el estado de la base de datos para las operaciones de escalado horizontal con Azure Portal.

Para comprobar el estado de los cambios de DWU:

1. Conéctese a la base de datos maestra asociada al servidor.
2. Envíe la consulta siguiente para comprobar el estado de la base de datos.

```sql
SELECT    *
FROM      sys.databases
;
```

1. Envíe la consulta siguiente para comprobar el estado de la operación.

```sql
SELECT    *
FROM      sys.dm_operation_status
WHERE     resource_type_desc = 'Database'
AND       major_resource_id = 'MySQLDW'
;
```

Esta DMV devuelve información sobre varias operaciones de administración en el grupo SQL, como la operación y el estado de esta, que es IN_PROGRESS o COMPLETED.

### <a name="the-scaling-workflow"></a>Flujo de trabajo de escalado

Cuando se inicia una operación de escalado, el sistema elimina primero todas las sesiones abiertas y revierte todas las transacciones abiertas para garantizar un estado coherente. Para las operaciones de escalado, el escalado solo se producirá una vez completada esta reversión transaccional.  

- Para una operación de escalado vertical, el sistema desasocia todos los nodos de proceso, aprovisiona los nodos de proceso adicionales y luego se vuelve a asociar a la capa de almacenamiento.
- Para una operación de reducción vertical, el sistema desasocia todos los nodos de proceso y luego solo vuelve a asociar los nodos necesarios para la capa de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo administrar el rendimiento, consulte [Clases de recursos para la administración de cargas de trabajo](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) y [Límites de memoria y simultaneidad](../sql-data-warehouse/memory-concurrency-limits.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
