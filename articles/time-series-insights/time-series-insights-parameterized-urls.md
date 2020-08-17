---
title: 'Uso compartido de vistas personalizadas con direcciones URL con parámetros: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a crear direcciones URL con parámetros para compartir fácilmente vistas del explorador personalizadas en Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.topic: conceptual
ms.workload: big-data
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: 2673bb70582640cda97160eb31f16f7c7f1d60e6
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421188"
---
# <a name="share-a-custom-view-using-a-parameterized-url"></a>Uso compartido de una vista personalizada mediante una dirección URL con parámetros

Para compartir una vista personalizada en el Explorador de Azure Time Series Insights, puede crear una dirección URL con parámetros de la vista personalizada mediante programación.

El Explorador de Time Series Insights admite parámetros de consulta de dirección URL para especificar vistas en la experiencia directamente desde la dirección URL. Por ejemplo, con solo la dirección URL, puede especificar un entorno de destino, un predicado de búsqueda y un intervalo de tiempo deseado. Cuando un usuario selecciona la dirección URL personalizada, la interfaz proporciona un vínculo directo a dicho recurso en el portal de Azure Time Series Insights. Se aplican directivas de acceso a datos.

> [!TIP]
> * Vea la [demostración gratuita de Azure Time Series Insights](https://insights.timeseries.azure.com/samples).
> * Lea la documentación complementaria sobre el [Explorador de Azure Time Series Insights](./time-series-insights-explorer.md).

## <a name="environment-id"></a>Identificador de entorno

El parámetro `environmentId=<guid>` especifica el identificador de entorno de destino. Se trata de un componente del FQDN de acceso a datos y se puede encontrar en la esquina superior derecha de la información general del entorno en Azure Portal. Es todo lo que va delante de `env.timeseries.azure.com`.

Un ejemplo de un parámetro de identificador de entorno es `?environmentId=10000000-0000-0000-0000-100000000108`.

## <a name="time"></a>Time

Puede especificar valores de tiempos absolutos o relativos con una dirección URL con parámetros.

### <a name="absolute-time-values"></a>Valores de tiempo absolutos

Con los valores de tiempo absolutos, use los parámetros `from=<integer>` y `to=<integer>`.

* `from=<integer>` es un valor de JavaScript en milisegundos de la hora de inicio del intervalo de búsqueda.
* `to=<integer>` es un valor de JavaScript en milisegundos de la hora de finalización del intervalo de búsqueda.

> [!TIP]
> Para trasladar fácilmente fechas a milisegundos de JavaScript, pruebe el [convertidor de marca de tiempo de Epoch & Unix](https://www.freeformatter.com/epoch-timestamp-to-date-converter.html).

### <a name="relative-time-values"></a>Valores de tiempo relativos

Para establecer un valor de tiempo relativo, use `relativeMillis=<value>`, donde el *valor* está en milisegundos de JavaScript a partir de la marca de tiempo más reciente recibida de la API.

Por ejemplo, `&relativeMillis=3600000` muestra los 60 minutos de datos más recientes.

Los valores aceptados corresponden al menú de **tiempo rápido** del Explorador de Azure Time Series Insights, e incluyen:

* `1800000` (Últimos 30 minutos)
* `3600000` (Últimos 60 minutos)
* `10800000` (Últimas 3 horas)
* `21600000` (Últimas 6 horas)
* `43200000` (Últimas 12 horas)
* `86400000` (Últimas 24 horas)
* `604800000` (Últimos 7 días)
* `2592000000` (Últimas 30 horas)

### <a name="optional-parameters"></a>Parámetros opcionales

El parámetro `timeSeriesDefinitions=<collection of term objects>` especifica los términos del predicado que aparecerán en una vista de Azure Time Series Insights:

| Parámetro | Elemento de URL | Descripción |
| --- | --- | --- |
| **name** | `\<string>` | El nombre del *término*. |
| **splitBy** | `\<string>` | El nombre de columna para *dividido por*. |
| **measureName** | `\<string>` | El nombre de columna de *medida*. |
| **predicate** | `\<string>` | La cláusula *where* para el filtrado en el lado del servidor. |
| **useSum** | `true` | Un parámetro opcional que especifica el uso de sum para la medida. |

> [!NOTE]
> Si `Events` es la medida **useSum** seleccionada, el recuento se selecciona de forma predeterminada.  
> Si `Events` no se selecciona, se selecciona la media de forma predeterminada. |

* El par clave-valor `multiChartStack=<true/false>` permite el apilamiento en el gráfico.
* El par clave-valor `multiChartSameScale=<true/false>` permite la misma escala del eje Y por los términos de un parámetro opcional.  
* `timeBucketUnit=<Unit>&timeBucketSize=<integer>` permite ajustar el control deslizante de intervalos para proporcionar una vista más pormenorizada y agregada del gráfico.  
* El parámetro `timezoneOffset=<integer>` permite establecer la zona horaria del gráfico que se va a ver como un desplazamiento a la hora UTC.

| Par(es) | Descripción |
| --- | --- |
| `multiChartStack=false` | `true` está habilitado de forma predeterminada, así que pase `false` a la pila. |
| `multiChartStack=false&multiChartSameScale=true` | Para usar la misma escala del eje Y en todos los términos es preciso habilitar el apilamiento.  De forma predeterminada es `false`, por lo que al pasar `true` se habilita esta funcionalidad. |
| `timeBucketUnit=<Unit>&timeBucketSize=<integer>` | Unidades = `days`, `hours`, `minutes`, `seconds`, `milliseconds`.  La unidad siempre debe escribirse en mayúsculas. </br> Para definir el número de unidades, pase el entero que desee para **timeBucketSize**.  |
| `timezoneOffset=-<integer>` | El entero siempre está en milisegundos. |

> [!NOTE]
> Los valores de **timeBucketUnit** se pueden suavizar hasta 7 días.
> Los valores de **timezoneOffset** no son la hora UTC ni la hora local.

### <a name="examples"></a>Ejemplos

Para agregar definiciones de series temporales a un entorno de Azure Time Series Insights como un parámetro de dirección URL, anexe:

```URL parameter
&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},
{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

Use los ejemplos de definiciones de series temporales para:

* El identificador de entorno
* Los últimos 60 minutos de datos
* Los términos (**F1PressureID**, **F2TempStation** y **F3VibrationPL**) que componen los parámetros opcionales

Puede construir la siguiente dirección URL con parámetros para una vista:

```URL
https://insights.timeseries.azure.com/classic/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

[![Dirección URL con parámetros del Explorador de Azure Time Series Insights](media/parameterized-url/share-parameterized-url.png)](media/parameterized-url/share-parameterized-url.png#lightbox)

> [!TIP]
> Vea en directo el Explorador [usando el anterior ejemplo de dirección URL](https://insights.timeseries.azure.com/classic/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[%7B%22name%22:%22F1PressureId%22,%22splitBy%22:%22Id%22,%22measureName%22:%22Pressure%22,%22predicate%22:%22%27Factory1%27%22%7D,%7B%22name%22:%22F2TempStation%22,%22splitBy%22:%22Station%22,%22measureName%22:%22Temperature%22,%22predicate%22:%22%27Factory2%27%22%7D,%7B%22name%22:%22F3VibrationPL%22,%22splitBy%22:%22ProductionLine%22,%22measureName%22:%22Vibration%22,%22predicate%22:%22%27Factory3%27%22%7D]
).

La dirección URL anterior describe y genera la vista del Explorador de Azure Time Series Insights con parámetros. 

* Los predicados parametrizados.

  [![Predicados con parámetros del Explorador de Azure Time Series Insights.](media/parameterized-url/share-parameterized-url-predicates.png)](media/parameterized-url/share-parameterized-url-predicates.png#lightbox)

* La vista de gráfico completo compartida.

  [![La vista de gráfico completo compartida.](media/parameterized-url/share-parameterized-url-full-chart.png)](media/parameterized-url/share-parameterized-url-full-chart.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [consultar datos con C#](time-series-insights-query-data-csharp.md).

* Más información sobre el [Explorador de Azure Time Series Insights](./time-series-insights-explorer.md).
