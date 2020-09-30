---
title: Creación de un trabajo de streaming de T-SQL en Azure SQL Edge
description: Obtenga información sobre la creación de trabajos de Stream Analytics en Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: f0fcdf7aab5f43a0412cd28a1c15188b19770dc6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888093"
---
# <a name="create-a-data-streaming-job-in-azure-sql-edge"></a>Creación de un trabajo de streaming de datos en Azure SQL Edge 

En este artículo se explica cómo crear un trabajo de streaming de T-SQL en Azure SQL Edge. Cree los objetos de entrada y salida de transmisión externa y, a continuación, defina la consulta de trabajo de streaming como parte de la creación del trabajo de streaming.

## <a name="configure-the-external-stream-input-and-output-objects"></a>Configuración de los objetos de entrada y salida de transmisión externa

El streaming de T-SQL utiliza la funcionalidad de origen de datos externo de SQL Server para definir los orígenes de datos asociados a las entradas y salidas de transmisión externa del trabajo de streaming. Utilice los siguientes comandos de T-SQL para crear un objeto de entrada o salida de transmisión externa:

- [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-external-file-format-transact-sql)

- [CREATE EXTERNAL DATA SOURCE (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-external-data-source-transact-sql)

- [CREATE EXTERNAL STREAM (Transact-SQL)](#example-create-an-external-stream-object-to-azure-sql-database)

Además, si utiliza Azure SQL Edge, SQL Server o Azure SQL Database como transmisión de salida, necesita [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-database-scoped-credential-transact-sql). Este comando de T-SQL define las credenciales para acceder a la base de datos.

### <a name="supported-input-and-output-stream-data-sources"></a>Orígenes de datos de transmisión de entrada y salida admitidos

Azure SQL Edge actualmente solo admite los siguientes orígenes de datos como entradas y salidas de transmisión.

| Tipo de origen de datos | Entrada | Output | Descripción |
|------------------|-------|--------|------------------|
| Centro de Azure IoT Edge | Y | Y | Origen de datos para leer y escribir datos de streaming en un centro de Azure IoT Edge. Para más información, consulte [Centro de IoT Edge](https://docs.microsoft.com/azure/iot-edge/iot-edge-runtime#iot-edge-hub).|
| SQL Database | N | Y | Conexión de origen de datos para escribir datos de streaming en SQL Database. La base de datos puede ser una base de datos local de Azure SQL Edge, o una base de datos remota de SQL Server o Azure SQL Database.|
| Kafka | Y | N | Origen de datos para leer datos de streaming de un tema de Kafka. Este adaptador solo está disponible actualmente para las versiones de Intel o AMD de Azure SQL Edge. No está disponible para la versión ARM64 de Azure SQL Edge.|

### <a name="example-create-an-external-stream-inputoutput-object-for-azure-iot-edge-hub"></a>Ejemplo: Creación de un objeto de entrada o salida de transmisión externa para el centro de Azure IoT Edge

En el ejemplo siguiente se crea un objeto de transmisión externa para el centro de Azure IoT Edge. Para crear un origen de datos de entrada/salida de transmisión externa para el centro de Azure IoT Edge, primero debe crear un formato de archivo externo para el diseño de los datos que se leen o escriben.

1. Creación de un formato de archivo externo del tipo JSON.

    ```sql
    Create External file format InputFileFormat
    WITH 
    (  
       format_type = JSON,
    )
    go
    ```

2. Cree un origen de datos externo para el centro de Azure IoT Edge. El siguiente script de T-SQL crea una conexión de origen de datos a un centro de IoT Edge que se ejecuta en el mismo host de Docker que Azure SQL Edge.

    ```sql
    CREATE EXTERNAL DATA SOURCE EdgeHubInput 
    WITH 
    (
        LOCATION = 'edgehub://'
    )
    go
    ```

3. Cree el objeto de transmisión externa para el centro de Azure IoT Edge. El siguiente script de T-SQL crea un objeto de transmisión para el centro de IoT Edge. En el caso de un objeto de transmisión del centro de IoT Edge, el parámetro LOCATION es el nombre del tema o canal del centro de IoT Edge en el que se lee o se escribe.

    ```sql
    CREATE EXTERNAL STREAM MyTempSensors 
    WITH 
    (
        DATA_SOURCE = EdgeHubInput,
        FILE_FORMAT = InputFileFormat,
        LOCATION = N'TemperatureSensors',
        INPUT_OPTIONS = N'',
        OUTPUT_OPTIONS = N''
    );
    go
    ```

### <a name="example-create-an-external-stream-object-to-azure-sql-database"></a>Ejemplo: Creación de un objeto de transmisión externa en Azure SQL Database

En el ejemplo siguiente se crea un objeto de transmisión externa en la base de datos local de Azure SQL Edge. 

1. Cree una clave maestra en la base de datos. Esto es necesario para cifrar el secreto de credencial.

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<<Strong_Password_For_Master_Key_Encryption>>';
    ```

2. Cree una credencial de ámbito de base de datos para acceder al origen de SQL Server. En el ejemplo siguiente se crea una credencial para el origen de datos externo con IDENTITY = username y SECRET = password.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL SQLCredential
    WITH IDENTITY = '<SQL_Login>', SECRET = '<SQL_Login_PASSWORD>'
    go
    ```

3. Cree un origen de datos externo con CREATE EXTERNAL DATA SOURCE. En el ejemplo siguiente:

    * Se crea un origen de datos externo denominado *LocalSQLOutput*.
    * Se identifica el origen de datos externo (LOCATION = '<vendor>://<server>[:<port>]'). En el ejemplo, se apunta a una instancia local de Azure SQL Edge.
    * Se usa la credencial creada antes.

    ```sql
    CREATE EXTERNAL DATA SOURCE LocalSQLOutput 
    WITH 
    (
        LOCATION = 'sqlserver://tcp:.,1433',
        CREDENTIAL = SQLCredential
    )
    go
    ```

4. Cree el objeto de transmisión externa. En el ejemplo siguiente se crea un objeto de transmisión externo que apunta a una tabla *dbo.TemperatureMeasurements* en la base de datos *MySQLDatabase*.

    ```sql
    CREATE EXTERNAL STREAM TemperatureMeasurements 
    WITH 
    (
        DATA_SOURCE = LocalSQLOutput,
        LOCATION = N'MySQLDatabase.dbo.TemperatureMeasurements',
        INPUT_OPTIONS = N'',
        OUTPUT_OPTIONS = N''
    );
    ```

### <a name="example-create-an-external-stream-object-for-kafka"></a>Ejemplo: Creación de un objeto de transmisión externos para Kafka

En el ejemplo siguiente se crea un objeto de transmisión externa en la base de datos local de Azure SQL Edge. En este ejemplo se da por supuesto que el servidor de Kafka está configurado para el acceso anónimo. 

1. Cree un origen de datos externo con CREATE EXTERNAL DATA SOURCE. En el ejemplo siguiente:

    ```sql
    Create EXTERNAL DATA SOURCE [KafkaInput] 
    With
    (
        LOCATION = N'kafka://<kafka_bootstrap_server_name_ip>:<port_number>'
    )
    GO
    ```
2. Cree un formato de archivo externo para la entrada de Kafka. En el ejemplo siguiente se crea un formato de archivo JSON con compresión GZIP. 

   ```sql
   CREATE EXTERNAL FILE FORMAT JsonGzipped  
    WITH 
    (  
        FORMAT_TYPE = JSON , 
        DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec' 
    )
   ```
    
3. Cree el objeto de transmisión externa. En el ejemplo siguiente se crea un objeto de transmisión externa que apunta al tema de Kafka `*TemperatureMeasurement*`.

    ```sql
    CREATE EXTERNAL STREAM TemperatureMeasurement 
    WITH 
    (  
        DATA_SOURCE = KafkaInput, 
        FILE_FORMAT = JsonGzipped,
        LOCATION = 'TemperatureMeasurement',     
        INPUT_OPTIONS = 'PARTITIONS: 10' 
    ); 
    ```

## <a name="create-the-streaming-job-and-the-streaming-queries"></a>Creación del trabajo de streaming y las consultas de streaming

Use el procedimiento almacenado del sistema `sys.sp_create_streaming_job` para definir las consultas de streaming y crear el trabajo de streaming. El procedimiento almacenado `sp_create_streaming_job` utiliza los siguientes parámetros:

- `job_name`: nombre del trabajo de streaming. Los nombres de los trabajos de streaming son únicos en la instancia.
- `statement`: instrucciones de consultas de streaming basadas en el [lenguaje de consulta de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference?).

En el ejemplo siguiente se crea un trabajo de streaming simple con una consulta de streaming. Esta consulta lee las entradas del centro de IoT Edge y escribe en `dbo.TemperatureMeasurements` en la base de datos.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob1',
@statement= N'Select * INTO TemperatureMeasurements from MyEdgeHubInput'
```

En el ejemplo siguiente se crea un trabajo de streaming más complejo con varias consultas diferentes. Estas consultas incluyen una que utiliza la función integrada `AnomalyDetection_ChangePoint` para identificar anomalías en los datos de temperatura.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob2', @statement=
N' Select * INTO TemperatureMeasurements1 from MyEdgeHubInput1

Select * Into TemperatureMeasurements2 from MyEdgeHubInput2

Select * Into TemperatureMeasurements3 from MyEdgeHubInput3

SELECT
Timestamp as [Time],
[Temperature] As [Temperature],
GetRecordPropertyValue(AnomalyDetection_ChangePoint(Temperature, 80, 1200) OVER(LIMIT DURATION(minute, 20)), ''Score'') as ChangePointScore,
GetRecordPropertyValue(AnomalyDetection_ChangePoint(Temperature, 80, 1200) OVER(LIMIT DURATION(minute, 20)), ''IsAnomaly'') as IsChangePointAnomaly
INTO TemperatureAnomalies FROM MyEdgeHubInput2;
'
go
```

## <a name="start-stop-drop-and-monitor-streaming-jobs"></a>Inicio, detención, abandono y supervisión de trabajos de streaming

Para iniciar un trabajo de streaming en Azure SQL Edge, ejecute el procedimiento almacenado `sys.sp_start_streaming_job`. El procedimiento almacenado requiere que se inicie el nombre del trabajo de streaming como entrada.

```sql
exec sys.sp_start_streaming_job @name=N'StreamingJob1'
go
```

Para detener un trabajo de streaming, ejecute el procedimiento almacenado `sys.sp_stop_streaming_job`. El procedimiento almacenado requiere que se detenga el nombre del trabajo de streaming como entrada.

```sql
exec sys.sp_stop_streaming_job @name=N'StreamingJob1'
go
```

Para abandonar (o eliminar) un trabajo de streaming, ejecute el procedimiento almacenado `sys.sp_drop_streaming_job`. El procedimiento almacenado requiere que se abandone el nombre del trabajo de streaming como entrada.

```sql
exec sys.sp_drop_streaming_job @name=N'StreamingJob1'
go
```

Para obtener el estado actual de un trabajo de streaming, ejecute el procedimiento almacenado `sys.sp_get_streaming_job`. El procedimiento almacenado requiere que se abandone el nombre del trabajo de streaming como entrada. Proporciona el nombre y el estado actual del trabajo de streaming.

```sql
exec sys.sp_get_streaming_job @name=N'StreamingJob1'
        WITH RESULT SETS
(
       (
       name nvarchar(256),
       status nvarchar(256)
       )
)
```

El trabajo de streaming puede tener cualquiera de los siguientes estados:

| Estado | Descripción |
|--------| ------------|
| Creado | El trabajo de streaming se ha creado, pero aún no se ha iniciado. |
| Iniciando | El trabajo de streaming se encuentra en la fase de inicio. |
| Inactivo | El trabajo de streaming se está ejecutando, pero no hay ninguna entrada para procesar. |
| Processing | El trabajo de streaming se está ejecutando y está procesando entradas. Este estado indica un estado de mantenimiento del trabajo de streaming. |
| Degradado | El trabajo de streaming se está ejecutando, pero se han producido algunos errores no graves durante el procesamiento de la entrada. El trabajo de entrada seguirá ejecutándose, pero quitará las entradas que encuentren errores. |
| Detenido | El trabajo de streaming se ha detenido. |
| Con error | Error del trabajo de streaming. Suele indicar un error grave durante el procesamiento. |

## <a name="next-steps"></a>Pasos siguientes

- [Visualización de los metadatos asociados a los trabajos de streaming en Azure SQL Edge](streaming-catalog-views.md) 
- [Creación de una transmisión externa](create-external-stream-transact-sql.md)
