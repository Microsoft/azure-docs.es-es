---
title: Copia de datos de un servidor SFTP mediante Azure Data Factory | Microsoft Docs
description: Obtenga información sobre el conector MySQL de Azure Data Factory que permite copiar los datos desde un servidor SFTP a un almacén de datos admitido como receptor.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 04/29/2019
ms.author: jingwang
ms.openlocfilehash: 1a62e9e8377705af1a70e356f554cfa549c58f20
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233470"
---
# <a name="copy-data-from-sftp-server-using-azure-data-factory"></a>Copia de datos desde un servidor SFTP mediante Azure Data Factory
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [Versión 1](v1/data-factory-sftp-connector.md)
> * [Versión actual](connector-sftp.md)

En este artículo se describe cómo copiar datos desde un servidor SFTP. Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector SFTP es compatible con las siguientes actividades:

- [Actividad de copia](copy-activity-overview.md) con [admite la matriz de origen/receptor](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)

En concreto, este conector SFTP admite las siguientes funcionalidades:

- Copiar datos mediante la autenticación **Básica** o **SshPublicKey**.
- Copiar los archivos tal cual, o bien analizarlos con los [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de SFTP.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado SFTP:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Sftp**. |Sí |
| host | Nombre o dirección IP del servidor SFTP. |Sí |
| port | Puerto en el que escucha el servidor SFTP.<br/>Los valores permitidos son: enteros; el valor predeterminado es **22**. |No |
| skipHostKeyValidation | Especifique si desea omitir la validación de claves de host.<br/>Los valores válidos son **true** y **false** (valor predeterminado).  | No |
| hostKeyFingerprint | Especifique la huella dactilar de la clave de host. | Sí, si "skipHostKeyValidation" está establecido en false.  |
| authenticationType | Especifique el tipo de autenticación.<br/>Los valores permitidos son: **Basic**, **SshPublicKey**. Consulte las secciones [Uso de autenticación básica](#using-basic-authentication) y [Uso de autenticación de clave pública SSH](#using-ssh-public-key-authentication) sobre más propiedades y ejemplos JSON respectivamente. |Sí |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Puede usar los entornos Integration Runtime (autohospedado) (si el almacén de datos se encuentra en una red privada) o Azure Integration Runtime. Si no se especifica, se usará Azure Integration Runtime. |No |

### <a name="using-basic-authentication"></a>Uso de autenticación básica

Para usar la autenticación básica, establezca la propiedad "authenticationType" en **Básica** y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| userName | Usuario que tiene acceso al servidor SFTP. |Sí |
| password | Contraseña para el usuario (nombre de usuario). Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |

**Ejemplo:**

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
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="using-ssh-public-key-authentication"></a>Uso de autenticación de clave pública SSH

Para usar la autenticación de clave pública SSH, establezca la propiedad "authenticationType" en **SshPublicKey** y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| userName | Usuario que tiene acceso al servidor SFTP. |Sí |
| privateKeyPath | Especifique una ruta de acceso absoluta al archivo de clave privada al que pueda acceder Integration Runtime. Solo se aplica cuando se especifica un tipo autohospedado de un entorno de ejecución de integración en "connectVia". | Especifique `privateKeyPath` o `privateKeyContent`.  |
| privateKeyContent | Contenido de clave privada SSH codificada en Base64. La clave privada SSH debe tener el formato OpenSSH. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Especifique `privateKeyPath` o `privateKeyContent`. |
| passPhrase | Especifique la contraseña o la frase de contraseña para descifrar la clave privada si el archivo de clave está protegido por una frase de contraseña. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí, si el archivo de clave privada está protegido por una frase de contraseña. |

> [!NOTE]
> El conector SFTP solo admite la clave OpenSSH de RSA/DSA. Asegúrese de que el contenido del archivo de claves comienza con "-----BEGIN [RSA/DSA] PRIVATE KEY-----". Si el archivo de clave privada es un archivo con formato ppk, use la herramienta Putty para convertir del formato .ppk a OpenSSH. 

**Ejemplo 1: Autenticación de SshPublicKey mediante el valor de filePath de clave privada**

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
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo 2: Autenticación de SshPublicKey mediante contenido de clave privada**

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
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). 

- Para **Parquet y formato de texto delimitado**, consulte [conjunto de datos con formato Parquet y texto delimitado](#parquet-and-delimited-text-format-dataset) sección.
- Para otros formatos como **formato ORC/Avro/JSON/binario**, consulte [otro conjunto de datos de formato](#other-format-dataset) sección.

### <a name="parquet-and-delimited-text-format-dataset"></a>Conjunto de datos con formato parquet y texto delimitado

Para copiar datos desde SFTP en **Parquet o formato de texto delimitado**, consulte [formato Parquet](format-parquet.md) y [formato de texto delimitado](format-delimited-text.md) artículo en el conjunto de datos de formato y compatibles Configuración. Se admiten las siguientes propiedades de SFTP en `location` configuración en el conjunto de datos basado en el formato:

| Propiedad   | DESCRIPCIÓN                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad type en `location` en el conjunto de datos debe establecerse en **SftpLocation**. | Sí      |
| folderPath | La ruta de acceso a la carpeta. Si desea usar el carácter comodín a la carpeta de filtro, omita esta configuración y especificar en la configuración del origen de actividad. | Sin        |
| fileName   | El nombre de archivo en folderPath determinado. Si desea usar el carácter comodín para filtrar los archivos, omita esta configuración y especificar en la configuración del origen de actividad. | Sin        |

> [!NOTE]
> **Recurso compartido de archivos** todavía se admite el tipo de conjunto de datos con formato Parquet/texto que se mencionan en la siguiente sección como-actividad de copia/búsqueda/GetMetadata para compatibilidad con versiones anteriores de. Se sugieren para usar este nuevo modelo a partir de ahora, y ha cambiado la interfaz de usuario de creación de ADF para generar estos nuevos tipos.

**Ejemplo:**

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

### <a name="other-format-dataset"></a>Otro conjunto de datos de formato

Para copiar datos desde SFTP en **formato ORC/Avro/JSON/binario**, se admiten las siguientes propiedades:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **FileShare** |Sí |
| folderPath | Ruta de acceso a la carpeta. Se admite el filtro de comodín, los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real del archivo tiene un carácter comodín o este carácter de escape dentro. <br/><br/>Ejemplos: rootfolder/subfolder/ver más en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). |Sí |
| fileName |  **Filtro de nombre o de comodín** para los archivos de la ruta "folderPath" especificada. Si no especifica ningún valor para esta propiedad, el conjunto de datos apunta a todos los archivos de la carpeta. <br/><br/>Para filtrar, los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"fileName": "*.csv"`<br/>- Ejemplo 2: `"fileName": "???20180427.txt"`<br/>Use `^` como escape si el nombre real de la carpeta contiene un carácter comodín o este carácter de escape. |No |
| modifiedDatetimeStart | Filtro de archivos basado en el atributo: Última modificación. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que el rendimiento general del movimiento de datos se verán afectado por si habilita a esta configuración cuando desee del filtro de archivos de grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL que significa que no se aplicará ningún filtro de atributo de archivo para el conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| Sin  |
| modifiedDatetimeEnd | Filtro de archivos basado en el atributo: Última modificación. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que el rendimiento general del movimiento de datos se verán afectado por si habilita a esta configuración cuando desee del filtro de archivos de grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL que significa que no se aplicará ningún filtro de atributo de archivo para el conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| Sin  |
| format | Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si quiere analizar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs.md#text-format), [Formato Json](supported-file-formats-and-compression-codecs.md#json-format), [Formato Avro](supported-file-formats-and-compression-codecs.md#avro-format), [Formato Orc](supported-file-formats-and-compression-codecs.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md#compression-support).<br/>Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**.<br/>Estos son los niveles que se admiten: **Optimal** y **Fastest**. |Sin  |

>[!TIP]
>Para copiar todos los archivos en una carpeta, especifique solo **folderPath**.<br>Para copiar un único archivo con un nombre determinado, especifique **folderPath** con el elemento de carpeta y **fileName** con el nombre de archivo.<br>Para copiar un subconjunto de archivos en una carpeta, especifique **folderPath** con el elemento de carpeta y **fileName** con el filtro de comodín.

>[!NOTE]
>Si estaba usando la propiedad "fileFilter" para el filtro de archivos, esta todavía se admite como está, aunque se le sugiere que use la nueva funcionalidad de filtro agregada a "fileName" de ahora en adelante.

**Ejemplo:**

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

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen SFTP.

### <a name="sftp-as-source"></a>SFTP como origen

- Para copiar desde **Parquet y formato de texto delimitado**, consulte [Parquet y origen del formato de texto delimitado](#parquet-and-delimited-text-format-source) sección.
- Para copiar desde otros formatos como **formato ORC/Avro/JSON/binario**, consulte [otro origen de formato](#other-format-source) sección.

#### <a name="parquet-and-delimited-text-format-source"></a>Parquet y origen del formato de texto delimitado

Para copiar datos desde SFTP en **Parquet o formato de texto delimitado**, consulte [formato Parquet](format-parquet.md) y [formato de texto delimitado](format-delimited-text.md) artículo sobre el origen de la actividad de copia basada en el formato y configuraciones admitidas. Se admiten las siguientes propiedades de SFTP en `storeSettings` configuración en el origen de copia basada en el formato:

| Propiedad                 | DESCRIPCIÓN                                                  | Obligatorio                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propiedad type en `storeSettings` debe establecerse en **SftpReadSetting**. | Sí                                           |
| recursive                | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando recursive se establece en true y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. Los valores permitidos son: **True** (valor predeterminado) y **False**. | Sin                                             |
| wildcardFolderPath       | La ruta de acceso de carpeta con caracteres comodín para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro. <br>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sin                                             |
| wildcardFileName         | El nombre de archivo con caracteres comodín en el folderPath/wildcardFolderPath determinado para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro.  Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí si `fileName` no se especifica en el conjunto de datos |
| modifiedDatetimeStart    | Filtro de archivos basado en el atributo: Última modificación. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br> Las propiedades pueden ser NULL, lo que significa que no se aplicará ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora. | Sin                                             |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | Sin                                             |
| maxConcurrentConnections | El número de conexiones para conectarse al almacén de almacenamiento al mismo tiempo. Especifique solo cuando desee limitar las conexiones simultáneas al almacén de datos. | No                                            |

> [!NOTE]
> Para Parquet/texto delimitado con **FileSystemSource** todavía se admite como origen de la actividad de copia tipo mencionado en la siguiente sección: sirve para la compatibilidad con versiones anteriores. Se sugieren para usar este nuevo modelo a partir de ahora, y ha cambiado la interfaz de usuario de creación de ADF para generar estos nuevos tipos.

**Ejemplo:**

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
                    "type": "DelimitedTextReadSetting",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "SftpReadSetting",
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

#### <a name="other-format-source"></a>Otro origen de formato

Para copiar datos desde SFTP en **formato ORC/Avro/JSON/binario**, se admiten las siguientes propiedades en la actividad de copia **origen** sección:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **FileSystemSource** |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando recursive se establezca en true y el receptor sea un almacén basado en archivos, la carpeta o subcarpeta vacías no se copiarán ni crearán en el receptor.<br/>Los valores permitidos son: **True** (valor predeterminado) y **False** | Sin  |
| maxConcurrentConnections | El número de conexiones para conectarse al almacén de almacenamiento al mismo tiempo. Especifique solo cuando desee limitar las conexiones simultáneas al almacén de datos. | Sin  |

**Ejemplo:**

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

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

Esta sección describe el comportamiento resultante de la ruta de acceso de la carpeta y el nombre de archivo con los filtros de carácter comodín.

| folderPath | fileName | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en **negrita**)|
|:--- |:--- |:--- |:--- |
| `Folder*` | (vacío, usar el valor predeterminado) | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | (vacío, usar el valor predeterminado) | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md##supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
