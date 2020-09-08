---
title: 'Exploración y modelado de datos con Spark: Proceso de ciencia de datos en equipos'
description: Muestra las funcionalidades de exploración y modelado de datos del kit de herramientas Spark MLlib en HDInsight Spark.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: sample
ms.date: 06/03/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath, contperfq4
ms.openlocfilehash: 406092466b7ab5ca729a08f7c703bcb30812901d
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86027518"
---
# <a name="data-exploration-and-modeling-with-spark"></a>Exploración y modelado de datos con Spark

Aprenda a usar HDInsight Spark para entrenar modelos de aprendizaje automático para la predicción de tarifas de taxi mediante Spark MLlib.

En este ejemplo se muestran los distintos pasos del [proceso de ciencia de datos en equipo](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/). Se usa un subconjunto del conjunto de datos de carreras y tarifas de taxi de 2013 en la ciudad de Nueva York para cargar, explorar y preparar los datos. A continuación, con Spark MLlib, se entrenan los modelos de clasificación binaria y regresión para predecir si se pagará una propina por el viaje y se calculará el importe de esta.

## <a name="prerequisites"></a>Requisitos previos

Necesita una cuenta de Azure y un clúster de Spark 1.6 o Spark 2.0 HDInsight para completar este tutorial. Consulte el artículo [Información general sobre la ciencia de los datos con Spark en HDInsight de Azure](spark-overview.md) para obtener instrucciones sobre cómo satisfacer estos requisitos. Ese tema también contiene una descripción de los datos de taxis de Nueva York de 2013 que se usan aquí, además de instrucciones sobre cómo ejecutar el código de un cuaderno de Jupyter Notebook en el clúster Spark. 

### <a name="spark-clusters-and-notebooks"></a>Clústeres y cuadernos de Spark

Los pasos de instalación y el código proporcionado en este tutorial son para HDInsight Spark 1.6. Sin embargo, Jupyter Notebooks se proporciona para clústeres de HDInsight Spark 1.6 y Spark 2.0. Se proporciona una descripción de los cuadernos y de los vínculos a estos en el archivo [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) del repositorio de GitHub que los contiene. No obstante, este código y los cuadernos vinculados son genéricos y deberían funcionar en cualquier clúster de Spark. Los pasos de configuración y administración del clúster pueden ser ligeramente diferentes de los que se muestran aquí si no está usando Spark en HDInsight. Para mayor comodidad, estos son los vínculos a los cuadernos de Jupyter para Spark 1.6 (para ejecutarse en el kernel pySpark del servidor de Jupyter Notebook) y Spark 2.0 (para ejecutarse en el kernel pySpark3 del servidor de Jupyter Notebook):

- [Cuadernos de Spark 1.6](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/): ofrecen información sobre cómo realizar el modelado, puntuación y exploración de datos con diversos algoritmos.
- [Cuadernos de Spark 2.0](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/): ofrecen información sobre cómo realizar tareas de regresión y clasificación. Los conjuntos de datos pueden variar, pero los pasos y conceptos son aplicables a varios conjuntos de datos.

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

> [!NOTE]
> Las descripciones siguientes guardan relación con el uso de Spark 1.6. En las versiones de Spark 2.0, use los cuadernos descritos anteriormente, con sus correspondientes vínculos. 

## <a name="setup"></a>Configurar

Spark puede leer y escribir en una instancia de Azure Storage Blob (también conocida como WASB), por lo que cualquiera de los datos existentes almacenados allí pueden procesarse mediante Spark y volver a almacenarse en WASB.

Para guardar modelos o archivos en WASB, la ruta de acceso debe especificarse correctamente. Se puede hacer referencia al contenedor predeterminado asociado al clúster de Spark con un ruta que comience con: "wasb///". Para hacer referencia a otras ubicaciones, se usa "wasb://".

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>Establecimiento de rutas de directorio para las ubicaciones de almacenamiento de WASB

El ejemplo de código siguiente especifica la ubicación de los datos que se van a leer y la ruta de acceso del directorio de almacenamiento del modelo donde se guardará la salida del modelo:

```python
# SET PATHS TO FILE LOCATIONS: DATA AND MODEL STORAGE

# LOCATION OF TRAINING DATA
taxi_train_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv";

# SET THE MODEL STORAGE DIRECTORY PATH 
# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS NEEDED.
modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 
```

### <a name="import-libraries"></a>Importación de bibliotecas

Para la configuración también es necesario importar las bibliotecas necesarias. Establezca el contexto de Spark e importe las bibliotecas necesarias con el siguiente código:

```python
# IMPORT LIBRARIES
import pyspark
from pyspark import SparkConf
from pyspark import SparkContext
from pyspark.sql import SQLContext
import matplotlib
import matplotlib.pyplot as plt
from pyspark.sql import Row
from pyspark.sql.functions import UserDefinedFunction
from pyspark.sql.types import *
import atexit
from numpy import array
import numpy as np
import datetime
```

### <a name="preset-spark-context-and-pyspark-magics"></a>Contexto de Spark preestablecido e instrucciones mágicas de PySpark

Los kernels de PySpark que se proporcionan con cuadernos de Jupyter Notebook tienen contextos preestablecidos. No es necesario establecer explícitamente los contextos de Spark o Hive antes de empezar a trabajar con la aplicación que se esté desarrollando. Estos contextos están disponibles de forma predeterminada. Estos contextos son:

* sc: Para Spark 
* sqlContext: Para Hive

El kernel PySpark proporciona algunas “instrucciones mágicas” predefinidas, que son comandos especiales que se pueden llamar con %%. Hay dos comandos de este tipo que se utilizan en estos ejemplos de código.

* **%%local**: especifica que el código de las líneas siguientes se ejecutará localmente. El código debe ser un código de Python válido.
* **%%sql -o \<variable name>** Ejecuta una consulta de Hive en sqlContext. Si se pasa el parámetro -o, el resultado de la consulta se conserva en el contexto %%local de Python como trama de datos de Pandas.

Para más información sobre los kernels de los cuadernos de Jupyter Notebook y las instrucciones mágicas predefinidas, consulte [Kernels disponibles para cuadernos de Jupyter con clústeres de Spark en HDInsight basados en Linux en HDInsight (versión preliminar)](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

## <a name="load-the-data"></a>Carga de los datos

El primer paso del proceso de ciencia de datos consiste en incorporar la información que se va a analizar desde los orígenes donde residen, a su entorno de exploración y modelado de datos. Este entorno es Spark en este tutorial. Esta sección contiene el código para completar una serie de tareas:

* incorporar la muestra de datos que se va a modelar
* leer el conjunto de datos de entrada (almacenado como un archivo .tsv)
* dar formato y limpiar los datos
* crear objetos en la memoria (RDD o tramas de datos) y almacenarlos en caché
* registrarlos como una tabla temporal en el contexto de SQL

Este es el código para la incorporación de los datos.

```python
# INGEST DATA

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT FILE FROM PUBLIC BLOB
taxi_train_file = sc.textFile(taxi_train_file_loc)

# GET SCHEMA OF THE FILE FROM HEADER
schema_string = taxi_train_file.first()
fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
fields[7].dataType = IntegerType() #Pickup hour
fields[8].dataType = IntegerType() # Pickup week
fields[9].dataType = IntegerType() # Weekday
fields[10].dataType = IntegerType() # Passenger count
fields[11].dataType = FloatType() # Trip time in secs
fields[12].dataType = FloatType() # Trip distance
fields[19].dataType = FloatType() # Fare amount
fields[20].dataType = FloatType() # Surcharge
fields[21].dataType = FloatType() # Mta_tax
fields[22].dataType = FloatType() # Tip amount
fields[23].dataType = FloatType() # Tolls amount
fields[24].dataType = FloatType() # Total amount
fields[25].dataType = IntegerType() # Tipped or not
fields[26].dataType = IntegerType() # Tip class
taxi_schema = StructType(fields)

# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
taxi_header = taxi_train_file.filter(lambda l: "medallion" in l)
taxi_temp = taxi_train_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))


# CREATE DATA FRAME
taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)

# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
taxi_df_train_cleaned = taxi_train_df.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
    .drop('direct_distance').drop('surcharge')\
    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )


# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_train_cleaned.cache()
taxi_df_train_cleaned.count()

# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
taxi_df_train_cleaned.registerTempTable("taxi_train")

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 51,72 segundos

## <a name="explore-the-data"></a>Exploración de los datos

Una vez incorporados los datos en Spark, el siguiente paso del proceso de la ciencia de los datos es conocer mejor los datos mediante la exploración y la visualización. En esta sección, examinaremos los datos de taxi mediante consultas SQL y trazaremos las variables de destino y las posibles características para su inspección visual. En concreto, trazaremos la frecuencia de los recuentos de pasajeros en las carreras de taxi, la frecuencia de los importes de las propinas y cómo varían las propinas según el tipo y el importe del pago.

### <a name="plot-a-histogram-of-passenger-count-frequencies-in-the-sample-of-taxi-trips"></a>Trazado de un histograma de frecuencias de recuento de pasajeros en el ejemplo de carreras de taxi

Este código y fragmentos de código siguientes utilizan instrucciones mágicas de SQL para consultar los ejemplos e instrucciones mágicas locales para trazar los datos.

* **Instrucciones mágicas de SQL (`%%sql`)** El kernel PySpark de HDInsight admite consultas sencillas de HiveQL en línea en sqlContext. El argumento (-o VARIABLE_NAME) conserva la salida de la consulta SQL como una trama de datos de Pandas en el servidor de Jupyter. Este valor hace que el resultado esté disponible en el modo local.
* La **`%%local`** se utiliza para ejecutar código de forma local en el servidor de Jupyter, que es el nodo principal del clúster de HDInsight. Normalmente, se utilizan juntas las instrucciones mágicas `%%local` y `%%sql` con el parámetro -o. El parámetro -o conservaría la salida de la consulta SQL localmente y luego la instrucción mágica %%local desencadenaría el siguiente conjunto de fragmento de código para ejecutarse localmente en la salida de las consultas SQL que se conserva localmente.

La salida se visualizará automáticamente después de ejecutar el código.

Esta consulta recupera los viajes por el número de pasajeros. 

```sql
# PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

# HIVEQL QUERY AGAINST THE sqlContext
%%sql -q -o sqlResults
SELECT passenger_count, COUNT(*) as trip_counts 
FROM taxi_train 
WHERE passenger_count > 0 and passenger_count < 7 
GROUP BY passenger_count 
```

Este código crea una trama de datos local a partir de la salida de la consulta y traza los datos. La instrucción mágica `%%local` crea una trama de datos local, `sqlResults`, que puede usarse para trazar la información con matplotlib. 

> [!NOTE]
> Esta instrucción mágica de PySpark se utiliza varias veces en este tutorial. Si la cantidad de datos es grande, debe muestrear para crear una trama de datos que pueda caber en la memoria local.

```python
#CREATE LOCAL DATA-FRAME AND USE FOR MATPLOTLIB PLOTTING

# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES. 
# CLICK ON THE TYPE OF PLOT TO BE GENERATED (E.G. LINE, AREA, BAR ETC.)
sqlResults
```

Este es el código para trazar los viajes por recuentos de pasajeros:

```python
# PLOT PASSENGER NUMBER VS. TRIP COUNTS
%%local
import matplotlib.pyplot as plt
%matplotlib inline

x_labels = sqlResults['passenger_count'].values
fig = sqlResults[['trip_counts']].plot(kind='bar', facecolor='lightblue')
fig.set_xticklabels(x_labels)
fig.set_title('Counts of trips by passenger count')
fig.set_xlabel('Passenger count in trips')
fig.set_ylabel('Trip counts')
plt.show()
```

**SALIDA:**

![Frecuencia de las carreras por número de pasajeros](./media/spark-data-exploration-modeling/trip-freqency-by-passenger-count.png)

Puede seleccionar entre diferentes tipos de visualizaciones (tabla, circular, línea, área o barra) mediante los botones del menú **Tipo** del cuaderno. Aquí se muestra el gráfico de barras.

### <a name="plot-a-histogram-of-tip-amounts-and-how-tip-amount-varies-by-passenger-count-and-fare-amounts"></a>Trazado de un histograma de importes de propinas y de las variaciones en las propinas según número de pasajeros y tarifas.

Utilice una consulta SQL para muestrear datos.

```sql
# PLOT HISTOGRAM OF TIP AMOUNTS AND VARIATION BY PASSENGER COUNT AND PAYMENT TYPE

# HIVEQL QUERY AGAINST THE sqlContext
%%sql -q -o sqlResults
SELECT fare_amount, passenger_count, tip_amount, tipped 
FROM taxi_train 
WHERE passenger_count > 0 
AND passenger_count < 7 
AND fare_amount > 0 
AND fare_amount < 200 
AND payment_type in ('CSH', 'CRD') 
AND tip_amount > 0 
AND tip_amount < 25
```

Esta celda de código usa la consulta SQL para crear tres trazados de los datos.

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# HISTOGRAM OF TIP AMOUNTS AND PASSENGER COUNT
ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
ax1.set_title('Tip amount distribution')
ax1.set_xlabel('Tip Amount ($)')
ax1.set_ylabel('Counts')
plt.suptitle('')
plt.show()

# TIP BY PASSENGER COUNT
ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
ax2.set_title('Tip amount by Passenger count')
ax2.set_xlabel('Passenger count')
ax2.set_ylabel('Tip Amount ($)')
plt.suptitle('')
plt.show()

# TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
ax.set_title('Tip amount by Fare amount')
ax.set_xlabel('Fare Amount ($)')
ax.set_ylabel('Tip Amount ($)')
plt.axis([-2, 100, -2, 20])
plt.show()
```

**SALIDA:** 

![Distribución del importe de las propinas](./media/spark-data-exploration-modeling/tip-amount-distribution.png)

![Importe de las propinas por número de pasajeros](./media/spark-data-exploration-modeling/tip-amount-by-passenger-count.png)

![Importe de las propinas por importe de la tarifa](./media/spark-data-exploration-modeling/tip-amount-by-fare-amount.png)

## <a name="prepare-the-data"></a>Preparación de los datos

Esta sección describe y proporciona el código para los procedimientos que se usan para preparar los datos para su uso en el modelado de aprendizaje automático. Muestra cómo realizar las siguientes tareas:

* Creación de una nueva característica mediante la discretización de horas en cubos de tiempo de tráfico
* Indexación y codificación de características categóricas
* Creación de objetos de punto con etiqueta para la entrada en funciones de aprendizaje automático
* Creación de una submuestra aleatoria de datos y su división en conjuntos de entrenamiento y de pruebas
* Ajuste de la escala de las características
* Almacenamiento de objetos en caché

### <a name="create-a-new-feature-by-binning-hours-into-traffic-time-buckets"></a>Creación de una nueva característica mediante la discretización de horas en cubos de tiempo de tráfico

Este código muestra cómo crear una nueva característica mediante la discretización de horas en cubos de tiempo de tráfico y, después, cómo almacenar en caché la trama de datos resultante en memoria. Cuando se usan repetidamente conjuntos de datos distribuidos resistentes (RDD) y tramas de datos, el almacenamiento en caché mejora los tiempos de ejecución. Por lo tanto, almacenaremos en caché los RDD y las tramas de datos en varias fases del tutorial. 

```python
# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_train 
"""
taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
# THE .COUNT() GOES THROUGH THE ENTIRE DATA-FRAME,
# MATERIALIZES IT IN MEMORY, AND GIVES THE COUNT OF ROWS.
taxi_df_train_with_newFeatures.cache()
taxi_df_train_with_newFeatures.count()
```

**SALIDA:**

126050

### <a name="index-and-encode-categorical-features-for-input-into-modeling-functions"></a>Indexación y codificación de características categóricas para la entrada en funciones de modelado

Esta sección muestra cómo indexar o codificar las características categóricas para la entrada en las funciones de modelado. Las funciones de modelado y predicción de MLlib requieren características con datos de entrada categóricos indexados o codificados antes de usarlos. Dependiendo del modelo, deberá indexar o codificarlos de maneras diferentes:  

* **modelado basado en árboles** requiere que las categorías se codifiquen como valores numéricos (por ejemplo, una característica con tres categorías se puede codificar con 0, 1, 2). Este algoritmo lo proporciona la función [StringIndexer](https://spark.apache.org/docs/latest/ml-features.html#stringindexer) de MLlib. Esta función codifica una columna de cadena de etiquetas en una columna de índices de etiqueta que se ordenan por frecuencias de etiqueta. Aunque se indexan con valores numéricos para facilitar la entrada y la manipulación de los datos, los algoritmos basados en árboles se pueden configurar para que los traten como categorías. 
* Los **modelos de regresión logística y lineal** requieren una codificación "uno de n", donde una característica con 3 categorías puede ampliarse a 3 columnas de característica, cada una de las cuales contiene 0 o 1 según la categoría de una observación. MLlib proporciona la función [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) para realizar la codificación "one-hot". Este codificador asigna una columna de índices de etiqueta a una columna de vectores binarios, con un solo valor uno como máximo. Esta codificación permite aplicar algoritmos que esperan características con valores numéricos, como la regresión logística, a características categóricas.

Este es el código para indexar y codificar características categóricas:

```python
# INDEX AND ENCODE CATEGORICAL FEATURES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES    
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer

# INDEX AND ENCODE VENDOR_ID
stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
model = stringIndexer.fit(taxi_df_train_with_newFeatures) # Input data-frame is the cleaned one from above
indexed = model.transform(taxi_df_train_with_newFeatures)
encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
encoded1 = encoder.transform(indexed)

# INDEX AND ENCODE RATE_CODE
stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
model = stringIndexer.fit(encoded1)
indexed = model.transform(encoded1)
encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
encoded2 = encoder.transform(indexed)

# INDEX AND ENCODE PAYMENT_TYPE
stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
model = stringIndexer.fit(encoded2)
indexed = model.transform(encoded2)
encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
encoded3 = encoder.transform(indexed)

# INDEX AND TRAFFIC TIME BINS
stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
model = stringIndexer.fit(encoded3)
indexed = model.transform(encoded3)
encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
encodedFinal = encoder.transform(indexed)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 1,28 segundos

### <a name="create-labeled-point-objects-for-input-into-ml-functions"></a>Creación de objetos de punto con etiqueta para la entrada en funciones de aprendizaje automático

Esta sección contiene código que muestra cómo indexar datos de texto categóricos como un tipo de datos de punto con etiqueta, y codificarlos para poder usarlos para entrenar y probar la regresión logística de MLlib y otros modelos de clasificación. Los objetos de punto con etiqueta son conjuntos de datos distribuidos resistentes (RDD) con el formato de datos de entrada que necesita la mayoría de los algoritmos de aprendizaje automático de MLlib. Un [punto con etiqueta](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) es un vector local, denso o disperso, asociado con una etiqueta o respuesta.  

Esta sección contiene código que muestra cómo indexar datos de texto categóricos como un tipo de [datos de punto](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) con etiqueta, y codificarlos para poder usarlos para entrenar y probar la regresión logística de MLlib y otros modelos de clasificación. Los objetos de punto con etiqueta son conjuntos de datos distribuidos resistentes (RDD) que constan de una etiqueta (variable destino/respuesta) y un vector de característica. Muchos algoritmos de aprendizaje automático de MLlib necesitan este formato de entrada.

Este es el código para indexar y codificar las características de texto para la clasificación binaria.

```python
# FUNCTIONS FOR BINARY CLASSIFICATION

# LOAD LIBRARIES
from pyspark.mllib.regression import LabeledPoint
from numpy import array

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingBinary(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    labPt = LabeledPoint(line.tipped, features)
    return  labPt

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC REGRESSION MODELS
def parseRowOneHotBinary(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    labPt = LabeledPoint(line.tipped, features)
    return  labPt
```

Este es el código para codificar e indexar características de texto de categorías para el análisis de regresión lineal.

```python
# FUNCTIONS FOR REGRESSION WITH TIP AMOUNT AS TARGET VARIABLE

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingRegression(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])

    labPt = LabeledPoint(line.tip_amount, features)
    return  labPt

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
def parseRowOneHotRegression(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    labPt = LabeledPoint(line.tip_amount, features)
    return  labPt
```

### <a name="create-a-random-subsampling-of-the-data-and-split-it-into-training-and-testing-sets"></a>Creación de una submuestra aleatoria de datos y su división en conjuntos de entrenamiento y de pruebas

Este código crea una muestra aleatoria de los datos (aquí se usa el 25 %). Aunque no es necesario para este ejemplo debido al tamaño del conjunto de datos, se muestra cómo realizar la muestra para que sepa cómo hacerlo cuando lo necesite. Cuando las muestras son grandes, el muestreo puede ahorrar mucho tiempo al entrenar modelos. Después, dividimos la muestra en una parte de entrenamiento (75 %) y una parte de pruebas (25 %) para el modelado de clasificación y regresión.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.sql.functions import rand

# SPECIFY SAMPLING AND SPLITTING FRACTIONS
samplingFraction = 0.25;
trainingFraction = 0.75; testingFraction = (1-trainingFraction);
seed = 1234;
encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)

# SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST
# INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS (FOR USE LATER IN AN ADVANCED TOPIC)
dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTRAINbinary = trainData.map(parseRowIndexingBinary)
indexedTESTbinary = testData.map(parseRowIndexingBinary)
oneHotTRAINbinary = trainData.map(parseRowOneHotBinary)
oneHotTESTbinary = testData.map(parseRowOneHotBinary)

# FOR REGRESSION TRAINING AND TESTING
indexedTRAINreg = trainData.map(parseRowIndexingRegression)
indexedTESTreg = testData.map(parseRowIndexingRegression)
oneHotTRAINreg = trainData.map(parseRowOneHotRegression)
oneHotTESTreg = testData.map(parseRowOneHotRegression)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 0,24 segundos

### <a name="feature-scaling"></a>Ajuste de la escala de las características

El ajuste de la escala de las características, también conocido como normalización de los datos, garantiza que características con valores situados muy en los extremos no tengan un peso excesivo en la función objetivo. El código para ajustar la escala de las características usa [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) para ajustarlas a la varianza de la unidad. MLlib lo proporciona para su uso en la regresión lineal con descenso de gradiente estocástico (SGD), un popular algoritmo para entrenar una amplia variedad de otros modelos de aprendizaje automático tales como regresiones regularizadas o máquinas de vectores de soporte (SVM).

> [!NOTE]
> Hemos descubierto que el algoritmo LinearRegressionWithSGD es sensible al ajuste de la escala de las características.

Con este código, se ajusta la escala de variables para usarlas con el algoritmo SGD lineal regularizado.

```python
# FEATURE SCALING

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.regression import LabeledPoint
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.feature import StandardScaler, StandardScalerModel
from pyspark.mllib.util import MLUtils

# SCALE VARIABLES FOR REGULARIZED LINEAR SGD ALGORITHM
label = oneHotTRAINreg.map(lambda x: x.label)
features = oneHotTRAINreg.map(lambda x: x.features)
scaler = StandardScaler(withMean=False, withStd=True).fit(features)
dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
oneHotTRAINregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))

label = oneHotTESTreg.map(lambda x: x.label)
features = oneHotTESTreg.map(lambda x: x.features)
scaler = StandardScaler(withMean=False, withStd=True).fit(features)
dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
oneHotTESTregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Tiempo empleado en ejecutar la celda anterior: 13,17 segundos

### <a name="cache-objects-in-memory"></a>Almacenamiento de objetos en caché

Para reducir el tiempo necesario para entrenar y probar los algoritmos de aprendizaje automático, puede almacenar en caché los objetos de trama de datos de entrada usados para clasificación, regresión y características con ajuste de la escala.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTRAINbinary.cache()
indexedTESTbinary.cache()
oneHotTRAINbinary.cache()
oneHotTESTbinary.cache()

# FOR REGRESSION TRAINING AND TESTING
indexedTRAINreg.cache()
indexedTESTreg.cache()
oneHotTRAINreg.cache()
oneHotTESTreg.cache()

# SCALED FEATURES
oneHotTRAINregScaled.cache()
oneHotTESTregScaled.cache()

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:** 

Tiempo empleado en ejecutar la celda anterior: 0,15 segundos

## <a name="train-a-binary-classification-model"></a>Entrenamiento de un modelo de clasificación binaria

Esta sección muestra cómo usar tres modelos para la tarea de clasificación binaria para predecir si se dio propina o no en una carrera de taxi. Los modelos que se presentan son:

* Regresión logística regularizada 
* Modelo de bosque aleatorio
* Árboles impulsados por gradiente

Cada sección de código de generación del modelo se dividirá en pasos: 

1. **entrenamiento del modelo** con un conjunto de parámetros
2. **Evaluación del modelo** en un conjunto de datos de prueba con métricas
3. **Guardado del modelo** en un blob para utilizarse en el futuro

### <a name="classification-using-logistic-regression"></a>Clasificación mediante regresión logística

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de regresión logística con [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) , que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras y tarifas de taxi de la ciudad de Nueva York.

**Entrenamiento del modelo de regresión logística con CV y barrido de hiperparámetros**

```python
# LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

# GET ACCURACY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD LIBRARIES
from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
from sklearn.metrics import roc_curve,auc
from pyspark.mllib.evaluation import BinaryClassificationMetrics
from pyspark.mllib.evaluation import MulticlassMetrics


# CREATE MODEL WITH ONE SET OF PARAMETERS
logitModel = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, iterations=20, initialWeights=None, 
                                               regParam=0.01, regType='l2', intercept=True, corrections=10, 
                                               tolerance=0.0001, validateData=True, numClasses=2)

# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
# NOTE: There are 20 coefficient terms for the 10 features, 
#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
print("Coefficients: " + str(logitModel.weights))
print("Intercept: " + str(logitModel.intercept))

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:** 

Coefficients: [0.0082065285375, -0.0223675576104, -0.0183812028036, -3.48124578069e-05, -0.00247646947233, -0.00165897881503, 0.0675394837328, -0.111823113101, -0.324609912762, -0.204549780032, -1.36499216354, 0.591088507921, -0.664263411392, -1.00439726852, 3.46567827545, -3.51025855172, -0.0471341112232, -0.043521833294, 0.000243375810385, 0.054518719222]

Intercept: -0.0111216486893

Tiempo empleado en ejecutar la celda anterior: 14,43 segundos

**Evaluación del modelo de clasificación binaria con métricas estándares**

```python
#EVALUATE LOGISTIC REGRESSION MODEL WITH LBFGS

# RECORD START TIME
timestart = datetime.datetime.now()

# PREDICT ON TEST DATA WITH MODEL
predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitModel.predict(lp.features)), lp.label))

# INSTANTIATE METRICS OBJECT
metrics = BinaryClassificationMetrics(predictionAndLabels)

# AREA UNDER PRECISION-RECALL CURVE
print("Area under PR = %s" % metrics.areaUnderPR)

# AREA UNDER ROC CURVE
print("Area under ROC = %s" % metrics.areaUnderROC)
metrics = MulticlassMetrics(predictionAndLabels)

# OVERALL STATISTICS
precision = metrics.precision()
recall = metrics.recall()
f1Score = metrics.fMeasure()
print("Summary Stats")
print("Precision = %s" % precision)
print("Recall = %s" % recall)
print("F1 Score = %s" % f1Score)


## SAVE MODEL WITH DATE-STAMP
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
dirfilename = modelDir + logisticregressionfilename;
logitModel.save(sc, dirfilename);

# OUTPUT PROBABILITIES AND REGISTER TEMP TABLE
logitModel.clearThreshold(); # This clears threshold for classification (0.5) and outputs probabilities
predictionAndLabelsDF = predictionAndLabels.toDF()
predictionAndLabelsDF.registerTempTable("tmp_results");

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**SALIDA:** 

Area under PR = 0.985297691373

Area under ROC = 0.983714670256

Summary Stats

Precision = 0.984304060189

Recall = 0.984304060189

F1 Score = 0.984304060189

Tiempo empleado en ejecutar la celda anterior: 57,61 segundos

**Trazado de la curva ROC.**

*predictionAndLabelsDF* está registrado como una tabla, *tmp_results*, en la celda anterior. *tmp_results* puede utilizarse para hacer consultas y mostrar los resultados en la trama de datos de sqlResults para el trazado. Este es el código.

```python
# QUERY RESULTS                              
%%sql -q -o sqlResults
SELECT * from tmp_results
```

Este es el código para realizar predicciones y trazar la curva ROC.

```python
# MAKE PREDICTIONS AND PLOT ROC-CURVE

# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline
from sklearn.metrics import roc_curve,auc

# MAKE PREDICTIONS
predictions_pddf = test_predictions.rename(columns={'_1': 'probability', '_2': 'label'})
prob = predictions_pddf["probability"] 
fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
roc_auc = auc(fpr, tpr)

# PLOT ROC CURVE
plt.figure(figsize=(5,5))
plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
plt.plot([0, 1], [0, 1], 'k--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend(loc="lower right")
plt.show()
```

**SALIDA:**

![Logistic regression ROC curve.png](./media/spark-data-exploration-modeling/logistic-regression-roc-curve.png)

### <a name="random-forest-classification"></a>Clasificación de bosque aleatorio

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de bosque aleatorio que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras de taxi y tarifas de Nueva York.

```python
#PREDICT WHETHER A TIP IS PAID OR NOT USING RANDOM FOREST

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import RandomForest, RandomForestModel
from pyspark.mllib.util import MLUtils
from pyspark.mllib.evaluation import BinaryClassificationMetrics
from pyspark.mllib.evaluation import MulticlassMetrics

# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

# TRAIN RANDOMFOREST MODEL
rfModel = RandomForest.trainClassifier(indexedTRAINbinary, numClasses=2, 
                                       categoricalFeaturesInfo=categoricalFeaturesInfo,
                                       numTrees=25, featureSubsetStrategy="auto",
                                       impurity='gini', maxDepth=5, maxBins=32)
## UN-COMMENT IF YOU WANT TO PRINT TREES
#print('Learned classification forest model:')
#print(rfModel.toDebugString())

# PREDICT ON TEST DATA AND EVALUATE
predictions = rfModel.predict(indexedTESTbinary.map(lambda x: x.features))
predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

# AREA UNDER ROC CURVE
metrics = BinaryClassificationMetrics(predictionAndLabels)
print("Area under ROC = %s" % metrics.areaUnderROC)

# PERSIST MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfclassificationfilename = "RandomForestClassification_" + datestamp;
dirfilename = modelDir + rfclassificationfilename;

rfModel.save(sc, dirfilename);

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Area under ROC = 0.985297691373

Tiempo empleado en ejecutar la celda anterior: 31,09 segundos

### <a name="gradient-boosting-trees-classification"></a>Clasificación de árboles impulsados por gradiente

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de árboles impulsados por gradiente que predice si se dio propina o no en una carrera, en el conjunto de datos de carreras de taxi y tarifas de Nueva York.

```python
#PREDICT WHETHER A TIP IS PAID OR NOT USING GRADIENT BOOSTING TREES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo, numIterations=5)
## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
#print('Learned classification GBT model:')
#print(bgtModel.toDebugString())

# PREDICT ON TEST DATA AND EVALUATE
predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

# AREA UNDER ROC CURVE
metrics = BinaryClassificationMetrics(predictionAndLabels)
print("Area under ROC = %s" % metrics.areaUnderROC)

# PERSIST MODEL IN A BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp;
dirfilename = modelDir + btclassificationfilename;

gbtModel.save(sc, dirfilename)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Area under ROC = 0.985297691373

Tiempo empleado en ejecutar la celda anterior: 19,76 segundos

## <a name="train-a-regression-model"></a>Entrenamiento de un modelo de regresión

Esta sección muestra cómo usar tres modelos para la tarea de regresión para predecir el importe de la propina para una carrera de taxi en función de otras características de propina. Los modelos que se presentan son:

* Regresión lineal regularizada
* Bosque aleatorio
* Árboles impulsados por gradiente

Estos modelos se describieron en la introducción. Cada sección de código de generación del modelo se dividirá en pasos: 

1. **entrenamiento del modelo** con un conjunto de parámetros
2. **Evaluación del modelo** en un conjunto de datos de prueba con métricas
3. **Guardado del modelo** en un blob para utilizarse en el futuro

### <a name="linear-regression-with-sgd"></a>regresión lineal con SGD

El código en esta sección muestra cómo usar características con ajuste de la escala para entrenar una regresión lineal que usa el descenso de gradiente estocástico (SGD) para la optimización, y cómo puntuar, evaluar y guardar el modelo en Azure Storage Blob (WASB).

> [!TIP]
> Nuestra experiencia nos indica que puede haber problemas con la convergencia de los modelos LinearRegressionWithSGD y es necesario cambiar u optimizar los parámetros cuidadosamente para obtener un modelo válido. El ajuste de la escala de las variables ayuda con la convergencia.

```python
#PREDICT TIP AMOUNTS USING LINEAR REGRESSION WITH SGD

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD LIBRARIES
from pyspark.mllib.regression import LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel
from pyspark.mllib.evaluation import RegressionMetrics
from scipy import stats

# USE SCALED FEATURES TO TRAIN MODEL
linearModel = LinearRegressionWithSGD.train(oneHotTRAINregScaled, iterations=100, step = 0.1, regType='l2', regParam=0.1, intercept = True)

# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
# NOTE: There are 20 coefficient terms for the 10 features, 
#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
print("Coefficients: " + str(linearModel.weights))
print("Intercept: " + str(linearModel.intercept))

# SCORE ON SCALED TEST DATA-SET & EVALUATE
predictionAndLabels = oneHotTESTregScaled.map(lambda lp: (float(linearModel.predict(lp.features)), lp.label))
testMetrics = RegressionMetrics(predictionAndLabels)

# PRINT TEST METRICS
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL WITH DATE-STAMP IN THE DEFAULT BLOB FOR THE CLUSTER
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
dirfilename = modelDir + linearregressionfilename;

linearModel.save(sc, dirfilename)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

Coefficients: [0.00457675809917, -0.0226314167349, -0.0191910355236, 0.246793409578, 0.312047890459, 0.359634405999, 0.00928692253981, -0.000987181489428, -0.0888306617845, 0.0569376211553, 0.115519551711, 0.149250164995, -0.00990211159703, -0.00637410344522, 0.545083566179, -0.536756072402, 0.0105762393099, -0.0130117577055, 0.0129304737772, -0.00171065945959]

Intercept: 0,853872718283

RMSE = 1.24190115863

R-sqr = 0.608017146081

Tiempo empleado en ejecutar la celda anterior: 58,42 segundos

### <a name="random-forest-regression"></a>Regresión con bosque aleatorio

El código de esta sección muestra cómo entrenar, evaluar y guardar una regresión de bosque aleatoria que predice el importe de las propinas en los datos de carreras de taxi de la ciudad de Nueva York.

```python
#PREDICT TIP AMOUNTS USING RANDOM FOREST

# RECORD START TIME
timestart= datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import RandomForest, RandomForestModel
from pyspark.mllib.util import MLUtils
from pyspark.mllib.evaluation import RegressionMetrics


## TRAIN MODEL
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                    numTrees=25, featureSubsetStrategy="auto",
                                    impurity='variance', maxDepth=10, maxBins=32)
## UN-COMMENT IF YOU WANT TO PRING TREES
#print('Learned classification forest model:')
#print(rfModel.toDebugString())

## PREDICT AND EVALUATE ON TEST DATA-SET
predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)

# TEST METRICS
testMetrics = RegressionMetrics(predictionAndLabels)
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfregressionfilename = "RandomForestRegression_" + datestamp;
dirfilename = modelDir + rfregressionfilename;

rfModel.save(sc, dirfilename);

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

RMSE = 0.891209218139

R-sqr = 0.759661334921

Tiempo empleado en ejecutar la celda anterior: 49,21 segundos

### <a name="gradient-boosting-trees-regression"></a>Regresión con árboles impulsados por gradiente

El código de esta sección muestra cómo entrenar, evaluar y guardar un modelo de árboles impulsados por gradiente que predice el importe de las propinas en los datos de carreras de taxi de Nueva York.

**Entrenamiento y evaluación**

```python
#PREDICT TIP AMOUNTS USING GRADIENT BOOSTING TREES

# RECORD START TIME
timestart= datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
from pyspark.mllib.util import MLUtils

## TRAIN MODEL
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
                                                numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)

## EVALUATE A TEST DATA-SET
predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

# TEST METRICS
testMetrics = RegressionMetrics(predictionAndLabels)
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
dirfilename = modelDir + btregressionfilename;
gbtModel.save(sc, dirfilename)

# CONVERT RESULTS TO DF AND REGISTER TEMP TABLE
test_predictions = sqlContext.createDataFrame(predictionAndLabels)
test_predictions.registerTempTable("tmp_results");

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SALIDA:**

RMSE = 0.908473148639

R-sqr = 0.753835096681

Tiempo empleado en ejecutar la celda anterior: 34,52 segundos

**Trazado**

*tmp_results* se registra como una tabla de Hive en la celda anterior. Los resultados de la tabla se envían a la trama de datos *sqlResults* para el trazado. Este es el código.

```python
# PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES

# SELECT RESULTS
%%sql -q -o sqlResults
SELECT * from tmp_results
```

Este es el código para trazar los datos mediante el servidor de Jupyter.

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline
import numpy as np

# PLOT 
ax = test_predictions_pddf.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
fit = np.polyfit(test_predictions_pddf['_1'], test_predictions_pddf['_2'], deg=1)
ax.set_title('Actual vs. Predicted Tip Amounts ($)')
ax.set_xlabel("Actual")
ax.set_ylabel("Predicted")
ax.plot(test_predictions_pddf['_1'], fit[0] * test_predictions_pddf['_1'] + fit[1], color='magenta')
plt.axis([-1, 20, -1, 20])
plt.show(ax)
```

**SALIDA:**

![Actual-vs-predicted-tip-amounts](./media/spark-data-exploration-modeling/actual-vs-predicted-tips.png)

## <a name="clean-up-objects-from-memory"></a>Limpieza de objetos de la memoria

Use `unpersist()` para eliminar objetos almacenados en la memoria caché.

```python
# REMOVE ORIGINAL DFs
taxi_df_train_cleaned.unpersist()
taxi_df_train_with_newFeatures.unpersist()

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTRAINbinary.unpersist()
indexedTESTbinary.unpersist()
oneHotTRAINbinary.unpersist()
oneHotTESTbinary.unpersist()

# FOR REGRESSION TRAINING AND TESTING
indexedTRAINreg.unpersist()
indexedTESTreg.unpersist()
oneHotTRAINreg.unpersist()
oneHotTESTreg.unpersist()

# SCALED FEATURES
oneHotTRAINregScaled.unpersist()
oneHotTESTregScaled.unpersist()
```

## <a name="save-the-models"></a>Guardar los modelos

Para consumir y puntuar un conjunto de datos independiente que se describe en el tema [Puntuación de modelos de aprendizaje automático creados con Spark](spark-model-consumption.md), deberá copiar y pegar estos nombres de archivo (que contienen los modelos guardados creados aquí) en el cuaderno de consumo de Jupyter Notebook. Este es el código para imprimir las rutas de acceso a los archivos de modelo que necesita.

```python
# MODEL FILE LOCATIONS FOR CONSUMPTION
print "logisticRegFileLoc = modelDir + \"" + logisticregressionfilename + "\"";
print "linearRegFileLoc = modelDir + \"" + linearregressionfilename + "\"";
print "randomForestClassificationFileLoc = modelDir + \"" + rfclassificationfilename + "\"";
print "randomForestRegFileLoc = modelDir + \"" + rfregressionfilename + "\"";
print "BoostedTreeClassificationFileLoc = modelDir + \"" + btclassificationfilename + "\"";
print "BoostedTreeRegressionFileLoc = modelDir + \"" + btregressionfilename + "\"";
```

**OUTPUT**

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-05-0317_03_23.516568"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-05-0317_05_21.577773"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-05-0317_04_11.950206"

randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-05-0317_06_08.723736"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-05-0317_04_36.346583"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-05-0317_06_51.737282"

## <a name="whats-next"></a>¿Qué sigue?

Ahora que ha creado los modelos de clasificación y regresión con Spark MlLib, está listo para aprender a puntuar y evaluar estos modelos. El Notebook de exploración y modelado de datos avanzado profundiza más en la inclusión de la validación cruzada, el barrido de los hiperparámetros y en la evaluación de modelos. 

**Consumo de modelos:** Para saber cómo puntuar y evaluar los modelos de clasificación y regresión creados en este tema, consulte [Puntuación y evaluación de modelos de aprendizaje automático creados con Spark](spark-model-consumption.md).

**Validación cruzada y barrido de hiperparámetros**: Consulte [Exploración y modelado avanzados de datos con Spark](spark-advanced-data-exploration-modeling.md) para saber cómo se pueden entrenar modelos con el barrido de hiperparámetros y la validación cruzada.