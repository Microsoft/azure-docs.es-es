---
title: 'CREATE EXTERNAL STREAM (Transact-SQL): Azure SQL Edge'
description: Obtenga información sobre la instrucción CREATE EXTERNAL STREAM en Azure SQL Edge
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: e28ce4cd46cb802241e02e4060441747389d3989
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888173"
---
# <a name="create-external-stream-transact-sql"></a>CREATE EXTERNAL STREAM (Transact-SQL)

El objeto EXTERNAL STREAM tiene una finalidad doble de un flujo de entrada y salida. Se puede usar como entrada para consultar datos de streaming de servicios de ingesta de eventos, como Azure Event Hub, Azure IoT Hub, o centro de Edge, o Kafka, o bien se puede usar como salida para especificar dónde y cómo almacenar los resultados de una consulta de streaming.

También se puede especificar y crear un parámetro EXTERNAL STREAM como salida y como entrada para servicios como Event Hubs o Blob Storage. Facilita escenarios de encadenamiento en los que una consulta de streaming conserva los resultados en la transmisión externa como salida, y otra consulta de streaming los lee de la misma transmisión externa como entrada.

Azure SQL Edge actualmente solo admite los siguientes orígenes de datos como entradas y salidas de transmisión.

| Tipo de origen de datos | Entrada | Output | Descripción |
|------------------|-------|--------|------------------|
| Centro de Azure IoT Edge | Y | Y | Origen de datos para leer y escribir datos de streaming en un centro de Azure IoT Edge. Para más información, consulte [Centro de IoT Edge](https://docs.microsoft.com/azure/iot-edge/iot-edge-runtime#iot-edge-hub).|
| SQL Database | N | Y | Conexión de origen de datos para escribir datos de streaming en SQL Database. La base de datos puede ser una base de datos local de Azure SQL Edge, o una base de datos remota de SQL Server o Azure SQL Database.|
| Kafka | Y | N | Origen de datos para leer datos de streaming de un tema de Kafka. La compatibilidad con Kafka no está disponible para la versión ARM64 de Azure SQL Edge.|



## <a name="syntax"></a>Sintaxis

```sql
CREATE EXTERNAL STREAM {external_stream_name}  
( <column_definition> [, <column_definition> ] * ) -- Used for Inputs - optional 
WITH  ( <with_options> )

<column_definition> ::=
  column_name <column_data_type>

<data_type> ::=
[ type_schema_name . ] type_name
    [ ( precision [ , scale ] | max ) ]

<with_options> ::=
  DATA_SOURCE = data_source_name, 
  LOCATION = location_name, 
  [FILE_FORMAT = external_file_format_name], --Used for Inputs - optional 
  [<optional_input_options>],
  [<optional_output_options>], 
  TAGS = <tag_column_value>

<optional_input_options> ::= 
  INPUT_OPTIONS = '[<Input_options_data>]'

<Input_option_data> ::= 
      <input_option_values> [ , <input_option_values> ]

<input_option_values> ::=
  PARTITIONS: [number_of_partitions]
  | CONSUMER_GROUP: [ consumer_group_name ] 
  | TIME_POLICY: [ time_policy ] 
  | LATE_EVENT_TOLERANCE: [ late_event_tolerance_value ] 
  | OUT_OF_ORDER_EVENT_TOLERANCE: [ out_of_order_tolerance_value ]
  
<optional_output_options> ::= 
  OUTPUT_OPTIONS = '[<output_option_data>]'

<output_option_data> ::= 
      <output_option_values> [ , <output_option_values> ]

<output_option_values> ::=
   REJECT_POLICY: [ reject_policy ] 
   | MINIMUM_ROWS: [ row_value ] 
   | MAXIMUM_TIME: [ time_value_minutes] 
   | PARTITION_KEY_COLUMN: [ partition_key_column_name ] 
   | PROPERTY_COLUMNS: [ ( [ output_col_name ] ) ] 
   | SYSTEM_PROPERTY_COLUMNS: [ ( [ output_col_name ] ) ] 
   | PARTITION_KEY: [ partition_key_name ] 
   | ROW_KEY: [ row_key_name ] 
   | BATCH_SIZE: [ batch_size_value ] 
   | MAXIMUM_BATCH_COUNT: [ batch_value ] 
   | STAGING_AREA: [ blob_data_source ]
 
<tag_column_value> ::= -- Reserved for Future Usage
); 
```

## <a name="arguments"></a>Argumentos

- [DATA_SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql/)
- [FILE_FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql/)
- **LOCATION**: especifica el nombre de los datos o la ubicación reales en el origen de datos. 
   - En el caso de objetos de transmisión del centro de Edge o de Kafka, la ubicación especifica el nombre del centro de Edge o el tema Kafka del que se va a leer o en el que se va a escribir.
   - Si se trata de la ubicación de los objetos de transmisión de SQL (SQL Server, Azure SQL Database o Azure SQL Edge), especifica el nombre de la tabla. Si el flujo se crea en la misma base de datos y en el mismo esquema que la tabla de destino, con el nombre Tabla es suficiente. En caso contrario, debe proporcionar el nombre completo de la tabla (<database_name.schema_name.table_name).
   - En caso de la ubicación de los objetos de transmisión de Azure Blob Storage, consulte el patrón de la ruta de acceso que se va a usar dentro del contenedor de blobs. Para más información sobre esta característica, vea (/articles/stream-analytics/stream-analytics-define-outputs.md#blob-storage-and-azure-data-lake-gen2).

- **INPUT_OPTIONS**: especifique opciones como pares clave-valor para servicios como Kafka o el centro de IoT Edge que sean entradas para las consultas de streaming.
    - PARTITIONS: número de particiones definidas para un tema. El número máximo de particiones que se pueden utilizar está limitado a 32.
      - Se aplica a los flujos de entrada de Kafka.
    - CONSUMER_GROUP: Event Hubs e IoT Hub limitan el número de lectores dentro de un grupo de consumidores (a 5). Si deja este campo en blanco se usará el grupo de consumidores "$Default".
      - Reservado para uso futuro. No se aplica a Azure SQL Edge.  
    - TIME_POLICY: describe si se deben quitar eventos o ajustar la hora del evento cuando eventos con demora o eventos desordenados superen el valor de tolerancia.
      - Reservado para uso futuro. No se aplica a Azure SQL Edge.
    - LATE_EVENT_TOLERANCE: retraso máximo aceptable. El retraso representa la diferencia entre la marca de tiempo del evento y el reloj del sistema.
      - Reservado para uso futuro. No se aplica a Azure SQL Edge.
    - OUT_OF_ORDER_EVENT_TOLERANCE: Los eventos pueden llegar desordenados después de haber realizado el viaje desde la entrada a la consulta de streaming. Estos eventos se pueden aceptar tal cual, o puede optar por pausarlos durante un período establecido para reordenarlos.
      - Reservado para uso futuro. No se aplica a Azure SQL Edge.
        
- **OUTPUT_OPTIONS**: especifique opciones como pares clave-valor para los servicios admitidos que sean entradas para las consultas de streaming. 
  - REJECT_POLICY:  DROP | RETRY especifica directivas de control de errores de datos cuando se producen errores de conversión de datos. 
    - Se aplica a todas las salidas admitidas. 
  - MINIMUM_ROWS:  
    Filas mínimas requeridas por lote escrito en una salida. En el caso de Parquet, cada lote creará un archivo. 
    - Se aplica a todas las salidas admitidas. 
  - MAXIMUM_TIME:  
    tiempo de espera máximo en minutos por lote. Después de este tiempo, el lote se escribirá en la salida aunque no se cumpla el requisito de filas mínimas. 
    - Se aplica a todas las salidas admitidas. 
  - PARTITION_KEY_COLUMN:  
    columna que se usa para la clave de partición. 
    - Reservado para uso futuro. No se aplica a Azure SQL Edge.
  - PROPERTY_COLUMNS:  
    lista separada por comas de los nombres de las columnas de salida que se adjuntarán a los mensajes como propiedades personalizadas, si se proporcionan.  
    - Reservado para uso futuro. No se aplica a Azure SQL Edge. 
  - SYSTEM_PROPERTY_COLUMNS:  
    colección con formato JSON de pares nombre-valor de los nombres de propiedades del sistema y columnas de salida que se rellenarán en los mensajes de Service Bus. P. ej.: { "MessageId": "column1", "PartitionKey": "column2"}. 
    - Reservado para uso futuro. No se aplica a Azure SQL Edge. 
  - PARTITION_KEY:  
    El nombre de la columna de salida que contiene la clave de partición. La clave de partición es un identificador único de la partición dentro de una tabla determinada que constituye la primera parte de la clave principal de la entidad. Es un valor de cadena que puede tener un tamaño de hasta 1 KB. 
    - Reservado para uso futuro. No se aplica a Azure SQL Edge.
  - ROW_KEY:  
    El nombre de la columna de salida que contiene la clave de la fila. La clave de fila es un identificador único de una identidad dentro de una partición determinada. Forma la segunda parte de la clave principal de la entidad. La clave de fila es un valor de cadena que puede tener un tamaño de hasta 1 KB. 
    - Reservado para uso futuro. No se aplica a Azure SQL Edge.
  - BATCH_SIZE:  
    representa el número de transacciones de Table Storage donde el máximo puede ascender a 100 registros. Para Azure Functions, representa el tamaño del lote en bytes enviados a la función por llamada; el valor predeterminado es 256 kB. 
    - Reservado para uso futuro. No se aplica a Azure SQL Edge. 
  - MAXIMUM_BATCH_COUNT:  
    número máximo de eventos enviados a la función por cada llamada para Azure Functions. El valor predeterminado es de 100. Para SQL Database, representa el número máximo de registros que se envía con cada transacción de inserción masiva. El valor predeterminado es 10 000. 
    - Se aplica a todas las salidas basadas en SQL. 
  - STAGING_AREA: objeto EXTERNAL DATA SOURCE para Blob Storage. El área de almacenamiento provisional para la ingesta de datos de alto rendimiento en Azure Synapse Analytics. 
    - Reservado para uso futuro. No se aplica a Azure SQL Edge.


## <a name="examples"></a>Ejemplos

### <a name="example-1---edgehub"></a>Ejemplo 1: centro de Edge

Escriba:  Entrada o salida<br>

Sintaxis:

```sql
CREATE EXTERNAL DATA SOURCE MyEdgeHub 
WITH  
(      
  LOCATION = 'edgehub://'       
); 
 
CREATE EXTERNAL FILE FORMAT myFileFormat  
WITH (  
   FORMAT_TYPE = 'JSON', 
); 
 
CREATE EXTERNAL STREAM Stream_A  
WITH    
(   
   DATA_SOURCE = MyEdgeHub, 
   FILE_FORMAT = myFileFormat, 
   LOCATION = '<mytopicname>', 
   OUTPUT_OPTIONS =   
     'REJECT_TYPE: Drop'
);
```

### <a name="example-2---azure-sql-database-azure-sql-edge-sql-server"></a>Ejemplo 2: Azure SQL Database, Azure SQL Edge, SQL Server

Escriba:  Output<br>

Sintaxis:

```sql
CREATE DATABASE SCOPED CREDENTIAL SQLCredName 
WITH IDENTITY = '<user>', 
SECRET = '<password>'; 
 
-- Azure SQL Database 
CREATE EXTERNAL DATA SOURCE MyTargetSQLTabl 
WITH 
(     
  LOCATION = '<my_server_name>.database.windows.net', 
  CREDENTIAL = SQLCredName 
); 
 
--SQL Server or Azure SQL Edge
CREATE EXTERNAL DATA SOURCE MyTargetSQLTabl 
WITH 
(     
  LOCATION = ' <sqlserver://<ipaddress>,<port>', 
  CREDENTIAL = SQLCredName 
); 

CREATE EXTERNAL STREAM Stream_A 
WITH   
(  
    DATA_SOURCE = MyTargetSQLTable, 
    LOCATION = '<DatabaseName>.<SchemaName>.<TableName>' ,
   --Note: If table is contained in the database, <TableName> should be sufficient 
    OUTPUT_OPTIONS =  
      'REJECT_TYPE: Drop'
); 
```

### <a name="example-3---kafka"></a>Ejemplo 3: Kafka

Escriba:  Entrada<br>

Sintaxis:
```sql
CREATE EXTERNAL DATA SOURCE MyKafka_tweets 
WITH 
( 
  --The location maps to KafkaBootstrapServer 
  LOCATION = 'kafka://<kafkaserver>:<ipaddress>', 
  CREDENTIAL = kafkaCredName 
); 
 
CREATE EXTERNAL FILE FORMAT myFileFormat 
WITH ( 
    FORMAT_TYPE = JSON, 
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'
); 

CREATE EXTERNAL STREAM Stream_A (user_id VARCHAR, tweet VARCHAR) 
WITH   
(  
    DATA_SOURCE = MyKafka_tweets, 
    LOCATION = '<KafkaTopicName>', 
    FILE_FORMAT = myFileFormat,  
    INPUT_OPTIONS =  
      'PARTITIONS: 5'
); 
```

## <a name="see-also"></a>Consulte también

- [DROP EXTERNAL STREAM (Transact-SQL)](drop-external-stream-transact-sql.md) 

