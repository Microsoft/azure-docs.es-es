---
title: Patrones de consulta comunes en Azure Stream Analytics
description: En este artículo se describe una serie de patrones de consulta comunes y diseños que son útiles en los trabajos de Azure Stream Analytics.
services: stream-analytics
author: jseb225
ms.author: jeanb
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/16/2019
ms.openlocfilehash: f6971038be7404850d958de67eb4755ae7d21a29
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761971"
---
# <a name="query-examples-for-common-stream-analytics-usage-patterns"></a>Ejemplos de consulta para patrones de uso comunes de Stream Analytics

Las consultas de Azure Stream Analytics se expresan en un lenguaje de consulta similar a SQL. Estas construcciones de lenguaje se documentan en la guía [Referencia de lenguaje de consulta de Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). 

El diseño de consulta puede expresar una lógica sencilla de paso a través para mover datos de evento desde un flujo de entrada a un almacén de datos de salida o hacer análisis temporal y coincidencia de patrón enriquecido para calcular agregados durante varias ventanas de tiempo como en el [compilar IoT solución mediante el uso de Stream Analytics](stream-analytics-build-an-iot-solution-using-stream-analytics.md) guía. Puede combinar datos de varias entradas para combinar eventos de streaming y se pueden realizar búsquedas en datos de referencia estáticos para enriquecer los valores del evento. También puede escribir datos en varias salidas.

En este artículo se describe las soluciones para varios patrones comunes de consultas basadas en escenarios del mundo real.

## <a name="work-with-complex-data-types-in-json-and-avro"></a>Trabajo con tipos de datos complejos en JSON y AVRO

Azure Stream Analytics admite eventos de procesamiento en formatos de datos CSV, JSON y Avro.

Tanto JSON como Avro pueden contener tipos complejos como objetos anidados (registros) o matrices. Para obtener más información sobre cómo trabajar con estos tipos de datos complejos, consulte el [datos de análisis de JSON y AVRO](stream-analytics-parsing-json.md) artículo.

## <a name="query-example-convert-data-types"></a>Ejemplo de consulta: Convertir tipos de datos

**Descripción**: Defina los tipos de propiedades en el flujo de entrada. Por ejemplo, el peso del vehículo se incorpora a la secuencia de entrada como cadenas y debe convertirse a **INT** realizar **suma**.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |"1000" |
| Honda |2015-01-01T00:00:02.0000000Z |"2000" |

**Salida**:

| Asegúrese | Peso |
| --- | --- |
| Honda |3000 |

**Solución**:

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

**Explicación**: Use una instrucción **CAST** en el campo **Peso** para especificar su tipo de datos. Vea la lista de tipos de datos admitidos en [Data types (Azure Stream Analytics)](/stream-analytics-query/data-types-azure-stream-analytics) [Tipos de datos (Azure Stream Analytics)].

## <a name="query-example-use-likenot-like-to-do-pattern-matching"></a>Ejemplo de consulta: Utilizar LIKE/NOT LIKE para realizar la coincidencia de patrones

**Descripción**: Compruebe que el valor de un campo del evento coincide con un patrón determinado.
Por ejemplo, compruebe que el resultado devuelve las matrículas que empiezan por A y terminan en nueve.

**Entrada**:

| Asegúrese | LicensePlate | Time |
| --- | --- | --- |
| Honda |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Toyota |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Nissan |ABC-369 |2015-01-01T00:00:03.0000000Z |

**Salida**:

| Asegúrese | LicensePlate | Time |
| --- | --- | --- |
| Toyota |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Nissan |ABC-369 |2015-01-01T00:00:03.0000000Z |

**Solución**:

```SQL
    SELECT
        *
    FROM
        Input TIMESTAMP BY Time
    WHERE
        LicensePlate LIKE 'A%9'
```

**Explicación**: Use la instrucción **LIKE** para comprobar el valor del campo **LicensePlate**. Debe empezar por la letra A, a continuación, seguida de cualquier cadena de cero o más caracteres y, a continuación, terminar con el número 9. 

## <a name="query-example-specify-logic-for-different-casesvalues-case-statements"></a>Ejemplo de consulta: Especificación de la lógica para los distintos casos/valores (instrucciones CASE)

**Descripción**: Proporcione un cálculo diferente para un campo en función de un criterio concreto. Por ejemplo, proporcione una descripción de cadena para el número de vehículos que han pasado de la misma marca, con un caso especial para 1.

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**Salida**:

| CarsPassed | Time |
| --- | --- |
| 1 Honda |2015-01-01T00:00:10.0000000Z |
| 2 Toyotas |2015-01-01T00:00:10.0000000Z |

**Solución**:

```SQL
    SELECT
        CASE
            WHEN COUNT(*) = 1 THEN CONCAT('1 ', Make)
            ELSE CONCAT(CAST(COUNT(*) AS NVARCHAR(MAX)), ' ', Make, 's')
        END AS CarsPassed,
        System.TimeStamp() AS Time
    FROM
        Input TIMESTAMP BY Time
    GROUP BY
        Make,
        TumblingWindow(second, 10)
```

**Explicación**: La expresión **CASE** compara una expresión con un conjunto de expresiones simples para determinar el resultado. En este ejemplo, las marcas de vehículos con un recuento de 1 han devuelto una descripción de cadena diferente de la de las marcas de vehículos con un recuento distinto a 1.

## <a name="query-example-send-data-to-multiple-outputs"></a>Ejemplo de consulta: Envío de datos a varias salidas

**Descripción**: Envíe datos a varios destinos de salida desde un único trabajo. Por ejemplo, analice los datos para una alerta de umbral y archive todos los eventos en Blob Storage.

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Honda |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**Salida1**:

| Asegúrese | Time |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Honda |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**Salida2**:

| Asegúrese | Time | Count |
| --- | --- | --- |
| Toyota |2015-01-01T00:00:10.0000000Z |3 |

**Solución**:

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

**Explicación**: La cláusula **INTO** indica a Stream Analytics en cuál de las salidas se escribirán los datos de esta instrucción. La primera consulta es un paso a través de los datos recibidos a una salida llamada **ArchiveOutput**. La segunda consulta hace simple agregación y filtrado y envía los resultados a un sistema de alertas descendente, **AlertOutput**.

Tenga en cuenta que también puede reutilizar los resultados de las expresiones de tabla comunes (CTE), como las instrucciones **WITH**, en varias instrucciones de salida. Esta opción ofrece el beneficio adicional de la apertura de algunos lectores para el origen de entrada.

Por ejemplo: 

```SQL
    WITH AllRedCars AS (
        SELECT
            *
        FROM
            Input TIMESTAMP BY Time
        WHERE
            Color = 'red'
    )
    SELECT * INTO HondaOutput FROM AllRedCars WHERE Make = 'Honda'
    SELECT * INTO ToyotaOutput FROM AllRedCars WHERE Make = 'Toyota'
```

## <a name="query-example-count-unique-values"></a>Ejemplo de consulta: Recuento de valores únicos

**Descripción**: Cuente el número de valores de campo únicos que aparecen en el flujo durante un período de tiempo determinado. Por ejemplo, ¿cuántas marcas de vehículos únicas pasan a través de la cabina de peaje en un intervalo de dos segundos?

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Honda |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**Salida:**

| CountMake | Time |
| --- | --- |
| 2 |2015-01-01T00:00:02.000Z |
| 1 |2015-01-01T00:00:04.000Z |

**Solución:**

```SQL
SELECT
     COUNT(DISTINCT Make) AS CountMake,
     System.TIMESTAMP() AS TIME
FROM Input TIMESTAMP BY TIME
GROUP BY 
     TumblingWindow(second, 2)
```


**Explicación:**
**COUNT(DISTINCT Make)** devuelve la cantidad de valores distintos de la columna **Make** dentro de una ventana de tiempo.

## <a name="query-example-determine-if-a-value-has-changed"></a>Ejemplo de consulta: Determinar si un valor ha cambiado

**Descripción**: Busque un valor anterior para determinar si es diferente del valor actual. Por ejemplo, ¿el vehículo anterior de la autopista de peaje es de la misma marca que el actual?

**Entrada**:

| Asegúrese | Time |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |

**Salida**:

| Asegúrese | Time |
| --- | --- |
| Toyota |2015-01-01T00:00:02.0000000Z |

**Solución**:

```SQL
    SELECT
        Make,
        Time
    FROM
        Input TIMESTAMP BY Time
    WHERE
        LAG(Make, 1) OVER (LIMIT DURATION(minute, 1)) <> Make
```

**Explicación**: Use **LAG** para revisar en el flujo de entrada un evento anterior y obtener el valor **Make**. A continuación, compárelo con el de la **marca** del evento actual y genere la salida del evento si son distintos.

## <a name="query-example-find-the-first-event-in-a-window"></a>Ejemplo de consulta: Búsqueda del primer evento en una ventana

**Descripción**: Busque el primer vehículo en un intervalo de cada diez minutos.

**Entrada**:

| LicensePlate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Ford |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Honda |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Toyota |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Toyota |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**Salida**:

| LicensePlate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |

**Solución**:

```SQL
    SELECT 
        LicensePlate,
        Make,
        Time
    FROM 
        Input TIMESTAMP BY Time
    WHERE 
        IsFirst(minute, 10) = 1
```

Ahora vamos a cambiar el problema y buscar el primer vehículo de una marca concreta en cada intervalo de 10 minutos.

| LicensePlate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Ford |2015-07-27T00:02:17.0000000Z |
| YHN 6970 |Toyota |2015-07-27T00:06:00.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**Solución**:

```SQL
    SELECT 
        LicensePlate,
        Make,
        Time
    FROM 
        Input TIMESTAMP BY Time
    WHERE 
        IsFirst(minute, 10) OVER (PARTITION BY Make) = 1
```

## <a name="query-example-find-the-last-event-in-a-window"></a>Ejemplo de consulta: Búsqueda del último evento en una ventana

**Descripción**: Busque el último vehículo en un intervalo de cada diez minutos.

**Entrada**:

| LicensePlate | Asegúrese | Time |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Ford |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Honda |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Toyota |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Toyota |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**Salida**:

| LicensePlate | Asegúrese | Time |
| --- | --- | --- |
| VFE 1616 |Toyota |2015-07-27T00:09:31.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**Solución**:

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
        Input.LicensePlate,
        Input.Make,
        Input.Time
    FROM
        Input TIMESTAMP BY Time 
        INNER JOIN LastInWindow
        ON DATEDIFF(minute, Input, LastInWindow) BETWEEN 0 AND 10
        AND Input.Time = LastInWindow.LastEventTime
```

**Explicación**: Hay dos pasos en la consulta. El primero consiste en buscar la última marca de tiempo en ventanas de diez minutos. El segundo paso combina los resultados de la primera consulta con el flujo original para buscar eventos que coinciden con las últimas marcas de tiempo en cada ventana. 

## <a name="query-example-detect-the-absence-of-events"></a>Ejemplo de consulta: Detección de la ausencia de eventos

**Descripción**: Compruebe que un flujo no tiene ningún valor que cumpla un criterio determinado.
Por ejemplo, ¿han entrado dos vehículos consecutivos de la misma marca en la autopista de peaje durante los últimos noventa segundos?

**Entrada**:

| Asegúrese | LicensePlate | Time |
| --- | --- | --- |
| Honda |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Honda |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Toyota |DEF-987 |2015-01-01T00:00:03.0000000Z |
| Honda |GHI-345 |2015-01-01T00:00:04.0000000Z |

**Salida**:

| Asegúrese | Time | CurrentCarLicensePlate | FirstCarLicensePlate | FirstCarTime |
| --- | --- | --- | --- | --- |
| Honda |2015-01-01T00:00:02.0000000Z |AAA-999 |ABC-123 |2015-01-01T00:00:01.0000000Z |

**Solución**:

```SQL
    SELECT
        Make,
        Time,
        LicensePlate AS CurrentCarLicensePlate,
        LAG(LicensePlate, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarLicensePlate,
        LAG(Time, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarTime
    FROM
        Input TIMESTAMP BY Time
    WHERE
        LAG(Make, 1) OVER (LIMIT DURATION(second, 90)) = Make
```

**Explicación**: Use **LAG** para revisar en el flujo de entrada un evento anterior y obtener el valor **Make**. Compárelo con el valor **MARCA** del evento actual y después genere la salida del evento en caso de que los valores sean los mismos. También puede usar **LAG** para obtener datos sobre el vehículo anterior.

## <a name="query-example-detect-the-duration-between-events"></a>Ejemplo de consulta: Detección de la duración entre eventos

**Descripción**: Busque la duración de un evento determinado. Por ejemplo, dada una secuencia de clics de web, determine el tiempo invertido en una característica.

**Entrada**:  

| Usuario | Característica | Evento | Time |
| --- | --- | --- | --- |
| user@location.com |RightMenu |Inicio |2015-01-01T00:00:01.0000000Z |
| user@location.com |RightMenu |Finalizar |2015-01-01T00:00:08.0000000Z |

**Salida**:  

| Usuario | Característica | Duration |
| --- | --- | --- |
| user@location.com |RightMenu |7 |

**Solución**:

```SQL
    SELECT
        [user], feature, DATEDIFF(second, LAST(Time) OVER (PARTITION BY [user], feature LIMIT DURATION(hour, 1) WHEN Event = 'start'), Time) as duration
    FROM input TIMESTAMP BY Time
    WHERE
        Event = 'end'
```

**Explicación**: Use la función **LAST** para recuperar el último valor **TIME** en el que el tipo de evento era **Start**. La función **LAST** usa **PARTITION BY [user]** para indicar que el resultado se calcula por usuario único. La consulta tiene un umbral máximo de una hora para el intervalo de tiempo entre eventos **Start** y **Stop**, pero se puede configurar según sea necesario **(LIMIT DURATION(hour, 1)**.

## <a name="query-example-detect-the-duration-of-a-condition"></a>Ejemplo de consulta: Detección de la duración de una condición
**Descripción**: Averigüe la duración de una condición.
Por ejemplo, supongamos que por error todos los vehículos tienen un peso incorrecto (por encima de 20 000 libras), y debe calcularse la duración del error.

**Entrada**:

| Asegúrese | Time | Peso |
| --- | --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |2000 |
| Toyota |2015-01-01T00:00:02.0000000Z |25000 |
| Honda |2015-01-01T00:00:03.0000000Z |26000 |
| Toyota |2015-01-01T00:00:04.0000000Z |25000 |
| Honda |2015-01-01T00:00:05.0000000Z |26000 |
| Toyota |2015-01-01T00:00:06.0000000Z |25000 |
| Honda |2015-01-01T00:00:07.0000000Z |26000 |
| Toyota |2015-01-01T00:00:08.0000000Z |2000 |

**Salida**:

| StartFault | EndFault |
| --- | --- |
| 2015-01-01T00:00:02.000Z |2015-01-01T00:00:07.000Z |

**Solución**:

```SQL
    WITH SelectPreviousEvent AS
    (
    SELECT
    *,
        LAG([time]) OVER (LIMIT DURATION(hour, 24)) as previousTime,
        LAG([weight]) OVER (LIMIT DURATION(hour, 24)) as previousWeight
    FROM input TIMESTAMP BY [time]
    )

    SELECT 
        LAG(time) OVER (LIMIT DURATION(hour, 24) WHEN previousWeight < 20000 ) [StartFault],
        previousTime [EndFault]
    FROM SelectPreviousEvent
    WHERE
        [weight] < 20000
        AND previousWeight > 20000
```

**Explicación**: Use **LAG** para ver el flujo de entrada que se produjo durante 24 horas y buscar instancias donde**StartFault** y **StopFault** superan el peso de < 20 000.

## <a name="query-example-fill-missing-values"></a>Ejemplo de consulta: Rellenar valores que faltan

**Descripción**: Para el flujo de eventos con valores que faltan, genere un flujo de eventos con intervalos regulares. Por ejemplo, genere un evento cada cinco segundos que notifique el punto de datos visto más recientemente.

**Entrada**:

| t | value |
| --- | --- |
| "2014-01-01T06:01:00" |1 |
| "2014-01-01T06:01:05" |2 |
| "2014-01-01T06:01:10" |3 |
| "2014-01-01T06:01:15" |4 |
| "2014-01-01T06:01:30" |5 |
| "2014-01-01T06:01:35" |6 |

**Salida (10 primeras filas)**:

| windowend | lastevent.t | lastevent.value |
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

**Solución**:

```SQL
    SELECT
        System.Timestamp() AS windowEnd,
        TopOne() OVER (ORDER BY t DESC) AS lastEvent
    FROM
        input TIMESTAMP BY t
    GROUP BY HOPPINGWINDOW(second, 300, 5)
```

**Explicación**: Esta consulta genera eventos cada cinco segundos y genera como resultado el último evento que se recibió anteriormente. El [ventana de salto](/stream-analytics-query/hopping-window-azure-stream-analytics) duración determina cuánto tiempo que remontará la consulta para encontrar el evento más reciente (300 segundos en este ejemplo).


## <a name="query-example-correlate-two-event-types-within-the-same-stream"></a>Ejemplo de consulta: Correlacionar dos tipos de evento dentro del mismo flujo

**Descripción**: A veces, es necesario generar alertas basadas en varios tipos de eventos que se produjeron en un intervalo de tiempo determinado. Por ejemplo, en un escenario de IoT de hornos domésticos, se debe generar una alerta cuando la temperatura del ventilador sea inferior a 40 y la potencia máxima durante los últimos 3 minutos sea inferior a 10.

**Entrada**:

| time | deviceId | sensorName | value |
| --- | --- | --- | --- |
| "2018-01-01T16:01:00" | "Oven1" | "temp" |120 |
| "2018-01-01T16:01:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:02:00" | "Oven1" | "temp" |100 |
| "2018-01-01T16:02:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:03:00" | "Oven1" | "temp" |70 |
| "2018-01-01T16:03:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:04:00" | "Oven1" | "temp" |50 |
| "2018-01-01T16:04:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:05:00" | "Oven1" | "temp" |30 |
| "2018-01-01T16:05:00" | "Oven1" | "power" |8 |
| "2018-01-01T16:06:00" | "Oven1" | "temp" |20 |
| "2018-01-01T16:06:00" | "Oven1" | "power" |8 |
| "2018-01-01T16:07:00" | "Oven1" | "temp" |20 |
| "2018-01-01T16:07:00" | "Oven1" | "power" |8 |
| "2018-01-01T16:08:00" | "Oven1" | "temp" |20 |
| "2018-01-01T16:08:00" | "Oven1" | "power" |8 |

**Salida**:

| eventTime | deviceId | temp | alertMessage | maxPowerDuringLast3mins |
| --- | --- | --- | --- | --- | 
| "2018-01-01T16:05:00" | "Oven1" |30 | "Elementos calefactores de cortocircuito" |15 |
| "2018-01-01T16:06:00" | "Oven1" |20 | "Elementos calefactores de cortocircuito" |15 |
| "2018-01-01T16:07:00" | "Oven1" |20 | "Elementos calefactores de cortocircuito" |15 |

**Solución**:

```SQL
WITH max_power_during_last_3_mins AS (
    SELECT 
        System.TimeStamp() AS windowTime,
        deviceId,
        max(value) as maxPower
    FROM
        input TIMESTAMP BY t
    WHERE 
        sensorName = 'power' 
    GROUP BY 
        deviceId, 
        SlidingWindow(minute, 3) 
)

SELECT 
    t1.t AS eventTime,
    t1.deviceId, 
    t1.value AS temp,
    'Short circuit heating elements' as alertMessage,
    t2.maxPower AS maxPowerDuringLast3mins
    
INTO resultsr

FROM input t1 TIMESTAMP BY t
JOIN max_power_during_last_3_mins t2
    ON t1.deviceId = t2.deviceId 
    AND t1.t = t2.windowTime
    AND DATEDIFF(minute,t1,t2) between 0 and 3
    
WHERE
    t1.sensorName = 'temp'
    AND t1.value <= 40
    AND t2.maxPower > 10
```

**Explicación**: La primera consulta `max_power_during_last_3_mins`, usa la [ventana deslizante](/stream-analytics-query/sliding-window-azure-stream-analytics) para encontrar el valor máximo del sensor de potencia para cada dispositivo, durante los últimos tres minutos. La segunda consulta se combina con la primera para encontrar el valor de potencia en la ventana más reciente relacionada con el evento actual. Y entonces, siempre que se cumplan las condiciones, se genera una alerta para el dispositivo.

## <a name="query-example-process-events-independent-of-device-clock-skew-substreams"></a>Ejemplo de consulta: Procesar los eventos de manera independiente del sesgo de reloj del dispositivo (subflujos)

**Descripción**: Los eventos pueden llegar tarde o desordenados debido a sesgos de reloj entre los productores de eventos, a sesgos de reloj entre particiones o a la latencia de red. En el ejemplo siguiente, el reloj del dispositivo para TollID 2 es de cinco segundos detrás TollID 1 y el reloj del dispositivo para TollID 3 es de diez segundos detrás de 1 TollID. 

**Entrada**:

| LicensePlate | Asegúrese | Time | TollId |
| --- | --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:01.0000000Z | 1 |
| YHN 6970 |Toyota |2015-07-27T00:00:05.0000000Z | 1 |
| QYF 9358 |Honda |2015-07-27T00:00:01.0000000Z | 2 |
| GXF 9462 |BMW |2015-07-27T00:00:04.0000000Z | 2 |
| VFE 1616 |Toyota |2015-07-27T00:00:10.0000000Z | 1 |
| RMV 8282 |Honda |2015-07-27T00:00:03.0000000Z | 3 |
| MDR 6128 |BMW |2015-07-27T00:00:11.0000000Z | 2 |
| YZK 5704 |Ford |2015-07-27T00:00:07.0000000Z | 3 |

**Salida**:

| TollId | Count |
| --- | --- |
| 1 | 2 |
| 2 | 2 |
| 1 | 1 |
| 3 | 1 |
| 2 | 1 |
| 3 | 1 |

**Solución**:

```SQL
SELECT
      TollId,
      COUNT(*) AS Count
FROM input
      TIMESTAMP BY Time OVER TollId
GROUP BY TUMBLINGWINDOW(second, 5), TollId
```

**Explicación**: La cláusula [TIMESTAMP BY OVER](/stream-analytics-query/timestamp-by-azure-stream-analytics#over-clause-interacts-with-event-ordering) examina la escala de tiempo de cada dispositivo por separado mediante subflujos. Los eventos de salida para cada TollID se generan a medida que se calculan, lo que significa que los eventos están en orden con respecto a cada TollID en lugar de que se vuelvan a ordenar como si todos los dispositivos estuvieran en el mismo reloj.

## <a name="query-example-remove-duplicate-events-in-a-window"></a>Ejemplo de consulta: Quitar eventos duplicados de una ventana

**Descripción**: Al realizar una operación, como calcular promedios de eventos en un período de tiempo determinado, se deben filtrar los eventos duplicados. En el ejemplo siguiente, el segundo evento es un duplicado de la primera.

**Entrada**:  

| DeviceId | Time | Atributo | Valor |
| --- | --- | --- | --- |
| 1 |2018-07-27T00:00:01.0000000Z |Temperatura |50 |
| 1 |2018-07-27T00:00:01.0000000Z |Temperatura |50 |
| 2 |2018-07-27T00:00:01.0000000Z |Temperatura |40 |
| 1 |2018-07-27T00:00:05.0000000Z |Temperatura |60 |
| 2 |2018-07-27T00:00:05.0000000Z |Temperatura |50 |
| 1 |2018-07-27T00:00:10.0000000Z |Temperatura |100 |

**Salida**:  

| AverageValue | DeviceId |
| --- | --- |
| 70 | 1 |
|45 | 2 |

**Solución**:

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

**Explicación**: [COUNT(DISTINCT Time)](/stream-analytics-query/count-azure-stream-analytics) devuelve el número de valores distintos de la columna Time dentro de una ventana de tiempo. A continuación, puede usar la salida de este paso para calcular el promedio por dispositivo descartando los duplicados.

## <a name="get-help"></a>Obtener ayuda

Para obtener ayuda adicional, pruebe nuestro [foro de Azure Stream Analytics](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics).

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)

