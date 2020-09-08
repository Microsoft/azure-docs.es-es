---
title: Patrones de consulta comunes en Azure Stream Analytics
description: En este artículo se describen varios patrones de consulta comunes y diseños que son útiles en los trabajos de Azure Stream Analytics.
services: stream-analytics
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/18/2019
ms.custom: devx-track-javascript
ms.openlocfilehash: 0eabe147563ee712e20e57aafc1029daf2a8610a
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88641048"
---
# <a name="common-query-patterns-in-azure-stream-analytics"></a>Patrones de consulta comunes en Azure Stream Analytics

Las consultas de Azure Stream Analytics se expresan en un lenguaje de consulta similar a SQL. Estas construcciones de lenguaje se documentan en la guía [Referencia de lenguaje de consulta de Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). 

El diseño de consultas puede expresar una lógica sencilla de paso a través para mover datos de evento desde un flujo de entrada a un almacén de datos de salida, o puede hacer la coincidencia de patrones enriquecidos y análisis temporal para calcular los agregados durante distintas ventanas de horarios como en la guía [Compilación de una solución de IoT con Stream Analytics](stream-analytics-build-an-iot-solution-using-stream-analytics.md). Puede combinar datos de varias entradas para combinar eventos de streaming y realizar búsquedas en datos de referencia estáticos que enriquecerán los valores de evento. También puede escribir datos en varias salidas.

En este artículo se describen las soluciones para varios patrones de consulta comunes basados en situaciones del mundo real.

## <a name="supported-data-formats"></a>Formatos de datos compatibles

Azure Stream Analytics admite eventos de procesamiento en formatos de datos CSV, JSON y Avro.

Tanto JSON como Avro pueden contener tipos complejos como objetos anidados (registros) o matrices. Para obtener más información sobre cómo trabajar con estos tipos de datos complejos, consulte el artículo [Análisis de datos JSON y AVRO](stream-analytics-parsing-json.md).

## <a name="send-data-to-multiple-outputs"></a>Envío de datos a varias salidas

Se pueden usar varias instrucciones **SELECT** para generar datos en distintos receptores de salida. Por ejemplo, una instrucción **SELECT** puede generar una alerta basada en umbral, mientras que otra puede generar eventos en el almacenamiento de blobs.

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Salida ArchiveOutput**:

| Asegúrese | Time |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Salida AlertOutput**:

| Asegúrese | Time | Count |
| --- | --- | --- |
| Make2 |2015-01-01T00:00:10.0000000Z |3 |

**Consultar**

```SQL
SELECT
    *
INTO
    ArchiveOutput
FROM
    Input TIMESTAMP BY Time

SELECT
    Make,
    System.TimeStamp() AS Time,
    COUNT(*) AS [Count]
INTO
    AlertOutput
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Make,
    TumblingWindow(second, 10)
HAVING
    [Count] >= 3
```

La cláusula **INTO** indica a Stream Analytics en cuál de las salidas se escribirán los datos. La primera instrucción **SELECT** define una consulta de paso a través que recibe datos de la entrada y los envía a la salida denominada **ArchiveOutput**. La segunda consulta hace una agregación y un filtrado simples, antes de enviar los resultados a la salida de un sistema de alertas descendente, denominada **AlertOutput**.

Tenga en cuenta que se puede usar la cláusula **WITH** para definir varios bloques de subconsultas. Esta opción ofrece la ventaja de tener que abrir menos lectores para el origen de entrada.

**Consultar**

```SQL
WITH ReaderQuery AS (
    SELECT
        *
    FROM
        Input TIMESTAMP BY Time
)

SELECT * INTO ArchiveOutput FROM ReaderQuery

SELECT 
    Make,
    System.TimeStamp() AS Time,
    COUNT(*) AS [Count] 
INTO AlertOutput 
FROM ReaderQuery
GROUP BY
    Make,
    TumblingWindow(second, 10)
HAVING [Count] >= 3
```

Para más información, consulte la [cláusula**WITH**](/stream-analytics-query/with-azure-stream-analytics).

## <a name="simple-pass-through-query"></a>Consulta de paso a través simple

Se puede usar una consulta de paso a través simple para copiar los datos del flujo de entrada en la salida. Por ejemplo, si un flujo de datos que contiene información sobre vehículos en tiempo real debe guardarse en una base de datos SQL para un análisis de letras, una consulta de paso a través simple podrá encargarse de la tareas.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |"1000" |
| Make1 |2015-01-01T00:00:02.0000000Z |"2000" |

**Salida**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |"1000" |
| Make1 |2015-01-01T00:00:02.0000000Z |"2000" |

**Consultar**

```SQL
SELECT
    *
INTO Output
FROM Input
```

Una consulta **SELECT** * proyecta todos los campos de un evento entrante y los envía a la salida. Del mismo modo, **SELECT** también puede usarse para proyectar solo los campos obligatorios de la entrada. En este ejemplo, si *Make* y *Time* del vehículo son los únicos campos obligatorios que se van a guardar, esos campos se pueden especificar en la instrucción **SELECT**.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |1000 |
| Make1 |2015-01-01T00:00:02.0000000Z |2000 |
| Make2 |2015-01-01T00:00:04.0000000Z |1\.500 |

**Salida**:

| Asegúrese | Time |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:04.0000000Z |

**Consultar**

```SQL
SELECT
    Make, Time
INTO Output
FROM Input
```

## <a name="string-matching-with-like-and-not-like"></a>Coincidencia de cadenas con LIKE y NOT LIKE

**LIKE** y **NOT LIKE** se pueden usar para verificar si un campo coincide con un patrón determinado. Por ejemplo, se puede crear un filtro para devolver solo las matrículas que comienzan con la letra "A" y terminan con el número 9.

**Entrada**:

| Asegúrese | License_plate | Time |
| --- | --- | --- |
| Make1 |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Make2 |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Make3 |ABC-369 |2015-01-01T00:00:03.0000000Z |

**Salida**:

| Asegúrese | License_plate | Time |
| --- | --- | --- |
| Make2 |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Make3 |ABC-369 |2015-01-01T00:00:03.0000000Z |

**Consultar**

```SQL
SELECT
    *
FROM
    Input TIMESTAMP BY Time
WHERE
    License_plate LIKE 'A%9'
```

Use la instrucción **LIKE** para comprobar el valor del campo **License_plate**. Debe comenzar con la letra "A", seguida de cualquier cadena de ceros o más caracteres y terminar con el número 9.

## <a name="calculation-over-past-events"></a>Cálculo con eventos pasados

La función **LAG** se puede usar para ver los eventos pasados dentro de una ventana de tiempo y compararlos con el evento actual. Por ejemplo, se puede obtener la marca del vehículo actual si difiere de la del último automóvil que pasó por el peaje.

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |

**Salida**:

| Asegúrese | Time |
| --- | --- |
| Make2 |2015-01-01T00:00:02.0000000Z |

**Consultar**

```SQL
SELECT
    Make,
    Time
FROM
    Input TIMESTAMP BY Time
WHERE
    LAG(Make, 1) OVER (LIMIT DURATION(minute, 1)) <> Make
```

Utilice **LAG** para observar el flujo de entrada del evento anterior, recuperar el valor de *Make* y compararlo con el valor de *Make* del evento actual y emitir el evento.

Para obtener más información, consulte [**LAG**](/stream-analytics-query/lag-azure-stream-analytics).

## <a name="return-the-last-event-in-a-window"></a>Devolución del último evento en una ventana

Dado que el sistema consume los eventos en tiempo real, no hay ninguna función que pueda determinar si un evento será el último en llegar para ese período de tiempo. Para ello, el flujo de entrada debe unirse con otro, donde el tiempo de un evento sea el tiempo máximo para todos los eventos de esa ventana.

**Entrada**:

| License_plate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Make3 |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Make1 |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Make2 |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Make2 |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Salida**:

| License_plate | Asegúrese | Time |
| --- | --- | --- |
| VFE 1616 |Make2 |2015-07-27T00:09:31.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Consultar**

```SQL
WITH LastInWindow AS
(
    SELECT 
        MAX(Time) AS LastEventTime
    FROM 
        Input TIMESTAMP BY Time
    GROUP BY 
        TumblingWindow(minute, 10)
)

SELECT 
    Input.License_plate,
    Input.Make,
    Input.Time
FROM
    Input TIMESTAMP BY Time 
    INNER JOIN LastInWindow
    ON DATEDIFF(minute, Input, LastInWindow) BETWEEN 0 AND 10
    AND Input.Time = LastInWindow.LastEventTime
```

El primer paso de la consulta encuentra la marca de tiempo máxima en ventanas de 10 minutos, que es la marca de tiempo del último evento para esa ventana. El segundo paso combina los resultados de la primera consulta con el flujo original para buscar el evento que coincide con las últimas marcas de tiempo en cada ventana. 

**DATEDIFF** es una función específica de fecha que compara y devuelve la diferencia temporal entre dos campos DateTime. Para obtener más información, consulte las [funciones de fecha](https://docs.microsoft.com/stream-analytics-query/date-and-time-functions-azure-stream-analytics).

Para obtener más información sobre cómo combinar flujos, consulte [**JOIN**](/stream-analytics-query/join-azure-stream-analytics).

## <a name="data-aggregation-over-time"></a>Agregación de datos a lo largo del tiempo

Para procesar información en un período de tiempo, los datos se pueden agregar juntos. En este ejemplo, un recuento se calcula en los últimos 10 segundos de tiempo por cada marca de coche específica.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |1000 |
| Make1 |2015-01-01T00:00:02.0000000Z |2000 |
| Make2 |2015-01-01T00:00:04.0000000Z |1\.500 |

**Salida**:

| Asegúrese | Count |
| --- | --- |
| Make1 | 2 |
| Make2 | 1 |

**Consultar**

```SQL
SELECT
    Make,
    COUNT(*) AS Count
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Make,
    TumblingWindow(second, 10)
```

Esta agregación agrupa los automóviles por *Make* y los cuenta cada 10 segundos. La salida tiene los valores de *Make* y *Count* de automóviles que han pasado por el peaje.

TumblingWindow es una función de ventana que se utiliza para agrupar eventos. Se puede aplicar una agregación a todos los eventos agrupados. Para más información, consulte las [funciones de ventana](stream-analytics-window-functions.md).

Para más información sobre la agregación, consulte [funciones de agregado](/stream-analytics-query/aggregate-functions-azure-stream-analytics).

## <a name="periodically-output-values"></a>Emisión periódica de valores

En caso de eventos irregulares o ausentes, se puede generar una salida de intervalo periódico a partir de una entrada de datos más dispersa. Por ejemplo, genere un evento cada cinco segundos que notifique el punto de datos visto más recientemente.

**Entrada**:

| Time | Value |
| --- | --- |
| "2014-01-01T06:01:00" |1 |
| "2014-01-01T06:01:05" |2 |
| "2014-01-01T06:01:10" |3 |
| "2014-01-01T06:01:15" |4 |
| "2014-01-01T06:01:30" |5 |
| "2014-01-01T06:01:35" |6 |

**Salida (10 primeras filas)** :

| Window_end | Last_event.Time | Last_event.Value |
| --- | --- | --- |
| 2014-01-01T14:01:00.000Z |2014-01-01T14:01:00.000Z |1 |
| 2014-01-01T14:01:05.000Z |2014-01-01T14:01:05.000Z |2 |
| 2014-01-01T14:01:10.000Z |2014-01-01T14:01:10.000Z |3 |
| 2014-01-01T14:01:15.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:20.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:25.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:30.000Z |2014-01-01T14:01:30.000Z |5 |
| 2014-01-01T14:01:35.000Z |2014-01-01T14:01:35.000Z |6 |
| 2014-01-01T14:01:40.000Z |2014-01-01T14:01:35.000Z |6 |
| 2014-01-01T14:01:45.000Z |2014-01-01T14:01:35.000Z |6 |

**Consultar**

```SQL
SELECT
    System.Timestamp() AS Window_end,
    TopOne() OVER (ORDER BY Time DESC) AS Last_event
FROM
    Input TIMESTAMP BY Time
GROUP BY
    HOPPINGWINDOW(second, 300, 5)
```

Esta consulta genera eventos cada cinco segundos y genera como resultado el último evento que se recibió anteriormente. La duración **HOPPINGWINDOW** determina cuánto tiempo atrás buscará la consulta para encontrar el evento más reciente.

Para más información, consulte [Ventana de salto](/stream-analytics-query/hopping-window-azure-stream-analytics).

## <a name="correlate-events-in-a-stream"></a>Correlación de eventos en un flujo

La correlación de eventos en el mismo flujo se puede realizar observando los eventos pasados con la función **LAG**. Por ejemplo, se puede generar una salida cada vez que dos automóviles consecutivos con el mismo valor de *Make* pasan por el peaje durante los últimos 90 segundos.

**Entrada**:

| Asegúrese | License_plate | Time |
| --- | --- | --- |
| Make1 |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Make1 |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Make2 |DEF-987 |2015-01-01T00:00:03.0000000Z |
| Make1 |GHI-345 |2015-01-01T00:00:04.0000000Z |

**Salida**:

| Asegúrese | Time | Current_car_license_plate | First_car_license_plate | First_car_time |
| --- | --- | --- | --- | --- |
| Make1 |2015-01-01T00:00:02.0000000Z |AAA-999 |ABC-123 |2015-01-01T00:00:01.0000000Z |

**Consultar**

```SQL
SELECT
    Make,
    Time,
    License_plate AS Current_car_license_plate,
    LAG(License_plate, 1) OVER (LIMIT DURATION(second, 90)) AS First_car_license_plate,
    LAG(Time, 1) OVER (LIMIT DURATION(second, 90)) AS First_car_time
FROM
    Input TIMESTAMP BY Time
WHERE
    LAG(Make, 1) OVER (LIMIT DURATION(second, 90)) = Make
```

La función **LAG** puede observar el flujo de entrada del evento anterior y recuperar el valor de *Make*, y compararlo con el valor de *Make* del evento actual.  Una vez que se cumple la condición, se pueden proyectar los datos del evento anterior mediante **LAG** en la instrucción **SELECT**.

Para obtener más información, consulte [LAG](/stream-analytics-query/lag-azure-stream-analytics).

## <a name="detect-the-duration-between-events"></a>Detección de la duración entre eventos

La duración de un evento se puede calcular examinando el último evento Start una vez que se recibe un evento End. Esta consulta puede resultar útil para determinar el tiempo que un usuario emplea en una página o una característica.

**Entrada**:  

| Usuario | Característica | Evento | Time |
| --- | --- | --- | --- |
| user@location.com |RightMenu |Start |2015-01-01T00:00:01.0000000Z |
| user@location.com |RightMenu |End |2015-01-01T00:00:08.0000000Z |

**Salida**:  

| Usuario | Característica | Duration |
| --- | --- | --- |
| user@location.com |RightMenu |7 |

**Consultar**

```SQL
SELECT
    [user],
    feature,
    DATEDIFF(
        second,
        LAST(Time) OVER (PARTITION BY [user], feature LIMIT DURATION(hour, 1) WHEN Event = 'start'),
        Time) as duration
FROM input TIMESTAMP BY Time
WHERE
    Event = 'end'
```

La función **LAST** se puede usar para recuperar el último evento dentro de una condición específica. En este ejemplo, la condición es un evento de tipo Start, donde la búsqueda se particiona mediante el usuario y la característica **PARTITION BY**. De este modo, cada usuario y característica se tratan de forma independiente al buscar el evento Start. **LIMIT DURATION** limita la búsqueda en el tiempo a 1 hora entre los eventos End y Start.

## <a name="count-unique-values"></a>Recuento de valores únicos

**COUNT** y **DISTINCT** pueden usarse para contar el número de valores de campo únicos que aparecen en la transmisión durante un período de tiempo determinado. Puede crearse una consulta para calcular cuántas *marcas* de vehículos únicas pasan por la cabina de peaje en una ventana de 2 segundos.

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Salida:**

| Count_make | Time |
| --- | --- |
| 2 |2015-01-01T00:00:02.000Z |
| 1 |2015-01-01T00:00:04.000Z |

**Consulta:**

```SQL
SELECT
     COUNT(DISTINCT Make) AS Count_make,
     System.TIMESTAMP() AS Time
FROM Input TIMESTAMP BY TIME
GROUP BY 
     TumblingWindow(second, 2)
```

**COUNT(DISTINCT Make)** devuelve la cantidad de valores distintos de la columna **Make** dentro de una ventana de tiempo.
Para más información, consulte la [función de agregado**COUNT**](/stream-analytics-query/count-azure-stream-analytics).

## <a name="retrieve-the-first-event-in-a-window"></a>Recuperación del primer evento en una ventana

Se puede usar **IsFirst** para recuperar el primer evento en una ventana de tiempo. Por ejemplo, para emitir la información del primer automóvil en cada intervalo de 10 minutos.

**Entrada**:

| License_plate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Make3 |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Make1 |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Make2 |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Make2 |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Salida**:

| License_plate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |

**Consultar**

```SQL
SELECT 
    License_plate,
    Make,
    Time
FROM 
    Input TIMESTAMP BY Time
WHERE 
    IsFirst(minute, 10) = 1
```

Además, **IsFirst** puede particionar los datos y calcular el primer evento para cada *Make* de automóvil específico que encuentre en cada intervalo de 10 minutos.

**Salida**:

| License_plate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Make3 |2015-07-27T00:02:17.0000000Z |
| YHN 6970 |Make2 |2015-07-27T00:06:00.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Consultar**

```SQL
SELECT 
    License_plate,
    Make,
    Time
FROM 
    Input TIMESTAMP BY Time
WHERE 
    IsFirst(minute, 10) OVER (PARTITION BY Make) = 1
```

Para obtener más información, consulte [**IsFirst**](/stream-analytics-query/isfirst-azure-stream-analytics).

## <a name="remove-duplicate-events-in-a-window"></a>Quitar eventos duplicados de una ventana

Al realizar una operación, como calcular promedios de eventos en un período de tiempo determinado, se deben filtrar los eventos duplicados. En el ejemplo siguiente, el segundo evento es un duplicado del primero.

**Entrada**:  

| deviceId | Time | Atributo | Value |
| --- | --- | --- | --- |
| 1 |2018-07-27T00:00:01.0000000Z |Temperatura |50 |
| 1 |2018-07-27T00:00:01.0000000Z |Temperatura |50 |
| 2 |2018-07-27T00:00:01.0000000Z |Temperatura |40 |
| 1 |2018-07-27T00:00:05.0000000Z |Temperatura |60 |
| 2 |2018-07-27T00:00:05.0000000Z |Temperatura |50 |
| 1 |2018-07-27T00:00:10.0000000Z |Temperatura |100 |

**Salida**:  

| AverageValue | deviceId |
| --- | --- |
| 70 | 1 |
|45 | 2 |

**Consultar**

```SQL
With Temp AS (
SELECT
    COUNT(DISTINCT Time) AS CountTime,
    Value,
    DeviceId
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Value,
    DeviceId,
    SYSTEM.TIMESTAMP()
)

SELECT
    AVG(Value) AS AverageValue, DeviceId
INTO Output
FROM Temp
GROUP BY DeviceId,TumblingWindow(minute, 5)
```

**COUNT(DISTINCT Time)** devuelve el número de valores distintos de la columna Time dentro de una ventana de tiempo. La salida del primer paso puede usarse para calcular el promedio por dispositivo, descartando los duplicados.

Para más información, consulte [COUNT(DISTINCT Time)](/stream-analytics-query/count-azure-stream-analytics).

## <a name="specify-logic-for-different-casesvalues-case-statements"></a>Especificación de la lógica para los distintos casos/valores (instrucciones CASE)

Las instrucciones **CASE** pueden proporcionar diferentes cálculos para distintos campos, en función de un criterio determinado. Por ejemplo, asigne el carril "A" a los automóviles de *Make1* y el carril "B" a cualquier otra marca.

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Salida**:

| Asegúrese |Dispatch_to_lane | Time |
| --- | --- | --- |
| Make1 |"A" |2015-01-01T00:00:01.0000000Z |
| Make2 |"B" |2015-01-01T00:00:02.0000000Z |

**Solución**:

```SQL
SELECT
    Make
    CASE
        WHEN Make = "Make1" THEN "A"
        ELSE "B"
    END AS Dispatch_to_lane,
    System.TimeStamp() AS Time
FROM
    Input TIMESTAMP BY Time
```

La expresión **CASE** compara una expresión con un conjunto de expresiones simples para determinar el resultado. En este ejemplo, los vehículos de *Make1* se envían al carril "A", mientras que los vehículos de cualquier otra marca se asignarán al carril "B".

Para más información, consulte la [expresión CASE](/stream-analytics-query/case-azure-stream-analytics).

## <a name="data-conversion"></a>Conversión de datos

Los datos se pueden convertir en tiempo real mediante el método **CAST**. Por ejemplo, el peso del automóvil se puede convertir del tipo **nvarchar(max)** al tipo **bigint** y usarse en un cálculo numérico.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |"1000" |
| Make1 |2015-01-01T00:00:02.0000000Z |"2000" |

**Salida**:

| Asegúrese | Peso |
| --- | --- |
| Make1 |3000 |

**Consultar**

```SQL
SELECT
    Make,
    SUM(CAST(Weight AS BIGINT)) AS Weight
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Make,
    TumblingWindow(second, 10)
```

Use una instrucción **CAST** para especificar su tipo de datos. Vea la lista de tipos de datos admitidos en [Tipos de datos (Azure Stream Analytics)](/stream-analytics-query/data-types-azure-stream-analytics).

Para más información, consulte las [funciones de conversión de tipos](/stream-analytics-query/conversion-functions-azure-stream-analytics).

## <a name="detect-the-duration-of-a-condition"></a>Detección de la duración de una condición

En el caso de las condiciones que abarcan varios eventos, se puede usar la función **LAG** para identificar la duración de esa condición. Por ejemplo, supongamos que por error todos los vehículos tienen un peso incorrecto (por encima de 20 000 libras), y debe calcularse la duración del error.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |2000 |
| Make2 |2015-01-01T00:00:02.0000000Z |25000 |
| Make1 |2015-01-01T00:00:03.0000000Z |26000 |
| Make2 |2015-01-01T00:00:04.0000000Z |25000 |
| Make1 |2015-01-01T00:00:05.0000000Z |26000 |
| Make2 |2015-01-01T00:00:06.0000000Z |25000 |
| Make1 |2015-01-01T00:00:07.0000000Z |26000 |
| Make2 |2015-01-01T00:00:08.0000000Z |2000 |

**Salida**:

| Start_fault | End_fault |
| --- | --- |
| 2015-01-01T00:00:02.000Z |2015-01-01T00:00:07.000Z |

**Consultar**

```SQL
WITH SelectPreviousEvent AS
(
SELECT
    *,
    LAG([time]) OVER (LIMIT DURATION(hour, 24)) as previous_time,
    LAG([weight]) OVER (LIMIT DURATION(hour, 24)) as previous_weight
FROM input TIMESTAMP BY [time]
)

SELECT 
    LAG(time) OVER (LIMIT DURATION(hour, 24) WHEN previous_weight < 20000 ) [Start_fault],
    previous_time [End_fault]
FROM SelectPreviousEvent
WHERE
    [weight] < 20000
    AND previous_weight > 20000
```
La primera instrucción **SELECT** correlaciona la medida de peso actual con la medida anterior, y la proyecta junto con la medida actual. La segunda **SELECT** vuelve a observar el último evento donde *previous_weight* es inferior a 20000, donde el peso actual es menor que 20000, y *previous_weight* del evento actual era mayor que 20000.

End_fault es el evento actual sin errores donde el evento anterior presentaba errores, y Start_fault es el último evento sin errores anterior a ese.

## <a name="process-events-with-independent-time-substreams"></a>Procesamiento de eventos con tiempo independiente (subflujos)

Los eventos pueden llegar tarde o desordenados debido a sesgos de reloj entre los productores de eventos, a sesgos de reloj entre particiones o a la latencia de red.
Por ejemplo, el reloj del dispositivo para *TollID* 2 va cinco segundos atrasado respecto a *TollID* 1, y el reloj del dispositivo para *TollID* 3 va diez segundos atrasado respecto a *TollID* 1. Se puede generar un cálculo de forma independiente para cada peaje, teniendo en cuenta solo sus propios datos de reloj como marca de tiempo.

**Entrada**:

| LicensePlate | Asegúrese | Time | TollId |
| --- | --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:01.0000000Z | 1 |
| YHN 6970 |Make2 |2015-07-27T00:00:05.0000000Z | 1 |
| QYF 9358 |Make1 |2015-07-27T00:00:01.0000000Z | 2 |
| GXF 9462 |Make3 |2015-07-27T00:00:04.0000000Z | 2 |
| VFE 1616 |Make2 |2015-07-27T00:00:10.0000000Z | 1 |
| RMV 8282 |Make1 |2015-07-27T00:00:03.0000000Z | 3 |
| MDR 6128 |Make3 |2015-07-27T00:00:11.0000000Z | 2 |
| YZK 5704 |Make4 |2015-07-27T00:00:07.0000000Z | 3 |

**Salida**:

| TollId | Count |
| --- | --- |
| 1 | 2 |
| 2 | 2 |
| 1 | 1 |
| 3 | 1 |
| 2 | 1 |
| 3 | 1 |

**Consultar**

```SQL
SELECT
      TollId,
      COUNT(*) AS Count
FROM input
      TIMESTAMP BY Time OVER TollId
GROUP BY TUMBLINGWINDOW(second, 5), TollId
```

La cláusula **TIMESTAMP BY OVER** examina la escala de tiempo de cada dispositivo de manera independiente mediante subflujos. Los eventos de salida para cada *TollID* se generan a medida que se calculan, lo que significa que los eventos están en orden con respecto a cada *TollID*, en lugar de que se vuelvan a ordenar como si todos los dispositivos estuvieran en el mismo reloj.

Para más información, consulte este [TIMESTAMP BY OVER](/stream-analytics-query/timestamp-by-azure-stream-analytics#over-clause-interacts-with-event-ordering).

## <a name="session-windows"></a>Ventanas de sesión

Una ventana de sesión es una ventana que se expande a medida que se producen eventos y se cierra para el cálculo si no se recibe ningún evento después de un período de tiempo específico, o si la ventana alcanza su duración máxima.
Esta ventana resulta especialmente útil al calcular datos de interacción de un usuario. Una ventana se inicia cuando un usuario empieza a interactuar con el sistema y se cierra cuando no se observan más eventos; es decir, que el usuario ha dejado de interactuar.
Por ejemplo, un usuario interactúa con una página web donde se registra el número de clics; se puede usar una ventana de sesión para averiguar cuánto tiempo ha interactuado el usuario con el sitio.

**Entrada**:

| User_id | Time | URL |
| --- | --- | --- |
| 0 | 2017-01-26T00:00:00.0000000Z | "www.ejemplo.com/a.html" |
| 0 | 2017-01-26T00:00:20.0000000Z | "www.ejemplo.com/b.html" |
| 1 | 2017-01-26T00:00:55.0000000Z | "www.ejemplo.com/c.html" |
| 0 | 2017-01-26T00:01:10.0000000Z | "www.ejemplo.com/d.html" |
| 1 | 2017-01-26T00:01:15.0000000Z | "www.ejemplo.com/e.html" |

**Salida**:

| User_id | StartTime | EndTime | Duration_in_seconds |
| --- | --- | --- | --- |
| 0 | 2017-01-26T00:00:00.0000000Z | 2017-01-26T00:01:10.0000000Z | 70 |
| 1 | 2017-01-26T00:00:55.0000000Z | 2017-01-26T00:01:15.0000000Z | 20 |

**Consultar**

``` SQL
SELECT
    user_id,
    MIN(time) as StartTime,
    MAX(time) as EndTime,
    DATEDIFF(second, MIN(time), MAX(time)) AS duration_in_seconds
FROM input TIMESTAMP BY time
GROUP BY
    user_id,
    SessionWindow(minute, 1, 60) OVER (PARTITION BY user_id)
```

**SELECT** proyecta los datos pertinentes a la interacción del usuario, junto con la duración de la interacción. Agrupa los datos por usuario y una **SessionWindow** que se cierra si no se produce ninguna interacción en 1 minuto, con un tamaño máximo de la ventana de 60 minutos.

Para obtener más información sobre **SessionWindow**, consulte [Ventana de sesión](/stream-analytics-query/session-window-azure-stream-analytics).

## <a name="language-extensibility-with-user-defined-function-in-javascript-and-c"></a>Extensibilidad del lenguaje con la función definida por el usuario en JavaScript y C#

El lenguaje de consultas de Azure Stream Analytics se puede ampliar con funciones personalizadas escritas en los lenguajes JavaScript o C#. Las funciones definidas por el usuario (UDF) son cálculos personalizados o complejos que no se pueden expresar fácilmente mediante el lenguaje **SQL**. Estas UDF se pueden definir una vez y usarse varias veces dentro de una consulta. Por ejemplo, se puede usar una UDF para convertir un valor hexadecimal *nvarchar(max)* en un valor *bigint*.

**Entrada**:

| Device_id | HexValue |
| --- | --- |
| 1 | "B4" |
| 2 | "11B" |
| 3 | "121" |

**Salida**:

| Device_id | Decimal |
| --- | --- |
| 1 | 180 |
| 2 | 283 |
| 3 | 289 |

```JavaScript
function hex2Int(hexValue){
    return parseInt(hexValue, 16);
}
```

```C#
public static class MyUdfClass {
    public static long Hex2Int(string hexValue){
        return int.Parse(hexValue, System.Globalization.NumberStyles.HexNumber);
    }
}
```

```SQL
SELECT
    Device_id,
    udf.Hex2Int(HexValue) AS Decimal
From
    Input
```

La función definida por el usuario calculará el valor *bigint* de HexValue en cada evento consumido.

Para más información, consulte [JavaScript](/azure/stream-analytics/stream-analytics-javascript-user-defined-functions) y [C#](/azure/stream-analytics/stream-analytics-edge-csharp-udf).

## <a name="advanced-pattern-matching-with-match_recognize"></a>Coincidencia de patrones avanzada con MATCH_RECOGNIZE

**MATCH_RECOGNIZE** es un mecanismo de coincidencia de patrones avanzada que se puede usar para hacer coincidir una secuencia de eventos con un patrón de expresión regular bien definido.
Por ejemplo, se supervisa un cajero automático en tiempo real en busca de errores, durante el funcionamiento del cajero, si hay dos mensajes de advertencia consecutivos, debe darse aviso al administrador.

**Entrada**:

| ATM_id | Operation_id | Return_Code | Time |
| --- | --- | --- | --- |
| 1 | "Introducción de PIN" | "Success" | 2017-01-26T00:10:00.0000000Z |
| 2 | "Apertura del dispensador de dinero" | "Success" | 2017-01-26T00:10:07.0000000Z |
| 2 | "Cierre del dispensador de dinero" | "Success" | 2017-01-26T00:10:11.0000000Z |
| 1 | "Introducción de la cantidad del retiro" | "Success" | 2017-01-26T00:10:08.0000000Z |
| 1 | "Apertura del dispensador de dinero" | "Warning" | 2017-01-26T00:10:14.0000000Z |
| 1 | "Impresión del saldo bancario" | "Warning" | 2017-01-26T00:10:19.0000000Z |

**Salida**:

| ATM_id | First_Warning_Operation_id | Warning_Time |
| --- | --- | --- |
| 1 | "Apertura del dispensador de dinero" | 2017-01-26T00:10:14.0000000Z |

```SQL
SELECT *
FROM input TIMESTAMP BY time OVER ATM_id
MATCH_RECOGNIZE (
    LIMIT DURATION(minute, 1)
    PARTITON BY ATM_id
    MEASURES
        First(Warning.ATM_id) AS ATM_id,
        First(Warning.Operation_Id) AS First_Warning_Operation_id,
        First(Warning.Time) AS Warning_Time
    AFTER MATCH SKIP TO NEXT ROW
    PATTERN (Success* Warning{2,})
    DEFINE
        Success AS Succes.Return_Code = 'Success',
        Failure AS Warning.Return_Code <> 'Success'
) AS patternMatch
```

Esta consulta coincide al menos con dos eventos de error consecutivos y genera una alarma cuando se cumplen las condiciones.
**PATTERN** define la expresión regular que se utilizará en la búsqueda de coincidencias, en este caso, cualquier número de operaciones correctas seguidas de al menos dos errores consecutivos.
Success y Failure se definen mediante el valor Return_Code y, una vez que se cumple la condición, **MEASURES** se proyectan con *ATM_id*, la primera operación de advertencia y la primera hora de advertencia.

Para obtener más información, consulte [MATCH_RECOGNIZE](/stream-analytics-query/match-recognize-stream-analytics).

## <a name="geofencing-and-geospatial-queries"></a>Consultas de geovalla y geoespaciales

Azure Stream Analytics proporciona funciones geoespaciales integradas que se pueden usar para implementar escenarios como, por ejemplo, la administración de flotas, transporte compartido, automóviles conectados y seguimiento de recursos.
Los datos geoespaciales se pueden ingerir en formato GeoJSON o WKT como parte del flujo de eventos o datos de referencia.
Por ejemplo, una empresa especializada en la fabricación de máquinas para imprimir pasaportes, alquila sus equipos a Gobiernos y consulados. La ubicación de esas máquinas está muy controlada a fin de evitar su extravío y el uso posible para la falsificación de pasaportes. Cada máquina está equipada con un rastreador GPS, cuya información se retransmite a un trabajo de Azure Stream Analytics.
El fabricante quiere realizar un seguimiento de la ubicación de esas máquinas y recibir una alerta si una de ellas abandona un área autorizada, de manera que puedan deshabilitarla de forma remota, alertar a las autoridades y recuperar el equipo.

**Entrada**:

| Equipment_id | Equipment_current_location | Time |
| --- | --- | --- |
| 1 | "POINT(-122.13288797982818 47.64082002051315)" | 2017-01-26T00:10:00.0000000Z |
| 1 | "POINT(-122.13307252987875 47.64081350934929)" | 2017-01-26T00:11:00.0000000Z |
| 1 | "POINT(-122.13308862313283 47.6406508603241)" | 2017-01-26T00:12:00.0000000Z |
| 1 | "POINT(-122.13341048821462 47.64043760861279)" | 2017-01-26T00:13:00.0000000Z |

**Entrada de datos de referencia**:

| Equipment_id | Equipment_lease_location |
| --- | --- |
| 1 | "POLYGON((-122.13326028450979 47.6409833866794,-122.13261655434621 47.6409833866794,-122.13261655434621 47.64061471602751,-122.13326028450979 47.64061471602751,-122.13326028450979 47.6409833866794))" |

**Salida**:

| Equipment_id | Equipment_alert_location | Time |
| --- | --- | --- |
| 1 | "POINT(-122.13341048821462 47.64043760861279)" | 2017-01-26T00:13:00.0000000Z |

```SQL
SELECT
    input.Equipment_id AS Equipment_id,
    input.Equipment_current_location AS Equipment_current_location,
    input.Time AS Time
FROM input TIMESTAMP BY time
JOIN
    referenceInput 
    ON input.Equipment_id = referenceInput.Equipment_id
    WHERE 
        ST_WITHIN(input.Equipment_currenct_location, referenceInput.Equipment_lease_location) = 1
```

La consulta permite al fabricante supervisar la ubicación de los equipos automáticamente y recibir alertas cuando una máquina abandona la geovalla permitida. La función geoespacial integrada permite a los usuarios usar datos GPS dentro de la consulta sin bibliotecas de terceros.

Para más información, consulte el artículo [Escenarios de agregación geoespacial y de geovalla con Azure Stream Analytics](geospatial-scenarios.md).

## <a name="get-help"></a>Obtener ayuda

Para más ayuda, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
