---
title: 'Receta: Mantenimiento predictivo con Cognitive Services para macrodatos'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido se muestra cómo realizar la detección de anomalías distribuida con Cognitive Services para macrodatos.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: how-to
ms.date: 07/06/2020
ms.author: marhamil
ms.custom: devx-track-python
ms.openlocfilehash: b29fa17c21b2cdb2e764c905af3819142d8c341e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844072"
---
# <a name="recipe-predictive-maintenance-with-the-cognitive-services-for-big-data"></a>Receta: Mantenimiento predictivo con Cognitive Services para macrodatos

Esta receta muestra cómo puede usar Azure Synapse Analytics y Cognitive Services en Spark para el mantenimiento predictivo de dispositivos IoT. Seguiremos con el ejemplo de [CosmosDB y Synapse Link](https://github.com/Azure-Samples/cosmosdb-synapse-link-samples). En aras de la simplicidad, en esta receta se leerán los datos directamente desde un archivo CSV en lugar de obtener datos transmitidos a través de CosmosDB y Synapse Link. Le recomendamos encarecidamente que examine el ejemplo de Synapse Link.

## <a name="hypothetical-scenario"></a>Escenario hipotético

El escenario hipotético es una planta de energía donde los dispositivos IoT supervisan [turbinas de vapor](https://en.wikipedia.org/wiki/Steam_turbine). La colección IoTSignals tiene datos de revoluciones por minuto (RPM) y megawatts (MW) para cada turbina. Las señales de las turbinas de vapor se analizan y se detectan señales anómalas.

Podría haber valores atípicos en los datos de frecuencia aleatoria. En esas situaciones, los valores RPM aumentarán y la salida de MW se reducirá para proteger el circuito. La idea es ver los datos que se modifican al mismo tiempo, pero con distintas señales.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/).
* [Área de trabajo de Azure Synapse](https://docs.microsoft.com/azure/synapse-analytics/quickstart-create-workspace) configurada con un [grupo de Spark](https://docs.microsoft.com/azure/synapse-analytics/quickstart-create-apache-spark-pool)

## <a name="setup"></a>Configurar

### <a name="create-an-anomaly-detector-resource"></a>Creación de un recurso de Anomaly Detector

Los servicios de Azure Cognitive Services se representan por medio de recursos de Azure a los que se suscribe. Cree un recurso para Traductor mediante [Azure Portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) o la [CLI de Azure](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli). También puede:

- Vea un recurso existente en [Azure Portal](https://portal.azure.com/).

Anote el punto de conexión y la clave de este recurso, lo necesitará en esta guía.

## <a name="enter-your-service-keys"></a>Escriba las claves del servicio.

Empezaremos por agregar su clave y ubicación.

```python
service_key = None # Paste your anomaly detector key here
location = None # Paste your anomaly detector location here

assert (service_key is not None)
assert (location is not None)
```

## <a name="read-data-into-a-dataframe"></a>Lectura de datos en un elemento DataFrame

A continuación, vamos a leer el archivo IoTSignals en un DataFrame. Abra un nuevo cuaderno en el área de trabajo de Synapse y cree un elemento DataFrame desde el archivo.

```python
df_device_info = spark.read.csv("wasbs://publicwasb@mmlspark.blob.core.windows.net/iot/IoTSignals.csv", header=True, inferSchema=True)
```

### <a name="run-anomaly-detection-using-cognitive-services-on-spark"></a>Ejecución de la detección de anomalías mediante Cognitive Services en Spark

El objetivo es encontrar las instancias en las que las señales de los dispositivos de IoT estaban generando valores anómalos para ver cuándo existe algún problema y realizar un mantenimiento predictivo. Para ello, vamos a usar Anomaly Detector en Spark:

```python
from pyspark.sql.functions import col, struct
from mmlspark.cognitive import SimpleDetectAnomalies
from mmlspark.core.spark import FluentAPI

detector = (SimpleDetectAnomalies()
    .setSubscriptionKey(service_key)
    .setLocation(location)
    .setOutputCol("anomalies")
    .setGroupbyCol("grouping")
    .setSensitivity(95)
    .setGranularity("secondly"))

df_anomaly = (df_signals
    .where(col("unitSymbol") == 'RPM')
    .withColumn("timestamp", col("dateTime").cast("string"))
    .withColumn("value", col("measureValue").cast("double"))
    .withColumn("grouping", struct("deviceId"))
    .mlTransform(detector)).cache()

df_anomaly.createOrReplaceTempView('df_anomaly')
```

Veamos los datos:

```python
df_anomaly.select("timestamp","value","deviceId","anomalies.isAnomaly").show(3)
```

Esta celda debe producir un resultado similar al siguiente:

| timestamp           |   value | deviceId   | isAnomaly   |
|:--------------------|--------:|:-----------|:------------|
| 2020-05-01 18:33:51 |    3174 | dev-7      | False       |
| 2020-05-01 18:33:52 |    2976 | dev-7      | False       |
| 2020-05-01 18:33:53 |    2714 | dev-7      | False       |


 ## <a name="visualize-anomalies-for-one-of-the-devices"></a>Visualización de anomalías para uno de los dispositivos

IoTSignals.csv tiene señales procedentes de varios dispositivos IoT. Nos centraremos en un dispositivo específico y visualizaremos las salidas anómalas del dispositivo.

```python
df_anomaly_single_device = spark.sql("""
select
  timestamp,
  measureValue,
  anomalies.expectedValue,
  anomalies.expectedValue + anomalies.upperMargin as expectedUpperValue,
  anomalies.expectedValue - anomalies.lowerMargin as expectedLowerValue,
  case when anomalies.isAnomaly=true then 1 else 0 end as isAnomaly
from
  df_anomaly
where deviceid = 'dev-1' and timestamp < '2020-04-29'
order by timestamp
limit 200""")
```

Ahora que hemos creado una elemento DataFrame que representa las anomalías de un dispositivo determinado, podemos visualizar estas anomalías:

```python
import matplotlib.pyplot as plt
from pyspark.sql.functions import col

adf = df_anomaly_single_device.toPandas()
adf_subset = df_anomaly_single_device.where(col("isAnomaly") == 1).toPandas()

plt.figure(figsize=(23,8))
plt.plot(adf['timestamp'],adf['expectedUpperValue'], color='darkred', linestyle='solid', linewidth=0.25, label='UpperMargin')
plt.plot(adf['timestamp'],adf['expectedValue'], color='darkgreen', linestyle='solid', linewidth=2, label='Expected Value')
plt.plot(adf['timestamp'],adf['measureValue'], 'b', color='royalblue', linestyle='dotted', linewidth=2, label='Actual')
plt.plot(adf['timestamp'],adf['expectedLowerValue'],  color='black', linestyle='solid', linewidth=0.25, label='Lower Margin')
plt.plot(adf_subset['timestamp'],adf_subset['measureValue'], 'ro', label = 'Anomaly')
plt.legend()
plt.title('RPM Anomalies with Confidence Intervals')
plt.show()
```

Si es correcto, el resultado será similar al siguiente:

![Trazado de Anomaly Detector](../media/anomaly-output.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo realizar el mantenimiento predictivo a escala con Azure Cognitive Services, Azure Synapse Analytics y Azure CosmosDB. Para obtener más información, vea el ejemplo completo en [GitHub](https://github.com/Azure-Samples/cosmosdb-synapse-link-samples/tree/master/IoT).
