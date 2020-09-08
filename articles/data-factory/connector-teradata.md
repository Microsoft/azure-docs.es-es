---
title: Copia de datos de Teradata Vantage mediante Azure Data Factory
description: El conector Teradata del servicio Data Factory le permite copiar datos desde Teradata Vantage a almacenes de datos compatibles con Data Factory como receptores.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: jingwang
ms.openlocfilehash: 182e04625f829304168bfdefe000bb8797646c75
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926899"
---
# <a name="copy-data-from-teradata-vantage-by-using-azure-data-factory"></a>Copia de datos de Teradata Vantage mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
>
> * [Versión 1](v1/data-factory-onprem-teradata-connector.md)
> * [Versión actual](connector-teradata.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde Teradata Vantage. Se basa en la [introducción a la actividad de copia](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Teradata es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde Teradata Vantage a cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector Teradata admite las siguientes funcionalidades:

- Teradata **versión 14.10, 15.0, 15.10, 16.0, 16.10 y 16.20**.
- La copia de datos con autenticación **básica** o de **Windows**.
- Copia en paralelo desde un origen Teradata. Consulte la sección [Copia en paralelo desde Teradata](#parallel-copy-from-teradata) para obtener más detalles.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

Si utiliza el entorno de ejecución de integración autohospedado, tenga en cuenta que proporciona un controlador de Teradata integrado a partir de la versión 3.18. No es necesario instalar manualmente ninguno. El controlador requiere "Visual C++ Redistributable 2012 Update 4" en la máquina del entorno de ejecución de integración autohospedado. Si aún no está instalada, descárguela de [aquí](https://www.microsoft.com/en-sg/download/details.aspx?id=30679).

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Teradata.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado Teradata:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type se debe establecer en **Teradata**. | Sí |
| connectionString | Especifica la información necesaria para conectarse a la instancia de Teradata. Consulte los ejemplos siguientes.<br/>También puede poner una contraseña en Azure Key Vault y extraer la configuración de `password` de la cadena de conexión. Consulte el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) para obtener información detallada. | Sí |
| username | Especifique un nombre de usuario para conectarse a Teradata. Se aplica cuando se usa autenticación de Windows. | No |
| password | Especifique la contraseña de la cuenta de usuario que se especificó para el nombre de usuario. También puede [hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). <br>Se aplica al utilizar la autenticación de Windows o hacer referencia a la contraseña en Key Vault para la autenticación básica. | No |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usará Azure Integration Runtime. |No |

Puede establecer más propiedades de conexión en la cadena de conexión, según su caso:

| Propiedad | Descripción | Valor predeterminado |
|:--- |:--- |:--- |
| UseDataEncryption | Especifica si se va a cifrar toda la comunicación con la base de datos de Teradata. Los valores permitidos son 0 o 1.<br><br/>- **0 (deshabilitado, valor predeterminado)** : cifra únicamente la información de autenticación.<br/>- **1 (habilitado)** : cifra todos los datos que se pasan entre el controlador y la base de datos. | No |
| CharacterSet | El juego de caracteres que se va a utilizar para la sesión. Por ejemplo, `CharacterSet=UTF16`.<br><br/>Este valor puede ser un juego de caracteres definido por el usuario o uno de los siguientes juegos de caracteres predefinidos: <br/>- ASCII<br/>- UTF8<br/>- UTF16<br/>- LATIN1252_0A<br/>- LATIN9_0A<br/>- LATIN1_0A<br/>- Shift-JIS (Windows, compatible con DOS, KANJISJIS_0S)<br/>- EUC (compatible con Unix, KANJIEC_0U)<br/>- IBM Mainframe (KANJIEBCDIC5035_0I)<br/>- KANJI932_1S0<br/>- BIG5 (TCHBIG5_1R0)<br/>- GB (SCHGB2312_1T0)<br/>- SCHINESE936_6R0<br/>- TCHINESE950_8R0<br/>- NetworkKorean (HANGULKSC5601_2R4)<br/>- HANGUL949_7R0<br/>- ARABIC1256_6A0<br/>- CYRILLIC1251_2A0<br/>- HEBREW1255_5A0<br/>- LATIN1250_1A0<br/>- LATIN1254_7A0<br/>- LATIN1258_8A0<br/>- THAI874_4A0 | El valor predeterminado es `ASCII`. |
| MaxRespSize |El tamaño máximo del búfer de respuesta para las solicitudes SQL, en kilobytes (KB). Por ejemplo, `MaxRespSize=‭10485760‬`.<br/><br/>En Teradata Database versión 16.00 o posterior, el valor máximo es 7361536. En el caso de las conexiones que usan versiones anteriores, el valor máximo es 1048576. | El valor predeterminado es `65536`. |

**Ejemplo de uso de la autenticación básica**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>;Uid=<username>;Pwd=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo de uso de la autenticación de Windows**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>",
            "username": "<username>",
            "password": "<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

> [!NOTE]
>
> Todavía se admite la carga siguiente. En el futuro, sin embargo, debe usar la nueva.

**Carga anterior:**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "server": "<server>",
            "authenticationType": "<Basic/Windows>",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
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

En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Teradata. Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos](concepts-datasets-linked-services.md).

Para copiar datos de Teradata, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en `TeradataTable`. | Sí |
| database | El nombre de la instancia de Teradata. | No (si se especifica "query" en el origen de la actividad) |
| table | El nombre de la tabla de la instancia de Teradata. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**:

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "TeradataTable",
        "typeProperties": {},
        "schema": [],        
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

> [!NOTE]
>
> El conjunto de datos del tipo `RelationalTable` todavía se admite. Sin embargo, se recomienda usar el nuevo.

**Carga anterior:**

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admite el origen de Teradata. Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). 

### <a name="teradata-as-source"></a>Teradata como origen

>[!TIP]
>Para cargar datos desde Teradata de manera eficaz con la creación de particiones de datos, obtenga más información en la sección [Copia en paralelo desde Teradata](#parallel-copy-from-teradata).

Para copiar datos desde Teradata, en la sección **source** de la actividad de copia se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en `TeradataSource`. | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Un ejemplo es `"SELECT * FROM MyTable"`.<br>Si habilita la carga con particiones, deberá enlazar todos los parámetros de partición integrados correspondientes en la consulta. Consulte la sección [Copia en paralelo desde Teradata](#parallel-copy-from-teradata) para obtener algunos ejemplos. | No (si se especifica la tabla en el conjunto de datos) |
| partitionOptions | Especifica las opciones de creación de particiones de datos que se usan para cargar datos desde Teradata. <br>Los valores permitidos son los siguientes: **None** (valor predeterminado), **Hash** y **DynamicRange**.<br>Cuando se habilita una opción de partición (es decir, no `None`), el grado de paralelismo para cargar simultáneamente datos desde Teradata se controla mediante la configuración [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) en la actividad de copia. | No |
| partitionSettings | Especifique el grupo de configuración para la creación de particiones de datos. <br>Se aplica cuando la opción de partición no es `None`. | No |
| partitionColumnName | Especifique el nombre de la columna de origen que usará la partición por rangos o la partición de hash para la copia en paralelo. Si no se especifica, se detecta automáticamente el índice principal de la tabla y se usa como columna de partición. <br>Se aplica si la opción de partición es `Hash` o `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfHashPartitionCondition` o `?AdfRangePartitionColumnName` en la cláusula WHERE. Consulte un ejemplo en la sección [Copia en paralelo desde Teradata](#parallel-copy-from-teradata). | No |
| partitionUpperBound | El valor máximo de la columna de partición para copiar datos. <br>Se aplica cuando la opción de partición es `DynamicRange`. Si usa la consulta para recuperar datos de origen, enlace `?AdfRangePartitionUpbound` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Teradata](#parallel-copy-from-teradata) para ver un ejemplo. | No |
| partitionLowerBound | El valor mínimo de la columna de partición para copiar datos. <br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfRangePartitionLowbound` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Teradata](#parallel-copy-from-teradata) para ver un ejemplo. | No |

> [!NOTE]
>
> `RelationalSource` todavía se admite, pero no es compatible con la nueva carga en paralelo integrada desde Teradata (opciones de partición). Sin embargo, se recomienda usar el nuevo.

**Ejemplo: copia de datos mediante una consulta básica sin partición**

```json
"activities":[
    {
        "name": "CopyFromTeradata",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Teradata input dataset name>",
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
                "type": "TeradataSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="parallel-copy-from-teradata"></a>Copia en paralelo desde Teradata

El conector de Teradata de Data Factory proporciona la creación de particiones de datos integrados para copiar datos de Teradata en paralelo. Puede encontrar las opciones de creación de particiones de datos en la pestaña **Origen** de la actividad de copia.

![Captura de pantalla de las opciones de partición](./media/connector-teradata/connector-teradata-partition-options.png)

Al habilitar la copia con particiones, Data Factory ejecuta consultas en paralelo en el origen de Teradata para cargar los datos mediante particiones. El grado en paralelo se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de la actividad de copia. Por ejemplo, si establece `parallelCopies` como cuatro, Data Factory genera y ejecuta al mismo tiempo cuatro consultas de acuerdo con la configuración y la opción de partición que ha especificado, y cada consulta recupera una porción de datos de Teradata.

Es recomendable que habilite la copia en paralelo con la creación de particiones de datos, especialmente si carga grandes cantidades de datos de Teradata. Estas son algunas configuraciones sugeridas para diferentes escenarios. Cuando se copian datos en un almacén de datos basado en archivos, se recomienda escribir en una carpeta como varios archivos (solo especifique el nombre de la carpeta), en cuyo caso el rendimiento es mejor que escribir en un único archivo.

| Escenario                                                     | Configuración sugerida                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Carga completa de una tabla grande.                                   | **Opción de partición**: hash. <br><br/>Durante la ejecución, Data Factory detecta automáticamente la columna de índice principal, le aplica un hash y copia los datos mediante particiones. |
| Cargue grandes cantidades de datos mediante una consulta personalizada.                 | **Opción de partición**: hash.<br>**Consulta**: `SELECT * FROM <TABLENAME> WHERE ?AdfHashPartitionCondition AND <your_additional_where_clause>`.<br>**Columna de partición**: especifique la columna usada para aplicar la partición hash. Si no se especifica, Data Factory detectará automáticamente la columna PK de la tabla que ha especificado en el conjunto de datos de Teradata.<br><br>Durante la ejecución, Data Factory reemplaza `?AdfHashPartitionCondition` por la lógica de partición hash y la envía a Teradata. |
| Carga de grandes cantidades de datos mediante una consulta personalizada, con una columna de enteros con valor distribuido uniformemente para la creación de particiones por rangos. | **Opciones de partición**: partición por rangos dinámica.<br>**Consulta**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Columna de partición**: especifique la columna usada para crear la partición de datos. Puede crear particiones en la columna con un tipo de datos entero.<br>**Límite de partición superior** y **límite de partición inferior**: especifique si quiere filtrar en la columna de partición para recuperar solo los datos entre el intervalo inferior y el superior.<br><br>Durante la ejecución, Data Factory reemplaza `?AdfRangePartitionColumnName`, `?AdfRangePartitionUpbound` y `?AdfRangePartitionLowbound` por el nombre real de la columna y los intervalos de valor de cada partición y se los envía a Teradata. <br>Por ejemplo, si establece la columna de partición "ID" con un límite inferior de 1 y un límite superior de 80, con la copia en paralelo establecida en 4, Data Factory recupera los datos mediante 4 particiones. Los identificadores están comprendidos entre [1, 20], [21, 40], [41, 60] y [61, 80] respectivamente. |

**Ejemplo: consulta con partición hash**

```json
"source": {
    "type": "TeradataSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfHashPartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "Hash",
    "partitionSettings": {
        "partitionColumnName": "<hash_partition_column_name>"
    }
}
```

**Ejemplo: consulta con partición por rangos dinámica**

```json
"source": {
    "type": "TeradataSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<dynamic_range_partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="data-type-mapping-for-teradata"></a>Asignación de tipos de datos para Teradata

Al copiar datos desde Teradata, se aplican las siguientes asignaciones. Para más información acerca de la forma en que la actividad de copia asigna el tipo de datos y el esquema de origen al receptor, consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md).

| Tipo de datos de Teradata | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| BigInt |Int64 |
| Blob |Byte[] |
| Byte |Byte[] |
| ByteInt |Int16 |
| Char |String |
| Clob |String |
| Date |DateTime |
| Decimal |Decimal |
| Double |Double |
| Graphic |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Entero |Int32 |
| Interval Day |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Day To Hour |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Day To Minute |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Day To Second |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Hour |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Hour To Minute |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Hour To Second |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Minute |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Minute To Second |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Month |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Second |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Year |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Interval Year To Month |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Number |Double |
| Period (Date) |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Period (Time) |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Period (Time With Time Zone) |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Period (Timestamp) |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Period (Timestamp With Time Zone) |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| SmallInt |Int16 |
| Time |TimeSpan |
| Time With Time Zone |TimeSpan |
| Timestamp |DateTime |
| Timestamp With Time Zone |DateTime |
| VarByte |Byte[] |
| VarChar |String |
| VarGraphic |No compatible. Se aplica la conversión explícita en la consulta de origen. |
| Xml |No compatible. Se aplica la conversión explícita en la consulta de origen. |


## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Pasos siguientes
Para ver la lista de almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
