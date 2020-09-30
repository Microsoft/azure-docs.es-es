---
title: Copia de datos del servidor SFTP o en este
description: Aprenda a copiar datos de un servidor SFTP y en este mediante Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/28/2020
ms.openlocfilehash: e8067f2aa485202412f0f0b6101eafb7768b222a
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89181908"
---
# <a name="copy-data-from-and-to-the-sftp-server-by-using-azure-data-factory"></a>Copia de datos hacia y desde un servidor SFTP mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que está usando:"]
> * [Versión 1](v1/data-factory-sftp-connector.md)
> * [Versión actual](connector-sftp.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe cómo copiar datos hacia o desde el servidor FTP seguro (SFTP). Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

El conector de SFTP admite las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad de eliminación](delete-activity.md)

En concreto, el conector de SFTP admite:

- La copia de archivos desde y hacia el servidor SFTP mediante la autenticación *Básica* o *SshPublicKey*.
- La copia de archivos tal cual, o bien el análisis o generación de archivos con los [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de SFTP.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Se admiten las siguientes propiedades para el servicio vinculado de SFTP:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en *Sftp*. |Sí |
| host | Nombre o dirección IP del servidor SFTP. |Sí |
| port | Puerto en el que escucha el servidor SFTP.<br/>El valor permitido es un entero y el valor predeterminado es *22*. |No |
| skipHostKeyValidation | Especifique si desea omitir la validación de claves de host.<br/>Los valores permitidos son *true* y *false* (predeterminado).  | No |
| hostKeyFingerprint | Especifique la huella dactilar de la clave de host. | Sí, si "skipHostKeyValidation" está establecido en false.  |
| authenticationType | Especifique el tipo de autenticación.<br/>Los valores permitidos son *Básica* y *SshPublicKey*. Para conocer más propiedades, consulte la sección [Uso de la autenticación básica](#use-basic-authentication). Para ver ejemplos de JSON, consulte la sección [Uso de la autenticación de clave pública de SSH](#use-ssh-public-key-authentication). |Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Para más información, consulte la sección [Requisitos previos](#prerequisites). Si no se especifica el entorno de ejecución de integración, el servicio usa la instancia predeterminada de Azure Integration Runtime. |No |

### <a name="use-basic-authentication"></a>Uso de la autenticación básica

Para usar la autenticación básica, establezca la propiedad *authenticationType* en *Básica* y especifique las siguientes propiedades, además de las genéricas, del conector de SFTP que se introdujeron en la sección anterior:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| userName | Usuario que tiene acceso al servidor SFTP. |Sí |
| password | La contraseña para el usuario (userName). Marque este campo como SecureString para almacenarlo de forma segura en la factoría de datos o bien [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |

**Ejemplo**:

```json
{
    "name": "SftpLinkedService",
    "type": "linkedservices",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<sftp server>",
            "port": 22,
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "authenticationType": "Basic",
            "userName": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-ssh-public-key-authentication"></a>Uso de la autenticación de clave pública SSH

Para usar la autenticación de clave pública SSH, establezca la propiedad "authenticationType" en **SshPublicKey** y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| userName | Usuario que tiene acceso al servidor SFTP. |Sí |
| privateKeyPath | Especifique una ruta de acceso absoluta al archivo de clave privada al que el entorno de ejecución de integración pueda acceder. Esto solo se aplica cuando se especifica un tipo autohospedado de un entorno de ejecución de integración en "connectVia". | Especifique `privateKeyPath` o `privateKeyContent`.  |
| privateKeyContent | Contenido de clave privada SSH codificada en Base64. La clave privada SSH debe tener el formato OpenSSH. Marque este campo como SecureString para almacenarlo de forma segura en la factoría de datos o bien [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Especifique `privateKeyPath` o `privateKeyContent`. |
| passPhrase | Especifique la contraseña o la frase de contraseña para descifrar la clave privada si el archivo de clave o el contenido de clave está protegido por una frase de contraseña. Marque este campo como SecureString para almacenarlo de forma segura en la factoría de datos o bien [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí, si el archivo de clave privada o el contenido de clave está protegido por una frase de contraseña. |

> [!NOTE]
> El conector de SFTP solo admite una clave OpenSSH de RSA/DSA. Asegúrese de que el contenido del archivo de claves comienza por "-----BEGIN [RSA/DSA] PRIVATE KEY-----". Si el archivo de clave privada es un archivo con formato PPK, use la herramienta PuTTY para convertir del formato PPK a OpenSSH. 

**Ejemplo 1: Autenticación de SshPublicKey mediante el valor de filePath de clave privada**

```json
{
    "name": "SftpLinkedService",
    "type": "Linkedservices",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<sftp server>",
            "port": 22,
            "skipHostKeyValidation": true,
            "authenticationType": "SshPublicKey",
            "userName": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": {
                "type": "SecureString",
                "value": "<pass phrase>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo 2: Autenticación de SshPublicKey mediante contenido de clave privada**

```json
{
    "name": "SftpLinkedService",
    "type": "Linkedservices",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<sftp server>",
            "port": 22,
            "skipHostKeyValidation": true,
            "authenticationType": "SshPublicKey",
            "userName": "<username>",
            "privateKeyContent": {
                "type": "SecureString",
                "value": "<base64 string of the private key content>"
            },
            "passPhrase": {
                "type": "SecureString",
                "value": "<pass phrase>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Para obtener una lista completa de las secciones y las propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Se admiten las siguientes propiedades para SFTP en el valor `location` del conjunto de datos basado en formato:

| Propiedad   | Descripción                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad *type* de `location` en el conjunto de datos debe establecerse en *SftpLocation*. | Sí      |
| folderPath | Ruta de acceso a la carpeta. Si quiere usar un carácter comodín para filtrar la carpeta, omita este valor y especifique la ruta de acceso en la configuración del origen de actividad. | No       |
| fileName   | Nombre de archivo en la propiedad folderPath especificada. Si quiere usar un carácter comodín para filtrar archivos, omita este valor y especifique el nombre de archivo en la configuración del origen de actividad. | No       |

**Ejemplo**:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<SFTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "SftpLocation",
                "folderPath": "root/folder/subfolder"
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

Para obtener una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que el origen SFTP admite.

### <a name="sftp-as-source"></a>SFTP como origen

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Se admiten las siguientes propiedades para SFTP en el valor `storeSettings` del origen de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propiedad *type* en `storeSettings` debe establecerse como *SftpReadSettings*. | Sí                                           |
| ***Buscar los archivos para copiar*** |  |  |
| OPCIÓN 1: ruta de acceso estática<br> | Copia de la ruta de acceso de archivo o carpeta especificada en el conjunto de datos. Si quiere copiar todos los archivos de una carpeta, especifique también `wildcardFileName` como `*`. |  |
| OPCIÓN 2: carácter comodín<br>- wildcardFolderPath | Ruta de acceso de carpeta con caracteres comodín para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son `*` (coincide con cero o varios caracteres) y `?` (coincide con cero o con un carácter individual); use `^` para el escape si el nombre real de la carpeta contiene un carácter comodín o este carácter de escape dentro. <br>Para ver más ejemplos, consulte [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | No                                            |
| OPCIÓN 2: carácter comodín<br>- wildcardFileName | Nombre de archivo con caracteres comodín en la propiedad folderPath o wildcardFolderPath especificada para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son `*` (coincide con cero o varios caracteres) y `?` (coincide con cero o con un carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro.  Para ver más ejemplos, consulte [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí |
| OPCIÓN 3: una lista de archivos<br>- fileListPath | Indica que se copie un conjunto de archivos especificado. Apunte a un archivo de texto que incluya una lista de archivos que desee copiar (un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos).<br/>Al usar esta opción, no especifique un nombre de archivo en el conjunto de datos. Para ver más ejemplos, consulte [Ejemplos de lista de archivos](#file-list-examples). |No |
| ***Configuración adicional*** |  | |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Cuando recursive se establece en true y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. <br>Los valores permitidos son: *True* (valor predeterminado) y *False*.<br>Esta propiedad no se aplica al configurar `fileListPath`. |No |
| deleteFilesAfterCompletion | Indica si los archivos binarios se eliminarán del almacén de origen después de moverse correctamente al almacén de destino. Cada archivo se elimina individualmente, de modo que cuando se produzca un error en la actividad de copia, algunos archivos ya se habrán copiado al destino y se habrán eliminado del origen, mientras que otros seguirán aún en el almacén de origen. <br/>Esta propiedad solo es válida en el escenario de copia de archivos binarios, en el que los almacenes de origen de datos son Blob, ADLS Gen1, ADLS Gen2, S3, Google Cloud Storage, File, Azure File, SFTP o FTP. El valor predeterminado es false. |No |
| modifiedDatetimeStart    | Los archivos se filtran en función del atributo *Last Modified*. <br>Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de `modifiedDatetimeStart` a `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato *2018-12-01T05:00:00Z*. <br> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.<br/>Esta propiedad no se aplica al configurar `fileListPath`. | No                                            |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | No                                            |
| enablePartitionDiscovery | En el caso de archivos con particiones, especifique si quiere analizar las particiones de la ruta de acceso del archivo y agregarlas como columnas de origen adicionales.<br/>Los valores permitidos son **false** (valor predeterminado) y **true**. | No                                            |
| partitionRootPath | Cuando esté habilitada la detección de particiones, especifique la ruta de acceso raíz absoluta para poder leer las carpetas con particiones como columnas de datos.<br/><br/>Si no se especifica, de forma predeterminada,<br/>- Cuando se usa la ruta de acceso de archivo en un conjunto de datos o una lista de archivos del origen, la ruta de acceso raíz de la partición es la ruta de acceso configurada en el conjunto de datos.<br/>- Cuando se usa el filtro de carpeta con caracteres comodín, la ruta de acceso raíz de la partición es la subruta antes del primer carácter comodín.<br/><br/>Por ejemplo, supongamos que configura la ruta de acceso en el conjunto de datos como "root/folder/year=2020/month=08/day=27":<br/>- Si especifica la ruta de acceso raíz de la partición como "root/folder/year=2020", la actividad de copia generará dos columnas más, `month` y `day`, con el valor "08" y "27", respectivamente, además de las columnas de los archivos.<br/>- Si no se especifica la ruta de acceso raíz de la partición, no se generará ninguna columna adicional. | No                                            |
| maxConcurrentConnections | Número de conexiones que se pueden conectar al almacén de almacenamiento de forma simultánea. Especifique un valor solamente cuando desee limitar la conexión simultánea al almacén de datos. | No                                            |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSFTP",
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
                    "type": "SftpReadSettings",
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

### <a name="sftp-as-a-sink"></a>SFTP como un receptor

[!INCLUDE [data-factory-v2-file-sink-formats](../../includes/data-factory-v2-file-sink-formats.md)]

Las propiedades siguientes se admiten para SFTP en el valor `storeSettings` del receptor de la copia con formato:

| Propiedad                 | Descripción                                                  | Obligatorio |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propiedad *type* en `storeSettings` se debe establecer en *SftpWriteSettings*. | Sí      |
| copyBehavior             | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen a la carpeta de origen es idéntica que la ruta de acceso relativa del archivo de destino a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del archivo, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. | No       |
| maxConcurrentConnections | Número de conexiones que se pueden conectar al almacén de almacenamiento de forma simultánea. Especifique un valor solamente cuando desee limitar la conexión simultánea al almacén de datos. | No       |
| useTempFileRename | Indique si desea cargar en archivos temporales y cambiarles el nombre, o bien escribir directamente en la ubicación de la carpeta o archivo de destino. De forma predeterminada, Azure Data Factory primero escribe en archivos temporales y, a continuación, los cambia de nombre cuando finaliza la carga. Esta secuencia ayuda a (1) evitar conflictos que podrían dañar el archivo si tiene otros procesos que escriben en el mismo archivo y (2) asegurarse de que la versión original del archivo existe durante la transferencia. Si el servidor SFTP no admite la operación de cambio de nombre, deshabilite esta opción y asegúrese de que no tiene una escritura simultánea en el archivo de destino. Para más información, consulte la sugerencia de solución de problemas al final de esta tabla. | No. El valor predeterminado es *true*. |
| operationTimeout | Tiempo de espera máximo de las solicitudes de escritura en el servidor SFTP. El valor predeterminado es 60 min (01:00:00).|No |

>[!TIP]
>Si recibe el error "UserErrorSftpPathNotFound", "UserErrorSftpPermissionDenied" o "SftpOperationFail" cuando escribe datos en SFTP, y el usuario de SFTP que usa *tiene* los permisos adecuados, compruebe si funciona la operación de cambio de nombre del archivo de compatibilidad del servidor SFTP. Si no es así, deshabilite la opción **Cargar con archivo temporal** (`useTempFileRename`) e inténtelo de nuevo. Para más información sobre esta propiedad, consulte la tabla anterior. Si usa una instancia de entorno de ejecución de integración autohospedado para la actividad de copia, asegúrese de usar la versión 4.6 o posterior.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToSFTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
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
                "type": "<source type>"
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings":{
                    "type": "SftpWriteSettings",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

En esta sección se describe el comportamiento resultante del uso de filtros comodín con rutas de acceso a carpetas y nombres de archivo.

| folderPath | fileName | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en **negrita**)|
|:--- |:--- |:--- |:--- |
| `Folder*` | (vacío, usar el valor predeterminado) | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | (vacío, usar el valor predeterminado) | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="file-list-examples"></a>Ejemplos de lista de archivos

En esta tabla se describe el comportamiento resultante de usar una ruta de acceso de la lista de archivos en el origen de la actividad de copia. Se supone que tiene la siguiente estructura de carpetas de origen y quiere copiar los archivos que están en negrita:

| Estructura de origen de ejemplo                                      | Contenido de FileListToCopy.txt                             | Configuración de Azure Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| root<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Metadatos<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **El conjunto de datos:**<br>- Ruta de acceso de la carpeta: `root/FolderA`<br><br>**En el origen de la actividad de copia:**<br>- Ruta de acceso de la lista de archivos: `root/Metadata/FileListToCopy.txt` <br><br>La ruta de acceso de la lista de archivos apunta a un archivo de texto en el mismo almacén de datos que incluye una lista de archivos que se quiere copiar (un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos). |

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para más información sobre las propiedades de la actividad de búsqueda, consulte [Actividad de búsqueda en Azure Data Factory](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para más información sobre las propiedades de la actividad GetMetadata, consulte [Actividad GetMetadata en Azure Data Factory](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propiedades de la actividad de eliminación

Para más información sobre las propiedades de la actividad de eliminación, consulte [Actividad de eliminación en Azure Data Factory](delete-activity.md).

## <a name="legacy-models"></a>Modelos heredados

>[!NOTE]
>Estos modelos siguen siendo compatibles tal cuales con versiones anteriores. Se recomienda usar el nuevo modelo descrito anteriormente, ya que la UI de creación de Azure Data Factory ha cambiado para generar el nuevo modelo.

### <a name="legacy-dataset-model"></a>Modelo de conjunto de datos heredado

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad *type* del conjunto de datos debe establecerse en *FileShare*. |Sí |
| folderPath | Ruta de acceso a la carpeta. Se admite un filtro con caracteres comodín. Los caracteres comodín permitidos son `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter); use `^` como carácter de escape si el nombre de archivo real tiene un carácter comodín o este carácter de escape dentro. <br/><br/>Ejemplos: rootfolder/subfolder/ver más en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). |Sí |
| fileName |  **Filtro de nombre o de carácter comodín** para los archivos de la ruta "folderPath" especificada. Si no especifica ningún valor para esta propiedad, el conjunto de datos apunta a todos los archivos de la carpeta. <br/><br/>Para filtrar, los caracteres comodín permitidos son `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"fileName": "*.csv"`<br/>- Ejemplo 2: `"fileName": "???20180427.txt"`<br/>Use `^` como escape si el nombre real de la carpeta contiene un carácter comodín o este carácter de escape. |No |
| modifiedDatetimeStart | Los archivos se filtran en función del atributo *Last Modified*. Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de `modifiedDatetimeStart` a `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato *2018-12-01T05:00:00Z*. <br/><br/> El rendimiento general del movimiento de datos se verá afectado si habilita este valor cuando desee filtrar grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| No |
| modifiedDatetimeEnd | Los archivos se filtran en función del atributo *Last Modified*. Los archivos se seleccionan si la hora de su última modificación está dentro del intervalo de `modifiedDatetimeStart` a `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato *2018-12-01T05:00:00Z*. <br/><br/> El rendimiento general del movimiento de datos se verá afectado si habilita este valor cuando desee filtrar grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| No |
| format | Si quiere copiar los archivos tal cual entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si quiere analizar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: *TextFormat*, *JsonFormat*, *AvroFormat*, *OrcFormat* y *ParquetFormat*. Establezca la propiedad *type* de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Formato Json](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Formato Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Formato Orc](supported-file-formats-and-compression-codecs-legacy.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Los tipos admitidos son *GZip*, *Deflate*, *BZip2* y *ZipDeflate*.<br/>Niveles admitidos son *Optimal* y *Fastest*. |No |

>[!TIP]
>Para copiar todos los archivos en una carpeta, especifique solo *folderPath*.<br>Para copiar un único archivo con un nombre determinado, especifique *folderPath* con el elemento de carpeta y *fileName* con el nombre de archivo.<br>Para copiar un subconjunto de archivos en una carpeta, especifique *folderPath* con el elemento de carpeta y *fileName* con el filtro de comodín.

>[!NOTE]
>Si estaba usando la propiedad *fileFilter* para el filtro de archivos, todavía se admite tal cual, aunque se recomienda usar la nueva funcionalidad de filtro agregada a *fileName* de ahora en adelante.

**Ejemplo**:

```json
{
    "name": "SFTPDataset",
    "type": "Datasets",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<SFTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "folder/subfolder/",
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
| type | La propiedad *type* del origen de la actividad de copia debe establecerse en *FileSystemSource*. |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Si recursive se establece en *true* y el receptor es un almacén basado en archivos, no se copiará ni creará una subcarpeta o carpeta vacía en el receptor.<br/>Los valores permitidos son *true* (valor predeterminado) y *false*. | No |
| maxConcurrentConnections | Número de conexiones que se pueden conectar al almacén de almacenamiento de forma simultánea. Especifique un número solamente cuando desee limitar la conexión simultánea al almacén de datos. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSFTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SFTP input dataset name>",
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
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
