---
title: Copia y transformación de los datos en Azure Data Lake Storage Gen2
description: Aprenda a copiar datos utilizando Azure Data Lake Storage Gen2 como origen y como destino y a transformar los datos de Azure Data Lake Storage Gen2 mediante Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/18/2020
ms.openlocfilehash: 9eed71f5a264a6308a49ac594563c746bb8ea5b7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544781"
---
# <a name="copy-and-transform-data-in-azure-data-lake-storage-gen2-using-azure-data-factory"></a>Copia y transformación de los datos de Azure Data Lake Storage Gen2 mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Lake Storage Gen2 (ADLS Gen2) es un conjunto de funcionalidades dedicadas al análisis de macrodatos integrado en [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md). Puede usarlo para interactuar con los datos usando el paradigma de sistema de archivos o el de almacenamiento de objetos.

En este artículo, se explica el uso de la actividad de copia en Azure Data Factory para copiar datos utilizando Azure Data Lake Storage Gen2 como origen o destino y el uso de Data Flow para transformar datos en Azure Data Lake Storage Gen2. Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

>[!TIP]
>Para ver un escenario de migración de un lago de datos o un almacenamiento de datos, obtenga más información en [Uso de Azure Data Factory para migrar datos del lago de datos y el almacenamiento de datos a Azure](data-migration-guidance-overview.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Data Lake Storage Gen2 es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad de eliminación](delete-activity.md)

En el caso de la actividad de copia, con este conector puede:

- Copiar datos utilizando Azure Data Lake Storage Gen2 como origen o destino con la clave de cuenta, la entidad de servicio o identidades administradas para las autenticaciones de recursos de Azure.
- Copiar los archivos tal y como están, o bien analizarlos o generarlos con [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).
- [Conservación de los metadatos de archivo durante la copia](#preserve-metadata-during-copy).
- [Conservar ACL](#preserve-acls) al copiar desde Azure Data Lake Storage Gen1/Gen2.

>[!IMPORTANT]
>Si habilita la opción **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** en la configuración de firewall de Azure Storage y quiere usar Azure Integration Runtime para conectarse a Data Lake Storage Gen2, debe usar la [autenticación de identidad administrada](#managed-identity) para ADLS Gen2.


## <a name="get-started"></a>Introducción

>[!TIP]
>Para ver un tutorial sobre cómo usar el conector de Data Lake Storage Gen2, consulte [Carga de datos en Azure Data Lake Storage Gen2](load-azure-data-lake-storage-gen2.md).

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporciona información acerca de las propiedades que se usan para definir entidades de Data Factory específicas para Data Lake Storage Gen2.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

El conector de Azure Data Lake Storage Gen2 admite los siguientes tipos de autenticación. Consulte las secciones correspondientes para más información:

- [Autenticación de clave de cuenta](#account-key-authentication)
- [Autenticación de entidad de servicio](#service-principal-authentication)
- [Identidades administradas para la autenticación de recursos de Azure](#managed-identity)

>[!NOTE]
>Cuando usa PolyBase para cargar datos en SQL Data Warehouse, si la instancia de origen de Data Lake Storage Gen2 está configurada con un punto de conexión de Virtual Network, deberá usar la autenticación de identidad administrada como requiere PolyBase. Consulte la sección sobre [autenticación de identidad administrada](#managed-identity) que incluye más requisitos previos de configuración.

### <a name="account-key-authentication"></a>Autenticación de clave de cuenta

Para usar la autenticación de clave de cuenta de almacenamiento, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AzureBlobFS**. |Sí |
| url | Punto de conexión de Data Lake Storage Gen2 con el patrón de `https://<accountname>.dfs.core.windows.net`. | Sí |
| accountKey | Clave de cuenta para Data Lake Storage Gen2. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). |Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si el almacén de datos está en una red privada, se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado. Si no se especifica esta propiedad, se usa el valor predeterminado de Azure Integration Runtime. |No |

>[!NOTE]
>No se admite el punto de conexión del sistema de archivos ADLS secundario al usar la autenticación de clave de cuenta. Puede usar otros tipos de autenticación.

**Ejemplo**:

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "accountkey": { 
                "type": "SecureString", 
                "value": "<accountkey>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Antes de usar la autenticación de entidad de servicio, siga estos pasos.

1. Registre una entidad de aplicación en Azure Active Directory (Azure AD) como se indica en [Registro de la aplicación con un inquilino de Azure AD](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Anote los siguientes valores; los usará para definir el servicio vinculado:

    - Identificador de aplicación
    - Clave de la aplicación
    - Id. de inquilino

2. Conceda a la entidad de servicio el permiso adecuado. Consulte ejemplos sobre el funcionamiento del permiso en Data Lake Storage Gen2 en [Listas de control de acceso en archivos y directorios](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

    - **Como origen**: en el Explorador de Storage, conceda al menos permiso de **ejecución** para TODAS las carpetas de nivel superior y el sistema de archivos, junto con el permiso de **lectura** para los archivos que se van a copiar. Como alternativa, en el control de acceso (IAM), conceda al menos el rol **Lector de datos de Storage Blob**.
    - **Como receptor**: en el Explorador de Storage, conceda al menos permiso de **ejecución** para TODAS las carpetas de nivel superior y el sistema de archivos, junto con el permiso de **escritura** para la carpeta del receptor. Como alternativa, en el control de acceso (IAM), conceda al menos el rol **Colaborador de datos de Storage Blob**.

>[!NOTE]
>Si usa la interfaz de usuario de Data Factory para la creación, y la entidad de servicio no está configurada con el rol "Lector o colaborador de datos de Storage Blob" en IAM, al realizar las pruebas de conexión o al explorar las carpetas o navegar por ellas, elija "Test connection to file path" (Probar conexión con la ruta de acceso del archivo) o "Browse from specified path" (Examinar desde la ruta de acceso especificada) y especifique una ruta de acceso con el permiso de **lectura y ejecución** para continuar.

Estas propiedades son compatibles con el servicio vinculado:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AzureBlobFS**. |Sí |
| url | Punto de conexión de Data Lake Storage Gen2 con el patrón de `https://<accountname>.dfs.core.windows.net`. | Sí |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación. Marque este campo como `SecureString` para almacenarlo de forma segura en Data Factory. O bien puede [hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |
| azureCloudType | Para la autenticación de la entidad de servicio, especifique el tipo de entorno de nube de Azure en el que está registrada la aplicación de Azure Active Directory. <br/> Los valores permitidos son **AzurePublic**, **AzureChina**, **AzureUsGovernment** y **AzureGermany**. De forma predeterminada, se usa el entorno de nube de la factoría de datos. | No |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si el almacén de datos está en una red privada, se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado. Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

**Ejemplo**:

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>" 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Identidades administradas para la autenticación de recursos de Azure

Una factoría de datos se puede asociar con una [identidad administrada para recursos de Azure](data-factory-service-identity.md), que representa esa factoría de datos concreta. Puede usar directamente esta identidad administrada para la autenticación en Data Lake Storage Gen2, de forma similar a como usa su propia entidad de servicio. Permite que esta fábrica designada acceda a datos y los copie en Data Lake Storage Gen2 o desde este.

Para usar identidades administradas para la autenticación de recursos de Azure, siga estos pasos.

1. [Recupere la información de la identidad administrada de Data Factory](data-factory-service-identity.md#retrieve-managed-identity) mediante la copia del valor de **Id. del objeto de identidad administrada** que se ha generado junto con la factoría.

2. Conceda el permiso adecuado de identidad administrada. Consulte ejemplos sobre el funcionamiento del permiso en Data Lake Storage Gen2 en [Listas de control de acceso en archivos y directorios](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

    - **Como origen**: en el Explorador de Storage, conceda al menos permiso de **ejecución** para TODAS las carpetas de nivel superior y el sistema de archivos, junto con el permiso de **lectura** para los archivos que se van a copiar. Como alternativa, en el control de acceso (IAM), conceda al menos el rol **Lector de datos de Storage Blob**.
    - **Como receptor**: en el Explorador de Storage, conceda al menos permiso de **ejecución** para TODAS las carpetas de nivel superior y el sistema de archivos, junto con el permiso de **escritura** para la carpeta del receptor. Como alternativa, en el control de acceso (IAM), conceda al menos el rol **Colaborador de datos de Storage Blob**.

>[!NOTE]
>Si usa la interfaz de usuario de Data Factory para la creación, y la identidad administrada no está configurada con el rol "Lector o colaborador de datos de Storage Blob" en IAM, al realizar las pruebas de conexión o al explorar las carpetas o navegar por ellas, elija "Test connection to file path" (Probar conexión con la ruta de acceso del archivo) o "Browse from specified path" (Examinar desde la ruta de acceso especificada) y especifique una ruta de acceso con el permiso de **lectura y ejecución** para continuar.

>[!IMPORTANT]
>Si usa PolyBase para cargar datos desde Data Lake Storage Gen2 en SQL Data Warehouse, al utilizar la autenticación de identidad administrada de Data Lake Storage Gen2 asegúrese de que también sigue los pasos 1 y 2 de [esta guía](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage) para 1) registrar el servidor en Azure Active Directory (Azure AD) y 2) asignar el rol Colaborador de datos de Storage Blob a su servidor de SQL Database; Data Factory controlará lo demás. Si la instancia de Data Lake Storage Gen2 está configurada con un punto de conexión de Azure Virtual Network, para usar PolyBase para cargar datos desde este deberá usar la autenticación de identidad administrada como requiere PolyBase.

Estas propiedades son compatibles con el servicio vinculado:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AzureBlobFS**. |Sí |
| url | Punto de conexión de Data Lake Storage Gen2 con el patrón de `https://<accountname>.dfs.core.windows.net`. | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si el almacén de datos está en una red privada, se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado. Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

**Ejemplo**:

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos](concepts-datasets-linked-services.md).

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para Data Lake Store Gen2 en la configuración `location` del conjunto de datos basado en formato:

| Propiedad   | Descripción                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad type de `location` del conjunto de datos debe establecerse en **AzureBlobFSLocation**. | Sí      |
| fileSystem | Nombre del sistema de archivos de Data Lake Storage Gen2.                              | No       |
| folderPath | Ruta de acceso a una carpeta en el sistema de archivos especificado. Si quiere usar un carácter comodín para filtrar carpetas, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |
| fileName   | Nombre de archivo en la propiedad fileSystem + folderPath indicada. Si quiere usar un carácter comodín para filtrar archivos, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |

**Ejemplo**:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Data Lake Storage Gen2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobFSLocation",
                "fileSystem": "filesystemname",
                "folderPath": "folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte los artículos sobre [configuraciones de actividades de copia](copy-activity-overview.md#configuration) y [canalizaciones y actividades](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que el receptor y el origen de Data Lake Storage Gen2 admiten.

### <a name="azure-data-lake-storage-gen2-as-a-source-type"></a>Azure Data Lake Storage Gen2 como tipo de origen

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Tiene varias opciones para copiar datos de ADLS Gen2:

- Copiar la ruta de acceso de archivo especificada en el conjunto de datos.
- Filtrar con caracteres comodín en la ruta de acceso de la carpeta o el nombre de archivo (vea `wildcardFolderPath` y `wildcardFileName`).
- Copiar los archivos definidos en un archivo de texto determinado como un conjunto de archivos (vea `fileListPath`).

Las propiedades siguientes se admiten para Data Lake Store Gen2 en la configuración `storeSettings` en el origen de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propiedad type de `storeSettings` se debe establecer en **AzureBlobFSReadSettings**. | Sí                                           |
| ***Buscar los archivos que se van a copiar:*** |  |  |
| OPCIÓN 1: ruta de acceso estática<br> | Copia de la ruta de acceso de carpeta/sistema de archivos especificada en el conjunto de datos. Si quiere copiar todos los archivos de una carpeta o un sistema de archivos, especifique también `wildcardFileName` como `*`. |  |
| OPCIÓN 2: carácter comodín<br>- wildcardFolderPath | Ruta de acceso de carpeta con caracteres comodín en el sistema de archivos especificado configurado en el conjunto de datos para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro. <br>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | No                                            |
| OPCIÓN 2: carácter comodín<br>- wildcardFileName | Nombre de archivo con caracteres comodín en la propiedad file system + folderPath o wildcardFolderPath indicada para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro.  Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí |
| OPCIÓN 3: una lista de archivos<br>- fileListPath | Indica que se copie un conjunto de archivos determinado. Apunte a un archivo de texto que incluya una lista de los archivos que quiere copiar, con un archivo por línea, que sea la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos.<br/>Al utilizar esta opción, no especifique el nombre de archivo en el conjunto de datos. Ver más ejemplos en [Ejemplos de lista de archivos](#file-list-examples). |No |
| ***Configuración adicional:*** |  | |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando recursive se establece en true y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. <br>Los valores permitidos son: **True** (valor predeterminado) y **False**.<br>Esta propiedad no se aplica al configurar `fileListPath`. |No |
| deleteFilesAfterCompletion | Indica si los archivos binarios se eliminarán del almacén de origen después de moverse correctamente al almacén de destino. Cada archivo se elimina individualmente, de modo que cuando se produzca un error en la actividad de copia, algunos archivos ya se habrán copiado al destino y se habrán eliminado del origen, mientras que otros seguirán aún en el almacén de origen. <br/>Esta propiedad solo es válida en el escenario de copia de archivos binarios, en el que los almacenes de origen de datos son Blob, ADLS Gen1, ADLS Gen2, S3, Google Cloud Storage, File, Azure File, SFTP o FTP. El valor predeterminado es false. |No |
| modifiedDatetimeStart    | Filtro de archivos basado en el atributo: Última modificación. <br>Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.<br/>Esta propiedad no se aplica al configurar `fileListPath`. | No                                            |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | No                                            |
| maxConcurrentConnections | Número de conexiones para conectarse al almacén de almacenamiento de forma simultánea. Solo se especifica cuando se quiere limitar la conexión simultánea al almacén de datos. | No                                            |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AzureBlobFSReadSettings",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-data-lake-storage-gen2-as-a-sink-type"></a>Azure Data Lake Storage Gen2 como tipo de receptor

[!INCLUDE [data-factory-v2-file-sink-formats](../../includes/data-factory-v2-file-sink-formats.md)]

Las propiedades siguientes se admiten para Data Lake Store Gen2 en la configuración `storeSettings` en el destino de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propiedad type de `storeSettings` se debe establecer en **AzureBlobFSWriteSettings**. | Sí      |
| copyBehavior             | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen a la carpeta de origen es idéntica que la ruta de acceso relativa del archivo de destino a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del archivo, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. | No       |
| blockSizeInMB | Especifique el tamaño de bloque en MB que se usa para escribir datos en ADLS Gen2. Más información [sobre los blobs en bloques](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs). <br/>El valor permitido está **entre 4 y 100 MB**. <br/>De forma predeterminada, ADF determina automáticamente el tamaño de bloque en función del tipo y los datos del almacén de origen. En el caso de una copia no binaria en ADLS Gen2, el tamaño de bloque predeterminado es de 100 MB, con el fin de ajustarse como máximo a 4,95 TB de datos. Este tamaño puede no ser óptimo cuando los datos no son grandes, en especial cuando se usa el entorno de ejecución de integración autohospedado con una red deficiente, lo que produce un problema de tiempo de espera de operación o rendimiento. Puede especificar explícitamente un tamaño de bloque, al tiempo que se garantiza que blockSizeInMB *50000 es lo suficientemente grande como para almacenar los datos; de lo contrario, se producirá un error en la ejecución de la actividad de copia. | No |
| maxConcurrentConnections | Número de conexiones para conectarse al almacén de datos de forma simultánea. Solo se especifica cuando se quiere limitar la conexión simultánea al almacén de datos. | No       |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Parquet output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "ParquetSink",
                "storeSettings":{
                    "type": "AzureBlobFSWriteSettings",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

Esta sección describe el comportamiento resultante de la ruta de acceso de la carpeta y el nombre de archivo con los filtros de carácter comodín.

| folderPath | fileName | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en **negrita**)|
|:--- |:--- |:--- |:--- |
| `Folder*` | (Vacío, usar el valor predeterminado) | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | (Vacío, usar el valor predeterminado) | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="file-list-examples"></a>Ejemplos de lista de archivos

En esta sección se describe el comportamiento resultante de usar la ruta de acceso de la lista de archivos en el origen de la actividad de copia.

Suponga que tiene la siguiente estructura de carpetas de origen y quiere copiar los archivos en negrita:

| Estructura de origen de ejemplo                                      | Contenido de FileListToCopy.txt                             | Configuración de ADF                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| fileSystem<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Metadatos<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **En el conjunto de datos:**<br>Sistema de archivos: `filesystem`<br>Ruta de acceso a la carpeta: `FolderA`<br><br>**En origen de la actividad de copia:**<br>- Ruta de acceso de la lista de archivos: `filesystem/Metadata/FileListToCopy.txt` <br><br>La ruta de acceso de la lista de archivos apunta a un archivo de texto en el mismo almacén de datos que incluye una lista de archivos que se quiere copiar, un archivo por línea con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos. |


### <a name="some-recursive-and-copybehavior-examples"></a>Algunos ejemplos de recursive y copyBehavior

En esta sección se describe el comportamiento resultante de la operación de copia para diferentes combinaciones de valores recursive y copyBehavior.

| recursive | copyBehavior | Estructura de carpetas de origen | Destino resultante |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | El destino Folder1 se crea con la misma estructura que el origen:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File5 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 + File3 + File4 + File5 se combina en un archivo con un nombre de archivo generado automáticamente. |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>No se selecciona la subcarpeta Subfolder1, con File3, File4 y File5. |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File2<br/><br/>No se selecciona la subcarpeta Subfolder1, con File3, File4 y File5. |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 se combina en un archivo con un nombre de archivo generado automáticamente. nombre de archivo generado automáticamente para File1<br/><br/>No se selecciona la subcarpeta Subfolder1, con File3, File4 y File5. |

## <a name="preserve-metadata-during-copy"></a>Conservación de los metadatos durante la copia

Al copiar archivos de Amazon S3/Blob de Azure/Azure Data Lake Storage Gen2 a Azure Data Lake Storage Gen2/Blob de Azure, puede optar por conservar los metadatos de archivo junto con los datos. Más información en [Conservación de metadatos](copy-activity-preserve-metadata.md#preserve-metadata).

## <a name="preserve-acls-from-data-lake-storage-gen1gen2"></a><a name="preserve-acls"></a> Conservación de ACL desde Data Lake Storage Gen1/Gen2

Al copiar archivos de Azure Data Lake Storage Gen1/Gen2 a Gen2, puede elegir conservar las listas de control de acceso (ACL) de POSIX junto con los datos. Obtenga más información en [Conservar las ACL de Data Lake Storage Gen1/Gen2 en Gen2 ](copy-activity-preserve-metadata.md#preserve-acls).

>[!TIP]
>Para copiar datos desde Azure Data Lake Storage Gen1 en Gen2 en general, consulte [Copia de datos de Azure Data Lake Storage Gen1 en Gen2 con Azure Data Factory](load-azure-data-lake-storage-gen2-from-gen1.md) para ver un tutorial y los procedimientos recomendados.

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Al transformar datos en los flujos de datos de asignación, puede leer y escribir archivos de Azure Data Lake Storage Gen2 en los siguientes formatos:
* [Avro](format-avro.md#mapping-data-flow-properties)
* [Common Data Model (versión preliminar)](format-common-data-model.md#mapping-data-flow-properties)
* [Texto delimitado](format-delimited-text.md#mapping-data-flow-properties)
* [Delta](format-delta.md#mapping-data-flow-properties)
* [Excel](format-excel.md#mapping-data-flow-properties)
* [JSON](format-json.md#mapping-data-flow-properties)
* [Parquet](format-parquet.md#mapping-data-flow-properties)

La configuración específica de formato se encuentra en la documentación de ese formato. Para obtener más información, consulte los artículos sobre la [transformación de origen en un flujo de datos de asignación](data-flow-source.md) y la [transformación de receptor en un flujo de datos de asignación](data-flow-sink.md).

### <a name="source-transformation"></a>Transformación de origen

En la transformación de origen, puede leer de un contenedor, una carpeta o un archivo individual en Azure Data Lake Storage Gen2. La pestaña **Opciones de origen** permite administrar cómo se leen los archivos. 

![Opciones de origen](media/data-flow/sourceOptions1.png "Opciones de origen")

**Ruta de acceso de comodín:** El uso de un patrón de caracteres comodín indicará a ADF que recorra todos los archivos y carpetas que coincidan en una única transformación del origen. Se trata de una manera eficaz de procesar varios archivos en un único flujo. Agregue varios patrones de coincidencia de caracteres comodín con el signo + que aparece al desplazar el puntero sobre el patrón de caracteres comodín existente.

En el contenedor de origen, elija una serie de archivos que coincidan con un patrón. Solo se puede especificar un contenedor en el conjunto de datos. La ruta de acceso con carácter comodín, por tanto, también debe incluir la ruta de acceso de la carpeta de la carpeta raíz.

Ejemplos de caracteres comodín:

* ```*``` Representa cualquier conjunto de caracteres.
* ```**``` Representa el anidamiento recursivo de directorios.
* ```?``` Reemplaza un carácter.
* ```[]``` Coincide con uno de los caracteres entre corchetes.

* ```/data/sales/**/*.csv``` Obtiene todos los archivos csv que se encuentra en /data/sales.
* ```/data/sales/20??/**/``` Obtiene todos los archivos del siglo XX.
* ```/data/sales/*/*/*.csv``` Obtiene los archivos CSV en dos niveles bajo /data/sales.
* ```/data/sales/2004/*/12/[XY]1?.csv``` Obtiene todos los archivos csv de diciembre de 2004 que comienzan con X o Y precedido por un número de dos dígitos.

**Partition Root Path:** (Ruta de acceso de la raíz de la partición) Si tiene carpetas con particiones en el origen de archivo con un formato ```key=value``` (por ejemplo, year=2019), puede asignar el nivel superior del árbol de carpetas de la partición a un nombre de columna del flujo de datos.

En primer lugar, establezca un comodín que incluya todas las rutas de acceso que sean carpetas con particiones y, además, los archivos de hoja que desee leer.

![Configuración del archivo de origen de partición](media/data-flow/partfile2.png "Configuración del archivo de partición")

Use el valor de Partition Root Path (Ruta de acceso de la raíz de la partición) para definir cuál es el nivel superior de la estructura de carpetas. Cuando vea el contenido de los datos mediante una vista previa, verá que ADF agregará las particiones resueltas que se encuentran en cada uno de los niveles de las carpetas.

![Ruta de acceso raíz de la partición](media/data-flow/partfile1.png "Vista previa de la ruta de acceso raíz de la partición")

**Lista de archivos**: Se trata de un conjunto de archivos. Cree un archivo de texto que incluya una lista de archivos de ruta de acceso relativa para procesar. Apunte a este archivo de texto.

**Column to store file name:** (Columna para almacenar el nombre de archivo) Almacene el nombre del archivo de origen en una columna de los datos. Escriba aquí el nombre de una nueva columna para almacenar la cadena de nombre de archivo.

**After completion:** (Tras finalizar) Elija no hacer nada con el archivo de origen después de que se ejecute el flujo de datos o bien elimine o mueva el archivo de origen. Las rutas de acceso para mover los archivos de origen son relativas.

Para mover archivos de origen a otra ubicación posterior al procesamiento, primero seleccione "Mover" para la operación de archivo. A continuación, establezca el directorio "from". Si no usa ningún carácter comodín para la ruta de acceso, la configuración de "from" será la misma carpeta que la carpeta de origen.

Si tiene una ruta de acceso de origen con un comodín, su sintaxis será como esta:

```/data/sales/20??/**/*.csv```

puede especificar "from" como

```/data/sales```

y "to" como

```/backup/priorSales```

En este caso, todos los subdirectorios cuyo origen se encuentra en /data/sales se mueven a /backup/priorSales.

> [!NOTE]
> Las operaciones de archivo solo se ejecutan cuando el flujo de datos se inicia desde una ejecución de canalización (depuración o ejecución de canalización) que usa la actividad de ejecución de Data Flow de una canalización. Las operaciones de archivo *no* se ejecutan en modo de depuración de Data Flow.

**Filter by last modified:** (Filtrar últimos modificados) Puede filtrar los archivos que desea procesar especificando un intervalo de fechas de la última vez que se modificaron. Todas las horas están en formato UTC. 

### <a name="sink-properties"></a>Propiedades del receptor

En la transformación del receptor, puede escribir en un contenedor o una carpeta en Azure Data Lake Storage Gen2. La pestaña **Configuración** le permite administrar cómo se escriben los archivos.

![Opciones del receptor](media/data-flow/file-sink-settings.png "opciones de receptor")

**Clear the folder** (Borrar la carpeta): determina si se borra o no la carpeta de destino antes de escribir los datos.

**File name option** (Opción de nombre de archivo): determina cómo se denominan los archivos de destino en la carpeta de destino. Las opciones de nombre de archivo son:
   * **Valor predeterminado**: permita que Spark nombre los archivos según los valores predeterminados de PART.
   * **Patrón**: escriba un patrón que enumere los archivos de salida por partición. Por ejemplo, **loans[n].csv** creará loans1.csv, loans2.csv, etc.
   * **Por partición**: escriba un nombre de archivo por partición.
   * **Como datos de columna**: establezca el archivo de salida en el valor de una columna. La ruta de acceso es relativa al contenedor del conjunto de datos, no a la carpeta de destino. Si tiene una ruta de acceso de carpeta en el conjunto de datos, se invalidará.
   * **Salida en un solo archivo**: combine los archivos de salida con particiones en un solo archivo con nombre. La ruta de acceso es relativa a la carpeta del conjunto de datos. Tenga en cuenta que la operación de combinación probablemente produzca un error en función del tamaño del nodo. Esta opción no se recomienda para conjuntos de datos de gran tamaño.

**Quote all** (Entrecomillar todo): determina si se deben entrecomillar todos los valores.

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para información detallada sobre las propiedades, consulte [Actividad de obtención de metadatos](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propiedades de la actividad de eliminación

Para información detallada sobre las propiedades, consulte [Actividad de eliminación](delete-activity.md).

## <a name="legacy-models"></a>Modelos heredados

>[!NOTE]
>Estos modelos siguen siendo compatibles con versiones anteriores. Se recomienda usar el nuevo modelo mencionado en la sección anterior de ahora en adelante; además, la interfaz de usuario de creación de ADF ha pasado a generar el nuevo modelo.

### <a name="legacy-dataset-model"></a>Modelo de conjunto de datos heredado

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **AzureBlobFSFile**. |Sí |
| folderPath | Ruta de acceso a la carpeta de Data Lake Storage Gen2. Si no se especifica, apunta a la raíz. <br/><br/>Se admiten filtros con caracteres comodín. Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre real de la carpeta contiene un carácter comodín o este carácter de escape. <br/><br/>Ejemplos: filesystem/folder/. Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). |No |
| fileName | Filtro de nombre o de comodín para los archivos de la ruta "folderPath" especificada. Si no especifica ningún valor para esta propiedad, el conjunto de datos apunta a todos los archivos de la carpeta. <br/><br/>Para filtrar, los caracteres comodín permitidos son `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"fileName": "*.csv"`<br/>- Ejemplo 2: `"fileName": "???20180427.txt"`<br/>Use `^` como escape si el nombre de archivo real contiene un comodín o este carácter de escape.<br/><br/>Cuando fileName no se especifica para un conjunto de datos de salida y **preserveHierarchy** no se determina en el receptor de la actividad, la actividad de copia generará automáticamente el nombre de archivo con el siguiente patrón: "*Data.[GUID de Id. de ejecución de actividad].[GUID si FlattenHierarchy].[formato si se ha configurado].[compresión si se ha configurado]* ", por ejemplo, "Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz". Si realiza una copia desde el origen tabular mediante el nombre de una tabla en lugar de una consulta, el patrón del nombre es " *[nombre de tabla].[formato].[compresión, si está configurada]* ", por ejemplo, "MyTable.csv". |No |
| modifiedDatetimeStart | Filtro de archivos basado en el atributo Last Modified. Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> El rendimiento general del movimiento de datos se ve afectado si habilita esta configuración cuando quiere filtrar archivos con grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos. Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora. Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| No |
| modifiedDatetimeEnd | Filtro de archivos basado en el atributo Last Modified. Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> El rendimiento general del movimiento de datos se ve afectado si habilita esta configuración cuando quiere filtrar archivos con grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos. Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora. Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| No |
| format | Si desea copiar los archivos tal cual entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si quiere analizar o generar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** y **ParquetFormat**. Establezca la propiedad **type** en **format** en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Formato Json](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Formato Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Formato ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Los tipos admitidos son **GZip**, **Deflate**, **BZip2** y **ZipDeflate**.<br/>Niveles admitidos son **Optimal** y **Fastest**. |No |

>[!TIP]
>Para copiar todos los archivos en una carpeta, especifique solo **folderPath**.<br>Para copiar un único archivo con un nombre determinado, especifique **folderPath** con un elemento de carpeta y **fileName** con un nombre de archivo.<br>Para copiar un subconjunto de archivos en una carpeta, especifique **folderPath** con un elemento de carpeta y **fileName** con un filtro de comodín. 

**Ejemplo**:

```json
{
    "name": "ADLSGen2Dataset",
    "properties": {
        "type": "AzureBlobFSFile",
        "linkedServiceName": {
            "referenceName": "<Azure Data Lake Storage Gen2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "myfilesystem/myfolder",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

### <a name="legacy-copy-activity-source-model"></a>Modelo de origen de actividad de copia heredada

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **AzureBlobFSSource**. |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Cuando recursive se establece en true y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor.<br/>Los valores permitidos son: **True** (valor predeterminado) y **False**. | No |
| maxConcurrentConnections | Número de conexiones para conectarse al almacén de datos de forma simultánea. Solo se especifica cuando se quiere limitar la conexión simultánea al almacén de datos. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ADLS Gen2 input dataset name>",
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
                "type": "AzureBlobFSSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="legacy-copy-activity-sink-model"></a>Modelo de receptor de actividad de copia heredada

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del receptor de la actividad de copia debe establecerse en **AzureBlobFSSink**. |Sí |
| copyBehavior | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen a la carpeta de origen es idéntica que la ruta de acceso relativa del archivo de destino a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del archivo, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. | No |
| maxConcurrentConnections | Número de conexiones para conectarse al almacén de datos de forma simultánea. Solo se especifica cuando se quiere limitar la conexión simultánea al almacén de datos. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<ADLS Gen2 output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureBlobFSSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

Para ver la lista de almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
