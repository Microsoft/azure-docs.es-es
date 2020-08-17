---
title: Uso de SQL a petición (versión preliminar) para analizar Azure Open Datasets en Azure Synapse Studio (versión preliminar)
description: En este tutorial, aprenderá a realizar con facilidad el análisis de datos exploratorios mediante la combinación de diferentes Azure Open Datasets con SQL a petición (versión preliminar) y a visualizar los resultados en Azure Synapse Studio.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 2fc98e927fcf9686f0f39dae600f944b485c5a06
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089148"
---
# <a name="use-sql-on-demand-to-analyze-azure-open-datasets-and-visualize-the-results-in-azure-synapse-studio"></a>Uso de SQL a petición para analizar Azure Open Datasets y visualizar los resultados en Azure Synapse Studio

En este tutorial, aprenderá a realizar el análisis de datos exploratorios mediante la combinación de diferentes Azure Open Datasets con SQL a petición y, después, visualizando los resultados en Azure Synapse Studio.

En concreto, se analiza el [conjunto de datos de taxis de Nueva York (NYC)](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) que incluye:

- Fechas y horas de recogida y llegada a destino.
- Ubicaciones de recogida y llegada a destino. 
- Distancias de la carrera.
- Tarifas desglosadas.
- Tipos de tarifa.
- Formas de pago. 
- Recuentos de pasajeros indicados por el conductor.

## <a name="automatic-schema-inference"></a>Inferencia automática del esquema

Puesto que los datos se almacenan en el formato de archivo parquet, está disponible la inferencia de esquemas automática. Puede consultar fácilmente los datos sin enumerar los tipos de datos de todas las columnas de los archivos. También puede utilizar el mecanismo de la columna virtual y la función FilePath para filtrar un determinado subconjunto de archivos.

En primer lugar, vamos a familiarizarnos con los datos de los taxis de Nueva York mediante la ejecución de la siguiente consulta:

```sql
SELECT TOP 100 * FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
```

El fragmento a continuación muestra el resultado para los datos de los taxis de Nueva York:

![Fragmento del resultado de datos de taxis de Nueva York](./media/tutorial-data-analyst/1.png)

Del mismo modo, puede consultar el conjunto de datos de los días festivos locales y nacionales mediante la siguiente consulta:

```sql
SELECT TOP 100 * FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/holidaydatacontainer/Processed/*.parquet',
        FORMAT='PARQUET'
    ) AS [holidays]
```

El siguiente fragmento muestra el resultado para el conjunto de datos de los días festivos locales y nacionales:

![Fragmento del resultado de conjunto de datos de festivos locales y nacionales](./media/tutorial-data-analyst/2.png)

Por último, también puede consultar el conjunto de datos meteorológicos mediante la siguiente consulta:

```sql
SELECT
    TOP 100 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/isdweatherdatacontainer/ISDWeather/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [weather]
```

El siguiente fragmento muestra el resultado para el conjunto de datos meteorológicos:

![Fragmento del resultado del conjunto de datos meteorológicos](./media/tutorial-data-analyst/3.png)

Puede obtener más información sobre el significado de las columnas individuales en las descripciones de los conjuntos de datos de [taxis de Nueva York](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/), [días festivos](https://azure.microsoft.com/services/open-datasets/catalog/public-holidays/) y [datos meteorológicos](https://azure.microsoft.com/services/open-datasets/catalog/noaa-integrated-surface-data/).

## <a name="time-series-seasonality-and-outlier-analysis"></a>Análisis de series temporales, estacionalidad y valores atípicos

Puede resumir fácilmente el número anual de carreras de taxi con la siguiente consulta:

```sql
SELECT
    YEAR(tpepPickupDateTime) AS current_year,
    COUNT(*) AS rides_per_year
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
WHERE nyc.filepath(1) >= '2009' AND nyc.filepath(1) <= '2019'
GROUP BY YEAR(tpepPickupDateTime)
ORDER BY 1 ASC
```

El fragmento a continuación muestra el resultado para el número anual de carreras de taxi:

![Fragmento del resultado del número anual de carreras de taxi](./media/tutorial-data-analyst/4.png)

Los datos se pueden visualizar en Synapse Studio realizando un cambio de la vista de **Tabla** a la de **Gráfico**. Puede elegir entre diferentes tipos de gráficos: **de área**, **de barras**, **de columnas**, **de líneas**, **circular** y **de dispersión**. En este caso, vamos a trazar el gráfico **de columnas** con la columna de **Categoría** establecida en **current_year**:

![Gráfico de columnas que muestra las carreras por año](./media/tutorial-data-analyst/5.png)

A partir de esta visualización, se puede ver claramente una tendencia a la disminución en el número de carreras a lo largo de los años. Posiblemente, esta disminución se debe al aumento de la popularidad de las empresas de uso compartido de vehículos.

> [!NOTE]
> En el momento de escribir este tutorial, los datos de 2019 están incompletos. Como resultado, hay una gran caída en el número de carreras de ese año.

A continuación, vamos a centrar el análisis en un solo año, por ejemplo, 2016. La siguiente consulta devuelve el número diario de carreras durante ese año:

```sql
SELECT
    CAST([tpepPickupDateTime] AS DATE) AS [current_day],
    COUNT(*) as rides_per_day
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
WHERE nyc.filepath(1) = '2016'
GROUP BY CAST([tpepPickupDateTime] AS DATE)
ORDER BY 1 ASC
```

El fragmento siguiente muestra el resultado de esta consulta:

![Fragmento del resultado del número diario de carreras para 2016](./media/tutorial-data-analyst/6.png)

De nuevo, puede visualizar los datos fácilmente trazando el gráfico de **columnas** con la columna de **Categoría** establecida en **current_day** y la columna de **Leyenda (series)** establecida en **rides_per_day**.

![Gráfico de columnas que muestra el número diario de carreras para 2016](./media/tutorial-data-analyst/7.png)

En el gráfico trazado, puede ver que hay un patrón semanal, con sábados como el día de máxima actividad. Durante los meses de verano, hay menos carreras de taxi debido al período de vacaciones. También hay algunas reducciones significativas en el número de carreras de taxi sin un patrón claro de cuándo y por qué se producen.

A continuación, vamos a ver si esas reducciones pueden estar correlacionadas con los festivos locales y nacionales. Para ello, uniremos el conjunto de datos de carreras de los taxis de Nueva York con el conjunto de datos de días festivos locales y nacionales:

```sql
WITH taxi_rides AS
(
    SELECT
        CAST([tpepPickupDateTime] AS DATE) AS [current_day],
        COUNT(*) as rides_per_day
    FROM  
        OPENROWSET(
            BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
            FORMAT='PARQUET'
        ) AS [nyc]
    WHERE nyc.filepath(1) = '2016'
    GROUP BY CAST([tpepPickupDateTime] AS DATE)
),
public_holidays AS
(
    SELECT
        holidayname as holiday,
        date
    FROM
        OPENROWSET(
            BULK 'https://azureopendatastorage.blob.core.windows.net/holidaydatacontainer/Processed/*.parquet',
            FORMAT='PARQUET'
        ) AS [holidays]
    WHERE countryorregion = 'United States' AND YEAR(date) = 2016
)
SELECT
*
FROM taxi_rides t
LEFT OUTER JOIN public_holidays p on t.current_day = p.date
ORDER BY current_day ASC
```

![Visualización de los resultados de los conjuntos de datos de carreras de taxi de Nueva York y días festivos locales y nacionales](./media/tutorial-data-analyst/8.png)

Esta vez, queremos resaltar el número de carreras de taxi durante los días festivos locales y nacionales. Para ello, elegiremos **none** en la columna de **Categoría** y **rides_per_day** y **holiday** como columnas de **Leyenda (series)** .

![Gráfico trazado del número de carreras de taxi durante los días festivos locales y nacionales](./media/tutorial-data-analyst/9.png)

En el gráfico trazado, puede ver que durante los días festivos locales y nacionales, el número de carreras de taxi es inferior. Todavía hay una gran reducción no explicada el 23 de enero. Vamos a comprobar el tiempo en Nueva York en ese día consultando el conjunto de datos meteorológicos:

```sql
SELECT
    AVG(windspeed) AS avg_windspeed,
    MIN(windspeed) AS min_windspeed,
    MAX(windspeed) AS max_windspeed,
    AVG(temperature) AS avg_temperature,
    MIN(temperature) AS min_temperature,
    MAX(temperature) AS max_temperature,
    AVG(sealvlpressure) AS avg_sealvlpressure,
    MIN(sealvlpressure) AS min_sealvlpressure,
    MAX(sealvlpressure) AS max_sealvlpressure,
    AVG(precipdepth) AS avg_precipdepth,
    MIN(precipdepth) AS min_precipdepth,
    MAX(precipdepth) AS max_precipdepth,
    AVG(snowdepth) AS avg_snowdepth,
    MIN(snowdepth) AS min_snowdepth,
    MAX(snowdepth) AS max_snowdepth
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/isdweatherdatacontainer/ISDWeather/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [weather]
WHERE countryorregion = 'US' AND CAST([datetime] AS DATE) = '2016-01-23' AND stationname = 'JOHN F KENNEDY INTERNATIONAL AIRPORT'
```

![Visualización del resultado del conjunto de datos meteorológico](./media/tutorial-data-analyst/10.png)

Los resultados de la consulta indican que la reducción del número de viajes de taxi se debió a:

- Una tormenta de nieve ese día en Nueva York con gran acumulación de nieve (~30 cm).
- Hizo frío (una temperatura inferior a cero grados Celsius).
- Hizo viento (~ 10 m/s).

En este tutorial se ha mostrado cómo el analista de datos puede realizar rápidamente análisis de datos exploratorios, combinar de forma sencilla diferentes conjuntos de datos mediante SQL a petición y visualizar los resultados con Azure Synapse Studio.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo conectar SQL a petición a Power BI Desktop y crear informes, consulte el artículo [Conexión de SQL a petición a Power BI Desktop y creación de un informe](tutorial-connect-power-bi-desktop.md).
 
