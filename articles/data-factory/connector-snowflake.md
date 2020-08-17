---
title: Copia de datos desde y hacia Snowflake
description: Aprenda a copiar datos desde y hacia Snowflake mediante Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/30/2020
ms.openlocfilehash: 48248b07b64278d5c8d4f297bf83df813aa486fe
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529507"
---
# <a name="copy-data-from-and-to-snowflake-by-using-azure-data-factory"></a>Copia de datos desde y hacia Snowflake mediante Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde y hacia Snowflake. Para más información sobre Data Factory, consulte el [artículo de introducción](introduction.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector Snowflake es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con tabla de [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Para la actividad de copia, este conector Snowflake admite las siguientes funciones:

- Copie los datos de Snowflake que usan el comando [COPY into [ubicación]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html) de Snowflake para lograr el mejor rendimiento.
- Copie los datos a Snowflake que aprovechan el comando [ [tabla]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html) de Snowflake para lograr el mejor rendimiento. Es compatible con Snowflake en Azure.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que definen entidades de Data Factory específicas del conector de Snowflake.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Snowflake.

| Propiedad         | Descripción                                                  | Obligatorio |
| :--------------- | :----------------------------------------------------------- | :------- |
| type             | La propiedad type debe establecerse en **Snowflake**.              | Sí      |
| connectionString | Especifica la información necesaria para conectarse a la instancia de Snowflake. Puede optar por colocar una contraseña o una cadena de conexión completa en Azure Key Vault. Consulte los ejemplos que aparecen debajo de la tabla, así como el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) para información detallada.<br><br>Estos son algunos valores de configuración habituales:<br>- **Nombre de cuenta**: el [nombre completo de la cuenta](https://docs.snowflake.net/manuals/user-guide/connecting.html#your-snowflake-account-name) de Snowflake (incluidos los segmentos adicionales que identifican la región y la plataforma en la nube); por ejemplo, xy12345.east-us-2.azure.<br/>- **Nombre de usuario**: el nombre de inicio de sesión del usuario para la conexión.<br>- **Contraseña**: Contraseña del usuario.<br>- **Base de datos**: la base de datos predeterminada que se va a usar una vez conectado. Debe ser una base de datos existente para la que el rol especificado tenga privilegios.<br>- **Almacén**: el almacén virtual que se va a usar una vez conectado. Debe ser un almacén existente para el que el rol especificado tenga privilegios.<br>- **Rol**: El rol de control de acceso predeterminado que se va a usar en la sesión de Snowflake. El rol especificado debe ser un rol existente que ya se haya asignado al usuario especificado. El rol predeterminado es PUBLIC. | Sí      |
| connectVia       | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se utiliza para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos se encuentra en una red privada). Si no se especifica, se usará Azure Integration Runtime. | No       |

**Ejemplo**:

```json
{
    "name": "SnowflakeLinkedService",
    "properties": {
        "type": "Snowflake",
        "typeProperties": {
            "connectionString": "jdbc:snowflake://<accountname>.snowflakecomputing.com/?user=<username>&password=<password>&db=<database>&warehouse=<warehouse>&role=<myRole>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Contraseña en Azure Key Vault:**

```json
{
    "name": "SnowflakeLinkedService",
    "properties": {
        "type": "Snowflake",
        "typeProperties": {
            "connectionString": "jdbc:snowflake://<accountname>.snowflakecomputing.com/?user=<username>&db=<database>&warehouse=<warehouse>&role=<myRole>",
            "password": {
                "type": "AzureKeyVaultSecret",
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>",
                    "type": "LinkedServiceReference"
                }, 
                "secretName": "<secretName>"
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). 

Las siguientes propiedades son compatibles con el conjunto de datos de Snowflake.

| Propiedad  | Descripción                                                  | Obligatorio                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | La propiedad type del conjunto de datos se debe establecer en **SnowflakeTable**. | Sí                         |
| esquema | Nombre del esquema. |No para el origen, sí para el receptor  |
| table | Nombre de la tabla o vista. |No para el origen, sí para el receptor  |

**Ejemplo**:

```json
{
    "name": "SnowflakeDataset",
    "properties": {
        "type": "SnowflakeTable",
        "typeProperties": {
            "schema": "<Schema name for your Snowflake database>",
            "table": "<Table name for your Snowflake database>"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "linkedServiceName": {
            "referenceName": "<name of linked service>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen de Snowflake.

### <a name="snowflake-as-the-source"></a>Snowflake como origen.

El conector de Snowflake usa el comando [COPY into [location]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html) de Snowflake para lograr el mejor rendimiento.

Si el almacén de datos y el formato del receptor se admiten de forma nativa con el comando COPY de Snowflake, puede usar la actividad de copia para copiar directamente de Snowflake al receptor. Consulte [Copia directa de Snowflake](#direct-copy-from-snowflake) para obtener detalles. De lo contrario, use la [Copia almacenada provisionalmente desde Snowflake](#staged-copy-from-snowflake).

Para copiar datos desde Snowflake, en la sección **source** de la actividad de copia se admiten las propiedades siguientes.

| Propiedad                     | Descripción                                                  | Obligatorio |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | La propiedad type del origen de la actividad de copia debe establecerse en **SnowflakeSource**. | Sí      |
| Query          | Especifica la consulta SQL para leer datos de Snowflake.<br>No se admite la ejecución de procedimientos almacenados. | No       |
| exportSettings | Configuración avanzada utilizada para recuperar datos de Snowflake. Se pueden configurar los parámetros que admite el comando COPY into que Data Factory pasará al invocar la instrucción. | No       |
| ***En`exportSettings`:*** |  |  |
| type | El tipo de comando de exportación, establecido en **SnowflakeExportCopyCommand**. | Sí |
| additionalCopyOptions | Opciones de copia adicionales, proporcionadas como un diccionario de pares clave-valor. Ejemplos: MAX_FILE_SIZE, OVERWRITE. Para obtener más información consulte el documento sobre las [opciones de copia de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html#copy-options-copyoptions). | No |
| additionalFormatOptions | Opciones adicionales de formato de archivo que se proporcionan al comando COPY como un diccionario de pares clave-valor. Ejemplos: DATE_FORMAT, TIME_FORMAT, TIMESTAMP_FORMAT. Para obtener más información consulte [opciones de tipo de formato de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html#format-type-options-formattypeoptions). | No |

#### <a name="direct-copy-from-snowflake"></a>Copia directa desde Snowflake

Si el almacén de datos y el formulario del receptor cumplen los criterios descritos en esta sección, puede usar la actividad de copia para copiar directamente de Snowflake al receptor. Data Factory comprueba la configuración y produce un error en la ejecución de la actividad de copia si no se cumplen los siguientes criterios:

- El **servicio vinculado al receptor** es [**Azure Blob Storage**](connector-azure-blob-storage.md) con la autenticación de la **firma de acceso de recurso compartido**.

- El **formato de datos de receptor** es de **Parquet**, **texto delimitado** o **JSON** con estas configuraciones:

    - Para el formato **Parquet**, el códec de compresión es **None**, **Snappy**, o **Lzo**.
    - Para el formato de **texto delimitado**:
        - `rowDelimiter` es **\r\n** o cualquier carácter individual.
        - `compression` puede ser **no compression**, **gzip**, **bzip2**, o **deflate**.
        - `encodingName` se deja con el valor predeterminado o se establece en **utf-8**.
        - `quoteChar` es **double quote**, **single quote** o **empty string** (ningún carácter de comillas).
    - Para el formato **JSON**, la copia directa solo admite el caso en que el resultado de la consulta o la tabla Snowflake de origen solo tiene una columna y el tipo de datos de esta columna es **VARIANT**, **OBJECT** o **ARRAY**.
        - `compression` puede ser **no compression**, **gzip**, **bzip2** o **deflate**.
        - `encodingName` se deja con el valor predeterminado o se establece en **utf-8**.
        - `filePattern` en el receptor de la actividad de copia se deja como el valor predeterminado o se establece en **setOfObjects**.

- En el origen de la actividad de copia, no se especifica `additionalColumns`.
- No se ha especificado la asignación de columnas.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Snowflake input dataset name>",
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
                "type": "SnowflakeSource",
                "sqlReaderQuery": "SELECT * FROM MyTable",
                "exportSettings": {
                    "type": "SnowflakeExportCopyCommand",
                    "additionalCopyOptions": {
                        "MAX_FILE_SIZE": "64000000",
                        "OVERWRITE": true
                    },
                    "additionalFormatOptions": {
                        "DATE_FORMAT": "'MM/DD/YYYY'"
                    }
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="staged-copy-from-snowflake"></a>Copia almacenada provisionalmente desde Snowflake

Cuando el formato o el almacén de datos receptor no sea compatible de forma nativa con el comando COPY de Snowflake, como se mencionó en la última sección, habilite la copia preconfigurada de Azure BLOB Storage con una instancia intermedia de almacenamiento de blobs de Azure. La característica de copia almacenada provisionalmente también proporciona un mejor rendimiento. Data Factory exporta datos de Snowflake al almacenamiento provisional, copia los datos en el receptor y, por último, limpia los datos temporales del almacenamiento provisional. Consulte [Copia almacenada provisionalmente](copy-activity-performance-features.md#staged-copy) para obtener más información sobre cómo copiar datos mediante el almacenamiento provisional.

Para utilizar esta característica, cree un [servicio vinculado de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) que haga referencia a la cuenta de Azure Storage como almacenamiento provisional temporal. Luego especifique las propiedades `enableStaging` y `stagingSettings` en la actividad de copia.

> [!NOTE]
> El servicio vinculado de almacenamiento provisional de Azure Blob debe usar la autenticación de la firma de acceso compartido, como requiere el comando COPY de Snowflake. 

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Snowflake input dataset name>",
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
                "type": "SnowflakeSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

### <a name="snowflake-as-sink"></a>Snowflake como receptor

El conector de Snowflake utiliza el comando [COPY into [location]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html) de Snowflake para lograr un mejor rendimiento. Admite la escritura de datos de Snowflake en Azure.

Si el almacén de datos y el formato del origen se admiten de forma nativa con el comando COPY de Snowflake, puede usar la actividad de copia para copiar directamente del origen a Snowflake. Consulte [Copia directa de Snowflake](#direct-copy-to-snowflake) para obtener detalles. De lo contrario, use la [Copia almacenada provisionalmente de Snowflake](#staged-copy-to-snowflake).

Para copiar datos a Snowflake, se admiten las siguientes propiedades en la sección **sink** de la actividad de copia.

| Propiedad          | Descripción                                                  | Obligatorio                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| type              | La propiedad type del receptor de la actividad de copia se establece en **SnowflakeSink**. | Sí                                           |
| preCopyScript     | Especifique una consulta SQL para que la actividad de copia se ejecute antes de escribir datos en Snowflake en cada ejecución. Esta propiedad se usa para limpiar los datos cargados previamente. | No                                            |
| importSettings | Configuración avanzada utilizada para escribir datos en Snowflake. Se pueden configurar los parámetros que admite el comando COPY into que Data Factory pasará al invocar la instrucción. | No |
| ***En`importSettings`:*** |                                                              |  |
| type | El tipo de comando de importación, establecido en **SnowflakeImportCopyCommand**. | Sí |
| additionalCopyOptions | Opciones de copia adicionales, proporcionadas como un diccionario de pares clave-valor. Ejemplos: ON_ERROR, FORCE, LOAD_UNCERTAIN_FILES. Para obtener más información, consulte el documento sobre las [opciones de copia de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html#copy-options-copyoptions). | No |
| additionalFormatOptions | Opciones de formato de archivo adicionales que se proporcionan al comando COPY, que se proporciona como un diccionario de pares clave-valor. Ejemplos: DATE_FORMAT, TIME_FORMAT, TIMESTAMP_FORMAT. Para obtener más información consulte [opciones de tipo de formato de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html#format-type-options-formattypeoptions). | No |

#### <a name="direct-copy-to-snowflake"></a>Copia directa a Snowflake

Si el almacén de datos y el formulario de origen cumplen los criterios descritos en esta sección, puede usar la actividad de copia para copiar directamente desde el origen a Snowflake. Azure Data Factory comprueba la configuración y devuelve un error en la ejecución de la actividad de copia si no se cumplen los siguientes criterios:

- El **servicio vinculado al origen** es [**Azure Blob Storage**](connector-azure-blob-storage.md) con la autenticación de la **firma de acceso de recurso compartido**.

- El **formato de datos de origen** es **Parquet**, **texto delimitado** o **JSON** con estas configuraciones:

    - Para el formato **Parquet**, el códec de compresión es **None** o **Snappy**.

    - Para el formato de **texto delimitado**:
        - `rowDelimiter` es **\r\n** o cualquier carácter individual. Si el delimitador de filas no es "\r\n", `firstRowAsHeader` debe ser **false** y no se ha especificado `skipLineCount`.
        - `compression` puede ser **no compression**, **gzip**, **bzip2** o **deflate**.
        - `encodingName` se deja como valor predeterminado o se establece en "UTF-8", "UTF-16", "UTF-16BE", "UTF-32", "UTF-32BE", "BIG5", "EUC-JP", "EUC-KR", "GB18030", "ISO-2022-JP", "ISO-2022-KR", "ISO-8859-1", "ISO-8859-2", "ISO-8859-5", "ISO-8859-6", "ISO-8859-7", "ISO-8859-8", "ISO-8859-9", "WINDOWS-1250", "WINDOWS-1251", "WINDOWS-1252", "WINDOWS-1253", "WINDOWS-1254", "WINDOWS-1255".
        - `quoteChar` es **double quote**, **single quote** o **empty string** (ningún carácter de comillas).
    - Para el formato **JSON**, la copia directa solo admite el caso en que la tabla Snowflake de receptor solo tiene una columna y el tipo de datos de esta columna es **VARIANT**, **OBJECT** o **ARRAY**.
        - `compression` puede ser **no compression**, **gzip**, **bzip2** o **deflate**.
        - `encodingName` se deja con el valor predeterminado o se establece en **utf-8**.
        - No se ha especificado la asignación de columnas.

- En el origen de la actividad de copia: 

   -  `additionalColumns` no se especifica.
   - Si el origen es una carpeta, `recursive` se establece en true.
   - `prefix`, `modifiedDateTimeStart`, `modifiedDateTimeEnd` no se especifican.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Snowflake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SnowflakeSink",
                "importSettings": {
                    "type": "SnowflakeImportCopyCommand",
                    "copyOptions": {
                        "FORCE": "TRUE",
                        "ON_ERROR": "SKIP_FILE",
                    },
                    "fileFormatOptions": {
                        "DATE_FORMAT": "YYYY-MM-DD",
                    }
                }
            }
        }
    }
]
```

#### <a name="staged-copy-to-snowflake"></a>Copia almacenada provisionalmente en Snowflake

Cuando el formato o el almacén de datos receptor no sea compatible de forma nativa con el comando COPY de Snowflake, como se mencionó en la última sección, habilite la copia preconfigurada de Azure BLOB Storage con una instancia intermedia de almacenamiento de blobs de Azure. La característica de copia almacenada provisionalmente también proporciona un mejor rendimiento. Data Factory convierte automáticamente los datos para satisfacer los requisitos del formato de datos de Snowflake. A continuación, invoca el comando COPY para cargar datos en Snowflake. Por último, limpia los datos temporales del almacenamiento de blobs. Consulte [Copia almacenada provisionalmente](copy-activity-performance-features.md#staged-copy) para obtener más información sobre cómo copiar datos con el almacenamiento provisional.

Para utilizar esta característica, cree un [servicio vinculado de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) que haga referencia a la cuenta de Azure Storage como almacenamiento provisional temporal. Luego especifique las propiedades `enableStaging` y `stagingSettings` en la actividad de copia.

> [!NOTE]
> El servicio vinculado de almacenamiento provisional de Azure Blob necesita usar la autenticación de la firma de acceso compartido, como requiere el comando COPY de Snowflake.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Snowflake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SnowflakeSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```


## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para más información sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte los [formatos y almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de los almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores.
