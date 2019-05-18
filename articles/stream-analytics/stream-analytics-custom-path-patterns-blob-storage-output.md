---
title: Particionamiento de la salida de blobs personalizada en Azure Stream Analytics
description: En este artículo se describen las características de patrones personalizados de rutas de acceso de fecha y hora y de atributo o campo personalizado para la salida de Blob Storage desde trabajos de Azure Stream Analytics.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 02/07/2019
ms.custom: seodec18
ms.openlocfilehash: e06313cf83768421bedc6c7baddd30c2ef2e4846
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789418"
---
# <a name="azure-stream-analytics-custom-blob-output-partitioning"></a>Particionamiento de la salida de blobs personalizada en Azure Stream Analytics

Azure Stream Analytics admite la creación de particiones de salida de blob personalizada con atributos o campos personalizados y patrones personalizados de ruta de acceso de fecha y hora. 

## <a name="custom-field-or-attributes"></a>Atributos o campos personalizados

Los campos o atributos de entrada personalizados mejoran los flujos de trabajo de procesamiento de datos e informes descendentes, ya que le otorgan más control sobre la salida.

### <a name="partition-key-options"></a>Opciones de clave de partición

La clave de partición o nombre de columna que se usa para particionar los datos de entrada puede contener caracteres alfanuméricos, guiones, guiones bajos y espacios. No es posible usar los campos anidados como clave de partición, a menos que se usen junto con los alias. La clave de partición debe ser nvarchar (max).

### <a name="example"></a>Ejemplo

Supongamos que un trabajo toma datos de entrada de sesiones de usuarios en directo que están conectadas a un servicio externo de videojuegos en el que los datos incorporados contienen una columna **client_id** para identificar las sesiones. Para particionar los datos por **client_id**, configure el campo de patrón de ruta de acceso de blob para que incluya un token de partición **{client_id}** en las propiedades de salida de blob al crear un trabajo. A medida que los datos con varios valores de **client_id** pasan por el trabajo de Stream Analytics, los datos de salida se guardan en carpetas independientes con un valor de **client_id** único por carpeta.

![Patrón de ruta de acceso con el Id. de cliente](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-path-pattern-client-id.png)

De forma similar, si la entrada del trabajo eran datos de sensor de millones de sensores (de forma que cada uno tenía un **sensor_id**), el patrón de ruta de acceso sería **{sensor_id}** para particionar los datos de cada sensor en una carpeta diferente.  


Con la API REST, la sección de salida de un archivo JSON que se usó para dicha solicitud puede ser similar a la siguiente:  

![Salida de la API REST](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-rest-output.png)

Una vez que el trabajo empieza a ejecutarse, el contenedor de *clientes* puede tener un aspecto similar al siguiente:  

![Contenedor de clientes](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-clients-container.png)

Cada carpeta puede contener varios blobs, de forma que cada blob contiene uno o más registros. En el ejemplo anterior, hay un único blob en una carpeta etiquetada como "06000000" con el siguiente contenido:

![Contenido del blob](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-blob-contents.png)

Tenga en cuenta que cada registro en el blob tiene una columna **client_id** que coincide con el nombre de la carpeta, ya que se usó la columna **client_id** para particionar la salida en la ruta de acceso.

### <a name="limitations"></a>Limitaciones

1. Solo se permite una clave de partición personalizada en la propiedad de patrón de ruta de acceso de la salida del blob. Todos los patrones de ruta de acceso siguientes son válidos:

   * cluster1/{fecha}/{unCampoDeLosDatos}  
   * cluster1/{hora}/{unCampoDeLosDatos}  
   * cluster1/{unCampoDeLosDatos}  
   * cluster1/{fecha}/{hora}/{unCampoDeLosDatos} 
   
2. Las claves de partición no distinguen mayúsculas de minúsculas, por lo que los valores como "Juan" y "juan" son equivalentes. Además, no pueden usarse expresiones como clave de partición. Por ejemplo, **{columnaA + columnaB}** no funciona.  

3. Cuando un flujo de entrada consta de registros con una cardinalidad de clave de partición menor que 8000, los registros se anexarán a los blobs existentes y solo se crearán blobs nuevos cuando sea necesario. Si la cardinalidad es mayor que 8000, no garantizamos que se escribirá en los blobs existentes ni que no se crearán blobs nuevos para un número arbitrario de registros con la misma clave de partición.

## <a name="custom-datetime-path-patterns"></a>Patrones personalizados de ruta de acceso de fecha y hora

Los patrones de la ruta de acceso de fecha y hora personalizados le permiten especificar un formato de salida que se alinee con las convenciones de Hive Streaming, lo que le proporciona a Azure Stream Analytics la capacidad de enviar datos a Azure HDInsight y Azure Databricks para su procesamiento posterior. Los patrones de la ruta de acceso de fecha personalizados se implementan fácilmente utilizando la palabra clave `datetime` en el campo Prefijo de ruta de acceso de la salida de blob, así como el especificador de formato. Por ejemplo, `{datetime:yyyy}`.

### <a name="supported-tokens"></a>Tokens admitidos

Los siguientes tokens del especificador de formato se pueden utilizar solos o en combinación para conseguir formatos personalizados de fecha y hora:

|Especificador de formato   |DESCRIPCIÓN   |Resultados en la hora de ejemplo 2018-01-02T10: 06: 08|
|----------|-----------|------------|
|{datetime:yyyy}|El año como un número de cuatro dígitos|2018|
|{datetime:MM}|Mes desde 01 hasta 12|01|
|{datetime:M}|Mes desde 1 hasta 12|1|
|{datetime:dd}|Día desde 01 hasta 31|02|
|{datetime:d}|Día desde 1 hasta 12|2|
|{datetime:HH}|Hora con el formato de 24 horas, de 00 a 23|10|
|{datetime:mm}|Minutos de 00 a 24|06|
|{datetime:m}|Minutos de 0 a 24|6|
|{datetime:ss}|Segundos de 00 a 60|08|

Si no quiere usar patrones de fecha y hora personalizados, puede agregar el token {date} o {time} al prefijo de ruta de acceso para crear un menú desplegable con los formatos incorporados de fecha y hora.

![Formatos antiguos de fecha y hora de Stream Analytics](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-old-date-time-formats.png)

### <a name="extensibility-and-restrictions"></a>Extensibilidad y restricciones

Puede usar tantos tokens (`{datetime:<specifier>}`) como quiera en el patrón de la ruta de acceso hasta que alcance el límite de caracteres del prefijo de ruta de acceso. Los especificadores de formato no se pueden combinar en un solo token si van más allá de las combinaciones ya enumeradas en los menús desplegables de fecha y hora. 

Para una partición de ruta de `logs/MM/dd`:

|Expresión válida   |Expresión no válida   |
|----------|-----------|
|`logs/{datetime:MM}/{datetime:dd}`|`logs/{datetime:MM/dd}`|

Puede usar el mismo especificador de formato varias veces en el prefijo de ruta de acceso. El token debe repetirse cada vez.

### <a name="hive-streaming-conventions"></a>Convenciones de Hive Streaming

Los patrones de ruta de acceso personalizados para Blob Storage se pueden usar con la convención Hive Streaming, que espera que las carpetas estén etiquetadas con `column=` en el nombre de la carpeta.

Por ejemplo, `year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}/hour={datetime:HH}`.

La salida personalizada elimina la necesidad de tener que alterar las tablas y agregar particiones manualmente a los datos de puertos que estén entre Azure Stream Analytics y Hive. En su lugar, muchas carpetas se pueden agregar automáticamente mediante:

```SQL
MSCK REPAIR TABLE while hive.exec.dynamic.partition true
```

### <a name="example"></a>Ejemplo

Cree una cuenta de almacenamiento, un grupo de recursos, un trabajo de Stream Analytics y un origen de entrada de acuerdo con la guía de inicio rápido de [Azure Stream Analytics mediante Azure Portal ](stream-analytics-quick-create-portal.md). Use los mismos datos de muestra utilizados en la guía de inicio rápido, que también está disponible en [GitHub](https://raw.githubusercontent.com/Azure/azure-stream-analytics/master/Samples/GettingStarted/HelloWorldASA-InputStream.json).

Cree un receptor de salida de blob con la siguiente configuración:

![Stream Analytics crea un receptor de salida de blob](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-create-output-sink.png)

El patrón de la ruta de acceso completo es el siguiente:


`year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}`


Cuando inicie el trabajo, se creará una estructura de carpetas basada en el patrón de la ruta de acceso en el contenedor de blobs. Puede explorar en profundidad hasta llegar al nivel del día.

![Salida de blobs de Stream Analytics con patrón de ruta de acceso personalizado](./media/stream-analytics-custom-path-patterns-blob-storage-output/stream-analytics-blob-output-folder-structure.png)

## <a name="next-steps"></a>Pasos siguientes

* [Información sobre las salidas desde Azure Stream Analytics](stream-analytics-define-outputs.md)
