---
title: Descripción de los conceptos de código de Apache Spark para los desarrolladores de U-SQL de Azure Data Lake Analytics.
description: En este artículo se describen conceptos de Apache Spark para ayudar a los desarrolladores de U-SQL a comprender los conceptos de código de Spark.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.custom: Understand-apache-spark-code-concepts
ms.date: 10/15/2019
ms.openlocfilehash: 7b5be20bb8b5eb1d56c1214104037d5d824445b3
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132354"
---
# <a name="understand-apache-spark-code-for-u-sql-developers"></a>Descripción del código de Apache Spark para los desarrolladores de U-SQL

En esta sección se proporcionan instrucciones generales sobre cómo transformar scripts de U-SQL en Apache Spark.

- Empieza con una [comparación de los paradigmas de procesamiento de los dos lenguajes](#understand-the-u-sql-and-spark-language-and-processing-paradigms).
- Proporciona recomendaciones sobre cómo:
   - [Transformar scripts](#transform-u-sql-scripts), incluidas las [expresiones de conjunto de filas](#transform-u-sql-rowset-expressions-and-sql-based-scalar-expressions) de U-SQL.
   - [Código .NET](#transform-net-code).
   - [Tipos de datos](#transform-typed-values)
   - [Objetos de catálogo](#transform-u-sql-catalog-objects).

## <a name="understand-the-u-sql-and-spark-language-and-processing-paradigms"></a>Descripción de los paradigmas de procesamiento y lenguaje de U-SQL y Spark

Antes de empezar a migrar scripts de U-SQL de Azure Data Lake Analytics a Spark, resulta útil comprender las filosofías generales de lenguaje y procesamiento de los dos sistemas.

U-SQL es un lenguaje de consulta declarativo parecido a SQL que utiliza un paradigma de flujo de datos y permite insertar y escalar horizontalmente de forma sencilla el código de usuario escrito en .NET (por ejemplo, C#), Python y R. Las extensiones de usuario pueden implementar expresiones sencillas o funciones definidas por el usuario, pero también pueden proporcionar al usuario la capacidad de implementar los llamados operadores definidos por el usuario, que implementan operadores personalizados para realizar transformaciones de nivel de conjunto de filas, extracciones y escritura de salida.

Spark es un marco de escalado horizontal que ofrece varios enlaces de lenguaje en Scala, Java, Python, .NET, etc., donde el código se escribe principalmente en uno de estos lenguajes, se crean abstracciones de datos llamadas conjuntos de datos distribuidos resistentes (RDD), tramas de datos y conjuntos de datos y, después, se usa un lenguaje específico de dominio (DSL) parecido a LINQ para transformarlos. También proporciona SparkSQL como sublenguaje declarativo en las abstracciones de los conjuntos de datos y tramas de datos. El Lenguaje específico de dominio proporciona dos categorías de operaciones, transformaciones y acciones. Aplicar transformaciones a las abstracciones de datos no ejecutará la transformación sino que, en su lugar, creará el plan de ejecución que se enviará para su evaluación con una acción (por ejemplo, escribiendo el resultado en una tabla o un archivo temporal, o imprimiendo el resultado).

Por tanto, al convertir un script de U-SQL en un programa de Spark, tendrá que decidir qué lenguaje quiere usar para, al menos, generar la abstracción de la trama de datos (que actualmente es la abstracción de datos usada con más frecuencia) y si quiere escribir las transformaciones declarativas de flujos de datos con DSL o SparkSQL. En algunos casos más complejos, es posible que tenga que dividir el script de U-SQL en una secuencia de Spark y en otros pasos implementados con Azure Batch o Azure Functions.

Además, Azure Data Lake Analytics ofrece U-SQL en un entorno de servicio de trabajo sin servidor, mientras que Azure Databricks y Azure HDInsight ofrecen Spark en forma de servicio de clúster. Al transformar la aplicación, deberá tener en cuenta las implicaciones que conlleva ahora crear, cambiar el tamaño, escalar y retirar los clústeres.

## <a name="transform-u-sql-scripts"></a>Transformación de scripts de U-SQL

Los scripts de U-SQL siguen el siguiente patrón de procesamiento:

1. Los datos se leen desde archivos no estructurados, mediante la instrucción `EXTRACT`, una ubicación o una especificación del conjunto de archivos, el extractor integrado o definido por el usuario, y el esquema deseado, o bien desde las tablas U-SQL (tablas administradas o externas). Se representan como un conjunto de filas.
2. Los conjuntos de filas se transforman en varias instrucciones de U-SQL que aplican expresiones de U-SQL a los conjuntos de filas y generan nuevos conjuntos de filas.
3. Por último, los conjuntos de filas resultantes se envían a los dos archivos mediante la instrucción `OUTPUT`, que especifica las ubicaciones y un outputter integrado o definido por el usuario, o a una tabla de U-SQL.

El script se evalúa de forma diferida, lo que significa que cada paso de extracción y transformación se compone en un árbol de expresión y se evalúa globalmente (el flujo de datos).

Los programas de Spark son similares en cuanto a que se usarían conectores de Spark para leer los datos y crear las tramas de datos y, después, se aplicarían las transformaciones en las tramas de datos mediante DSL o SparkSQL de tipo LINQ, y luego se escribiría el resultado en archivos, tablas temporales de Spark, algunos tipos de lenguajes de programación o la consola.

## <a name="transform-net-code"></a>Transformación de código .NET

El lenguaje de expresión de U-SQL es C# y ofrece varias formas de escalar horizontalmente el código .NET personalizado.

Dado que Spark actualmente no admite la ejecución de código .NET de forma nativa, se tendrán que volver a escribir las expresiones en una expresión Spark, Scala, Java o Python equivalente, o bien encontrar una manera de llamar al código .NET. Si el script usa bibliotecas .NET, tiene las siguientes opciones:

- Traducir el código .NET a Scala o Python.
- Dividir el script de U-SQL en varios pasos, en los que use procesos de Azure Batch para aplicar las transformaciones de .NET (si puede obtener una escala aceptable).
- Usar un enlace de lenguaje .NET disponible en código abierto llamado Moebius. Este proyecto no está en un estado admitido.

En cualquier caso, si tiene una gran cantidad de lógica .NET en los scripts de U-SQL, póngase en contacto con nosotros a través de su representante de cuenta de Microsoft para obtener más información.

Los detalles siguientes sirven para los distintos casos de uso de .NET y C# de los scripts de U-SQL.

### <a name="transform-scalar-inline-u-sql-c-expressions"></a>Transformación de expresiones insertadas escalares de C# en U-SQL

El lenguaje de las expresiones de U-SQL es C#. Muchas de las expresiones de U-SQL insertadas escalares se implementan de forma nativa para mejorar el rendimiento, mientras que las expresiones más complejas pueden ejecutarse a través de llamadas a .NET Framework.

Spark tiene su propio lenguaje de expresiones escalares (ya sea como parte de DSL o en SparkSQL) y permite llamar a funciones definidas por el usuario escritas en su lenguaje de hospedaje.

Si tiene expresiones escalares en U-SQL, primero debe buscar la expresión escalar de Spark de forma nativa más adecuada para obtener el máximo rendimiento y, después, asignar las demás expresiones en una función definida por el usuario del lenguaje de hospedaje de Spark que prefiera.

Tenga en cuenta que .NET y C# tienen una semántica de tipos distinta que la de los lenguajes de hospedaje de Spark y que la del DSL de Spark. Vea [a continuación](#transform-typed-values) para obtener más información sobre las diferencias del sistema de tipos.

### <a name="transform-user-defined-scalar-net-functions-and-user-defined-aggregators"></a>Transformación de funciones .NET escalares definidas por el usuario y de agregadores definidos por el usuario

U-SQL proporciona maneras de llamar a funciones de .NET escalares arbitrarias y de llamar a agregadores definidos por el usuario escritos en .NET.

Spark también ofrece compatibilidad con funciones definidas por el usuario y agregadores definidos por el usuario escritos en la mayoría de lenguajes de hospedaje a los que se puede llamar desde el DSL de Spark y SparkSQL.

### <a name="transform-user-defined-operators-udos"></a>Transformación de operadores definidos por el usuario (UDO)

U-SQL proporciona varias categorías de operadores definidos por el usuario (UDO), como extractores, outputters, reductores, procesadores, aplicadores y combinadores que se pueden escribir en .NET (y, hasta cierto punto, en Python y R).

Spark no ofrece el mismo modelo de extensibilidad para los operadores, pero tiene funciones equivalentes para algunos.

El equivalente de Spark a los extractores y outputters son los conectores de Spark. Para muchos extractores de U-SQL, se puede encontrar un conector equivalente en la comunidad de Spark. Para el resto, se tendrá que escribir un conector personalizado. Si el extractor de U-SQL es complejo y utiliza varias bibliotecas .NET, puede ser preferible compilar un conector en Scala que use la interoperabilidad para llamar a la biblioteca .NET que realiza el procesamiento real de los datos. En ese caso, tendrá que implementar el runtime de .NET Core en el clúster de Spark y asegurarse de que las bibliotecas .NET a las que se hace referencia son compatibles con .NET Standard 2.0.

El resto de tipos de UDO de U-SQL deberán volver a escribirse con funciones definidas por el usuario y agregadores, y la expresión DSL de Spark o SparkSQL semánticamente adecuada. Por ejemplo, un procesador se puede asignar a una selección de varias invocaciones de UDF, empaquetadas como una función que toma una trama de datos como argumento y devuelve una trama de datos.

### <a name="transform-u-sqls-optional-libraries"></a>Transformación de bibliotecas de U-SQL opcionales

U-SQL proporciona un conjunto de bibliotecas opcionales y de demostración que ofrecen [Python](data-lake-analytics-u-sql-python-extensions.md), [R](data-lake-analytics-u-sql-r-extensions.md), [compatibilidad con JSON, XML y AVRO](https://github.com/Azure/usql/tree/master/Examples/DataFormats), así como algunas [funciones de servicios cognitivos](data-lake-analytics-u-sql-cognitive.md).

Spark ofrece su propia integración de Python y R, pySpark y SparkR, respectivamente, y proporciona conectores para leer y escribir JSON, XML y AVRO.

Si se necesita transformar un script que hace referencia a las bibliotecas de servicios cognitivos, recomendamos que se ponga en contacto con nosotros a través de su representante de cuenta de Microsoft.

## <a name="transform-typed-values"></a>Transformación de valores de tipo

Dado que el sistema de tipos de U-SQL se basa en el sistema de tipos de .NET y Spark tiene su propio sistema de tipos, que se ve afectado por el enlace de lenguaje del host, tendrá que asegurarse de que los tipos en los que trabaja sean próximos y, para determinados tipos, los rangos de tipo, la precisión y la escala pueden ser ligeramente diferentes. Además, U-SQL y Spark tratan los valores `null` de manera distinta.

### <a name="data-types"></a>Tipos de datos

En la tabla siguiente se proporcionan los tipos equivalentes en Spark, Scala y PySpark para los tipos de U-SQL especificados.

| U-SQL | Spark |  Scala | PySpark |
| ------ | ------ | ------ | ------ |
|`byte`       ||||
|`sbyte`      |`ByteType` |`Byte` | `ByteType`|
|`int`        |`IntegerType` |`Int` | `IntegerType`|
|`uint`       ||||
|`long`       |`LongType` |`Long` | `LongType`|
|`ulong`      ||||
|`float`      |`FloatType` |`Float` | `FloatType`|
|`double`     |`DoubleType` |`Double` | `DoubleType`|
|`decimal`    |`DecimalType` |`java.math.BigDecimal` | `DecimalType`|
|`short`      |`ShortType` |`Short` | `ShortType`|
|`ushort`     ||||
|`char`   | |`Char`||
|`string` |`StringType` |`String` |`StringType` |
|`DateTime`   |`DateType`, `TimestampType` |`java.sql.Date`, `java.sql.Timestamp` | `DateType`, `TimestampType`|
|`bool`   |`BooleanType` |`Boolean` | `BooleanType`|
|`Guid`   ||||
|`byte[]` |`BinaryType` |`Array[Byte]` | `BinaryType`|
|`SQL.MAP<K,V>`   |`MapType(keyType, valueType, valueContainsNull)` |`scala.collection.Map` | `MapType(keyType, valueType, valueContainsNull=True)`|
|`SQL.ARRAY<T>`   |`ArrayType(elementType, containsNull)` |`scala.collection.Seq` | `ArrayType(elementType, containsNull=True)`|

Para más información, consulte:

- [org.apache.spark.sql.types](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.types.package)
- [Tipos Spark SQL y DataFrames](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
- [Tipos de valor Scala](https://www.scala-lang.org/api/current/scala/AnyVal.html)
- [pyspark.sql.types](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#module-pyspark.sql.types)

### <a name="treatment-of-null"></a>Tratamiento del valor NULL

En Spark, los tipos admiten valores NULL de forma predeterminada, mientras que en U-SQL, se marca explícitamente el valor escalar, que no es de objeto, como capaz de admitir un valor NULL. Aunque Spark permite definir que una columna no admita un valor NULL, no se aplicará la restricción y [podría producir un resultado incorrecto](https://medium.com/@weshoffman/apache-spark-parquet-and-troublesome-nulls-28712b06f836).

En Spark, NULL indica que el valor es desconocido. Un valor NULL de Spark es diferente de cualquier valor, incluido el mismo valor. Las comparaciones entre dos valores NULL de Spark, o entre un valor NULL y cualquier otro valor, devuelven un resultado desconocido porque el valor de cada NULL es desconocido.  

Este comportamiento es diferente de U-SQL, que sigue la semántica de C#, donde `null` es diferente de cualquier valor, pero es igual a sí mismo.  

Por tanto, una instrucción `SELECT` de SparkSQL que usa `WHERE column_name = NULL` devuelve cero filas aunque haya valores NULL en `column_name`, mientras que en U-SQL, devolvería las filas en las que el elemento `column_name` está establecido en `null`. De igual forma, una instrucción `SELECT` de Spark que usa `WHERE column_name != NULL` devuelve cero filas aunque haya valores que no sean NULL en `column_name`, mientras que en U-SQL, devolvería las filas cuyo valor no sea NULL. Por tanto, si quiere la semántica de comprobación de valores NULL de U-SQL, debe usar [isnull](https://spark.apache.org/docs/2.3.0/api/sql/index.html#isnull) e [isnotnull](https://spark.apache.org/docs/2.3.0/api/sql/index.html#isnotnull) respectivamente (o su equivalente de DSL).

## <a name="transform-u-sql-catalog-objects"></a>Transformación de objetos de catálogo de U-SQL

Una diferencia importante es que los scripts de U-SQL pueden utilizar sus objetos de catálogo, muchos de los cuales no tienen ningún equivalente de Spark directo.

Spark proporciona compatibilidad con los conceptos de metastore de Hive, principalmente las bases de datos y tablas, por lo que se pueden asignar esquemas y bases de datos de U-SQL a bases de datos de Hive, y tablas de U-SQL a tablas de Spark (vea [Traslado de los datos almacenados en tablas U-SQL](understand-spark-data-formats.md#move-data-stored-in-u-sql-tables)), pero no admite vistas, funciones con valores de tabla (TVF), procedimientos almacenados, ensamblados de U-SQL, orígenes de datos externos, etc.

Los objetos de código de U-SQL, como vistas, TVF, procedimientos almacenados y ensamblados, se pueden modelar a través de las funciones de código y las bibliotecas de Spark, y se puede hacer referencia a ellos mediante la función del lenguaje de host y los mecanismos de abstracción de procedimientos (por ejemplo, mediante la importación de módulos de Python o haciendo referencia a funciones de Scala).

Si el catálogo de U-SQL se ha usado para compartir datos y objetos de código entre proyectos y equipos, se deben usar mecanismos equivalentes para compartir (por ejemplo, Maven para objetos de código de uso compartido).

## <a name="transform-u-sql-rowset-expressions-and-sql-based-scalar-expressions"></a>Transformación de expresiones de conjunto de filas de U-SQL y expresiones escalares basadas en SQL

El lenguaje principal de U-SQL está transformando los conjuntos de filas y se basa en SQL. A continuación se muestra una lista no exhaustiva de las expresiones de conjunto de filas más comunes que se ofrecen en U-SQL:

- `SELECT`/`FROM`/`WHERE`/`GROUP BY`+Aggregates+`HAVING`/`ORDER BY`+`FETCH`
- Expresiones `INNER`/`OUTER`/`CROSS`/`SEMI` `JOIN`
- Expresiones `CROSS`/`OUTER` `APPLY`
- Expresiones `PIVOT`/`UNPIVOT`
- Constructor de conjunto de filas `VALUES`

- Expresiones de conjunto `UNION`/`OUTER UNION`/`INTERSECT`/`EXCEPT`

Además, U-SQL proporciona una serie de expresiones escalares basadas en SQL, como

- expresiones `OVER` basadas en ventanas
- diversos agregadores integrados y funciones de categoría (`SUM`, `FIRST` etc.).
- Aquí se muestran algunas de las expresiones SQL escalares más conocidas: `CASE`, `LIKE`, (`NOT`) `IN`, `AND`, `OR`, etc.

Spark ofrece expresiones equivalentes en formato DSL y SparkSQL para la mayoría de estas expresiones. Algunas de las expresiones que no se admiten de forma nativa en Spark tendrán que reescribirse mediante una combinación de expresiones de Spark nativas y los patrones semánticamente equivalentes. Por ejemplo, `OUTER UNION` tendrá que traducirse en la combinación equivalente de proyecciones y uniones.

Debido al control distinto de los valores NULL, una combinación de U-SQL siempre coincidirá con una fila si las dos columnas que se comparan contienen un valor NULL, mientras que una combinación en Spark no coincidirá con estas columnas a menos que se agreguen comprobaciones de valores NULL explícitas.

## <a name="transform-other-u-sql-concepts"></a>Transformación de otros conceptos de U-SQL

U-SQL también ofrece una serie de otros conceptos y características, como consultas federadas en bases de datos de SQL Server, parámetros, escalares y variables de expresión lambda, variables del sistema y sugerencias de `OPTION`.

### <a name="federated-queries-against-sql-server-databasesexternal-tables"></a>Consultas federadas en bases de datos o tablas externas de SQL Server

U-SQL proporciona tablas externas y de origen de datos, así como consultas directas en Azure SQL Database. Aunque Spark no ofrece las mismas abstracciones de objeto, proporciona un [conector de Spark para Azure SQL Database](../azure-sql/database/spark-connector.md) que se puede usar para consultar bases de datos SQL.

### <a name="u-sql-parameters-and-variables"></a>Parámetros y variables de U-SQL

Los parámetros y las variables de usuario tienen conceptos equivalentes en Spark y sus lenguajes de hospedaje.

En Scala, por ejemplo, se puede definir una variable con la palabra clave `var`:

```
var x = 2 * 3;
println(x)
```

Las variables del sistema de U-SQL (variables que empiezan por `@@`) se pueden dividir en dos categorías:

- Variables del sistema configurables que se pueden establecer en valores específicos para afectar al comportamiento de los scripts.
- Variables del sistema informativas que consultan información de nivel de trabajo y de sistema.

La mayoría de variables del sistema configurables no tienen ningún equivalente directo en Spark. Algunas de las variables del sistema informativas se pueden modelar pasando la información como argumentos durante la ejecución del trabajo, mientras que otras pueden tener una función equivalente en el lenguaje de hospedaje de Spark.

### <a name="u-sql-hints"></a>Sugerencias de U-SQL

U-SQL ofrece varias maneras sintácticas de proporcionar sugerencias al optimizador de consultas y al motor de ejecución:  

- Establecer una variable del sistema de U-SQL.
- Una cláusula `OPTION` asociada a la expresión de conjunto de filas para proporcionar una sugerencia de datos o de plan.
- Una sugerencia de combinación en la sintaxis de la expresión de combinación (por ejemplo, `BROADCASTLEFT`).

El optimizador de consultas basado en el costo de Spark tiene sus propias funciones para proporcionar sugerencias y ajustar el rendimiento de las consultas. Consulte la documentación correspondiente.

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los formatos de datos de Spark para desarrolladores de U-SQL](understand-spark-data-formats.md)
- [.NET para Apache Spark](https://docs.microsoft.com/dotnet/spark/what-is-apache-spark-dotnet)
- [Actualización de las soluciones de análisis de macrodatos de Azure Data Lake Storage Gen1 a Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-upgrade.md)
- [Transformación de datos mediante la actividad de Spark en Azure Data Factory](../data-factory/transform-data-using-spark.md)
- [Transformación de datos mediante la actividad de Hive de Hadoop en Azure Data Factory](../data-factory/transform-data-using-hadoop-hive.md)
- [Qué es Apache Spark en Azure HDInsight](../hdinsight/spark/apache-spark-overview.md)
