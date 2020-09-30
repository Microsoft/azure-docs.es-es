---
title: Implementación del modelo de ML en Azure SQL Edge mediante ONNX
description: En la tercera parte de este tutorial de tres partes de Azure SQL Edge para predecir impurezas de mineral de hierro, ejecutará los modelos de aprendizaje automático de ONNX en SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: abd1bc9b53c303d094d74683da0d6e3fad4d715f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90887474"
---
# <a name="deploy-ml-model-on-azure-sql-edge-using-onnx"></a>Implementación del modelo de ML en Azure SQL Edge mediante ONNX 

En la tercera parte de este tutorial de tres partes para predecir impurezas de mineral de hierro en Azure SQL Edge, hará lo siguiente:

1. Usará Azure Data Studio para conectarse a SQL Database en la instancia de Azure SQL Edge.
2. Predecirá las impurezas de mineral de hierro con ONNX en Azure SQL Edge.

## <a name="connect-to-the-sql-database-in-the-azure-sql-edge-instance"></a>Conexión a SQL Database en la instancia de Azure SQL Edge

1. Abra Azure Data Studio.

2. En la pestaña **Bienvenida**, inicie una nueva conexión con los detalles siguientes:

   |_Campo_|_Valor_|
   |-------|-------|
   |Tipo de conexión| Microsoft SQL Server|
   |Server|Dirección IP pública mencionada en la VM que se creó para esta demostración.|
   |Nombre de usuario|sa|
   |Contraseña|Contraseña segura que se usó al crear la instancia de Azure SQL Edge.|
   |Base de datos|Valor predeterminado|
   |Grupo del servidor|Valor predeterminado|
   |Nombre (opcional)|Puede proporcionar un nombre opcional.|

3. Haga clic en **Conectar**

4. En la sección **Archivo**, abra un cuaderno nuevo o use el método abreviado de teclado Alt + Windows + N. 

5. Establezca el kernel en Python 3.

## <a name="predict-iron-ore-impurities-with-onnx"></a>Predicción de las impurezas de mineral de hierro con ONNX

Escriba el siguiente código de Python en el cuaderno de Azure Data Studio y ejecútelo.

1. En primer lugar, instale e importe los paquetes necesarios.

   ```python
   !pip install azureml.core -q
   !pip install azureml.train.automl -q
   !pip install matplotlib -q
   !pip install pyodbc -q
   !pip install spicy -q
   
   import logging
   from matplotlib import pyplot as plt
   import numpy as np
   import pandas as pd
   import pyodbc
   
   from scipy import stats
   from scipy.stats import skew #for some statistics
   
   import azureml.core
   from azureml.core.experiment import Experiment
   from azureml.core.workspace import Workspace
   from azureml.train.automl import AutoMLConfig
   from azureml.train.automl import constants
   ```

1. Defina el área de trabajo de Azure AutoML y la configuración del experimento de AutoML para el experimento de regresión.

   ```python
   ws = Workspace(subscription_id="<Azure Subscription ID>",
                  resource_group="<resource group name>",
                  workspace_name="<ML workspace name>")
   # Choose a name for the experiment.
   experiment_name = 'silic_percent2-Forecasting-onnx'
   experiment = Experiment(ws, experiment_name)
   ```

1. Importe el conjunto de datos en un marco Panda. Para el propósito del aprendizaje del modelo, use el conjunto de datos de aprendizaje [Predicción de calidad en un proceso de minería de datos](https://www.kaggle.com/edumagalhaes/quality-prediction-in-a-mining-process) de Kaggle. Descargue el archivo de datos y guárdelo localmente en la máquina de desarrollo. Usará estos datos para predecir cuánta impureza hay en el concentrado de minerales.

   ```python
   df = pd.read_csv("<local path where you have saved the data file>",decimal=",",parse_dates=["date"],infer_datetime_format=True)
   df = df.drop(['date'],axis=1)
   df.describe()
   ```

1. Analice los datos para identificar cualquier asimetría. Durante este proceso, examine la distribución y la información de asimetría para cada una de las columnas de la trama de datos.

   ```python
   ## We can use a histogram chart to view the data distribution for the Dataset. In this example, we are looking at the histogram for the "% Silica Concentrate" 
   ## and the "% Iron Feed". From the histogram, you'll notice the data distribution is skewed for most of the features in the dataset. 
   
   f, (ax1,ax2,ax3) = plt.subplots(1,3)
   ax1.hist(df['% Iron Feed'], bins='auto')
   #ax1.title = 'Iron Feed'
   ax2.hist(df['% Silica Concentrate'], bins='auto')
   #ax2.title = 'Silica Concentrate'
   ax3.hist(df['% Silica Feed'], bins='auto')
   #ax3.title = 'Silica Feed'
   ```

1. Compruebe y corrija el nivel de asimetría en los datos.

   ```python
   ##Check data skewness with the skew or the kurtosis function in spicy.stats
   ##Skewness using the spicy.stats skew function
   for i in list(df):
           print('Skew value for column "{0}" is: {1}'.format(i,skew(df[i])))
   
   #Fix the Skew using Box Cox Transform
   from scipy.special import boxcox1p
   for i in list(df):
       if(abs(skew(df[i])) >= 0.20):
           #print('found skew in column - {0}'.format(i))
           df[i] = boxcox1p(df[i], 0.10)
           print('Skew value for column "{0}" is: {1}'.format(i,skew(df[i])))
   ```

1. Compruebe la correlación de otras características con la característica de predicción. Si la correlación no es alta, quite esas características.

   ```python
   silic_corr = df.corr()['% Silica Concentrate']
   silic_corr = abs(silic_corr).sort_values()
   drop_index= silic_corr.index[:8].tolist()
   df = df.drop(drop_index, axis=1)
   df.describe()
   ```

1. Inicie el experimento de AzureML para buscar y entrenar el mejor algoritmo. En este caso, está probando con todos los algoritmos de regresión, con una métrica principal de Error de desviación media cuadrática normalizada (NRMSE). Para obtener más información, consulte [Métrica principal de experimentos de Azure ML](https://docs.microsoft.com/azure/machine-learning/how-to-configure-auto-train#primary-metric). En el código siguiente se iniciará una ejecución local del experimento de ML.

   ```python
   ## Define the X_train and the y_train data sets for the AutoML experiments. X_Train are the inputs or the features, while y_train is the outcome or the prediction result. 
   
   y_train = df['% Silica Concentrate']
   x_train = df.iloc[:,0:-1]
   automl_config = AutoMLConfig(task = 'regression',
                                primary_metric = 'normalized_root_mean_squared_error',
                                iteration_timeout_minutes = 60,
                                iterations = 10,                        
                                X = x_train, 
                                y = y_train,
                                featurization = 'off',
                                enable_onnx_compatible_models=True)
   
   local_run = experiment.submit(automl_config, show_output = True)
   best_run, onnx_mdl = local_run.get_output(return_onnx_model=True)
   ```

1. Cargue el modelo en la base de datos de Azure SQL Edge para la puntuación local.

   ```python
   ## Load the Model into a SQL Database.
   ## Define the Connection string parameters. These connection strings will be used later also in the demo.
   server = '<SQL Server IP address>'
   username = 'sa' # SQL Server username
   password = '<SQL Server password>'
   database = 'IronOreSilicaPrediction'
   db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
   conn = pyodbc.connect(db_connection_string, autocommit=True)
   cursor = conn.cursor()
   
   # Insert the ONNX model into the models table
   query = f"insert into models ([description], [data]) values ('Silica_Percentage_Predict_Regression_NRMSE_New1',?)"
   model_bits = onnx_mdl.SerializeToString()
   insert_params  = (pyodbc.Binary(model_bits))
   cursor.execute(query, insert_params)
   conn.commit()
   cursor.close()
   conn.close()
   ```

1. Por último, use el modelo de Azure SQL Edge para realizar predicciones con el modelo formado.

   ```python
   ## Define the Connection string parameters. These connection strings will be used later also in the demo.
   server = '<SQL Server IP address>'
   username = 'sa' # SQL Server username
   password = '<SQL Server password>'
   database = 'IronOreSilicaPrediction'
   db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
   conn = pyodbc.connect(db_connection_string, autocommit=True)
   #cursor = conn.cursor()
   query = \
           f'declare @model varbinary(max) = (Select [data] from [dbo].[Models] where [id] = 1);' \
           f' with d as ( SELECT  [timestamp] ,cast([cur_Iron_Feed] as real) [__Iron_Feed] ,cast([cur_Silica_Feed]  as real) [__Silica_Feed]' \
           f',cast([cur_Starch_Flow] as real) [Starch_Flow],cast([cur_Amina_Flow] as real) [Amina_Flow]' \
           f' ,cast([cur_Ore_Pulp_pH] as real) [Ore_Pulp_pH] ,cast([cur_Flotation_Column_01_Air_Flow] as real) [Flotation_Column_01_Air_Flow]' \
           f' ,cast([cur_Flotation_Column_02_Air_Flow] as real) [Flotation_Column_02_Air_Flow]' \
           f' ,cast([cur_Flotation_Column_03_Air_Flow] as real) [Flotation_Column_03_Air_Flow]' \
           f' ,cast([cur_Flotation_Column_07_Air_Flow] as real) [Flotation_Column_07_Air_Flow]' \
           f' ,cast([cur_Flotation_Column_04_Level] as real) [Flotation_Column_04_Level]' \
           f' ,cast([cur_Flotation_Column_05_Level] as real) [Flotation_Column_05_Level]' \
           f' ,cast([cur_Flotation_Column_06_Level] as real) [Flotation_Column_06_Level]' \
           f' ,cast([cur_Flotation_Column_07_Level] as real) [Flotation_Column_07_Level]' \
           f' ,cast([cur_Iron_Concentrate] as real) [__Iron_Concentrate]' \
           f' FROM [dbo].[IronOreMeasurements1]' \
           f' where timestamp between dateadd(hour,-1,getdate()) and getdate()) ' \
           f' SELECT d.*, p.variable_out1' \
           f' FROM PREDICT(MODEL = @model, DATA = d) WITH(variable_out1 numeric(25,17)) as p;' 
     
   df_result = pd.read_sql(query,conn)
   df_result.describe()
   ```

1. Con Python, cree un gráfico del porcentaje de sílice predicho en relación con la fuente de hierro, el valor datetime y la fuente de sílice.

   ```python
   import plotly.graph_objects as go
   fig = go.Figure()
   fig.add_trace(go.Scatter(x=df_result['timestamp'],y=df_result['__Iron_Feed'],mode='lines+markers',name='Iron Feed',line=dict(color='firebrick', width=2)))
   fig.add_trace(go.Scatter(x=df_result['timestamp'],y=df_result['__Silica_Feed'],mode='lines+markers',name='Silica Feed',line=dict(color='green', width=2)))
   fig.add_trace(go.Scatter(x=df_result['timestamp'],y=df_result['variable_out1'],mode='lines+markers',name='Silica Percent',line=dict(color='royalblue', width=3)))
   fig.update_layout(height= 600, width=1500,xaxis_title='Time')
   fig.show()
   ```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el uso de modelos de ONNX en Azure SQL Edge, consulte [Aprendizaje automático e IA con ONNX en SQL Edge](onnx-overview.md).
