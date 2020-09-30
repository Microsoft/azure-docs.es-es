---
title: Clases de recursos para la administración de cargas de trabajo
description: Instrucciones de uso de las clases de recursos para administrar los recursos de proceso y la simultaneidad de las consultas en Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 7c3793daa820d0cb5b5b6900402704756f206425
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89488395"
---
# <a name="workload-management-with-resource-classes-in-azure-synapse-analytics"></a>Administración de cargas de trabajo con clases de recursos en Azure Synapse Analytics

Instrucciones de uso de las clases de recursos para administrar la memoria y la simultaneidad de las consultas del grupo de SQL de Synapse en Azure Synapse.  

## <a name="what-are-resource-classes"></a>¿Qué son las clases de recursos?

La capacidad de rendimiento de una consulta viene determinada por la clase de recursos del usuario.  Las clases de recursos son límites de recursos predeterminados en el grupo de SQL de Synapse que rigen los recursos de proceso y la simultaneidad de la ejecución de las consultas. Las clases de recursos pueden ayudarle a configurar recursos para las consultas mediante el establecimiento de límites en el número de consultas que se ejecutan simultáneamente y en los recursos de proceso que se asignan a cada consulta.  Gracias a esto, la memoria y la simultaneidad se compensan.

- Las clases de recursos más pequeñas reducen la memoria máxima por cada consulta, pero aumentan la simultaneidad.
- Las clases de recursos más grandes aumentan la memoria máxima por cada consulta, pero reducen la simultaneidad.

Existen dos tipos de clases de recursos:

- Las clases de recursos estáticos, que son adecuadas para obtener una mayor simultaneidad en el tamaño de conjunto de datos que se ha fijado.
- Las clases de recursos dinámicos, que son adecuadas para conjuntos de datos cuyo tamaño está en constante crecimiento y necesitan mayor rendimiento a medida que aumenta el nivel de servicio.

Las clases de recursos utilizan espacios de simultaneidad para medir el consumo de recursos.  Los [espacios de simultaneidad](#concurrency-slots) se explican posteriormente en este artículo.

- Para ver el uso de recursos para las clases de recursos, consulte [Límites de memoria y simultaneidad](memory-concurrency-limits.md).
- Para ajustar la clase de recurso, puede ejecutar la consulta bajo un usuario diferente o [cambiar la pertenencia de la clase de recurso del usuario actual](#change-a-users-resource-class).

### <a name="static-resource-classes"></a>Clases de recursos estáticos

Las clases de recursos estáticos asignan la misma cantidad de memoria independientemente del nivel de rendimiento actual, que se mide en [unidades de almacenamiento de datos](what-is-a-data-warehouse-unit-dwu-cdwu.md). Puesto que obtienen la misma asignación de memoria independientemente del nivel de rendimiento, el [escalado horizontal del almacenamiento de datos](quickstart-scale-compute-portal.md) permite la ejecución de más consultas dentro de una clase de recursos.  Las clases de recursos estáticos son ideales si se conoce el volumen de datos y este es constante.

Las clases de recursos estáticos se implementan con estos roles de base de datos predefinidos:

- staticrc10
- staticrc20
- staticrc30
- staticrc40
- staticrc50
- staticrc60
- staticrc70
- staticrc80

### <a name="dynamic-resource-classes"></a>Clases de recursos dinámicos

Las clases de recursos dinámicos asignan una cantidad variable de memoria en función del nivel de servicio actual. Si bien las clases de recursos estáticos son beneficiosas para una mayor simultaneidad y volúmenes de datos estáticos, las clases de recursos dinámicos son más adecuadas para trabajar con una cantidad creciente o variable de datos.  Esto significa que, al escalar a un nivel de servicio mayor, las consultas obtienen más memoria automáticamente.  

Las clases de recursos dinámicos se implementan con estos roles de base de datos predefinidos:

- smallrc
- mediumrc
- largerc
- xlargerc

La asignación de memoria para cada clase de recurso es la siguiente.

| Nivel de servicio  | smallrc           | mediumrc               | largerc                | xlargerc               |
|:--------------:|:-----------------:|:----------------------:|:----------------------:|:----------------------:|
| DW100c         | 25 %               | 25 %                    | 25 %                    | 70%                    |
| DW200c         | 12,5 %             | 12,5 %                  | 22 %                    | 70%                    |
| DW300c         | 8 %                | 10 %                    | 22 %                    | 70%                    |
| DW400c         | 6,25 %             | 10 %                    | 22 %                    | 70%                    |
| DW500c         | 5 %                | 10 %                    | 22 %                    | 70%                    |
| DW1000c a<br> DW30000c | 3 %       | 10 %                    | 22 %                    | 70%                    |

### <a name="default-resource-class"></a>Clase de recursos predeterminada

De forma predeterminada, cada usuario es miembro de la clase de recursos dinámicos **smallrc**.

La clase de recursos del administrador de servicios está fijada en smallrc y no se puede cambiar.  El administrador de servicios es el usuario que se creó durante el proceso de aprovisionamiento.  En este contexto, el administrador de servicios es el inicio de sesión especificado como inicio de sesión del administrador del servidor cuando se crea un nuevo grupo de SQL de Synapse con un nuevo servidor.

> [!NOTE]
> Los usuarios o grupos definidos como administrador de Active Directory también son administradores de servicio.
>
>

## <a name="resource-class-operations"></a>Operaciones de clases de recursos

Las clases de recursos están diseñadas para mejorar el rendimiento de las actividades de administración y manipulación de datos. Las consultas más complejas también pueden beneficiarse de la ejecución en una clase de recursos más grande. Por ejemplo, el rendimiento de las consultas para criterios de combinación y ordenación de gran tamaño puede mejorar cuando la clase de recursos es lo suficientemente grande como para permitir que la consulta se ejecute en la memoria.

### <a name="operations-governed-by-resource-classes"></a>Operaciones regidas por clases de recursos

Estas operaciones están regidas por clases de recursos:

- INSERT-SELECT, UPDATE, DELETE
- SELECT (al consultar las tablas de usuario)
- ALTER INDEX - REBUILD o REORGANIZE
- ALTER TABLE REBUILD
- CREATE INDEX
- CREATE CLUSTERED COLUMNSTORE INDEX
- CREATE TABLE AS SELECT (CTAS)
- Carga de datos
- Operaciones de movimiento de datos llevadas a cabo por el servicio de movimiento de datos (DMS)

> [!NOTE]  
> Ninguno de los límites de simultaneidad se encarga de regular las instrucciones SELECT de las vistas de administración dinámicas (DMV) o de otras vistas del sistema. Puede supervisar el sistema independientemente del número de consultas que se ejecutan en él.
>
>

### <a name="operations-not-governed-by-resource-classes"></a>Operaciones no regidas por clases de recursos

Algunas consultas siempre se ejecutan en la clase de recursos smallrc incluso si el usuario es miembro de una clase de recursos más grande. Estas consultas exentas no se cuentan en el límite de simultaneidad. Por ejemplo, si el límite de simultaneidad es de 16, muchos usuarios pueden realizar selecciones desde las vistas del sistema, sin que esto afecte a las ranuras de simultaneidad disponibles.

Las instrucciones siguientes están exentas de las clases de recursos y siempre se ejecutan en smallrc:

- CREATE o DROP TABLE
- La instrucción ALTER TABLE ... SWITCH, SPLIT o MERGE PARTITION
- ALTER INDEX DISABLE
- DROP INDEX
- CREATE, UPDATE o DROP STATISTICS
- TRUNCATE TABLE
- ALTER AUTHORIZATION
- CREATE LOGIN
- CREATE, ALTER o DROP USER
- CREATE, ALTER o DROP PROCEDURE
- CREATE o DROP VIEW
- INSERT VALUES
- SELECT (desde vistas del sistema y DMV)
- EXPLAIN
- DBCC

<!--
Removed as these two are not confirmed / supported under Azure Synapse Analytics
- CREATE REMOTE TABLE AS SELECT
- CREATE EXTERNAL TABLE AS SELECT
- REDISTRIBUTE
-->

## <a name="concurrency-slots"></a>Espacios de simultaneidad

Los espacios de simultaneidad son una manera cómoda de realizar un seguimiento de los recursos disponibles para la ejecución de la consulta. Son como las entradas que compra para reservar un asiento en un concierto porque estos son limitados. El número total de espacios de simultaneidad por almacenamiento de datos viene determinado por el nivel de servicio. Para que una consulta se pueda ejecutar, debe poder reservar suficientes espacios de simultaneidad. Cuando una consulta finaliza, libera sus intervalos de simultaneidad.  

- Una consulta que se ejecute con 10 espacios de simultaneidad puede tener acceso a 5 veces más recursos de proceso que una consulta que se ejecute con 2 espacios de simultaneidad.
- Si cada consulta requiere 10 espacios de simultaneidad y hay 40, solo se pueden ejecutar 4 consultas simultáneamente.

Solo las consultas regidas por recursos consumen espacios de simultaneidad. Las consultas del sistema y algunas consultas sencillas no consumen ningún espacio. El número exacto de espacios de simultaneidad que se usa está determinado por la clase de recurso de la consulta.

## <a name="view-the-resource-classes"></a>Visualización de las clases de recursos

Las clases de recursos se implementan como roles de base de datos predefinidos. Existen dos tipos de clases de recursos: estáticas y dinámicas. Para ver una lista de las clases de recursos, utilice la siguiente consulta:

```sql
SELECT name
FROM   sys.database_principals
WHERE  name LIKE '%rc%' AND type_desc = 'DATABASE_ROLE';
```

## <a name="change-a-users-resource-class"></a>Cambio de una clase de recursos de usuario

Para implementar clases de recursos, debe asignar usuarios a los roles de base de datos. Cuando un usuario ejecuta una consulta, la consulta se ejecuta con la clase de recurso del usuario. Por ejemplo, si un usuario es miembro del rol de base de datos "staticrc10", las consultas se ejecutan con pequeñas cantidades de memoria. Si un usuario de base de datos es miembro del rol de base de datos "xlargerc" o "staticrc80", las consultas se ejecutan con grandes cantidades de memoria.

Para aumentar la clase de recursos de un usuario, use [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para agregar el usuario a un rol de base de datos de una clase de recursos grande.  El siguiente código agrega un usuario al rol de base de datos "largerc".  Cada solicitud obtiene un 22 % de la memoria del sistema.

```sql
EXEC sp_addrolemember 'largerc', 'loaduser';
```

Para reducir la clase de recursos, use [sp_droprolemember](/sql/relational-databases/system-stored-procedures/sp-droprolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Si "loaduser" no es un miembro o cualquier otra clase de recurso, entran en la clase de recursos "smallrc" predeterminada con una concesión de memoria del 3 %.  

```sql
EXEC sp_droprolemember 'largerc', 'loaduser';
```

## <a name="resource-class-precedence"></a>Prioridad de la clase de recursos

Los usuarios pueden ser miembros de varias clases de recursos. Cuando un usuario pertenece a más de una clase de recursos:

- Las clases de recursos dinámicas tienen prioridad sobre las clases de recursos estáticas. Por ejemplo, si un usuario es miembro de mediumrc (dinámico) y staticrc80 (estático), las consultas se ejecutarán con mediumrc.
- Las clases de recursos más grandes tienen prioridad sobre las clases de recursos más pequeñas. Por ejemplo, si un usuario es miembro de mediumrc y largerc, las consultas se ejecutarán con largerc. Del mismo modo, si un usuario es miembro de staticrc20 y statirc80, las consultas se ejecutarán con las asignaciones de recursos de staticrc80.

## <a name="recommendations"></a>Recomendaciones

>[!NOTE]
>Considere la posibilidad de aprovechar las funcionalidades de administración de cargas de trabajo ([aislamiento de carga de trabajo](sql-data-warehouse-workload-isolation.md), [clasificación](sql-data-warehouse-workload-classification.md) e [importancia](sql-data-warehouse-workload-importance.md)) para obtener un mayor control sobre la carga de trabajo y un rendimiento predecible.  
>
>

Se recomienda crear un usuario que se dedique a ejecutar un tipo específico de consulta u operación de carga. Conceda a ese usuario una clase de recursos permanente en lugar de cambiar la clase de recursos con frecuencia. Las clases de recursos estáticas, en general, proporcionan un mayor control sobre la carga de trabajo, por lo que le recomendamos que use dichas clases antes de plantearse usar las clases de recursos dinámicas.

### <a name="resource-classes-for-load-users"></a>Clases de recursos para los usuarios de carga

`CREATE TABLE` usa índices de almacén de columnas en clúster de manera predeterminada. Comprimir datos en un índice de almacén de columnas es una operación que utiliza mucha memoria y, debido a ello, la presión en la memoria puede reducir la calidad del índice. La presión de memoria puede conducir a la necesidad de una clase de recursos superior al cargar los datos. Para asegurarse de que las cargas tienen suficiente memoria, puede crear un usuario designado para ejecutar cargas y asignarlo a una clase de recursos más alta.

La memoria necesaria para procesar las cargas eficazmente depende de la naturaleza de la tabla cargada y del tamaño de los datos. Para obtener más información sobre los requisitos de memoria, consulte [Maximizing rowgroup quality (Maximizar la calidad de un grupo de filas)](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).

Una vez que haya determinado los requisitos de la memoria, elija si quiere asignar el usuario de carga a una clase de recursos estática o dinámica.

- Use una clase de recursos estática cuando los requisitos de memoria de la tabla se encuentren dentro de un intervalo específico. Las cargas se deben ejecutar con la memoria adecuada. Asimismo, al escalar el almacenamiento de datos, las cargas no necesitan más memoria. Mediante el uso de una clase de recursos estática, las asignaciones de memoria permanecen constantes. Esta coherencia ahorra memoria y permite que se ejecuten más consultas simultáneamente. Le recomendamos que use las clases de recursos estáticas con las nuevas soluciones, ya que estas proporcionan un mayor control.
- Puede usar una clase de recursos dinámica si los requisitos de memoria de tabla varían considerablemente. Es posible que las cargas necesiten más memoria que la que proporciona el nivel DWU o cDWU actual. El escalado del almacenamiento de datos agrega más memoria a las operaciones de carga, lo que permite agilizar los procesos de carga.

### <a name="resource-classes-for-queries"></a>Clases de recursos para consultas

Algunas consultas son procesos intensivos y otras no.  

- Seleccione una clase de recursos dinámica cuando las consultas sean complejas, pero que no necesiten una simultaneidad alta.  Por ejemplo, el proceso de generar informes diarios o semanales es una necesidad ocasional de recursos. Si los informes procesan grandes cantidades de datos, al escalar el almacenamiento de datos se proporcionará más memoria a la clase de recursos existente del usuario.
- Seleccione una clase de recursos estática cuando varíen las expectativas de recursos a lo largo del día. Por ejemplo, una clase de recursos estática funciona bien cuando varios usuarios consultan el almacenamiento de datos. Al escalar el almacenamiento de datos, no cambia la cantidad de memoria asignada al usuario. Por lo tanto, se pueden ejecutar más consultas en paralelo en el sistema.

Las concesiones de memoria adecuadas dependen de varios factores como, por ejemplo, la cantidad de datos consultados, la naturaleza de los esquemas de tabla y los diversos predicados de grupo, selección y combinación. Desde un punto de vista general, si asigna más memoria permitirá que las consultas se completen más rápido, pero podría reducir la simultaneidad global. Si la simultaneidad no es un problema, asignar una cantidad de memoria mayor de la necesaria no resulta perjudicial para el rendimiento.

Para optimizar el rendimiento, utilice las diferentes clases de recursos. En la siguiente sección se proporciona un procedimiento almacenado que le ayudará a averiguar cuál es la mejor clase de recursos.

## <a name="example-code-for-finding-the-best-resource-class"></a>Código de ejemplo para encontrar la mejor clase de recursos

Puede usar el siguiente procedimiento almacenado especificado para averiguar la concesión de memoria y simultaneidad por clase de recurso en un SLO determinado y la clase de recurso recomendada para operaciones de CCI que usen mucho la memoria en la tabla CCI sin particiones en una clase de recurso determinado:

Este es el propósito de este procedimiento almacenado:

1. Para ver la simultaneidad y la concesión de memoria por clase de recursos en un SLO determinado. El usuario tiene que proporcionar un valor NULL tanto para el esquema como para el nombre de tabla, tal como se muestra en este ejemplo.  
2. Para ver cuál es la clase de recursos más adecuada para las operaciones de CCI que usan mucha memoria (carga, tabla de copia, regeneración de índices, etc.) en la tabla CCI sin particiones con una clase de recursos determinada. En este caso, el procedimiento almacenado usa el esquema de tabla para averiguar la concesión de memoria necesaria.

### <a name="dependencies--restrictions"></a>Dependencias y restricciones

- Este procedimiento almacenado no está diseñado para calcular los requisitos de memoria de una tabla CCI con particiones.
- Este procedimiento almacenado no tiene en cuenta los requisitos de memoria de la parte SELECT de CTAS/INSERT-SELECT y se da por supuesto que es una instrucción SELECT.
- Este procedimiento almacenado utiliza una tabla temporal que está disponible en la sesión donde el procedimiento almacenado se creó.
- Este procedimiento almacenado depende de las ofertas actuales (por ejemplo, configuración de hardware, configuración DMS) y, si algo cambiara, no funcionará correctamente.  
- Este procedimiento almacenado depende de las ofertas del límite de simultaneidad existentes y, si estas cambian, dicho procedimiento no funcioná correctamente.  
- Este procedimiento almacenado depende de las ofertas de la clase de recursos existentes y, si estas cambian, dicho procedimiento no funcioná correctamente.  

>[!NOTE]  
>Si no se obtienen resultados después de ejecutar el procedimiento almacenado con los parámetros proporcionados, podrían darse dos circunstancias.
>
>1. Que algún parámetro del almacenamiento de datos contenga un valor de SLO no válido.
>2. O bien, que no haya ninguna clase de recursos coincidente de la operación CCI en la tabla.
>
>Por ejemplo, en DW100c, la concesión de memoria máxima disponible es de 1 GB y el esquema de tabla es lo suficientemente ancho como para superar el requisito de 1 GB.

### <a name="usage-example"></a>Ejemplo de uso

Sintaxis:  
`EXEC dbo.prc_workload_management_by_DWU @DWU VARCHAR(7), @SCHEMA_NAME VARCHAR(128), @TABLE_NAME VARCHAR(128)`
  
1. @DWU: proporcione un parámetro NULL para extraer la unidad de almacenamiento de datos actual de la base de datos de almacenamiento de datos o proporcione una unidad admitida con el formato 'DW100c'
2. @SCHEMA_NAME: proporcione un nombre de esquema de la tabla
3. @TABLE_NAME: proporcione un nombre de tabla del interés

Ejemplos de ejecución de este procedimiento almacenado:

```sql
EXEC dbo.prc_workload_management_by_DWU 'DW2000c', 'dbo', 'Table1';  
EXEC dbo.prc_workload_management_by_DWU NULL, 'dbo', 'Table1';  
EXEC dbo.prc_workload_management_by_DWU 'DW6000c', NULL, NULL;  
EXEC dbo.prc_workload_management_by_DWU NULL, NULL, NULL;  
```

La siguiente instrucción crea la tabla "Table1" que se usa en los ejemplos anteriores.
`CREATE TABLE Table1 (a int, b varchar(50), c decimal (18,10), d char(10), e varbinary(15), f float, g datetime, h date);`

### <a name="stored-procedure-definition"></a>Definición del procedimiento almacenado

```sql
-------------------------------------------------------------------------------
-- Dropping prc_workload_management_by_DWU procedure if it exists.
-------------------------------------------------------------------------------
IF EXISTS (SELECT * FROM sys.objects WHERE type = 'P' AND name = 'prc_workload_management_by_DWU')
DROP PROCEDURE dbo.prc_workload_management_by_DWU
GO

-------------------------------------------------------------------------------
-- Creating prc_workload_management_by_DWU.
-------------------------------------------------------------------------------
CREATE PROCEDURE dbo.prc_workload_management_by_DWU
(@DWU VARCHAR(7),
 @SCHEMA_NAME VARCHAR(128),
 @TABLE_NAME VARCHAR(128)
)
AS

IF @DWU IS NULL
BEGIN
-- Selecting proper DWU for the current DB if not specified.

SELECT @DWU = 'DW'+ CAST(CASE WHEN Mem> 4 THEN Nodes*500
  ELSE Mem*100
  END AS VARCHAR(10)) +'c'
    FROM (
      SELECT Nodes=count(distinct n.pdw_node_id), Mem=max(i.committed_target_kb/1000/1000/60)
        FROM sys.dm_pdw_nodes n
        CROSS APPLY sys.dm_pdw_nodes_os_sys_info i
        WHERE type = 'COMPUTE')A
END

-- Dropping temp table if exists.
IF OBJECT_ID('tempdb..#ref') IS NOT NULL
BEGIN
  DROP TABLE #ref;
END;

-- Creating ref. temp table (CTAS) to hold mapping info.
CREATE TABLE #ref
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
WITH
-- Creating concurrency slots mapping for various DWUs.
alloc
AS
(
SELECT 'DW100c' AS DWU,4 AS max_queries,4 AS max_slots,1 AS slots_used_smallrc,1 AS slots_used_mediumrc,2 AS slots_used_largerc,4 AS slots_used_xlargerc,1 AS slots_used_staticrc10,2 AS slots_used_staticrc20,4 AS slots_used_staticrc30,4 AS slots_used_staticrc40,4 AS slots_used_staticrc50,4 AS slots_used_staticrc60,4 AS slots_used_staticrc70,4 AS slots_used_staticrc80
  UNION ALL
   SELECT 'DW200c',8,8,1,2,4,8,1,2,4,8,8,8,8,8
  UNION ALL
   SELECT 'DW300c',12,12,1,2,4,8,1,2,4,8,8,8,8,8
  UNION ALL
   SELECT 'DW400c',16,16,1,4,8,16,1,2,4,8,16,16,16,16
  UNION ALL
   SELECT 'DW500c',20,20,1,4,8,16,1,2,4,8,16,16,16,16
  UNION ALL
   SELECT 'DW1000c',32,40,1,4,8,28,1,2,4,8,16,32,32,32
  UNION ALL
   SELECT 'DW1500c',32,60,1,6,13,42,1,2,4,8,16,32,32,32
  UNION ALL
   SELECT 'DW2000c',48,80,2,8,17,56,1,2,4,8,16,32,64,64
  UNION ALL
   SELECT 'DW2500c',48,100,3,10,22,70,1,2,4,8,16,32,64,64
  UNION ALL
   SELECT 'DW3000c',64,120,3,12,26,84,1,2,4,8,16,32,64,64
  UNION ALL
   SELECT 'DW5000c',64,200,6,20,44,140,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW6000c',128,240,7,24,52,168,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW7500c',128,300,9,30,66,210,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW10000c',128,400,12,40,88,280,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW15000c',128,600,18,60,132,420,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW30000c',128,1200,36,120,264,840,1,2,4,8,16,32,64,128
)
-- Creating workload mapping to their corresponding slot consumption and default memory grant.
,map  
AS
(
  SELECT CONVERT(varchar(20), 'SloDWGroupSmall') AS wg_name, slots_used_smallrc AS slots_used FROM alloc WHERE DWU = @DWU
UNION ALL
  SELECT CONVERT(varchar(20), 'SloDWGroupMedium') AS wg_name, slots_used_mediumrc AS slots_used FROM alloc WHERE DWU = @DWU
UNION ALL
  SELECT CONVERT(varchar(20), 'SloDWGroupLarge') AS wg_name, slots_used_largerc AS slots_used FROM alloc WHERE DWU = @DWU
UNION ALL
  SELECT CONVERT(varchar(20), 'SloDWGroupXLarge') AS wg_name, slots_used_xlargerc AS slots_used FROM alloc WHERE DWU = @DWU
  UNION ALL
  SELECT 'SloDWGroupC00',1
  UNION ALL
    SELECT 'SloDWGroupC01',2
  UNION ALL
    SELECT 'SloDWGroupC02',4
  UNION ALL
    SELECT 'SloDWGroupC03',8
  UNION ALL
    SELECT 'SloDWGroupC04',16
  UNION ALL
    SELECT 'SloDWGroupC05',32
  UNION ALL
    SELECT 'SloDWGroupC06',64
  UNION ALL
    SELECT 'SloDWGroupC07',128
)

-- Creating ref based on current / asked DWU.
, ref
AS
(
  SELECT  a1.*
  ,       m1.wg_name          AS wg_name_smallrc
  ,       m1.slots_used * 250 AS tgt_mem_grant_MB_smallrc
  ,       m2.wg_name          AS wg_name_mediumrc
  ,       m2.slots_used * 250 AS tgt_mem_grant_MB_mediumrc
  ,       m3.wg_name          AS wg_name_largerc
  ,       m3.slots_used * 250 AS tgt_mem_grant_MB_largerc
  ,       m4.wg_name          AS wg_name_xlargerc
  ,       m4.slots_used * 250 AS tgt_mem_grant_MB_xlargerc
  ,       m5.wg_name          AS wg_name_staticrc10
  ,       m5.slots_used * 250 AS tgt_mem_grant_MB_staticrc10
  ,       m6.wg_name          AS wg_name_staticrc20
  ,       m6.slots_used * 250 AS tgt_mem_grant_MB_staticrc20
  ,       m7.wg_name          AS wg_name_staticrc30
  ,       m7.slots_used * 250 AS tgt_mem_grant_MB_staticrc30
  ,       m8.wg_name          AS wg_name_staticrc40
  ,       m8.slots_used * 250 AS tgt_mem_grant_MB_staticrc40
  ,       m9.wg_name          AS wg_name_staticrc50
  ,       m9.slots_used * 250 AS tgt_mem_grant_MB_staticrc50
  ,       m10.wg_name          AS wg_name_staticrc60
  ,       m10.slots_used * 250 AS tgt_mem_grant_MB_staticrc60
  ,       m11.wg_name          AS wg_name_staticrc70
  ,       m11.slots_used * 250 AS tgt_mem_grant_MB_staticrc70
  ,       m12.wg_name          AS wg_name_staticrc80
  ,       m12.slots_used * 250 AS tgt_mem_grant_MB_staticrc80
  FROM alloc a1
  JOIN map   m1  ON a1.slots_used_smallrc     = m1.slots_used and m1.wg_name = 'SloDWGroupSmall'
  JOIN map   m2  ON a1.slots_used_mediumrc    = m2.slots_used and m2.wg_name = 'SloDWGroupMedium'
  JOIN map   m3  ON a1.slots_used_largerc     = m3.slots_used and m3.wg_name = 'SloDWGroupLarge'
  JOIN map   m4  ON a1.slots_used_xlargerc    = m4.slots_used and m4.wg_name = 'SloDWGroupXLarge'
  JOIN map   m5  ON a1.slots_used_staticrc10    = m5.slots_used and m5.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m6  ON a1.slots_used_staticrc20    = m6.slots_used and m6.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m7  ON a1.slots_used_staticrc30    = m7.slots_used and m7.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m8  ON a1.slots_used_staticrc40    = m8.slots_used and m8.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m9  ON a1.slots_used_staticrc50    = m9.slots_used and m9.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m10  ON a1.slots_used_staticrc60    = m10.slots_used and m10.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m11  ON a1.slots_used_staticrc70    = m11.slots_used and m11.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m12  ON a1.slots_used_staticrc80    = m12.slots_used and m12.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  WHERE   a1.DWU = @DWU
)
SELECT  DWU
,       max_queries
,       max_slots
,       slots_used
,       wg_name
,       tgt_mem_grant_MB
,       up1 as rc
,       (ROW_NUMBER() OVER(PARTITION BY DWU ORDER BY DWU)) as rc_id
FROM
(
    SELECT  DWU
    ,       max_queries
    ,       max_slots
    ,       slots_used
    ,       wg_name
    ,       tgt_mem_grant_MB
    ,       REVERSE(SUBSTRING(REVERSE(wg_names),1,CHARINDEX('_',REVERSE(wg_names),1)-1)) as up1
    ,       REVERSE(SUBSTRING(REVERSE(tgt_mem_grant_MBs),1,CHARINDEX('_',REVERSE(tgt_mem_grant_MBs),1)-1)) as up2
    ,       REVERSE(SUBSTRING(REVERSE(slots_used_all),1,CHARINDEX('_',REVERSE(slots_used_all),1)-1)) as up3
    FROM    ref AS r1
    UNPIVOT
    (
        wg_name FOR wg_names IN (wg_name_smallrc,wg_name_mediumrc,wg_name_largerc,wg_name_xlargerc,
        wg_name_staticrc10, wg_name_staticrc20, wg_name_staticrc30, wg_name_staticrc40, wg_name_staticrc50,
        wg_name_staticrc60, wg_name_staticrc70, wg_name_staticrc80)
    ) AS r2
    UNPIVOT
    (
        tgt_mem_grant_MB FOR tgt_mem_grant_MBs IN (tgt_mem_grant_MB_smallrc,tgt_mem_grant_MB_mediumrc,
        tgt_mem_grant_MB_largerc,tgt_mem_grant_MB_xlargerc, tgt_mem_grant_MB_staticrc10, tgt_mem_grant_MB_staticrc20,
        tgt_mem_grant_MB_staticrc30, tgt_mem_grant_MB_staticrc40, tgt_mem_grant_MB_staticrc50,
        tgt_mem_grant_MB_staticrc60, tgt_mem_grant_MB_staticrc70, tgt_mem_grant_MB_staticrc80)
    ) AS r3
    UNPIVOT
    (
        slots_used FOR slots_used_all IN (slots_used_smallrc,slots_used_mediumrc,slots_used_largerc,
        slots_used_xlargerc, slots_used_staticrc10, slots_used_staticrc20, slots_used_staticrc30,
        slots_used_staticrc40, slots_used_staticrc50, slots_used_staticrc60, slots_used_staticrc70,
        slots_used_staticrc80)
    ) AS r4
) a
WHERE   up1 = up2
AND     up1 = up3
;

-- Getting current info about workload groups.
WITH  
dmv  
AS  
(
  SELECT
          rp.name                                           AS rp_name
  ,       rp.max_memory_kb*1.0/1048576                      AS rp_max_mem_GB
  ,       (rp.max_memory_kb*1.0/1024)
          *(request_max_memory_grant_percent/100)           AS max_memory_grant_MB
  ,       (rp.max_memory_kb*1.0/1048576)
          *(request_max_memory_grant_percent/100)           AS max_memory_grant_GB
  ,       wg.name                                           AS wg_name
  ,       wg.importance                                     AS importance
  ,       wg.request_max_memory_grant_percent               AS request_max_memory_grant_percent
  FROM    sys.dm_pdw_nodes_resource_governor_workload_groups wg
  JOIN    sys.dm_pdw_nodes_resource_governor_resource_pools rp    ON  wg.pdw_node_id  = rp.pdw_node_id
                                                                  AND wg.pool_id      = rp.pool_id
  WHERE   rp.name = 'SloDWPool'
  GROUP BY
          rp.name
  ,       rp.max_memory_kb
  ,       wg.name
  ,       wg.importance
  ,       wg.request_max_memory_grant_percent
)
-- Creating resource class name mapping.
,names
AS
(
  SELECT 'smallrc' as resource_class, 1 as rc_id
  UNION ALL
    SELECT 'mediumrc', 2
  UNION ALL
    SELECT 'largerc', 3
  UNION ALL
    SELECT 'xlargerc', 4
  UNION ALL
    SELECT 'staticrc10', 5
  UNION ALL
    SELECT 'staticrc20', 6
  UNION ALL
    SELECT 'staticrc30', 7
  UNION ALL
    SELECT 'staticrc40', 8
  UNION ALL
    SELECT 'staticrc50', 9
  UNION ALL
    SELECT 'staticrc60', 10
  UNION ALL
    SELECT 'staticrc70', 11
  UNION ALL
    SELECT 'staticrc80', 12
)
,base AS
(   SELECT  schema_name
    ,       table_name
    ,       SUM(column_count)                   AS column_count
    ,       ISNULL(SUM(short_string_column_count),0)   AS short_string_column_count
    ,       ISNULL(SUM(long_string_column_count),0)    AS long_string_column_count
    FROM    (   SELECT  sm.name                                             AS schema_name
                ,       tb.name                                             AS table_name
                ,       COUNT(co.column_id)                                 AS column_count
                           ,       CASE    WHEN co.system_type_id IN (36,43,106,108,165,167,173,175,231,239)
                                AND  co.max_length <= 32
                                THEN COUNT(co.column_id)
                        END                                                 AS short_string_column_count
                ,       CASE    WHEN co.system_type_id IN (165,167,173,175,231,239)
                                AND  co.max_length > 32 and co.max_length <=8000
                                THEN COUNT(co.column_id)
                        END                                                 AS long_string_column_count
                FROM    sys.schemas AS sm
                JOIN    sys.tables  AS tb   on sm.[schema_id] = tb.[schema_id]
                JOIN    sys.columns AS co   ON tb.[object_id] = co.[object_id]
                           WHERE tb.name = @TABLE_NAME AND sm.name = @SCHEMA_NAME
                GROUP BY sm.name
                ,        tb.name
                ,        co.system_type_id
                ,        co.max_length            ) a
GROUP BY schema_name
,        table_name
)
, size AS
(
SELECT  schema_name
,       table_name
,       75497472                                            AS table_overhead

,       column_count*1048576*8                              AS column_size
,       short_string_column_count*1048576*32                       AS short_string_size,       (long_string_column_count*16777216) AS long_string_size
FROM    base
UNION
SELECT CASE WHEN COUNT(*) = 0 THEN 'EMPTY' END as schema_name
         ,CASE WHEN COUNT(*) = 0 THEN 'EMPTY' END as table_name
         ,CASE WHEN COUNT(*) = 0 THEN 0 END as table_overhead
         ,CASE WHEN COUNT(*) = 0 THEN 0 END as column_size
         ,CASE WHEN COUNT(*) = 0 THEN 0 END as short_string_size

,CASE WHEN COUNT(*) = 0 THEN 0 END as long_string_size
FROM   base
)
, load_multiplier as
(
SELECT  CASE
          WHEN FLOOR(8 * (CAST (CAST(REPLACE(REPLACE(@DWU,'DW',''),'c','') AS INT) AS FLOAT)/6000)) > 0
            AND CHARINDEX(@DWU,'c')=0
          THEN FLOOR(8 * (CAST (CAST(REPLACE(REPLACE(@DWU,'DW',''),'c','') AS INT) AS FLOAT)/6000))
          ELSE 1
        END AS multiplication_factor
)
       SELECT  r1.DWU
       , schema_name
       , table_name
       , rc.resource_class as closest_rc_in_increasing_order
       , max_queries_at_this_rc = CASE
             WHEN (r1.max_slots / r1.slots_used > r1.max_queries)
                  THEN r1.max_queries
             ELSE r1.max_slots / r1.slots_used
                  END
       , r1.max_slots as max_concurrency_slots
       , r1.slots_used as required_slots_for_the_rc
       , r1.tgt_mem_grant_MB  as rc_mem_grant_MB
       , CAST((table_overhead*1.0+column_size+short_string_size+long_string_size)*multiplication_factor/1048576    AS DECIMAL(18,2)) AS est_mem_grant_required_for_cci_operation_MB
       FROM    size
       , load_multiplier
       , #ref r1, names  rc
       WHERE r1.rc_id=rc.rc_id
                     AND CAST((table_overhead*1.0+column_size+short_string_size+long_string_size)*multiplication_factor/1048576    AS DECIMAL(18,2)) < r1.tgt_mem_grant_MB
       ORDER BY ABS(CAST((table_overhead*1.0+column_size+short_string_size+long_string_size)*multiplication_factor/1048576    AS DECIMAL(18,2)) - r1.tgt_mem_grant_MB)
GO
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo administrar los usuarios y la seguridad de la base de datos, consulte [Protección de una base de datos en Synapse SQL](sql-data-warehouse-overview-manage-security.md). Para obtener más información sobre la manera en que las clases de recursos más grandes pueden mejorar la calidad de los índices de almacén de columnas en clúster, consulte [Memory optimizations for columnstore compression (Optimizaciones de memoria para la compresión del almacén de columnas)](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).
