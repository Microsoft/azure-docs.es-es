---
title: 'Receta: Exploración de arte inteligente con Cognitive Services para macrodatos'
titleSuffix: Azure Cognitive Services
description: Esta receta muestra cómo crear una base de datos de arte en la que se pueden hacer búsquedas con Azure Search y MMLSpark.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: how-to
ms.date: 07/06/2020
ms.author: marhamil
ms.custom: devx-track-python
ms.openlocfilehash: 0a94c66eb51298db226ceec5da5c86666576052a
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87850498"
---
# <a name="recipe-intelligent-art-exploration-with-the-cognitive-services-for-big-data"></a>Receta: Exploración de arte inteligente con Cognitive Services para macrodatos

En este ejemplo, usaremos Cognitive Services para macrodatos para agregar anotaciones inteligentes a la colección de acceso abierto del Museo Metropolitano de Arte (MET). Esto nos permitirá crear un motor de búsqueda inteligente mediante Azure Search incluso sin anotaciones manuales. 

## <a name="prerequisites"></a>Requisitos previos

* Debe tener una clave de suscripción para Computer Vision y Cognitive Search. Siga las instrucciones que se indican en [Creación de una cuenta de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) para suscribirse a Computer Vision y obtener su clave.
  > [!NOTE]
  > Para obtener información sobre precios, consulte [Azure Cognitive Search](https://azure.microsoft.com/services/search/#pricing).

## <a name="import-libraries"></a>Importación de bibliotecas

Ejecute el siguiente comando para importar bibliotecas para esta receta.

```python
import os, sys, time, json, requests
from pyspark.ml import Transformer, Estimator, Pipeline
from pyspark.ml.feature import SQLTransformer
from pyspark.sql.functions import lit, udf, col, split
```

## <a name="set-up-subscription-keys"></a>Configuración de claves de suscripción

Ejecute el comando siguiente para configurar las variables de las claves de servicio. Inserte las claves de suscripción para Computer Vision y Azure Cognitive Search.

```python
VISION_API_KEY = 'INSERT_COMPUTER_VISION_SUBSCRIPTION_KEY'
AZURE_SEARCH_KEY = 'INSERT_AZURE_COGNITIVE_SEARCH_SUBSCRIPTION_KEY'
search_service = "mmlspark-azure-search"
search_index = "test"
```

## <a name="read-the-data"></a>Lectura de los datos

Ejecute el siguiente comando para cargar los datos de la colección de acceso abierto del MET.

```python
data = spark.read\
  .format("csv")\
  .option("header", True)\
  .load("wasbs://publicwasb@mmlspark.blob.core.windows.net/metartworks_sample.csv")\
  .withColumn("searchAction", lit("upload"))\
  .withColumn("Neighbors", split(col("Neighbors"), ",").cast("array<string>"))\
  .withColumn("Tags", split(col("Tags"), ",").cast("array<string>"))\
  .limit(25)
```

<a name="AnalyzeImages"></a>

## <a name="analyze-the-images"></a>Análisis de las imágenes

Ejecute el siguiente comando para usar Computer Vision en la colección de ilustraciones de obras de arte de acceso abierto del MET. Como resultado, obtendrá características visuales de las obras de arte.

```python
from mmlspark.cognitive import AnalyzeImage
from mmlspark.stages import SelectColumns

#define pipeline
describeImage = (AnalyzeImage()
  .setSubscriptionKey(VISION_API_KEY)
  .setLocation("eastus")
  .setImageUrlCol("PrimaryImageUrl")
  .setOutputCol("RawImageDescription")
  .setErrorCol("Errors")
  .setVisualFeatures(["Categories", "Tags", "Description", "Faces", "ImageType", "Color", "Adult"])
  .setConcurrency(5))

df2 = describeImage.transform(data)\
  .select("*", "RawImageDescription.*").drop("Errors", "RawImageDescription")
```

<a name="CreateSearchIndex"></a>

## <a name="create-the-search-index"></a>Creación del índice de búsqueda

Ejecute el comando siguiente para escribir los resultados en Azure Search para crear un motor de búsqueda de las obras de arte con metadatos enriquecidos de Computer Vision.

```python
from mmlspark.cognitive import *
df2.writeToAzureSearch(
  subscriptionKey=AZURE_SEARCH_KEY,
  actionCol="searchAction",
  serviceName=search_service,
  indexName=search_index,
  keyCol="ObjectID"
)
```

## <a name="query-the-search-index"></a>Consulta del índice de búsqueda

Ejecute el siguiente comando para consultar el índice de Azure Search.

```python
url = 'https://{}.search.windows.net/indexes/{}/docs/search?api-version=2019-05-06'.format(search_service, search_index)
requests.post(url, json={"search": "Glass"}, headers = {"api-key": AZURE_SEARCH_KEY}).json()
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a usar [Cognitive Services para macrodatos para la detección de anomalías](anomaly-detection.md).

