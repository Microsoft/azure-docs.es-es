---
title: Análisis del esquema con matrices y estructuras anidadas
description: Procedimiento para analizar matrices y estructuras anidadas con Apache Spark y SQL
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: ''
ms.date: 06/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: fdf3dc56575a45ad0c9e716054184ba2691133ba
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87831709"
---
# <a name="analyze-complex-data-types-in-azure-synapse-analytics"></a>Análisis de tipos de datos complejos en Azure Synapse Analytics

Este artículo es relevante para archivos y contenedores de Parquet en [Synapse Link para Azure Cosmos DB](.\synapse-link\how-to-connect-synapse-link-cosmos-db.md). Se explica cómo pueden los usuarios usar Spark o SQL para leer o transformar datos con esquemas complejos como matrices o estructuras anidadas. El ejemplo siguiente se realiza con un solo documento, pero se puede escalar con facilidad a miles de millones de documentos con Spark o SQL. En el código incluido en este artículo se usa PySpark (Python).

## <a name="use-case"></a>Caso de uso

Los tipos de datos complejos son cada vez más comunes y representan un desafío para los ingenieros de datos, ya que el análisis de esquemas anidados y matrices tiende a incluir consultas SQL complejas y lentas. Además, puede ser difícil cambiar el nombre o convertir el tipo de datos de las columnas anidadas. También surgen problemas de rendimiento al trabajar con objetos con demasiados niveles de anidamiento.

Los ingenieros de datos deben entender cómo procesar de forma eficaz los tipos de datos complejos y hacer que sean fácilmente accesibles para todos.

En el ejemplo siguiente se usa Synapse Spark para leer y transformar objetos en una estructura plana por medio de tramas de datos. Se usa Synapse SQL sin servidor para consultar estos objetos directamente y devolver los resultados como una tabla convencional.

## <a name="what-are-arrays-and-nested-structures"></a>¿Qué son las matrices y estructuras anidadas?

El objeto siguiente procede de [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview). En este objeto hay estructuras anidadas y matrices que contienen estructuras anidadas.

```json
{
    "id": "66532691-ab20-11ea-8b1d-936b3ec64e54",
    "context": {
        "data": {
            "eventTime": "2020-06-10T13:43:34.553Z",
            "samplingRate": "100.0",
            "isSynthetic": "false"
        },
        "session": {
            "isFirst": "false",
            "id": "38619c14-7a23-4687-8268-95862c5326b1"
        },
        "custom": {
            "dimensions": [
                {
                    "customerInfo": {
                        "ProfileType": "ExpertUser",
                        "RoomName": "",
                        "CustomerName": "diamond",
                        "UserName": "XXXX@yahoo.com"
                    }
                },
                {
                    "customerInfo": {
                        "ProfileType": "Novice",
                        "RoomName": "",
                        "CustomerName": "topaz",
                        "UserName": "XXXX@outlook.com"
                    }
                }
            ]
        }
    }
}
```

### <a name="schema-example-of-arrays-and-nested-structures"></a>Ejemplo de esquema de matrices y estructuras anidadas
Al imprimir el esquema de la trama de datos del objeto (denominado **df**) con el comando `df.printschema`, se ve la siguiente representación:

* El color amarillo representa la estructura anidada
* El color verde representa una matriz con dos elementos

[![Origen del esquema](./media/how-to-complex-schema/schema-origin.png)](./media/how-to-complex-schema/schema-origin.png#lightbox)

**_rid**, **_ts** y **_etag** se han agregado al sistema a medida que el documento se ingería en el almacén transaccional de Azure Cosmos DB.

En la trama de datos anterior solo hay 5 columnas y 1 fila. Después de la transformación, la trama de datos seleccionada tendrá 13 columnas y 2 filas en un formato tabular.

## <a name="flatten-nested-structures-and-explode-arrays-with-apache-spark"></a>Acoplamiento de estructuras anidadas y expansión de matrices con Apache Spark

Con Synapse Spark, resulta sencillo transformar estructuras anidadas en columnas y elementos de matriz en varias filas. Los pasos siguientes se pueden usar para la implementación.

[![Pasos de transformaciones de Spark](./media/how-to-complex-schema/spark-transform-steps.png)](./media/how-to-complex-schema/spark-transform-steps.png#lightbox)

**Paso 1**: se define una función para acoplar el esquema anidado. Esta función se puede usar sin cambios. Cree una celda en un [cuaderno de PySpark](quickstart-apache-spark-notebook.md) con la siguiente función:

```python
from pyspark.sql.functions import col

def flatten_df(nested_df):
    stack = [((), nested_df)]
    columns = []

    while len(stack) > 0:
        parents, df = stack.pop()

        flat_cols = [
            col(".".join(parents + (c[0],))).alias("_".join(parents + (c[0],)))
            for c in df.dtypes
            if c[1][:6] != "struct"
        ]

        nested_cols = [
            c[0]
            for c in df.dtypes
            if c[1][:6] == "struct"
        ]

        columns.extend(flat_cols)

        for nested_col in nested_cols:
            projected_df = df.select(nested_col + ".*")
            stack.append((parents + (nested_col,), projected_df))

    return nested_df.select(columns)
```

**Paso 2**: use la función para acoplar el esquema anidado de la trama de datos (**df**) en una nueva trama de datos `df_flat`:

```python
from pyspark.sql.types import StringType, StructField, StructType
df_flat = flatten_df(df)
display(df_flat.limit(10))
```

La función display debe devolver 10 columnas y 1 fila. La matriz y sus elementos anidados todavía están allí.

**Paso 3**: transforme la matriz `context_custom_dimensions` de la trama de datos `df_flat` en una nueva trama de datos `df_flat_explode`. En el código siguiente también se define la columna que se va a seleccionar:

```python
from pyspark.sql.functions import explode
from pyspark.sql.functions import flatten
from pyspark.sql.functions import arrays_zip
df_flat_explode = df_flat.select("_rid","_ts","id","_etag",explode(df_flat.context_custom_dimensions),"context_session_isFirst","context_session_id","context_data_eventTime","context_data_samplingRate","context_data_isSynthetic")\
.select("_rid","_ts","id","_etag","col.*","context_session_isFirst","context_session_id","context_data_eventTime","context_data_samplingRate","context_data_isSynthetic")
display(df_flat_explode.limit(10))

```

La función display debe devolver 10 columnas y 2 filas. El siguiente paso consiste en acoplar los esquemas anidados con la función definida en el paso 1.

**Paso 4**: use la función para acoplar el esquema anidado de la trama de datos `df_flat_explode` en una nueva trama de datos `df_flat_explode_flat`:
```python
df_flat_explode_flat = flatten_df(df_flat_explode)
display(df_flat_explode_flat.limit(10))
```

La función display debe mostrar 13 columnas y 2 filas.

La función `printSchema` de la trama de datos `df_flat_explode_flat` devuelve el siguiente resultado:

[![Esquema final](./media/how-to-complex-schema/schema-final.png)](./media/how-to-complex-schema/schema-final.png#lightbox)

## <a name="read-arrays-and-nested-structures-directly-with-sql-serverless"></a>Lectura de matrices y estructuras anidadas directamente con SQL sin servidor

SQL sin servidor permite la consulta y creación de vistas y tablas en esos objetos.

En primer lugar, en función de cómo se hayan almacenado los datos, los usuarios deben usar la siguiente taxonomía. Todo lo que aparece en MAYÚSCULAS es específico del caso de uso:

| BULK              | FORMAT |
| -------------------- | --- |
| 'https://ACCOUNTNAME.dfs.core.windows.net/FILESYSTEM/PATH/FINENAME.parquet ' |"Parquet" (ADLSg2)|
| N'endpoint=https://ACCOUNTNAME.documents-staging.windows-ppe.net:443/;account=ACCOUNTNAME;database=DATABASENAME;collection=COLLECTIONNAME;region=REGIONTOQUERY', SECRET='YOURSECRET' |"CosmosDB" (Synapse Link)|


> [!NOTE]
> SQL sin servidor admite Servicio vinculado de Synapse Link para Azure Cosmos DB y autenticación de paso a través de AAD. La funcionalidad está actualmente en versión preliminar controlada para Synapse Link.

Reemplace cada campo de esta manera:
* "YOUR BULK ABOVE" = cadena de conexión del origen de datos al que se conecta
* "YOUR TYPE ABOVE" = formato que usa para conectarse al origen

```sql
select *
FROM
openrowset(
    BULK 'YOUR BULK ABOVE',
    FORMAT='YOUR TYPE ABOVE'
)
with (id varchar(50),
        contextdataeventTime varchar(50) '$.context.data.eventTime',
        contextdatasamplingRate varchar(50) '$.context.data.samplingRate',
        contextdataisSynthetic varchar(50) '$.context.data.isSynthetic',
        contextsessionisFirst varchar(50) '$.context.session.isFirst',
        contextsessionid varchar(50) '$.context.session.id',
        contextcustomdimensions varchar(max) '$.context.custom.dimensions'
) as q 
cross apply openjson (contextcustomdimensions) 
with ( ProfileType varchar(50) '$.customerInfo.ProfileType',
            RoomName varchar(50) '$.customerInfo.RoomName',
            CustomerName varchar(50) '$.customerInfo.CustomerName',
            UserName varchar(50) '$.customerInfo.UserName'
    )
```

Hay dos tipos distintos de operaciones:

El primer tipo de operación se indica en la siguiente línea de código, que define la columna denominada `contextdataeventTime` que hace referencia al elemento anidado: Context.Data.eventTime 
```sql
contextdataeventTime varchar(50) '$.context.data.eventTime'
```

Esta línea definirá la columna denominada contextdataeventTime que hace referencia al elemento anidado: Context>Data>eventTime

El segundo tipo de operación usa `cross apply` para crear nuevas filas para cada elemento de la matriz y luego define cada objeto anidado similar a la primera viñeta: 
```sql
cross apply openjson (contextcustomdimensions) 
with ( ProfileType varchar(50) '$.customerInfo.ProfileType', 
```

Si la matriz tuviera 5 elementos con 4 estructuras anidadas, SQL sin servidor devolvería 5 filas y 4 columnas. SQL sin servidor puede consultar en contexto, asignar la matriz en 2 filas y, mostrar todas las estructuras anidadas en columnas.

## <a name="next-steps"></a>Pasos siguientes

* [Información para consultar Synapse Link para Azure Cosmos DB con Spark](./synapse-link/how-to-query-analytical-store-spark.md)
* [Consulta de tipos anidados de Parquet](./sql/query-parquet-nested-types.md) 
