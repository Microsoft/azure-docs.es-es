---
title: Creación de conjuntos de datos en Azure Data Factory
description: Aprenda a crear conjuntos de datos en Azure Data Factory con ejemplos que usan propiedades como offset y anchorDateTime.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 13ee8c5166e3b31ec186ca9aa2702d15de36e09f
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86522404"
---
# <a name="datasets-in-azure-data-factory-version-1"></a>Conjuntos de datos en Azure Data Factory (versión  1)
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-create-datasets.md)
> * [Versión 2 (versión actual)](../concepts-datasets-linked-services.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el artículo sobre [conjuntos de datos in V2](../concepts-datasets-linked-services.md).

En este artículo se describe qué son los conjuntos de datos, cómo se definen en formato JSON y cómo se usan en canalizaciones de Azure Data Factory. Se incluyen detalles sobre cada sección (por ejemplo, structure, availability y policy) en la definición JSON del conjunto de datos. También se ofrecen ejemplos de uso de las propiedades **offset**, **anchorDateTime** y **style** en una definición JSON de conjunto de datos.

> [!NOTE]
> Si no está familiarizado con Data Factory, consulte [Introducción a Azure Data Factory](data-factory-introduction.md) para obtener información general. Si no tiene experiencia práctica con la creación de factorías de datos, lea el [tutorial de transformación de datos](data-factory-build-your-first-pipeline.md) y el [tutorial de movimiento de datos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para adquirir una mejor comprensión.

## <a name="overview"></a>Información general
Una factoría de datos puede tener una o más canalizaciones. Una **canalización** es una agrupación lógica de **actividades** que realizan una tarea. Las actividades de una canalización definen las acciones que se van a realizar en los datos. Por ejemplo, se podría usar una actividad de copia para copiar datos de una base de datos de SQL Server a una instancia de Azure Blob Storage. Después, podría usar una actividad de Hive que ejecute un script de Hive en un clúster de Azure HDInsight para procesar datos de Blob Storage con el fin de generar datos de salida. Finalmente, podría usar segunda actividad de copia para copiar los datos de salida en una instancia de Azure SQL Data Warehouse en función de qué soluciones de generación de informes de inteligencia empresarial (BI) estén integradas. Para más información sobre canalizaciones y actividades, consulte el artículo [Canalizaciones y actividades en Azure Data Factory](data-factory-create-pipelines.md).

Una actividad puede tomar diversos **conjuntos de datos** de entrada, o ninguno, y generar uno o varios conjuntos de datos de salida. Un conjunto de datos de entrada representa la entrada para una actividad de la canalización y un conjunto de datos de salida representa la salida de la actividad. Los conjuntos de datos identifican datos en distintos almacenes de datos, como tablas, archivos, carpetas y documentos. Por ejemplo, un conjunto de datos de blob de Azure especifica el contenedor de blobs y la carpeta de Blob Storage de los que la canalización debe leer los datos.

Antes de crear un conjunto de datos, cree un **servicio vinculado** para vincular su almacén de datos a la factoría de datos. Los servicios vinculados son muy similares a las cadenas de conexión que definen la información de conexión necesaria para que Data Factory se conecte a recursos externos. Los conjuntos de datos identifican datos en los almacenes de datos vinculados, como tablas, archivos, carpetas y documentos de SQL. Por ejemplo, un servicio vinculado Azure Storage vincula una cuenta de almacenamiento a la factoría de datos. Un conjunto de datos de blobs de Azure representa el contenedor de blobs y la carpeta que contiene los blobs de entrada que se van a procesar.

Este es un escenario de ejemplo. Para copiar datos de Blob Storage en SQL Database, se crean dos servicios vinculados: Microsoft Azure Storage y Azure SQL Database. Después, creará dos conjuntos de datos: el conjunto de datos de un blob de Azure (que hace referencia al servicio vinculado Azure Storage) y el conjunto de datos de Azure SQL Table (que hace referencia al servicio vinculado Azure SQL Database). Los servicios vinculados de Azure Storage y Azure SQL Database contienen cadenas de conexión que Data Factory usa en tiempo de ejecución para conectarse a las instancias de Azure Storage y Azure SQL Database, respectivamente. El conjunto de datos Azure Blob especifica el contenedor de blobs y la carpeta de blobs que contiene los blobs de entrada de Blob Storage. El conjunto de datos Azure SQL Table especifica la tabla de SQL de la base de datos SQL en la que se van a copiar los datos.

En el siguiente diagrama se muestra la relación entre la canalización, la actividad, el conjunto de datos y el servicio vinculado en Data Factory:

![Relación entre la canalización, la actividad, el conjunto de datos y los servicios vinculados](media/data-factory-create-datasets/relationship-between-data-factory-entities.png)

## <a name="dataset-json"></a>Conjunto de datos JSON
Un conjunto de datos de Data Factory se define con formato JSON de la manera siguiente:

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": "<boolean flag to indicate external data. only for input datasets>",
        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        },
        "availability": {
            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
        },
        "policy":
        {
        }
    }
}
```

La tabla siguiente describe las propiedades del JSON anterior:

| Propiedad | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| name |Nombre del conjunto de datos. Consulte [Azure Data Factory: reglas de nomenclatura](data-factory-naming-rules.md) para ver este tipo de reglas. |Sí |N/D |
| type |Tipo de conjunto de datos. Especifique uno de los tipos admitidos por Data Factory (por ejemplo: AzureBlob, AzureSqlTable). <br/><br/>Para más información, consulte [Tipo de conjunto de datos](#Type). |Sí |N/D |
| structure |Esquema del conjunto de datos.<br/><br/>Para más información, consulte [Estructura del conjunto de datos](#Structure). |No |N/D |
| typeProperties | Las propiedades de tipo son diferentes para cada tipo (por ejemplo: blob de Azure, tabla de Azure SQL). Para más información sobre los tipos admitidos y sus propiedades, consulte [Tipo de conjunto de datos](#Type). |Sí |N/D |
| external | Marca booleana para especificar si un conjunto de datos es generado explícitamente por una canalización de la factoría de datos o no. Si la canalización actual no produce el conjunto de datos de entrada de una actividad, defina esta marca en true. Establezca esta marca en true para el conjunto de datos de entrada de la primera actividad de la canalización.  |No |false |
| availability | Define la ventana de procesamiento (por ejemplo, horas, días, etc.) o el modelo de segmentación para la producción del conjunto de datos. Cada unidad de datos consumida y producida por la ejecución de una actividad se denomina segmento de datos. Si la disponibilidad de un conjunto de datos de salida se establece en diaria (frecuencia: día, intervalo: 1), se produce un segmento diariamente. <br/><br/>Para más información, consulte Disponibilidad del conjunto de datos. <br/><br/>Para más información sobre el modelo de segmentación del conjunto de datos, consulte el artículo [Programación y ejecución de Data Factory](data-factory-scheduling-and-execution.md). |Sí |N/D |
| policy |Define los criterios o la condición que deben cumplir los segmentos del conjunto de datos. <br/><br/>Para más información, consulte la sección [Directiva del conjunto de datos](#Policy). |No |N/D |

## <a name="dataset-example"></a>Ejemplo de conjunto de datos
En el siguiente ejemplo, el conjunto de datos representa una tabla llamada **MyTable** en una base de datos SQL.

```json
{
    "name": "DatasetSample",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties":
        {
            "tableName": "MyTable"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Tenga en cuenta los siguientes puntos:

* **type** está establecido en AzureSqlTable.
* La propiedad de tipo **tableName** (específica del tipo AzureSqlTable) está establecida en MyTable.
* **linkedServiceName** hace referencia a un servicio vinculado de tipo AzureSqlDatabase, que se define en el siguiente fragmento JSON.
* **availability frequency** está establecido en Day e **interval** está establecido en 1. Esto significa que el fragmento de conjunto de datos se genera diariamente.

**AzureSqlLinkedService** se define de la siguiente forma:

```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>@<servername>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

En el fragmento JSON anterior:

* **type** está establecido en AzureSqlDatabase.
* La propiedad de tipo **connectionString** especifica información para conectarse a una base de datos SQL.

Como puede ver, el servicio vinculado define cómo conectarse a una base de datos SQL. El conjunto de datos define qué tabla se usa como entrada y salida de la actividad en una canalización.

> [!IMPORTANT]
> A menos que se produzca un conjunto de datos mediante la canalización, debe marcarse como **external**. Esta configuración se aplica generalmente a las entradas de la primera actividad de una canalización.

## <a name="dataset-type"></a><a name="Type"></a> Tipo de conjunto de datos
El tipo del conjunto de datos depende del almacén de datos que utilice. Vea la tabla siguiente para obtener una lista de almacenes de datos compatibles con Data Factory. Haga clic en un almacén de datos para obtener información sobre cómo crear un servicio vinculado y un conjunto de datos para ese almacén de datos.

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Los almacenes de datos con * pueden ser locales o de infraestructura como servicio (IaaS) de Azure. Estos almacenes de datos requieren que instale [Data Management Gateway](data-factory-data-management-gateway.md).

En el ejemplo en la sección anterior, el tipo del conjunto de datos se establece en **AzureSqlTable**. De forma similar, para un conjunto de datos Azure Blob, el tipo del conjunto de datos se establece en **AzureBlob**, tal y como se muestra en el siguiente esquema JSON:

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

## <a name="dataset-structure"></a><a name="Structure"></a>Estructura del conjunto de datos
La sección **structure** es opcional. Define el esquema del conjunto de datos al contener una colección de nombres y tipos de datos de columnas. La sección "structure" se usa para proporcionar el tipo de información que se utiliza para convertir tipos y columnas de mapas del origen al destino. En el ejemplo siguiente, el conjunto de datos tiene tres columnas: `slicetimestamp`, `projectname` y `pageviews`. Estas columnas son del tipo String, String, y Decimal, respectivamente.

```json
structure:
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

Cada columna de la estructura contiene las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| name |Nombre de la columna. |Sí |
| type |Tipo de datos de la columna.  |No |
| culture |Referencia cultural basada en .NET que se usará cuando se trate de un tipo .NET: `Datetime` o `Datetimeoffset`. El valor predeterminado es `en-us`. |No |
| format |Cadena de formato que se usará cuando se trate de un tipo .NET: `Datetime` o `Datetimeoffset`. |No |

Las siguientes instrucciones le ayudan a determinar cuándo incluir información de estructura y qué incluir en la sección **structure**.

* **Para orígenes de datos estructurados**, especifique la sección "structure" solo si desea asignar columnas de origen a columnas de receptor y sus nombres no son iguales. Este tipo de origen de datos estructurados almacena información de esquema y tipo de datos junto con los datos propiamente dichos. Ejemplos de orígenes de datos estructurados son SQL Server, Oracle y Azure Table.
  
    Dado que la información de tipo ya está disponible para orígenes de datos estructurados, no debe incluir información de tipo cuando se incluye la sección "structure".
* **En cuanto al esquema de los orígenes de datos de lectura (en concreto, Blob Storage)** , puede optar por guardar los datos sin almacenar ningún esquema o información de tipos con ellos. Para estos tipos de orígenes de datos, incluya "structure" cuando desee asignar columnas de origen a columnas de receptor. Incluya también "structure" cuando el conjunto de datos sea una entrada para una actividad de copia y los tipos de datos del conjunto de datos de origen se deban convertir a tipos nativos para el receptor.
    
    Data Factory admite los siguientes valores para proporcionar información de tipo en la estructura: **Int16, Int32, Int64, Single, Double, Decimal, Byte[], Boolean, String, Guid, Datetime, Datetimeoffset y Timespan**. Estos valores son valores de tipos basados en .NET compatibles con Common Language Specification (CLS).

Data Factory realiza automáticamente las conversiones de tipo al mover datos desde un almacén de datos de origen a un almacén de datos de receptor.

## <a name="dataset-availability"></a>Conjunto de datos availability
En la sección **availability** de un conjunto de datos, se define la ventana de procesamiento (cada hora, diariamente, semanalmente, etc.) del conjunto de datos. Para más información sobre las ventanas de actividad, consulte el artículo sobre [programación y ejecución](data-factory-scheduling-and-execution.md).

La sección "availability" siguiente especifica que el conjunto de datos de salida se produce cada hora (o) que el conjunto de datos de entrada está disponible cada hora:

```json
"availability":
{
    "frequency": "Hour",
    "interval": 1
}
```

Si la canalización tiene las siguientes horas de inicio y finalización:

```json
    "start": "2016-08-25T00:00:00Z",
    "end": "2016-08-25T05:00:00Z",
```

El conjunto de datos de salida se genera cada hora dentro de las horas de inicio y finalización de la canalización. Por lo tanto, esta canalización genera cinco segmentos de conjuntos de datos, uno por cada ventana de actividad (12 a.m. - 1 a.m., 1 a.m. - 2 a.m., 2 a.m. - 3 a.m., 3 a.m. - 4 a.m., 4 a.m. - 5 a.m.).

La tabla siguiente describe las propiedades que puede utilizar en la sección de disponibilidad:

| Propiedad | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| frequency |Especifica la unidad de tiempo para la producción de segmentos del conjunto de datos.<br/><br/><b>Frecuencia admitida</b>: Minute, Hour, Day, Week, Month |Sí |N/D |
| interval |Especifica un multiplicador para "frequency".<br/><br/>"Frequency x interval" determina la frecuencia con la que se produce el segmento. Por ejemplo, si necesita segmentar el conjunto de datos cada hora, establezca <b>frequency</b> en <b>hour</b> e <b>interval</b> en <b>1</b>.<br/><br/>Observe que si especifica **frequency** como **Minute**, debe establecer el intervalo en un valor no inferior a 15. |Sí |N/D |
| style |Especifica si el segmento debe producirse al principio o al final del intervalo.<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul>Si **frequency** se establece en **Month** y **style** se establece en **EndOfInterval**, el segmento se produce el último día del mes. Si **style** se establece en **StartOfInterval**, el segmento se produce el primer día del mes.<br/><br/>Si **frequency** se establece en **Day** y **style** se establece en **EndOfInterval**, el segmento se produce la última hora del día.<br/><br/>Si **frequency** se establece en **Hour** y **style** se establece en **EndOfInterval**, el segmento se produce al final de la hora. Por ejemplo, para un segmento en el período de 1 p.m. - 2 p.m., el segmento se producirá a las 2 p.m. |No |EndOfInterval |
| anchorDateTime |Define la posición absoluta en el tiempo usada por el programador para calcular los límites del segmento de conjunto de datos. <br/><br/>Tenga en cuenta que si esta propiedad tiene partes de fecha que son más específicas que la frecuencia indicada, las partes más específicas se ignoran. Por ejemplo, si el valor de **interval** es **hourly** (frecuencia: hora e intervalo: 1) y **anchorDateTime** contiene **minutes and seconds**, las partes de minutes and seconds de **anchorDateTime** no se tienen en cuenta. |No |01/01/0001 |
| offset |Intervalo de tiempo en función del cual se desplazan el inicio y el final de todos los segmentos del conjunto de datos. <br/><br/>Tenga en cuenta que si se especifican **anchorDateTime** y **offset**, el resultado es el desplazamiento combinado. |No |N/D |

### <a name="offset-example"></a>Ejemplo de offset
De forma predeterminada, los segmentos diarios (`"frequency": "Day", "interval": 1`) comienzan a las 12 p.m. (medianoche), Hora universal coordinada (UTC). Si desea que, en su lugar, la hora de inicio sea a las 6:00 UTC, defina el desplazamiento como se muestra en el siguiente fragmento de código:

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
### <a name="anchordatetime-example"></a>Ejemplo de anchorDateTime
En el ejemplo siguiente, el conjunto de datos se produce una vez cada 23 horas. El primer segmento se inicia a la hora especificada en **anchorDateTime**, que se establece en `2017-04-19T08:00:00`.

```json
"availability":
{
    "frequency": "Hour",
    "interval": 23,
    "anchorDateTime":"2017-04-19T08:00:00"
}
```

### <a name="offsetstyle-example"></a>Ejemplo de offset/style
El siguiente conjunto de datos es mensual y se produce el tercer día de cada mes a las 8 a.m. (`3.08:00:00`):

```json
"availability": {
    "frequency": "Month",
    "interval": 1,
    "offset": "3.08:00:00", 
    "style": "StartOfInterval"
}
```

## <a name="dataset-policy"></a><a name="Policy"></a>Directiva del conjunto de datos
En la sección **policy** de la definición del conjunto de datos se definen los criterios o condiciones que deben cumplir los segmentos del conjunto de datos.

### <a name="validation-policies"></a>Directivas de validación
| Nombre de la directiva | Descripción | Se aplica a | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- | --- |
| minimumSizeMB |Valida que los datos de **Azure Blob Storage** satisfacen los requisitos de tamaño mínimo (en megabytes). |Azure Blob Storage |No |N/D |
| minimumRows |Valida que los datos de una **base de datos de Azure SQL** o una **tabla de Azure** contienen el número mínimo de filas. |<ul><li>Azure SQL Database</li><li>Tabla de Azure</li></ul> |No |N/D |

#### <a name="examples"></a>Ejemplos
**minimumSizeMB:**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

**minimumRows:**

```json
"policy":
{
    "validation":
    {
        "minimumRows": 100
    }
}
```

### <a name="external-datasets"></a>Conjuntos de datos externos
Los conjuntos de datos externos son los que no son producidos por una canalización de ejecución en la factoría de datos. Si el conjunto de datos está marcado como **external**, puede definirse la directiva **ExternalData** para influir en el comportamiento de la disponibilidad de los segmentos del conjunto de datos.

A menos que se esté produciendo un conjunto de datos mediante Data Factory, debe marcarse como **external**. Generalmente, esta configuración se aplicará a las entradas de la primera actividad de una canalización a menos que se desee usar el encadenamiento de actividades o canalizaciones.

| Nombre | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| dataDelay |Tiempo de retraso de la comprobación de la disponibilidad de los datos externos para el segmento especificado. Por ejemplo, puede retrasar una comprobación horaria mediante esta configuración.<br/><br/>Esta configuración solo se aplica a la hora actual. Por ejemplo, si es la 1:00 p.m. y este valor es de 10 minutos, la validación se inicia a la 1:10 p.m.<br/><br/>Tenga en cuenta que esta configuración no afecta a los segmentos del pasado. Los segmentos con **Slice End Time** + **dataDelay** < **Now** se procesan sin retraso.<br/><br/>Los tiempos mayores que 23:59 horas se deben especificar mediante el formato `day.hours:minutes:seconds`. Por ejemplo, para especificar 24 horas, no use 24:00:00. En su lugar, use 1.00:00:00. Si usa 24:00:00, se tratará como 24 días (24.00:00:00). Para 1 día y 4 horas, especifique 1:04:00:00. |No |0 |
| retryInterval |El tiempo de espera entre un error y el siguiente intento. Esta configuración se aplica a la hora actual. Si se ha producido algún error en el intento anterior, el siguiente intento se realiza después del período **retryInterval**. <br/><br/>Si ahora es la 1:00 p. m., empezaremos el primer intento. Si la duración para completar la primera comprobación de validación es 1 minuto y la operación produce un error, el siguiente reintento será a la 1:00 + 1 minuto (duración) + 1 minuto (intervalo de reintento) = 1:02 p.m. <br/><br/>En el caso de los segmentos en el pasado, no habrá ningún retraso. El reintento se producirá inmediatamente. |No |00:01:00 (1 minuto) |
| retryTimeout |El tiempo de espera de cada reintento.<br/><br/>Si esta propiedad se establece en 10 minutos, la validación se debe completar en 10 minutos. Si la validación tarda más de 10 minutos en realizarse, el reintento agotará el tiempo de espera.<br/><br/>Si se agota el tiempo de espera de todos los intentos de validación, el segmento se marca como **TimedOut**. |No |00:10:00 (10 minutos) |
| maximumRetry |Número de veces que se va a comprobar la disponibilidad de los datos externos. El máximo permitido es de 10. |No |3 |


## <a name="create-datasets"></a>Creación de conjuntos de datos
Puede crear conjuntos de datos mediante el uso de algunas de estas herramientas o SDK:

- Asistente para copia
- Visual Studio
- PowerShell
- Plantilla del Administrador de recursos de Azure
- API DE REST
- API de .NET

Consulte los siguientes tutoriales para obtener instrucciones paso a paso sobre cómo crear canalizaciones y conjuntos de datos con una de estas herramientas o SDK:

- [Creación de una canalización con una actividad de transformación de datos](data-factory-build-your-first-pipeline.md)
- [Creación de una canalización con una actividad de movimiento de datos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

Una vez que se haya creado e implementado la canalización, puede administrar y supervisar sus canalizaciones mediante las hojas de Azure Portal o la aplicación de supervisión y administración. Consulte los siguientes temas para obtener instrucciones paso a paso:

- [Supervisión y administración de canalizaciones con las hojas de Azure Portal](data-factory-monitor-manage-pipelines.md)
- [Supervisión y administración de canalizaciones mediante la aplicación de supervisión y administración](data-factory-monitor-manage-app.md)

## <a name="scoped-datasets"></a>Conjuntos de datos limitados
Puede crear conjuntos de datos que se limitan a una canalización mediante la propiedad **datasets** . Estos conjuntos de datos solo los pueden usar las actividades dentro de esta canalización, pero no las actividades de otras canalizaciones. En el ejemplo siguiente se define una canalización con dos conjuntos de datos: InputDataset-rdc y OutputDataset-rdc, que se usarán dentro de la canalización:

> [!IMPORTANT]
> Los conjuntos de datos con ámbito solo se admiten con canalizaciones de un solo uso (donde **pipelineMode** está establecido en **OneTime**). Consulte la sección [Canalización de una vez](data-factory-create-pipelines.md#onetime-pipeline) para más información.
>
>

```json
{
    "name": "CopyPipeline-rdc",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-rdc"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-rdc"
                    }
                ],
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1,
                    "style": "StartOfInterval"
                },
                "name": "CopyActivity-0"
            }
        ],
        "start": "2016-02-28T00:00:00Z",
        "end": "2016-02-28T00:00:00Z",
        "isPaused": false,
        "pipelineMode": "OneTime",
        "expirationTime": "15.00:00:00",
        "datasets": [
            {
                "name": "InputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "InputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/input",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": true,
                    "policy": {}
                }
            },
            {
                "name": "OutputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "OutputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/output",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": false,
                    "policy": {}
                }
            }
        ]
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre las canalizaciones, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md).
- Para más información sobre cómo programar y ejecutar canalizaciones en Azure Data Factory, consulte [este artículo](data-factory-scheduling-and-execution.md).
