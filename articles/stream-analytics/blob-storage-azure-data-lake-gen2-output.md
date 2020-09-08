---
title: Salida de Blob Storage y Azure Data Lake Gen2 de Azure Stream Analytics
description: En este artículo se describe Blob Storage y Azure Data Lake Gen 2 como salida para Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 2ab45f4c64e6993f70f08f04ee413211abb0307d
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875532"
---
# <a name="blob-storage-and-azure-data-lake-gen2-output-from-azure-stream-analytics"></a>Salida de Blob Storage y Azure Data Lake Gen2 de Azure Stream Analytics

Data Lake Storage Gen2 convierte a Azure Storage en los cimientos para crear lagos de datos empresariales en Azure. Diseñado desde el principio para servir varios petabytes de información y mantener cientos de gigabits de rendimiento, Data Lake Storage Gen2 le ofrece una forma fácil de administrar cantidades masivas de datos. Una parte fundamental de Data Lake Storage Gen2 es la incorporación de un espacio de nombres jerárquico a Blob Storage.

Azure Blob Storage ofrece una solución rentable y escalable para almacenar grandes cantidades de datos no estructurados en la nube. Para obtener una introducción a Blob Storage y su uso, consulte [Carga, descarga y enumeración de blobs con Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md).

## <a name="output-configuration"></a>Configuración de salida

En la siguiente tabla se enumeran los nombres de propiedad y su descripción para crear un blob o una salida de ADLS Gen2.

| Nombre de propiedad       | Descripción                                                                      |
| ------------------- | ---------------------------------------------------------------------------------|
| Alias de salida        | Un nombre descriptivo usado en las consultas para dirigir la salida de la consulta a este almacenamiento de blobs. |
| Cuenta de almacenamiento     | Nombre de la cuenta de almacenamiento a donde está enviando la salida.               |
| Clave de cuenta de almacenamiento | La clave secreta asociada con la cuenta de almacenamiento.                              |
| Contenedor de almacenamiento   | Agrupación lógica de los blobs almacenados en Azure Blob service. Cuando carga un blob a Blob service, debe especificar un contenedor para ese blob. |
| Patrón de la ruta de acceso | Opcional. Patrón de la ruta de acceso al archivo que se usa para escribir los blobs dentro del contenedor especificado. <br /><br /> En el patrón de la ruta de acceso, puede optar por usar una o más instancias de las variables de fecha y hora para especificar la frecuencia con la que se escriben los blobs: <br /> {date}, {time} <br /><br />Puede usar la creación de particiones de blobs personalizada para especificar un nombre {field} personalizado de los datos de eventos para crear particiones de los blobs. El nombre del campo es alfanumérico y puede incluir espacios, guiones y caracteres de subrayado. Entre las restricciones en los campos personalizados se incluyen las siguientes: <ul><li>Los nombres de campo no distinguen entre mayúsculas y minúsculas. Por ejemplo, el servicio no puede diferenciar entre la columna "ID" y la columna "id".</li><li>No se permiten los campos anidados. En su lugar, utilice un alias en la consulta del trabajo para "aplanar" el campo.</li><li>No pueden usarse expresiones como nombre de campo.</li></ul> <br />Esta característica también permite usar configuraciones del especificador de un formato de fecha y hora personalizado en la ruta de acceso. Los formatos de fecha y hora personalizados se deben especificar de uno en uno, delimitados por la palabra clave {datetime:\<specifier>}. Las entradas permitidas para \<specifier> son aaaa, MM, M, dd, d, HH, H, mm, m, ss o s. La palabra clave {datetime:\<specifier>} se puede utilizar varias veces en la ruta de acceso para formar configuraciones de fecha y hora personalizadas. <br /><br />Ejemplos: <ul><li>Ejemplo 1: cluster1/logs/{date}/{time}</li><li>Ejemplo 2: cluster1/logs/{date}</li><li>Ejemplo 3: cluster1/{client_id}/{date}/{time}</li><li>Ejemplo 4: cluster1/{datetime:ss}/{myField}, donde la consulta es: SELECT data.myField AS myField FROM Input;</li><li>Ejemplo 5: cluster1/year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}</ul><br />La marca de tiempo de la estructura de carpetas que creó usa la zona horaria UTC y no la hora local.<br /><br />La nomenclatura de los archivos usa la siguiente convención: <br /><br />{Patrón del prefijo de la ruta de acceso}/schemaHashcode_Guid_Number.extension<br /><br />Archivos de salida de ejemplo:<ul><li>Myoutput/20170901/00/45434_gguid_1.csv</li>  <li>Myoutput/20170901/01/45434_gguid_1.csv</li></ul> <br />Para obtener más información acerca de esta característica, consulte [Particionamiento de la salida de blobs personalizada en Azure Stream Analytics](stream-analytics-custom-path-patterns-blob-storage-output.md). |
| Formato de fecha | Opcional. Si el token de fecha se usa en la ruta de acceso de prefijo, puede seleccionar el formato de fecha en el que se organizan los archivos. Ejemplo: AAAA/MM/DD |
| Formato de hora | Opcional. Si el token de hora se usa en la ruta de acceso de prefijo, puede seleccionar el formato de hora en el que se organizan los archivos. Actualmente, el único valor admitido es HH. |
| Formato de serialización de eventos | Formato de serialización para los datos de salida. JSON, CSV, Avro y Parquet son compatibles. |
|Número mínimo de filas |Número mínimo de filas por lote. En el caso de Parquet, cada lote creará un archivo. El valor predeterminado actual es 2000 filas y el máximo permitido es 10 000 filas.|
|Tiempo máximo |Tiempo de espera máximo por lote. Después de este tiempo, el lote se escribirá en la salida aunque no se cumpla el requisito de filas mínimas. El valor predeterminado actual es 1 minuto y el máximo permitido es 2 horas. Si la salida del blob tiene una frecuencia de patrón de ruta de acceso, el tiempo de espera no puede ser mayor que el intervalo de tiempo de la partición.|
| Encoding    | Si usa el formato CSV o JSON, debe especificar una codificación. Por el momento, UTF-8 es el único formato de codificación compatible. |
| Delimitador   | Solo se aplica para la serialización de CSV. Stream Analytics admite un número de delimitadores comunes para la serialización de datos CSV. Los valores admitidos son la coma, punto y coma, espacio, tabulador y barra vertical. |
| Formato      | Solo se aplica para la serialización de JSON. La opción **Separado por líneas** especifica que en el formato de la salida cada objeto JSON está separado por una línea nueva. Si selecciona **Separado por líneas**, el objeto JSON se lee un objeto a la vez. El contenido total en sí mismo no sería un JSON válido. La opción **Matriz** especifica que el formato de la salida es una matriz de objetos JSON. Esta matriz se cierra cuando el trabajo se detenga o Stream Analytics haya pasado a la siguiente ventana de tiempo. En general, es preferible utilizar JSON separado por líneas, ya que no requiere ningún control especial mientras todavía se esté escribiendo en el archivo de salida. |

## <a name="blob-output-files"></a>Archivos de salida de blob

Cuando usa el almacenamiento de blobs como salida, se crea un nuevo archivo en el blob en los siguientes casos:

* Si el archivo excede el número máximo de bloques permitidos (actualmente, 50 000). Puede alcanzar el número máximo permitido de bloques sin que se alcance el tamaño máximo permitido de blobs. Por ejemplo, si la velocidad de salida es alta, puede ver más bytes por bloque y el tamaño de archivo es mayor. Si la velocidad de salida es baja, cada bloque tiene menos datos y el tamaño de archivo es menor.
* Si hay un cambio de esquema en la salida y el formato de salida requiere un esquema fijo (CSV y Avro).
* Si se reinicia un trabajo, ya sea externamente por un usuario que lo detiene e inicia, o internamente para el mantenimiento del sistema o la recuperación tras un error.
* Si la consulta está completamente particionada y se crea un nuevo archivo para cada partición de salida.
* Si el usuario elimina un archivo o un contenedor de la cuenta de almacenamiento.
* Si la salida está particionada por tiempo mediante el patrón de prefijo de ruta de acceso y se usa un nuevo blob cuando la consulta pasa a la hora siguiente.
* Si la salida tiene particiones por un campo personalizado y se crea un nuevo blob por clave de partición, si no existe.
* Si la salida tiene particiones por un campo personalizado, donde la cardinalidad de clave de partición supera el valor de 8000, y se puede crear un nuevo blob por clave de partición.

## <a name="partitioning"></a>Creación de particiones

Para la clave de partición, use los tokens {date} y {time} de los campos de evento del patrón de la ruta de acceso. Elija el formato de fecha, como AAAA/MM/DD, DD/MM/AAAA o MM-DD-AAAA. HH se usa para el formato de hora. La salida de blob puede particionarse con un solo atributo de evento personalizado {fieldname} o {datetime:\<specifier>}. El número de escritores de salida sigue las particiones de entrada para [consultas que se pueden paralelizar totalmente](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Tamaño de lote de salida

Para obtener el tamaño máximo del mensaje, consulte los [límites de Azure Storage](../azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits). El tamaño máximo del bloque de blobs es 4 MB y el número máximo del bloque de blobs es 50 000. |

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-vs-code.md)
