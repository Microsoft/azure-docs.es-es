---
title: Formato Excel en Azure Data Factory
description: En este tema se describe cómo tratar el formato Excel en Azure Data Factory.
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/08/2020
ms.author: jingwang
ms.openlocfilehash: a937548c9318d98e8832720706626b74167d32d9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87044399"
---
# <a name="excel-format-in-azure-data-factory"></a>Formato Excel en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Siga este artículo cuando desee **analizar los archivos de Excel**. Azure Data Factory admite ".xls" and ".xlsx".

El formato Excel es compatible con los conectores siguientes: [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) y [SFTP](connector-sftp.md). Se admite como origen, pero no como receptor.

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de Excel.

| Propiedad         | Descripción                                                  | Obligatorio |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propiedad type del conjunto de datos debe establecerse en **Excel**.   | Sí      |
| ubicación         | Configuración de ubicación de los archivos. Cada conector basado en archivos tiene su propio tipo de ubicación y propiedades compatibles en `location`. | Sí      |
| sheetName        | Nombre de la hoja de cálculo de Excel para leer los datos.                       | Sí      |
| range            | El rango de celdas de la hoja de cálculo especificada para buscar los datos selectivos, por ejemplo, `A3:H5` (una tabla de A3 a H5), `A3` (una tabla a partir de la celda A3), `A3:A3` (una sola celda). Si no se especifica, el ADF lee en toda la hoja de cálculo como una tabla. | No       |
| firstRowAsHeader | Especifica si se debe tratar la primera fila del rango o la hoja de cálculo determinados como una línea de encabezado con nombres de columnas.<br>Los valores permitidos son **true** y **false** (predeterminado). | No       |
| nullValue        | Especifica la representación de cadena del valor null. <br>El valor predeterminado es una **cadena vacía**. | No       |
| compression | Grupo de propiedades para configurar la compresión de archivo. Configure esta sección si desea realizar la compresión o descompresión durante la ejecución de la actividad. | No |
| type<br/>(*en `compression`* ) | El códec de compresión usado para leer y escribir archivos JSON. <br>Los valores permitidos son **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **snappy** o **lz4** para usar al guardar el archivo. La opción predeterminada no se comprime.<br>**Tenga en cuenta** que actualmente la actividad de copia no admite "snappy" ni "lz4", y que el flujo de datos de asignación no admite "ZipDeflate".<br>**Tenga en cuenta** que cuando utilice la actividad de copia para descomprimir archivos **ZipDeflate** y escribir en el almacén de datos de receptores basado en archivos, los archivos se extraerán a la carpeta `<path specified in dataset>/<folder named as source zip file>/`. | No.  |
| level<br/>(*en `compression`* ) | La razón de compresión. <br>Los valores permitidos son **Optimal** o **Fastest**.<br>- **Fastest:** la operación de compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante no se comprime de forma óptima.<br>- **Optimal**: la operación de compresión se debe comprimir óptimamente, incluso si tarda más tiempo en completarse. Para más información, consulte el tema [Nivel de compresión](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) . | No       |

A continuación, se muestra un ejemplo de un conjunto de datos de Excel en Azure Blob Storage:

```json
{
    "name": "ExcelDataset",
    "properties": {
        "type": "Excel",
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
            "sheetName": "MyWorksheet",
            "range": "A3:H5",
            "firstRowAsHeader": true
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el origen de Excel.

### <a name="excel-as-source"></a>Excel como origen 

En la sección ***\*source\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en **ExcelSource**. | Sí      |
| storeSettings | Un grupo de propiedades sobre cómo leer datos de un almacén de datos. Cada conector basado en archivos tiene su propia configuración de lectura admitida en `storeSettings`. | No       |

```json
"activities": [
    {
        "name": "CopyFromExcel",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "ExcelSource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            ...
        }
        ...
    }
]
```

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

En los flujos de datos de asignación, puede leer en formato Excel en los siguientes almacenes de datos: [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties). Puede apuntar a archivos de Excel mediante un conjunto de datos de Excel o mediante un [conjunto de datos alineado](data-flow-source.md#inline-datasets).

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que admite un origen Excel. Puede editar estas propiedades en la pestaña **Opciones del origen**. Al usar un conjunto de valores alineados, verá configuraciones de archivo adicionales que son iguales a las propiedades descritas en la sección [Propiedades del conjunto de datos](#dataset-properties).

| Nombre                      | Descripción                                                  | Obligatorio | Valores permitidos                                            | Propiedad de script de flujo de datos         |
| ------------------------- | ------------------------------------------------------------ | -------- | --------------------------------------------------------- | --------------------------------- |
| Rutas de acceso comodín           | Se procesarán todos los archivos que coincidan con la ruta de acceso comodín. Reemplaza a la carpeta y la ruta de acceso del archivo establecidas en el conjunto de datos. | no       | String[]                                                  | wildcardPaths                     |
| Ruta de acceso raíz de la partición       | En el caso de datos de archivos con particiones, puede especificar una ruta de acceso raíz de la partición para leer las carpetas con particiones como columnas. | no       | String                                                    | partitionRootPath                 |
| Lista de archivos             | Si el origen apunta a un archivo de texto que enumera los archivos que se van a procesar. | no       | `true` o `false`                                         | fileList                          |
| Columna para almacenar el nombre de archivo | Se crea una nueva columna con el nombre y la ruta de acceso del archivo de origen.       | no       | String                                                    | rowUrlColumn                      |
| Después de finalizar          | Se eliminan o mueven los archivos después del procesamiento. La ruta de acceso del archivo comienza en la raíz del contenedor. | no       | Borrar: `true` o `false` <br> Mover: `['<from>', '<to>']` | purgeFiles <br> moveFiles         |
| Filtrar por última modificación   | Elija si desea filtrar los archivos en función de cuándo se modificaron por última vez. | no       | Timestamp                                                 | modifiedAfter <br> modifiedBefore |

### <a name="source-example"></a>Ejemplo de origen

En la imagen siguiente se presenta un ejemplo de una configuración de origen Excel en flujos de datos de asignación mediante un modo de conjunto de datos.

![Origen de Excel](media/data-flow/excel-source.png)

El script de flujo de datos asociado es:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    wildcardPaths:['*.xls']) ~> ExcelSource
```

Si usa un conjunto de datos alineado, verá las siguientes opciones de origen en el flujo de datos de asignación.

![Conjunto de datos alineado del origen de Excel](media/data-flow/excel-source-inline-dataset.png)

El script de flujo de datos asociado es:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    format: 'excel',
    fileSystem: 'container',
    folderPath: 'path',
    fileName: 'sample.xls',
    sheetName: 'worksheet',
    firstRowAsHeader: true) ~> ExcelSourceInlineDataset
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
