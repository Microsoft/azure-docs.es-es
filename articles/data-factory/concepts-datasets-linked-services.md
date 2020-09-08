---
title: Conjuntos de datos
description: Obtenga información sobre los conjuntos de datos en Data Factory. Los conjuntos de datos representan datos de entrada y salida.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/24/2020
ms.openlocfilehash: 747d780b8f679adf66810bdcdf6e9b263e8d241c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923762"
---
# <a name="datasets-in-azure-data-factory"></a>Conjuntos de datos en Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-create-datasets.md)
> * [Versión actual](concepts-datasets-linked-services.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]


En este artículo se describe qué son los conjuntos de datos, cómo se definen en formato JSON y cómo se usan en canalizaciones de Azure Data Factory.

Si no está familiarizado con Data Factory, consulte [Introducción a Azure Data Factory](introduction.md) para obtener información general.

## <a name="overview"></a>Información general
Una factoría de datos puede tener una o más canalizaciones. Una **canalización** es una agrupación lógica de **actividades** que realizan una tarea. Las actividades de una canalización definen las acciones que se van a realizar en los datos. Ahora, un **conjunto de datos** es una vista con nombre de los datos que simplemente apunta o hace referencia a los datos que desea usar en sus **actividades** como entradas y salidas. Los conjuntos de datos identifican datos en distintos almacenes de datos, como tablas, archivos, carpetas y documentos. Por ejemplo, un conjunto de datos de blob de Azure especifica el contenedor de blobs y la carpeta de Blob Storage de los que la actividad debe leer los datos.

Antes de crear un conjunto de datos, debe crear un [**servicio vinculado**](concepts-linked-services.md) para vincular su almacén de datos a la factoría de datos. Los servicios vinculados son muy similares a las cadenas de conexión que definen la información de conexión necesaria para que Data Factory se conecte a recursos externos. Considérelos de esta forma: el conjunto de datos representa la estructura de los datos dentro de los almacenes de datos vinculados y el servicio vinculado define la conexión al origen de datos. Por ejemplo, un servicio vinculado Azure Storage vincula una cuenta de almacenamiento a la factoría de datos. Un conjunto de datos de blobs de Azure representa el contenedor de blobs y la carpeta dentro de esa cuenta de Azure Storage que contiene los blobs de entrada que se van a procesar.

Este es un escenario de ejemplo. Para copiar datos de Blob Storage en SQL Database, se crean dos servicios vinculados: Azure Blob Storage y Azure SQL Database. Después, creará dos conjuntos de datos: el conjunto de datos de texto delimitado (que hace referencia al servicio vinculado de Azure Blob Storage, suponiendo que tiene archivos de texto como origen) y el conjunto de datos de Azure SQL Table (que hace referencia al servicio vinculado de Azure SQL Database). Los servicios vinculados de Azure Blob Storage y Azure SQL Database contienen cadenas de conexión que Data Factory usa en runtime para conectarse a las instancias de Azure Storage y Azure SQL Database, respectivamente. El conjunto de datos de texto delimitado especifica el contenedor de blobs y la carpeta de blobs que contiene los blobs de entrada de Blob Storage, junto con la configuración relacionada con el formato. El conjunto de datos Azure SQL Table especifica la tabla de SQL de SQL Database en la que se van a copiar los datos.

En el siguiente diagrama se muestra la relación entre la canalización, la actividad, el conjunto de datos y el servicio vinculado en Data Factory:

![Relación entre la canalización, la actividad, el conjunto de datos y los servicios vinculados](media/concepts-datasets-linked-services/relationship-between-data-factory-entities.png)


## <a name="dataset-json"></a>Conjunto de datos JSON
Un conjunto de datos de Data Factory se define con el siguiente formato JSON:

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: DelimitedText, AzureSqlTable etc...>",
        "linkedServiceName": {
                "referenceName": "<name of linked service>",
                "type": "LinkedServiceReference",
        },
        "schema":[

        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        }
    }
}
```
La tabla siguiente describe las propiedades del JSON anterior:

Propiedad | Descripción | Obligatorio |
-------- | ----------- | -------- |
name | Nombre del conjunto de datos. Consulte [Azure Data Factory - Naming rules](naming-rules.md) (Azure Data Factory: reglas de nomenclatura). |  Sí |
type | Tipo de conjunto de datos. Especifique uno de los tipos admitidos por Data Factory (por ejemplo: DelimitedText, AzureSqlTable). <br/><br/>Para más información, consulte [Dataset types](#dataset-type) (Tipo de conjunto de datos). | Sí |
esquema | Esquema del conjunto de datos. Representa la forma y tipo de datos físico. | No |
typeProperties | Las propiedades de tipo son diferentes para cada tipo. Para más información sobre los tipos admitidos y sus propiedades, consulte [Tipo de conjunto de datos](#dataset-type). | Sí |

Cuando importe el esquema de un conjunto de datos, seleccione el botón **Importar esquema** y elija Importar desde el origen o desde un archivo local. En la mayoría de los casos, importará el esquema directamente desde el origen. Pero si ya tiene un archivo de esquema local (un archivo Parquet o CSV con encabezados), puede indicarle a Data Factory que base el esquema en ese archivo.

En la actividad de copia, los conjunto de datos se usan en el origen y el receptor. El esquema definido en el conjunto de datos es opcional como referencia. Si quiere aplicar la asignación de campos o columnas entre el origen y el receptor, consulte [Asignación de esquemas y tipos de datos](copy-activity-schema-and-type-mapping.md).

En Data Flow, los conjuntos de datos se usan en las transformaciones de origen y receptor. Los conjuntos de datos definen los esquemas de datos básicos. Si los datos no tienen un esquema, puede utilizar un desfase de esquema para el origen y receptor. Los metadatos de los conjuntos de datos aparecen en la transformación de origen como la proyección de origen. La proyección en la transformación de origen representa los datos de Data Flow con nombres y tipos definidos.

## <a name="dataset-type"></a>Tipo de conjunto de datos

Azure Data Factory admite muchos tipos diferentes de conjuntos de datos, según los almacenes de datos que usa. Puede encontrar la lista de los almacenes de datos que admite Data Factory en el artículo [Introducción a los conectores](connector-overview.md). Haga clic en un almacén de datos para obtener información sobre cómo crear un servicio vinculado y un conjunto de datos para ese almacén de datos.

Por ejemplo, para un conjunto de datos de texto delimitado, el tipo de conjunto de datos se establece en **DelimitedText**, como se muestra en el siguiente ejemplo de JSON:

```json
{
    "name": "DelimitedTextInput",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorage",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "DelimitedText",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": "input.log",
                "folderPath": "inputdata",
                "container": "adfgetstarted"
            },
            "columnDelimiter": ",",
            "escapeChar": "\\",
            "quoteChar": "\""
        },
        "schema": []
    }
}
```

## <a name="create-datasets"></a>Creación de conjuntos de datos
Puede crear conjuntos de datos mediante una de estas herramientas o SDK: [API de .NET](quickstart-create-data-factory-dot-net.md), [PowerShell](quickstart-create-data-factory-powershell.md), [API de REST](quickstart-create-data-factory-rest-api.md), plantilla de Azure Resource Manager y Azure Portal

## <a name="current-version-vs-version-1-datasets"></a>Conjuntos de datos de la versión actual frente a los de la versión 1

A continuación se indican algunas diferencias entre los conjuntos de datos de Data Factory frente a los de Data Factory versión 1:

- La propiedad externa no se admite en la versión actual. Se sustituye por un [desencadenador](concepts-pipeline-execution-triggers.md).
- Las propiedades de directiva y disponibilidad no se admiten en la versión actual. La hora de inicio de una canalización depende de [desencadenadores](concepts-pipeline-execution-triggers.md).
- Los conjuntos de datos con ámbito (conjuntos de datos definidos en una canalización) no se admiten en la versión actual.

## <a name="next-steps"></a>Pasos siguientes
Consulte el siguiente tutorial para obtener instrucciones paso a paso sobre cómo crear canalizaciones y conjuntos de datos con una de estas herramientas o SDK.

- [Inicio rápido: create a data factory using .NET](quickstart-create-data-factory-dot-net.md) (Crear una factoría de datos mediante .NET)
- [Inicio rápido: create a data factory using PowerShell](quickstart-create-data-factory-powershell.md) (Crear una factoría de datos mediante PowerShell)
- [Inicio rápido: create a data factory using REST API](quickstart-create-data-factory-rest-api.md) (Crear una factoría de datos mediante la API de REST)
- [Inicio rápido: creación de una factoría de datos mediante Azure Portal](quickstart-create-data-factory-portal.md)
