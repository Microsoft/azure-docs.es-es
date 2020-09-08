---
title: Formato de texto delimitado en Azure Data Factory
description: En este tema se describe cómo tratar con el formato de texto delimitado en Azure Data Factory.
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/10/2020
ms.author: jingwang
ms.openlocfilehash: 81fdb404b99dc5456e9e544b6ff45dff73a7940d
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042843"
---
# <a name="delimited-text-format-in-azure-data-factory"></a>Formato de texto delimitado en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Siga este artículo cuando quiera **analizar los archivos de texto delimitado o escribir los datos en formato de texto delimitado**. 

El formato de texto delimitado se admite para los conectores siguientes: [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) y [SFTP](connector-sftp.md).

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de texto delimitado.

| Propiedad         | Descripción                                                  | Obligatorio |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propiedad type del conjunto de datos debe establecerse en **DelimitedText**. | Sí      |
| ubicación         | Configuración de ubicación de los archivos. Cada conector basado en archivos tiene su propio tipo de ubicación y propiedades compatibles en `location`.  | Sí      |
| columnDelimiter  | Los caracteres usados para separar las columnas en un archivo. <br>El valor predeterminado es **coma `,`** . Cuando el delimitador de columna se define como cadena vacía (es decir, ningún delimitador), toda la línea se toma como una sola columna.<br>Actualmente, solo se admite el delimitador de columna como cadena vacía o de varios caracteres para el flujo de datos de asignación, pero no para la actividad de copia.  | No       |
| rowDelimiter     | El carácter único o "\r\n" usado para separar las filas en un archivo. <br>El valor predeterminado es cualquiera de los siguientes valores **en lectura: ["\r\n", "\r", "\n"]** , y **"\n" o "\r\n" en escritura** mediante el flujo de datos de asignación y la actividad de copia, respectivamente. <br>Cuando el delimitador de fila se establece en ningún delimitador (cadena vacía), también debe establecerse el delimitador de columna como ningún delimitador (cadena vacía), lo que significa que se trata todo el contenido como un valor único.<br>Actualmente, solo se admite el delimitador de fila como cadena vacía para el flujo de datos de asignación, pero no para la actividad de copia. | No       |
| quoteChar        | El carácter único para entrecomillar los valores de columna si contiene el delimitador de columna. <br>El valor predeterminado es **comillas dobles** `"`. <br>Para el flujo de datos de asignación, `quoteChar` no puede ser una cadena vacía. <br>Para la actividad de copia, cuando `quoteChar` se define como una cadena vacía, significa que no hay ningún carácter de comillas y el valor de la columna no está entre comillas, y `escapeChar` se usa para escapar el delimitador de columna y a sí mismo. | No       |
| escapeChar       | El carácter único para escapar las comillas dentro de un valor entre comillas.<br>El valor predeterminado es **barra diagonal inversa`\`** . <br>Para el flujo de datos de asignación, `escapeChar` no puede ser una cadena vacía. <br/>Para la actividad de copia, cuando `escapeChar` se define como una cadena vacía, `quoteChar` también debe establecerse como una cadena vacía, en cuyo caso, asegúrese de que ninguno de los valores de columna contenga delimitadores. | No       |
| firstRowAsHeader | Especifica si se debe tratar o convertir la primera fila como una línea de encabezado con nombres de columnas.<br>Los valores permitidos son **true** y **false** (predeterminado).<br>Si la primera fila como encabezado es falsa, tenga en cuenta que la vista previa de los datos de la interfaz de usuario y la salida de la actividad de búsqueda generan automáticamente los nombres de columna como Prop_ {n} (a partir de 0) y la actividad de copia requiere una [asignación explícita](copy-activity-schema-and-type-mapping.md#explicit-mapping) del origen al receptor y busca las columnas por ordinal (a partir de 1) y flujo de datos de asignación, y busca las columnas con el nombre Column_{n} (a partir de 1).  | No       |
| nullValue        | Especifica la representación de cadena del valor null. <br>El valor predeterminado es una **cadena vacía**. | No       |
| encodingName     | El tipo de codificación usado para leer y escribir archivos de prueba. <br>Los valores permitidos son los siguientes: "UTF-8", "UTF-16", "UTF-16BE", "UTF-32", "UTF-32BE", "US-ASCII", “UTF-7”, "BIG5", "EUC-JP", "EUC-KR", "GB2312", "GB18030", "JOHAB", "SHIFT-JIS", "CP875", "CP866", "IBM00858", "IBM037", "IBM273", "IBM437", "IBM500", "IBM737", "IBM775", "IBM850", "IBM852", "IBM855", "IBM857", "IBM860", "IBM861", "IBM863", "IBM864", "IBM865", "IBM869", "IBM870", "IBM01140", "IBM01141", "IBM01142", "IBM01143", "IBM01144", "IBM01145", "IBM01146", "IBM01147", "IBM01148", "IBM01149", "ISO-2022-JP", "ISO-2022-KR", "ISO-8859-1", "ISO-8859-2", "ISO-8859-3", "ISO-8859-4", "ISO-8859-5", "ISO-8859-6", "ISO-8859-7", "ISO-8859-8", "ISO-8859-9", "ISO-8859-13", "ISO-8859-15", "WINDOWS-874", "WINDOWS-1250", "WINDOWS-1251", "WINDOWS-1252", "WINDOWS-1253", "WINDOWS-1254", "WINDOWS-1255", "WINDOWS-1256", "WINDOWS-1257", "WINDOWS-1258”.<br>Tenga en cuenta que el flujo de datos de asignación no admite la codificación UTF-7. | No       |
| compressionCodec | El códec de compresión usado para leer y escribir archivos de texto. <br>Los valores permitidos son **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **snappy** o **lz4** La opción predeterminada no se comprime. <br>**Tenga en cuenta** que actualmente la actividad de copia no admite "snappy" ni "lz4", y que el flujo de datos de asignación no admite "ZipDeflate". <br>**Tenga en cuenta** que, al usar la actividad de copia para descomprimir archivos **ZipDeflate** y escribir en el almacén de datos receptor basado en archivos, los archivos se extraen de forma predeterminada en la carpeta: `<path specified in dataset>/<folder named as source zip file>/`. Use `preserveZipFileNameAsFolder` en el [origen de la actividad de copia](#delimited-text-as-source) para controlar si se debe conservar el nombre del archivo ZIP como una estructura de carpetas. | No       |
| compressionLevel | La razón de compresión. <br>Los valores permitidos son **Optimal** o **Fastest**.<br>- **Fastest:** la operación de compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante no se comprime de forma óptima.<br>- **Optimal**: la operación de compresión se debe comprimir óptimamente, incluso si tarda más tiempo en completarse. Para más información, consulte el tema [Nivel de compresión](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) . | No       |

A continuación encontrará un ejemplo de un conjunto de datos de texto delimitado en Azure Blob Storage:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "escapeChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el origen y el receptor de texto delimitado.

### <a name="delimited-text-as-source"></a>Texto delimitado como origen 

En la sección ***\*source\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad       | Descripción                                                  | Obligatorio |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | La propiedad type del origen de la actividad de copia debe establecerse en **DelimitedTextSource**. | Sí      |
| formatSettings | Un grupo de propiedades. Consulte la tabla **Configuración de lectura de texto delimitado** a continuación. | No       |
| storeSettings  | Un grupo de propiedades sobre cómo leer datos de un almacén de datos. Cada conector basado en archivos tiene su propia configuración de lectura admitida en `storeSettings`. | No       |

**Configuración de lectura de texto delimitado** admitida en `formatSettings`:

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type de formatSettings debe establecerse en **DelimitedTextReadSettings**. | Sí      |
| skipLineCount | Indica el número de filas **no vacías** que se omitirán al leer datos de archivos de entrada. <br>Si se especifican ambos valores skipLineCount y firstRowAsHeader, las líneas se omiten primero y, luego, la información del encabezado se lee a partir del archivo de entrada. | No       |
| compressionProperties | Un grupo de propiedades sobre cómo descomprimir datos para un códec de compresión determinado. | No       |
| preserveZipFileNameAsFolder<br>(*en `compressionProperties`* ) | Se aplica cuando el conjunto de datos de entrada se configura con compresión **ZipDeflate**. Indica si se debe conservar el nombre del archivo ZIP de origen como estructura de carpetas durante la copia.<br>- Cuando se establece en **true (valor predeterminado)** , Data Factory escribe archivos descomprimidos en `<path specified in dataset>/<folder named as source zip file>/`.<br>- Cuando se establece en **false**, Data Factory escribe los archivos descomprimidos directamente en `<path specified in dataset>`. Asegúrese de que no tenga nombres de archivo duplicados en archivos ZIP de origen diferentes para evitar comportamientos acelerados o inesperados.  | No |

```json
"activities": [
    {
        "name": "CopyFromDelimitedText",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "DelimitedTextSource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                },
                "formatSettings": {
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 3,
                    "compressionProperties": {
                        "type": "ZipDeflateReadSettings",
                        "preserveZipFileNameAsFolder": false
                    }
                }
            },
            ...
        }
        ...
    }
]
```

### <a name="delimited-text-as-sink"></a>Texto delimitado como receptor

En la sección ***\*sink\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad       | Descripción                                                  | Obligatorio |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | La propiedad type del origen de la actividad de copia debe establecerse en **DelimitedTextSink**. | Sí      |
| formatSettings | Un grupo de propiedades. Consulte la tabla **Configuración de escritura de texto delimitado** a continuación. |          |
| storeSettings  | Un grupo de propiedades sobre cómo escribir datos en un almacén de datos. Cada conector basado en archivos tiene su propia configuración de escritura admitida en `storeSettings`.  | No       |

**Configuración de escritura de texto delimitado** admitida en `formatSettings`:

| Propiedad      | Descripción                                                  | Obligatorio                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | La propiedad type de formatSettings debe establecerse en **DelimitedTextWriteSettings**. | Sí                                                   |
| fileExtension | La extensión de archivo que se usa para denominar los archivos de salida, por ejemplo: `.csv`, `.txt`. Debe especificarse cuando no se especifica `fileName` en el conjunto de datos DelimitedText de salida. Cuando el nombre de archivo se configure en el conjunto de resultados de salida, se usará como nombre de archivo receptor y se omitirá la configuración de la extensión de archivo.  | Sí, cuando no se especifica el nombre de archivo en el conjunto de datos de salida |

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

En los flujos de datos de asignación, puede leer y escribir en formato de texto delimitado en los siguientes almacenes de datos: [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties).

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que un origen de texto delimitado admite. Puede editar estas propiedades en la pestaña **Source options** (Opciones de origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Rutas de acceso comodín | Se procesarán todos los archivos que coincidan con la ruta de acceso comodín. Reemplaza a la carpeta y la ruta de acceso del archivo establecidas en el conjunto de datos. | no | String[] | wildcardPaths |
| Ruta de acceso raíz de la partición | En el caso de datos de archivos con particiones, puede especificar una ruta de acceso raíz de la partición para leer las carpetas con particiones como columnas. | no | String | partitionRootPath |
| Lista de archivos | Si el origen apunta a un archivo de texto que enumera los archivos que se van a procesar. | no | `true` o `false` | fileList |
| Filas de varias líneas | El archivo de código fuente contiene filas que abarcan varias líneas. Los valores de varias líneas deben estar entre comillas. | no `true` ni `false` | multiLineRow |
| Columna para almacenar el nombre de archivo | Se crea una nueva columna con el nombre y la ruta de acceso del archivo de origen. | no | String | rowUrlColumn |
| Después de finalizar | Se eliminan o mueven los archivos después del procesamiento. La ruta de acceso del archivo comienza en la raíz del contenedor. | no | Borrar: `true` o `false` <br> Mover: `['<from>', '<to>']` | purgeFiles <br> moveFiles |
| Filtrar por última modificación | Elija si desea filtrar los archivos en función de cuándo se modificaron por última vez. | no | Timestamp | modifiedAfter <br> modifiedBefore |

### <a name="source-example"></a>Ejemplo de origen

En la imagen siguiente se presenta un ejemplo de una configuración de origen de texto delimitado en flujos de datos de asignación.

![Origen DelimitedText](media/data-flow/delimited-text-source.png)

El script de flujo de datos asociado es:

```
source(
    allowSchemaDrift: true,
    validateSchema: false,
    multiLineRow: true,
    wildcardPaths:['*.csv']) ~> CSVSource
```

### <a name="sink-properties"></a>Propiedades del receptor

En la tabla siguiente se enumeran las propiedades que un receptor de texto delimitado admite. Puede editar estas propiedades en la pestaña **Configuración**.

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Borrar la carpeta | Si la carpeta de destino se borra antes de escribir. | no | `true` o `false` | truncate |
| Opción de nombre de archivo | El formato de nombre de los datos escritos. De forma predeterminada, un archivo por partición en formato `part-#####-tid-<guid>`. | no | Patrón: String <br> Por partición: String[] <br> Como datos de columna: String <br> Salida en un solo archivo: `['<fileName>']`  | filePattern <br> partitionFileNames <br> rowUrlColumn <br> partitionFileNames |
| Entrecomillar todo | Incluye todos los valores entre comillas. | no | `true` o `false` | quoteAll |

### <a name="sink-example"></a>Ejemplo de receptor

En la imagen siguiente se presenta un ejemplo de una configuración de receptor de texto delimitado en flujos de datos de asignación.

![Receptor DelimitedText](media/data-flow/delimited-text-sink.png)

El script de flujo de datos asociado es:

```
CSVSource sink(allowSchemaDrift: true,
    validateSchema: false,
    truncate: true,
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> CSVSink
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
