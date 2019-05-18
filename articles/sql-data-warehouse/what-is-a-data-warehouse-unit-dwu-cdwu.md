---
title: Unidades de almacenamiento de datos (DWU y cDWU) en Azure SQL Data Warehouse | Microsoft Docs
description: Se incluyen recomendaciones acerca de cómo elegir el número ideal de unidades de almacenamiento de datos (DWUs, cDWUs) para optimizar el precio y el rendimiento y cómo cambiar el número de unidades.
services: sql-data-warehouse
author: happynicolle
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: design
ms.date: 04/17/2018
ms.author: nicw
ms.reviewer: igorstan
ms.openlocfilehash: d71594e745fdd57ba383b98b547cceb0be1b61bc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797835"
---
# <a name="data-warehouse-units-dwus-and-compute-data-warehouse-units-cdwus"></a>Unidades de almacenamiento de datos (DWU) y Unidades de almacenamiento de datos de proceso (cDWU)
Se incluyen recomendaciones acerca de cómo elegir el número ideal de unidades de almacenamiento de datos (DWUs, cDWUs) para optimizar el precio y el rendimiento y cómo cambiar el número de unidades. 

## <a name="what-are-data-warehouse-units"></a>¿Qué son las Unidades de almacenamiento de datos?
Con SQL Data Warehouse, la CPU, la memoria y la E/S se agrupan en unidades de escalado de proceso denominadas Unidades de almacenamiento de datos (DWU). Una DWU representa una medida abstracta y normalizada de recursos de proceso y rendimiento. Al cambiar el nivel de servicio, modifica el número de DWU que se asignan al sistema, y que a su vez ajusta el rendimiento y el costo del sistema. 

Para pagar un mayor rendimiento, puede aumentar el número de unidades de almacenamiento de datos. Para pagar un menor rendimiento, reduzca las unidades de almacenamiento de datos. Los costos de almacenamiento y de proceso se facturan por separado, por lo que cambiar las unidades de almacenamiento de datos no afecta a los costos de almacenamiento.

El rendimiento de las unidades de almacenamiento de datos se basa en estas métricas de carga de trabajo de almacenamiento de datos:

- ¿Con qué rapidez puede una consulta de almacenamiento de datos estándar examinar un gran número de filas y, después, realizar una agregación compleja? Esta es una operación de gran consumo de E/S y de CPU.
- ¿Con qué rapidez el almacenamiento de datos puede ingerir datos de Azure Storage Blob o de Azure Data Lake? Esta es una operación de gran consumo de red y CPU. 
- ¿Con qué rapidez puede el comando T-SQL [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) copiar una tabla? Esta operación implica la lectura de datos del almacenamiento, su distribución entre los nodos del dispositivo y su nueva escritura en el almacenamiento. Esta es una operación de gran consumo de CPU, E/S y red.

Aumentar las DWU:
- Cambia linealmente el rendimiento del sistema para exámenes, agregaciones e instrucciones CTAS.
- Aumenta el número de lectores y escritores para las operaciones de carga de PolyBase.
- Aumenta el número máximo de consultas simultáneas y ranuras de simultaneidad.

## <a name="service-level-objective"></a>Objetivo de nivel de servicio
El objetivo de nivel de servicio (SLO) es la opción de escalabilidad que determina el nivel de costo y el rendimiento del almacenamiento de datos. Los niveles de servicio de Gen2 se miden en unidades de almacenamiento de datos de proceso (cDWU); por ejemplo, DW2000c. Los niveles de servicio de Gen1 se miden en DWU; por ejemplo, DW2000.
  > [!NOTE]
  > Azure SQL Data Warehouse Gen2 ha agregado recientemente funcionalidades de escalado adicionales compatibles con niveles de proceso tan bajos como 100 cDWU. Los almacenes de datos existentes actualmente en Gen1 que requieren los niveles de proceso más bajos ahora pueden actualizarse a Gen2 en las regiones que están actualmente disponibles sin ningún costo adicional.  Si esto no se admite aún en su región, aún puede actualizar a una región admitida. Para más información, consulte [Actualización a Gen2](upgrade-to-latest-generation.md).

En T-SQL, el valor de SERVICE_OBJECTIVE determina el nivel de servicio y el nivel de rendimiento del almacenamiento de datos.

```sql
--Gen1
CREATE DATABASE myElasticSQLDW
WITH
(    SERVICE_OBJECTIVE = 'DW1000'
)
;

--Gen2
CREATE DATABASE myComputeSQLDW
WITH
(    SERVICE_OBJECTIVE = 'DW1000c'
)
;
```

## <a name="performance-tiers-and-data-warehouse-units"></a>Niveles de rendimiento y unidades de almacenamiento de datos

Cada nivel de rendimiento usa una unidad de medida ligeramente diferente para sus unidades de almacenamiento de datos. Esta diferencia se refleja en la factura, ya que la unidad de escala se traduce directamente en la facturación.

- Los almacenamientos de datos de Gen1 se miden en unidades de almacenamiento de datos (DWU).
- Los almacenamientos de datos de Gen2 se miden en unidades de almacenamiento de datos de proceso (cDWU). 

Tanto las DWU como las cDWU admiten el escalado vertical y la reducción vertical del proceso, así como pausar el proceso cuando no es necesario usar el almacén de datos. Estas operaciones son a petición. El nivel Gen2 usa una memoria caché basada en disco local en los nodos de proceso para mejorar el rendimiento. Al escalar o pausar el sistema, se invalida la memoria caché y es necesario un período de calentamiento de la memoria caché para conseguir un rendimiento óptimo.  

A medida que aumente unidades de almacenamiento de datos, también se aumentan linealmente los recursos informáticos. Gen2 proporciona el mejor rendimiento de las consultas y la escala más alta. Estos sistemas hacen el mayor uso de la memoria caché.

### <a name="capacity-limits"></a>Límites de capacidad
Cada servidor SQL Server (por ejemplo, myserver.database.windows.net) tiene una cuota de [unidad de transacción de base de datos (DTU)](../sql-database/sql-database-what-is-a-dtu.md) que permite un número específico de unidades de almacenamiento de datos. Para más información, consulte los [límites de capacidad de administración de cargas de trabajo](sql-data-warehouse-service-capacity-limits.md#workload-management).


## <a name="how-many-data-warehouse-units-do-i-need"></a>¿Cuántas unidades de almacenamiento de datos necesito?
El número ideal de unidades de almacenamiento de datos depende en gran medida de la carga de trabajo y la cantidad de datos que cargó en el sistema.

Pasos para encontrar la mejor DWU para la carga de trabajo:

1. Comience por seleccionar una DWU más pequeña. 
2. Supervise el rendimiento de su aplicación a medida que prueba cargas de datos en el sistema, observando el número de DWU seleccionadas en comparación con el rendimiento que observe.
3. Identifique los requisitos adicionales para períodos de máxima actividad periódicos. Si la carga de trabajo muestra importantes altibajos en la actividad y hay una buena razón para escalar con frecuencia.

SQL Data Warehouse es un sistema de escalado horizontal que puede aprovisionar grandes cantidades de procesos y consultar cantidades considerables de datos. Para ver sus verdaderas capacidades de escalado, especialmente en DWU más grandes, se recomienda escalar el conjunto de datos para asegurar que tiene suficientes datos como para alimentar las CPU. Para probar la escala, se recomienda usar al menos 1 TB.

> [!NOTE]
>
> El rendimiento de las consultas solo aumenta con más paralelización si el trabajo se puede dividir entre nodos de proceso. Si ve que el escalado no cambia el rendimiento, es posible que deba ajustar el diseño de las tablas o de las consultas. Para obtener instrucciones para el ajuste de consultas, vea [Manage user queries](sql-data-warehouse-overview-manage-user-queries.md) (Administración de consultas de usuarios). 

## <a name="permissions"></a>Permisos

Para cambiar las unidades de almacenamiento de datos es necesario disponer de los permisos descritos en [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql). 

Los roles integrados para los recursos de Azure, tales como Colaborador de base de datos SQL y Colaborador de SQL Server, pueden cambiar la configuración de DWU. 

## <a name="view-current-dwu-settings"></a>Ver la configuración de DWU actual

Para ver la configuración actual de DWU:

1. Abra el Explorador de objetos de SQL Server en Visual Studio.
2. Conéctese a la base de datos maestra asociada al servidor de SQL Database lógico.
3. Seleccione en la vista de administración dinámica sys.database_service_objectives. Este es un ejemplo: 

```sql
SELECT  db.name [Database]
,       ds.edition [Edition]
,       ds.service_objective [Service Objective]
FROM    sys.database_service_objectives   AS ds
JOIN    sys.databases                     AS db ON ds.database_id = db.database_id
;
```

## <a name="change-data-warehouse-units"></a>Cambiar unidades de almacenamiento de datos

### <a name="azure-portal"></a>Azure Portal
Para cambiar DWU o cDWU:

1. Abra [Azure Portal](https://portal.azure.com), abra la base de datos y haga clic en **Escalar**.

2. En **Escalar**, mueva el control deslizante izquierdo o derecho para cambiar el valor de DWU.

3. Haga clic en **Save**(Guardar). Aparece un mensaje de confirmación. Haga clic en **Sí** para confirmar o **No** para cancelar.

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para cambiar Dwu o Cdwu, use el [conjunto AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) cmdlet de PowerShell. En el ejemplo siguiente se establece el objetivo de nivel de servicio en DW1000 para la base de datos MySQLDW que se hospeda en el servidor MyServer.

```Powershell
Set-AzSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer" -RequestedServiceObjectiveName "DW1000"
```

Para obtener más información, vea [Cmdlets de PowerShell y API REST para SQL Data Warehouse](sql-data-warehouse-reference-powershell-cmdlets.md).

### <a name="t-sql"></a>T-SQL
Con T-SQL puede ver la configuración actual de DWU o cDWU, modificarla y comprobar el progreso. 

Para cambiar DWU o cDWU:

1. Conéctese a la base de datos maestra asociada al servidor lógico de SQL Database.
2. Utilice la instrucción TSQL [ALTER DATABAS](/sql/t-sql/statements/alter-database-transact-sql) . En el ejemplo siguiente se establece el objetivo de nivel de servicio en DW1000 para la base de datos MySQLDW. 

```Sql
ALTER DATABASE MySQLDW
MODIFY (SERVICE_OBJECTIVE = 'DW1000')
;
```

### <a name="rest-apis"></a>API de REST

Para cambiar las DWU, utilice la API REST [Create or Update Database](/rest/api/sql/databases/createorupdate) (Creación o actualización de base de datos). En el ejemplo siguiente se establece el objetivo de nivel de servicio en DW1000 para la base de datos MySQLDW que se hospeda en el servidor MyServer. El servidor está en un grupo de recursos de Azure denominado ResourceGroup1.

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}?api-version=2014-04-01-preview HTTP/1.1
Content-Type: application/json; charset=UTF-8

{
    "properties": {
        "requestedServiceObjectiveName": DW1000
    }
}
```

Para obtener más ejemplos de API REST, consulte [API REST para Azure SQL Data Warehouse](sql-data-warehouse-manage-compute-rest-api.md).

## <a name="check-status-of-dwu-changes"></a>Comprobar el estado de los cambios de DWU

Los cambios de DWU pueden tardar varios minutos en completarse. Si está realizando una operación de escalado automáticamente, considere implementar la lógica para asegurarse de que ciertas operaciones se completaron antes de pasar a realizar otra acción. 

La comprobación del estado de la base de datos a través de varios puntos de conexión le permitirá implementar correctamente la automatización. El portal le proporcionará una notificación tras la finalización de una operación y el estado actual de las bases de datos, pero no permitirá la comprobación programática del estado. 

No se puede comprobar el estado de la base de datos para las operaciones de escalado horizontal con Azure Portal.

Para comprobar el estado de los cambios de DWU:

1. Conéctese a la base de datos maestra asociada al servidor lógico de SQL Database.
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

Esta DMV devuelve información sobre varias operaciones de administración en SQL Data Warehouse, como la operación y el estado de esta, que es IN_PROGRESS o COMPLETED.

## <a name="the-scaling-workflow"></a>Flujo de trabajo de escalado

Cuando se inicia una operación de escalado, el sistema elimina primero todas las sesiones abiertas y revierte todas las transacciones abiertas para garantizar un estado coherente. Para las operaciones de escalado, el escalado solo se producirá una vez completada esta reversión transaccional.  

- Para una operación de escalado vertical, el sistema aprovisiona el proceso adicional y, después, vuelve a adjuntar la capa de almacenamiento. 
- Para una operación de reducción vertical, los nodos innecesarios se desasocian del almacenamiento y se vuelven a asociar a los nodos restantes.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de cómo administrar el rendimiento, consulte [Clases de recursos para la administración de cargas de trabajo](resource-classes-for-workload-management.md) y [Límites de memoria y simultaneidad](memory-and-concurrency-limits.md).



