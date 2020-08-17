---
title: Copia y transformación de los datos en Azure Cosmos DB (SQL API)
description: Aprenda a copiar datos con Azure Cosmos DB (SQL API) como origen y destino, y a transformar datos en Azure Cosmos DB (SQL API) mediante Data Factory.
services: data-factory, cosmosdb
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: multiple
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/11/2019
ms.openlocfilehash: be6dbb74883f12498c5c011e35fa955509ff627c
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042775"
---
# <a name="copy-and-transform-data-in-azure-cosmos-db-sql-api-by-using-azure-data-factory"></a>Copia y transformación de datos en Azure Cosmos DB (SQL API) mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-azure-documentdb-connector.md)
> * [Versión actual](connector-azure-cosmos-db.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia en Azure Data Factory para copiar datos con Azure Cosmos DB (SQL API) como origen y destino, y el uso de Data Flow para transformarlos en Azure Cosmos DB (SQL API). Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).



>[!NOTE]
>Este conector solo admite SQL API de Cosmos DB. Para MongoDB API, consulte [Conector de la API de Azure Cosmos DB para MongoDB](connector-azure-cosmos-db-mongodb-api.md). Actualmente, no se admiten otros tipos de API.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Cosmos DB (SQL API) es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Para la actividad de copia, este conector de Azure Cosmos DB (SQL API) admite lo siguiente:

- Copiar datos desde y hacia la [API de SQL](https://docs.microsoft.com/azure/cosmos-db/documentdb-introduction) de Azure Cosmos DB.
- Escriba en Azure Cosmos DB como **insert** o **upsert**.
- Importe y exporte documentos JSON tal cual, o copie datos desde o hacia un conjunto de datos tabular. Entre los ejemplos se incluyen una base de datos SQL y un archivo CSV. Para copiar documentos tal cual desde archivos JSON o hacia estos, o desde otra colección de Azure Cosmos DB o hacia esta, consulte [Importación y exportación de documentos JSON](#import-and-export-json-documents).

Data Factory se integra con la [biblioteca BulkExecutor en Azure Cosmos DB](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started) para proporcionar el mejor rendimiento de escritura en Azure Cosmos DB.

> [!TIP]
> El [vídeo de migración de datos](https://youtu.be/5-SRNiC_qOU) le guía por los pasos para copiar datos desde Azure Blob Storage hasta Azure Cosmos DB. En el vídeo también se describen consideraciones sobre la optimización del rendimiento para la ingesta de datos a Azure Cosmos DB en general.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que puede usar para definir entidades de Data Factory específicas de Azure Cosmos DB (API de SQL).

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Azure Cosmos DB (API de SQL):

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **CosmosDb**. | Sí |
| connectionString |Especifique la información necesaria para conectarse a la base de datos de Azure Cosmos DB.<br />**Nota**: Debe especificar la información de la base de datos en la cadena de conexión como se muestra en los ejemplos siguientes. <br/> También puede colocar la clave de cuenta en Azure Key Vault y extraer la configuración `accountKey` de la cadena de conexión. Consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. |Sí |
| connectVia | Instancia de [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos se encuentra en una red privada). Si no se especifica esta propiedad, se usa el valor predeterminado de Azure Integration Runtime. |No |

**Ejemplo**

```json
{
    "name": "CosmosDbSQLAPILinkedService",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la clave de cuenta en Azure Key Vault**

```json
{
    "name": "CosmosDbSQLAPILinkedService",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "connectionString": "AccountEndpoint=<EndpointUrl>;Database=<Database>",
            "accountKey": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Para ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos y servicios vinculados](concepts-datasets-linked-services.md).

Se admiten las siguientes propiedades con el conjunto de datos de Azure Cosmos DB (SQL API): 

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del conjunto de datos debe establecerse en **CosmosDbSqlApiCollection**. |Sí |
| collectionName |Nombre de la colección de documentos de Azure Cosmos DB. |Sí |

Si usa un conjunto de datos de tipo "DocumentDbCollection", se sigue admitiendo tal cual para la compatibilidad con versiones anteriores de la actividad de copia y búsqueda y no se admite para Data Flow. Se sugiere usar el nuevo modelo de aquí en adelante.

**Ejemplo**

```json
{
    "name": "CosmosDbSQLAPIDataset",
    "properties": {
        "type": "CosmosDbSqlApiCollection",
        "linkedServiceName":{
            "referenceName": "<Azure Cosmos DB linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [],
        "typeProperties": {
            "collectionName": "<collection name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admiten el origen y el receptor de Azure Cosmos DB (API de SQL). Para ver una lista completa de las secciones y propiedades que hay disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md).

### <a name="azure-cosmos-db-sql-api-as-source"></a>Azure Cosmos DB (API de SQL) como origen

Para copiar datos desde Azure Cosmos DB (API de SQL), establezca el tipo de **origen** de la actividad de copia en **DocumentDbCollectionSource**. 

La sección **source** de la actividad de copia admite las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type1** del origen de la actividad de copia debe establecerse en **CosmosDbSqlApiSource**. |Sí |
| Query |Especifique la consulta de Azure Cosmos DB para leer datos.<br/><br/>Ejemplo:<br /> `SELECT c.BusinessEntityID, c.Name.First AS FirstName, c.Name.Middle AS MiddleName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |No <br/><br/>Si no se especifica, se ejecuta la instrucción SQL: `select <columns defined in structure> from mycollection` |
| preferredRegions | Lista preferida de regiones a las que se conectará cuando recupere los datos de Cosmos DB. | No |
| pageSize | Número de documentos por página del resultado de la consulta. El valor predeterminado es "-1", que significa el uso del tamaño de página dinámica del servicio hasta 1000. | No |

Si utiliza el origen de tipo "DocumentDbCollectionSource", todavía se admite tal cual para la compatibilidad con versiones anteriores. Se recomienda utilizar el nuevo modelo en el futuro, que proporciona funcionalidades más enriquecidas para copiar datos de Cosmos DB.

**Ejemplo**

```json
"activities":[
    {
        "name": "CopyFromCosmosDBSQLAPI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Cosmos DB SQL API input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "CosmosDbSqlApiSource",
                "query": "SELECT c.BusinessEntityID, c.Name.First AS FirstName, c.Name.Middle AS MiddleName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"",
                "preferredRegions": [
                    "East US"
                ]
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Al copiar datos de Cosmos DB, a menos que desee [exportar documentos JSON tal cual](#import-and-export-json-documents), el procedimiento recomendado es especificar la asignación en la actividad de copia. Data Factory respeta la asignación especificada en la actividad: si una fila no contiene un valor para una columna, se proporciona un valor null para el valor de la columna. Si no especifica una asignación, Data Factory deduce el esquema mediante la primera fila de los datos. Si la primera fila no contiene el esquema completo, algunas columnas se pueden perder en el resultado de la operación de la actividad.

### <a name="azure-cosmos-db-sql-api-as-sink"></a>Azure Cosmos DB (API de SQL) como receptor

Para copiar datos en Azure Cosmos DB (API de SQL), establezca el tipo de **receptor** de la actividad de copia en **DocumentDbCollectionSink**. 

La sección **sink** de la actividad de copia admite las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del receptor de la actividad de copia tiene que establecerse en **CosmosDbSqlApiSink**. |Sí |
| writeBehavior |Describe cómo escribir datos en Azure Cosmos DB. Valores permitidos: **insert** y **upsert**.<br/><br/>El comportamiento de **upsert** consiste en reemplazar el documento si ya existe un documento con el mismo identificador; en caso contrario, inserta el documento.<br /><br />**Nota**: Data Factory genera automáticamente un identificador para un documento si no se especifica un identificador en el documento original o mediante la asignación de columnas. Esto significa que debe asegurarse de que, para que **upsert** funcione según lo esperado, el documento tenga un identificador. |No<br />(el valor predeterminado es **insert**) |
| writeBatchSize | Data Factory usa la [biblioteca BulkExecutor en Azure Cosmos DB](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started) para escribir datos en Azure Cosmos DB. La propiedad **writeBatchSize** controla el tamaño de los documentos que ADF proporciona a la biblioteca. Puede aumentar el valor de **writeBatchSize** para mejorar el rendimiento y reducir el valor si el documento tiene un tamaño grande: a continuación encontrará sugerencias. |No<br />(el valor predeterminado es **10 000**) |
| disableMetricsCollection | Data Factory recopila métricas, como las RU de Cosmos DB, para la optimización del rendimiento de copia y la obtención de recomendaciones. Si le preocupa este comportamiento, especifique `true` para desactivarlo. | No (el valor predeterminado es `false`) |

>[!TIP]
>Para importar documentos JSON tal cual, consulte la sección [Importar o exportar documentos JSON](#import-and-export-json-documents); para copiar de datos con formato tabular, consulte [Migración de la base de datos relacional a Cosmos DB](#migrate-from-relational-database-to-cosmos-db).

>[!TIP]
>Cosmos DB limita el tamaño de las solicitudes únicas a 2 MB. La fórmula es Tamaño de la solicitud = tamaño de documento único x tamaño de lote de escritura. Si aparece un error con el texto **"El tamaño de solicitud es demasiado grande."** , **reduzca el valor de `writeBatchSize`** en la configuración del receptor de copia.

Si utiliza el origen de tipo "DocumentDbCollectionSink", todavía se admite tal cual para la compatibilidad con versiones anteriores. Se recomienda utilizar el nuevo modelo en el futuro, que proporciona funcionalidades más enriquecidas para copiar datos de Cosmos DB.

**Ejemplo**

```json
"activities":[
    {
        "name": "CopyToCosmosDBSQLAPI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Document DB output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "CosmosDbSqlApiSink",
                "writeBehavior": "upsert"
            }
        }
    }
]
```

### <a name="schema-mapping"></a>Asignación de esquemas

Para copiar datos de Azure Cosmos DB en un receptor tabular o inverso, consulte [Asignación de esquemas](copy-activity-schema-and-type-mapping.md#schema-mapping).

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Al transformar datos en el flujo de datos de asignación, puede leer y escribir en colecciones de Cosmos DB. Para más información, consulte la [transformación de origen](data-flow-source.md) y la [transformación de receptor](data-flow-sink.md) en los flujos de datos de asignación.

### <a name="source-transformation"></a>Transformación de origen

La configuración específica de Azure Comos DB está disponible en la pestaña **Source Options** (Opciones de origen) de la transformación de origen. 

**Include system columns** (Incluir las columnas del sistema): si su valor es true, ```id```, ```_ts``` y otras columnas del sistema se incluirán en los metadatos del flujo de datos de Cosmos DB. Al actualizar las colecciones, es importante incluirla para poder tomar el Id. de fila existente.

**Page size** (Tamaño de página): Número de documentos por página del resultado de la consulta. El valor predeterminado es "-1", que utiliza la página dinámica del servicio hasta 1000.

**Throughput** (Capacidad de proceso): Establezca un valor opcional para el número de RU que desea aplicar a la colección de CosmosDB para cada ejecución de este flujo de datos durante la operación de lectura. El mínimo es de 400.

**Preferred regions** (Regiones preferidas): elija las regiones de lectura preferidas para este proceso.

#### <a name="json-settings"></a>Configuración de JSON

**Documento único:** seleccione esta opción si ADF debe tratar todo el archivo como un solo documento JSON.

**Nombres de columnas sin comillas:** seleccione esta opción si los nombres de columna del archivo JSON no están entre comillas.

**Tiene comentarios:** use esta selección si los documentos JSON tienen comentarios en los datos.

**Con comillas simples:** debe seleccionar esta opción si las columnas y los valores del documento se incluyen entre comillas simples.

**Barra diagonal inversa con escape:** si usa barras diagonales inversas para el escape de caracteres en el archivo JSON, elija esta opción.

### <a name="sink-transformation"></a>Transformación de receptor

La configuración específica de Azure Cosmos DB está disponible en la pestaña **Settings** (Configuración) de la transformación de receptor.

**Update method** (Método de actualización): determina qué operaciones se permiten en el destino de la base de datos. El valor predeterminado es permitir solamente las inserciones. Para realizar las operaciones update, upsert o delete rows, se requiere una transformación de alteración de filas para etiquetar esas acciones. En el caso de las operaciones update, upsert y delete, se debe establecer una o varias columnas de clave para determinar la fila que se va a modificar.

**Collection action** (Acción de colección): determina si se debe volver a crear la colección de destino antes de escribir.
* None (Ninguno): no se realizará ninguna acción en la colección.
* Recreate (Volver a crear): se quitará la colección y se volverá a crear.

**Tamaño del lote**: controla el número de filas que se escriben en cada cubo. Los tamaños de lote más grandes mejoran la compresión y la optimización de memoria, pero podrían obtener excepciones de memoria al almacenar datos en caché.

**Partition Key** (Clave de partición): Escriba una cadena que represente la clave de partición de la colección. Ejemplo: ```/movies/title```

**Throughput** (Capacidad de proceso): Establezca un valor opcional para el número de RU que desea aplicar a la colección de CosmosDB para cada ejecución de este flujo de datos. El mínimo es de 400.

**Write throughput budget** (Presupuesto de capacidad de proceso de escritura): un entero que representa el número de RU que desea asignar al trabajo de Spark de ingesta en bloque. Este número no se considera en la capacidad de proceso total asignada a la colección.

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="import-and-export-json-documents"></a>Importación y exportación de documentos JSON

Puede utilizar este conector de Azure Cosmos DB (API de SQL) para hacer lo siguiente con facilidad:

* Copiar documentos entre dos colecciones de Azure Cosmos DB tal cual.
* Importar documentos JSON desde varios orígenes a Azure Cosmos DB, incluido Azure Blob Storage, Azure Data Lake Store y otros almacenes basados en archivos compatibles con Azure Data Factory.
* Exportar documentos JSON de una colección de Azure Cosmos DB a varios almacenes basados en archivos.

Para lograr una copia independiente del esquema:

* Cuando use la herramienta Copiar datos, seleccione la opción **Export as-is to JSON files or Cosmos DB collection** (Exportar tal cual a archivos JSON o colección de Cosmos DB).
* Cuando use la creación de actividades, elija el formato JSON con el almacén de archivos correspondiente para el origen o el receptor.

## <a name="migrate-from-relational-database-to-cosmos-db"></a>Migración de la base de datos relacional a Cosmos DB

Al migrar desde una base de datos relacional, por ejemplo de SQL Server a Azure Cosmos DB, la actividad de copia puede asignar fácilmente datos tabulares del origen a documentos JSON en Cosmos DB. En algunos casos puede que desee rediseñar el modelo de datos para optimizarlo para los casos de uso de NoSQL según lo que se indica en [Modelado de datos en Azure Cosmos DB](../cosmos-db/modeling-data.md), por ejemplo, para desnormalizar los datos mediante la inserción de todos los subelementos relacionados en un documento JSON. En este caso, consulte [este artículo](../cosmos-db/migrate-relational-to-cosmos-db-sql-api.md), que ofrece un tutorial sobre cómo lograrlo mediante la actividad de copia de Azure Data Factory.

## <a name="next-steps"></a>Pasos siguientes

Para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes y receptores en Azure Data Factory, consulte los [almacenes de datos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
