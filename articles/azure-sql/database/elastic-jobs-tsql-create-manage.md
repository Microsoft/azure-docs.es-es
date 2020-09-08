---
title: Creación y administración de trabajos de Base de datos elástica (versión preliminar) con Transact-SQL (T-SQL)
description: Ejecute scripts en muchas bases de datos con el agente de trabajo de Elastic Database mediante Transact-SQL (T-SQL).
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
ms.author: jaredmoo
author: jaredmoo
ms.reviewer: sstein
ms.date: 02/07/2020
ms.openlocfilehash: 339b66310d75e228bc6107635ac39b8d27d774c1
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119120"
---
# <a name="use-transact-sql-t-sql-to-create-and-manage-elastic-database-jobs-preview"></a>Use Transact-SQL (T-SQL) para crear y administrar trabajos de Base de datos elástica (versión preliminar)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se proporcionan muchos escenarios de ejemplo para empezar a trabajar con trabajos elásticos mediante T-SQL.

En los ejemplos se utilizan los [procedimientos almacenados](#job-stored-procedures) y las [vistas](#job-views) disponibles en la [*base de datos de trabajo*](job-automation-overview.md#job-database).

Transact-SQL (T-SQL) se utiliza para crear, configurar, ejecutar y administrar los trabajos. La creación del agente de trabajos elásticos no se admite en T-SQL, por lo que debe crear primero un *agente de trabajos elásticos* mediante el portal o [PowerShell](elastic-jobs-powershell-create.md#create-the-elastic-job-agent).

## <a name="create-a-credential-for-job-execution"></a>Creación de una credencial para la ejecución del trabajo

La credencial se usa para conectarse a las bases de datos de destino para la ejecución del script. La credencial necesita permisos adecuados en las bases de datos especificadas por el grupo de destino para ejecutar correctamente el script. Cuando se utiliza un [servidor SQL lógico](logical-servers.md) o un miembro del grupo de destino del grupo, se recomienda encarecidamente crear una credencial maestra para actualizar la credencial antes de la expansión del grupo o del servidor en el momento de ejecución del trabajo. La credencial de ámbito de base de datos se crea en la base de datos del agente de trabajo. La misma credencial debe usarse para *crear un inicio de sesión* y *crear un usuario desde el inicio de sesión para conceder los permisos de base de datos de inicio de sesión* en las bases de datos de destino.

```sql
--Connect to the job database specified when creating the job agent

-- Create a db master key if one does not already exist, using your own password.  
CREATE MASTER KEY ENCRYPTION BY PASSWORD='<EnterStrongPasswordHere>';  
  
-- Create a database scoped credential.  
CREATE DATABASE SCOPED CREDENTIAL myjobcred WITH IDENTITY = 'jobcred',
    SECRET = '<EnterStrongPasswordHere>';
GO

-- Create a database scoped credential for the master database of server1.
CREATE DATABASE SCOPED CREDENTIAL mymastercred WITH IDENTITY = 'mastercred',
    SECRET = '<EnterStrongPasswordHere>';
GO
```

## <a name="create-a-target-group-servers"></a>Creación de un grupo de destino (servidores)

En el ejemplo siguiente se muestra cómo ejecutar un trabajo en todas las bases de datos de un servidor.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
-- Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group 'ServerGroup1'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
'ServerGroup1',
@target_type = 'SqlServer',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in a server
@server_name='server1.database.windows.net'

--View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name='ServerGroup1';
SELECT * FROM jobs.target_group_members WHERE target_group_name='ServerGroup1';
```

## <a name="exclude-an-individual-database"></a>Exclusión de una base de datos individual

En el ejemplo siguiente se muestra cómo ejecutar un trabajo en todas las bases de datos de un servidor, excepto en la denominada *MappingDB*.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC [jobs].sp_add_target_group N'ServerGroup'
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in a server
@server_name=N'London.database.windows.net'
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in a server
@server_name='server2.database.windows.net'
GO

--Exclude a database target member from the server target group
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@membership_type = N'Exclude',
@target_type = N'SqlDatabase',
@server_name = N'server1.database.windows.net',
@database_name =N'MappingDB'
GO

--View the recently created target group and target group members
SELECT * FROM [jobs].target_groups WHERE target_group_name = N'ServerGroup';
SELECT * FROM [jobs].target_group_members WHERE target_group_name = N'ServerGroup';
```

## <a name="create-a-target-group-pools"></a>Creación de un grupo de destino (grupos)

En el ejemplo siguiente se muestra cómo establecer como destino todas las bases de datos de uno o varios grupos elásticos.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing pool(s)
EXEC jobs.sp_add_target_group 'PoolGroup'

-- Add an elastic pool(s) target member
EXEC jobs.sp_add_target_group_member
'PoolGroup',
@target_type = 'SqlElasticPool',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in a server
@server_name='server1.database.windows.net',
@elastic_pool_name='ElasticPool-1'

-- View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name = N'PoolGroup';
SELECT * FROM jobs.target_group_members WHERE target_group_name = N'PoolGroup';
```

## <a name="deploy-new-schema-to-many-databases"></a>Implementación del nuevo esquema en muchas bases de datos

En el ejemplo siguiente se muestra cómo implementar el nuevo esquema en todas las bases de datos.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

--Add job for create table
EXEC jobs.sp_add_job @job_name='CreateTableTest', @description='Create Table Test'

-- Add job step for create table
EXEC jobs.sp_add_jobstep @job_name='CreateTableTest',
@command=N'IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id(''Test''))
CREATE TABLE [dbo].[Test]([TestId] [int] NOT NULL);',
@credential_name='myjobcred',
@target_group_name='PoolGroup'
```

## <a name="data-collection-using-built-in-parameters"></a>Recopilación de datos mediante los parámetros integrados

En muchos escenarios de recopilación de datos puede ser útil incluir algunas de estas variables de scripting para ayudar al posprocesamiento de los resultados del trabajo.

- $(job_name)
- $(job_id)
- $(job_version)
- $(step_id)
- $(step_name)
- $(job_execution_id)
- $(job_execution_create_time)
- $(target_group_name)

Por ejemplo, para agrupar todos los resultados de la misma ejecución del trabajo, use la variable *$(job_execution_id)* tal como se muestra en el siguiente comando:

```sql
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());'
```

## <a name="monitor-database-performance"></a>Supervisión del rendimiento de una base de datos

En el ejemplo siguiente se crea un nuevo trabajo para recopilar datos de rendimiento de varias bases de datos.

De forma predeterminada, el agente de trabajo crea la tabla de salida para almacenar los resultados devueltos. Por lo tanto, la entidad de seguridad de base de datos asociada con la credencial de salida debe tener, como mínimo, los siguientes permisos: `CREATE TABLE` en la base de datos, `ALTER`, `SELECT`, `INSERT` y `DELETE` en la tabla de salida o su esquema y `SELECT` en la vista de catálogo [sys.indexes](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql).

Si desea crear manualmente la tabla antes de tiempo, debe tener las siguientes propiedades:

1. Columnas con el nombre y los tipos de datos correctos para el conjunto de resultados.
2. Columna adicional para internal_execution_id con el tipo de datos de uniqueidentifier.
3. Un índice no agrupado denominado `IX_<TableName>_Internal_Execution_ID` en la columna internal_execution_id.
4. Todos los permisos enumerados anteriormente, salvo el permiso `CREATE TABLE` en la base de datos.

Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute los siguientes comandos:

```sql
--Connect to the job database specified when creating the job agent

-- Add a job to collect perf results
EXEC jobs.sp_add_job @job_name ='ResultsJob', @description='Collection Performance data from all customers'

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsJob',
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());',
@credential_name='myjobcred',
@target_group_name='PoolGroup',
@output_type='SqlDatabase',
@output_credential_name='myjobcred',
@output_server_name='server1.database.windows.net',
@output_database_name='<resultsdb>',
@output_table_name='<resutlstable>'
Create a job to monitor pool performance
--Connect to the job database specified when creating the job agent

-- Add a target group containing master database
EXEC jobs.sp_add_target_group 'MasterGroup'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name='MasterGroup',
@target_type='SqlDatabase',
@server_name='server1.database.windows.net',
@database_name='master'

-- Add a job to collect perf results
EXEC jobs.sp_add_job
@job_name='ResultsPoolsJob',
@description='Demo: Collection Performance data from all pools',
@schedule_interval_type='Minutes',
@schedule_interval_count=15

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsPoolsJob',
@command=N'declare @now datetime
DECLARE @startTime datetime
DECLARE @endTime datetime
DECLARE @poolLagMinutes datetime
DECLARE @poolStartTime datetime
DECLARE @poolEndTime datetime
SELECT @now = getutcdate ()
SELECT @startTime = dateadd(minute, -15, @now)
SELECT @endTime = @now
SELECT @poolStartTime = dateadd(minute, -30, @startTime)
SELECT @poolEndTime = dateadd(minute, -30, @endTime)

SELECT elastic_pool_name , end_time, elastic_pool_dtu_limit, avg_cpu_percent, avg_data_io_percent, avg_log_write_percent, max_worker_percent, max_session_percent,
        avg_storage_percent, elastic_pool_storage_limit_mb FROM sys.elastic_pool_resource_stats
        WHERE end_time > @poolStartTime and end_time <= @poolEndTime;
'),
@credential_name='myjobcred',
@target_group_name='MasterGroup',
@output_type='SqlDatabase',
@output_credential_name='myjobcred',
@output_server_name='server1.database.windows.net',
@output_database_name='resultsdb',
@output_table_name='resutlstable'
```

## <a name="view-job-definitions"></a>Visualización de definiciones de trabajos

En el ejemplo siguiente se muestra cómo ver las definiciones de trabajos actuales.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

-- View all jobs
SELECT * FROM jobs.jobs

-- View the steps of the current version of all jobs
SELECT js.* FROM jobs.jobsteps js
JOIN jobs.jobs j
  ON j.job_id = js.job_id AND j.job_version = js.job_version

-- View the steps of all versions of all jobs
select * from jobs.jobsteps
```

## <a name="begin-unplanned-execution-of-a-job"></a>Inicio de la ejecución no planeada de un trabajo

En el ejemplo siguiente se muestra cómo iniciar un trabajo inmediatamente.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

-- Execute the latest version of a job
EXEC jobs.sp_start_job 'CreateTableTest'

-- Execute the latest version of a job and receive the execution id
declare @je uniqueidentifier
exec jobs.sp_start_job 'CreateTableTest', @job_execution_id = @je output
select @je

select * from jobs.job_executions where job_execution_id = @je

-- Execute a specific version of a job (e.g. version 1)
exec jobs.sp_start_job 'CreateTableTest', 1
```

## <a name="schedule-execution-of-a-job"></a>Programación de la ejecución de un trabajo

El ejemplo siguiente muestra cómo programar un trabajo para una ejecución futura.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_update_job
@job_name='ResultsJob',
@enabled=1,
@schedule_interval_type='Minutes',
@schedule_interval_count=15
```

## <a name="monitor-job-execution-status"></a>Supervisión del estado de ejecución de trabajos

En el ejemplo siguiente se muestra cómo ver los detalles del estado de ejecución de todos los trabajos.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

--View top-level execution status for the job named 'ResultsPoolJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'ResultsPoolsJob' and step_id IS NULL
ORDER BY start_time DESC

--View all top-level execution status for all jobs
SELECT * FROM jobs.job_executions WHERE step_id IS NULL
ORDER BY start_time DESC

--View all execution statuses for job named 'ResultsPoolsJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'ResultsPoolsJob'
ORDER BY start_time DESC

-- View all active executions
SELECT * FROM jobs.job_executions
WHERE is_active = 1
ORDER BY start_time DESC
```

## <a name="cancel-a-job"></a>Cancelación de un trabajo

En el ejemplo siguiente se muestra cómo cancelar un trabajo.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

-- View all active executions to determine job execution id
SELECT * FROM jobs.job_executions
WHERE is_active = 1 AND job_name = 'ResultPoolsJob'
ORDER BY start_time DESC
GO

-- Cancel job execution with the specified job execution id
EXEC jobs.sp_stop_job '01234567-89ab-cdef-0123-456789abcdef'
```

## <a name="delete-old-job-history"></a>Eliminación del historial de trabajos antiguos

En el ejemplo siguiente se muestra cómo eliminar el historial de trabajos anteriores a una fecha concreta.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

-- Delete history of a specific job’s executions older than the specified date
EXEC jobs.sp_purge_jobhistory @job_name='ResultPoolsJob', @oldest_date='2016-07-01 00:00:00'

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="delete-a-job-and-all-its-job-history"></a>Eliminación de un trabajo y de todo el historial de trabajos

En el ejemplo siguiente se muestra cómo eliminar un trabajo y el historial de todos los trabajos relacionados.  
Conéctese a la [*base de datos de trabajo*](job-automation-overview.md#job-database) y ejecute el siguiente comando:

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_delete_job @job_name='ResultsPoolsJob'

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="job-stored-procedures"></a>Procedimientos almacenados de trabajos

Los siguientes procedimientos almacenados se encuentran en la [base de datos de trabajos](job-automation-overview.md#job-database).

|Procedimiento almacenado  |Descripción  |
|---------|---------|
|[sp_add_job](#sp_add_job)     |     Agrega un nuevo trabajo.    |
|[sp_update_job](#sp_update_job)    |      Actualiza un trabajo existente.   |
|[sp_delete_job](#sp_delete_job)     |      Elimina un trabajo existente.   |
|[sp_add_jobstep](#sp_add_jobstep)    |    Agrega un paso a un trabajo.     |
|[sp_update_jobstep](#sp_update_jobstep)     |     Actualiza un paso de trabajo.    |
|[sp_delete_jobstep](#sp_delete_jobstep)     |     Elimina un paso de trabajo.    |
|[sp_start_job](#sp_start_job)    |  Inicia la ejecución de un trabajo.       |
|[sp_stop_job](#sp_stop_job)     |     Detiene una ejecución de trabajo.   |
|[sp_add_target_group](#sp_add_target_group)    |     Agrega un grupo de destino.    |
|[sp_delete_target_group](#sp_delete_target_group)     |    Elimina un grupo de destino.     |
|[sp_add_target_group_member](#sp_add_target_group_member)     |    Agrega una base de datos o un grupo de bases de datos a un grupo de destino.     |
|[sp_delete_target_group_member](#sp_delete_target_group_member)     |     Quita un miembro de un grupo de destino.    |
|[sp_purge_jobhistory](#sp_purge_jobhistory)    |    Quita los registros de historial de un trabajo.     |

### <a name="sp_add_job"></a><a name="sp_add_job"></a>sp_add_job

Agrega un nuevo trabajo.
  
#### <a name="syntax"></a>Sintaxis  
  
```syntaxsql
[jobs].sp_add_job [ @job_name = ] 'job_name'  
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
  [ , [ @job_id = ] job_id OUTPUT ]
```

#### <a name="arguments"></a>Argumentos  

[ **\@job_name =** ] 'job_name'  
Nombre del trabajo. El nombre debe ser único y no puede contener el carácter de porcentaje (%). job_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@description =** ] 'description'  
Descripción del trabajo. description es nvarchar(512), con un valor predeterminado de NULL. Si description se omite, se utiliza una cadena vacía.

[ **\@enabled =** ] enabled  
Indica si la programación del trabajo está habilitada. El valor de enabled es de tipo bit, con un valor predeterminado de 0 (deshabilitado). Si es 0, el trabajo no se habilita y no se ejecuta según la programación; sin embargo, se puede ejecutar manualmente. Si es 1, el trabajo se ejecuta según la programación y también se puede ejecutar manualmente.

[ **\@schedule_interval_type =** ] schedule_interval_type  
El valor indica cuándo se ejecutará el trabajo. schedule_interval_type es nvarchar(50), con un valor predeterminado de Once y puede tener uno de los siguientes valores:

- 'Once',
- 'Minutes'
- 'Hours'
- 'Days'
- 'Weeks'
- 'Months'

[ **\@schedule_interval_count =** ] schedule_interval_count  
Número de períodos de schedule_interval_type que se producirán entre las ejecuciones del trabajo. schedule_interval_count es int, con un valor predeterminado de 1. El valor debe ser mayor o igual que 1.

[ **\@schedule_start_time =** ] schedule_start_time  
Fecha en la que puede comenzar la ejecución del trabajo. schedule_start_time es DATETIME2, con un valor predeterminado de 0001-01-01 00:00:00.0000000.

[ **\@schedule_end_time =** ] schedule_end_time  
Fecha en la que puede detenerse la ejecución del trabajo. schedule_end_time es DATETIME2, con un valor predeterminado de 9999-12-31 11:59:59.0000000.

[ **\@job_id =** ] job_id OUTPUT  
Número de identificación que se ha asignado al trabajo si este se ha creado correctamente. job_id es una variable de salida de tipo uniqueidentifier.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

sp_add_job se debe ejecutar desde la base de datos del agente de trabajo especificada al crear al agente de trabajo.
Después de ejecutar sp_add_job para agregar un trabajo, sp_add_jobstep puede utilizarse para agregar pasos que realice las actividades del trabajo. El número de versión inicial del trabajo es 0, que se incrementará en 1 cuando se agregue el primer paso.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_update_job"></a><a name="sp_update_job"></a>sp_update_job

Actualiza un trabajo existente.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_update_job [ @job_name = ] 'job_name'  
  [ , [ @new_name = ] 'new_name' ]
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
Nombre del trabajo que se debe actualizar. job_name es nvarchar(128).

[ **\@new_name =** ] 'new_name'  
Nuevo nombre del trabajo. new_name es nvarchar(128).

[ **\@description =** ] 'description'  
Descripción del trabajo. description es nvarchar(512).

[ **\@enabled =** ] enabled  
Especifica si la programación del trabajo está habilitado (1) o no habilitada (0). El valor de enabled es de tipo bit.

[ **\@schedule_interval_type=** ] schedule_interval_type  
El valor indica cuándo se ejecutará el trabajo. schedule_interval_type es nvarchar(50) y puede tener uno de los siguientes valores:

- 'Once',
- 'Minutes'
- 'Hours'
- 'Days'
- 'Weeks'
- 'Months'

[ **\@schedule_interval_count=** ] schedule_interval_count  
Número de períodos de schedule_interval_type que se producirán entre las ejecuciones del trabajo. schedule_interval_count es int, con un valor predeterminado de 1. El valor debe ser mayor o igual que 1.

[ **\@schedule_start_time=** ] schedule_start_time  
Fecha en la que puede comenzar la ejecución del trabajo. schedule_start_time es DATETIME2, con un valor predeterminado de 0001-01-01 00:00:00.0000000.

[ **\@schedule_end_time=** ] schedule_end_time  
Fecha en la que puede detenerse la ejecución del trabajo. schedule_end_time es DATETIME2, con un valor predeterminado de 9999-12-31 11:59:59.0000000.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Después de ejecutar sp_add_job para agregar un trabajo, sp_add_jobstep puede utilizarse para agregar pasos que realice las actividades del trabajo. El número de versión inicial del trabajo es 0, que se incrementará en 1 cuando se agregue el primer paso.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_delete_job"></a><a name="sp_delete_job"></a>sp_delete_job

Elimina un trabajo existente.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_delete_job [ @job_name = ] 'job_name'
  [ , [ @force = ] force ]
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
Nombre del trabajo que se debe eliminar. job_name es nvarchar(128).

[ **\@force =** ] force  
Especifica la eliminación si el trabajo tiene ejecuciones en curso y la cancelación de todas las ejecuciones en curso (1) o un error si existen ejecuciones de trabajos en curso (0). El valor de force es de tipo bit.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

El historial de trabajos se elimina automáticamente cuando se elimina un trabajo.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_add_jobstep"></a><a name="sp_add_jobstep"></a>sp_add_jobstep

Agrega un paso a un trabajo.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_add_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] step_name ]
     [ , [ @command_type = ] 'command_type' ]
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]
     [ , [ @retry_attempts = ] retry_attempts ]
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]
     [ , [ @output_type = ] 'output_type' ]
     [ , [ @output_credential_name = ] 'output_credential_name' ]
     [ , [ @output_subscription_id = ] 'output_subscription_id' ]
     [ , [ @output_resource_group_name = ] 'output_resource_group_name' ]
     [ , [ @output_server_name = ] 'output_server_name' ]
     [ , [ @output_database_name = ] 'output_database_name' ]
     [ , [ @output_schema_name = ] 'output_schema_name' ]
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
El nombre del trabajo al que se agrega el paso. job_name es nvarchar(128).

[ **\@step_id =** ] step_id  
Número de identificación de secuencia del paso del trabajo. Los números de identificación de pasos se inician en 1 e incrementan sin interrupciones. Si un paso existente ya tiene este identificador, dicho paso y todos los pasos siguientes verán su identificador incrementado para que se pueda insertar el nuevo paso en la secuencia. Si no se especifica, step_id se asignará automáticamente al último paso de la secuencia. step_id es int.

[ **\@step_name =** ] step_name  
Nombre del paso. Debe especificarse, excepto para el primer paso de un trabajo que (por comodidad) tiene un nombre predeterminado de 'JobStep'. step_name es nvarchar(128).

[ **\@command_type =** ] 'command_type'  
Tipo de comando que ejecuta jobstep. command_type es nvarchar(50), con un valor predeterminado de TSql, lo que significa que el valor del parámetro @command_type es un script T-SQL.

Si se especifica, el valor debe ser TSql.

[ **\@command_source =** ] 'command_source'  
Tipo de ubicación donde se almacena el comando. command_source is nvarchar(50), con un valor predeterminado de Inline, lo que significa que el valor del parámetro @command_source es el texto literal del comando.

Si se especifica, el valor debe ser Inline.

[ **\@command =** ] 'command'  
El comando debe ser un script T-SQL válido, que se ejecutará a continuación en este paso de trabajo. command es nvarchar(max), con un valor predeterminado de NULL.

[ **\@credential_name =** ] 'credential_name'  
Nombre de la credencial de ámbito de base de datos almacenada en esta base de datos de control de trabajos, que se usa para conectarse a cada una de las bases de datos de destino en del grupo de destino cuando se ejecuta este paso. credential_name es nvarchar(128).

[ **\@target_group_name =** ] 'target-group_name'  
Nombre del grupo de destino que contiene las bases de datos de destino donde se ejecutará el paso de trabajo. target_group_name is nvarchar(128).

[ **\@initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
Retraso que se producirá antes del primer reintento,si el paso de trabajo no se ejecuta correctamente en el intento de ejecución inicial. initial_retry_interval_seconds es int, con un valor predeterminado de 1.

[ **\@maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
Retraso máximo entre los reintentos. Si el retraso entre los reintentos creciera por encima de este valor, se limitaría a este valor en su lugar. maximum_retry_interval_seconds es int, con un valor predeterminado de 120.

[ **\@retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
Multiplicador que se aplicará al retraso de reintentos si varios intentos de ejecución del paso de trabajo generan un error. Por ejemplo, si el primer reintento tuvo un retraso de 5 segundos y el multiplicador de retroceso es 2.0, el segundo reintento tendrá un retraso de 10 segundos y el tercero, de 20 segundos. retry_interval_backoff_multiplier es real, con un valor predeterminado de 2.0.

[ **\@retry_attempts =** ] retry_attempts  
Número de veces que reintentará la ejecución si se produce un error en el intento inicial. Por ejemplo, si el valor de retry_attempts es 10, habrá 1 intento inicial y 10 reintentos, que suman un total de 11 intentos. Si se produce un error en el reintento final, la ejecución del trabajo terminará con el valor Failed para el ciclo de vida. retry_attempts es int, con un valor predeterminado de 10.

[ **\@step_timeout_seconds =** ] step_timeout_seconds  
Cantidad máxima de tiempo permitida para la ejecución del paso. Si se supera este tiempo, la ejecución del trabajo terminará con el valor TimedOut para el ciclo de vida. step_timeout_seconds es int, con un valor predeterminado de 43 200 segundos (12 horas).

[ **\@output_type =** ] 'output_type'  
Si no es null, tipo de destino en el que se escribirá el primer conjunto de resultados del comando. output_type es nvarchar(50), con un valor predeterminado de NULL.

Si se especifica, el valor debe ser SqlDatabase.

[ **\@output_credential_name =** ] 'output_credential_name'  
Si no es null, nombre de la credencial de ámbito de base de datos que se usa para conectarse a la base de datos de destino de salida. Debe especificarse si output_type es igual a SqlDatabase. output_credential_name es nvarchar(128), con un valor predeterminado de NULL.

[ **\@output_subscription_id =** ] 'output_subscription_id'  
La descripción es necesaria.

[ **\@output_resource_group_name =** ] 'output_resource_group_name'  
La descripción es necesaria.

[ **\@output_server_name =** ] 'output_server_name'  
Si no es null, nombre DNS completo del servidor que contiene la base de datos de destino de salida. Debe especificarse si output_type es igual a SqlDatabase. output_database_name es nvarchar(256), con un valor predeterminado de NULL.

[ **\@output_database_name =** ] 'output_database_name'  
Si no es null, nombre de la base de datos que contiene la tabla de destino de salida. Debe especificarse si output_type es igual a SqlDatabase. output_database_name es nvarchar(128), con un valor predeterminado de NULL.

[ **\@output_schema_name =** ] 'output_schema_name'  
Si no es null, nombre del esquema SQL que contiene la tabla de destino de salida. Si output_type es igual a SqlDatabase, el valor predeterminado es dbo. output_schema_name es nvarchar(128).

[ **\@output_table_name =** ] 'output_table_name'  
Si no es null, nombre de la tabla en la que se escribirá el primer conjunto de resultados del comando. Si la tabla no existe todavía, se creará de acuerdo con el esquema del conjunto de resultados devuelto. Debe especificarse si output_type es igual a SqlDatabase. output_table_name es nvarchar(128), con un valor predeterminado de NULL.

[ **\@job_version =** ] job_version OUTPUT  
Parámetro de salida al que se asignará el número de versión de trabajo nuevo. job_version es int.

[ **\@max_parallelism =** ] max_parallelism OUTPUT  
Nivel máximo de paralelismo por grupo elástico. Si está establecido, el paso de trabajo se restringirá para que solo se ejecute en un máximo de las bases de datos por grupo elástico. Esto se aplica a cada grupo elástico que se incluya directamente en el grupo de destino o se encuentre dentro de un servidor incluido en el grupo de destino. max_parallelism es int.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Si sp_add_jobstep se ejecuta correctamente, se incrementa el número de versión actual del trabajo. La próxima vez que se ejecute el trabajo, se usará la nueva versión. Si el trabajo se está ejecutando actualmente, la ejecución en curso no contendrá el nuevo paso.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:  

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_update_jobstep"></a><a name="sp_update_jobstep"></a>sp_update_jobstep

Actualiza un paso de trabajo.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_update_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]
     [ , [ @new_id = ] new_id ]
     [ , [ @new_name = ] 'new_name' ]
     [ , [ @command_type = ] 'command_type' ]
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]
     [ , [ @retry_attempts = ] retry_attempts ]
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]
     [ , [ @output_type = ] 'output_type' ]
     [ , [ @output_credential_name = ] 'output_credential_name' ]
     [ , [ @output_server_name = ] 'output_server_name' ]
     [ , [ @output_database_name = ] 'output_database_name' ]
     [ , [ @output_schema_name = ] 'output_schema_name' ]
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
El nombre del trabajo al que pertenece el paso. job_name es nvarchar(128).

[ **\@step_id =** ] step_id  
Número de identificación del paso de trabajo que va a modificarse. Debe especificarse step_id o step_name. step_id es int.

[ **\@step_name =** ] 'step_name'  
Nombre del paso que se va a modificar. Debe especificarse step_id o step_name. step_name es nvarchar(128).

[ **\@new_id =** ] new_id  
Nuevo número de identificación de secuencia del paso de trabajo. Los números de identificación de pasos se inician en 1 e incrementan sin interrupciones. Si un paso se reordena, los otros pasos se renumerarán automáticamente.

[ **\@new_name =** ] 'new_name'  
Nuevo nombre del paso. new_name es nvarchar(128).

[ **\@command_type =** ] 'command_type'  
Tipo de comando que ejecuta jobstep. command_type es nvarchar(50), con un valor predeterminado de TSql, lo que significa que el valor del parámetro @command_type es un script T-SQL.

Si se especifica, el valor debe ser TSql.

[ **\@command_source =** ] 'command_source'  
Tipo de ubicación donde se almacena el comando. command_source is nvarchar(50), con un valor predeterminado de Inline, lo que significa que el valor del parámetro @command_source es el texto literal del comando.

Si se especifica, el valor debe ser Inline.

[ **\@command =** ] 'command'  
Los valores de command deben ser un script T-SQL válido, que se ejecutará en este paso de trabajo. command es nvarchar(max), con un valor predeterminado de NULL.

[ **\@credential_name =** ] 'credential_name'  
Nombre de la credencial de ámbito de base de datos almacenada en esta base de datos de control de trabajos, que se usa para conectarse a cada una de las bases de datos de destino en del grupo de destino cuando se ejecuta este paso. credential_name es nvarchar(128).

[ **\@target_group_name =** ] 'target-group_name'  
Nombre del grupo de destino que contiene las bases de datos de destino donde se ejecutará el paso de trabajo. target_group_name is nvarchar(128).

[ **\@initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
Retraso que se producirá antes del primer reintento,si el paso de trabajo no se ejecuta correctamente en el intento de ejecución inicial. initial_retry_interval_seconds es int, con un valor predeterminado de 1.

[ **\@maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
Retraso máximo entre los reintentos. Si el retraso entre los reintentos creciera por encima de este valor, se limitaría a este valor en su lugar. maximum_retry_interval_seconds es int, con un valor predeterminado de 120.

[ **\@retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
Multiplicador que se aplicará al retraso de reintentos si varios intentos de ejecución del paso de trabajo generan un error. Por ejemplo, si el primer reintento tuvo un retraso de 5 segundos y el multiplicador de retroceso es 2.0, el segundo reintento tendrá un retraso de 10 segundos y el tercero, de 20 segundos. retry_interval_backoff_multiplier es real, con un valor predeterminado de 2.0.

[ **\@retry_attempts =** ] retry_attempts  
Número de veces que reintentará la ejecución si se produce un error en el intento inicial. Por ejemplo, si el valor de retry_attempts es 10, habrá 1 intento inicial y 10 reintentos, que suman un total de 11 intentos. Si se produce un error en el reintento final, la ejecución del trabajo terminará con el valor Failed para el ciclo de vida. retry_attempts es int, con un valor predeterminado de 10.

[ **\@step_timeout_seconds =** ] step_timeout_seconds  
Cantidad máxima de tiempo permitida para la ejecución del paso. Si se supera este tiempo, la ejecución del trabajo terminará con el valor TimedOut para el ciclo de vida. step_timeout_seconds es int, con un valor predeterminado de 43 200 segundos (12 horas).

[ **\@output_type =** ] 'output_type'  
Si no es null, tipo de destino en el que se escribirá el primer conjunto de resultados del comando. Para restablecer el valor de output_type a NULL, establece el valor de este parámetro en '' (cadena vacía). output_type es nvarchar(50), con un valor predeterminado de NULL.

Si se especifica, el valor debe ser SqlDatabase.

[ **\@output_credential_name =** ] 'output_credential_name'  
Si no es null, nombre de la credencial de ámbito de base de datos que se usa para conectarse a la base de datos de destino de salida. Debe especificarse si output_type es igual a SqlDatabase. Para restablecer el valor de output_credential_name a NULL, establece el valor de este parámetro en '' (cadena vacía). output_credential_name es nvarchar(128), con un valor predeterminado de NULL.

[ **\@output_server_name =** ] 'output_server_name'  
Si no es null, nombre DNS completo del servidor que contiene la base de datos de destino de salida. Debe especificarse si output_type es igual a SqlDatabase. Para restablecer el valor de output_server_name a NULL, establece el valor de este parámetro en '' (cadena vacía). output_database_name es nvarchar(256), con un valor predeterminado de NULL.

[ **\@output_database_name =** ] 'output_database_name'  
Si no es null, nombre de la base de datos que contiene la tabla de destino de salida. Debe especificarse si output_type es igual a SqlDatabase. Para restablecer el valor de output_database_name a NULL, establece el valor de este parámetro en '' (cadena vacía). output_database_name es nvarchar(128), con un valor predeterminado de NULL.

[ **\@output_schema_name =** ] 'output_schema_name'  
Si no es null, nombre del esquema SQL que contiene la tabla de destino de salida. Si output_type es igual a SqlDatabase, el valor predeterminado es dbo. Para restablecer el valor de output_schema_name a NULL, establece el valor de este parámetro en '' (cadena vacía). output_schema_name es nvarchar(128).

[ **\@output_table_name =** ] 'output_table_name'  
Si no es null, nombre de la tabla en la que se escribirá el primer conjunto de resultados del comando. Si la tabla no existe todavía, se creará de acuerdo con el esquema del conjunto de resultados devuelto. Debe especificarse si output_type es igual a SqlDatabase. Para restablecer el valor de output_server_name a NULL, establece el valor de este parámetro en '' (cadena vacía). output_table_name es nvarchar(128), con un valor predeterminado de NULL.

[ **\@job_version =** ] job_version OUTPUT  
Parámetro de salida al que se asignará el número de versión de trabajo nuevo. job_version es int.

[ **\@max_parallelism =** ] max_parallelism OUTPUT  
Nivel máximo de paralelismo por grupo elástico. Si está establecido, el paso de trabajo se restringirá para que solo se ejecute en un máximo de las bases de datos por grupo elástico. Esto se aplica a cada grupo elástico que se incluya directamente en el grupo de destino o se encuentre dentro de un servidor incluido en el grupo de destino. Para restablecer el valor de max_parallelism a null, establece el valor de este parámetro en -1. max_parallelism es int.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Las ejecuciones en curso del trabajo no se verán afectadas. Si sp_update_jobstep se ejecuta correctamente, se incrementa el número de versión del trabajo. La próxima vez que se ejecute el trabajo, se usará la nueva versión.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_delete_jobstep"></a><a name="sp_delete_jobstep"></a>sp_delete_jobstep

Quita un paso de un trabajo.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_delete_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
Nombre del trabajo del que se quitará el paso. job_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@step_id =** ] step_id  
Número de identificación del paso de trabajo que se va a eliminar. Debe especificarse step_id o step_name. step_id es int.

[ **\@step_name =** ] 'step_name'  
Nombre del paso que se va a eliminar. Debe especificarse step_id o step_name. step_name es nvarchar(128).

[ **\@job_version =** ] job_version OUTPUT  
Parámetro de salida al que se asignará el número de versión de trabajo nuevo. job_version es int.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Las ejecuciones en curso del trabajo no se verán afectadas. Si sp_update_jobstep se ejecuta correctamente, se incrementa el número de versión del trabajo. La próxima vez que se ejecute el trabajo, se usará la nueva versión.

Los demás pasos del trabajo se renumerarán automáticamente para ocupar el espacio que dejó el paso de trabajo eliminado.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_start_job"></a><a name="sp_start_job"></a>sp_start_job

Inicia la ejecución de un trabajo.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_start_job [ @job_name = ] 'job_name'
     [ , [ @job_execution_id = ] job_execution_id OUTPUT ]
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
Nombre del trabajo del que se quitará el paso. job_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@job_execution_id =** ] job_execution_id OUTPUT  
Parámetro de salida al que se asignará el identificador de la ejecución del trabajo. job_version es uniqueidentifier.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Ninguno.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_stop_job"></a><a name="sp_stop_job"></a>sp_stop_job

Detiene una ejecución de trabajo.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_stop_job [ @job_execution_id = ] ' job_execution_id '
```

#### <a name="arguments"></a>Argumentos

[ **\@job_execution_id =** ] job_execution_id  
Número de identificación de la ejecución del trabajo que se va a detener. job_execution_id es uniqueidentifier, con un valor predeterminado de NULL.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Ninguno.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_add_target_group"></a><a name="sp_add_target_group"></a>sp_add_target_group

Agrega un grupo de destino.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_add_target_group [ @target_group_name = ] 'target_group_name'
     [ , [ @target_group_id = ] target_group_id OUTPUT ]
```

#### <a name="arguments"></a>Argumentos

[ **\@target_group_name =** ] 'target_group_name'  
Nombre del grupo de destino que se va a crear. target_group_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@target_group_id =** ] target_group_id OUTPUT Número de identificación del grupo de destino asignado al trabajo si se creó correctamente. target_group_id es una variable de salida de tipo uniqueidentifier, con un valor predeterminado de NULL.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Los grupos de destino proporcionan una manera sencilla de dirigir un trabajo a una colección de bases de datos.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_delete_target_group"></a><a name="sp_delete_target_group"></a>sp_delete_target_group

Elimina un grupo de destino.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_delete_target_group [ @target_group_name = ] 'target_group_name'
```

#### <a name="arguments"></a>Argumentos

[ **\@target_group_name =** ] 'target_group_name'  
Nombre del grupo de destino que se va a eliminar. target_group_name es nvarchar(128), sin ningún valor predeterminado.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Ninguno.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

### <a name="sp_add_target_group_member"></a><a name="sp_add_target_group_member"></a>sp_add_target_group_member

Agrega una base de datos o un grupo de bases de datos a un grupo de destino.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_add_target_group_member [ @target_group_name = ] 'target_group_name'
         [ @membership_type = ] 'membership_type' ]
        [ , [ @target_type = ] 'target_type' ]
        [ , [ @refresh_credential_name = ] 'refresh_credential_name' ]
        [ , [ @server_name = ] 'server_name' ]
        [ , [ @database_name = ] 'database_name' ]
        [ , [ @elastic_pool_name = ] 'elastic_pool_name' ]
        [ , [ @shard_map_name = ] 'shard_map_name' ]
        [ , [ @target_id = ] 'target_id' OUTPUT ]
```

#### <a name="arguments"></a>Argumentos

[ **\@target_group_name =** ] 'target_group_name'  
Nombre del grupo de destino al que se agregará el miembro. target_group_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@membership_type =** ] 'membership_type'  
Especifica si se incluirá o excluirá el miembro del grupo de destino. target_group_name es nvarchar(128), con un valor predeterminado de ‘Include’. Los valores válidos de target_group_name son ‘Include’ o ‘Exclude’.

[ **\@target_type =** ] 'target_type'  
Tipo de base de datos de destino o colección de bases de datos que incluye todas las bases de datos de un servidor, todas las bases de datos de un grupo elástico, todas las bases de datos de un mapa de particiones o una base de datos individual. target_type es nvarchar(128), sin ningún valor predeterminado. Los valores válidos para target_type son ‘SqlServer’, ‘SqlElasticPool’, ‘SqlDatabase’ o ‘SqlShardMap’.

[ **\@refresh_credential_name =** ] 'refresh_credential_name'  
El nombre del servidor. refresh_credential_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@server_name =** ] 'server_name'  
Nombre del servidor que se debe agregar al grupo de destino especificado. server_name debe especificarse cuando target_type es ‘SqlServer’. server_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@database_name =** ] 'database_name'  
Nombre de la base de datos que debe agregarse al grupo de destino especificado. database_name debe especificarse cuando target_type es ‘SqlDatabase’. database_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@elastic_pool_name =** ] 'elastic_pool_name'  
Nombre del grupo elástico que se debe agregar al grupo de destino especificado. elastic_pool_name debe especificarse cuando target_type es ‘SqlElasticPool’. elastic_pool_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@shard_map_name =** ] 'shard_map_name'  
Nombre del grupo de mapas de particiones que se debe agregar al grupo de destino especificado. elastic_pool_name debe especificarse cuando target_type es ‘SqlSqlShardMap’. shart_map_name es nvarchar(128), sin ningún valor predeterminado.

[ **\@target_id =** ] target_group_id OUTPUT  
Número de identificación de destino asignado al miembro del grupo de destino si se agregó al grupo de destino. target_id es una variable de salida de tipo uniqueidentifier, con un valor predeterminado de NULL.
Valores de código de retorno 0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Un trabajo se ejecuta en todas las bases de datos únicas de un servidor o grupo elástico en tiempo de ejecución, cuando un servidor o un grupo elástico está incluido en el grupo de destino.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

#### <a name="examples"></a>Ejemplos

En el ejemplo siguiente se agregan todas las bases de datos de los servidores de Londres y Nueva York al grupo Servers Maintaining Customer Information. Debe conectarse a la base de datos de trabajos que especificó al crear al agente de trabajo, que en este caso es ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ;
GO

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group @target_group_name =  N'Servers Maintaining Customer Information'
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'London.database.windows.net' ;
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'NewYork.database.windows.net' ;
GO

--View the recently added members to the target group
SELECT * FROM [jobs].target_group_members WHERE target_group_name= N'Servers Maintaining Customer Information';
GO
```

### <a name="sp_delete_target_group_member"></a><a name="sp_delete_target_group_member"></a>sp_delete_target_group_member

Quita un miembro de un grupo de destino.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_delete_target_group_member [ @target_group_name = ] 'target_group_name'
   [ , [ @target_id = ] 'target_id']
```

#### <a name="arguments"></a>Argumentos

[ @target_group_name = ] 'target_group_name'  
Nombre del grupo de destino del que se va a quitar el miembro del grupo de destino. target_group_name es nvarchar(128), sin ningún valor predeterminado.

[ @target_id = ] target_id  
 Número de identificación de destino asignado al miembro del grupo de destino que se va a quitar. target_id es un identificador único, con el valor predeterminado NULL.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Los grupos de destino proporcionan una manera sencilla de dirigir un trabajo a una colección de bases de datos.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

#### <a name="examples"></a>Ejemplos

En el ejemplo siguiente se quita el servidor de Londres del grupo Servers Maintaining Customer Information. Debe conectarse a la base de datos de trabajos que especificó al crear al agente de trabajo, que en este caso es ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ;
GO

-- Retrieve the target_id for a target_group_members
declare @tid uniqueidentifier
SELECT @tid = target_id FROM [jobs].target_group_members WHERE target_group_name = 'Servers Maintaining Customer Information' and server_name = 'London.database.windows.net'

-- Remove a target group member of type server
EXEC jobs.sp_delete_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_id = @tid
GO
```

### <a name="sp_purge_jobhistory"></a><a name="sp_purge_jobhistory"></a>sp_purge_jobhistory

Quita los registros de historial de un trabajo.

#### <a name="syntax"></a>Sintaxis

```syntaxsql
[jobs].sp_purge_jobhistory [ @job_name = ] 'job_name'
      [ , [ @job_id = ] job_id ]
      [ , [ @oldest_date = ] oldest_date []
```

#### <a name="arguments"></a>Argumentos

[ **\@job_name =** ] 'job_name'  
Nombre del trabajo del que se van a eliminar registros de historial. job_name es nvarchar(128), con un valor predeterminado de NULL. Se debe especificar job_id o job_name, pero no pueden especificarse ambos.

[ **\@job_id =** ] job_id  
 Número de identificación del trabajo para los registros que se van a eliminar. job_id es uniqueidentifier, con un valor predeterminado de NULL. Se debe especificar job_id o job_name, pero no pueden especificarse ambos.

[ **\@oldest_date =** ] oldest_date  
 Registro más antiguo que se mantendrá en el historial. oldest_date es DATETIME2, con un valor predeterminado de NULL. Si se especifica oldest_date, sp_purge_jobhistory solo quita los registros más antiguos que el valor especificado.

#### <a name="return-code-values"></a>Valores de código de retorno

0 (correcto) o 1 (error)

#### <a name="remarks"></a>Observaciones

Los grupos de destino proporcionan una manera sencilla de dirigir un trabajo a una colección de bases de datos.

#### <a name="permissions"></a>Permisos

De forma predeterminada, los miembros del rol de servidor fijo sysadmin pueden ejecutar este procedimiento almacenado. La restricción de un usuario para que solo pueda supervisar trabajos le permite autorizar al usuario para que forme parte del rol de base de datos siguiente en la base de datos del agente de trabajo especificado al crear el agente del trabajo:

- jobs_reader

Para obtener más información acerca de los permisos de estos roles, consulte la sección Permisos de este documento. Solo los miembros de sysadmin pueden usar este procedimiento almacenado para editar los atributos de los trabajos que pertenecen a otros usuarios.

#### <a name="examples"></a>Ejemplos

En el ejemplo siguiente se agregan todas las bases de datos de los servidores de Londres y Nueva York al grupo Servers Maintaining Customer Information. Debe conectarse a la base de datos de trabajos que especificó al crear al agente de trabajo, que en este caso es ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent

EXEC sp_delete_target_group_member
    @target_group_name = N'Servers Maintaining Customer Information',  
    @server_name = N'London.database.windows.net';  
GO
```

## <a name="job-views"></a>Vistas de trabajos

Las siguientes vistas están disponibles en la [base de datos de trabajos](job-automation-overview.md#job-database).

|Ver  |Descripción  |
|---------|---------|
|[job_executions](#job_executions-view)     |  Muestra el historial de ejecuciones de trabajos.      |
|[jobs](#jobs-view)     |   Muestra todos los trabajos.      |
|[job_versions](#job_versions-view)     |   Muestra todas las versiones del trabajo.      |
|[jobsteps](#jobsteps-view)     |     Muestra todos los pasos de la versión actual de cada trabajo.    |
|[jobstep_versions](#jobstep_versions-view)     |     Muestra todos los pasos de todas las versiones de cada trabajo.    |
|[target_groups](#target_groups-view)     |      Muestra todos los grupos de destino.   |
|[target_group_members](#target_group_members-view)     |   Muestra todos los miembros de todos los grupos de destino.      |

### <a name="job_executions-view"></a><a name="job_executions-view"></a>vista de job_executions

[jobs].[job_executions]

Muestra el historial de ejecuciones de trabajos.

|Nombre de la columna | Tipo de datos | Descripción |
|---------|---------|---------|
|**job_execution_id** | UNIQUEIDENTIFIER | Identificador único de una instancia de ejecución de un trabajo.
|**job_name** | nvarchar(128) | Nombre del trabajo.
|**job_id** | UNIQUEIDENTIFIER | Id. único del trabajo.
|**job_version** | int | Versión del trabajo (se actualiza automáticamente cada vez que el trabajo se modifica).
|**step_id** |int | Identificador único del paso (en este trabajo). NULL indica que se trata de la ejecución del trabajo principal.
|**is_active** | bit | Indica si la información está activa o inactiva. 1 indica trabajos activos y 0, trabajos inactivos.
|**lifecycle** | nvarchar(50) | Valor que indica el estado del trabajo: "Created", "In Progress", "Failed", "Succeeded", "Skipped", "SucceededWithSkipped".|
|**create_time**| datetime2(7) | Fecha y hora en que se creó el trabajo.
|**start_time** | datetime2(7) | Fecha y hora en que el trabajo inició la ejecución. NULL si todavía no se ejecutó el trabajo.
|**end_time** | datetime2(7) | Fecha y hora en que el trabajo finalizó la ejecución. NULL si todavía no se ejecutó el trabajo o no se completó la ejecución.
|**current_attempts** | int | Número de veces que se reintentó el paso. El trabajo principal será 0 y las ejecuciones del trabajo secundario, 1 o un valor mayor según la directiva de ejecución.
|**current_attempt_start_time** | datetime2(7) | Fecha y hora en que el trabajo inició la ejecución. NULL indica que se trata de la ejecución del trabajo principal.
|**last_message** | nvarchar(max) | Mensaje del historial de trabajos o pasos.
|**target_type** | nvarchar(128) | Tipo de base de datos de destino o colección de bases de datos, incluidas todas las bases de datos de un servidor, todas las bases de datos de un grupo elástico o una base de datos. Los valores válidos para target_type son ‘SqlServer’, ‘SqlElasticPool’, ‘SqlDatabase’ o ‘SqlDatabase’. NULL indica que se trata de la ejecución del trabajo principal.
|**target_id** | UNIQUEIDENTIFIER | Identificador único del miembro del grupo de destino.  NULL indica que se trata de la ejecución del trabajo principal.
|**target_group_name** | nvarchar(128) | Nombre del grupo de destino. NULL indica que se trata de la ejecución del trabajo principal.
|**target_server_name** | nvarchar(256)  | Nombre del servidor incluido en el grupo de destino. Solo se especifica si target_type es ‘SqlServer’. NULL indica que se trata de la ejecución del trabajo principal.
|**target_database_name** | nvarchar(128) | Nombre de la base de datos incluida en el grupo de destino. Solo se especifica si target_type es ‘SqlDatabase’. NULL indica que se trata de la ejecución del trabajo principal.

### <a name="jobs-view"></a>vista jobs

[jobs].[jobs]

Muestra todos los trabajos.

|Nombre de la columna | Tipo de datos |Descripción|
|------|------|-------|
|**job_name** | nvarchar(128) | Nombre del trabajo.|
|**job_id**| UNIQUEIDENTIFIER |Id. único del trabajo.|
|**job_version** |int |Versión del trabajo (se actualiza automáticamente cada vez que el trabajo se modifica).|
|**description** |nvarchar(512)| Descripción del trabajo. Bit habilitado: Indica si el trabajo está habilitado o deshabilitado. 1 indica trabajos habilitados y 0, trabajos deshabilitados.|
|**schedule_interval_type**|nvarchar(50) |Valor que indica cuándo se ejecutará el trabajo: 'Once', 'Minutes', 'Hours', 'Days', 'Weeks' o 'Months'
|**schedule_interval_count**|int|Número de períodos schedule_interval_type que se producirán entre las ejecuciones del trabajo.|
|**schedule_start_time**|datetime2(7)|Fecha y hora en que el trabajo inició la ejecución por última vez.|
|**schedule_end_time**|datetime2(7)|Fecha y hora en que el trabajo completó la ejecución por última vez.|

### <a name="job_versions-view"></a><a name="job_versions-view"></a>vista job_versions

[jobs].[job_versions]

Muestra todas las versiones del trabajo.

|Nombre de la columna|Tipo de datos|Descripción|
|------|------|-------|
|**job_name**|nvarchar(128)|Nombre del trabajo.|
|**job_id**|UNIQUEIDENTIFIER|Id. único del trabajo.|
|**job_version**|int|Versión del trabajo (se actualiza automáticamente cada vez que el trabajo se modifica).|

### <a name="jobsteps-view"></a>vista jobsteps

[jobs].[jobsteps]

Muestra todos los pasos de la versión actual de cada trabajo.

|Nombre de la columna|Tipo de datos|Descripción|
|------|------|-------|
|**job_name**|nvarchar(128)|Nombre del trabajo.|
|**job_id**|UNIQUEIDENTIFIER|Id. único del trabajo.|
|**job_version**|int|Versión del trabajo (se actualiza automáticamente cada vez que el trabajo se modifica).|
|**step_id**|int|Identificador único del paso (en este trabajo).|
|**step_name**|nvarchar(128)|Nombre único (de este trabajo) para el paso.|
|**command_type**|nvarchar(50)|Tipo de comando que se ejecutará en el paso de trabajo. Para v1, el valor debe ser igual a ‘TSql’, que es el valor predeterminado.|
|**command_source**|nvarchar(50)|Ubicación del comando. Para v1, ‘Inline’ es el valor predeterminado y el único aceptado.|
|**command**|nvarchar(max)|Comandos que ejecutarán los trabajos elásticos por medio de command_type.|
|**credential_name**|nvarchar(128)|Nombre de la credencial de ámbito de base de datos usada para la ejecución del trabajo.|
|**target_group_name**|nvarchar(128)|Nombre del grupo de destino.|
|**target_group_id**|UNIQUEIDENTIFIER|Identificador único del grupo de destino.|
|**initial_retry_interval_seconds**|int|Retraso antes del primer reintento. El valor predeterminado es 1.|
|**maximum_retry_interval_seconds**|int|Retraso máximo entre los reintentos. Si el retraso entre los reintentos creciera por encima de este valor, se limitaría a este valor en su lugar. El valor predeterminado es 120.|
|**retry_interval_backoff_multiplier**|real|Multiplicador que se aplicará al retraso de reintentos si varios intentos de ejecución del paso de trabajo generan un error. El valor predeterminado es 2.0.|
|**retry_attempts**|int|Número de reintentos en caso de que el paso dé error. El valor predeterminado de 10, que indica que no se realiza ningún reintento.|
|**step_timeout_seconds**|int|Tiempo en minutos entre reintentos. El valor predeterminado es 0, que indica un intervalo de 0 minutos.|
|**output_type**|nvarchar(11)|Ubicación del comando. En la versión preliminar actual, ‘Inline’ es el valor predeterminado y el único aceptado.|
|**output_credential_name**|nvarchar(128)|Nombre de las credenciales que se usarán para conectarse al servidor de destino para almacenar el conjunto de resultados.|
|**output_subscription_id**|UNIQUEIDENTIFIER|Identificador único de la suscripción del servidor o la base de destino de destino para el conjunto de resultados de la ejecución de la consulta.|
|**output_resource_group_name**|nvarchar(128)|Nombre del grupo de recursos donde reside el servidor de destino.|
|**output_server_name**|nvarchar(256)|Nombre del servidor de destino para el conjunto de resultados.|
|**output_database_name**|nvarchar(128)|Nombre de la base de datos de destino para el conjunto de resultados.|
|**output_schema_name**|nvarchar(max)|Nombre del esquema de destino. Si no se especifica, se establece de forma predeterminada en dbo.|
|**output_table_name**|nvarchar(max)|Nombre de la tabla donde se almacenará el conjunto de resultados de los resultados de la consulta. La tabla se creará automáticamente en función del esquema del conjunto de resultados si aún no existe. El esquema debe coincidir con el esquema del conjunto de resultados.|
|**max_parallelism**|int|Número máximo de bases de datos por grupo elástico en el que se ejecutará el paso de trabajo a la vez. El valor predeterminado es NULL, lo que significa que no hay ningún límite. |

### <a name="jobstep_versions-view"></a><a name="jobstep_versions-view"></a>vista jobstep_versions

[jobs].[jobstep_versions]

Muestra todos los pasos de todas las versiones de cada trabajo. El esquema es idéntico al de [jobsteps](#jobsteps-view).

### <a name="target_groups-view"></a><a name="target_groups-view"></a>vista target_groups

[jobs].[target_groups]

Enumera todos los grupos de destino.

|Nombre de la columna|Tipo de datos|Descripción|
|-----|-----|-----|
|**target_group_name**|nvarchar(128)|Nombre del grupo de destino, una colección de bases de datos.
|**target_group_id**|UNIQUEIDENTIFIER|Identificador único del grupo de destino.

### <a name="target_group_members-view"></a><a name="target_group_members-view"></a>Vista target_group_members

[jobs].[target_group_members]

Muestra todos los miembros de todos los grupos de destino.

|Nombre de la columna|Tipo de datos|Descripción|
|-----|-----|-----|
|**target_group_name**|nvarchar(128|Nombre del grupo de destino, una colección de bases de datos. |
|**target_group_id**|UNIQUEIDENTIFIER|Identificador único del grupo de destino.|
|**membership_type**|int|Especifica si se incluirá o excluirá el miembro del grupo de destino en el grupo de destino. Los valores válidos de target_group_name son ‘Include’ o ‘Exclude’.|
|**target_type**|nvarchar(128)|Tipo de base de datos de destino o colección de bases de datos, incluidas todas las bases de datos de un servidor, todas las bases de datos de un grupo elástico o una base de datos. Los valores válidos para target_type son ‘SqlServer’, ‘SqlElasticPool’, ‘SqlDatabase’ o ‘SqlShardMap’.|
|**target_id**|UNIQUEIDENTIFIER|Identificador único del miembro del grupo de destino.|
|**refresh_credential_name**|nvarchar(128)|Nombre de la credencial de ámbito de base de datos usada para conectarse al miembro del grupo de destino.|
|**subscription_id**|UNIQUEIDENTIFIER|Identificador único de la suscripción.|
|**resource_group_name**|nvarchar(128)|Nombre del grupo de recursos en el que reside el miembro del grupo de destino.|
|**server_name**|nvarchar(128)|Nombre del servidor incluido en el grupo de destino. Solo se especifica si target_type es ‘SqlServer’. |
|**database_name**|nvarchar(128)|Nombre de la base de datos incluida en el grupo de destino. Solo se especifica si target_type es ‘SqlDatabase’.|
|**elastic_pool_name**|nvarchar(128)|Nombre del grupo elástico incluido en el grupo de destino. Solo se especifica si target_type es ‘SqlDatabase’.|
|**shard_map_name**|nvarchar(128)|Nombre de los mapas de particiones incluidos en el grupo de destino. Solo se especifica si target_type es ‘SqlShardMap’.|

## <a name="resources"></a>Recursos

- ![Icono de vínculo de tema](https://docs.microsoft.com/sql/database-engine/configure-windows/media/topic-link.gif "Icono de vínculo de tema") [Convenciones de sintaxis de Transact-SQL](https://docs.microsoft.com/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql)  

## <a name="next-steps"></a>Pasos siguientes

- [Creación y administración de trabajos elásticos mediante PowerShell](elastic-jobs-powershell-create.md)
- [Autorización y permisos](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server)
