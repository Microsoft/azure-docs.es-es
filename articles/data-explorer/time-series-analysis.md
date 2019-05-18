---
title: Analizar datos de series temporales mediante el Explorador de datos de Azure
description: Aprenda a analizar datos de series temporales en la nube mediante el Explorador de datos de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/07/2019
ms.openlocfilehash: 7415e13a445a73af197362c6cfbd3a865a2fea02
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604048"
---
# <a name="time-series-analysis-in-azure-data-explorer"></a>Análisis de series temporales en Azure Data Explorer

Azure Data Explorer realiza la recopilación continua de datos de telemetría desde dispositivos en la nube o dispositivos de IoT. Estos datos se pueden analizar para obtener diversa información, como la supervisión del estado de los servicios, los procesos físicos de producción y las tendencias de utilización. El análisis se realiza en series temporales de métricas seleccionadas para encontrar una desviación en el patrón comparado con su patrón típico de línea de base.
Azure Data Explorer contiene compatibilidad nativa para la creación, manipulación y análisis de varias series temporales. En este tema, aprenderá cómo se usa Azure Data Explorer para crear y analizar **miles de serie temporales en segundos**, lo que permite flujos de trabajo y soluciones de supervisión casi en tiempo real.

## <a name="time-series-creation"></a>Creación de series temporales

En esta sección, vamos a crear un gran conjunto de series temporales regulares de forma sencilla e intuitiva mediante el operador `make-series` y vamos a rellenar los valores que faltan según sea necesario.
El primer paso en el análisis de series temporales es dividir y transformar la tabla de telemetría original en un conjunto de series temporales. La tabla generalmente contiene una columna de marca de tiempo, dimensiones contextuales y métricas opcionales. Las dimensiones se usan para particionar los datos. El objetivo es crear miles de series temporales por partición a intervalos de tiempo regulares.

La tabla de entrada *demo_make_series1* contiene registros de 600 K de tráfico del servicio web arbitrario. Utilice el siguiente comando para muestrear 10 registros:

```kusto
demo_make_series1 | take 10 
```

La tabla resultante contiene una columna de marca de tiempo, tres columnas de dimensiones contextuales y ninguna métrica:

|   |   |   |   |   |
| --- | --- | --- | --- | --- |
|   | Marca de tiempo | BrowserVer | OsVer | País o región |
|   | 2016-08-25 09:12:35.4020000 | Chrome 51.0 | Windows 7 | Reino Unido |
|   | 2016-08-25 09:12:41.1120000 | Chrome 52.0 | Windows 10 |   |
|   | 2016-08-25 09:12:46.2300000 | Chrome 52.0 | Windows 7 | Reino Unido |
|   | 2016-08-25 09:12:46.5100000 | Chrome 52.0 | Windows 10 | Reino Unido |
|   | 2016-08-25 09:12:46.5570000 | Chrome 52.0 | Windows 10 | República de Lituania |
|   | 2016-08-25 09:12:47.0470000 | Chrome 52.0 | Windows 8.1 | India |
|   | 2016-08-25 09:12:51.3600000 | Chrome 52.0 | Windows 10 | Reino Unido |
|   | 2016-08-25 09:12:51.6930000 | Chrome 52.0 | Windows 7 | Países bajos |
|   | 2016-08-25 09:12:56.4240000 | Chrome 52.0 | Windows 10 | Reino Unido |
|   | 2016-08-25 09:13:08.7230000 | Chrome 52.0 | Windows 10 | India |

Dado que no hay métricas, solo podemos crear un conjunto de series temporales que representen el mismo recuento de tráfico, particionado por el sistema operativo mediante la siguiente consulta:

```kusto
let min_t = toscalar(demo_make_series1 | summarize min(TimeStamp));
let max_t = toscalar(demo_make_series1 | summarize max(TimeStamp));
demo_make_series1
| make-series num=count() default=0 on TimeStamp in range(min_t, max_t, 1h) by OsVer
| render timechart 
```

- Utilice el operador [`make-series`](/azure/kusto/query/make-seriesoperator) para crear un conjunto de tres series temporales, donde:
    - `num=count()`: serie temporal de tráfico
    - `range(min_t, max_t, 1h)`: la serie temporal se crea en intervalos de 1 hora en el intervalo de tiempo (marcas de tiempo más recientes y más antiguas de los registros de la tabla)
    - `default=0`: especifique el método de relleno para los intervalos que faltan para crear series temporales regulares. O bien use [`series_fill_const()`](/azure/kusto/query/series-fill-constfunction), [`series_fill_forward()`](/azure/kusto/query/series-fill-forwardfunction), [`series_fill_backward()`](/azure/kusto/query/series-fill-backwardfunction) y [`series_fill_linear()`](/azure/kusto/query/series-fill-linearfunction) para los cambios
    - `byOsVer`: partición por sistema operativo
- La estructura de datos de series temporales reales es una matriz numérica del valor agregado por cada intervalo temporal. Usamos `render timechart` para la visualización.

En la tabla anterior, tenemos tres particiones. Podemos crear una serie temporal independiente: Windows 10 (rojo), 7 (azul) y 8.1 (verde) para cada versión de sistema operativo, tal y como se muestra en el gráfico:

![Partición de series temporales](media/time-series-analysis/time-series-partition.png)

## <a name="time-series-analysis-functions"></a>Funciones de análisis de series temporales

En esta sección, realizaremos las funciones típicas de procesamiento en serie.
Cuando se ha creado un conjunto de series temporales, Azure Data Explorer admite una lista creciente de funciones para procesar y analizarlas que se encuentran en la [documentación de series temporales](/azure/kusto/query/machine-learning-and-tsa). Describiremos algunas funciones representativas para el procesamiento y análisis de series temporales.

### <a name="filtering"></a>Filtrado

El filtrado es una práctica común en el procesamiento de señales y muy útil para tareas de procesamiento de series temporales (por ejemplo, suavizar una señal con ruido o detectar cambios).
- Hay dos funciones de filtrado genéricas:
    - [`series_fir()`](/azure/kusto/query/series-firfunction): aplicación de filtro FIR. Se utiliza para el cálculo simple de la media acumulada y la diferenciación de las series temporales para la detección de cambios.
    - [`series_iir()`](/azure/kusto/query/series-iirfunction): aplicación de filtro IIR. Se utiliza para el suavizado exponencial y la suma acumulativa.
- `Extend` la serie temporal establecida mediante la adición de una nueva serie de media acumulada de intervalos de tamaño 5 (denominada *ma_num*) a la consulta:

```kusto
let min_t = toscalar(demo_make_series1 | summarize min(TimeStamp));
let max_t = toscalar(demo_make_series1 | summarize max(TimeStamp));
demo_make_series1
| make-series num=count() default=0 on TimeStamp in range(min_t, max_t, 1h) by OsVer
| extend ma_num=series_fir(num, repeat(1, 5), true, true)
| render timechart
```

![Filtrado de series temporales](media/time-series-analysis/time-series-filtering.png)

### <a name="regression-analysis"></a>Análisis de regresión

Azure Data Explorer admite el análisis de regresión lineal segmentado para calcular la tendencia de la serie temporal.
- Utilice [series_fit_line()](/azure/kusto/query/series-fit-linefunction) para ajustar la mejor línea a una serie temporal para la detección general de tendencias.
- Utilice [series_fit_2lines()](/azure/kusto/query/series-fit-2linesfunction) para detectar cambios de tendencia, en relación con la línea de base, que sean útiles en los escenarios de supervisión.

Ejemplo de las funciones `series_fit_line()` y `series_fit_2lines()` en una consulta de serie temporal:

```kusto
demo_series2
| extend series_fit_2lines(y), series_fit_line(y)
| render linechart with(xcolumn=x)
```

![Regresión de serie temporal](media/time-series-analysis/time-series-regression.png)

- Azul: serie temporal original
- Verde: línea ajustada
- Rojo: dos líneas ajustadas

> [!NOTE]
> La función ha detectado con precisión el punto de salto (cambio de nivel).

### <a name="seasonality-detection"></a>Detección de estacionalidad

Muchas métricas siguen los patrones estacionales (periódicos). El tráfico de usuarios de los servicios en la nube suele contener patrones diarios y semanales que son más altos en la mitad del día laborable y más bajos por la noche y durante el fin de semana. Medida de sensores de IoT en intervalos periódicos. Las medidas físicas, como temperatura, presión y humedad también pueden mostrar el comportamiento estacional.

En el ejemplo siguiente se aplica la detección de estacionalidad en el tráfico de un mes de un servicio web (intervalos de 2 horas):

```kusto
demo_series3
| render timechart 
```

![Estacionalidad de series temporales](media/time-series-analysis/time-series-seasonality.png)

- Utilice [series_periods_detect()](/azure/kusto/query/series-periods-detectfunction) para detectar automáticamente los períodos de la serie temporal. 
- Utilice [series_periods_validate()](/azure/kusto/query/series-periods-validatefunction) si sabemos que una métrica debe tener uno o varios períodos distintos específicos y queremos comprobar su existencia.

> [!NOTE]
> Es una anomalía si no existen períodos específicos distintos.

```kusto
demo_series3
| project (periods, scores) = series_periods_detect(num, 0., 14d/2h, 2) //to detect the periods in the time series
| mv-expand periods, scores
| extend days=2h*todouble(periods)/1d
```

|   |   |   |   |
| --- | --- | --- | --- |
|   | periods | scores | días |
|   | 84 | 0.820622786055595 | 7 |
|   | 12 | 0.764601405803502 | 1 |

La función detecta la estacionalidad diaria y semanal. La puntuación diaria es menor que la semanal porque los días de fin de semana son diferentes de los días de la semana.

### <a name="element-wise-functions"></a>Funciones por elemento

Las operaciones aritméticas y lógicas pueden realizarse en una serie temporal. Con [series_subtract()](/azure/kusto/query/series-subtractfunction) podemos calcular una serie temporal residual, es decir, la diferencia entre la métrica sin procesar original y una suavizada, y buscar anomalías en la señal residual:

```kusto
let min_t = toscalar(demo_make_series1 | summarize min(TimeStamp));
let max_t = toscalar(demo_make_series1 | summarize max(TimeStamp));
demo_make_series1
| make-series num=count() default=0 on TimeStamp in range(min_t, max_t, 1h) by OsVer
| extend ma_num=series_fir(num, repeat(1, 5), true, true)
| extend residual_num=series_subtract(num, ma_num) //to calculate residual time series
| where OsVer == "Windows 10"   // filter on Win 10 to visualize a cleaner chart 
| render timechart
```

![Operaciones en series temporales](media/time-series-analysis/time-series-operations.png)

- Azul: serie temporal original
- Rojo: series temporales suavizadas
- Verde: series temporales residuales

## <a name="time-series-workflow-at-scale"></a>Flujo de trabajo de series temporales a escala

En el siguiente ejemplo se muestra cómo estas funciones pueden ejecutarse a escala en miles de series temporales en segundos para la detección de anomalías. Para ver algunos registros de telemetría de ejemplo de una métrica de recuento de lecturas de un servicio de base de datos durante cuatro días, ejecute la siguiente consulta:

```kusto
demo_many_series1
| take 4 
```

|   |   |   |   |   |   |
| --- | --- | --- | --- | --- | --- |
|   | TIMESTAMP | Loc | anonOp | DB | DataRead |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | 5117853934049630089 | 262 | 0 |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | 5117853934049630089 | 241 | 0 |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | -865998331941149874 | 262 | 279862 |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | 371921734563783410 | 255 | 0 |

Y estadísticas sencillas:

```kusto
demo_many_series1
| summarize num=count(), min_t=min(TIMESTAMP), max_t=max(TIMESTAMP) 
```

|   |   |   |   |
| --- | --- | --- | --- |
|   | num | min\_t | max\_t |
|   | 2177472 | 2016-09-08 00:00:00.0000000 | 2016-09-11 23:00:00.0000000 |

La creación de una serie temporal en intervalos de 1 hora de la métrica de lectura (total de cuatro días * 24 horas = 96 puntos) da como resultado una fluctuación normal del patrón:

```kusto
let min_t = toscalar(demo_many_series1 | summarize min(TIMESTAMP));  
let max_t = toscalar(demo_many_series1 | summarize max(TIMESTAMP));  
demo_many_series1
| make-series reads=avg(DataRead) on TIMESTAMP in range(min_t, max_t, 1h)
| render timechart with(ymin=0) 
```

![Serie temporal a escala](media/time-series-analysis/time-series-at-scale.png)

El comportamiento anterior es engañoso, ya que la única serie temporal normal se agrega a partir de miles de instancias diferentes que pueden tener patrones anormales. Por lo tanto, creamos una serie temporal por instancia. Una instancia se define por Loc (ubicación), anonOp (operación) y DB (máquina específica).

¿Cuántas series temporales podemos crear?

```kusto
demo_many_series1
| summarize by Loc, Op, DB
| count
```

|   |   |
| --- | --- |
|   | Count |
|   | 18339 |

Ahora, vamos a crear un conjunto de 18339 series temporales de la métrica de recuento de lecturas. Agregamos la cláusula `by` a la instrucción make-series, aplicamos la regresión lineal y seleccionamos las dos primeras series temporales que tuvieron la tendencia decreciente más significativa:

```kusto
let min_t = toscalar(demo_many_series1 | summarize min(TIMESTAMP));  
let max_t = toscalar(demo_many_series1 | summarize max(TIMESTAMP));  
demo_many_series1
| make-series reads=avg(DataRead) on TIMESTAMP in range(min_t, max_t, 1h) by Loc, Op, DB
| extend (rsquare, slope) = series_fit_line(reads)
| top 2 by slope asc 
| render timechart with(title='Service Traffic Outage for 2 instances (out of 18339)')
```

![Dos primeras series temporales](media/time-series-analysis/time-series-top-2.png)

Mostrar las instancias:

```kusto
let min_t = toscalar(demo_many_series1 | summarize min(TIMESTAMP));  
let max_t = toscalar(demo_many_series1 | summarize max(TIMESTAMP));  
demo_many_series1
| make-series reads=avg(DataRead) on TIMESTAMP in range(min_t, max_t, 1h) by Loc, Op, DB
| extend (rsquare, slope) = series_fit_line(reads)
| top 2 by slope asc
| project Loc, Op, DB, slope 
```

|   |   |   |   |   |
| --- | --- | --- | --- | --- |
|   | Loc | Op | DB | slope |
|   | Loc 15 | 37 | 1151 | -102743.910227889 |
|   | Loc 13 | 37 | 1249 | -86303.2334644601 |

En menos de dos minutos, Azure Data Explorer analizó aproximadamente 20 000 series temporales y detectó dos series temporales anormales en las que el recuento de lecturas bajó repentinamente.

Estas funcionalidades avanzadas, combinadas con el rápido rendimiento de Azure Data Explorer, proporcionan una solución única y eficaz para el análisis de series temporales.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre [del tiempo de detección de anomalías de serie y previsión](/azure/data-explorer/anomaly-detection) en el Explorador de datos de Azure.
* Obtenga información sobre [capacidades de aprendizaje automático](/azure/data-explorer/machine-learning-clustering) en el Explorador de datos de Azure.