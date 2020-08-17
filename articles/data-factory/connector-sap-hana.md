---
title: Copia de datos desde SAP HANA
description: Obtenga información sobre cómo copiar datos desde SAP HANA en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/22/2020
ms.openlocfilehash: 92cc94170a01aceaa3e6bd058f4ae6628db04f18
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529592"
---
# <a name="copy-data-from-sap-hana-using-azure-data-factory"></a>Copia de datos de SAP HANA mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-sap-hana-connector.md)
> * [Versión actual](connector-sap-hana.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde una base de datos SAP HANA. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

>[!TIP]
>Para obtener información sobre la compatibilidad general de ADF con el escenario de integración de datos de SAP, consulte el [informe técnico sobre la integración de datos de SAP mediante Azure Data Factory](https://github.com/Azure/Azure-DataFactory/blob/master/whitepaper/SAP%20Data%20Integration%20using%20Azure%20Data%20Factory.pdf) que contiene una introducción detallada con comparaciones y una guía sobre cada conector de SAP.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector SAP HANA es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde una base de datos SAP HANA a cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector SAP HANA admite las siguientes funcionalidades:

- Copiar datos de cualquier versión de la base de datos SAP HANA.
- Copiar de datos de **modelos de información de HANA** (como las vistas de análisis y cálculo) y las **tablas de fila o columna**.
- La copia de datos con autenticación **básica** o de **Windows**.
- Copia paralela desde un origen de SAP HANA. Consulte la sección [Copia paralela desde SAP HANA](#parallel-copy-from-sap-hana) para más información.

> [!TIP]
> Para copiar datos **en** un almacén de datos SAP HANA, use un conector ODBC genérico. Consulte la sección sobre el [receptor de SAP HANA](#sap-hana-sink) con detalles. Tenga en cuenta que los servicios vinculados para el conector SAP HANA y el conector ODBC tienen tipos distintos y, por tanto, no se pueden reutilizar.

## <a name="prerequisites"></a>Prerrequisitos

Para usar este conector SAP HANA, necesitará lo siguiente:

- Configurar un entorno Integration Runtime autohospedado. Consulte el artículo sobre [Integration Runtime autohospedado](create-self-hosted-integration-runtime.md) para más información.
- Instale el controlador ODBC de SAP HANA en la máquina de Integration Runtime. Puede descargar el controlador ODBC de SAP HANA desde el [centro de descarga de software de SAP](https://support.sap.com/swdc). Busque con la palabra clave **SAP HANA CLIENT for Windows** (Cliente SAP HANA para Windows).

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector SAP HANA.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado SAP HANA:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **SapHana** | Sí |
| connectionString | Especifique la información necesaria para conectarse a SAP HANA mediante la **autenticación básica** o la **autenticación de Windows**. Consulte los ejemplos siguientes.<br>En la cadena de conexión, el servidor o el puerto es obligatorio (el puerto predeterminado es 30015), y el nombre de usuario y la contraseña son obligatorios cuando se usa la autenticación básica. Para obtener configuraciones avanzadas adicionales, consulte [Propiedades de conexión ODBC de SAP HANA](<https://help.sap.com/viewer/0eec0d68141541d1b07893a39944924e/2.0.02/en-US/7cab593774474f2f8db335710b2f5c50.html>)<br/>También puede colocar la contraseña en Azure Key Vault y extraer la configuración de contraseña de la cadena de conexión. Consulte el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) para obtener información detallada. | Sí |
| userName | Especifique el nombre de usuario cuando use la autenticación de Windows. Ejemplo: `user@domain.com` | No |
| password | Especifique la contraseña para la cuenta de usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | No |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Tal y como se mencionó en los [requisitos previos](#prerequisites), se requiere un entorno Integration Runtime autohospedado. |Sí |

**Ejemplo: use la autenticación básica**

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "connectionString": "SERVERNODE=<server>:<port (optional)>;UID=<userName>;PWD=<Password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: use la autenticación de Windows**

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "connectionString": "SERVERNODE=<server>:<port (optional)>;",
            "userName": "<username>", 
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

Si usaba el servicio SAP HANA vinculado a la siguiente carga, todavía se admite tal cual, aunque se aconseja usar la nueva en el futuro.

**Ejemplo**:

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "server": "<server>:<port (optional)>",
            "authenticationType": "Basic",
            "userName": "<username>",
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de SAP HANA.

Para copiar datos de una tabla de SAP HANA, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **SapHanaTable** | Sí |
| esquema | Nombre del esquema de la base de datos de SAP HANA. | No (si se especifica "query" en el origen de la actividad) |
| table | Nombre de la tabla de la base de datos de SAP HANA. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**:

```json
{
    "name": "SAPHANADataset",
    "properties": {
        "type": "SapHanaTable",
        "typeProperties": {
            "schema": "<schema name>",
            "table": "<table name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SAP HANA linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

Si estaba usando un conjunto de datos de tipo `RelationalTable`, todavía se admite tal cual, aunque se aconseja usar el nuevo en el futuro.

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de SAP HANA.

### <a name="sap-hana-as-source"></a>SAP HANA como origen

>[!TIP]
>Para cargar datos desde SAP HANA de manera eficaz mediante la creación de particiones de datos, puede encontrar más información en [Copia paralela desde SAP HANA](#parallel-copy-from-sap-hana).

Para copiar datos desde SAP HANA, en la sección **source** de la actividad de copia se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **SapHanaSource** | Sí |
| Query | Especifica la consulta SQL para leer datos de la instancia de SAP HANA. | Sí |
| partitionOptions | Especifica los opciones de creación de particiones de datos que se usan para ingerir datos desde SAP HANA. Más información en la sección [Copia paralela desde SAP HANA](#parallel-copy-from-sap-hana).<br>Los valores permitidos son: **None** (valor predeterminado), **PhysicalPartitionsOfTable**, **SapHanaDynamicRange**. Más información en la sección [Copia paralela desde SAP HANA](#parallel-copy-from-sap-hana). `PhysicalPartitionsOfTable` solo se puede usar cuando se copian datos de una tabla pero no de una consulta. <br>Cuando se habilita una opción de partición (es decir, el valor no es `None`), el grado de paralelismo para cargar datos de manera simultánea desde SAP HANA se controla mediante la opción [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) en la actividad de copia. | False |
| partitionSettings | Especifique el grupo de configuración para la creación de particiones de datos.<br>Se aplica cuando la opción de partición es `SapHanaDynamicRange`. | False |
| partitionColumnName | Especifique el nombre de la columna de origen que usará la partición para la copia paralela. Si no se especifica, el índice o la clave primaria de la tabla se detectan automáticamente y se usan como columna de partición.<br>Se aplica si la opción de partición es  `SapHanaDynamicRange`. Si usa una consulta para recuperar los datos de origen, enlace  `?AdfHanaDynamicRangePartitionCondition` en la cláusula WHERE. Consulte un ejemplo en la sección [Copia paralela desde SAP HANA](#parallel-copy-from-sap-hana). | Sí, cuando se usa la partición `SapHanaDynamicRange`. |
| packetSize | Especifica el tamaño del paquete de red (en kilobytes) para dividir los datos en varios bloques. Si tiene una gran cantidad de datos para copiar, si aumenta el tamaño de los paquetes aumentará a su vez la velocidad de lectura de SAP HANA en la mayoría de los casos. Se recomienda realizar pruebas de rendimiento al ajustar el tamaño de los paquetes. | No.<br>El valor predeterminado es 2048 (2 MB). |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSAPHANA",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP HANA input dataset name>",
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
                "type": "SapHanaSource",
                "query": "<SQL query for SAP HANA>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Si estaba usando un origen de copia de tipo `RelationalSource`, todavía se admite tal cual, aunque se aconseja usar el nuevo en el futuro.

## <a name="parallel-copy-from-sap-hana"></a>Copia paralela desde SAP HANA

El conector de SAP HANA de Data Factory proporciona la creación de particiones de datos integrada para copiar datos de SAP HANA en paralelo. Puede encontrar las opciones de creación de particiones de datos en la pestaña **Origen** de la actividad de copia.

![Captura de pantalla de las opciones de partición](./media/connector-sap-hana/connector-sap-hana-partition-options.png)

Al habilitar la copia con particiones, Data Factory ejecuta consultas en paralelo en el origen de SAP HANA para recuperar datos mediante particiones. El grado en paralelo se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de la actividad de copia. Por ejemplo, si establece `parallelCopies` en cuatro, Data Factory genera y ejecuta al mismo tiempo cuatro consultas de acuerdo con la configuración y la opción de partición que ha especificado, y cada consulta recupera una porción de datos de SAP HANA.

Es recomendable que habilite la copia en paralelo con la creación de particiones de datos, especialmente si ingiere grandes cantidades de datos de SAP HANA. Estas son algunas configuraciones sugeridas para diferentes escenarios. Cuando se copian datos en un almacén de datos basado en archivos, se recomienda escribirlos en una carpeta como varios archivos (solo especifique el nombre de la carpeta), en cuyo caso el rendimiento es mejor que escribirlos en un único archivo.

| Escenario                                           | Configuración sugerida                                           |
| -------------------------------------------------- | ------------------------------------------------------------ |
| Carga completa de una tabla grande.                        | **Opción de partición**: particiones físicas de la tabla. <br><br/>Durante la ejecución, Data Factory detecta automáticamente el tipo de partición física de la tabla de SAP HANA especificada y elige la estrategia de partición correspondiente:<br>- **Creación de particiones por rangos**: obtenga la columna de partición y los intervalos de partición definidos para la tabla y, luego, copia los datos por intervalo. <br>- **Creación de particiones por hash**: use la clave de partición hash como columna de partición y, luego, divida y copie los datos en función de los intervalos calculados de ADF. <br>- **Creación de particiones round-robin** o **Sin partición**: use la clave principal como columna de partición y, luego, divida y copie los datos en función de los intervalos calculados de ADF. |
| Cargue grandes cantidades de datos mediante una consulta personalizada. | **Opción de partición**: partición por rangos dinámica.<br>**Consulta**: `SELECT * FROM <TABLENAME> WHERE ?AdfHanaDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Columna de partición**: especifique la columna que se usa para aplicar la partición de intervalos dinámicos. <br><br>Durante la ejecución, Data Factory calcula en primer lugar los intervalos de valores de la columna de partición especificada y distribuye uniformemente las filas en un número de depósitos según el número de valores de columna de partición distintos y la configuración de copia paralela de ADF. Luego, reemplaza `?AdfHanaDynamicRangePartitionCondition` por el filtrado del intervalo de valores de la columna de partición de cada partición y lo envía a SAP HANA.<br><br>Si quiere usar varias columnas como columna de partición, puede concatenar los valores de cada columna como una columna de la consulta y especificarla como columna de partición en ADF, como `SELECT * FROM (SELECT *, CONCAT(<KeyColumn1>, <KeyColumn2>) AS PARTITIONCOLUMN FROM <TABLENAME>) WHERE ?AdfHanaDynamicRangePartitionCondition`. |

**Ejemplo: consulta con particiones físicas de una tabla**

```json
"source": {
    "type": "SapHanaSource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Ejemplo: consulta con partición por rangos dinámica**

```json
"source": {
    "type": "SapHanaSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfHanaDynamicRangePartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "SapHanaDynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<Partition_column_name>"
    }
}
```

## <a name="data-type-mapping-for-sap-hana"></a>Asignación de tipos de datos para SAP HANA

Al copiar datos desde SAP HANA, se usan las siguientes asignaciones de tipos de datos de SAP HANA en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de SAP HANA | Tipo de datos provisionales de Data Factory |
| ------------------ | ------------------------------ |
| ALPHANUM           | String                         |
| bigint             | Int64                          |
| BINARY             | Byte[]                         |
| BINTEXT            | String                         |
| BLOB               | Byte[]                         |
| BOOL               | Byte                           |
| CLOB               | String                         |
| DATE               | DateTime                       |
| DECIMAL            | Decimal                        |
| DOUBLE             | Double                         |
| FLOAT              | Double                         |
| INTEGER            | Int32                          |
| NCLOB              | String                         |
| NVARCHAR           | String                         |
| real               | Single                         |
| SECONDDATE         | DateTime                       |
| SHORTTEXT          | String                         |
| SMALLDECIMAL       | Decimal                        |
| SMALLINT           | Int16                          |
| STGEOMETRYTYPE     | Byte[]                         |
| STPOINTTYPE        | Byte[]                         |
| TEXT               | String                         |
| TIME               | TimeSpan                       |
| TINYINT            | Byte                           |
| VARCHAR            | String                         |
| timestamp          | DateTime                       |
| VARBINARY          | Byte[]                         |

## <a name="sap-hana-sink"></a>Receptor de SAP HANA

Actualmente, el conector de SAP HANA no se admite como receptor, aunque puede usar el conector ODBC genérico con el controlador de SAP HANA para escribir datos en SAP HANA. 

Siga los [requisitos previos](#prerequisites) de configuración de un entorno de ejecución de integración autohospedado e instale primero el controlador ODBC de SAP HANA. Cree un servicio vinculado de ODBC para conectarse al almacén de datos de SAP HANA como se muestra en el ejemplo siguiente. A continuación, cree el conjunto de datos y el receptor de la actividad de copia con el tipo ODBC, según corresponda. Obtenga más información en el artículo [conector ODBC](connector-odbc.md).

```json
{
    "name": "SAPHANAViaODBCLinkedService",
    "properties": {
        "type": "Odbc",
        "typeProperties": {
            "connectionString": "Driver={HDBODBC};servernode=<HANA server>.clouddatahub-int.net:30015",
            "authenticationType": "Basic",
            "userName": "<username>",
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

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
