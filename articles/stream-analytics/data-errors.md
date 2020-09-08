---
title: Errores de datos de registro de recursos de Azure Stream Analytics
description: En este artículo se explican los distintos errores de datos de entrada y salida que pueden producirse al usar Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: troubleshooting
ms.date: 08/07/2020
ms.openlocfilehash: 1c649499fd9eaedac0ca4ff9c182e13a9da223ef
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053157"
---
# <a name="azure-stream-analytics-data-errors"></a>Errores de datos de Azure Stream Analytics

Los errores de datos son errores que se producen al procesar los datos.  Estos errores suelen producirse durante la deserialización de datos, la serialización y las operaciones de escritura.  Cuando se producen errores de datos, Stream Analytics escribe información detallada y eventos de ejemplo en sus registros de recursos. Habilite los registros de diagnóstico en su trabajo para obtener estos detalles adicionales. En algunos casos, también se proporciona un resumen de esta información mediante las notificaciones del portal.

En este artículo se describen los diferentes tipos de errores, las causas y los detalles de registro de recursos de errores de datos de entrada y salida.

## <a name="resource-logs-schema"></a>Esquema de registros de recursos

Para ver el esquema de los registros de recursos, consulte [Solución de problemas de Azure Stream Analytics mediante registros de diagnóstico](stream-analytics-job-diagnostic-logs.md#resource-logs-schema). El siguiente código JSON es un valor de ejemplo para el campo **Propiedades** de un registro de recursos para un error de datos.

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
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Los mensajes con los errores de deserialización, como el tipo de compresión no válido, se eliminan de la entrada.
* Detalles del registro
   * Identificador de mensaje de entrada. Para el centro de eventos, el identificador es el PartitionId, el desplazamiento y el número de secuencia.

**Mensaje de error**

```json
"BriefMessage": "Unable to decompress events from resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt'. Please ensure compression setting fits the data being processed."
```

### <a name="inputdeserializererrorinvalidheader"></a>InputDeserializerError.InvalidHeader

* Causa: El encabezado de los datos de entrada no es válido. Por ejemplo, un archivo CSV tiene columnas con nombres duplicados.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Los mensajes con los errores de deserialización, como un encabezado no válido, se eliminan de la entrada.
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Invalid CSV Header for resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt'. Please make sure there are no duplicate field names."
```

### <a name="inputdeserializererrormissingcolumns"></a>InputDeserializerError.MissingColumns

* Causa: Las columnas de entrada definidas con CREATE TABLE o a través de TIMESTAMP BY no existen.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Los eventos con las columnas que faltan se eliminan de la entrada.
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Nombres de las columnas que faltan. 
   * Carga útil real hasta algunos kilobytes.

**Mensajes de error**

```json
"BriefMessage": "Could not deserialize the input event(s) from resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt' as Csv. Some possible reasons: 1) Malformed events 2) Input source configured with incorrect serialization format" 
```

```json
"Message": "Missing fields specified in query or in create table. Fields expected:ColumnA Fields found:ColumnB"
```

### <a name="inputdeserializererrortypeconversionerror"></a>InputDeserializerError.TypeConversionError

* Causa: No se puede convertir la entrada al tipo especificado en la instrucción CREATE TABLE.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Los eventos con error de conversión de tipo se eliminan de la entrada.
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

* Causa: Los datos de entrada no tienen el formato correcto. Por ejemplo, la entrada no es un JSON válido.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Todos los eventos en el mensaje después de que se haya encontrado un error de datos no válidos se eliminan de la entrada.
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

* Causa: El valor de la expresión TIMESTAMP BY no se puede convertir a datetime.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Los eventos con marca de tiempo de entrada no válida se eliminan de la entrada.
* Detalles del registro
   * Identificador de mensaje de entrada. 
   * Mensaje de error. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Unable to get timestamp for resource 'https:\\/\\/exampleBlob.blob.core.windows.net\\/inputfolder\\/csv.txt ' due to error 'Cannot convert string to datetime'"
```

### <a name="invalidinputtimestampkey"></a>InvalidInputTimeStampKey

* Causa: El valor de timestampColumn TIMESTAMP BY OVER es NULL.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto: Los eventos con clave de marca de tiempo de entrada no válida se eliminan de la entrada.
* Detalles del registro
   * La carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Unable to get value of TIMESTAMP BY OVER COLUMN"
```

### <a name="lateinputevent"></a>LateInputEvent

* Causa: La diferencia entre el tiempo de aplicación y la hora de llegada es mayor que la ventana de tolerancia de llegada tardía.
* Notificación del portal proporcionada: No
* Nivel de registro de recursos: Información
* Impacto:  Los eventos de entrada con retardo se controlan según la opción "Controlar otros eventos" de la sección Ordenación de eventos de la configuración del trabajo. Para más información, consulte [Directivas de control de tiempo](https://docs.microsoft.com/stream-analytics-query/time-skew-policies-azure-stream-analytics).
* Detalles del registro
   * Tiempo de aplicación y hora de llegada. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Input event with application timestamp '2019-01-01' and arrival time '2019-01-02' was sent later than configured tolerance."
```

### <a name="earlyinputevent"></a>EarlyInputEvent

* Causa: La diferencia entre el tiempo de aplicación y la hora de llegada es mayor que 5 minutos.
* Notificación del portal proporcionada: No
* Nivel de registro de recursos: Información
* Impacto:  Los eventos de entrada tempranos se controlan según la opción "Controlar otros eventos" de la sección Ordenación de eventos de la configuración del trabajo. Para más información, consulte [Directivas de control de tiempo](https://docs.microsoft.com/stream-analytics-query/time-skew-policies-azure-stream-analytics).
* Detalles del registro
   * Tiempo de aplicación y hora de llegada. 
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"BriefMessage": "Input event arrival time '2019-01-01' is earlier than input event application timestamp '2019-01-02' by more than 5 minutes."
```

### <a name="outoforderevent"></a>OutOfOrderEvent

* Causa: Se considera que el evento está desorganizado según la ventana de tolerancia de desorganización definida.
* Notificación del portal proporcionada: No
* Nivel de registro de recursos: Información
* Impacto:  Los eventos de entrada fuera de secuencia se controlan según la opción "Controlar otros eventos" de la sección Ordenación de eventos de la configuración del trabajo. Para más información, consulte [Directivas de control de tiempo](https://docs.microsoft.com/stream-analytics-query/time-skew-policies-azure-stream-analytics).
* Detalles del registro
   * Carga útil real hasta algunos kilobytes.

**Mensaje de error**

```json
"Message": "Out of order event(s) received."
```

## <a name="output-data-errors"></a>Errores de datos de salida

### <a name="outputdataconversionerrorrequiredcolumnmissing"></a>OutputDataConversionError.RequiredColumnMissing

* Causa: La columna necesaria para la salida no existe. Por ejemplo, no existe ninguna columna definida como Azure Table PartitionKey.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto:  Todos los errores de conversión de datos de salida que incluyen la falta de una columna obligatoria se controlan según la opción [Directiva de salida de datos](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-output-error-policy).
* Detalles del registro
   * Nombre de la columna y el identificador de registro o parte del registro.

**Mensaje de error**

```json
"Message": "The output record does not contain primary key property: [deviceId] Ensure the query output contains the column [deviceId] with a unique non-empty string less than '255' characters."
```

### <a name="outputdataconversionerrorcolumnnameinvalid"></a>OutputDataConversionError.ColumnNameInvalid

* Causa: El valor de la columna no se ajusta a la salida. Por ejemplo, el nombre de la columna no es una columna de tabla de Azure válida.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto:  Todos los errores de conversión de datos de salida que incluyen un nombre de columna no válido se controlan según la opción [Directiva de salida de datos](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-output-error-policy).
* Detalles del registro
   * Nombre de la columna y el identificador del registro o parte del registro.

**Mensaje de error**

```json
"Message": "Invalid property name #deviceIdValue. Please refer MSDN for Azure table property naming convention."
```

### <a name="outputdataconversionerrortypeconversionerror"></a>OutputDataConversionError.TypeConversionError

* Causa: No se puede convertir una columna a un tipo válido en la salida. Por ejemplo, el valor de la columna no es compatible con las restricciones o el tipo definido en la tabla de SQL.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto:  Todos los errores de conversión de datos de salida que incluyen un error de conversión de tipo se controlan según la opción [Directiva de salida de datos](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-output-error-policy).
* Detalles del registro
   * Nombre de la columna.
   * Identificador del registro o parte del registro.

**Mensaje de error**

```json
"Message": "The column [id] value null or its type is invalid. Ensure to provide a unique non-empty string less than '255' characters."
```

### <a name="outputdataconversionerrorrecordexceededsizelimit"></a>OutputDataConversionError.RecordExceededSizeLimit

* Causa: El valor del mensaje es mayor que el tamaño de salida admitido. Por ejemplo, un registro es mayor que 1 MB para la salida de un centro de eventos.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto:  Todos los errores de conversión de datos de salida que incluyen un límite de tamaño de registro superado se controlan según la opción [Directiva de salida de datos](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-output-error-policy).
* Detalles del registro
   * Identificador del registro o parte del registro.

**Mensaje de error**

```json
"BriefMessage": "Single output event exceeds the maximum message size limit allowed (262144 bytes) by Event Hub."
```

### <a name="outputdataconversionerrorduplicatekey"></a>OutputDataConversionError.DuplicateKey

* Causa: Un registro ya contiene una columna con el mismo nombre que una columna del sistema. Por ejemplo, una salida de CosmosDB con un identificador con nombre de columna cuando la columna del identificador es una columna diferente.
* Notificación del portal proporcionada: Sí
* Nivel de registro de recursos: Advertencia
* Impacto:  Todos los errores de conversión de datos de salida que incluyen una clave duplicada se controlan según la opción [Directiva de salida de datos](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-output-error-policy).
* Detalles del registro
   * Nombre de la columna.
   * Identificador del registro o parte del registro.

```json
"BriefMessage": "Column 'devicePartitionKey' is being mapped to multiple columns."
```

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de Azure Stream Analytics mediante registros de diagnóstico](stream-analytics-job-diagnostic-logs.md)

* [Descripción de la supervisión del trabajo de Stream Analytics y cómo supervisar consultas](stream-analytics-monitoring.md)
