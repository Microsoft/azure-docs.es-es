---
title: Formato XML en Azure Data Factory
description: En este tema se describe cómo tratar el formato XML en Azure Data Factory.
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/10/2020
ms.author: jingwang
ms.openlocfilehash: 6c0b03db281a054410b3c4f44e278dbccf32029f
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042690"
---
# <a name="xml-format-in-azure-data-factory"></a>Formato XML en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Siga este artículo cuando desee **analizar los archivos XML**. 

El formato XML es compatible con los conectores siguientes: [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) y [SFTP](connector-sftp.md). Se admite como origen, pero no como receptor.

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de XML.

| Propiedad         | Descripción                                                  | Obligatorio |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propiedad type del conjunto de datos debe establecerse en **Xml**. | Sí      |
| ubicación         | Configuración de ubicación de los archivos. Cada conector basado en archivos tiene su propio tipo de ubicación y propiedades compatibles en `location`. **Vea los detalles en el artículo de conectores -> sección de propiedades del conjunto de datos**. | Sí      |
| encodingName     | El tipo de codificación usado para leer y escribir archivos de prueba. <br>Los valores permitidos son los siguientes: "UTF-8", "UTF-16", "UTF-16BE", "UTF-32", "UTF-32BE", "US-ASCII", "UTF-7", "BIG5", "EUC-JP", "EUC-KR", "GB2312", "GB18030", "JOHAB", "SHIFT-JIS", "CP875", "CP866", "IBM00858", "IBM037", "IBM273", "IBM437", "IBM500", "IBM737", "IBM775", "IBM850", "IBM852", "IBM855", "IBM857", "IBM860", "IBM861", "IBM863", "IBM864", "IBM865", "IBM869", "IBM870", "IBM01140", "IBM01141", "IBM01142", "IBM01143", "IBM01144", "IBM01145", "IBM01146", "IBM01147", "IBM01148", "IBM01149", "ISO-2022-JP", "ISO-2022-KR", "ISO-8859-1", "ISO-8859-2", "ISO-8859-3", "ISO-8859-4", "ISO-8859-5", "ISO-8859-6", "ISO-8859-7", "ISO-8859-8", "ISO-8859-9", "ISO-8859-13", "ISO-8859-15", "WINDOWS-874", "WINDOWS-1250", "WINDOWS-1251", "WINDOWS-1252", "WINDOWS-1253", "WINDOWS-1254", "WINDOWS-1255", "WINDOWS-1256", "WINDOWS-1257", "WINDOWS-1258".| No       |
| nullValue | Especifica la representación de cadena del valor null.<br/>El valor predeterminado es una **cadena vacía**. | No |
| compression | Grupo de propiedades para configurar la compresión de archivo. Configure esta sección si desea realizar la compresión o descompresión durante la ejecución de la actividad. | No |
| type<br>(*en `compression`* ) | El códec de compresión usado para leer y escribir archivos XML. <br>Los valores permitidos son **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **snappy** o **lz4** para usar al guardar el archivo. La opción predeterminada no se comprime.<br>**Tenga en cuenta** que actualmente la actividad de copia no admite "snappy" ni "lz4", y que el flujo de datos de asignación no admite "ZipDeflate".<br>**Tenga en cuenta** que, al usar la actividad de copia para descomprimir archivos **ZipDeflate** y escribir en el almacén de datos receptor basado en archivos, los archivos se extraen de forma predeterminada en la carpeta `<path specified in dataset>/<folder named as source zip file>/`. Use `preserveZipFileNameAsFolder` en el [origen de la actividad de copia](#xml-as-source) para controlar si se debe conservar el nombre del archivo ZIP como una estructura de carpetas. | No.  |
| level<br/>(*en `compression`* ) | La razón de compresión. <br>Los valores permitidos son **Optimal** o **Fastest**.<br>- **Fastest:** la operación de compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante no se comprime de forma óptima.<br>- **Optimal**: la operación de compresión se debe comprimir óptimamente, incluso si tarda más tiempo en completarse. Para más información, consulte el tema [Nivel de compresión](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) . | No       |

A continuación, se muestra un ejemplo de un conjunto de datos de XML en Azure Blob Storage:

```json
{
    "name": "XMLDataset",
    "properties": {
        "type": "Xml",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "ZipDeflate"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el origen de XML.

Obtenga información acerca de cómo asignar datos XML y el formato o el almacén de datos receptor desde la [asignación de esquemas](copy-activity-schema-and-type-mapping.md). Al obtener una vista previa de los archivos XML, los datos se muestran con una jerarquía de JSON y la ruta de acceso de JSON se usa para apuntar a los campos.

### <a name="xml-as-source"></a>XML como origen

En la sección ***\*source\**** de la actividad de copia se admiten las siguientes propiedades. Obtenga más información del [comportamiento del conector XML](#xml-connector-behavior).

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en **XmlSource**. | Sí      |
| formatSettings | Un grupo de propiedades. Consulte la tabla **Configuración de lectura de XML** a continuación. | No       |
| storeSettings | Un grupo de propiedades sobre cómo leer datos de un almacén de datos. Cada conector basado en archivos tiene su propia configuración de lectura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

**Configuración de lectura de XML** admitida en `formatSettings`:

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | El tipo de formatSettings debe establecerse en **XmlReadSettings**. | Sí      |
| validationMode | Especifica si se debe validar el esquema XML.<br>Los valores permitidos son **ninguno** (predeterminado, sin validación), **xsd** (validar con XSD) y **dtd** (validar con DTD). | No |
| namespacePrefixes | Identificador URI del espacio de nombres a la asignación del prefijo, que se usa para asignar un nombre a los campos al analizar el archivo XML.<br/>Si un archivo XML tiene un espacio de nombres y este está habilitado, de forma predeterminada, el nombre del campo es el mismo que en el documento XML.<br>Si hay un elemento definido para el URI del espacio de nombres en esta asignación, el nombre del campo es `prefix:fieldName`. | No |
| compressionProperties | Un grupo de propiedades sobre cómo descomprimir datos para un códec de compresión determinado. | No       |
| preserveZipFileNameAsFolder<br>(*en `compressionProperties`* ) | Se aplica cuando el conjunto de datos de entrada se configura con compresión **ZipDeflate**. Indica si se debe conservar el nombre del archivo ZIP de origen como estructura de carpetas durante la copia.<br>- Cuando se establece en **true (valor predeterminado)** , Data Factory escribe archivos descomprimidos en `<path specified in dataset>/<folder named as source zip file>/`.<br>- Cuando se establece en **false**, Data Factory escribe archivos descomprimidos directamente en `<path specified in dataset>`. Asegúrese de que no tiene nombres de archivo duplicados en archivos ZIP de origen diferentes para evitar carreras o un comportamiento inesperado.  | No |

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

En los flujos de datos de asignación, puede leer y escribir en formato XML en los siguientes almacenes de datos: [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties). Puede apuntar a archivos de XML mediante un conjunto de datos de XML o mediante un [conjunto de datos alineado](data-flow-source.md#inline-datasets).

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que admite un origen XML. Puede editar estas propiedades en la pestaña **Opciones del origen**. Obtenga más información del [comportamiento del conector XML](#xml-connector-behavior). Al usar un conjunto de valores alineados, verá configuraciones de archivo adicionales que son iguales a las propiedades descritas en la sección [Propiedades del conjunto de datos](#dataset-properties). 

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Rutas de acceso comodín | Se procesarán todos los archivos que coincidan con la ruta de acceso comodín. Reemplaza a la carpeta y la ruta de acceso del archivo establecidas en el conjunto de datos. | No | String[] | wildcardPaths |
| Ruta de acceso raíz de la partición | En el caso de datos de archivos con particiones, puede especificar una ruta de acceso raíz de la partición para leer las carpetas con particiones como columnas. | No | String | partitionRootPath |
| Lista de archivos | Si el origen apunta a un archivo de texto que enumera los archivos que se van a procesar. | No | `true` o `false` | fileList |
| Columna para almacenar el nombre de archivo | Se crea una nueva columna con el nombre y la ruta de acceso del archivo de origen. | No | String | rowUrlColumn |
| Después de finalizar | Se eliminan o mueven los archivos después del procesamiento. La ruta de acceso del archivo comienza en la raíz del contenedor. | No | Borrar: `true` o `false` <br> Mover: `['<from>', '<to>']` | purgeFiles <br>moveFiles |
| Filtrar por última modificación | Elija si desea filtrar los archivos en función de cuándo se modificaron por última vez. | No | Timestamp | modifiedAfter <br>modifiedBefore |
| Modo de validación | Especifica si se debe validar el esquema XML. | No | `None` (predeterminado, sin validación)<br>`xsd` (validar con XSD)<br>`dtd` (validar con DTD). | validationMode |
| Espacios de nombres | Indica si habilitar el espacio de nombres al analizar los archivos XML. | No | `true` (predeterminado) o `false` | espacios de nombres |
| Pares de prefijo de espacio de nombres | Identificador URI del espacio de nombres a la asignación del prefijo, que se usa para asignar un nombre a los campos al analizar el archivo XML.<br/>Si un archivo XML tiene un espacio de nombres y este está habilitado, de forma predeterminada, el nombre del campo es el mismo que en el documento XML.<br>Si hay un elemento definido para el URI del espacio de nombres en esta asignación, el nombre del campo es `prefix:fieldName`. | No | Matriz con patrón`['URI1'->'prefix1','URI2'->'prefix2']` | namespacePrefixes |

### <a name="xml-source-script-example"></a>Ejemplo de script de origen XML

En el script siguiente se presenta un ejemplo de una configuración de origen XML en flujos de datos de asignación mediante un modo de conjunto de datos.

```
source(allowSchemaDrift: true,
    validateSchema: false,
    validationMode: 'xsd',
    namespaces: true) ~> XMLSource
```

El script siguiente es un ejemplo de una configuración de origen XML que utiliza el modo de conjunto de datos alineado.

```
source(allowSchemaDrift: true,
    validateSchema: false,
    format: 'xml',
    fileSystem: 'filesystem',
    folderPath: 'folder',
    validationMode: 'xsd',
    namespaces: true) ~> XMLSource
```

## <a name="xml-connector-behavior"></a>Comportamiento del conector XML

Tenga en cuenta lo siguiente al utilizar XML como origen.

- Atributos XML:

    - Los atributos de un elemento se analizan como subcampos del elemento en la jerarquía.
    - El nombre del campo de atributo sigue el patrón `@attributeName`.

- Validación de esquema XML:

    - Puede optar por no validar el esquema o bien validar el esquema mediante XSD o DTD.
    - Al utilizar XSD o DTD para validar archivos XML, se debe hacer referencia a XSD/DTD dentro de los archivos XML a través de la ruta de acceso relativa.

- Control de espacios de nombres:

    - El espacio de nombres se puede deshabilitar cuando se usa el flujo de datos, en cuyo caso los atributos que definen el espacio de nombres se analizarán como atributos normales.
    - Cuando el espacio de nombres está habilitado, los nombres del elemento y los atributos siguen el patrón `namespaceUri,elementName` y `namespaceUri,@attributeName` de forma predeterminada. Puede definir el prefijo del espacio de nombres para cada URI de espacio de nombres en el origen, en cuyo caso los nombres del elemento y los atributos siguen el patrón `definedPrefix:elementName` o `definedPrefix:@attributeName` en su lugar.

- Columna de valor:

    - Si un elemento XML tiene un valor de texto simple y atributos o elementos secundarios, el valor de texto simple se analiza como el valor de una "columna de valor" con el nombre de campo `_value_` integrado. Además, hereda también el espacio de nombres del elemento, si corresponde.

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
