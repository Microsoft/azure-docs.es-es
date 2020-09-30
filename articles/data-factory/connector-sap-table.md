---
title: Copia de datos de una tabla de SAP
description: Obtenga información sobre cómo copiar datos desde una tabla de SAP en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/01/2020
ms.openlocfilehash: 608694c07894c8bdff8b1101d607e07ea4383764
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279861"
---
# <a name="copy-data-from-an-sap-table-by-using-azure-data-factory"></a>Copia de datos de una tabla de SAP mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos con desde una tabla de SAP. Para obtener más información, consulte la [información general sobre la actividad de copia](copy-activity-overview.md).

>[!TIP]
>Para obtener información sobre la compatibilidad general de ADF con el escenario de integración de datos de SAP, consulte el [informe técnico sobre la integración de datos de SAP mediante Azure Data Factory](https://github.com/Azure/Azure-DataFactory/blob/master/whitepaper/SAP%20Data%20Integration%20using%20Azure%20Data%20Factory.pdf) que contiene una introducción detallada con comparaciones y una guía sobre cada conector de SAP.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector SAP Table es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de una tabla de SAP en cualquier almacén de datos de receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector para tablas de SAP admite las siguientes funcionalidades:

- Copiar datos de una tabla de SAP en:

  - Componente central de SAP ERP (SAP ECC) versión 7.01 o posterior (en una pila reciente de paquetes de soporte técnico de SAP lanzados después de 2015).
  - SAP Business Warehouse (SAP BW), versión 7.01 o posterior, (en una pila de paquetes de soporte técnico de SAP reciente lanzada después de 2015).
  - SAP S/4HANA.
  - Otros productos en SAP Business Suite, versión 7.01 o posterior, (en una pila de paquetes de soporte técnico de SAP reciente lanzada después de 2015).

- Copiar datos de una tabla transparente de SAP, una tabla agrupada, una tabla en clúster y una vista.
- Copiar datos mediante la autenticación básica o las comunicaciones de red segura (SNC), si es que tiene SNC configuradas.
- Conexión a un servidor de aplicaciones SAP o a un servidor de mensajes SAP.
- Recuperación de datos a través de RFC personalizado o predeterminado.

La versión 7.01 o posterior hace referencia a la versión de SAP NetWeaver en lugar de a la versión de SAP ECC. Por ejemplo, SAP ECC 6.0 EHP 7 en general tiene una versión de NetWeaver >= 7.4. En caso de que no esté seguro de su entorno, estos son los pasos para confirmar la versión del sistema de SAP:
1.  Use la GUI de SAP para conectarse al sistema de SAP. 
2.  Vaya a **System** -> **Status**. 
3.  Compruebe la versión de SAP_BASIS, asegúrese de que sea mayor o igual que 701.  
      ![Comprobar SAP_BASIS](./media/connector-sap-table/sap-basis.png)

## <a name="prerequisites"></a>Prerrequisitos

Para usar este conector de tabla de SAP, necesitará lo siguiente:

- Configure un entorno de ejecución de integración autohospedado (versión 3.17 o posterior). Para obtener más información, consulte [Creación y configuración de un entorno de ejecución de integración autohospedado](create-self-hosted-integration-runtime.md).

- Descargue el [conector de SAP de 64 bits para Microsoft.NET 3.0](https://support.sap.com/en/product/connectors/msnet.html) del sitio web de SAP e instálelo en la máquina del entorno de ejecución de integración autohospedado. Al instalarlo, asegúrese de seleccionar la opción **Install Assemblies to GAC** (Instalar ensamblados en GAC) en la ventana **Optional setup steps** (Pasos de configuración opcionales).

  ![Instale el conector de SAP para .NET](./media/connector-sap-business-warehouse-open-hub/install-sap-dotnet-connector.png)

- El usuario de SAP que se está usando en el conector de tabla SAP de Data Factory debe tener los siguientes permisos:

  - Autorización para usar los destinos de llamada de función remota (RFC).
  - Permisos para la actividad “Execute” del objeto de autorización S_SDSAUTH.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de tabla de SAP.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado del centro de SAP BW abierto:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| `type` | La propiedad `type` debe establecerse en `SapTable`. | Sí |
| `server` | El nombre del servidor en el que se encuentra la instancia de SAP.<br/>Úselo para conectarse a un servidor de aplicaciones de SAP. | No |
| `systemNumber` | Número del sistema de SAP.<br/>Úselo para conectarse a un servidor de aplicaciones de SAP.<br/>El valor permitido es: un número decimal de dos dígitos que se representa en forma de cadena. | No |
| `messageServer` | El nombre de host del servidor de mensajes de SAP.<br/>Úselo para conectarse a un servidor de mensajes de SAP. | No |
| `messageServerService` | El nombre del servicio o el número de puerto del servidor de mensajes.<br/>Úselo para conectarse a un servidor de mensajes de SAP. | No |
| `systemId` | El id. del sistema SAP en el que se encuentra la tabla.<br/>Úselo para conectarse a un servidor de mensajes de SAP. | No |
| `logonGroup` | El grupo de inicio de sesión para el sistema SAP.<br/>Úselo para conectarse a un servidor de mensajes de SAP. | No |
| `clientId` | El id. del cliente en el sistema SAP.<br/>El valor permitido es: un número decimal de tres dígitos que se representa en forma de cadena. | Sí |
| `language` | El idioma que usa el sistema SAP.<br/>El valor predeterminado es `EN`.| No |
| `userName` | El nombre del usuario que tiene acceso al servidor SAP. | Sí |
| `password` | Contraseña del usuario. Marque este campo con el tipo `SecureString` para almacenarlo de forma segura en Data Factory o para [hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| `sncMode` | El indicador de activación de SNC para acceder al servidor SAP en que se encuentra la tabla.<br/>Aplicable si quiere usar SNC para conectarse al servidor SAP.<br/>Los valores permitidos son `0` (OFF es el valor predeterminado) o `1` (ON). | No |
| `sncMyName` | El nombre SNC del iniciador para acceder al servidor de SAP en que se encuentra la tabla.<br/>Se aplica cuando `sncMode` está activado. | No |
| `sncPartnerName` | El nombre SNC del asociado de comunicación para obtener acceso al servidor SAP en el que se encuentra la tabla.<br/>Se aplica cuando `sncMode` está activado. | No |
| `sncLibraryPath` | La biblioteca del producto de seguridad externo para acceder al servidor SAP donde se encuentra la tabla.<br/>Se aplica cuando `sncMode` está activado. | No |
| `sncQop` | El nivel de calidad de protección de SNC a aplicar.<br/>Se aplica `sncMode` cuando está activado. <br/>Los valores permitidos son `1` (Autenticación), `2` (Integridad), `3` (Privacidad), `8` (valor predeterminado), `9` (Máximo). | No |
| `connectVia` | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Tal y como se mencionó en los [requisitos previos](#prerequisites), se requiere un entorno de ejecución de integración autohospedado. |Sí |

**Ejemplo 1: conectarse a un servidor de aplicaciones de SAP**

```json
{
    "name": "SapTableLinkedService",
    "properties": {
        "type": "SapTable",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client ID>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="example-2-connect-to-an-sap-message-server"></a>Ejemplo 2: conectarse a un servidor de mensajes de SAP

```json
{
    "name": "SapTableLinkedService",
    "properties": {
        "type": "SapTable",
        "typeProperties": {
            "messageServer": "<message server name>",
            "messageServerService": "<service name or port>",
            "systemId": "<system ID>",
            "logonGroup": "<logon group>",
            "clientId": "<client ID>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="example-3-connect-by-using-snc"></a>Ejemplo 3: conexión mediante SNC

```json
{
    "name": "SapTableLinkedService",
    "properties": {
        "type": "SapTable",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client ID>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            },
            "sncMode": 1,
            "sncMyName": "<SNC myname>",
            "sncPartnerName": "<SNC partner name>",
            "sncLibraryPath": "<SNC library path>",
            "sncQop": "8"
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Para obtener una lista completa de las secciones y propiedades para definir conjuntos de datos, consulte [Conjuntos de datos](concepts-datasets-linked-services.md). En la sección siguiente se proporciona una lista de las propiedades que admite el conjunto de datos de la tabla de SAP.

Para copiar datos desde y hacia el servicio vinculado del centro de SAP BW abierto, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| `type` | La propiedad `type` debe establecerse en `SapTableResource`. | Sí |
| `tableName` | El nombre de la tabla de SAP desde la que se copian los datos. | Sí |

### <a name="example"></a>Ejemplo

```json
{
    "name": "SAPTableDataset",
    "properties": {
        "type": "SapTableResource",
        "typeProperties": {
            "tableName": "<SAP table name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SAP table linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Para obtener una lista completa de las secciones y propiedades para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). En la sección siguiente se proporciona una lista de las propiedades que admite el origen de tabla de SAP.

### <a name="sap-table-as-source"></a>Tabla de SAP como origen

Para copiar datos de una tabla de SAP, se admiten las siguientes propiedades:

| Propiedad                         | Descripción                                                  | Obligatorio |
| :------------------------------- | :----------------------------------------------------------- | :------- |
| `type`                             | La propiedad `type` debe establecerse en `SapTableSource`.         | Sí      |
| `rowCount`                         | El número de filas que se van a recuperar.                              | No       |
| `rfcTableFields`                 | Los campos (columnas) que se copiarán de la tabla de SAP. Por ejemplo, `column0, column1`. | No       |
| `rfcTableOptions`                | Las opciones para filtrar las filas de la tabla de SAP. Por ejemplo, `COLUMN0 EQ 'SOMEVALUE'`. Consulte también la tabla del operador de consultas de SAP que encontrará en este artículo. | No       |
| `customRfcReadTableFunctionModule` | Un módulo de función RFC personalizado que puede usarse para leer datos de la tabla de SAP.<br>Puede usar el módulo de función RFC personalizado para definir cómo se recuperan los datos del sistema SAP y cómo se devuelven a Data Factory. El módulo de función personalizado debe tener una interfaz implementada (importación, exportación, tablas) que sea similar a `/SAPDS/RFC_READ_TABLE2`, que es la interfaz predeterminada que usa Data Factory.<br>Data Factory | No       |
| `partitionOption`                  | El mecanismo de partición para leer desde una tabla de SAP. Las opciones admitidas incluyen: <ul><li>`None`</li><li>`PartitionOnInt` (valores de entero normales o con relleno de ceros a la izquierda, como `0000012345`)</li><li>`PartitionOnCalendarYear` (4 dígitos con el formato "YYYY")</li><li>`PartitionOnCalendarMonth` (6 dígitos con el formato "YYYYMM")</li><li>`PartitionOnCalendarDate` (8 dígitos con el formato "YYYYMMDD")</li></ul> | No       |
| `partitionColumnName`              | El nombre de la columna que se usa para particionar los datos.                | No       |
| `partitionUpperBound`              | El valor máximo de la columna especificada en `partitionColumnName` que se usará para continuar con la partición. | No       |
| `partitionLowerBound`              | El valor mínimo de la columna especificada en `partitionColumnName` que se usará para continuar con la partición. (Nota: `partitionLowerBound` no puede ser "0" si la opción de partición es `PartitionOnInt`). | No       |
| `maxPartitionsNumber`              | Número máximo de particiones para dividir los datos.     | No       |
| `sapDataColumnDelimiter` | El único carácter que se usa como delimitador que se pasa al RFC de SAP para dividir los datos de salida. | No |

>[!TIP]
>Si su tabla de SAP tiene un gran volumen de datos, como varios miles de millones de filas, use `partitionOption` y `partitionSetting` para dividir los datos en particiones más pequeñas. En este caso, los datos se leen por partición, y cada partición de datos se recupera de su servidor SAP a través de una sola llamada a RFC.<br/>
<br/>
>Tomando `partitionOption` como `partitionOnInt` a modo de ejemplo, el número de filas de cada partición se calcula con esta fórmula: (filas totales entre `partitionUpperBound` y `partitionLowerBound`)/`maxPartitionsNumber`.<br/>
<br/>
>Para cargar particiones de datos en paralelo para acelerar la copia, el grado paralelo se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de la actividad de copia. Por ejemplo, si establece `parallelCopies` como cuatro, Data Factory genera y ejecuta al mismo tiempo cuatro consultas de acuerdo con la configuración y la opción de partición que ha especificado, y cada consulta recupera una porción de datos de la tabla de SAP. Se recomienda encarecidamente hacer que `maxPartitionsNumber` sea un múltiplo del valor de la propiedad `parallelCopies`. Cuando se copian datos en un almacén de datos basado en archivos, también se recomienda escribir en una carpeta como varios archivos (solo especifique el nombre de la carpeta), en cuyo caso el rendimiento es mejor que escribir en un único archivo.

En `rfcTableOptions`, puede usar los siguientes operadores de consulta SAP comunes para filtrar las filas:

| Operator | Descripción |
| :------- | :------- |
| `EQ` | Igual a |
| `NE` | No es igual a |
| `LT` | Menor que |
| `LE` | Menor o igual que |
| `GT` | Mayor que |
| `GE` | Mayor o igual que |
| `IN` | Como en `TABCLASS IN ('TRANSP', 'INTTAB')` |
| `LIKE` | Como en `LIKE 'Emma%'` |

### <a name="example"></a>Ejemplo

```json
"activities":[
    {
        "name": "CopyFromSAPTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP table input dataset name>",
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
                "type": "SapTableSource",
                "partitionOption": "PartitionOnInt",
                "partitionSettings": {
                     "partitionColumnName": "<partition column name>",
                     "partitionUpperBound": "2000",
                     "partitionLowerBound": "1",
                     "maxPartitionsNumber": 500
                 }
            },
            "sink": {
                "type": "<sink type>"
            },
            "parallelCopies": 4
        }
    }
]
```

## <a name="data-type-mappings-for-an-sap-table"></a>Asignaciones de tipos de datos para una tabla de SAP

Al copiar datos desde una tabla de SAP, se usan las siguientes asignaciones de tipos de datos de tabla de SAP en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para más información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo ABAP para SAP | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| `C` (cadena) | `String` |
| `I` (entero) | `Int32` |
| `F` (float) | `Double` |
| `D` (fecha) | `String` |
| `T` (hora) | `String` |
| `P` (BCD empaquetado, moneda, decimal, cantidad) | `Decimal` |
| `N` (numérico) | `String` |
| `X` (binario y sin procesar) | `String` |

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Pasos siguientes

Consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
