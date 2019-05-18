---
title: Uso compartido de vistas personalizadas de Azure Time Series Insights con direcciones URL con parámetros | Microsoft Docs
description: En este artículo se describe cómo desarrollar direcciones URL con parámetros en Azure Time Series Insights, de forma que una vista de cliente pueda compartirse fácilmente.
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.topic: conceptual
ms.workload: big-data
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: 3ddde600c2ac15c56b59051fbcd6bb0e8fbae1f6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787523"
---
# <a name="share-a-custom-view-using-a-parameterized-url"></a>Uso compartido de una vista personalizada mediante una dirección URL con parámetros

Para compartir una vista personalizada en el Explorador de Time Series Insights, puede crear mediante programación una dirección URL con parámetros de la vista personalizada.

El Explorador de Time Series Insights admite parámetros de consulta de dirección URL para especificar vistas de la experiencia de directamente desde la dirección URL. Por ejemplo, con solo la dirección URL, puede especificar un entorno de destino, un predicado de búsqueda y un intervalo de tiempo deseado. Cuando un usuario hace clic en la dirección URL personalizada, la interfaz proporciona un vínculo directo a dicho recurso en el portal de Time Series Insights. Se aplican directivas de acceso a datos.

> [!TIP]
> * Ver la versión gratuita [de demostración de Time Series Insights](https://insights.timeseries.azure.com/samples).
> * Leer el acompañamiento [Explorador de Time Series Insights](./time-series-insights-explorer.md) documentación.

## <a name="environment-id"></a>Identificador de entorno

El parámetro `environmentId=<guid>` especifica el identificador de entorno de destino. Es un componente de acceso a los datos FQDN y se puede encontrar en la esquina superior derecha de la información general del entorno en el portal de Azure. Es todo lo que va delante de `env.timeseries.azure.com`.

Un ejemplo de un parámetro de identificador de entorno es `?environmentId=10000000-0000-0000-0000-100000000108`.

## <a name="time"></a>Time

Puede especificar valores de tiempos absolutos o relativos con una dirección URL con parámetros.

### <a name="absolute-time-values"></a>Valores de tiempo absolutos

Con los valores de tiempo absolutos, use los parámetros `from=<integer>` y `to=<integer>`.

* `from=<integer>` es un valor de JavaScript en milisegundos de la hora de inicio del intervalo de búsqueda.
* `to=<integer>` es un valor de JavaScript en milisegundos de la hora de finalización del intervalo de búsqueda.

Para identificar los milisegundos de JavaScript para una fecha, consulte [Epoch & Unix Timestamp Converter](https://www.freeformatter.com/epoch-timestamp-to-date-converter.html) (Convertidor de marca de tiempo de Epoch y Unix).

### <a name="relative-time-values"></a>Valores de tiempo relativos

Con valores de tiempo relativos, use `relativeMillis=<value>`, donde *value* son los milisegundos de JavaScript a partir de los datos más recientes en el back-end.

Por ejemplo, `&relativeMillis=3600000` muestra los 60 minutos de datos más recientes.

Los valores aceptados corresponden al explorador de Time Series Insights **tiempo rápido** menú e incluyen:

* `1800000` (Últimos 30 minutos)
* `3600000` (Últimos 60 minutos)
* `10800000` (Últimas 3 horas)
* `21600000` (Últimas 6 horas)
* `43200000` (Últimas 12 horas)
* `86400000` (Últimas 24 horas)
* `604800000` (Últimos 7 días)
* `2592000000` (Últimas 30 horas)

### <a name="optional-parameters"></a>Parámetros opcionales

El `timeSeriesDefinitions=<collection of term objects>` parámetro especifica los términos de una vista de Time Series Insights:

| Parámetro | Elemento URL | DESCRIPCIÓN |
| --- | --- | --- |
| **name** | `\<string>` | El nombre del *término*. |
| **splitBy** | `\<string>` | El nombre de columna para *dividido por*. |
| **measureName** | `\<string>` | El nombre de columna de *medida*. |
| **predicate** | `\<string>` | La cláusula *where* para el filtrado en el lado del servidor. |
| **useSum** | `true` | Un parámetro opcional que especifica el uso de sum para la medida. </br>  Tenga en cuenta que si `Events` es la medida seleccionada, recuento está activado de forma predeterminada.  </br>  Si `Events` está desactivada, se selecciona la media de forma predeterminada. |

* El `multiChartStack=<true/false>` par clave-valor permite el apilamiento en el gráfico.
* El `multiChartSameScale=<true/false>` par clave-valor permite la misma escala del eje y en los términos de un parámetro opcional.  
* El `timeBucketUnit=<Unit>&timeBucketSize=<integer>` permite ajustar el control deslizante de intervalo para proporcionar más granulares o Suavizador, agrega más de vista del gráfico.  
* El `timezoneOffset=<integer>` parámetro le permite establecer la zona horaria para el gráfico a verse como un desplazamiento a la hora UTC.

| Pares de | DESCRIPCIÓN |
| --- | --- |
| `multiChartStack=false` | `true` está habilitado de forma predeterminada por lo tanto pasar `false` apilar. |
| `multiChartStack=false&multiChartSameScale=true` | Para usar la misma escala del eje Y en todos los términos es preciso habilitar el apilamiento.  Tiene `false` de forma predeterminada, por lo que pasa 'true' habilita esta funcionalidad. |
| `timeBucketUnit=<Unit>&timeBucketSize=<integer>` | Unidades = días, horas, minutos, segundos y milisegundos.  La unidad siempre debe escribirse en mayúsculas. </br> Para definir el número de unidades, pase el entero que desea a timeBucketSize.  Tenga en cuenta que puede agregar hasta de 7 días.  |
| `timezoneOffset=-<integer>` | El entero siempre está en milisegundos. </br> Tenga en cuenta que esta funcionalidad es ligeramente diferente a la que se habilita en el Explorador de Time Series Insights, donde se le permiten elegir (hora del explorador) local o UTC. |

### <a name="examples"></a>Ejemplos

Para agregar definiciones de series temporales en un entorno de Time Series Insights como un parámetro de dirección URL, agregue:

```plaintext
&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},
{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

Use las definiciones de series temporales de ejemplo para:

* El identificador de entorno
* Los últimos 60 minutos de datos
* Los términos (F1PressureID, F2TempStation y F3VibrationPL) que comprenden los parámetros opcionales

puede construir la dirección URL con parámetros siguiente para obtener una vista:

```plaintext
https://insights.timeseries.azure.com/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

> [!TIP]
> Consulte el Explorador de live [utilizando la dirección URL](https://insights.timeseries.azure.com/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]).

La dirección URL anterior describe y genera la vista del explorador de Time Series Insights:

[![Términos de tiempo Series Insights explorer](media/parameterized-url/url1.png)](media/parameterized-url/url1.png#lightbox)

La vista completa (incluido el gráfico):

[![Vista de gráfico](media/parameterized-url/url2.png)](media/parameterized-url/url2.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [consulta datos mediante C# ](time-series-insights-query-data-csharp.md).

* Obtenga información sobre la [tiempo explorador Series Insights](./time-series-insights-explorer.md).
