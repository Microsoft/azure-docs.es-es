---
title: Tablas de metadatos compartidos
description: Azure Synapse Analytics proporciona un modelo de metadatos compartidos donde la creación de una tabla en Apache Spark hará que sea accesible desde sus motores de SQL a petición (versión preliminar) y de grupo de SQL sin duplicar los datos.
services: sql-data-warehouse
author: MikeRys
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: metadata
ms.date: 05/01/2020
ms.author: mrys
ms.reviewer: jrasnick
ms.openlocfilehash: 28f666fe295b2b49fb6795306e9fad489c867517
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387224"
---
# <a name="azure-synapse-analytics-shared-metadata-tables"></a>Tablas de metadatos compartidos de Azure Synapse Analytics

[!INCLUDE [synapse-analytics-preview-terms](../../../includes/synapse-analytics-preview-terms.md)]

Azure Synapse Analytics permite que los diferentes motores de cálculo de áreas de trabajo compartan bases de datos y tablas con el respaldo de Parquet entre sus grupos de Apache Spark (versión preliminar) y el motor de SQL On-Demand (versión preliminar).

Una vez creada una base de datos con un trabajo de Spark, puede crear en ella, mediante Spark, tablas que usen Parquet como formato de almacenamiento. Estas tablas estarán disponibles de forma inmediata para que cualquiera de los grupos de Spark del área de trabajo de Azure Synapse realice consultas en ellas. También se pueden usar desde cualquiera de los trabajos de Spark sujetos a permisos.

Las tablas creadas, administradas y externas de Spark también están disponibles como tablas externas con el mismo nombre en la base de datos sincronizada correspondiente en SQL On-Demand. [La exposición de una tabla de Spark en SQL](#expose-a-spark-table-in-sql) proporciona más información sobre la sincronización de la tabla.

Dado que las tablas se sincronizan con SQL On-Demand de forma asincrónica, se producirá un retraso hasta que aparezcan.

## <a name="manage-a-spark-created-table"></a>Administración de una tabla creada con Spark

Use Spark para administrar las bases de datos creadas con Spark. Por ejemplo, elimínelas mediante un trabajo de grupo de Spark y cree tablas en ellas desde Spark.

Si crea objetos en una base de datos de este tipo desde SQL a petición o intenta anular la base de datos, la operación se realizará correctamente, pero la base de datos original de Spark no se cambiará.

## <a name="expose-a-spark-table-in-sql"></a>Exposición de una tabla de Spark en SQL

### <a name="shared-spark-tables"></a>Tablas de Spark compartidas

Spark proporciona dos tipos de tablas que Azure Synapse expone en SQL automáticamente:

- Tablas administradas

  Spark proporciona muchas opciones para almacenar datos en tablas administradas, como TEXT, CSV, JSON, JDBC, PARQUET, ORC, HIVE, DELTA y LIBSVM. Normalmente, estos archivos se almacenan en el directorio `warehouse` en donde se guardan los datos de las tablas administradas.

- Tablas externas

  Spark también proporciona maneras de crear tablas externas basadas en datos existentes, ya sea proporcionando la opción `LOCATION` o usando el formato de Hive. Estas tablas externas se pueden realizar con diferentes formatos de datos, incluido Parquet.

En la actualidad, Azure Synapse solo comparte tablas de Spark administradas y externas que almacenan sus datos en formato Parquet con los motores SQL. Las tablas que se basan en otros formatos no se sincronizan automáticamente. Es posible que pueda sincronizar estas tablas de forma explícita como una tabla externa en su propia base de datos SQL, siempre que el motor de SQL admita el formato subyacente de la tabla.

### <a name="share-spark-tables"></a>Uso compartido de tablas de Spark

Las tablas de Spark administradas y externas que se pueden compartir se exponen en el motor SQL como tablas externas con las siguientes propiedades:

- El origen de datos de la tabla externa de SQL es el origen de datos que representa la carpeta de ubicación de la tabla de Spark.
- El formato de archivo de la tabla externa de SQL es Parquet.
- La credencial de acceso de la tabla externa de SQL es de paso a través.

Dado que todos los nombres de tabla de Spark son nombres válidos para una tabla SQL y los nombres de columna de Spark también son nombres válidos para una columna SQL, los nombres de columna y tabla de Spark se usarán para la tabla externa de SQL.

Las tablas de Spark proporcionan tipos de datos diferentes a los motores de Synapse SQL. En la tabla siguiente se asignan los tipos de datos de la tabla Spark a los tipos SQL:

| Tipo de datos de Spark | Tipo de datos de SQL | Comentarios |
|---|---|---|
| `byte`      | `smallint`       ||
| `short`     | `smallint`       ||
| `integer`   |    `int`            ||
| `long`      |    `bigint`         ||
| `float`     | `real`           |<!-- need precision and scale-->|
| `double`    | `float`          |<!-- need precision and scale-->|
| `decimal`      | `decimal`        |<!-- need precision and scale-->|
| `timestamp` |    `datetime2`      |<!-- need precision and scale-->|
| `date`      | `date`           ||
| `string`    |    `varchar(max)`   | Con intercalación `Latin1_General_CP1_CI_AS_UTF8` |
| `binary`    |    `varbinary(max)` ||
| `boolean`   |    `bit`            ||
| `array`     |    `varchar(max)`   | Serializa en JSON con intercalación `Latin1_General_CP1_CI_AS_UTF8` |
| `map`       |    `varchar(max)`   | Serializa en JSON con intercalación `Latin1_General_CP1_CI_AS_UTF8` |
| `struct`    |    `varchar(max)`   | Serializa en JSON con intercalación `Latin1_General_CP1_CI_AS_UTF8` |

<!-- TODO: Add precision and scale to the types mentioned above -->

## <a name="security-model"></a>Modelo de seguridad

Las bases de datos y las tablas de Spark, al igual que sus representaciones sincronizadas en el motores SQL, estarán protegidas en el nivel de almacenamiento subyacente. Teniendo en cuenta que actualmente no tienen permisos para los propios objetos, estos se pueden visualizar en el explorador de objetos.

La entidad de seguridad que crea una tabla administrada se considera el propietario de dicha tabla y tiene todos los derechos en la tabla, así como en las carpetas y los archivos subyacentes. Además, el propietario de la base de datos se convertirá automáticamente en copropietario de la tabla.

Si crea una tabla externa de Spark o SQL con la autenticación de paso a través, los datos solo se protegen en los niveles de carpeta y archivo. Si alguien consulta este tipo de tabla externa, la identidad de seguridad del remitente de la consulta se pasa al sistema de archivos, que comprobará los derechos de acceso.

Para más información sobre cómo establecer permisos en las carpetas y los archivos, consulte [Base de datos compartida de Azure Synapse Analytics](database.md).

## <a name="examples"></a>Ejemplos

### <a name="create-a-managed-table-backed-by-parquet-in-spark-and-query-from-sql-on-demand"></a>Creación de una tabla administrada respaldada por Parquet en Spark y realización de consultas desde SQL a petición

En este escenario, tiene una base de datos de Spark denominada `mytestdb`. Consulte el apartado [Creación y conexión a una base de datos de Spark con SQL a petición](database.md#create-and-connect-to-spark-database-with-sql-on-demand).

Cree una tabla administrada de Spark con SparkSQL mediante la ejecución del siguiente comando:

```sql
    CREATE TABLE mytestdb.myParquetTable(id int, name string, birthdate date) USING Parquet
```

Esto crea la tabla `myParquetTable` en la base de datos `mytestdb`. Después de un breve retraso, puede ver la tabla en SQL a petición. Por ejemplo, ejecute la siguiente instrucción desde SQL a petición.

```sql
    USE mytestdb;
    SELECT * FROM sys.tables;
```

Compruebe que `myParquetTable` se incluye en los resultados.

>[!NOTE]
>Una tabla que no use Parquet como formato de almacenamiento no se sincronizará.

A continuación, inserte algunos valores en la tabla de Spark, por ejemplo, con las siguientes instrucciones C# de Spark en un cuaderno C#:

```csharp
using Microsoft.Spark.Sql.Types;

var data = new List<GenericRow>();

data.Add(new GenericRow(new object[] { 1, "Alice", new Date(2010, 1, 1)}));
data.Add(new GenericRow(new object[] { 2, "Bob", new Date(1990, 1, 1)}));

var schema = new StructType
    (new List<StructField>()
        {
            new StructField("id", new IntegerType()),
            new StructField("name", new StringType()),
            new StructField("birthdate", new DateType())
        }
    );

var df = spark.CreateDataFrame(data, schema);
df.Write().Mode(SaveMode.Append).InsertInto("mytestdb.myParquetTable");
```

Ahora puede leer los datos de SQL a petición como sigue:

```sql
SELECT * FROM mytestdb.dbo.myParquetTable WHERE name = 'Alice';
```

Debe obtener la siguiente fila como resultado:

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

### <a name="create-an-external-table-backed-by-parquet-in-spark-and-query-from-sql-on-demand"></a>Creación de una tabla externa respaldada por Parquet en Spark y realización de consultas desde SQL a petición

En este ejemplo, cree una tabla de Spark externa con los archivos de datos de Parquet que creó en el ejemplo anterior para la tabla administrada.

Por ejemplo, con SparkSQL ejecute:

```sql
CREATE TABLE mytestdb.myExternalParquetTable
    USING Parquet
    LOCATION "abfss://<fs>@arcadialake.dfs.core.windows.net/synapse/workspaces/<synapse_ws>/warehouse/mytestdb.db/myparquettable/"
```

Reemplace el marcador de posición `<fs>` por el nombre del sistema de archivos que es el sistema de archivos predeterminado del área de trabajo y el marcador de posición `<synapse_ws>` por el nombre del área de trabajo de Synapse que está usando para ejecutar este ejemplo.

En el ejemplo anterior se crea la tabla `myExtneralParquetTable` en la base de datos `mytestdb`. Después de un breve retraso, puede ver la tabla en SQL a petición. Por ejemplo, ejecute la siguiente instrucción desde SQL a petición.

```sql
USE mytestdb;
SELECT * FROM sys.tables;
```

Compruebe que `myExternalParquetTable` se incluye en los resultados.

Ahora puede leer los datos de SQL a petición como sigue:

```sql
SELECT * FROM mytestdb.dbo.myExternalParquetTable WHERE name = 'Alice';
```

Debe obtener la siguiente fila como resultado:

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre los metadatos compartidos de Azure Synapse Analytics](overview.md)
- [Más información sobre las bases de datos de metadatos compartidos de Azure Synapse Analytics](database.md)


