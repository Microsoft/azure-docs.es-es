---
title: Copia de datos de Amazon Simple Storage Service (S3)
description: Obtenga información sobre cómo copiar datos desde Amazon Simple Storage Service (S3) a almacenes de datos de receptor compatibles mediante Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/31/2020
ms.openlocfilehash: 584aafccf146f3f261a7d375ecb57dcde18d03d5
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182554"
---
# <a name="copy-data-from-amazon-simple-storage-service-by-using-azure-data-factory"></a>Copia de datos desde Amazon Simple Storage Service mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que esté usando:"]
>
> * [Versión 1](v1/data-factory-amazon-simple-storage-service-connector.md)
> * [Versión actual](connector-amazon-simple-storage-service.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe cómo copiar datos desde Amazon Simple Storage Service (Amazon S3). Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

>[!TIP]
>Más información sobre el escenario de migración de datos de Amazon S3 a Azure Storage, consulte el artículo [Uso de Azure Data Factory para migrar datos de Amazon S3 a Azure Storage](data-migration-guidance-s3-azure-storage.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Amazon S3 es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad de eliminación](delete-activity.md)

Concretamente, este conector de Amazon S3 admite la copia de archivos tal cual, o el análisis de los mismos con los [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md). También puede optar por [conservar los metadatos de archivo durante la copia](#preserve-metadata-during-copy). El conector usa [AWS Signature versión 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) para autenticar las solicitudes a S3.

>[!TIP]
>Puede usar este conector de Amazon S3 para copiar datos desde *cualquier proveedor de almacenamiento compatible con S3*, como [Google Cloud Storage](connector-google-cloud-storage.md). Especifique la dirección URL del servicio correspondiente en la configuración del servicio vinculado.

## <a name="required-permissions"></a>Permisos necesarios

Para copiar datos de Amazon S3, asegúrese de que se han concedido los siguientes permisos:

- **Para la ejecución de la actividad de copia**: `s3:GetObject` y `s3:GetObjectVersion` para las operaciones de objeto de Amazon S3.
- **Para la creación de GUI de Data Factory**: `s3:ListAllMyBuckets` y `s3:ListBucket`/`s3:GetBucketLocation` para las operaciones de cubo de Amazon S3. También se requieren permisos para operaciones como la prueba de conexiones y la exploración de rutas de acceso de archivo. Si no quiere conceder estos permisos, omita la conexión de prueba en la página de creación de servicios vinculados y especifique la ruta de acceso directamente en la configuración del conjunto de datos.

Para obtener la lista completa de los permisos de Amazon S3, consulte [Specifying Permissions in a Policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html) (Especificación de permisos en una directiva) en el sitio de AWS.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)] 

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de Amazon S3.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Amazon S3:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AmazonS3**. | Sí |
| accessKeyId | Id. de la clave de acceso secreta. |Sí |
| secretAccessKey | La propia clave de acceso secreta. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. |Sí |
| serviceUrl | Especifique el punto de conexión personalizado S3 si va a copiar datos desde un proveedor de almacenamiento compatible con S3 distinto al servicio Amazon S3 oficial. Por ejemplo, para copiar datos de Google Cloud Storage, especifique `https://storage.googleapis.com`. | No |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. |No |

Este conector requiere claves de acceso para que una cuenta de AWS Identity and Access Management (IAM) pueda copiar datos de Amazon S3. Ya no se admiten las [credenciales de seguridad temporal](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html).

>[!TIP]
>Especifique la dirección URL personalizada del servicio S3 si va a copiar datos desde un almacenamiento compatible con S3 que sea distinto del servicio Amazon S3 oficial.

Este es un ejemplo:

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AmazonS3",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": {
                "type": "SecureString",
                "value": "<secret access key>"
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

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para Amazon S3 en la configuración `location` de un conjunto de datos basado en formato:

| Propiedad   | Descripción                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad **type** en la sección `location` de un conjunto de datos se debe establecer en **AmazonS3Location**. | Sí      |
| bucketName | Nombre del depósito de S3.                                          | Sí      |
| folderPath | Ruta de acceso a la carpeta del cubo especificado. Si quiere usar un carácter comodín para filtrar la carpeta, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |
| fileName   | Nombre de archivo en el cubo y la ruta de acceso de la carpeta indicados. Si quiere usar un carácter comodín para filtrar archivos, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |
| version | La versión del objeto S3 si está habilitado el control de versiones de S3. Si no se especifica, se obtendrá la versión más reciente. |No |

**Ejemplo**:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AmazonS3Location",
                "bucketName": "bucketname",
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de Amazon S3.

### <a name="amazon-s3-as-a-source-type"></a>Amazon S3 como tipo de origen

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para Amazon S3 en la configuración `storeSettings` de un origen de la actividad de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio                                                    |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| type                     | La propiedad **type** de la sección `storeSettings` se debe establecer en **AmazonS3ReadSettings**. | Sí                                                         |
| ***Buscar los archivos que se van a copiar:*** |  |  |
| OPCIÓN 1: Ruta de acceso estática<br> | Realice la copia desde el cubo o la ruta de acceso de archivos o carpeta especificadas en el conjunto de datos. Si quiere copiar todos los archivos de un cubo o carpeta, especifique también `wildcardFileName` como `*`. |  |
| OPCIÓN 2: Prefijo S3<br>- prefix | Prefijo del nombre de la clave de S3 en el cubo especificado que se configuró en el conjunto de datos para filtrar archivos de S3 de origen. Se seleccionan las claves de S3 cuyo nombre comienza con `bucket_in_dataset/this_prefix`. Emplea el filtro del servicio de S3, que proporciona un mejor rendimiento que el filtro de un carácter comodín. | No |
| OPCIÓN 3: carácter comodín<br>- wildcardFolderPath | Ruta de acceso de carpeta con caracteres comodín en el cubo especificado configurado en un conjunto de datos para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de la carpeta contiene un carácter comodín o este carácter de escape. <br>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | No                                            |
| OPCIÓN 3: carácter comodín<br>- wildcardFileName | Nombre de archivo con caracteres comodín en el cubo y la ruta de carpeta (o ruta de carpeta con carácter comodín) indicada para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de la carpeta contiene un carácter comodín o este carácter de escape.  Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí |
| OPCIÓN 4: una lista de archivos<br>- fileListPath | Indica que se copie un conjunto de archivos determinado. Apunte a un archivo de texto que incluya una lista de los archivos que quiere copiar, con un archivo por línea, que sea la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos.<br/>Al usar esta opción, no especifique un nombre de archivo en el conjunto de datos. Ver más ejemplos en [Ejemplos de lista de archivos](#file-list-examples). |No |
| ***Configuración adicional:*** |  | |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando **recursive** se establece en **true** y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. <br>Los valores permitidos son: **True** (valor predeterminado) y **False**.<br>Esta propiedad no se aplica al configurar `fileListPath`. |No |
| deleteFilesAfterCompletion | Indica si los archivos binarios se eliminarán del almacén de origen después de moverse correctamente al almacén de destino. Cada archivo se elimina individualmente, de modo que cuando se produzca un error en la actividad de copia, algunos archivos ya se habrán copiado al destino y se habrán eliminado del origen, mientras que otros seguirán aún en el almacén de origen. <br/>Esta propiedad solo es válida en el escenario de copia de archivos binarios, en el que los almacenes de origen de datos son Blob, ADLS Gen1, ADLS Gen2, S3, Google Cloud Storage, File, Azure File, SFTP o FTP. El valor predeterminado es false. |No |
| modifiedDatetimeStart    | Los archivos se filtran en función del atributo Last Modified. <br>Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a una zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.<br/>Esta propiedad no se aplica al configurar `fileListPath`. | No                                            |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | No                                                          |
| enablePartitionDiscovery | En el caso de archivos con particiones, especifique si quiere analizar las particiones de la ruta de acceso del archivo y agregarlas como columnas de origen adicionales.<br/>Los valores permitidos son **false** (valor predeterminado) y **true**. | No                                            |
| partitionRootPath | Cuando esté habilitada la detección de particiones, especifique la ruta de acceso raíz absoluta para poder leer las carpetas con particiones como columnas de datos.<br/><br/>Si no se especifica, de forma predeterminada,<br/>- Cuando se usa la ruta de acceso de archivo en un conjunto de datos o una lista de archivos del origen, la ruta de acceso raíz de la partición es la ruta de acceso configurada en el conjunto de datos.<br/>- Cuando se usa el filtro de carpeta con caracteres comodín, la ruta de acceso raíz de la partición es la subruta antes del primer carácter comodín.<br/>- Cuando se usa un prefijo, la ruta de acceso raíz de la partición es la subruta antes del último "/". <br/><br/>Por ejemplo, supongamos que configura la ruta de acceso en el conjunto de datos como "root/folder/year=2020/month=08/day=27":<br/>- Si especifica la ruta de acceso raíz de la partición como "root/folder/year=2020", la actividad de copia generará dos columnas más, `month` y `day`, con el valor "08" y "27", respectivamente, además de las columnas de los archivos.<br/>- Si no se especifica la ruta de acceso raíz de la partición, no se generará ninguna columna adicional. | No                                            |
| maxConcurrentConnections | Número de conexiones simultáneas al almacén de datos. Solo se especifica cuando se quieren limitar las conexiones simultáneas al almacén de datos. | No                                                          |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromAmazonS3",
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
                    "type": "AmazonS3ReadSettings",
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

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

Esta sección describe el comportamiento resultante de la ruta de acceso de la carpeta y el nombre de archivo con los filtros de carácter comodín.

| bucket | key | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en negrita)|
|:--- |:--- |:--- |:--- |
| bucket | `Folder*/*` | false | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| bucket | `Folder*/*` | true | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| bucket | `Folder*/*.csv` | false | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| bucket | `Folder*/*.csv` | true | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="file-list-examples"></a>Ejemplos de lista de archivos

En esta sección se describe el comportamiento resultante de usar una ruta de acceso de la lista de archivos en un origen de la actividad de copia.

Suponga que tiene la siguiente estructura de carpetas de origen y quiere copiar los archivos en negrita:

| Estructura de origen de ejemplo                                      | Contenido de FileListToCopy.txt                             | Configuración de Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Metadatos<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **En el conjunto de datos:**<br>- Cubo: `bucket`<br>- Ruta de acceso de la carpeta: `FolderA`<br><br>**En el origen de la actividad de copia:**<br>- Ruta de acceso de la lista de archivos: `bucket/Metadata/FileListToCopy.txt` <br><br>La ruta de acceso de la lista de archivos apunta a un archivo de texto en el mismo almacén de datos que incluye una lista de los archivos que se quieren copiar, con un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos. |

## <a name="preserve-metadata-during-copy"></a>Conservación de los metadatos durante la copia

Al copiar archivos de Amazon S3 en Azure Data Lake Storage Gen2 o Azure Blob Storage, puede optar por conservar los metadatos de archivo junto con los datos. Más información en [Conservación de metadatos](copy-activity-preserve-metadata.md#preserve-metadata).

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para información detallada sobre las propiedades, consulte [la actividad GetMetadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propiedades de la actividad de eliminación

Para información detallada sobre las propiedades, consulte [Actividad de eliminación](delete-activity.md).

## <a name="legacy-models"></a>Modelos heredados

>[!NOTE]
>Estos modelos siguen siendo compatibles con versiones anteriores. Se recomienda usar el nuevo modelo mencionado anteriormente. La interfaz de usuario de creación en Data Factory ha cambiado para generar el nuevo modelo.

### <a name="legacy-dataset-model"></a>Modelo de conjunto de datos heredado

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del conjunto de datos debe establecerse en 8**AmazonS3Object**. |Sí |
| bucketName | Nombre del depósito de S3. No se admite el filtros con caracteres comodín. |Sí, para la actividad de copia o búsqueda; no, para la actividad GetMetadata |
| key | El filtro de nombre o de carácter comodín de la clave del objeto S3 del cubo especificado. Solo se aplica cuando no se especifica la propiedad **prefix**. <br/><br/>El filtro de comodín se admite tanto para el elemento de carpeta como para el elemento del nombre de archivo. Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"key": "rootfolder/subfolder/*.csv"`<br/>- Ejemplo 2: `"key": "rootfolder/subfolder/???20180427.txt"`<br/>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). Use `^` como escape si el nombre real de la carpeta o archivo contiene un carácter comodín o este carácter de escape. |No |
| prefix | Prefijo de la clave del objeto S3. Se seleccionan objetos cuyas claves comienzan por este prefijo. Solo se aplica cuando no se especifica la propiedad **key**. |No |
| version | La versión del objeto S3 si está habilitado el control de versiones de S3. Si no se especifica una versión, se obtendrá la más reciente. |No |
| modifiedDatetimeStart | Los archivos se filtran en función del atributo Last Modified. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que al habilitar esta configuración se verá afectado el rendimiento general del movimiento de datos cuando quiera filtrar grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.| No |
| modifiedDatetimeEnd | Los archivos se filtran en función del atributo Last Modified. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que al habilitar esta configuración se verá afectado el rendimiento general del movimiento de datos cuando quiera filtrar grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.| No |
| format | Si quiere copiar los archivos tal cual entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si quiere analizar o generar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** en **format** en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Formato JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Formato AVRO](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Formato ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Los tipos admitidos son **GZip**, **Deflate**, **BZip2** y **ZipDeflate**.<br/>Niveles admitidos son **Optimal** y **Fastest**. |No |

>[!TIP]
>Para copiar todos los archivos en una carpeta, especifique **bucketName** para el cubo y **prefix** para el elemento de carpeta.
>
>Para copiar un único archivo con un nombre determinado, especifique **bucketName** para el cubo y **key** para el elemento de carpeta más el nombre del archivo.
>
>Para copiar un subconjunto de archivos en una carpeta, especifique **bucketName** para el cubo y **key** para el elemento de carpeta más el filtro de comodín.

**Ejemplo: uso de prefijo**

```json
{
    "name": "AmazonS3Dataset",
    "properties": {
        "type": "AmazonS3Object",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "bucketName": "testbucket",
            "prefix": "testFolder/test",
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

**Ejemplo: uso de clave y versión (opcional)**

```json
{
    "name": "AmazonS3Dataset",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "bucketName": "testbucket",
            "key": "testFolder/testfile.csv.gz",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
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

### <a name="legacy-source-model-for-the-copy-activity"></a>Modelo de origen heredado para la actividad de copia

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia debe establecerse en **FileSystemSource**. |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando **recursive** esté establecido en **true** y el receptor sea un almacén basado en archivos, no se copiará ni creará una subcarpeta o carpeta vacía en el receptor.<br/>Los valores permitidos son: **True** (valor predeterminado) y **False**. | No |
| maxConcurrentConnections | Número de conexiones para conectarse al almacén de datos de forma simultánea. Solo se especifica cuando se quieren limitar las conexiones simultáneas al almacén de datos. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromAmazonS3",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Amazon S3 input dataset name>",
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
                "type": "FileSystemSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes
Para ver una lista de los almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores, consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats).
