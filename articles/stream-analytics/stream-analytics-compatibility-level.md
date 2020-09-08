---
title: Niveles de compatibilidad de Azure Stream Analytics
description: Obtenga información acerca de cómo establecer un nivel de compatibilidad para un trabajo de análisis de Azure Stream Analytics y de los principales cambios en el nivel de compatibilidad más reciente.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: a82f3c347c75d658e3e7ec52d51107f5a240ee5b
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056523"
---
# <a name="compatibility-level-for-azure-stream-analytics-jobs"></a>Nivel de compatibilidad de los trabajos de Azure Stream Analytics

En este artículo se explica la opción de nivel de compatibilidad de Azure Stream Analytics. Stream Analytics es un servicio administrado con actualizaciones regulares de características y mejoras de rendimiento. La mayoría de las actualizaciones de runtimes del servicio se ponen a disposición de los usuarios finales automáticamente. 

Pero algunas funcionalidades nuevas del servicio pueden presentar un cambio importante, como un cambio de comportamiento de un trabajo existente o un cambio en la forma en que los trabajos en ejecución usan los datos. Puede mantener los trabajos de Stream Analytics existentes en ejecución sin cambios importantes si rebaja la opción de nivel de compatibilidad. Cuando esté listo para los comportamientos de runtime más recientes, puede aumentar el nivel de compatibilidad. 

## <a name="choose-a-compatibility-level"></a>Elegir un nivel de compatibilidad

Los niveles de compatibilidad controlan el comportamiento en tiempo de ejecución de un trabajo de Stream Analytics. 

Actualmente, Azure Stream Analytics admite tres niveles de compatibilidad:

* 1.0: el nivel de compatibilidad original, se incorporó durante la disponibilidad general de Azure Stream Analytics hace varios años.
* 1.1: comportamiento anterior
* 1.2: comportamiento más nuevo con las mejoras más recientes

Al crear un nuevo trabajo de Stream Analytics, se recomienda hacerlo con el nivel de compatibilidad más reciente. Inicie el diseño del trabajo basándose en los comportamientos más recientes a fin de evitar la incorporación de cambio y complejidad más adelante.

## <a name="set-the-compatibility-level"></a>Definición del nivel de compatibilidad

Puede establecer el nivel de compatibilidad de un trabajo de Stream Analytics desde Azure Portal o con la [llamada a la API de REST del trabajo Create](/rest/api/streamanalytics/stream-analytics-job).

Para actualizar el nivel de compatibilidad del trabajo en Azure Portal:

1. Use [Azure Portal](https://portal.azure.com) para buscar el trabajo de Stream Analytics.
2. **Detenga** el trabajo antes de actualizar el nivel de compatibilidad. No se puede actualizar el nivel de compatibilidad si el trabajo está en un estado de ejecución.
3. En el encabezado **Configurar**, seleccione **Nivel de compatibilidad**.
4. Seleccione el valor de nivel de compatibilidad que quiera.
5. Seleccione **Guardar** en la parte inferior de la página.

![Nivel de compatibilidad de Stream Analytics en Azure Portal](media/stream-analytics-compatibility-level/stream-analytics-compatibility.png)

Cuando se actualiza el nivel de compatibilidad, el compilador de T valida el trabajo con la sintaxis que se corresponde con el nivel de compatibilidad seleccionado.

## <a name="compatibility-level-12"></a>Nivel de compatibilidad 1.2

En el nivel de compatibilidad 1.2 se han incorporado los siguientes cambios de importancia:

###  <a name="amqp-messaging-protocol"></a>Protocolo de mensajería de AMQP

**Nivel 1.2**: Azure Stream Analytics usa el protocolo de mensajería [Advanced Message Queuing Protocol (AMQP)](../service-bus-messaging/service-bus-amqp-overview.md) para escribir en colas y temas de Service Bus. AMQP le permite construir aplicaciones híbridas, entre plataformas, utilizando un protocolo estándar abierto.

### <a name="geospatial-functions"></a>Funciones geoespaciales

**Niveles anteriores:** Azure Stream Analytics usaba cálculos geográficos.

**Nivel 1.2:** Azure Stream Analytics permite calcular coordenadas geográficas proyectadas geométricas. No hay ningún cambio en la firma de las funciones geoespaciales, aunque su semántica es ligeramente diferente, lo que permite un cálculo más preciso que antes.

Azure Stream Analytics admite la indexación de datos de referencia geoespaciales. Los datos de referencia que contienen elementos geoespaciales se pueden indexar para un cálculo de combinación más rápido.

Las funciones geoespaciales actualizadas aportan toda la expresividad del formato geoespacial Well Known Text (WKT). Puede especificar otros componentes geoespaciales que anteriormente no eran compatibles con GeoJson.

Para obtener más información, vea [Updates to geospatial features in Azure Stream Analytics – Cloud and IoT Edge](https://azure.microsoft.com/blog/updates-to-geospatial-functions-in-azure-stream-analytics-cloud-and-iot-edge/) (Actualizaciones de características geoespaciales en Azure Stream Analytics: nube e IoT Edge).

### <a name="parallel-query-execution-for-input-sources-with-multiple-partitions"></a>Ejecución de consultas en paralelo para los orígenes de entrada con varias particiones

**Niveles anteriores:** las consultas de Azure Stream Analytics requerían el uso de la cláusula PARTITION BY para ejecutar en paralelo el procesamiento de consultas entre particiones de origen de entrada.

**Nivel 1.2:** si la lógica de consulta se puede ejecutar en paralelo entre particiones de origen de entrada, Azure Stream Analytics crea instancias de consulta independientes y ejecuta los cálculos en paralelo.

### <a name="native-bulk-api-integration-with-cosmosdb-output"></a>Integración nativa de API Bulk con la salida de CosmosDB

**Niveles anteriores:** el comportamiento de upsert era *insertar o combinar*.

**Nivel 1.2:** la integración nativa de API Bulk con la salida de CosmosDB maximiza el rendimiento y controla con eficacia las solicitudes de limitación. Para obtener más información, vea [la página sobre la salida de Azure Stream Analytics a Azure Cosmos DB](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-documentdb-output#improved-throughput-with-compatibility-level-12).

El comportamiento de upsert es *insertar o reemplazar*.

### <a name="datetimeoffset-when-writing-to-sql-output"></a>DateTimeOffset al escribir en la salida de SQL

**Niveles anteriores:** los tipos [DateTimeOffset](https://docs.microsoft.com/sql/t-sql/data-types/datetimeoffset-transact-sql?view=sql-server-2017) se ajustaban a UTC.

**Nivel 1.2:** DateTimeOffset ya no se ajusta.

### <a name="long-when-writing-to-sql-output"></a>Largo al escribir en la salida de SQL

**Niveles anteriores:** los valores se truncaron en función del tipo de destino.

**Nivel 1.2:** los valores que no encajan en el tipo de destino se controlan conforme a la directiva de error de salida.

### <a name="record-and-array-serialization-when-writing-to-sql-output"></a>Serialización de registros y matrices al escribir en la salida de SQL

**Niveles anteriores:** Los registros se escribieron como "Record" y las matrices se escribieron como "Array".

**Nivel 1.2:** Los registros y matrices se serializan en formato JSON.

### <a name="strict-validation-of-prefix-of-functions"></a>Validación estricta de prefijo de funciones

**Niveles anteriores:** no había validación estricta de prefijos de función.

**Nivel 1.2:** Azure Stream Analytics tiene una validación estricta de prefijos de función. La adición de un prefijo a una función integrada produce un error. Por ejemplo, `myprefix.ABS(…)` no es compatible.

La adición de un prefijo a agregados integrados también produce un error. Por ejemplo, `myprefix.SUM(…)` no es compatible.

El empleo del prefijo "system" para cualquier función definida por el usuario produce un error.

### <a name="disallow-array-and-object-as-key-properties-in-cosmos-db-output-adapter"></a>No se permiten Array ni Object como propiedades de clave en el adaptador de salida de Cosmos DB

**Niveles anteriores:** los tipos Array y Object se admitían como una propiedad de clave.

**Nivel 1.2:** los tipos Array y Object ya no se admiten como una propiedad de clave.

## <a name="compatibility-level-11"></a>Nivel de compatibilidad 1.1

En el nivel de compatibilidad 1.1 se introdujeron los siguientes cambios principales:

### <a name="service-bus-xml-format"></a>Formato XML de Service Bus

**Nivel 1.0:** Azure Stream Analytics usaba DataContractSerializer, por lo que el contenido del mensaje incluía etiquetas XML. Por ejemplo:

`@\u0006string\b3http://schemas.microsoft.com/2003/10/Serialization/\u0001{ "SensorId":"1", "Temperature":64\}\u0001`

**Nivel 1.1:** El contenido del mensaje contiene directamente la secuencia sin ninguna etiqueta adicional. Por ejemplo: `{ "SensorId":"1", "Temperature":64}`

### <a name="persisting-case-sensitivity-for-field-names"></a>Distinción entre mayúsculas y minúsculas persistente para nombres de campo

**Nivel 1.0:** Los nombres de campo se cambiaban a minúsculas cuando los procesaba el motor de Azure Stream Analytics.

**Nivel 1.1:** la distinción entre mayúsculas y minúsculas se mantiene para nombres de campo que procesa el motor de Azure Stream Analytics.

> [!NOTE]
> La distinción entre mayúsculas y minúsculas persistente aún no está disponible para trabajos de Stream Analytics hospedados mediante un entorno de Edge. En consecuencia, todos los nombres de campo se convierten a minúsculas si el trabajo se hospeda en Edge.

### <a name="floatnandeserializationdisabled"></a>FloatNaNDeserializationDisabled

**Nivel 1.0:** El comando CREATE TABLE no filtraba eventos con NaN (por ejemplo, Infinity, -Infinity) en una columna de tipo FLOAT, porque están fuera del intervalo documentado para esos números.

**Nivel 1.1:** El comando CREATE TABLE le permite especificar un esquema seguro. El motor de Stream Analytics valida que los datos se ajustan a este esquema. Con este modelo, el comando puede filtrar eventos con valores NaN.

### <a name="disable-automatic-conversion-of-datetime-strings-to-datetime-type-at-ingress-for-json"></a>Deshabilitación de la conversión automática de cadenas de fecha y hora a un tipo DateTime en la entrada de JSON

**Nivel 1.0:** El analizador JSON convertirá automáticamente los valores de cadena con información de fecha, hora y zona al tipo DATETIME en la entrada, de modo que el valor pierde inmediatamente su formato original y la información de zona horaria. Como esto se hace en la entrada, incluso si ese campo no se usó en la consulta, se convierte en el tipo DateTime de UTC.

**Nivel 1.1:** Ya no existe la conversión automática de valores de cadena con información de fecha, hora y zona al tipo DATETIME. Como resultado, se conserva la información de zona horaria y el formato original. Sin embargo, si se usa el campo NVARCHAR (MAX) en la consulta como parte de una expresión DATETIME (por ejemplo, la función DATEADD), se convierte al tipo DATETIME para realizar el cálculo y se pierde su formato original.

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de las conexiones de entrada de Azure Stream Analytics](stream-analytics-troubleshoot-input.md)
* [Estado de recursos de Stream Analytics](stream-analytics-resource-health.md)
