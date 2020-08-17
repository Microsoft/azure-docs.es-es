---
title: Solución de problemas con entradas de Azure Stream Analytics
description: En este artículo se describen técnicas para solucionar problemas de las conexiones de entrada en los trabajos de Azure Stream Analytics.
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: troubleshooting
ms.date: 05/01/2020
ms.custom: seodec18
ms.openlocfilehash: f4f79a28dbe8a49e608ca6fae1781a1e19646619
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448875"
---
# <a name="troubleshoot-input-connections"></a>Solución de problemas de conexiones de entrada

En este artículo se describen los problemas comunes relacionados con las conexiones de entrada de Azure Stream Analytics, cómo solucionar problemas de entrada y cómo corregir los problemas. Muchos pasos de solución de problemas requieren que los registros de recursos estén habilitados para el trabajo de Stream Analytics. Si los registros de recursos no están habilitados, consulte [Solución de problemas de Azure Stream Analytics mediante registros de recursos](stream-analytics-job-diagnostic-logs.md).

## <a name="input-events-not-received-by-job"></a>El trabajo no recibe los eventos de entrada 

1.  Pruebe la conectividad de entrada y salida. Compruebe la conectividad a entradas y salidas con el botón **Probar conexión** para cada entrada y salida.

2.  Examine los datos de entrada.

    1. Use el botón [**Datos de ejemplo**](stream-analytics-sample-data-input.md) para cada entrada. Descargue los datos de ejemplo de entrada.
        
    1. Inspeccione los datos de ejemplo para entender el esquema y los [tipos de datos](https://docs.microsoft.com/stream-analytics-query/data-types-azure-stream-analytics).
    
    1. Compruebe [métricas del centro de eventos](../event-hubs/event-hubs-metrics-azure-monitor.md) para asegurarse de que se envían los eventos. Las métricas de mensajes deben ser mayores que cero si Event Hubs está recibiendo mensajes.

3.  Asegúrese de que ha seleccionado un intervalo de tiempo en la versión preliminar de la entrada. Elija **Seleccionar intervalo de tiempo** y, a continuación, escriba una duración de ejemplo antes de probar la consulta.

## <a name="malformed-input-events-causes-deserialization-errors"></a>Los eventos de entrada con formato incorrecto provocan errores de deserialización 

Los problemas de deserialización surgen cuando el flujo de entrada del trabajo de Stream Analytics contiene mensajes con un formato incorrecto. Por ejemplo, que un mensaje tenga un formato incorrecto podría deberse a la falta de un paréntesis o una llave en un objeto JSON, o a un formato de marca de tiempo incorrecto en un campo de tiempo. 
 
Cuando un trabajo de Stream Analytics recibe un mensaje con formato incorrecto desde una entrada, descarta el mensaje y le notifica con una advertencia. Se muestra un símbolo de advertencia en el icono **Entradas** del trabajo de Stream Analytics. El siguiente símbolo de advertencia persiste mientras el trabajo está en estado de ejecución:

![Icono de entradas en Azure Stream Analytics](media/stream-analytics-malformed-events/stream-analytics-inputs-tile.png)

Habilite los registros de recursos para ver los detalles del error y el mensaje (carga) que provocó el error. Hay varios motivos por los que se pueden producir errores de deserialización. Para más información sobre errores de deserialización específicos, vea [Errores de los datos de entrada](data-errors.md#input-data-errors). Si los registros de recursos no están habilitados, habrá una breve notificación disponible en Azure Portal.

![Notificación de advertencia de detalles de entrada](media/stream-analytics-malformed-events/warning-message-with-offset.png)

En los casos en los que la carga del mensaje es mayor que 32 KB o está en formato binario, ejecute el código CheckMalformedEvents.cs disponible en el [repositorio de ejemplos de GitHub](https://github.com/Azure/azure-stream-analytics/tree/master/Samples/CheckMalformedEventsEH). Este código lee el identificador de la partición y el desplazamiento e imprime los datos de este último. 

## <a name="job-exceeds-maximum-event-hub-receivers"></a>El trabajo excede el máximo de receptores del centro de eventos

Un procedimiento recomendado para el uso de Event Hubs es usar varios grupos de consumidores para la escalabilidad del trabajo. El número de lectores en el trabajo de Stream Analytics para una entrada específica afecta a la cantidad de lectores en un grupo de consumidores único. El número exacto de destinatarios se basa en los detalles de implementación interna para la lógica de la topología de escalado horizontal y no está expuesto de forma externa. El número de lectores puede cambiar cuando inicia un trabajo o durante las actualizaciones del mismo.

Los mensajes de error siguientes se muestran cuando el número de receptores supera el máximo. El mensaje de error incluye una lista de las conexiones existentes realizadas al centro de eventos en un grupo de consumidores. La etiqueta `AzureStreamAnalytics` indica que las conexiones sea realizan desde servicio de streaming de Azure.

```
The streaming job failed: Stream Analytics job has validation errors: Job will exceed the maximum amount of Event Hub Receivers.

The following information may be helpful in identifying the connected receivers: Exceeded the maximum number of allowed receivers per partition in a consumer group which is 5. List of connected receivers – 
AzureStreamAnalytics_c4b65e4a-f572-4cfc-b4e2-cf237f43c6f0_1, 
AzureStreamAnalytics_c4b65e4a-f572-4cfc-b4e2-cf237f43c6f0_1, 
AzureStreamAnalytics_c4b65e4a-f572-4cfc-b4e2-cf237f43c6f0_1, 
AzureStreamAnalytics_c4b65e4a-f572-4cfc-b4e2-cf237f43c6f0_1, 
AzureStreamAnalytics_c4b65e4a-f572-4cfc-b4e2-cf237f43c6f0_1.
```

> [!NOTE]
> Cuando el número de lectores cambia durante una actualización de trabajo, se escriben advertencias transitorias en registros de auditoría. Los trabajos de Stream Analytics se recuperan automáticamente de estos problemas transitorios.

### <a name="add-a-consumer-group-in-event-hubs"></a>Agregar un grupo de consumidores a Event Hubs

Para agregar un nuevo grupo de consumidores a la instancia de Event Hubs, siga estos pasos:

1. Inicie sesión en Azure Portal.

2. Busque su centro de eventos.

3. Seleccione **Event Hubs** en el encabezado **Entidades**.

4. Seleccione un centro de eventos según el nombre.

5. En la página **Instancia de Event Hubs**, en el encabezado **Entidades**, seleccione **Grupos de consumidores**. Aparecerá un grupo de consumidores con nombre **$Default**.

6. Seleccione **+ Grupo de consumidores** para agregar un nuevo grupo de consumidores. 

   ![Agregar un grupo de consumidores a Event Hubs](media/stream-analytics-event-hub-consumer-groups/new-eh-consumer-group.png)

7. Cuando se crea la entrada en el trabajo de Stream Analytics para que apunte al centro de eventos, es necesario especificar el grupo de consumidores ahí. **$Default** se utiliza cuando no se especifica ninguno. Una vez que cree un nuevo grupo de consumidores, edite la entrada de Event Hubs en el trabajo de Stream Analytics y especifique el nombre del nuevo grupo de consumidores.

## <a name="readers-per-partition-exceeds-event-hubs-limit"></a>El número de lectores por partición supera el límite de Event Hubs

Si la sintaxis de la consulta de streaming hace referencia varias veces al mismo recurso de entrada de Event Hubs, el motor de trabajo puede utilizar varios lectores por cada consulta de ese mismo grupo de consumidores. Cuando hay demasiadas referencias al mismo grupo de consumidores, el trabajo puede superar el límite establecido de cinco y se produce un error. En tales circunstancias, puede dividir aún más mediante el uso de múltiples entradas en varios grupos de consumidores; para ello, use la solución que se describe en la siguiente sección. 

Los escenarios en los que el número de lectores por partición supera el límite de Event Hubs de cinco incluyen los siguientes:

* Varias instrucciones SELECT: si usa varias instrucciones SELECT que hacen referencia a la **misma** entrada de centro de eventos, cada instrucción SELECT provoca que se cree un nuevo destinatario.

* UNION: cuando se utiliza UNION, es posible tener varias entradas que hacen referencia al **mismo** centro de eventos y grupo de consumidores.

* SELF JOIN: cuando se usa una operación SELF JOIN, es posible hacer referencia al **mismo** centro de eventos varias veces.

Las siguientes prácticas recomendadas pueden ayudar a reducir los escenarios en los que el número de lectores por partición supera el límite de Event Hubs de cinco.

### <a name="split-your-query-into-multiple-steps-by-using-a-with-clause"></a>División de la consulta en varios pasos mediante una cláusula WITH

La cláusula WITH especifica un conjunto de resultados con nombre temporal al que se puede hacer referencia mediante una cláusula FROM en la consulta. Defina la cláusula WITH en el ámbito de ejecución de una única instrucción SELECT.

Por ejemplo, en lugar de esta consulta:

```SQL
SELECT foo 
INTO output1
FROM inputEventHub

SELECT bar
INTO output2
FROM inputEventHub 
…
```

Utilice esta consulta:

```SQL
WITH data AS (
   SELECT * FROM inputEventHub
)

SELECT foo
INTO output1
FROM data

SELECT bar
INTO output2
FROM data
…
```

### <a name="ensure-that-inputs-bind-to-different-consumer-groups"></a>Asegúrese de que las entradas se enlazan a grupos de consumidores diferentes

Para las consultas en las que hay conectadas tres o más entradas al mismo grupo de consumidores de Event Hubs, cree grupos de consumidores independientes. Esto requiere la creación de entradas de Stream Analytics adicionales.

### <a name="create-separate-inputs-with-different-consumer-groups"></a>Creación de entradas independientes con diferentes grupos de consumidores

Puede crear entradas independientes con diferentes grupos de consumidores para el mismo Centro de eventos. La siguiente consulta UNION es un ejemplo en el que *InputOne* e *InputTwo* hacen referencia al mismo origen del Centro de eventos. Cualquier consulta puede tener entradas independientes con diferentes grupos de consumidores. La consulta UNION es solo un ejemplo.

```sql
WITH 
DataOne AS 
(
SELECT * FROM InputOne 
),

DataTwo AS 
(
SELECT * FROM InputTwo 
),

SELECT foo FROM DataOne
UNION 
SELECT foo FROM DataTwo

```

## <a name="readers-per-partition-exceeds-iot-hub-limit"></a>El número de lectores por partición supera el límite de IoT Hub

Los trabajos de Stream Analytics usan el [punto de conexión compatible con Event Hub](../iot-hub/iot-hub-devguide-messages-read-builtin.md) integrado de IoT Hub para conectarse y leer eventos de IoT Hub. Si la lectura por partición supera los límites de IoT Hub, puede usar las [soluciones para Event Hub](#readers-per-partition-exceeds-event-hubs-limit) para resolverlo. Puede crear un grupo de consumidores para el punto de conexión integrado bien a través de la sesión del punto de conexión del portal de IoT Hub bien a través del [SDK de IoT Hub](https://docs.microsoft.com/rest/api/iothub/IotHubResource/CreateEventHubConsumerGroup).

## <a name="get-help"></a>Obtener ayuda

Para más ayuda, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
