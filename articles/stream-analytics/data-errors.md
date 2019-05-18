---
title: Errores de datos de registro de diagnóstico de Azure Stream Analytics
description: En este artículo se explica la entrada diferente y errores de datos de salida que pueden producirse al usar Azure Stream Analytics.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/09/2019
ms.openlocfilehash: b00eb12092838746f4bfe16f00eac55df9224b09
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65607234"
---
# <a name="azure-stream-analytics-data-errors"></a>Errores de datos de Azure Stream Analytics

Cuando hay una discrepancia en los datos que se procesan mediante un trabajo de Azure Stream Analytics, Stream Analytics envía un evento de error de datos a los registros de diagnóstico. Stream Analytics escribe información detallada y eventos de ejemplo, en sus registros de diagnóstico cuando se producen errores de datos. También se proporciona un resumen de esta información a través de las notificaciones del portal para algunos errores.

En este artículo se describe los diferentes tipos de errores, las causas y los detalles de registro de diagnóstico de errores de los datos de entrada y salida.

## <a name="diagnostic-log-schema"></a>Esquema de registro de diagnóstico

Consulte [solucionar problemas de Azure Stream Analytics mediante el uso de los registros de diagnóstico](stream-analytics-job-diagnostic-logs.md#diagnostics-logs-schema) para ver el esquema para los registros de diagnóstico. El siguiente JSON es un valor de ejemplo para el **propiedades** campo de un registro de diagnóstico para un error de datos.

```json
{
    "Source": "InputTelemetryData",
    "Type": "DataError",
    "DataErrorType": "InputDeserializerError.InvalidData",
    "BriefMessage": "Json input stream should either be an array of objects or line separated objects. Found token type: Integer",
    "Message": "Input Message Id: https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt Error: Json input stream should either be an array of objects or line separated objects. Found token type: Integer",
    "ExampleEvents": "[\"1,2\\\\u000d\\\\u000a3,4\\\\u000d\\\\u000a5,6\"]",
    "FromTimestamp": "2019-03-22T22:34:18.5664937Z",
    "ToTimestamp": "2019-03-22T22:34:18.5965248Z",
    "EventCount": 1
}
```

## <a name="input-data-errors"></a>Errores de datos de entrada

### <a name="inputdeserializererrorinvalidcompressiontype"></a>InputDeserializerError.InvalidCompressionType

* Causa: El tipo de compresión de entrada seleccionado no coincide con los datos.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador de mensaje de entrada. Centro de eventos, el identificador es el PartitionId, desplazamiento y el número de secuencia.

**Mensaje de error**

```json
"BriefMessage": "Unable to decompress events from resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt'. Please ensure compression setting fits the data being processed."
```

### <a name="inputdeserializererrorinvalidheader"></a>InputDeserializerError.InvalidHeader

* Causa: El encabezado de datos de entrada no es válido. Por ejemplo, un archivo CSV tiene las columnas con nombres duplicados.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Invalid CSV Header for resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt'. Please make sure there are no duplicate field names."
```

### <a name="inputdeserializererrormissingcolumns"></a>InputDeserializerError.MissingColumns

* Causa: Las columnas de entrada definidas con CREATE TABLE o a través de TIMESTAMP BY no existe.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Nombres de las columnas que faltan. 
   * Carga útil real hasta unos pocos kilobytes.

**Mensajes de error**

```json
"BriefMessage": "Could not deserialize the input event(s) from resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt' as Csv. Some possible reasons: 1) Malformed events 2) Input source configured with incorrect serialization format" 
```

```json
"Message": "Missing fields specified in query or in create table. Fields expected:ColumnA Fields found:ColumnB"
```

### <a name="inputdeserializererrortypeconversionerror"></a>InputDeserializerError.TypeConversionError

* Causa: No se puede convertir la entrada en el tipo especificado en la instrucción CREATE TABLE.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Nombre de la columna y el tipo esperado.

**Mensajes de error**

```json
"BriefMessage": "Could not deserialize the input event(s) from resource '''https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt ' as Csv. Some possible reasons: 1) Malformed events 2) Input source configured with incorrect serialization format" 
```

```json
"Message": "Unable to convert column: dateColumn to expected type."
```

### <a name="inputdeserializererrorinvaliddata"></a>InputDeserializerError.InvalidData

* Causa: Datos de entrada no están en el formato correcto. Por ejemplo, la entrada no es un JSON válido.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Carga útil real hasta algunos kilobytes.

**Mensajes de error**

```json
"BriefMessage": "Json input stream should either be an array of objects or line separated objects. Found token type: String"
```

```json
"Message": "Json input stream should either be an array of objects or line separated objects. Found token type: String"
```

### <a name="invalidinputtimestamp"></a>InvalidInputTimeStamp

* Causa: El valor de la expresión por la marca de tiempo no se puede convertir a datetime.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Mensaje de error. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Unable to get timestamp for resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt ' due to error 'Cannot convert string to datetime'"
```

### <a name="invalidinputtimestampkey"></a>InvalidInputTimeStampKey

* Causa: El valor de TIMESTAMP BY OVER timestampColumn es NULL.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * La carga real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Unable to get value of TIMESTAMP BY OVER COLUMN"
```

### <a name="lateinputevent"></a>LateInputEvent

* Causa: La diferencia entre el tiempo de aplicación y la hora de llegada es mayor que el período de tolerancia de llegada tardía.
* Proporcionados notificación del portal: No
* Nivel de registro de diagnóstico: Información
* Detalles del registro
   * Tiempo de aplicación y la hora de llegada. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Input event with application timestamp '2019-01-01' and arrival time '2019-01-02' was sent later than configured tolerance."
```

### <a name="earlyinputevent"></a>EarlyInputEvent

* Causa: La diferencia entre el tiempo de aplicación y la hora de llegada es mayor que 5 minutos.
* Proporcionados notificación del portal: No
* Nivel de registro de diagnóstico: Información
* Detalles del registro
   * Tiempo de aplicación y la hora de llegada. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Input event arrival time '2019-01-01' is earlier than input event application timestamp '2019-01-02' by more than 5 minutes."
```

### <a name="outoforderevent"></a>OutOfOrderEvent

* Causa: Evento se considera fuera de servicio según la ventana de tolerancia de desorden definida.
* Proporcionados notificación del portal: No
* Nivel de registro de diagnóstico: Información
* Detalles del registro
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"Message": "Out of order event(s) received."
```

## <a name="output-data-errors"></a>Errores de datos de salida

### <a name="outputdataconversionerrorrequiredcolumnmissing"></a>OutputDataConversionError.RequiredColumnMissing

* Causa: La columna necesaria para la salida no existe. Por ejemplo, existe una columna definida como does't PartitionKey de la tabla de Azure.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Nombre de la columna y el identificador de registro o parte del registro.

**Mensaje de error**

```json
"Message": "The output record does not contain primary key property: [deviceId] Ensure the query output contains the column [deviceId] with a unique non-empty string less than '255' characters."
```

### <a name="outputdataconversionerrorcolumnnameinvalid"></a>OutputDataConversionError.ColumnNameInvalid

* Causa: El valor de columna no se ajusta con la salida. Por ejemplo, el nombre de columna no es una columna de tabla de Azure válida.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Nombre de la columna y el identificador del registro o parte del registro.

**Mensaje de error**

```json
"Message": "Invalid property name #deviceIdValue. Please refer MSDN for Azure table property naming convention."
```

### <a name="outputdataconversionerrortypeconversionerror"></a>OutputDataConversionError.TypeConversionError

* Causa: No se puede convertir una columna a un tipo válido en la salida. Por ejemplo, el valor de columna no es compatible con las restricciones o tipo definido en la tabla de SQL.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Nombre de la columna.
   * Identificador del registro o en la parte del registro.

**Mensaje de error**

```json
"Message": "The column [id] value null or its type is invalid. Ensure to provide a unique non-empty string less than '255' characters."
```

### <a name="outputdataconversionerrorrecordexceededsizelimit"></a>OutputDataConversionError.RecordExceededSizeLimit

* Causa: El valor del mensaje es mayor que el tamaño de salida admitidos. Por ejemplo, un registro es mayor que 1 MB para la salida de un centro de eventos.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Identificador del registro o en la parte del registro.

**Mensaje de error**

```json
"BriefMessage": "Single output event exceeds the maximum message size limit allowed (262144 bytes) by Event Hub."
```

### <a name="outputdataconversionerrorduplicatekey"></a>OutputDataConversionError.DuplicateKey

* Causa: Un registro ya contiene una columna con el mismo nombre que una columna del sistema. Por ejemplo, la salida de CosmosDB con una columna denominada Id. cuando la columna de identificador es una columna diferente.
* Proporcionados notificación del portal: Sí
* Nivel de registro de diagnóstico: Advertencia
* Detalles del registro
   * Nombre de la columna.
   * Identificador del registro o en la parte del registro.

```json
"BriefMessage": "Column 'devicePartitionKey' is being mapped to multiple columns."
```

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de Azure Stream Analytics mediante el uso de los registros de diagnóstico](stream-analytics-job-diagnostic-logs.md)

* [Comprender la supervisión de trabajos de Stream Analytics y cómo supervisar consultas](stream-analytics-monitoring.md)
