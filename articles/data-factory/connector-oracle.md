---
title: Copia de datos con Oracle como origen o destino mediante Azure Data Factory
description: Aprenda a copiar datos desde almacenes de origen compatibles a una base de datos Oracle o desde Oracle a almacenes de receptor compatibles mediante Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/24/2020
ms.author: jingwang
ms.openlocfilehash: bac673f5c8c8d6a4e2b368938a0c08c893518022
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171274"
---
# <a name="copy-data-from-and-to-oracle-by-using-azure-data-factory"></a>Copia de datos con Oracle como origen o destino mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-onprem-oracle-connector.md)
> * [Versión actual](connector-oracle.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se resume el uso de la actividad de copia de Azure Data Factory para copiar datos con una base de datos Oracle como origen o destino. Se basa en la [introducción a la actividad de copia](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Oracle es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde una base de datos de Oracle en cualquier almacén de datos receptor compatible. También puede copiar datos desde cualquier almacén de datos de origen compatible a una base de datos de Oracle. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector de Oracle admite lo siguiente:

- Las siguientes versiones de una base de datos de Oracle:
    - Oracle 19c R1 (19.1) y superior
    - Oracle 18c R1 (18.1) y superior
    - Oracle 12c R1 (12.1) y superior
    - Oracle 11g R1 (11.1) y superior
    - Oracle 10g R1 (10.1) y superior
    - Oracle 9i R2 (9.2) y superior
    - Oracle 8i R3 (8.1.7) y superior
    - Oracle Database Cloud Exadata Service
- Copia en paralelo desde un origen de Oracle. Consulte la sección [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para obtener más detalles.

> [!Note]
> El servidor proxy de Oracle no se admite.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)] 

El entorno de ejecución de integración proporciona un controlador de Oracle integrado. Por tanto, no es necesario instalar manualmente un controlador para copiar datos con Oracle como origen y destino.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector Oracle.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado Oracle:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type se debe establecer en: **Oracle**. | Sí |
| connectionString | Especifica la información necesaria para conectarse a la instancia de Oracle Database. <br/>También puede poner una contraseña en Azure Key Vault y extraer la configuración de `password` de la cadena de conexión. Consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. <br><br>**Tipo de conexión admitido**: para identificar su base de datos, puede usar el **SID de Oracle** o el **nombre de servicio de Oracle**:<br>- Si usa el SID: `Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;`<br>- Si usa el nombre del servicio: `Host=<host>;Port=<port>;ServiceName=<servicename>;User Id=<username>;Password=<password>;`<br>En el caso de opciones avanzadas de conexión nativa de Oracle, puede optar por agregar una entrada en el archivo [TNSNAMES.ORA](http://www.orafaq.com/wiki/Tnsnames.ora) en el servidor de Oracle; por otro lado, en el servicio vinculado de Oracle de ADF, elija usar el tipo de conexión del nombre de servicio de Oracle y configure el correspondiente. | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

>[!TIP]
>Si recibe un error, "ORA-01025: parámetro UPI fuera del intervalo" y tiene la versión 8i de Oracle, agregue `WireProtocolMode=1` a la cadena de conexión. A continuación, inténtelo de nuevo.

Si tiene varias instancias de Oracle para el escenario de conmutación por error, puede crear un servicio vinculado de Oracle y rellenar el host principal, el puerto, el nombre de usuario, la contraseña, entre otros, y agregar una nueva sección "**Propiedades de conexión adicionales**" con el nombre de propiedad `AlternateServers` y el valor `(HostName=<secondary host>:PortNumber=<secondary port>:ServiceName=<secondary service name>)`; no olvide los corchetes y ponga atención al uso de dos puntos (`:`) como separadores. Por ejemplo, el siguiente valor de servidores alternativos define dos servidores de bases de datos alternativos para la conmutación por error de conexiones: `(HostName=AccountingOracleServer:PortNumber=1521:SID=Accounting,HostName=255.201.11.24:PortNumber=1522:ServiceName=ABackup.NA.MyCompany)`.

Puede establecer más propiedades de conexión en la cadena de conexión, según su caso:

| Propiedad | Descripción | Valores permitidos |
|:--- |:--- |:--- |
| ArraySize |El número de bytes que el conector puede capturar en un solo recorrido de ida y vuelta de red. Por ejemplo, `ArraySize=‭10485760‬`.<br/><br/>Los valores mayores aumentan la capacidad de proceso al reducir el número de veces que se capturan los datos en la red. Los valores más pequeños aumentan el tiempo de respuesta, ya que hay menos retraso en la espera para que el servidor transmita datos. | Entero de 1 a 4294967296 (4 GB). El valor predeterminado es `60000`. El valor 1 no define el número de bytes, sino que indica que se asigna espacio para exactamente una fila de datos. |

Para habilitar el cifrado en la conexión de Oracle, tiene dos opciones:

-   Para usar **el cifrado Triple-DES (3DES) y el Estándar de cifrado avanzado (AES)** , en el lado servidor de Oracle, vaya a Oracle Advanced Security (OAS) y configure las opciones de cifrado. Para más información, consulte esta [documentación de Oracle](https://docs.oracle.com/cd/E11882_01/network.112/e40393/asointro.htm#i1008759). El conector de Oracle Application Development Framework (ADF) negocia automáticamente el método de cifrado para usar el que configura en OAS al establecer la conexión con Oracle.

-   Para usar **TLS**:

    1.  Obtenga la información del certificado TLS/SSL. Obtenga la información del certificado codificado con reglas de codificación distinguida (DER) de su certificado TLS/SSL y guarde la salida (----- Begin Certificate… End Certificate -----) como archivo de texto.

        ```
        openssl x509 -inform DER -in [Full Path to the DER Certificate including the name of the DER Certificate] -text
        ```

        **Ejemplo**: extraiga la información del certificado de DERcert.cer; luego, guarde la salida en cert.txt.

        ```
        openssl x509 -inform DER -in DERcert.cer -text
        Output:
        -----BEGIN CERTIFICATE-----
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXX
        -----END CERTIFICATE-----
        ```
    
    2.  Compile `keystore` o `truststore`. El siguiente comando crea el archivo `truststore` con o sin contraseña en formato PKCS-12.

        ```
        openssl pkcs12 -in [Path to the file created in the previous step] -out [Path and name of TrustStore] -passout pass:[Keystore PWD] -nokeys -export
        ```

        **Ejemplo**: cree un archivo `truststore` PKCS12 denominado MyTrustStoreFile con una contraseña.

        ```
        openssl pkcs12 -in cert.txt -out MyTrustStoreFile -passout pass:ThePWD -nokeys -export  
        ```

    3.  Coloque el archivo `truststore` en la máquina del entorno de ejecución de integración autohospedado. Por ejemplo, coloque el archivo en C:\MyTrustStoreFile.
    4.  En Azure Data Factory, configure la cadena de conexión de Oracle con `EncryptionMethod=1` y con el valor correspondiente `TrustStore`/`TrustStorePassword`. Por ejemplo, `Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;EncryptionMethod=1;TrustStore=C:\\MyTrustStoreFile;TrustStorePassword=<trust_store_password>`.

**Ejemplo**:

```json
{
    "name": "OracleLinkedService",
    "properties": {
        "type": "Oracle",
        "typeProperties": {
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la contraseña en Azure Key Vault**

```json
{
    "name": "OracleLinkedService",
    "properties": {
        "type": "Oracle",
        "typeProperties": {
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;",
            "password": { 
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

En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Oracle. Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos](concepts-datasets-linked-services.md). 

Para copiar datos con Oracle como origen o destino, establezca la propiedad type del conjunto de datos en `OracleTable`. Se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en `OracleTable`. | Sí |
| esquema | Nombre del esquema. |No para el origen, sí para el receptor  |
| table | Nombre de la tabla o vista. |No para el origen, sí para el receptor  |
| tableName | Nombre de la tabla o vista con el esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Para la nueva carga de trabajo use `schema` y `table`. | No para el origen, sí para el receptor |

**Ejemplo**:

```json
{
    "name": "OracleDataset",
    "properties":
    {
        "type": "OracleTable",
        "schema": [],
        "typeProperties": {
            "schema": "<schema_name>",
            "table": "<table_name>"
        },
        "linkedServiceName": {
            "referenceName": "<Oracle linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen Oracle. Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). 

### <a name="oracle-as-source"></a>Oracle como origen

>[!TIP]
>Consulte más información en [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para cargar datos desde Oracle de manera eficaz con la creación de particiones de datos.

Para copiar datos desde Oracle, establezca el tipo de origen de la actividad de copia en `OracleSource`. En la sección **source** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en `OracleSource`. | Sí |
| oracleReaderQuery | Use la consulta SQL personalizada para leer los datos. Un ejemplo es `"SELECT * FROM MyTable"`.<br>Si habilita la carga con particiones, deberá enlazar todos los parámetros de partición integrados correspondientes en la consulta. Consulte la sección [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para obtener algunos ejemplos. | No |
| partitionOptions | Especifica los opciones de creación de particiones de datos que se usan para cargar datos desde Oracle. <br>Los valores permitidos son: **None** (valor predeterminado), **PhysicalPartitionsOfTable** y **DynamicRange**.<br>Cuando se habilita una opción de partición (es decir, no `None`), el grado de paralelismo para cargar simultáneamente datos de una base de datos Oracle se controla mediante la configuración [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) en la actividad de copia. | No |
| partitionSettings | Especifique el grupo de configuración para la creación de particiones de datos. <br>Se aplica si la opción de partición no es `None`. | No |
| partitionNames | Lista de particiones físicas que deben copiarse. <br>Se aplica si la opción de partición es `PhysicalPartitionsOfTable`. Si usa una consulta para recuperar datos de origen, enlace `?AdfTabularPartitionName` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para ver un ejemplo. | No |
| partitionColumnName | Especifique el nombre de la columna de origen **in integer type** que usará la creación de particiones por rangos para la copia en paralelo. Si no se especifica, se detectará automáticamente la clave principal de la tabla y se usará como columna de partición. <br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfRangePartitionColumnName` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para ver un ejemplo. | No |
| partitionUpperBound | El valor máximo de la columna de partición para copiar datos. <br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfRangePartitionUpbound` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para ver un ejemplo. | No |
| partitionLowerBound | El valor mínimo de la columna de partición para copiar datos. <br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfRangePartitionLowbound` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Oracle](#parallel-copy-from-oracle) para ver un ejemplo. | No |

**Ejemplo: copia de datos mediante una consulta básica sin partición**

```json
"activities":[
    {
        "name": "CopyFromOracle",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Oracle input dataset name>",
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
                "type": "OracleSource",
                "oracleReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="oracle-as-sink"></a>Oracle como receptor

Si va a copiar datos en Oracle, establezca el tipo de receptor de la actividad de copia en `OracleSink`. En la sección **sink** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del receptor de la actividad de copia debe establecerse en `OracleSink`. | Sí |
| writeBatchSize | Inserta datos en la tabla SQL cuando el tamaño del búfer alcanza el valor de `writeBatchSize`.<br/>Los valores permitidos son: enteros (número de filas). |No (el valor predeterminado es 10 000) |
| writeBatchTimeout | Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera.<br/>Los valores permitidos son intervalos de tiempo. Un ejemplo es 00:30:00 (30 minutos). | No |
| preCopyScript | Especifique una consulta SQL para que la actividad de copia se ejecute antes de escribir datos en Oracle en cada ejecución. Puede usar esta propiedad para limpiar los datos cargados previamente. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToOracle",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Oracle output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "OracleSink"
            }
        }
    }
]
```

## <a name="parallel-copy-from-oracle"></a>Copia en paralelo desde Oracle

El conector de Oracle de Data Factory proporciona la creación de particiones de datos integrados para copiar datos de Oracle en paralelo. Puede encontrar las opciones de creación de particiones de datos en la pestaña **Origen** de la actividad de copia.

![Captura de pantalla de las opciones de partición](./media/connector-oracle/connector-oracle-partition-options.png)

Al habilitar la copia con particiones, Data Factory ejecuta consultas en paralelo en el origen de Oracle para cargar los datos mediante particiones. El grado en paralelo se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de la actividad de copia. Por ejemplo, si establece `parallelCopies` como cuatro, Data Factory genera y ejecuta al mismo tiempo cuatro consultas de acuerdo con la configuración y la opción de partición que ha especificado, y cada consulta recupera una porción de datos de la base de datos de Oracle.

Se le sugiere que habilite la copia en paralelo con la creación de particiones de datos, especialmente si carga grandes cantidades de datos de su base de datos de Oracle. Estas son algunas configuraciones sugeridas para diferentes escenarios. Cuando se copian datos en un almacén de datos basado en archivos, se recomienda escribir en una carpeta como varios archivos (solo especifique el nombre de la carpeta), en cuyo caso el rendimiento es mejor que escribir en un único archivo.

| Escenario                                                     | Configuración sugerida                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Carga completa de una tabla grande con particiones físicas.          | **Opción de partición**: particiones físicas de la tabla. <br><br/>Durante la ejecución, Data Factory detecta automáticamente las particiones físicas y copia los datos mediante particiones. |
| Carga completa de una tabla grande, sin particiones físicas, aunque con una columna de enteros para la creación de particiones de datos. | **Opciones de partición**: partición por rangos dinámica.<br>**Columna de partición**: especifique la columna usada para crear la partición de datos. Si no se especifica, se usa la columna de clave principal. |
| Cargue una gran cantidad de datos mediante una consulta personalizada con particiones físicas. | **Opción de partición**: particiones físicas de la tabla.<br>**Consulta**: `SELECT * FROM <TABLENAME> PARTITION("?AdfTabularPartitionName") WHERE <your_additional_where_clause>`.<br>**Nombre de la partición**: especifique los nombres de las particiones desde las que se copiarán los datos. Si no se especifica, Data Factory detectará automáticamente las particiones físicas en la tabla que ha especificado en el conjunto de datos de Oracle.<br><br>Durante la ejecución, Data Factory reemplaza `?AdfTabularPartitionName` por el nombre real de la partición y se lo envía a Oracle. |
| Carga de grandes cantidades de datos mediante una consulta personalizada, sin particiones físicas, aunque cuenta con una columna de enteros para la creación de particiones de datos. | **Opciones de partición**: partición por rangos dinámica.<br>**Consulta**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Columna de partición**: especifique la columna usada para crear la partición de datos. Puede crear particiones en la columna con un tipo de datos entero.<br>**Límite de partición superior** y **límite de partición inferior**: especifique si quiere filtrar en la columna de partición para recuperar solo los datos entre el intervalo inferior y el superior.<br><br>Durante la ejecución, Data Factory reemplaza `?AdfRangePartitionColumnName`, `?AdfRangePartitionUpbound` y `?AdfRangePartitionLowbound` por el nombre real de la columna y los intervalos de valor de cada partición y se los envía a Oracle. <br>Por ejemplo, si establece la columna de partición "ID" con un límite inferior de 1 y un límite superior de 80, con la copia en paralelo establecida en 4, Data Factory recupera los datos mediante 4 particiones. Los identificadores están comprendidos entre [1, 20], [21, 40], [41, 60] y [61, 80] respectivamente. |

**Ejemplo: consulta con partición física**

```json
"source": {
    "type": "OracleSource",
    "query": "SELECT * FROM <TABLENAME> PARTITION(\"?AdfTabularPartitionName\") WHERE <your_additional_where_clause>",
    "partitionOption": "PhysicalPartitionsOfTable",
    "partitionSettings": {
        "partitionNames": [
            "<partitionA_name>",
            "<partitionB_name>"
        ]
    }
}
```

**Ejemplo: consulta con partición por rangos dinámica**

```json
"source": {
    "type": "OracleSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="data-type-mapping-for-oracle"></a>Asignación de tipos de datos para Oracle

Al copiar datos con Oracle como origen y destino, se aplican las siguientes asignaciones. Para más información acerca de la forma en que la actividad de copia asigna el tipo de datos y el esquema de origen al receptor, consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md).

| Tipo de datos de Oracle | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(solo se admite en Oracle 10g y versiones posteriores) |
| CHAR |String |
| CLOB |String |
| DATE |DateTime |
| FLOAT |Decimal, String (si la precisión > 28) |
| INTEGER |Decimal, String (si la precisión > 28) |
| LONG |String |
| LONG RAW |Byte[] |
| NCHAR |String |
| NCLOB |String |
| NUMBER |Decimal, String (si la precisión > 28) |
| NVARCHAR2 |String |
| RAW |Byte[] |
| ROWID |String |
| timestamp |DateTime |
| TIMESTAMP WITH LOCAL TIME ZONE |String |
| TIMESTAMP WITH TIME ZONE |String |
| UNSIGNED INTEGER |Number |
| VARCHAR2 |String |
| XML |String |

> [!NOTE]
> Los tipos de datos INTERVAL YEAR TO MONTH e INTERVAL DAY TO SECOND no son compatibles.

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Para ver la lista de almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
