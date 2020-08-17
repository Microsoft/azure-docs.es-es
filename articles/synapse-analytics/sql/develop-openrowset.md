---
title: Uso de OPENROWSET en SQL a petición (versión preliminar)
description: En este artículo se describe la sintaxis de OPENROWSET en SQL a petición (versión preliminar) y se explica cómo usar los argumentos.
services: synapse-analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/07/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: b7b8a0d98db1411a08afdb33fa272bb7e6d6313e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87280484"
---
# <a name="how-to-use-openrowset-with-sql-on-demand-preview"></a>Uso de OPENROWSET con SQL a petición (versión preliminar)

La función `OPENROWSET(BULK...)` permite acceder a archivos en Azure Storage. La función `OPENROWSET` lee el contenido de un origen de datos remoto (por ejemplo, un archivo) y devuelve el contenido como un conjunto de filas. En el recurso de SQL a petición (versión preliminar), para acceder al proveedor de conjuntos de filas BULK de OPENROWSET se llama a la función OPENROWSET y se especifica la opción BULK.  

Se puede hacer referencia a la función `OPENROWSET` en la cláusula `FROM` de una consulta como si fuera un nombre de tabla `OPENROWSET`. Admite operaciones masivas a través de un proveedor integrado BULK que permite que los datos se lean y se devuelvan en forma de conjunto de filas.

## <a name="data-source"></a>Origen de datos

La función OPENROWSET de Synapse SQL lee el contenido de los archivos de un origen de datos. El origen de datos es una cuenta de almacenamiento de Azure a la que se puede hacer referencia explícitamente en la función `OPENROWSET` o se puede deducir dinámicamente a partir de la dirección URL de los archivos que quiera leer.
La función `OPENROWSET` puede contener opcionalmente un parámetro `DATA_SOURCE` para especificar el origen de datos que contiene los archivos.
- `OPENROWSET` sin `DATA_SOURCE` se puede usar para leer directamente el contenido de los archivos desde la ubicación de la dirección URL especificada como opción `BULK`:

    ```sql
    SELECT *
    FROM OPENROWSET(BULK 'http://<storage account>.dfs.core.windows.net/container/folder/*.parquet',
                    FORMAT = 'PARQUET') AS file
    ```

Se trata de una manera rápida y sencilla de leer el contenido de los archivos sin necesidad de preconfiguración. Esta opción le permite usar la opción de autenticación básica para acceder al almacenamiento (acceso directo de Azure AD para inicios de sesión de Azure AD y tokens de SAS para inicios de sesión de SQL). 

- `OPENROWSET` con `DATA_SOURCE` se puede usar para tener acceso a los archivos de la cuenta de almacenamiento especificada:

    ```sql
    SELECT *
    FROM OPENROWSET(BULK '/folder/*.parquet',
                    DATA_SOURCE='storage', --> Root URL is in LOCATION of DATA SOURCE
                    FORMAT = 'PARQUET') AS file
    ```


    Esta opción le permite configurar la ubicación de la cuenta de almacenamiento en el origen de datos y especificar el método de autenticación que debe usarse para tener acceso al almacenamiento. 
    
    > [!IMPORTANT]
    > `OPENROWSET` sin `DATA_SOURCE` proporciona una forma rápida y sencilla de acceder a los archivos de almacenamiento, pero ofrece opciones de autenticación limitadas. Por ejemplo, las entidades de seguridad de Azure AD pueden acceder a los archivos solo mediante su [identidad de Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) o archivos disponibles públicamente. Si necesita opciones de autenticación más eficaces, use la opción `DATA_SOURCE` y defina las credenciales que quiera usar para tener acceso al almacenamiento.


## <a name="security"></a>Seguridad

Un usuario de base de datos debe tener permiso de `ADMINISTER BULK OPERATIONS` para utilizar la función `OPENROWSET`.

El administrador de almacenamiento también debe permitir que un usuario tenga acceso a los archivos proporcionando un token de SAS válido o habilitando la entidad de seguridad de Azure AD para tener acceso a los archivos de almacenamiento. Obtenga más información sobre el control de acceso de almacenamiento en [este artículo](develop-storage-files-storage-access-control.md).

`OPENROWSET` usa las siguientes reglas para determinar cómo autenticarse en el almacenamiento:
- En `OPENROWSET` sin `DATA_SOURCE` el mecanismo de autenticación depende del tipo de autor de llamada.
  - Cualquier usuario puede utilizar `OPENROWSET` sin `DATA_SOURCE` para leer los archivos disponibles públicamente en Azure Storage.
  - Los inicios de sesión de Azure AD solo pueden acceder a los archivos protegidos mediante su propia [identidad de Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity#supported-storage-authorization-types) si Azure Storage permite al usuario de Azure AD acceder a los archivos subyacentes (por ejemplo, si el autor de la llamada tiene permiso de `Storage Reader` en Azure Storage).
  - Los inicios de sesión de SQL también pueden usar `OPENROWSET` sin `DATA_SOURCE` para tener acceso a los archivos disponibles públicamente, a los archivos protegidos mediante el token de SAS o a la identidad administrada del área de trabajo Synapse. Debe [crear credenciales con ámbito en el servidor](develop-storage-files-storage-access-control.md#examples) para permitir el acceso a los archivos de almacenamiento. 
- En `OPENROWSET` con `DATA_SOURCE`, el mecanismo de autenticación se define en la credencial con ámbito en la base de datos asignada al origen de datos al que se hace referencia. Esta opción permite acceder al almacenamiento disponible públicamente o acceder al almacenamiento mediante el token de SAS, la identidad administrada del área de trabajo o la [identidad del autor de llamada de Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity#supported-storage-authorization-types) (si el autor de llamada es la entidad de seguridad de Azure AD). Si `DATA_SOURCE` hace referencia a una instancia de Azure Storage que no sea pública, deberá [crear una credencial cuyo ámbito sea la base de datos](develop-storage-files-storage-access-control.md#examples) y hacer referencia a ella en `DATA SOURCE` para permitir el acceso a los archivos de almacenamiento.

El autor de llamada debe tener permiso de `REFERENCES` en la credencial para usarla para autenticarse en el almacenamiento.

## <a name="syntax"></a>Sintaxis

```syntaxsql
--OPENROWSET syntax for reading Parquet files
OPENROWSET  
( { BULK 'unstructured_data_path' , [DATA_SOURCE = <data source name>, ]
    FORMAT='PARQUET' }  
)  
[WITH ( {'column_name' 'column_type' }) ]
[AS] table_alias(column_alias,...n)

--OPENROWSET syntax for reading delimited text files
OPENROWSET  
( { BULK 'unstructured_data_path' , [DATA_SOURCE = <data source name>, ] 
    FORMAT = 'CSV'
    [ <bulk_options> ] }  
)  
WITH ( {'column_name' 'column_type' [ 'column_ordinal'] })  
[AS] table_alias(column_alias,...n)
 
<bulk_options> ::=  
[ , FIELDTERMINATOR = 'char' ]    
[ , ROWTERMINATOR = 'char' ] 
[ , ESCAPE_CHAR = 'char' ] 
[ , FIRSTROW = 'first_row' ]     
[ , FIELDQUOTE = 'quote_characters' ]
[ , DATA_COMPRESSION = 'data_compression_method' ]
[ , PARSER_VERSION = 'parser_version' ]
```

## <a name="arguments"></a>Argumentos

Hay dos opciones para los archivos de entrada que contienen los datos de destino para realizar consultas. Los valores válidos son:

- "CSV": incluye cualquier archivo de texto delimitado con separadores de filas o columnas. Se puede usar cualquier carácter como separador de campo, como TSV: FIELDTERMINATOR = pestaña.

- "PARQUET": archivo binario en formato Parquet 

**"unstructured_data_path"**

El elemento unstructured_data_path que establece una ruta de acceso a los datos puede ser una ruta de acceso relativa o absoluta:
- La ruta de acceso absoluta con el formato "\<prefix>://\<storage_account_path>/\<storage_path>" permite que un usuario lea directamente los archivos.
- Ruta de acceso relativa con el formato "<storage_path>" que se debe usar con el parámetro `DATA_SOURCE` y describe el patrón de archivo en la ubicación <storage_account_path> definida en `EXTERNAL DATA SOURCE`. 

 A continuación, encontrará los valores de <storage account path> relevantes que se vincularán a su origen de datos externo concreto. 

| Origen de datos externo       | Prefijo | Ruta de acceso a la cuenta de almacenamiento                                 |
| -------------------------- | ------ | ---------------------------------------------------- |
| Azure Blob Storage         | http[s]  | \<storage_account>.blob.core.windows.net/path/file   |
| Azure Blob Storage         | wasb[s]  | \<container>@\<storage_account>.blob.core.windows.net/path/file |
| Azure Data Lake Store Gen1 | http[s]  | \<storage_account>.azuredatalakestore.net/webhdfs/v1 |
| Azure Data Lake Store Gen2 | http[s]  | \<storage_account>.dfs.core.windows.net /path/file   |
| Azure Data Lake Store Gen2 | abfs[s]  | [\<file_system>@\<account_name>.dfs.core.windows.net/path/file](../../storage/blobs/data-lake-storage-introduction-abfs-uri.md#uri-syntax)              |
||||

"\<storage_path>"

 Especifica una ruta de acceso en el almacenamiento que apunta a la carpeta o archivo que desea leer. Si la ruta de acceso apunta a un contenedor o a una carpeta, se leerán todos los archivos de ese contenedor o carpeta. No se incluirán los archivos de las subcarpetas. 

 Puede usar caracteres comodín si el destino son varios archivos o carpetas. Se permite el uso de varios caracteres comodín no consecutivos.
A continuación se muestra un ejemplo en el que se leen todos los archivos *csv* que comienzan por *population* de todas las carpetas a partir de */csv/population*:  
`https://sqlondemandstorage.blob.core.windows.net/csv/population*/population*.csv`

Si especifica que el elemento unstructured_data_path sea una carpeta, una consulta de SQL a petición recuperará los archivos de esa carpeta. 

> [!NOTE]
> A diferencia de Hadoop y PolyBase, SQL a petición no devuelve subcarpetas. También a diferencia de Hadoop y PolyBase, SQL a petición devuelve los archivos cuyo nombre comienza por un carácter de subrayado (_) o un punto (.).

En el ejemplo siguiente, si unstructured_data_path = `https://mystorageaccount.dfs.core.windows.net/webdata/`, una consulta de SQL a petición devolverá las filas de mydata.txt y _hidden.txt. No devolverá mydata2.txt y mydata3.txt porque se encuentran en una subcarpeta.

![Datos recursivos para tablas externas](./media/develop-openrowset/folder-traversal.png)

`[WITH ( {'column_name' 'column_type' [ 'column_ordinal'] }) ]`

La cláusula WITH le permite especificar las columnas que desea leer de los archivos.

- En el caso de los archivos de datos con la extensión .csv, para leer todas las columnas, especifique los nombres de columna y sus tipos de datos. Si desea un subconjunto de columnas, use números ordinales para seleccionar las columnas en los archivos de datos de origen por ordinal. Las columnas se enlazarán por la designación ordinal. 

    > [!IMPORTANT]
    > La cláusula WITH es obligatoria para los archivos .csv.
    >
    
- En el caso de los archivos de datos con formato Parquet, especifique nombres de columna que coincidan con los de los archivos de datos de origen. Las columnas se enlazarán por nombre. Si se omite la cláusula WITH, se devolverán todas las columnas de los archivos con formato Parquet.

column_name = Nombre de la columna de salida. Si se especifica, este nombre anula el nombre de la columna del archivo de origen.

column_type = Tipo de datos de la columna de salida. La conversión implícita del tipo de datos tendrá lugar aquí.

column_ordinal = Número ordinal de la columna en los archivos de origen. Este argumento se omite para los archivos con formato Parquet, ya que el enlace se realiza por nombre. En el ejemplo siguiente se devuelve una segunda columna solo desde un archivo .csv:

```sql
WITH (
    --[country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2 2
    --[year] smallint,
    --[population] bigint
)
```

**\<bulk_options>**

FIELDTERMINATOR ='field_terminator'

Especifica el terminador de campo que se va a usar. El terminador de campo predeterminado es una coma (" **,** ").

ROWTERMINATOR ="row_terminator"

Especifica el terminador de fila que se va a usar. Si no se especifica ningún terminador de fila, se usará uno de los terminadores predeterminados. Los terminadores predeterminados para PARSER_VERSION = "1.0" son \r\n, \n y \r. Los terminadores predeterminados para PARSER_VERSION = "2.0" son \r\n y \n.

ESCAPE_CHAR = "char"

Especifica el carácter del archivo que se usa como carácter de escape de sí mismo y de todos los valores de delimitador del archivo. Si el carácter de escape va seguido de un valor distinto de sí mismo o de cualquiera de los valores de delimitador, se quita al leer el valor. 

El parámetro ESCAPE_CHAR se aplicará independientemente de que FIELDQUOTE esté habilitado o no. No se utilizará como carácter de escape el carácter de comillas. El carácter de comillas se debe escapar con otro carácter de comillas. El carácter de comillas solo puede aparecer dentro del valor de la columna si el valor se encapsula entre caracteres de comillas.

FIRSTROW = "first_row" 

Especifica el número de la primera fila que se va a cargar. El valor predeterminado es 1. Indica la primera fila del archivo de datos especificado. Los números de fila vienen determinados por el recuento de terminadores de fila. FIRSTROW está en base 1.

FIELDQUOTE = "field_quote" 

Especifica un carácter que se usará como carácter de comillas en el archivo CSV. Si no se especifica, se usará el carácter de comillas ("). 

DATA_COMPRESSION = "data_compression_method"

Especifica el método de compresión. Se admite el siguiente método de compresión:

- org.apache.hadoop.io.compress.GzipCodec

PARSER_VERSION = "parser_version"

Especifica la versión del analizador que se utilizará al leer archivos. Las versiones del analizador de archivos .csv admitidas actualmente son 1.0 y 2.0:

- PARSER_VERSION = "1.0"
- PARSER_VERSION = "2.0"

La versión 1.0 del analizador de archivos .csv es el valor predeterminado y es rico en características, mientras que la versión 2.0 se basa en el rendimiento y no es compatible con todas las opciones y codificaciones. 

Detalles de la versión 2.0 del analizador de CSV:

- No se admiten todos los tipos de datos.
- El límite máximo de tamaño de fila es de 8 MB.
- Las siguientes opciones no se admiten: DATA_COMPRESSION.
- La cadena vacía entre comillas ("") se interpreta como una cadena vacía.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se devuelven solo dos columnas con los números ordinales 1 y 4 de los archivos population*.csv. Al no haber fila de encabezado en los archivos, empieza a leer desde la primera línea:

```sql
SELECT * 
FROM OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/population/population*.csv',
        FORMAT = 'CSV',
        FIRSTROW = 1
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2 1,
    [population] bigint 4
) AS [r]
```

En el ejemplo siguiente se devuelven todas las columnas de la primera fila del conjunto de datos del censo en formato Parquet sin especificar los nombres de columna ni los tipos de datos: 

```sql
SELECT 
    TOP 1 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        FORMAT='PARQUET'
    ) AS [r]
```

## <a name="next-steps"></a>Pasos siguientes

Para ver más ejemplos, consulte el [inicio rápido del almacenamiento de datos de consulta](query-data-storage.md) para aprender a usar `OPENROWSET`para leer los formatos de archivo [CSV ](query-single-csv-file.md),[PARQUET](query-parquet-files.md) y [JSON](query-json-files.md). También puede obtener información sobre cómo guardar los resultados de la consulta en Azure Storage mediante [CETAS](develop-tables-cetas.md).
