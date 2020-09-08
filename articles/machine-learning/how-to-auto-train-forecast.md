---
title: Entrenamiento automático de un modelo de previsión de series temporales
titleSuffix: Azure Machine Learning
description: Aprenda a usar Azure Machine Learning para entrenar un modelo de regresión de previsión de series temporales mediante aprendizaje automático automatizado.
services: machine-learning
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperfq1
ms.date: 08/20/2020
ms.openlocfilehash: 900e36ec3e508f9d3616cf0c0d19ea4ff067f775
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89144794"
---
# <a name="auto-train-a-time-series-forecast-model"></a>Entrenamiento automático de un modelo de previsión de series temporales
[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a configurar y entrenar un modelo de regresión de previsión de series temporales con aprendizaje automático automatizado, AutoML, en el [SDK de Python de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/?view=azure-ml-py). 

Para obtener una experiencia con poco código, vea el [tutorial de previsión de la demanda con el automatizado de aprendizaje automático](tutorial-automated-ml-forecast.md) para obtener un ejemplo de previsión de serie temporal que usa el aprendizaje automático automatizado en [Azure Machine Learning Studio](https://ml.azure.com/).

A diferencia de los métodos de series temporales clásicos, en el aprendizaje automático automatizado, los valores de series temporales anteriores se "dinamizan" para convertirse en dimensiones adicionales del regresor, junto con otros indicadores. Este enfoque incorpora varias variables contextuales y su relación entre sí durante el entrenamiento. Dado que varios factores pueden influir en una previsión, este método se adapta bien a los escenarios de previsión reales. Por ejemplo, al prever ventas, las interacciones de las tendencias históricas, la tasa de cambio y el precio son motores conjuntos del resultado de ventas. 

En los siguientes ejemplos se indica cómo:

* Preparar los datos para el modelado de series temporales
* Configurar parámetros específicos de las serie temporales en un objeto [`AutoMLConfig`](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig)
* Ejecutar predicciones con los datos de serie temporal

## <a name="prerequisites"></a>Requisitos previos

Para realizar este artículo, necesitará lo siguiente 

* Un área de trabajo de Azure Machine Learning. Para crear el área de trabajo, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* En este artículo se presupone una familiarización básica con la configuración de una experimento de aprendizaje de automático automatizado. Siga el [tutorial](tutorial-auto-train-models.md) o los [procedimientos](how-to-configure-auto-train.md) para ver los patrones de diseño del experimento de aprendizaje automático automatizado.

## <a name="preparing-data"></a>Preparación de los datos

La diferencia más importante entre el tipo de tarea de regresión de previsión y el de regresión en AutoML es que se incluye una característica en los datos que representa una serie de tiempo válida. Una serie de tiempo normal tiene una frecuencia coherente y bien definida, y un valor en cada punto de un intervalo de tiempo continuo. 

Tenga en cuenta la siguiente instantánea de un archivo `sample.csv`.
Este es un conjunto de datos de ventas diarios para una empresa que tiene dos almacenes diferentes, A y B. 

Además, hay características para

 *  `week_of_year`: permite que el modelo detecte la estacionalidad semanal.
* `day_datetime`: representa una serie temporal limpia con una frecuencia diaria.
* `sales_quantity`: columna de destino para la ejecución de predicciones. 

```output
day_datetime,store,sales_quantity,week_of_year
9/3/2018,A,2000,36
9/3/2018,B,600,36
9/4/2018,A,2300,36
9/4/2018,B,550,36
9/5/2018,A,2100,36
9/5/2018,B,650,36
9/6/2018,A,2400,36
9/6/2018,B,700,36
9/7/2018,A,2450,36
9/7/2018,B,650,36
```


Lea los datos en una trama de datos de Pandas y use la función `to_datetime` para asegurarse de que la serie temporal sea de tipo `datetime`.

```python
import pandas as pd
data = pd.read_csv("sample.csv")
data["day_datetime"] = pd.to_datetime(data["day_datetime"])
```

En este caso los datos ya están ordenados de manera ascendente por el campo de hora `day_datetime`. Sin embargo, al configurar un experimento, asegúrese de que la columna de tiempo deseada esté ordenada de manera ascendente para compilar una serie de tiempo válida. 

El código siguiente: 
* Asume que los datos contienen 1000 registros; y realiza una división determinista en los datos para crear conjuntos de datos de entrenamiento y prueba. 
* Identifica la columna de etiqueta como `sales_quantity`.
* Separa el campo de etiqueta de `test_data` para formar el conjunto `test_target`.

```python
train_data = data.iloc[:950]
test_data = data.iloc[-50:]

label =  "sales_quantity"
 
test_labels = test_data.pop(label).values
```

> [!IMPORTANT]
> Al entrenar un modelo para la previsión de valores futuros, asegúrese de que todas las características del entrenamiento se pueden usar al ejecutar predicciones para su horizonte previsto. Por ejemplo, al crear una previsión de demanda, incluir una característica para el precio de cotización actual podría aumentar la precisión del entrenamiento de manera exponencial. Sin embargo, si quiere una previsión con un horizonte lejano, no es posible predecir con precisión valores de cotización futuros correspondientes a momentos futuros en la serie temporal y la precisión del modelo podría verse afectada.

<a name="config"></a>

## <a name="training-and-validation-data"></a>Datos de entrenamiento y validación

Puede especificar conjuntos distintos de entrenamiento y validación directamente en el objeto `AutoMLConfig`.   Obtenga más información sobre [AutoMLConfig](#configure-experiment).

En el caso de la previsión de series temporales, se usa automáticamente la **Validación cruzada de origen variable (ROCV)** cuando se pasan los datos de entrenamiento y validación juntos, y se establece el número de pliegues de validación cruzada con el parámetro `n_cross_validations` en `AutoMLConfig`. ROCV divide la serie en datos de entrenamiento y validación con un punto temporal de origen. Al deslizar el origen en el tiempo, se generan subconjuntos de validación cruzada. Esta estrategia conserva la integridad de los datos de la serie temporal y elimina el riesgo de perder datos.

![texto alternativo](./media/how-to-auto-train-forecast/ROCV.svg)

Para otras opciones de validación cruzada y de división de datos, consulte [Configuración de las divisiones de datos y la validación cruzada en AutoML](how-to-configure-cross-validation-data-splits.md).


```python
automl_config = AutoMLConfig(task='forecasting',
                             n_cross_validations=3,
                             ...
                             **time_series_settings)
```

Obtenga más información sobre cómo AutoML aplica la validación cruzada para [evitar un sobreajuste de los modelos](concept-manage-ml-pitfalls.md#prevent-over-fitting).

## <a name="configure-experiment"></a>Configuración del experimento
El objeto [`AutoMLConfig`](https://docs.microsoft.com/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?view=azure-ml-py) define la configuración y los datos necesarios para una tarea de aprendizaje automático automatizado. La configuración de un modelo de previsión es similar a la configuración de un modelo de regresión estándar, pero existen determinados pasos y opciones de configuración de características que son específicos para los datos de series temporales. 

### <a name="featurization-steps"></a>Pasos de caracterización

En todos los experimentos de aprendizaje automático automatizado se aplican técnicas de escalado automático y normalización a los datos de forma predeterminada. Estas técnicas son tipos de **caracterización** que ayudan a *ciertos* algoritmos que son sensibles a las características a diferentes escalas. Más información sobre los pasos de caracterización predeterminados en [caracterización en AutoML](how-to-configure-auto-features.md#automatic-featurization)

Sin embargo, los siguientes pasos solo se realizan para los tipos de tarea `forecasting`:

* Detectar la frecuencia de muestreo de la serie temporal (por ejemplo, cada hora, cada día, cada semana, etc.) y crear nuevos registros para los momentos ausentes para que la serie sea continua.
* Imputar los valores que faltan en las columnas de destino (copia de los valores nulos hacia adelante) y de característica (con la mediana de los valores de la columna).
* Crear características basadas en los identificadores de serie temporal para habilitar los efectos fijos en las diferentes series.
* Crear características basadas en el tiempo para ayudar a aprender los patrones estacionales.
* Codificar las variables categóricas en cantidades numéricas.

Para obtener un resumen de las características que se crean como resultado de estos pasos, consulte [Transparencia de caracterización](how-to-configure-auto-features.md#featurization-transparency).

> [!NOTE]
> Los pasos de la caracterización del aprendizaje automático automatizado (normalización de características, control de los datos que faltan, conversión de valores de texto a numéricos, etc.) se convierten en parte del modelo subyacente. Cuando se usa el modelo para realizar predicciones, se aplican automáticamente a los datos de entrada los mismos pasos de caracterización que se aplican durante el entrenamiento.

#### <a name="customize-featurization"></a>Personalización de la caracterización

También tiene la opción de personalizar los valores de la caracterización para asegurarse de que los datos y las características que se usan para entrenar el modelo de Machine Learning generan predicciones pertinentes. 

Las personalizaciones admitidas para tareas `forecasting` incluyen:

|Personalización|Definición|
|--|--|
|**Actualización del propósito de la columna**|Invalida el tipo de característica detectado automáticamente para la columna especificada.|
|**Actualización de parámetros del transformador** |Actualizar los parámetros para el transformador especificado. Actualmente admite *Imputer* (fill_value y median).|
|**Quitar columnas** |Especifica las columnas que se van a eliminar de la caracterización.|

Para personalizar las caracterizaciones con el SDK, especifique `"featurization": FeaturizationConfig` en el objeto `AutoMLConfig`. Más información sobre [caracterizaciones personalizadas](how-to-configure-auto-features.md#customize-featurization).

```python
featurization_config = FeaturizationConfig()
# `logQuantity` is a leaky feature, so we remove it.
featurization_config.drop_columns = ['logQuantitity']
# Force the CPWVOL5 feature to be of numeric type.
featurization_config.add_column_purpose('CPWVOL5', 'Numeric')
# Fill missing values in the target column, Quantity, with zeroes.
featurization_config.add_transformer_params('Imputer', ['Quantity'], {"strategy": "constant", "fill_value": 0})
# Fill mising values in the `INCOME` column with median value.
featurization_config.add_transformer_params('Imputer', ['INCOME'], {"strategy": "median"})
```

Si usa Azure Machine Learning Studio para el experimento, vea el [artículo de procedimiento](how-to-use-automated-ml-for-ml-models.md#customize-featurization).

### <a name="configuration-settings"></a>Parámetros de configuración

Al igual que en un problema de regresión, debe definir los parámetros de entrenamiento estándar como tipo de tarea, número de iteraciones, datos de entrenamiento y número de validaciones cruzadas. Para las tareas de previsión existen parámetros adicionales que se deben establecer y que afectan al experimento. 

En la tabla siguiente se resumen estos parámetros adicionales. Consulte la [documentación de referencia](https://docs.microsoft.com/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?view=azure-ml-py) para más información sobre los patrones de diseño de sintaxis.

| Nombre del&nbsp;parámetro | Descripción | Obligatorio |
|-------|-------|-------|
|`time_column_name`|Se utiliza para especificar la columna de fecha y hora en los datos de entrada utilizada que se usa para compilar la serie temporal y se deduce su frecuencia.|✓|
|`forecast_horizon`|Define el número de períodos futuros que le gustaría pronosticar. El horizonte está en las unidades de la frecuencia de la serie temporal. Las unidades se basan en el intervalo de tiempo de los datos de entrenamiento (por ejemplo, semanales, mensuales) que debe predecir el pronosticador.|✓|
|`enable_dnn`|[Habilitar las DNN de previsión]().||
|`time_series_id_column_names`|Nombres de columna que se usan para identificar de forma única la serie temporal en los datos que tienen varias filas con la misma marca de tiempo. Si no se definen los identificadores de serie temporal, el conjunto de datos se presupone una serie temporal. Para más información sobre las series temporales únicas, consulte [energy_demand_notebook](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand).||
|`target_lags`|Número de filas para retrasar los valores de destino en función de la frecuencia de los datos. El retraso se representa como una lista o un entero único. El retraso se debe usar cuando la relación entre las variables independientes y la variable dependiente no coincide o está en correlación de forma predeterminada. ||
|`feature_lags`| El ML automatizado decidirá automáticamente las características que se van a retardar cuando se establezcan `target_lags` y `feature_lags` se establezca en `auto`. Habilitar los retardos de características puede ayudarle a mejorar la precisión. Los retardos de características están deshabilitados de forma predeterminada. ||
|`target_rolling_window_size`|*n* períodos históricos que se utilizarán para generar valores previstos, < = tamaño del conjunto de entrenamiento. Si se omite, *n* es el tamaño total del conjunto de entrenamiento. Especifique este parámetro si solo desea tener en cuenta una determinada cantidad de historial al entrenar el modelo. Más información sobre [agregación de ventanas con desplazamiento de objetivo](#target-rolling-window-aggregation).||


El código siguiente: 
* Crea `time-series settings` como un objeto de diccionario. 
* Establece `time_column_name` en el campo `day_datetime` en el conjunto de datos. 
* Define el parámetro `time_series_id_column_names` en `"store"`. Esto garantiza que se creen **dos grupos de series temporales independientes** para los datos, uno para los almacenes A y B.
* Establece el `forecast_horizon` en 50 para predecir el conjunto de pruebas completo. 
* Establece una ventana de previsión en 10 períodos con `target_rolling_window_size`
* Especifica un retardo único en los valores de destino para dos períodos por delante con el parámetro `target_lags`. 
* Establece `target_lags` en el valor "auto" recomendado, que detectará automáticamente este valor.

```python
time_series_settings = {
    "time_column_name": "day_datetime",
    "time_series_id_column_names": ["store"],
    "forecast_horizon": 50,
    "target_lags": "auto",
    "target_rolling_window_size": 10,
}
```

Estos `time_series_settings` se pasan en el objeto de `AutoMLConfig` estándar junto con el tipo de tarea `forecasting`, la métrica principal, los criterios de salida y los datos de entrenamiento. 

```python
from azureml.core.workspace import Workspace
from azureml.core.experiment import Experiment
from azureml.train.automl import AutoMLConfig
import logging

automl_config = AutoMLConfig(task='forecasting',
                             primary_metric='normalized_root_mean_squared_error',
                             experiment_timeout_minutes=15,
                             enable_early_stopping=True,
                             training_data=train_data,
                             label_column_name=label,
                             n_cross_validations=5,
                             enable_ensembling=False,
                             verbosity=logging.INFO,
                             **time_series_settings)
```

## <a name="optional-configurations"></a>Configuraciones opcionales

Hay configuraciones opcionales adicionales disponibles para las tareas de previsión, como la habilitación del aprendizaje profundo y la especificación de una agregación de ventana con desplazamiento de objetivo. 

### <a name="enable-deep-learning"></a>Habilitar el aprendizaje profundo.

> [!NOTE]
> La compatibilidad con DNN para la previsión en el aprendizaje automático automatizado se encuentra en **versión preliminar** y no es compatible con las ejecuciones locales.

También puede aprovechar el aprendizaje profundo con redes neuronal profundas, DNN, para mejorar las puntuaciones del modelo. El aprendizaje profundo de ML automatizado permite pronosticar datos de series temporales de variable única y de varias variables.

Los modelos de aprendizaje profundo tienen tres capacidades intrínsecas:
1. Pueden aprender de las asignaciones arbitrarias de las entradas a las salidas.
1. Admiten varias entradas y salidas.
1. Pueden extraer automáticamente patrones en datos de entrada que abarquen secuencias largas. 

Para habilitar el aprendizaje profundo, establezca el `enable_dnn=True` en el objeto `AutoMLConfig`.

```python
automl_config = AutoMLConfig(task='forecasting',
                             enable_dnn=True,
                             ...
                             **time_series_settings)
```
Para habilitar DNN para un experimento de AutoML creado en Azure Machine Learning Studio, consulte los [procedimientos para la configuración del tipo de tarea](how-to-use-automated-ml-for-ml-models.md#create-and-run-experiment).

ML automatizado proporciona a los usuarios modelos nativos de serie temporal y de aprendizaje profundo como parte del sistema de recomendaciones. 

Modelos| Descripción | Ventajas
----|----|---
Prophet (versión preliminar)|Prophet funciona mejor con series temporales que tienen efectos estacionales fuertes y varias estaciones de datos históricos. Para aprovechar este modelo, instálelo localmente mediante `pip install fbprophet`. | Es preciso y rápido, sólido para valores atípicos, datos que faltan y cambios drásticos de la serie temporal.
Auto-ARIMA (versión preliminar)|La media móvil integrada autorregresiva (ARIMA) funciona mejor cuando los datos son estacionales. Esto significa que sus propiedades estadísticas, como la media y la varianza, son constantes en todo el conjunto. Por ejemplo, si lanza una moneda al aire, la probabilidad de obtener cara es del 50 %, independientemente de si la lanza hoy, mañana o el año siguiente.| Excelente para las series univariables, ya que los valores anteriores se usan para predecir los futuros.
ForecastTCN (versión preliminar)| ForecastTCN es un modelo de red neuronal diseñado para abordar las tareas de previsión más exigentes, capturando tendencias locales y globales no lineales en los datos, así como las relaciones entre series temporales.|Capaz de aprovechar tendencias complejas en los datos y escalar fácilmente a los mayores conjuntos de datos.


Consulte en el [cuaderno de previsión de producción de bebidas](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-beer-remote/auto-ml-forecasting-beer-remote.ipynb) un ejemplo de código detallado que aprovecha las DNN.


### <a name="target-rolling-window-aggregation"></a>Agregación de ventanas con desplazamiento de objetivo
A menudo, la mejor información que puede tener una previsión es el valor reciente del objetivo.  Las agregaciones de ventanas con desplazamiento permiten la incorporación de una agregación gradual de valores de datos como características. La generación y el uso de estas características adicionales como datos contextuales extra contribuye a la precisión del modelo de entrenamiento.

Por ejemplo, suponga que desea predecir la demanda de energía. Puede que desee agregar una característica de ventana con desplazamiento de tres días para tener en cuenta los cambios térmicos de los espacios calentados. En este ejemplo, cree esta ventana estableciendo `target_rolling_window_size= 3` en el constructor `AutoMLConfig`. 

En la tabla se muestra el diseño de características que se produce cuando se aplica la agregación de ventanas. Las columnas para los valores **mínimo, máximo** y **suma** se generan en una ventana deslizante de tres según la configuración definida. Cada fila tiene una nueva característica calculada, en el caso de la marca de tiempo del 8 de septiembre de 2017 4:00 a. m., los valores máximo, mínimo y suma se calculan con los **valores de la demanda** del 8 de septiembre de 2017, de la 1:00 a. m. a las 3:00 a. m. Esta ventana tiene tres turnos juntos para rellenar los datos de las filas restantes.

![texto alternativo](./media/how-to-auto-train-forecast/target-roll.svg)


Vea un ejemplo de código de Python que aprovecha las [características para agregar ventanas con desplazamiento de objetivo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb).

## <a name="run-the-experiment"></a>Ejecución del experimento 

Cuando tenga el objeto `AutoMLConfig` listo, puede enviar el experimento. Una vez finalizado el modelo, recupere la iteración con la mejor ejecución.

```python
ws = Workspace.from_config()
experiment = Experiment(ws, "forecasting_example")
local_run = experiment.submit(automl_config, show_output=True)
best_run, fitted_model = local_run.get_output()
```

## <a name="forecasting-with-best-model"></a>Previsión con el mejor modelo

Use la mejor iteración del modelo para la previsión de valores para el conjunto de datos de prueba.

La función `forecast()` permite especificaciones de cuándo se deben iniciar las predicciones, a diferencia de `predict()`, que se usa normalmente para las tareas de clasificación y regresión.

En el siguiente ejemplo, primero se reemplazan todos los valores de `y_pred` con `NaN`. En este caso, el origen de la previsión estará al final de los datos de entrenamiento. Sin embargo, si reemplazó solo la segunda mitad de `y_pred` con `NaN`, la función dejó intactos los valores numéricos de la primera, pero realizó la previsión de los valores de `NaN` de la segunda mitad. La función devuelve tanto los valores previstos como las características alineadas.

También puede usar el parámetro `forecast_destination` de la función `forecast()` para la previsión de valores hasta una fecha especificada.

```python
label_query = test_labels.copy().astype(np.float)
label_query.fill(np.nan)
label_fcst, data_trans = fitted_pipeline.forecast(
    test_data, label_query, forecast_destination=pd.Timestamp(2019, 1, 8))
```

Calcule la raíz del error cuadrático medio (RMSE) entre los valores de `actual_labels` reales y los previstos en `predict_labels`.

```python
from sklearn.metrics import mean_squared_error
from math import sqrt

rmse = sqrt(mean_squared_error(actual_labels, predict_labels))
rmse
```

Ahora que se ha determinado la precisión del modelo global, el siguiente paso más realista es usar este para prever valores futuros desconocidos. 

Proporcione un conjunto de datos en el mismo formato que el de prueba (`test_data`) pero con fechas y horas futuras y el conjunto de predicción resultante son los valores previstos para cada paso de la serie temporal. Supongamos que los últimos registros de la serie temporal en el conjunto de datos fueron del 31/12/2018. Para prever la demanda para el día siguiente (o para los períodos que necesite previsión, <= `forecast_horizon`), cree un único registro de serie de tiempo para cada almacén para el 01/01/2019.

```output
day_datetime,store,week_of_year
01/01/2019,A,1
01/01/2019,A,1
```

Repita los pasos necesarios para cargar estos datos futuros a una trama de datos y ejecute `best_run.predict(test_data)` para predecir los valores futuros.

> [!NOTE]
> No se pueden predecir valores para períodos que superen el valor de `forecast_horizon`. El modelo debe volver a entrenarse con un horizonte mayor para predecir valores futuros más allá del actual.


## <a name="example-notebooks"></a>Cuadernos de ejemplo
Consulte los [cuadernos de ejemplo de previsión](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning) para ejemplos de código detallados de la configuración de predicciones avanzada, que incluye:

* [detección y caracterización de festividades](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/auto-ml-forecasting-bike-share.ipynb)
* [validación cruzada de origen variable](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb)
* [retrasos configurables](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/auto-ml-forecasting-bike-share.ipynb)
* [características de agregado en períodos acumulados](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb)
* [DNN](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-beer-remote/auto-ml-forecasting-beer-remote.ipynb)

## <a name="next-steps"></a>Pasos siguientes

* Siga el [tutorial](tutorial-auto-train-models.md) para aprender a crear experimentos con aprendizaje automático automatizado.
* Consulte la documentación de referencia del [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).
