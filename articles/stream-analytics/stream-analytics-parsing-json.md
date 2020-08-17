---
title: Análisis de JSON y AVRO en Azure Stream Analytics
description: En este artículo se describe cómo trabajar con tipos de datos complejos como matrices, JSON, datos formateados CSV.
ms.service: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.topic: conceptual
ms.date: 01/29/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: ff3ae9a787586a4d3f7c27353aca37326be32448
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432542"
---
# <a name="parse-json-and-avro-data-in-azure-stream-analytics"></a>Análisis de datos JSON y AVRO en Azure Stream Analytics

Azure Stream Analytics admite eventos de procesamiento en formatos de datos CSV, JSON y Avro. Tanto los datos JSON como los datos Avro se pueden estructurar y contener algunos tipos complejos, tales como objetos anidados (registros) y matrices. 

>[!NOTE]
>Los archivos de AVRO creados mediante la captura del centro de eventos usan un formato específico que requiere que se utilice la característica *deserializador personalizado*. Para más información, consulte [Lectura de entradas en cualquier formato mediante deserializadores personalizados de .NET](https://docs.microsoft.com/azure/stream-analytics/custom-deserializer-examples).
>
>La deserialización de AVRO de Stream Analytics no admite el tipo de asignación. Stream Analytics no puede leer los blobs de captura de EventHub porque la captura de EventHub usa asignaciones.


## <a name="record-data-types"></a>Tipos de datos de registro
Los tipos de datos de registro se utilizan para representar las matrices JSON y Avro cuando se usan los formatos correspondientes en los flujos de datos de entrada. Estos ejemplos muestran un sensor de ejemplo, que lee los eventos de entrada en formato JSON. Este es el ejemplo de un solo evento:

```json
{
    "DeviceId" : "12345",
    "Location" :
    {
        "Lat": 47,
        "Long": 122
    },
    "SensorReadings" :
    {
        "Temperature" : 80,
        "Humidity" : 70,
        "CustomSensor01" : 5,
        "CustomSensor02" : 99,
        "SensorMetadata" : 
        {
        "Manufacturer":"ABC",
        "Version":"1.2.45"
        }
    }
}
```

### <a name="access-nested-fields-in-known-schema"></a>Acceso a campos anidados en un esquema conocido
Use la notación de punto (.) para acceder fácilmente a los campos anidados directamente desde la consulta. Por ejemplo, esta consulta selecciona las coordenadas de latitud y longitud de la propiedad Location en los datos JSON anteriores. La notación de punto se puede usar para navegar por varios niveles, como se muestra a continuación.

```SQL
SELECT
    DeviceID,
    Location.Lat,
    Location.Long,
    SensorReadings.Temperature,
    SensorReadings.SensorMetadata.Version
FROM input
```

El resultado es el siguiente:

|DeviceID|Lat|long|Temperatura|Versión|
|-|-|-|-|-|
|12345|47|122|80|1.2.45|


### <a name="select-all-properties"></a>Selección de todas las propiedades
Puede seleccionar todas las propiedades de un registro anidado usando el comodín "*". Considere el ejemplo siguiente:

```SQL
SELECT
    DeviceID,
    Location.*
FROM input
```

El resultado es el siguiente:

|DeviceID|Lat|long|
|-|-|-|
|12345|47|122|


### <a name="access-nested-fields-when-property-name-is-a-variable"></a>Acceso a los campos anidados cuando el nombre de la propiedad es una variable

Use la función [GetRecordPropertyValue](https://docs.microsoft.com/stream-analytics-query/getrecordpropertyvalue-azure-stream-analytics) si el nombre de la propiedad es una variable. De esta manera, podrá crear consultas dinámicas sin codificar los nombres de propiedad.

Por ejemplo, imagine que el flujo de datos de ejemplo tiene que **combinarse con datos de referencia** que contienen los umbrales de cada sensor de dispositivo. A continuación se muestra un fragmento de código de tales datos de referencia.

```json
{
    "DeviceId" : "12345",
    "SensorName" : "Temperature",
    "Value" : 85
},
{
    "DeviceId" : "12345",
    "SensorName" : "Humidity",
    "Value" : 65
}
```

El objetivo aquí es combinar el conjunto de datos de ejemplo de la parte superior del artículo con el de los datos de referencia y generar un evento para cada medida de sensor que sobrepase su umbral. Esto significa que, gracias a la combinación, el evento anterior puede generar varios eventos de salida si varios sensores están por encima de sus respectivos umbrales. Para conseguir resultados similares sin una combinación, consulte la sección siguiente.

```SQL
SELECT
    input.DeviceID,
    thresholds.SensorName,
    "Alert : Sensor above threshold" AS AlertMessage
FROM input      -- stream input
JOIN thresholds -- reference data input
ON
    input.DeviceId = thresholds.DeviceId
WHERE
    GetRecordPropertyValue(input.SensorReadings, thresholds.SensorName) > thresholds.Value
```

**GetRecordPropertyValue** selecciona la propiedad en *SensorReadings*, cuyo nombre coincide con el nombre de la propiedad que procede de los datos de referencia. Luego, se extrae el valor asociado de *SensorReadings*.

El resultado es el siguiente:

|DeviceID|SensorName|AlertMessage|
|-|-|-|
|12345|Humedad|Alerta: sensor por encima del umbral|

### <a name="convert-record-fields-into-separate-events"></a>Conversión de campos de registro en eventos independientes

Para convertir los campos de registro en eventos independientes, use el operador [APPLY](https://docs.microsoft.com/stream-analytics-query/apply-azure-stream-analytics) con la función [GetRecordProperties](https://docs.microsoft.com/stream-analytics-query/getrecordproperties-azure-stream-analytics).

Con los datos de ejemplo originales, se podría usar la siguiente consulta para extraer propiedades en distintos eventos.

```SQL
SELECT
    event.DeviceID,
    sensorReading.PropertyName,
    sensorReading.PropertyValue
FROM input as event
CROSS APPLY GetRecordProperties(event.SensorReadings) AS sensorReading
```

El resultado es el siguiente:

|DeviceID|SensorName|AlertMessage|
|-|-|-|
|12345|Temperatura|80|
|12345|Humedad|70|
|12345|CustomSensor01|5|
|12345|CustomSensor02|99|
|12345|SensorMetadata|[object Object]|

Mediante [WITH](https://docs.microsoft.com/stream-analytics-query/with-azure-stream-analytics), es posible enrutar esos eventos a diferentes destinos:

```SQL
WITH Stage0 AS
(
    SELECT
        event.DeviceID,
        sensorReading.PropertyName,
        sensorReading.PropertyValue
    FROM input as event
    CROSS APPLY GetRecordProperties(event.SensorReadings) AS sensorReading
)

SELECT DeviceID, PropertyValue AS Temperature INTO TemperatureOutput FROM Stage0 WHERE PropertyName = 'Temperature'
SELECT DeviceID, PropertyValue AS Humidity INTO HumidityOutput FROM Stage0 WHERE PropertyName = 'Humidity'
```

### <a name="parse-json-record-in-sql-reference-data"></a>Análisis de registros JSON en datos de referencia de SQL
Cuando se usa Azure SQL Database como datos de referencia en el trabajo, se puede tener una columna con datos en formato JSON. A continuación se muestra un ejemplo.

|DeviceID|data|
|-|-|
|12345|{"clave" : "valor1"}|
|54321|{"clave" : "valor2"}|

Puede analizar el registro JSON en la columna *Datos* escribiendo una sencilla función de JavaScript definida por el usuario.

```javascript
function parseJson(string) {
return JSON.parse(string);
}
```

Después, puede crear un paso en la consulta de Stream Analytics, como se muestra a continuación, para tener acceso a los campos de los registros JSON.

 ```SQL
 WITH parseJson as
 (
 SELECT DeviceID, udf.parseJson(sqlRefInput.Data) as metadata,
 FROM sqlRefInput
 )
 
 SELECT metadata.key
 INTO output
 FROM streamInput
 JOIN parseJson 
 ON streamInput.DeviceID = parseJson.DeviceID
```

## <a name="array-data-types"></a>Tipos de datos de matriz

Los tipos de datos de matriz son una colección ordenada de valores. A continuación se detallan algunas operaciones típicas en valores de matriz. Estos ejemplos utilizan las funciones [GetArrayElement](https://docs.microsoft.com/stream-analytics-query/getarrayelement-azure-stream-analytics), [GetArrayElements](https://docs.microsoft.com/stream-analytics-query/getarrayelements-azure-stream-analytics), [GetArrayLength](https://docs.microsoft.com/stream-analytics-query/getarraylength-azure-stream-analytics) y el operador [APPLY](https://docs.microsoft.com/stream-analytics-query/apply-azure-stream-analytics).

Este es el ejemplo de un solo evento. Tanto `CustomSensor03` como `SensorMetadata` son de tipo **matriz**:

```json
{
    "DeviceId" : "12345",
    "SensorReadings" :
    {
        "Temperature" : 80,
        "Humidity" : 70,
        "CustomSensor01" : 5,
        "CustomSensor02" : 99,
        "CustomSensor03": [12,-5,0]
     },
    "SensorMetadata":[
        {          
            "smKey":"Manufacturer",
            "smValue":"ABC"                
        },
        {
            "smKey":"Version",
            "smValue":"1.2.45"
        }
    ]
}
```

### <a name="working-with-a-specific-array-element"></a>Trabajo con un elemento de matriz específico

Selección del elemento de matriz en un índice especificado (seleccionando el primer elemento de matriz):

```SQL
SELECT
    GetArrayElement(SensorReadings.CustomSensor03, 0) AS firstElement
FROM input
```

El resultado es el siguiente:

|firstElement|
|-|
|12|

### <a name="select-array-length"></a>Selección de la longitud de matriz

```SQL
SELECT
    GetArrayLength(SensorReadings.CustomSensor03) AS arrayLength
FROM input
```

El resultado es el siguiente:

|arrayLength|
|-|
|3|

### <a name="convert-array-elements-into-separate-events"></a>Conversión de los elementos de matriz en eventos independientes

Selección de todos los elemento de matriz como eventos individuales. El operador [APPLY](https://docs.microsoft.com/stream-analytics-query/apply-azure-stream-analytics) junto con la función integrada [GetArrayElements](https://docs.microsoft.com/stream-analytics-query/getarrayelements-azure-stream-analytics) extrae todos los elementos de matriz como eventos individuales:

```SQL
SELECT
    DeviceId,
    CustomSensor03Record.ArrayIndex,
    CustomSensor03Record.ArrayValue
FROM input
CROSS APPLY GetArrayElements(SensorReadings.CustomSensor03) AS CustomSensor03Record

```

El resultado es el siguiente:

|deviceId|ArrayIndex|ArrayValue|
|-|-|-|
|12345|0|12|
|12345|1|-5|
|12345|2|0|

```SQL
SELECT   
    i.DeviceId, 
    SensorMetadataRecords.ArrayValue.smKey as smKey,
    SensorMetadataRecords.ArrayValue.smValue as smValue
FROM input i
CROSS APPLY GetArrayElements(SensorMetadata) AS SensorMetadataRecords
 ```
 
El resultado es el siguiente:

|deviceId|smKey|smValue|
|-|-|-|
|12345|Fabricante|ABC|
|12345|Versión|1.2.45|

Si los campos extraídos deben aparecer en columnas, es posible dinamizar el conjunto de datos mediante la sintaxis [WITH](https://docs.microsoft.com/stream-analytics-query/with-azure-stream-analytics) además de la operación [JOIN](https://docs.microsoft.com/stream-analytics-query/join-azure-stream-analytics). Esa combinación requerirá una condición de [límite de tiempo](https://docs.microsoft.com/stream-analytics-query/join-azure-stream-analytics#BKMK_DateDiff) que impida la duplicación:

```SQL
WITH DynamicCTE AS (
    SELECT   
        i.DeviceId,
        SensorMetadataRecords.ArrayValue.smKey as smKey,
        SensorMetadataRecords.ArrayValue.smValue as smValue
    FROM input i
    CROSS APPLY GetArrayElements(SensorMetadata) AS SensorMetadataRecords 
)

SELECT
    i.DeviceId,
    i.Location.*,
    V.smValue AS 'smVersion',
    M.smValue AS 'smManufacturer'
FROM input i
LEFT JOIN DynamicCTE V ON V.smKey = 'Version' and V.DeviceId = i.DeviceId AND DATEDIFF(minute,i,V) BETWEEN 0 AND 0 
LEFT JOIN DynamicCTE M ON M.smKey = 'Manufacturer' and M.DeviceId = i.DeviceId AND DATEDIFF(minute,i,M) BETWEEN 0 AND 0
```

El resultado es el siguiente:

|deviceId|Lat|long|smVersion|smManufacturer|
|-|-|-|-|-|
|12345|47|122|1.2.45|ABC|

## <a name="see-also"></a>Consulte también
[Tipos de datos en Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/data-types-azure-stream-analytics)
