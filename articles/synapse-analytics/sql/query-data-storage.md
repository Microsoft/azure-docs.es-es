---
title: Consulta de los datos en el almacenamiento mediante SQL a petición (versión preliminar)
description: En este artículo se describe cómo consultar Azure Storage mediante el recurso de SQL a petición (versión preliminar) en Azure Synapse Analytics.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 93e6b373aa125facb3a3eddecc926438c919b335
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489748"
---
# <a name="query-storage-files-using-sql-on-demand-preview-resources-within-synapse-sql"></a>Consulta de archivos de almacenamiento mediante recursos de SQL a petición (versión preliminar) en Synapse SQL

SQL a petición (versión preliminar) permite consultar los datos de un lago de datos. Ofrece un área expuesta de consultas de T-SQL que admite consultas de datos semiestructurados y no estructurados. Para realizar consultas, se admiten los siguientes aspectos de T-SQL:

- Área expuesta [SELECT](/sql/t-sql/queries/select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) completa, que incluye la mayoría de los [operadores y funciones de SQL](overview-features.md).
- CREATE EXTERNAL TABLE AS SELECT ([CETAS](develop-tables-cetas.md)) crea una [tabla externa](develop-tables-external-tables.md) y, después, exporta en paralelo los resultados de una instrucción SELECT de Transact-SQL a Azure Storage.

Para más información sobre lo que se admite o no se admite actualmente, lea el artículo [Introducción a SQL a petición](on-demand-workspace-overview.md) o los artículos siguientes:
- [Desarrollo de acceso al almacenamiento](develop-storage-files-overview.md), donde puede aprender a usar una [tabla externa](develop-tables-external-tables.md) y la [función OPENROWSET](develop-openrowset.md) para leer datos del almacenamiento.
- [Control del acceso al almacenamiento](develop-storage-files-storage-access-control.md), donde puede aprender a habilitar Synapse SQL para acceder al almacenamiento mediante la autenticación de SAS o la identidad administrada del área de trabajo.

## <a name="overview"></a>Información general

Para tener una experiencia fluida en las consultas en contexto de los datos que se encuentran en los archivos de Azure Storage, SQL a petición usa la función [OPENROWSET](develop-openrowset.md) con funcionalidades adicionales:

- [Consulta de varios archivos o carpetas](#query-multiple-files-or-folders)
- [Formato de archivo PARQUET](#query-parquet-files)
- [Consulta de CSV y texto delimitado (terminador de campo, terminador de fila, carácter de escape)](#query-csv-files)
- [Lectura de un subconjunto de columnas elegido](#read-a-chosen-subset-of-columns)
- [Inferencia de esquemas](#schema-inference)
- [Función filename](#filename-function)
- [Función filepath](#filepath-function)
- [Uso con tipos complejos y estructuras de datos anidadas o repetidas](#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="query-parquet-files"></a>Consulta de archivos de PARQUET

Para consultar los datos de origen con formato Parquet, use FORMAT = 'PARQUET'

```syntaxsql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net//mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
WITH (C1 int, C2 varchar(20), C3 as varchar(max)) as rows
```

Para ver ejemplos de su uso, lea el artículo en el que se explica cómo [consultar archivos con formato Parquet](query-parquet-files.md).

## <a name="query-csv-files"></a>Consulta de archivo CSV

Para consultar los datos de origen con formato CSV, use FORMAT = 'CSV'. Cuando consulte archivos CSV puede especificar el esquema del archivo CSV como parte de la función `OPENROWSET`:

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.csv', FORMAT = 'CSV', PARSER_VERSION='2.0') 
WITH (C1 int, C2 varchar(20), C3 as varchar(max)) as rows
```

Hay algunas opciones adicionales que se pueden usar para ajustar las reglas de análisis al formato CSV personalizado:
- ESCAPE_CHAR = 'char' especifica el carácter del archivo que se usa como carácter de escape de sí mismo y de todos los valores de delimitador del archivo. Si el carácter de escape va seguido de un valor distinto de sí mismo o de cualquiera de los valores de delimitador, se quita al leer el valor.
El parámetro ESCAPE_CHAR se aplicará, independientemente de que FIELDQUOTE esté habilitado o no. No se utilizará como carácter de escape el carácter de comillas. El carácter de comillas se debe escapar con otro carácter de comillas. El carácter de comillas solo puede aparecer dentro del valor de la columna si el valor se encapsula entre caracteres de comillas.
- FIELDTERMINATOR='field_terminator' especifica el terminador de campo que se va a usar. El terminador de campo predeterminado es una coma (" **,** ")
- ROWTERMINATOR ='row_terminator' especifica el terminador de fila que se va a usar. El terminador de fila predeterminado es un carácter de nueva línea:  **\r\n**.

## <a name="file-schema"></a>Esquema de archivos

El lenguaje SQL de Synapse SQL permite definir el esquema del archivo como parte de la función `OPENROWSET` y leer todas las columnas o un subconjunto de ellas, o intenta determinar automáticamente los tipos de columna del archivo mediante la inferencia de esquemas.

### <a name="read-a-chosen-subset-of-columns"></a>Lectura de un subconjunto de columnas elegido

Para especificar las columnas que desea leer, puede especificar una cláusula WITH opcional en la instrucción `OPENROWSET`.

- Si hay archivos de datos con la extensión .csv, para leer todas las columnas, especifique los nombres de columna y sus tipos de datos. Si desea un subconjunto de columnas, use números ordinales para seleccionar las columnas en los archivos de datos de origen por ordinal. Las columnas se enlazarán por la designación ordinal.
- Si hay archivos de datos con formato Parquet, especifique nombres de columna que coincidan con los de los archivos de datos de origen. Las columnas se enlazarán por nombre.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
WITH (
      C1 int, 
      C2 varchar(20),
      C3 as varchar(max)
) as rows
```

Para cada columna debe especificar el nombre y el tipo de columna en la cláusula `WITH`.
Para ver algunos ejemplos, consulte el artículo en el que se proporciona información acerca de cómo [leer archivos .csv sin especificar todas las columnas](query-single-csv-file.md#return-a-subset-of-columns).

## <a name="schema-inference"></a>Inferencia de esquemas

Si omite la cláusula WITH de la instrucción `OPENROWSET`, puede indicar al servicio que detecte automáticamente (infiera) el esquema a partir de los archivos subyacentes.

> [!NOTE]
> Esto solo funciona con el formato de archivo PARQUET.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
```

Asegúrese de que se usan [tipos de datos inferidos adecuados](best-practices-sql-on-demand.md#check-inferred-data-types) para obtener un rendimiento óptimo. 

## <a name="query-multiple-files-or-folders"></a>Consulta de varios archivos o carpetas

Para ejecutar una consulta de T-SQL en un conjunto de archivos de una carpeta o un conjunto de carpetas, y tratarlos como una sola entidad o conjunto de filas, especifique una ruta de acceso a una carpeta o un patrón (mediante caracteres comodín) en un conjunto de archivos o carpetas.

Se aplican las reglas siguientes:

- Los patrones pueden aparecer en parte de una ruta de acceso de un directorio o en un nombre de archivo.
- Pueden aparecer varios patrones en el mismo nombre de archivo o paso de directorio.
- Si hay varios caracteres comodín, los archivos de todas las rutas de acceso coincidentes se incluirán en el conjunto de archivos resultante.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/myroot/*/mysubfolder/*.parquet', FORMAT = 'PARQUET' ) as rows
```

Para ver ejemplos de su uso, consulte el artículo acerca de la [consulta de carpetas y de varios archivos](query-folders-multiple-csv-files.md).

## <a name="file-metadata-functions"></a>Funciones de metadatos del archivo

### <a name="filename-function"></a>Función Filename

Esta función devuelve el nombre de archivo del que se origina la fila. 

Para consultar archivos concretos, lea la sección Filename del artículo en el que se indica cómo [consultar archivos específicos](query-specific-files.md#filename).

El tipo de datos devuelto es nvarchar (1024). Para obtener un rendimiento óptimo, convierta siempre el resultado de la función filename al tipo de datos adecuado. Si usa un tipo de datos de caracteres, asegúrese de que se usa la longitud apropiada.

### <a name="filepath-function"></a>Función Filepath

Esta función devuelve una ruta de acceso completa o una parte de una ruta de acceso:

- Cuando se llama sin parámetro, devuelve la ruta de acceso completa al archivo del que se origina una fila.
- Cuando se llama sin parámetro, devuelve la parte de la ruta de acceso que coincide con el carácter comodín en la posición especificada del parámetro. Por ejemplo, el valor 1 del parámetro devolverá la parte de la ruta de acceso que coincide con el primer carácter comodín.

Para más información, lea la sección Filepath del artículo en el que se indica cómo [consultar archivos específicos](query-specific-files.md#filepath).

El tipo de datos devuelto es nvarchar (1024). Para obtener un rendimiento óptimo, convierta siempre el resultado de la función filepath al tipo de datos adecuado. Si usa un tipo de datos de caracteres, asegúrese de que se usa la longitud apropiada.

## <a name="work-with-complex-types-and-nested-or-repeated-data-structures"></a>Uso con tipos complejos y estructuras de datos anidadas o repetidas

Para poder tener una experiencia fluida con los datos almacenados en tipos de datos anidados o repetidos como, por ejemplo, en los archivos con formato [Parquet](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#nested-types), SQL a petición ha agregado las siguientes extensiones.

#### <a name="project-nested-or-repeated-data"></a>Proyección de datos anidados o repetidos de proyecto

Para proyectar datos, ejecute una instrucción SELECT sobre el archivo con formato Parquet que contiene las columnas de los tipos de datos anidados. En la salida, los valores anidados se serializarán en JSON y se devolverán en forma de tipo de datos SQL varchar (8000).

```sql
    SELECT * FROM
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    [AS alias]
```

Para obtener información más detallada, consulte la sección relativa a la proyección de datos anidados o repetidos de proyecto del artículo en el que se explica cómo [consultar tipos anidados con formato Parquet](query-parquet-nested-types.md#project-nested-or-repeated-data).

#### <a name="access-elements-from-nested-columns"></a>Acceso a elementos de columnas anidadas

Para acceder a elementos anidados de una columna anidada, como Struct, utilice la "notación de puntos" para concatenar los nombres de campo en la ruta de acceso. Especifique la ruta de acceso como column_name en la cláusula WITH de la función `OPENROWSET`.

Este es un ejemplo de un fragmento de la sintaxis:

```syntaxsql
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    WITH ({'column_name' 'column_type',})
    [AS alias]
    'column_name' ::= '[field_name.] field_name'
```

De manera predeterminada, la función `OPENROWSET` hace coincidir el nombre de campo y la ruta de acceso de origen con los nombres de columna que se proporcionan en la cláusula WITH. Para acceder a elementos que se encuentran en distintos niveles de anidamiento del mismo archivo con formato Parquet, se debe usar la cláusula WITH.

**Valores devueltos**

- La función devuelve un valor escalar, como int, decimal y varchar, desde el elemento especificado y en la ruta de acceso especificada, para todos los tipos de archivo con formato Parquet que no están en el grupo de tipos anidados.
- Si la ruta de acceso apunta a un elemento de un tipo anidado, la función devuelve un fragmento de JSON que comienza en el elemento superior de la ruta de acceso especificada. Este fragmento es del tipo varchar (8000).
- Si la propiedad no se puede encontrar en la línea de column_name especificada, la función devuelve un error.
- Si la propiedad no se encuentra en la línea de column_path especificada, que depende del [modo de la ruta](/sql/relational-databases/json/json-path-expressions-sql-server?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest#PATHMODE), la función devuelve un error cuando está en modo strict o NULL cuando está en modo lax.

Para obtener ejemplos de consultas, revise la sección de acceso a elementos de columnas anidadas en el que se explica cómo [consultar tipos anidados con formato Parquet](query-parquet-nested-types.md#read-properties-from-nested-object-columns).

#### <a name="access-elements-from-repeated-columns"></a>Acceso a elementos de columnas repetidas

Para acceder a los elementos de una columna repetida, como un elemento de una matriz o una asignación, use la función [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) para todos los elementos escalares que necesite proyectar y proporcionar:

- Columna anidada o repetida, como primer parámetro
- Una [ruta de acceso JSON](/sql/relational-databases/json/json-path-expressions-sql-server?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) que especifica el elemento o la propiedad a la que se va a acceder, como un segundo parámetro

Para acceder a elementos no escalares de una columna repetida, use la función [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) para todos los elementos no escalares que necesite proyectar y proporcionar:

- Columna anidada o repetida, como primer parámetro
- Una [ruta de acceso JSON](/sql/relational-databases/json/json-path-expressions-sql-server?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) que especifica el elemento o la propiedad a la que se va a acceder, como segundo parámetro

Vea el fragmento de sintaxis siguiente:

```syntaxsql
    SELECT
       { JSON_VALUE (column_name, path_to_sub_element), }
       { JSON_QUERY (column_name [ , path_to_sub_element ]), )
    FROM
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    [AS alias]
```

Puede encontrar ejemplos de consultas para acceder a los elementos de las columnas repetidas en el artículo en el que se explica cómo [consultar tipos anidados con formato Parquet](query-parquet-nested-types.md#access-elements-from-repeated-columns).

## <a name="query-samples"></a>Ejemplos de consultas

Puede obtener más información sobre cómo consultar varios tipos de datos mediante las consultas de ejemplo.

### <a name="tools"></a>Herramientas

Las herramientas necesarias para emitir consultas:
    - Azure Synapse Studio (versión preliminar)
    - Azure Data Studio
    - SQL Server Management Studio

### <a name="demo-setup"></a>Configuración de demostración

El primer paso es **crear una base de datos** en la que se ejecutarán las consultas. Luego, inicializará los objetos, para lo que hay que ejecutar un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. 

Este script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan para leer datos en estos ejemplos.

> [!NOTE]
> Las bases de datos se usan solo para ver metadatos, no para datos reales.  Anote el nombre de la base de datos que use, ya que lo necesitará más adelante.

```sql
CREATE DATABASE mydbname;
```

### <a name="provided-demo-data"></a>Datos de demostración proporcionados

Los datos de demostración contienen los siguientes conjuntos de datos:

- NYC Taxi - Yellow Taxi Trip Records: parte del conjunto de datos públicos de Nueva York en formato CSV y Parquet
- Conjunto de datos de rellenado en formato CSV
- Archivos con formato Parquet de ejemplo con columnas anidadas
- Libros en formato JSON

| Ruta de acceso a la carpeta                                                  | Descripción                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /csv/                                                        | Carpeta principal para los datos en formato CSV                         |
| /csv/population/<br />/csv/population-unix/<br />/csv/population-unix-hdr/<br />/csv/population-unix-hdr-escape<br />/csv/population-unix-hdr-quoted | Carpetas con archivos de datos de población en diferentes formatos CSV. |
| /csv/taxi/                                                   | Carpeta con archivos de datos públicos de Nueva York en formato CSV              |
| /parquet/                                                    | Carpeta principal para los datos en formato Parquet                     |
| /parquet/taxi                                                | Archivos de datos públicos de Nueva York en formato Parquet, divididos por año y mes mediante el esquema de partición Hive/Hadoop. |
| /parquet/nested/                                             | Archivos con formato Parquet de ejemplo con columnas anidadas                     |
| /json/                                                       | Carpeta principal para los datos en formato JSON                        |
| /json/books/                                                 | Archivos JSON con datos de libros                                   |


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo consultar diferentes tipos de archivo y crear y usar vistas, consulte los siguientes artículos:

- [Consulta de archivos CSV](query-single-csv-file.md)
- [Consulta de archivos Parquet](query-parquet-files.md)
- [Consulta de archivos JSON](query-json-files.md)
- [Consulta de valores anidados](query-parquet-nested-types.md)
- [Consulta de carpetas y varios archivos .csv](query-folders-multiple-csv-files.md)
- [Uso de metadatos de archivo en consultas](query-specific-files.md)
- [Creación y uso de vistas](create-use-views.md)
