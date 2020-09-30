---
title: Visualizaciones de gráficos de libro de Azure Monitor
description: Obtenga información sobre todas las visualizaciones de gráficos de libro de Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: 139c8cdb1a227595d728d2acc0b09cf1eb210715
ms.sourcegitcommit: 3c66bfd9c36cd204c299ed43b67de0ec08a7b968
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "90006387"
---
# <a name="chart-visualizations"></a>Visualizaciones de gráficos

Los libros permiten que los datos de supervisión se presenten como gráficos. Los tipos de gráficos admitidos son de líneas, barras, barras por categoría, áreas, gráficos de dispersión, circular y de tiempo. Los autores pueden optar por personalizar el alto, el ancho, la paleta de colores, la leyenda, los títulos, los mensajes de datos faltantes, etc. del gráfico, así como los tipos de ejes y los colores de las series mediante la configuración del gráfico.

Los libros admiten gráficos para registros y orígenes de datos de métricas.

## <a name="log-charts"></a>Gráficos de registros

Los registros de Azure Monitor proporcionan información detallada sobre el funcionamiento de las aplicaciones y la infraestructura a los propietarios de los recursos. A diferencia de las métricas, la información de registro no se recopila de manera predeterminada y requiere de la incorporación de algún tipo de recopilación. Sin embargo, cuando existen, los registros proporcionan mucha información sobre el estado del recurso y datos útiles para el diagnóstico. Los libros permiten presentar los datos de registro como gráficos visuales para el análisis del usuario.

### <a name="adding-a-log-chart"></a>Agregar un gráfico de registro

En el ejemplo siguiente se muestra la tendencia de solicitudes a una aplicación durante días anteriores.

1. Para cambiar el libro al modo de edición, seleccione el elemento de la barra de herramientas **Editar**.
2. Use el vínculo **Agregar consulta** para agregar un control de consulta de registro al libro.
3. Seleccione el tipo de consulta **Registro**, el tipo de recurso (por ejemplo, Application Insights) y los recursos de destino.
4. Utilice el editor de consultas para especificar el [KQL](/azure/kusto/query/) para el análisis (por ejemplo, la tendencia de las solicitudes).
5. Establezca la visualización en una de las siguientes opciones: **Área**, **Barra**, **Barra (categoría)** , **Línea**, **Circular**, **Dispersión**o **tiempo**.
6. Si es necesario, establezca otros parámetros, como el intervalo de tiempo, la visualización, el tamaño, la paleta de colores y la leyenda.

[![Captura de pantalla del gráfico de registro en modo de edición](./media/workbooks-chart-visualizations/log-chart.png)](./media/workbooks-chart-visualizations/log-chart.png#lightbox)

### <a name="log-chart-parameters"></a>Parámetros de gráfico de registro

| Parámetro | Explicación | Ejemplo |
| ------------- |:-------------|:-------------|
| `Query Type` | Tipo de consulta que se va a usar. | Registro, Resource Graph, etc. |
| `Resource Type` | Tipo de recurso de destino. | Application Insights, Log Analytics o Azure-first |
| `Resources` | Conjunto de recursos del que se obtendrá el valor de las métricas. | MyApp1 |
| `Time Range` | Periodo de tiempo para mostrar en el gráfico de registro. | Última hora, últimas 24 horas, etc. |
| `Visualization` | Visualización que se usará. | Área, barra, línea, círculo, dispersión, hora, categoría de la barra |
| `Size` | Tamaño vertical del control. | Pequeño, mediano, grande o pantalla completa. |
| `Color palette` | Paleta de colores que se va a utilizar en el gráfico. Se omite en modo de varias métricas o segmentadas. | Azul, verde, rojo, etc. |
| `Legend` | Función de agregación que se va a usar para la leyenda. | Suma o promedio de valores o Máx, mín, primero, último valor |
| `Query` | Cualquier consulta de KQL que devuelva datos en el formato esperado por la visualización del gráfico. | _solicitudes \| las solicitudes make-series= número () valor predeterminado= 0 en la marca de tiempo de hace(1D) a ahora() paso 1h_ |

### <a name="time-series-charts"></a>Gráficos de serie temporal

Los gráficos de series temporales, como los de áreas, barras, líneas, dispersión y tiempo, se pueden crear fácilmente mediante el control de consultas en los libros. La clave está en tener la información de tiempo y métricas en el conjunto de resultados.

#### <a name="simple-time-series"></a>Serie temporal simple

La siguiente consulta devuelve una tabla con dos columnas: *timestamp* y *Requests*. El control de consulta usa *timestamp* el eje X y *Requests* para el eje Y.

```kusto
requests
| summarize Requests = count() by bin(timestamp, 1h)
```

[![Captura de pantalla del gráfico de líneas de registro de una serie temporal simple.](./media/workbooks-chart-visualizations/log-chart-line-simple.png)](./media/workbooks-chart-visualizations/log-chart-line-simple.png#lightbox)

#### <a name="time-series-with-multiple-metrics"></a>Series temporales con varias métricas

La siguiente consulta devuelve una tabla con tres columnas: *timestamp*, *Requests*y *Users*. El control de consulta usa *timestamp* para las solicitudes del eje X y *Requests* & *Users* como series independientes en el eje Y.

```kusto
requests
| summarize Requests = count(), Users = dcount(user_Id) by bin(timestamp, 1h)
```

[![Captura de pantalla del gráfico de líneas de registro de una serie temporal con varias métricas.](./media/workbooks-chart-visualizations/log-chart-line-multi-metric.png)](./media/workbooks-chart-visualizations/log-chart-line-multi-metric.png#lightbox)

#### <a name="segmented-time-series"></a>Serie temporal segmentada

La consulta siguiente devuelve una tabla con tres columnas: *timestamp*, *Requests*y *RequestName*, en la que *RequestName* es una columna de categorías con los nombres de las solicitudes. En este caso, el control de consulta usa *timestamp* para el eje X y agrega una serie por cada valor de *RequestName*.

```
requests
| summarize Request = count() by bin(timestamp, 1h), RequestName = name
```

[![Captura de pantalla del gráfico de líneas de registro de una serie temporal segmentada.](./media/workbooks-chart-visualizations/log-chart-line-segmented.png)](./media/workbooks-chart-visualizations/log-chart-line-segmented.png#lightbox)

### <a name="summarize-vs-make-series"></a>Operador summarize frente a make-series

En los ejemplos de la sección anterior se usa el operador `summarize`, porque es más fácil de entender. Sin embargo, summarize tiene una limitación importante, ya que omite la fila de resultados si no hay ningún elemento en el intervalo. Como resultado, puede desplazar el periodo de tiempo del gráfico en función de si los intervalos vacíos están al principio o al final del intervalo de tiempo.

Normalmente, es mejor usar el operador `make-series` para crear datos de series temporales, que tiene la opción de proporcionar valores predeterminados para los intervalos vacíos.

En la siguiente consulta se usa el operador `make-series`.

```kusto
requests
| make-series Requests = count() default = 0 on timestamp from ago(1d) to now() step 1h by RequestName = name
```

La siguiente consulta muestra un gráfico similar con el operador `summarize`,

```kusto
requests
| summarize Request = count() by bin(timestamp, 1h), RequestName = name
```

aunque el conjunto de resultados subyacente es diferente. El usuario solo debe establecer la visualización en áreas, líneas, barras o tiempos, y los libros se encargarán del resto.

[![Captura de pantalla del gráfico de líneas de registro realizado a partir de una consulta con make-series](./media/workbooks-chart-visualizations/log-chart-line-make-series.png)](./media/workbooks-chart-visualizations/log-chart-line-make-series.png#lightbox)

### <a name="categorical-bar-chart-or-histogram"></a>Gráfico de barras por categoría o histograma

Los gráficos por categorías permiten a los usuarios representar una dimensión o una columna en el eje X de un gráfico, lo que resulta especialmente útil en los histogramas. En el ejemplo siguiente se muestra la distribución de solicitudes por código de resultado.

```kusto
requests
| summarize Requests = count() by Result = strcat('Http ', resultCode)
| order by Requests desc
```

La consulta devuelve dos columnas: La métrica *Requests* y la categoría *Result*. Cada valor de la columna *Result* tendrá su propia barra en el gráfico, con una altura proporcional a la *métrica Requests*.

[![Captura de pantalla de un gráfico de barras por categoría que muestra las solicitudes por código de resultado](./media/workbooks-chart-visualizations/log-chart-categorical-bar.png)](./media/workbooks-chart-visualizations/log-chart-categorical-bar.png#lightbox)

### <a name="pie-charts"></a>Gráficos circulares

Los gráficos circulares permiten visualizar la proporción numérica. En el ejemplo siguiente se muestra la proporción de solicitudes por código de resultado.

```kusto
requests
| summarize Requests = count() by Result = strcat('Http ', resultCode)
| order by Requests desc
```

La consulta devuelve dos columnas: La métrica *Requests* y la categoría *Result*. Cada valor de la columna *Result* tendrá su propio segmento en el gráfico, con un tamaño proporcional a la *métrica Requests*.

[![Captura de pantalla del gráfico circular con segmentos que representan el código de resultado](./media/workbooks-chart-visualizations/log-chart-pie-chart.png)](./media/workbooks-chart-visualizations/log-chart-pie-chart.png#lightbox)

## <a name="metric-charts"></a>Gráficos de métricas

La mayoría de los recursos de Azure emiten datos de métrica sobre el estado y el mantenimiento (por ejemplo, uso de la CPU, disponibilidad del almacenamiento, recuento de transacciones de bases de datos, solicitudes de aplicación con errores, entre otros). Los libros permiten la visualización de estos datos como gráficos de series temporales.

### <a name="adding-a-metric-chart"></a>Agregar un gráfico de métricas

En el ejemplo siguiente se mostrará el número de transacciones en una cuenta de almacenamiento durante la hora anterior. De este modo, el propietario del almacenamiento podrá ver la tendencia de las transacciones y buscar anomalías en el comportamiento.

1. Para cambiar el libro al modo de edición, seleccione el elemento de la barra de herramientas **Editar**.
2. Use el vínculo **Agregar métricas** para agregar un control de métricas al libro.
3. Seleccione un tipo de recurso (por ejemplo, cuenta de almacenamiento), los recursos de destino, el espacio de nombres de métricas y el nombre, así como la agregación que se va a usar.
4. Si es necesario, establezca otros parámetros, como el intervalo de tiempo, dividir por, la visualización, el tamaño y la paleta de colores.

[![Captura de pantalla del gráfico de métricas en modo de edición](./media/workbooks-chart-visualizations/metric-chart.png)](./media/workbooks-chart-visualizations/metric-chart.png#lightbox)

### <a name="metric-chart-parameters"></a>Parámetros de gráfico de métricas

| Parámetro | Explicación | Ejemplo |
| ------------- |:-------------|:-------------|
| `Resource Type` | Tipo de recurso de destino. | Almacenamiento o máquina virtual. |
| `Resources` | Conjunto de recursos del que se obtendrá el valor de las métricas. | MyStorage1 |
| `Namespace` | Espacio de nombres con la métrica. | Almacenamiento > Blob |
| `Metric` | Métrica que se visualizará. | Almacenamiento > Blob > Transacciones |
| `Aggregation` | Función de agregación que se va a aplicar a la métrica. | Suma, recuento, promedio, etc. |
| `Time Range` | Periodo de tiempo que se mostrará de la métrica. | Última hora, últimas 24 horas, etc. |
| `Visualization` | Visualización que se usará. | Área, barras, líneas, dispersión, cuadrícula |
| `Split By` | Dividir, de manera opcional, la métrica en una dimensión. | Transacciones por tipo geográfico |
| `Size` | Tamaño vertical del control. | Pequeño, mediano o grande |
| `Color palette` | Paleta de colores que se va a utilizar en el gráfico. Se omite si se usa el parámetro `Split by`. | Azul, verde, rojo, etc. |

### <a name="examples"></a>Ejemplos

Transacciones divididas por nombre de API como gráfico de líneas:

[![Captura de pantalla del gráfico de líneas de métricas que muestra las transacciones de almacenamiento divididas por nombre de API](./media/workbooks-chart-visualizations/metric-chart-storage-split-line.png)](./media/workbooks-chart-visualizations/metric-chart-storage-split-line.png#lightbox)

Transacciones divididas por tipo de respuesta como un gráfico de barras de gran tamaño:

[![Captura de pantalla del gráfico de barras de métricas de gran tamaño que muestra las transacciones de almacenamiento divididas por tipo de respuesta](./media/workbooks-chart-visualizations/metric-chart-storage-bar-large.png)](./media/workbooks-chart-visualizations/metric-chart-storage-bar-large.png#lightbox)

Promedio de latencia como gráfico de dispersión:

[![Captura de pantalla del gráfico de dispersión de métricas que muestra la latencia de almacenamiento](./media/workbooks-chart-visualizations/metric-chart-storage-scatter.png)](./media/workbooks-chart-visualizations/metric-chart-storage-scatter.png#lightbox)

## <a name="chart-settings"></a>Configuración del gráfico

Los autores pueden usar la configuración del gráfico para personalizar los campos que se usan en los ejes del gráfico, las unidades del eje, el formato personalizado, los intervalos, los comportamientos de agrupación, las leyendas y los colores de las series.

### <a name="the-settings-tab"></a>La pestaña Configuración

La pestaña Configuración controla lo siguiente:

- La configuración del eje, incluido el formato personalizado de los campos que permite a los usuarios establecer el formato de número en los valores del eje y los intervalos personalizados.
- La configuración de agrupación, incluidos los límites de campo antes de crear un grupo "Otros".
- Configuración de la leyenda, incluida la visualización de métricas (nombre de la serie, colores y números) en la parte inferior o una leyenda (nombres y colores de la serie).

![Captura de pantalla de la configuración del gráfico.](./media/workbooks-chart-visualizations/chart-settings.png)

#### <a name="custom-formatting"></a>Formato personalizado

Entre las opciones de formato de números se incluyen:

| Opción de formato             | Explicación                                                                                           |
|:---------------------------- |:-------------------------------------------------------------------------------------------------------|
| `Units`                      | Unidades de la columna: varias opciones para porcentaje, recuentos, tiempo, bytes, recuento/tiempo, bytes/hora, etc. Por ejemplo, la unidad para un valor de 1234 se puede establecer en milisegundos y se representa como 1,234 s.                                  |
| `Style`                      | Formato para representarlo: decimal, divida, porcentaje.                                               |
| `Show group separator`       | Casilla para mostrar los separadores de grupos. Presenta 1234 como 1,234 en Estados Unidos.                                    |
| `Minimum integer digits`     | Número mínimo de dígitos enteros que se van a usar (el valor predeterminado es 1).                                                   |
| `Minimum fractional digits`  | Número mínimo de dígitos fraccionarios que se van a usar (el valor predeterminado es 0).                                                |
| `Maximum fractional digits`  | Número máximo de dígitos fraccionarios que se van a usar.                                                            |
| `Minimum significant digits` | Número mínimo de dígitos significativos que se van a usar (el valor predeterminado es 1).                                               |
| `Maximum significant digits` | Número máximo de dígitos significativos que se van a usar.                                                           |

![Captura de pantalla de la configuración del eje X.](./media/workbooks-chart-visualizations/number-format-settings.png)

### <a name="the-series-tab"></a>Pestaña Serie

La pestaña de configuración de la serie permite ajustar las etiquetas y los colores que se muestran en las series del gráfico.

- El campo `Series name` se usa para buscar coincidencias con una serie de los datos y, si coinciden, se verán la etiqueta y el color para mostrar.
- El campo `Comment` es útil para los autores de plantillas, ya que los traductores pueden usarlo para localizar las etiquetas de visualización.

![Captura de pantalla de la configuración de la serie.](./media/workbooks-chart-visualizations/series-settings.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo crear un [icono en los libros](workbooks-tile-visualizations.md).
- Obtenga información sobre cómo crear [libros interactivos](workbooks-interactive.md).