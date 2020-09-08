---
title: 'Tutorial: Entrenamiento del primer modelo de Azure Machine Learning en Python'
titleSuffix: Azure Machine Learning
description: En este tutorial aprenderá los patrones de diseño básicos de Azure Machine Learning y entrenará un modelo de scikit-learn sencillo basado en el conjunto de datos sobre diabetes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: sgilley
author: sdgilley
ms.date: 08/25/2020
ms.custom: devx-track-python
ms.openlocfilehash: fb380e4b71ba68daf694ab725c41be64f066805e
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854931"
---
# <a name="tutorial-train-your-first-ml-model"></a>Tutorial: Entrenamiento del primer modelo de Machine Learning

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Este tutorial es la **segunda parte de dos**. En el tutorial anterior, [creó un área de trabajo y eligió un entorno de desarrollo](tutorial-1st-experiment-sdk-setup.md). En este tutorial aprenderá los patrones de diseño básicos de Azure Machine Learning y entrenará un modelo de scikit-learn sencillo basado en el conjunto de datos sobre diabetes. Después de completar este tutorial, tendrá el conocimiento práctico del SDK que le permitirá escalar verticalmente al desarrollo de experimentos y flujos de trabajo más complejos.

En este tutorial, ha aprendido las tareas siguientes:

> [!div class="checklist"]
> * Conexión con el área de trabajo y creación de un experimento
> * Carga de datos y entrenamiento de modelos scikit-learn
> * Visualización de los resultados del entrenamiento en Studio
> * Recuperación del mejor modelo

## <a name="prerequisites"></a>Requisitos previos

El único requisito previo es ejecutar la parte primera de este tutorial, [Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md).

En esta parte del tutorial se ejecuta el código del cuaderno de Jupyter de muestra *tutorials/create-first-ml-experiment/tutorial-1st-experiment-sdk-train.ipynb* que se abrió al final de la primera parte. Este artículo le guía a través del mismo código que se encuentra en el cuaderno.

## <a name="open-the-notebook"></a>Apertura del cuaderno

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).

1. Abra **tutorial-1st-experiment-sdk-train.ipynb** en la carpeta, tal y como se muestra en la [primera parte](tutorial-1st-experiment-sdk-setup.md#open).

**No** debe crear un cuaderno *nuevo* en la interfaz de Jupyter. En el cuaderno *tutorials/create-first-ml-experiment/tutorial-1st-experiment-sdk-train.ipynb* se incluyen **todos los códigos y datos necesarios** para este tutorial.

## <a name="connect-workspace-and-create-experiment"></a>Conexión con el área de trabajo y creación de un experimento

<!-- nbstart https://raw.githubusercontent.com/Azure/MachineLearningNotebooks/master/tutorials/create-first-ml-experiment/tutorial-1st-experiment-sdk-train.ipynb -->

> [!TIP]
> Contenido de _tutorial-1st-experiment-sdk-train.ipynb_. Cambie ahora al cuaderno de Jupyter Notebook si desea leer a medida que ejecuta el código. Para ejecutar una sola celda de código en un cuaderno, haga clic en la celda y presione **Mayús + Entrar**. O bien, ejecute el cuaderno completo, para lo que debe elegir **Ejecutar todo** en la barra de herramientas superior.


Importe la clase `Workspace` y cargue la información de suscripción del archivo `config.json` mediante la función `from_config().`. Esto busca el archivo JSON en el directorio actual de forma predeterminada, pero también puede especificar un parámetro de ruta de acceso para que apunte al archivo utilizando `from_config(path="your/file/path")`. Si está ejecutando este cuaderno en un servidor de cuadernos en la nube en el área de trabajo, el archivo se encuentra automáticamente en el directorio raíz.

Si el código siguiente solicita autenticación adicional, simplemente pegue el vínculo en un explorador y escriba el token de autenticación. Además, si tiene más de un inquilino vinculado a su usuario, tendrá que agregar las siguientes líneas:
```
from azureml.core.authentication import InteractiveLoginAuthentication
interactive_auth = InteractiveLoginAuthentication(tenant_id="your-tenant-id")
Additional details on authentication can be found here: https://aka.ms/aml-notebook-auth 
```

```python
from azureml.core import Workspace
ws = Workspace.from_config()
```

Ahora cree un experimento en el área de trabajo. Un experimento es otro recurso básico de la nube que representa una colección de pruebas (ejecuciones individuales del modelo). En este tutorial usará el experimento para crear ejecuciones y realizar un seguimiento del entrenamiento del modelo en Azure Machine Learning Studio. Los parámetros incluyen la referencia del área de trabajo y un nombre de cadena para el experimento.


```python
from azureml.core import Experiment
experiment = Experiment(workspace=ws, name="diabetes-experiment")
```

## <a name="load-data-and-prepare-for-training"></a>Carga de datos y preparación para el entrenamiento

Para este tutorial se utiliza el conjunto de datos sobre diabetes, que usa características como edad, sexo e IMC para predecir la progresión de la diabetes. Cargue los datos de la clase [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/) y divídalos en conjuntos de entrenamiento y de prueba mediante `train_test_split()`. Esta función separa los datos para que el modelo tenga datos no detectados que se usarán para las pruebas después del entrenamiento.


```python
from azureml.opendatasets import Diabetes
from sklearn.model_selection import train_test_split

x_df = Diabetes.get_tabular_dataset().to_pandas_dataframe().dropna()
y_df = x_df.pop("Y")

X_train, X_test, y_train, y_test = train_test_split(x_df, y_df, test_size=0.2, random_state=66)
```

## <a name="train-a-model"></a>Entrenamiento de un modelo

El entrenamiento de un modelo scikit-learn simple se puede realizar fácilmente de forma local para el aprendizaje a pequeña escala, pero para el entrenamiento de muchas iteraciones con docenas de permutaciones de características y configuraciones de hiperparámetros diferentes, es fácil perderle la pista a los modelos que ha entrenado y a la forma en la que lo ha hecho. El modelo de diseño siguiente muestra cómo aprovechar el SDK para realizar fácilmente un seguimiento del entrenamiento en la nube.

Cree un script que entrene modelos Ridge en un bucle mediante diferentes valores alfa de hiperparámetro.


```python
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.externals import joblib
import math

alphas = [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]

for alpha in alphas:
    run = experiment.start_logging()
    run.log("alpha_value", alpha)
    
    model = Ridge(alpha=alpha)
    model.fit(X=X_train, y=y_train)
    y_pred = model.predict(X=X_test)
    rmse = math.sqrt(mean_squared_error(y_true=y_test, y_pred=y_pred))
    run.log("rmse", rmse)
    
    model_name = "model_alpha_" + str(alpha) + ".pkl"
    filename = "outputs/" + model_name
    
    joblib.dump(value=model, filename=filename)
    run.upload_file(name=model_name, path_or_stream=filename)
    run.complete()
```

En el código anterior se consigue lo siguiente:

1. Para cada valor de hiperparámetro alfa en la matriz `alphas`, se crea una nueva ejecución dentro del experimento. El valor alfa se registra para diferenciar entre cada ejecución.
1. En cada ejecución, se crea una instancia de un modelo Ridge, se entrena y se usa para ejecutar las predicciones. El error cuadrático medio se calcula para los valores reales frente a los previstos y, a continuación, se registra en la ejecución. En este momento, la ejecución tiene metadatos adjuntos para el valor alfa y la precisión del error cuadrático medio (RMSE).
1. Después, el modelo de cada ejecución se serializa y se carga en la ejecución. Esto le permite descargar el archivo de modelo desde la ejecución en Studio.
1. Al final de cada iteración, la ejecución se completa llamando a `run.complete()`.

Una vez completado el entrenamiento, llame a la variable `experiment` para capturar un vínculo al experimento en Studio.

```python
experiment
```

<table style="width:100%"><tr><th>Nombre</th><th>Área de trabajo</th><th>Página del informe</th><th>Página de documentos</th></tr><tr><td>diabetes-experiment</td><td>Nombre-área-de-trabajo</td><td>Vínculo a Azure Machine Learning Studio</td><td>Vínculo a la documentación</td></tr></table>

## <a name="view-training-results-in-studio"></a>Visualización de los resultados del entrenamiento en Estudio

El **vínculo de Azure Machine Learning Studio** le llevará a la página principal del experimento. Aquí verá todas las ejecuciones individuales del experimento. Los valores de registro personalizados (`alpha_value` y `rmse`, en este caso) se convierten en campos para cada ejecución y también están disponibles para los gráficos. Para trazar un nuevo gráfico con una métrica registrada, haga clic en "Agregar gráfico" y seleccione la métrica que desea trazar.

Al entrenar modelos a escala sobre cientos y miles de ejecuciones independientes, esta página facilita la consulta de todos los modelos que haya entrenado, la forma específica en la que se entrenó cada uno y cómo han cambiado las métricas únicas a lo largo del tiempo.

:::image type="content" source="./media/tutorial-1st-experiment-sdk-train/experiment-main.png" alt-text="Página principal del experimento en Studio.":::


Seleccione un vínculo de número de ejecución en la columna `RUN NUMBER` para ver la página de una ejecución individual. En la pestaña predeterminada **Detalles** se muestra información más detallada sobre cada ejecución. Vaya a la pestaña **Salidas + registros** y verá el archivo `.pkl` del modelo que se cargó en la ejecución durante cada iteración de entrenamiento. Aquí puede descargar el archivo de modelo, en lugar de tener que volver a entrenarlo manualmente.

:::image type="content" source="./media/tutorial-1st-experiment-sdk-train/model-download.png" alt-text="Página de detalles de la ejecución en Studio.":::

## <a name="get-the-best-model"></a>Obtención del mejor modelo

Además de poder descargar los archivos de modelo del experimento en Studio, también puede descargarlos mediante programación. El siguiente código recorre en iteración cada ejecución del experimento y obtiene acceso a las métricas de ejecución registradas y a los detalles de la ejecución (que contiene el run_id). Esto realiza un seguimiento de la mejor ejecución, en este caso, la ejecución con el error cuadrático medio más bajo.

```python
minimum_rmse_runid = None
minimum_rmse = None

for run in experiment.get_runs():
    run_metrics = run.get_metrics()
    run_details = run.get_details()
    # each logged metric becomes a key in this returned dict
    run_rmse = run_metrics["rmse"]
    run_id = run_details["runId"]
    
    if minimum_rmse is None:
        minimum_rmse = run_rmse
        minimum_rmse_runid = run_id
    else:
        if run_rmse < minimum_rmse:
            minimum_rmse = run_rmse
            minimum_rmse_runid = run_id

print("Best run_id: " + minimum_rmse_runid)
print("Best run_id rmse: " + str(minimum_rmse))    
```
```output
Best run_id: 864f5ce7-6729-405d-b457-83250da99c80
Best run_id rmse: 57.234760283951765
```

Use el mejor identificador de ejecución para capturar la ejecución individual mediante el constructor `Run` junto con el objeto de experimento. A continuación, llame a `get_file_names()` para ver todos los archivos disponibles para su descarga desde esta ejecución. En este caso, solo cargó un archivo para cada ejecución durante el entrenamiento.


```python
from azureml.core import Run
best_run = Run(experiment=experiment, run_id=minimum_rmse_runid)
print(best_run.get_file_names())
```

```output
['model_alpha_0.1.pkl']
```

Llame a `download()` en el objeto de ejecución, especificando el nombre del archivo de modelo que se va a descargar. De forma predeterminada, esta función se descarga en el directorio actual.


```python
best_run.download_file(name="model_alpha_0.1.pkl")
```
<!-- nbend -->

## <a name="clean-up-resources"></a>Limpieza de recursos

No complete esta sección si planea ejecutar otros tutoriales de Azure Machine Learning.

### <a name="stop-the-compute-instance"></a>Detención de la instancia de proceso

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]

### <a name="delete-everything"></a>Eliminar todo el contenido

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha realizado las tareas siguientes:

> [!div class="checklist"]
> * Ha conectado con el área de trabajo y creado un experimento
> * Ha cargado datos y entrenado modelos scikit-learn
> * Ha visualizado los resultados del entrenamiento en Studio y recuperado los modelos

[Implemente el modelo](tutorial-deploy-models-with-aml.md) con Azure Machine Learning.
Aprenda a desarrollar experimentos de [aprendizaje automático automatizados](tutorial-auto-train-models.md).
