---
title: Importación y exportación de datos entre grupos de Spark (versión preliminar) y grupos de SQL
description: En este artículo se proporciona información sobre cómo usar el conector personalizado para mover datos entre grupos de SQL y grupos de Spark (versión preliminar).
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 04/15/2020
ms.author: prgomata
ms.reviewer: euang
ms.openlocfilehash: 7b02296d5d9aed5866c0efcdf85fa1c9946617d0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501906"
---
# <a name="introduction"></a>Introducción

El conector entre Azure Synapse Apache Spark y Synapse SQL está diseñado para transferir datos de forma eficaz entre los grupos de Spark (versión preliminar) y los grupos de SQL de Azure Synapse. El conector entre Azure Synapse Apache Spark y Synapse SQL solo funciona en grupos SQL, no funciona con SQL On-demand (SQL a petición).

## <a name="design"></a>Diseño

La transferencia de datos entre grupos de Spark y grupos de SQL se puede realizar mediante JDBC. Sin embargo, dados dos sistemas distribuidos, como grupos de SQL y de Spark, JDBC tiende a convertirse en un cuello de botella con la transferencia de datos serie.

El conector entre el grupo de Azure Synapse Apache Spark y Synapse SQL es una implementación de origen de datos para Apache Spark. Se emplea Azure Data Lake Storage Gen2 y Polybase en los grupos de SQL para transferir datos de forma eficaz entre el clúster de Spark y la instancia de Synapse SQL.

![Arquitectura del conector](./media/synapse-spark-sqlpool-import-export/arch1.png)

## <a name="authentication-in-azure-synapse-analytics"></a>Autenticación en Azure Synapse Analytics

La autenticación entre sistemas se realiza sin problemas en Azure Synapse Analytics. Hay un servicio de token que se conecta con Azure Active Directory para obtener los tokens de seguridad que se van a usar al acceder a la cuenta de almacenamiento o al servidor de almacenamiento de datos.

Por esta razón, no es necesario crear credenciales ni especificarlas en la API del conector, siempre y cuando AAD-Auth esté configurado en la cuenta de almacenamiento y el servidor de almacenamiento de datos. Si no es así, se puede especificar la autenticación de SQL. Encontrará más detalles en la sección [Uso](#usage).

## <a name="constraints"></a>Restricciones

- Este conector solo funciona en Scala.

## <a name="prerequisites"></a>Requisitos previos

- Debe ser miembro del rol **db_exporter** en la base de datos o el grupo de SQL al que desea transferir datos, o del que desea recibirlos.
- Debe ser miembro del rol Colaborador de datos de Storage Blob de la cuenta de almacenamiento predeterminada.

Para crear usuarios, conéctese a la base de datos del grupo de SQL y siga estos ejemplos:

```sql
--SQL User
CREATE USER Mary FROM LOGIN Mary;

--Azure Active Directory User
CREATE USER [mike@contoso.com] FROM EXTERNAL PROVIDER;
```

Para asignar un rol:

```sql
--SQL User
EXEC sp_addrolemember 'db_exporter', 'Mary';

--Azure Active Directory User
EXEC sp_addrolemember 'db_exporter',[mike@contoso.com]
```

## <a name="usage"></a>Uso

Las instrucciones de importación no son necesarias, ya que se importan previamente para la experiencia del cuaderno.

### <a name="transfer-data-to-or-from-a-sql-pool-attached-with-the-workspace"></a>Transferencia de datos hacia un grupo de SQL asociado al área de trabajo, o desde él

> [!NOTE]
> **Importaciones no necesarias en la experiencia del cuaderno**

```scala
 import com.microsoft.spark.sqlanalytics.utils.Constants
 import org.apache.spark.sql.SqlAnalyticsConnector._
```

#### <a name="read-api"></a>Read API

```scala
val df = spark.read.sqlanalytics("<DBName>.<Schema>.<TableName>")
```

La API anterior funciona tanto con tablas internas (administradas) como externas en el grupo de SQL.

#### <a name="write-api"></a>Write API

```scala
df.write.sqlanalytics("<DBName>.<Schema>.<TableName>", <TableType>)
```

Write API crea la tabla en el grupo de SQL y, después, invoca Polybase para cargar los datos.  La tabla no debe existir en el grupo de SQL o se devolverá un error que indica que "Ya hay un objeto llamado..."

Valores de TableType

- Constantes.INTERNAL: tabla administrada en el grupo de SQL
- Constantes.EXTERNAL: tabla externa en el grupo de SQL

Tabla administrada del grupo de SQL

```scala
df.write.sqlanalytics("<DBName>.<Schema>.<TableName>", Constants.INTERNAL)
```

Tabla externa del grupo de SQL

Para escribir en una tabla externa del grupo de SQL, debe existir un ORIGEN DE DATOS EXTERNO y un FORMATO DE ARCHIVO EXTERNO en el grupo de SQL.  Para más información, lea el artículo acerca de cómo [crear un origen de datos externo](/sql/t-sql/statements/create-external-data-source-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) y [formatos de archivo externos](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) en un grupo de SQL.  A continuación, encontrará ejemplos de creación de un origen de datos externo y formatos de archivo externos en un grupo de SQL.

```sql
--For an external table, you need to pre-create the data source and file format in SQL pool using SQL queries:
CREATE EXTERNAL DATA SOURCE <DataSourceName>
WITH
  ( LOCATION = 'abfss://...' ,
    TYPE = HADOOP
  ) ;

CREATE EXTERNAL FILE FORMAT <FileFormatName>
WITH (  
    FORMAT_TYPE = PARQUET,  
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'  
);
```

Si se utiliza la autenticación de paso a través de Azure Active Directory en la cuenta de almacenamiento, no se necesita ningún objeto EXTERNAL CREDENTIAL.  Asegúrese de que es miembro del rol "Colaborador de datos de Storage Blob" de la cuenta de almacenamiento.

```scala

df.write.
    option(Constants.DATA_SOURCE, <DataSourceName>).
    option(Constants.FILE_FORMAT, <FileFormatName>).
    sqlanalytics("<DBName>.<Schema>.<TableName>", Constants.EXTERNAL)

```

### <a name="if-you-transfer-data-to-or-from-a-sql-pool-or-database-outside-the-workspace"></a>Si transfiere datos hacia un grupo de SQL o una base de datos fuera del área de trabajo, o desde él

> [!NOTE]
> Importaciones no necesarias en la experiencia del cuaderno

```scala
 import com.microsoft.spark.sqlanalytics.utils.Constants
 import org.apache.spark.sql.SqlAnalyticsConnector._
```

#### <a name="read-api"></a>Read API

```scala
val df = spark.read.
option(Constants.SERVER, "samplews.database.windows.net").
sqlanalytics("<DBName>.<Schema>.<TableName>")
```

#### <a name="write-api"></a>Write API

```scala
df.write.
option(Constants.SERVER, "samplews.database.windows.net").
sqlanalytics("<DBName>.<Schema>.<TableName>", <TableType>)
```

### <a name="use-sql-auth-instead-of-aad"></a>Uso de la autenticación de SQL, en lugar de AAD

#### <a name="read-api"></a>Read API

Actualmente, el conector no admite la autenticación basada en token en un grupo de SQL que esté fuera del área de trabajo. Tendrá que usar la autenticación de SQL.

```scala
val df = spark.read.
option(Constants.SERVER, "samplews.database.windows.net").
option(Constants.USER, <SQLServer Login UserName>).
option(Constants.PASSWORD, <SQLServer Login Password>).
sqlanalytics("<DBName>.<Schema>.<TableName>")
```

#### <a name="write-api"></a>Write API

```scala
df.write.
option(Constants.SERVER, "samplews.database.windows.net").
option(Constants.USER, <SQLServer Login UserName>).
option(Constants.PASSWORD, <SQLServer Login Password>).
sqlanalytics("<DBName>.<Schema>.<TableName>", <TableType>)
```

### <a name="use-the-pyspark-connector"></a>Uso del conector de PySpark

> [!NOTE]
> Este ejemplo está pensado únicamente para la experiencia del cuaderno.

Suponga que tiene una trama de datos "pyspark_df" que quiere escribir en el almacenamiento de datos.

Cree una tabla temporal mediante la trama de datos en PySpark:

```py
pyspark_df.createOrReplaceTempView("pysparkdftemptable")
```

Ejecute una celda de Scala en el cuaderno de PySpark mediante magics:

```scala
%%spark
val scala_df = spark.sqlContext.sql ("select * from pysparkdftemptable")

pysparkdftemptable.write.sqlanalytics("sqlpool.dbo.PySparkTable", Constants.INTERNAL)
```

Del mismo modo, en el escenario de lectura, lea los datos mediante Scala y escríbalos en una tabla temporal, y use Spark SQL en PySpark para consultar la tabla temporal en una trama de datos.

## <a name="allow-other-users-to-use-the-azure-synapse-apache-spark-to-synapse-sql-connector-in-your-workspace"></a>Permitir que otros usuarios empleen el conector de Apache Spark de Azure Synapse a Synapse SQL en su área de trabajo

Para modificar los permisos que faltan para otros usuarios es preciso ser propietario de los datos de Storage Blob en la cuenta de almacenamiento de ADLS Gen2 conectada al área de trabajo. Asegúrese de que el usuario tiene acceso al área de trabajo y permisos para ejecutar cuadernos.

### <a name="option-1"></a>Opción 1

- Convertir al usuario en un colaborador o propietario de datos de Storage Blob

### <a name="option-2"></a>Opción 2

- Especifique las siguientes ACL en la estructura de carpetas:

| Carpeta | / | synapse | workspaces  | \<workspacename> | sparkpools | \<sparkpoolname>  | sparkpoolinstances  |
|--|--|--|--|--|--|--|--|
| Permisos de acceso | --X | --X | --X | --X | --X | --X | -WX |
| Permisos predeterminados | ---| ---| ---| ---| ---| ---| ---|

- Debe ser capaz de usar la lista de control de acceso en todas las carpetas de "synapse" y hacia abajo desde Azure Portal. Para usar una lista de control de acceso en la carpeta raíz "/", siga las instrucciones que se indican a continuación.

- Conexión a la cuenta de almacenamiento conectada con el área de trabajo desde el Explorador de Storage mediante AAD
- Seleccione su cuenta y proporcione la dirección URL de ADLS Gen2 y el sistema de archivos predeterminado para el área de trabajo
- Una vez que pueda ver la cuenta de almacenamiento en la lista, haga clic con el botón derecho en el área de trabajo de la lista y seleccione "Manage Access" (Administrar acceso).
- Agregue el usuario a la carpeta / con el permiso de acceso de "ejecución". Seleccione "OK" (Aceptar).

> [!IMPORTANT]
> Asegúrese de no seleccionar "predeterminado" si no tiene previsto hacerlo.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un grupo de SQL mediante Azure Portal](../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md)
- [Creación de un grupo de Apache Spark mediante Azure Portal](../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md) 
