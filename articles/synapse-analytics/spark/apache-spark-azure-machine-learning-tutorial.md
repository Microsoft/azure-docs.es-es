---
title: Ejecución de experimentos mediante el aprendizaje automático automatizado de Azure
description: Ejecución de experimentos de aprendizaje automático mediante Apache Spark y el aprendizaje automático automatizado de Azure
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 06/30/2020
ms.author: midesa
ms.reviewer: jrasnick,
ms.openlocfilehash: 57830ae14c5d6653f6a2225924160fcb75646c12
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590539"
---
# <a name="run-experiments-using-azure-automated-ml-and-apache-spark"></a>Ejecución de experimentos mediante el aprendizaje automático automatizado de Azure y Apache Spark

Azure Machine Learning es un entorno en la nube que permite entrenar, implementar, automatizar, administrar y realizar un seguimiento de los modelos de aprendizaje automático. 

En este tutorial, usará el [aprendizaje automático automatizado](https://docs.microsoft.com/azure/machine-learning/concept-automated-ml) de Azure Machine Learning para crear un modelo de regresión para predecir las tarifas de taxi de Nueva York. Este proceso acepta tanto valores de configuración como datos de entrenamiento y, después, realiza una iteración automática por las combinaciones de diferentes métodos de estandarización o normalización de características, modelos y valores de hiperparámetros para llegar al mejor modelo.

En este tutorial, aprenderá a realizar las tareas siguientes:
- Descargar los datos mediante Apache Spark y Azure Open Datasets
- Transformar y limpiar los datos mediante dataframes de Apache Spark
- Entrenar un modelo de regresión con aprendizaje automático automatizado
- Calcular la precisión del modelo

### <a name="before-you-begin"></a>Antes de empezar
- Cree un grupo de Apache Spark, para lo que puede seguir el tutorial [Creación de un grupo de Apache Spark](../quickstart-create-apache-spark-pool-studio.md).
- Si no tiene un área de trabajo de Azure Machine Learning, complete el [tutorial de configuración del área de trabajo de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/tutorial-1st-experiment-sdk-setup). 

### <a name="understand-regression-models"></a>Descripción de los modelos de regresión
Los *modelos de regresión* predicen los valores de salida numéricos basados en indicadores independientes. En la regresión, el objetivo es ayudar a establecer la relación entre esas variables de predicción independientes mediante la estimación de cómo una variable afecta a las otras.  

### <a name="regression-analysis-example-on-the-nyc-taxi-data"></a>Ejemplo de análisis de regresión en los datos de los taxis de Nueva York
En este ejemplo, usará Spark para realizar un análisis de los datos de las propinas de los trayectos en taxi de Nueva York. Los datos están disponibles a través de [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Este subconjunto del conjunto de datos contiene información sobre las carreras de Yellow Taxi, incluida información sobre cada carrera, la hora y la ubicación de inicio y fin, el costo y otros atributos interesantes.

> [!IMPORTANT]
> 
> Puede haber cargos adicionales por la extracción de estos datos de su ubicación de almacenamiento. En los pasos siguientes, desarrollará un modelo para predecir las tarifas de los taxis de Nueva York. 
> 

##  <a name="download-and-prepare-the-data"></a>Descarga y preparación de los datos

1. Cree un cuaderno con el kernel de PySpark. Para obtener instrucciones al respecto, consulte la sección [Creación un cuaderno](https://docs.microsoft.com/azure/synapse-analytics/quickstart-apache-spark-notebook#create-a-notebook.).
   
   > [!Note]
   > 
   > Por la existencia del kernel PySpark, no necesitará crear ningún contexto explícitamente. El contexto de Spark se crea automáticamente al ejecutar la primera celda de código.
   >

2. Dado que los datos sin procesar están en formato de Parquet, puede usar el contexto de Spark para extraer el archivo a la memoria como dataframe directamente. Cree un dataframe de Spark, para lo que debe recuperar los datos mediante Open Datasets API. En este caso, usará las propiedades del *esquema de lectura* del dataframe de Spark para inferir los tipos de datos y el esquema. 
   
   ```python
   blob_account_name = "azureopendatastorage"
   blob_container_name = "nyctlc"
   blob_relative_path = "yellow"
   blob_sas_token = r""

   # Allow Spark to read from Blob remotely
   wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name),blob_sas_token)

   # Spark read parquet, note that it won't load any data yet by now
   df = spark.read.parquet(wasbs_path)
   ```

3. En función del tamaño del grupo de Spark (versión preliminar), los datos sin procesar pueden ser demasiado grandes o tardar demasiado para poder trabajar con ellos. Puede usar los filtros ```start_date``` y ```end_date``` para reducir el volumen de los datos. Este filtro devuelve un mes de datos. Una vez que se haya filtrado el dataframe, también ejecutará la función ```describe()``` en el dataframe nuevo para ver las estadísticas de resumen de cada campo. 

   Según las estadísticas de resumen, podemos ver que hay algunos valores irregulares y atípicos en los datos. Por ejemplo, las estadísticas muestran que la distancia recorrida mínima es menor que 0. Habrá que filtrar estos puntos de datos irregulares para que no aparezcan.
   
   ```python
   # Create an ingestion filter
   start_date = '2015-01-01 00:00:00'
   end_date = '2015-12-31 00:00:00'

   filtered_df = df.filter('tpepPickupDateTime > "' + start_date + '" and tpepPickupDateTime < "' + end_date + '"')

   filtered_df.describe().show()
   ```

4. Ahora, se generarán características desde el conjunto de datos, para lo que se selecciona un conjunto de columnas y se crean varias características basadas en tiempo en el campo de fecha y hora de recogida. También se filtrarán los valores atípicos que se identificaron en el paso anterior y, después, se quitarán las últimas columnas, que no son necesarias para el entrenamiento.
   
   ```python
   from datetime import datetime
   from pyspark.sql.functions import *

   # To make development easier, faster and less expensive down sample for now
   sampled_taxi_df = filtered_df.sample(True, 0.001, seed=1234)

   taxi_df = sampled_taxi_df.select('vendorID', 'passengerCount', 'tripDistance',  'startLon', 'startLat', 'endLon' \
                                , 'endLat', 'paymentType', 'fareAmount', 'tipAmount'\
                                , column('puMonth').alias('month_num') \
                                , date_format('tpepPickupDateTime', 'hh').alias('hour_of_day')\
                                , date_format('tpepPickupDateTime', 'EEEE').alias('day_of_week')\
                                , dayofmonth(col('tpepPickupDateTime')).alias('day_of_month')
                                ,(unix_timestamp(col('tpepDropoffDateTime')) - unix_timestamp(col('tpepPickupDateTime'))).alias('trip_time'))\
                        .filter((sampled_taxi_df.passengerCount > 0) & (sampled_taxi_df.passengerCount < 8)\
                                & (sampled_taxi_df.tipAmount >= 0)\
                                & (sampled_taxi_df.fareAmount >= 1) & (sampled_taxi_df.fareAmount <= 250)\
                                & (sampled_taxi_df.tipAmount < sampled_taxi_df.fareAmount)\
                                & (sampled_taxi_df.tripDistance > 0) & (sampled_taxi_df.tripDistance <= 200)\
                                & (sampled_taxi_df.rateCodeId <= 5)\
                                & (sampled_taxi_df.paymentType.isin({"1", "2"})))
   taxi_df.show(10)
   ```
   
Como puede ver, se creará un dataframe con columnas adicionales para el día del mes, la hora de recogida, el día de la semana y el tiempo total del recorrido. 

![Imagen del dataframe del taxi.](./media/apache-spark-machine-learning-aml-notebook/aml-dataset.png)

## <a name="generate-test-and-validation-datasets"></a>Generación de conjuntos de datos de prueba y validación

Una vez que tenemos el conjunto de datos final, podemos dividir los datos en conjuntos de entrenamiento y de prueba mediante el uso de la función ```random_ split ``` de Spark. Con las ponderaciones proporcionadas, esta función divide aleatoriamente los datos en el conjunto de datos de entrenamiento para el entrenamiento del modelo y el conjunto de datos de validación para las pruebas.

```python
# Random split dataset using spark, convert Spark to Pandas
training_data, validation_data = taxi_df.randomSplit([0.8,0.2], 223)

```

Este paso garantiza que los puntos de datos para probar el modelo terminado que no se hayan usado, se usarán para entrenar el modelo. 

## <a name="connect-to-an-azure-machine-learning-workspace"></a>Conexión a un área de trabajo de Azure Machine Learning
En Azure Machine Learning, un **área de trabajo** es una clase que acepta una suscripción de Azure e información sobre los recursos. También crea un recurso en la nube para supervisar y realizar un seguimiento de las ejecuciones del modelo. En este paso, se creará un objeto de área de trabajo a partir del área de trabajo de Azure Machine Learning existente.
   
```python
from azureml.core import Workspace

# Enter your workspace subscription, resource group, name, and region.
subscription_id = "<enter your subscription ID>" #you should be owner or contributor
resource_group = "<enter your resource group>" #you should be owner or contributor
workspace_name = "<enter your workspace name>" #your workspace name
workspace_region = "<enter workspace region>" #your region

ws = Workspace(workspace_name = workspace_name,
               subscription_id = subscription_id,
               resource_group = resource_group)

```

## <a name="convert-a-dataframe-to-an-azure-machine-learning-dataset"></a>Conversión de un dataframe en un conjunto de datos de Azure Machine Learning
Para enviar un experimento remoto, es necesario convertir el conjunto de información en un ```TabularDatset``` de Azure Machine Learning. [TabularDataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py) representa los datos en formato tabular, para lo que analiza los archivos proporcionados.

El código siguiente obtiene el área de trabajo existente y el almacén de datos predeterminado de Azure Machine Learning. Luego, pasa las ubicaciones del almacén de datos y de los archivos al parámetro de la ruta de acceso para crear un elemento ```TabularDataset```. 

```python
import pandas 
from azureml.core import Dataset

# Get the AML Default Datastore
datastore = ws.get_default_datastore()
training_pd = training_data.toPandas().to_csv('training_pd.csv', index=False)

# Convert into AML Tabular Dataset
datastore.upload_files(files = ['training_pd.csv'],
                       target_path = 'train-dataset/tabular/',
                       overwrite = True,
                       show_progress = True)
dataset_training = Dataset.Tabular.from_delimited_files(path = [(datastore, 'train-dataset/tabular/training_pd.csv')])
```

![Imagen del conjunto de datos cargado.](./media/apache-spark-machine-learning-aml-notebook/upload-dataset.png)

## <a name="submit-an-auto-ml-experiment"></a>Envío de un experimento de Machine Learning automatizado

#### <a name="define-training-settings"></a>Definición de la configuración del entrenamiento

1. Para enviar un experimento, es preciso defina los parámetros del mismo y la configuración de los modelos para el entrenamiento. [Aquí](https://docs.microsoft.com/azure/machine-learning/how-to-configure-auto-train) puede ver la lista completa de valores.

   ```python
   import logging

   automl_settings = {
       "iteration_timeout_minutes": 10,
       "experiment_timeout_minutes": 30,
       "enable_early_stopping": True,
       "primary_metric": 'r2_score',
       "featurization": 'auto',
       "verbosity": logging.INFO,
       "n_cross_validations": 2}
   ```

2. Ahora, se pasará la configuración de entrenamiento definida como si fuera un \*\*parámetro kwargs a un objeto AutoMLConfig. Como el entrenamiento se realiza en Spark, también hay que pasar el contexto de Spark, al que puede acceder automáticamente la variable ```sc```. Además, se especificarán los datos de entrenamiento y el tipo de modelo, que en este caso es regresión.

   ```python
   from azureml.train.automl import AutoMLConfig

   automl_config = AutoMLConfig(task='regression',
                             debug_log='automated_ml_errors.log',
                             training_data = dataset_training,
                             spark_context = sc,
                             model_explainability = False, 
                             label_column_name ="fareAmount",**automl_settings)
   ```

> [!NOTE]
> Los pasos previos al procesamiento del aprendizaje automático (normalización de características, control de los datos que faltan, conversión de valores de texto a numéricos, etc.) se convierten en parte del modelo subyacente. Cuando se utiliza el modelo para las predicciones, se aplican automáticamente a los datos de entrada los mismos pasos previos al procesamiento que se aplican durante el entrenamiento.

#### <a name="train-the-automatic-regression-model"></a>Entrenamiento del modelo de regresión automática 
Ahora, se creará un objeto de experimento en el área de trabajo de Azure Machine Learning. Un experimento actúa como un contenedor para las ejecuciones individuales. 

```python
from azureml.core.experiment import Experiment

# Start an experiment in Azure Machine Learning
experiment = Experiment(ws, "aml-synapse-regression")
tags = {"Synapse": "regression"}
local_run = experiment.submit(automl_config, show_output=True, tags = tags)

# Use the get_details function to retrieve the detailed output for the run.
run_details = local_run.get_details()
```

Cuando se haya completado el experimento, la salida devolverá detalles sobre las iteraciones completadas. Para cada iteración, verá el tipo de modelo, la duración de la ejecución y la precisión del entrenamiento. El campo BEST realiza el seguimiento de la mejor puntuación de entrenamiento activa en cada tipo de métrica.

![Captura de pantalla de la salida del modelo.](./media/apache-spark-machine-learning-aml-notebook/aml-model-output.png)

> [!NOTE]
> Tras el envío, el experimento de AutoML ejecutará varias iteraciones y tipos de modelo. Normalmente, esta ejecución tardará entre 1 y 1,5 horas. 

#### <a name="retrieve-the-best-model"></a>Recuperación del mejor modelo
Para seleccionar el mejor modelo de las iteraciones, se usará la función ```get_output```, ya que devuelve la mejor ejecución y el modelo idóneo. El código siguiente recuperará la mejor ejecución y el modelo idóneo para cualquier métrica registrada o para una iteración concreta.

```python
# Get best model
best_run, fitted_model = local_run.get_output()
```

#### <a name="test-model-accuracy"></a>Probar la precisión del modelo

1. Para probar la precisión del modelo, usaremos el mejor modelo para ejecutar predicciones de las tarifas de los taxis en el conjunto de datos de prueba. La función ```predict``` usa el mejor modelo y predice los valores de y (importe de la tarifa) a partir del conjunto de datos de validación. 

   ```python
   # Test best model accuracy
   validation_data_pd = validation_data.toPandas()
   y_test = validation_data_pd.pop("fareAmount").to_frame()
   y_predict = fitted_model.predict(validation_data_pd)
   ```

2. El error de la media cuadrática (RMSE) es una medida que usa con frecuencia e indica las diferencias entre los valores de ejemplo predichos por un modelo y los valores observados. Para calcular el error de la media cuadrática de los resultados, se compara el dataframe de y_test con los valores predichos por el modelo. 

   La función ```mean_squared_error``` toma dos matrices y calcula el promedio del error cuadrático entre ellos. Luego, se toma la raíz cuadrada del resultado. Esta métrica indica aproximadamente cuánto se alejan las predicciones de la tarifa del taxi de los valores reales.

   ```python
   from sklearn.metrics import mean_squared_error
   from math import sqrt

   # Calculate Root Mean Square Error
   y_actual = y_test.values.flatten().tolist()
   rmse = sqrt(mean_squared_error(y_actual, y_predict))

   print("Root Mean Square Error:")
   print(rmse)
   ```

   ```Output
   Root Mean Square Error:
   2.309997102577151
   ```
   
   El error de la media cuadrática es una buena medida para conocer la precisión con la que el modelo predice la respuesta. A partir de los resultados, verá que el modelo es bastante bueno en la predicción de las tarifas de taxi a partir de las características del conjunto de datos, normalmente con un margen de +/-2,00 USD.

3. Ejecute el siguiente código para calcular el error medio absoluto porcentual (MAPE). Esta métrica expresa la precisión como un porcentaje del error. Para ello, calcula una diferencia absoluta entre cada valor predicho y real, y después suma todas las diferencias. Luego, expresa esa suma en forma de porcentaje del total de los valores reales.

   ```python
   # Calculate MAPE and Model Accuracy 
   sum_actuals = sum_errors = 0

   for actual_val, predict_val in zip(y_actual, y_predict):
       abs_error = actual_val - predict_val
       if abs_error < 0:
           abs_error = abs_error * -1

       sum_errors = sum_errors + abs_error
       sum_actuals = sum_actuals + actual_val

   mean_abs_percent_error = sum_errors / sum_actuals

   print("Model MAPE:")
   print(mean_abs_percent_error)
   print()
   print("Model Accuracy:")
   print(1 - mean_abs_percent_error)
   ```

   ```Output
   Model MAPE:
   0.03655071038487368

   Model Accuracy:
   0.9634492896151263
   ```
   A partir de las dos métricas de precisión de la predicción, verá que el modelo es bastante bueno en la predicción de las tarifas de taxi a partir de las características del conjunto de datos. 

4. Después de ajustar el modelo de regresión lineal, será preciso determinar si el modelo se ajusta bien a los datos. Para ello, se trazarán los valores de las tarifas reales contra la salida predicha. Además, también calcularemos la medida de R cuadrado para saber lo cerca que están de la línea de regresión ajustada.

   ```python
   import matplotlib.pyplot as plt
   import numpy as np
   from sklearn.metrics import mean_squared_error, r2_score

   # Calculate the R2 score using the predicted and actual fare prices
   y_test_actual = y_test["fareAmount"]
   r2 = r2_score(y_test_actual, y_predict)

   # Plot the Actual vs Predicted Fare Amount Values
   plt.style.use('ggplot')
   plt.figure(figsize=(10, 7))
   plt.scatter(y_test_actual,y_predict)
   plt.plot([np.min(y_test_actual), np.max(y_test_actual)], [np.min(y_test_actual), np.max(y_test_actual)], color='lightblue')
   plt.xlabel("Actual Fare Amount")
   plt.ylabel("Predicted Fare Amount")
   plt.title("Actual vs Predicted Fare Amont R^2={}".format(r2))
   plt.show()
   ```
   
   ![Captura de pantalla del trazado de la regresión.](./media/apache-spark-machine-learning-aml-notebook/aml-fare-amount.png)

   En los resultados podemos ver que la medida de R cuadrado supone el 95 % de nuestra varianza. Esto también lo valida el trazado real, frente al observado. Cuanta más varianza tenga en cuenta el modelo de regresión, más cerca estarán los puntos de datos de la línea de regresión ajustada.  

## <a name="register-model-to-azure-machine-learning"></a>Registro del modelo en Azure Machine Learning
Una vez que se haya validado el mejor modelo, se puede registrar en Azure Machine Learning. Después de registrar el modelo, puede descargar o implementar el modelo registrado y recibir todos los archivos que registró.

```python
description = 'My AutoML Model'
model_path='outputs/model.pkl'
model = best_run.register_model(model_name = 'NYCGreenTaxiModel', model_path = model_path, description = description)
print(model.name, model.version)
```

```Output
NYCGreenTaxiModel 1
```

## <a name="view-results-in-azure-machine-learning"></a>Visualización de los resultados en Azure Machine Learning
Por último, también puede acceder a los resultados de las iteraciones desplazándose al experimento en el área de trabajo de Azure Machine Learning. Aquí podrá profundizar en los detalles adicionales sobre el estado de la ejecución, los modelos que se han probado y otras métricas del modelo. 

![Captura de pantalla del área de trabajo de Azure Machine Learning.](./media/apache-spark-machine-learning-mllib-notebook/apache-spark-aml-workspace.png)

## <a name="next-steps"></a>Pasos siguientes
- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
- [Tutorial de Apache Spark MLlib](./apache-spark-machine-learning-mllib-notebook.md)
