---
title: Introducción a Storage - Azure Time Series Insights Gen2 | Microsoft Docs
description: Información acerca del almacenamiento de datos en Azure Time Series Insights Gen2.
author: esung22
ms.author: elsung
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 08/25/2020
ms.custom: seodec18
ms.openlocfilehash: a0f1e7789c0cebdd1cb5b22f21151020a0be09c9
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855108"
---
# <a name="data-storage"></a>Almacenamiento de datos

Cuando se crea un entorno de Azure Time Series Insights Gen2, se crean también dos recursos de Azure:

* Un entorno de Azure Time Series Insights Gen2 que se puede configurar para el almacenamiento de datos intermedio.
* Una cuenta de Azure Storage para el almacenamiento de datos en frío.

Los datos del almacenamiento intermedio solo están disponibles a través de las [API Time Series Query](./time-series-insights-update-tsq.md) y el [explorador de Azure Time Series Insights TSI](./time-series-insights-update-explorer.md). El almacén intermedio contendrá datos recientes dentro del [período de retención](./time-series-insights-update-plan.md#the-preview-environment) seleccionado al crear el entorno de Azure Time Series Insights Gen2.

Azure Time Series Insights Gen2 guarda los datos del almacenamiento en reposo en Azure Blob Storage con el [formato de archivo de Parquet](#parquet-file-format-and-folder-structure). Azure Time Series Insights Gen2 administra estos datos del almacenamiento en reposo de forma exclusiva, aunque están disponibles para leerse directamente como archivos estándar de Parquet.

> [!WARNING]
> Como propietario de la cuenta de Azure Blob Storage donde residen los datos del almacenamiento intermedio, tiene acceso total a todos los datos de la cuenta. Este acceso incluye permisos de escritura y eliminación. No modifique ni elimine los datos que escribe Azure Time Series Insights Gen2, ya que estas operaciones pueden provocar una pérdida de datos.

## <a name="data-availability"></a>Disponibilidad de los datos

Azure Time Series Insights Gen2 crea particiones de los datos y los indexa para lograr un rendimiento óptimo de las consultas. Los datos están disponibles para realizar consultas tanto en almacenamiento parcialmente activo (si está habilitado) como en almacenamiento en reposo. La cantidad de datos que se va a ingerir puede afectar a esta disponibilidad.

> [!IMPORTANT]
> Es posible que experimente un período de hasta 60 segundos hasta que los datos estén disponibles. Si experimenta una latencia considerable de más de 60 segundos, envíe una incidencia de soporte técnico a través de Azure Portal.

## <a name="azure-storage"></a>Azure Storage

En esta sección se describen detalles de Azure Storage relativos a Azure Time Series Insights Gen2.

Para obtener una descripción detallada de Azure Blob Storage, lea [Introducción a Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md).

### <a name="your-storage-account"></a>Cuenta de almacenamiento

Cuando se crea un entorno de Azure Time Series Insights Gen2, se crea también una cuenta de Azure Storage como almacenamiento en reposo a largo plazo.  

Azure Time Series Insights Gen2 conserva un máximo de dos copias de cada evento en su cuenta de Azure Storage. Una copia almacena los eventos ordenados por hora de ingesta, permitiendo siempre el acceso a los eventos en una secuencia ordenada por tiempo. Con el tiempo, Azure Time Series Insights Gen2 también crea una copia con una nueva partición de los datos que se van a optimizar para las consultas eficaces.

Todos los datos se almacenan de forma indefinida en su cuenta de Azure Storage.

#### <a name="writing-and-editing-blobs"></a>Escritura y edición de blobs

Para garantizar el rendimiento de las consultas y la disponibilidad de los datos, no edite ni elimine los blobs que crea Azure Time Series Insights Gen2.

#### <a name="accessing-cold-store-data"></a>Acceso a datos de almacenamiento en reposo

Además de acceder a los datos desde el [explorador de Azure Time Series Insights TSI ](./time-series-insights-update-explorer.md) y las [API Time Series Query](./time-series-insights-update-tsq.md), puede que también desee acceder a los datos directamente desde los archivos Parquet almacenados en el almacenamiento en reposo. Por ejemplo, puede leer, transformar y limpiar los datos en un cuaderno de Jupyter y luego usarlo para entrenar el modelo de Azure Machine Learning en el mismo flujo de trabajo de Spark.

Para acceder a los datos directamente desde su cuenta de Azure Storage, necesita acceso de lectura a la cuenta usada para almacenar los datos de Azure Time Series Insights Gen2. A continuación, puede leer los datos seleccionados en función de la hora de creación del archivo de Parquet que se encuentra en la carpeta `PT=Time` que se describe a continuación en la sección [Formato de archivo de Parquet](#parquet-file-format-and-folder-structure).  Para más información sobre cómo habilitar el acceso de lectura a su cuenta de almacenamiento, consulte [Administración del acceso a los recursos de la cuenta de almacenamiento](../storage/blobs/storage-manage-access-to-resources.md).

#### <a name="data-deletion"></a>Eliminación de datos

No elimine los archivos de Azure Time Series Insights Gen2. Administre los datos relacionados solo desde Azure Time Series Insights Gen2.

### <a name="parquet-file-format-and-folder-structure"></a>Formato de archivo de Parquet y estructura de carpetas

Parquet es un formato de archivo en columnas de código abierto diseñado para lograr un almacenamiento y un rendimiento eficaces. Azure Time Series Insights Gen2 usa Parquet para habilitar el rendimiento de las consultas basadas en el identificador de la serie temporal a escala.  

Para más información sobre el tipo de archivo de Parquet, lea la [documentación de Parquet](https://parquet.apache.org/documentation/latest/).

Azure Time Series Insights Gen2 almacena copias de los datos de la siguiente manera:

* La primera copia inicial se particiona por hora de ingesta y almacena los datos más o menos en orden de llegada. Los datos residen en la carpeta `PT=Time`:

  `V=1/PT=Time/Y=<YYYY>/M=<MM>/<YYYYMMDDHHMMSSfff>_<TSI_INTERNAL_SUFFIX>.parquet`

* La segunda copia con particiones se agrupa por los identificadores de Time Series y reside en la carpeta `PT=TsId`:

  `V=1/PT=TsId/<TSI_INTERNAL_NAME>.parquet`

La marca de tiempo de los nombres de blob de la carpeta `PT=Time` corresponde a la hora de llegada de los datos a Azure Time Series Insights Gen2, no a la marca de tiempo de los eventos.

Los datos de la carpeta `PT=TsId` se optimizan para la consulta a lo largo del tiempo y no están estáticos. Durante la creación de particiones, es posible que algunos eventos estén presentes en varios blobs. No se garantiza que la nomenclatura de los blobs de esta carpeta siga siendo la misma.

En general, si necesita acceder a los datos directamente a través de los archivos de Parquet, use la carpeta `PT=Time`.  La futura funcionalidad permitirá el acceso eficaz a la carpeta `PT=TsId`.

> [!NOTE]
>
> * `<YYYY>` se asigna a una representación de año de cuatro dígitos.
> * `<MM>` se asigna a una representación de mes de dos dígitos.
> * `<YYYYMMDDHHMMSSfff>` se asigna a una representación de marca de tiempo con un año de cuatro dígitos (`YYYY`), un mes de dos dígitos (`MM`), un día de dos dígitos (`DD`), una hora de dos dígitos (`HH`), un minuto de dos dígitos (`MM`), un segundo de dos dígitos (`SS`) y un milisegundo de tres dígitos (`fff`).

Los eventos de Azure Time Series Insights Gen2 se asignan al contenido de los archivos de Parquet como se indica a continuación:

* Cada evento se asigna a una sola fila.
* Cada fila incluye la columna **timestamp** con una marca de tiempo del evento. La propiedad time-stamp nunca es null. Su valor predeterminado es **event enqueued time** si no se especifica en el origen del evento. La marca de tiempo almacenada siempre está en UTC.
* Cada fila incluye las columnas de identificadores de la serie temporal (TSID) como se define durante la creación del entorno de Azure Time Series Insights Gen2. El nombre de la propiedad del identificador de Time Series incluye el sufijo `_string`.
* Las demás propiedades enviadas como datos de telemetría se asignan a nombres de columna que terminan en `_bool` (booleano), `_datetime` (marca de tiempo), `_long` (largo) o `_double` (doble),`_string` (cadena) o `dynamic` (dinámico) en función del tipo de propiedad.  Para obtener más información, lea [Tipos de datos compatibles](./concepts-supported-data-types.md).
* Este esquema de asignación se aplica a la primera versión del formato de archivo, a la que se hace referencia como **V=1** y se almacena en la carpeta base del mismo nombre. A medida que esta característica evolucione, este esquema de asignación podría cambiar y se incrementaría el nombre de referencia.

## <a name="next-steps"></a>Pasos siguientes

* Lea sobre el [modelado de datos](./time-series-insights-update-tsm.md).

* Planeamiento del [entorno de Azure Time Series Insights Gen2](./time-series-insights-update-plan.md).
