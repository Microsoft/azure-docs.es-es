---
title: Seguimiento de experimentos e implementación de modelos
titleSuffix: Azure Data Science Virtual Machine
description: Obtenga información sobre cómo realizar el seguimiento y el registro de experimentos desde Data Science Virtual Machine con Azure Machine Learning o MLFlow.
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: samkemp
ms.author: samkemp
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 205aed1811c3d9d21a10be7bc4f01c73eb7295b7
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89254807"
---
# <a name="track-experiments-and-deploy-models-in-azure-machine-learning"></a>Seguimiento de experimentos e implementación de modelos en Azure Machine Learning

Realice el seguimiento de sus experimentos y supervise las métricas de ejecución para mejorar el proceso de creación del modelo. En este artículo, aprenderá a agregar código de registro a un script de entrenamiento con [MLflow](https://mlflow.org/) API y a realizar un seguimiento del experimento en Azure Machine Learning.

En el siguiente diagrama se ilustra que con Seguimiento de MLflow, se realiza un seguimiento de las métricas de ejecución de un experimento y se almacenan los artefactos del modelo en el área de trabajo de Azure Machine Learning.

![Seguimiento de experimentos](./media/how-to-track-experiments/mlflow-diagram-track.png)

## <a name="prerequisites"></a>Requisitos previos

* Necesitará [aprovisionar un área de trabajo de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/how-to-manage-workspace#create-a-workspace).

## <a name="create-a-new-notebook"></a>Creación un nuevo cuaderno

Los SDK de Azure Machine Learning y de MLFlow están preinstalados en Data Science VM y se puede acceder a ellos en el entorno de Conda **azureml_py36_\*** . En Jupyterlab, haga clic en el selector y seleccione el siguiente kernel:

![selección de kernel](./media/how-to-track-experiments/experiment-tracking-1.png)

## <a name="set-up-the-workspace"></a>Configuración del área de trabajo

Vaya a [Azure Portal](https://portal.azure.com) y seleccione el área de trabajo que ha aprovisionado como parte de los requisitos previos. Verá __Descargar config.json__ (consulte a continuación): descargue la configuración y asegúrese de que está almacenada en el directorio de trabajo en DSVM.

![Obtener archivo de configuración](./media/how-to-track-experiments/experiment-tracking-2.png)

La configuración contiene información, como el nombre del área de trabajo, la suscripción, etc., y significa que no es necesario codificar estos parámetros de forma rígida.

## <a name="track-dsvm-runs"></a>Seguimiento de ejecuciones de DSVM

Agregue el código siguiente al cuaderno o script para establecer el objeto del área de trabajo de AzureML.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

>[!NOTE]
El URI de seguimiento es válido hasta una hora como máximo. Si reinicia el script después de un tiempo de inactividad, use la API get_mlflow_tracking_uri para obtener un nuevo URI.

### <a name="load-the-data"></a>Carga de los datos

En este ejemplo se usa el conjunto de datos de Diabetes, un conocido conjunto de datos pequeño que viene con Scikit-learn. Esta celda carga el conjunto de datos y lo divide en conjuntos de pruebas y entrenamiento aleatorio.

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
from sklearn.externals import joblib

X, y = load_diabetes(return_X_y = True)
columns = ['age', 'gender', 'bmi', 'bp', 's1', 's2', 's3', 's4', 's5', 's6']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)
data = {
    "train":{"X": X_train, "y": y_train},
    "test":{"X": X_test, "y": y_test}
}

print ("Data contains", len(data['train']['X']), "training samples and",len(data['test']['X']), "test samples")
```

### <a name="add-tracking"></a>Adición de seguimiento

Agregue seguimiento del experimento mediante el SDK de Azure Machine Learning y cargue un modelo guardado en el registro de ejecución del experimento. El código siguiente agrega registros y carga un archivo de modelo en la ejecución del experimento. El modelo también se registra en el registro de modelos de Azure Machine Learning.

```python
# Get an experiment object from Azure Machine Learning
from azureml.mlflow import register_model
experiment_name = 'experiment_with_mlflow'
mlflow.set_experiment(experiment_name)

with mlflow.start_run():
    # Log the algorithm parameter alpha to the run
    mlflow.log_param('alpha', 0.03)

    # Create, fit, and test the scikit-learn Ridge regression model
    regression_model = Ridge(alpha=0.03)
    regression_model.fit(data['train']['X'], data['train']['y'])
    preds = regression_model.predict(data['test']['X'])

    # Output the Mean Squared Error to the notebook and to the run
    print('Mean Squared Error is', mean_squared_error(data['test']['y'], preds))
    mlflow.log_metric('mse', mean_squared_error(data['test']['y'], preds))

    # Save the model
    model_file_name = 'model.pkl'
    joblib.dump(value = regression_model, filename = model_file_name)

    # upload the model file explicitly into artifacts
    mlflow.log_artifact(model_file_name)
    # register the model
    register_model(mlflow.active_run(), 'diabetes_model', 'model.pkl', model_framework="ScikitLearn")
```

### <a name="view-runs-in-azure-machine-learning"></a>Visualización de las ejecuciones en Azure Machine Learning

Puede ver la ejecución del experimento en [Azure Machine Learning Studio](https://ml.azure.com) Haga clic en __Experimentos__ en el menú de la izquierda y seleccione "experiment_with_mlflow", o bien, si decidió asignar un nombre diferente al experimento en el fragmento de código anterior, haga clic en el nombre utilizado:

![seleccionar experimento](./media/how-to-track-experiments/mlflow-experiments.png)

Debería ver el error cuadrático medio (MSE) registrado:

![MSE](./media/how-to-track-experiments/mlflow-experiments-2.png)

Si hace clic en la ejecución, verá otros detalles y también el modelo tratado en __Outputs+logs__ (Salidas y registros).

## <a name="deploy-model-in-azure-machine-learning"></a>Implementación de modelos en Azure Machine Learning

En esta sección, se describe cómo implementar en Azure Machine Learning los modelos entrenados en DSVM.

### <a name="step-1-create-inference-compute"></a>Paso 1: Crear procesos de inferencia

En el menú de la izquierda de [AzureML Studio](https://ml.azure.com), haga clic en __Proceso__ y, a continuación, en la pestaña __Clústeres de inferencia__. A continuación, haga clic en __+ Nuevo__ como se describe a continuación:

![Crear procesos de inferencia](./media/how-to-track-experiments/mlflow-experiments-6.png)

En el panel __Nuevo clúster de inferencia__, rellene los detalles siguientes:

* Nombre del proceso.
* Kubernetes Service: seleccione Crear.
* Seleccione la región.
* Seleccione el tamaño de la máquina virtual (para los fines de este tutorial, el valor predeterminado Standard_D3_v2 es suficiente).
* Propósito del clúster: seleccione __Desarrollo/pruebas__.
* El número de nodos debe ser igual a __1__.
* Configuración de red: Básico.

A continuación, haga clic en __Crear__.

![Detalles del proceso](./media/how-to-track-experiments/mlflow-experiments-7.png)

### <a name="step-2-deploy-no-code-inference-service"></a>Paso 2: Implementar el servicio de inferencia sin código

Cuando se registra el modelo en el código mediante `register_model`, se especifica el marco como sklearn. Azure Machine Learning admite las implementaciones sin código para los siguientes marcos:

* scikit-learn
* Formato SavedModel de TensorFlow
* Formato de modelo ONNX

La implementación sin código significa que puede implementar directamente desde el artefacto del modelo sin necesidad de especificar ningún script de puntuación específico.

Para implementar el modelo de diabetes, vaya al menú de la izquierda en [Azure Machine Learning Studio](https://ml.azure.com) y seleccione __Modelos__. A continuación, haga clic en el modelo diabetes_model registrado:

![Seleccionar modelo](./media/how-to-track-experiments/mlflow-experiments-3.png)

A continuación, haga clic en el botón __Implementar__ en el panel Detalles del modelo:

![Implementación](./media/how-to-track-experiments/mlflow-experiments-4.png)

Implementaremos el modelo en el clúster de inferencia (Azure Kubernetes Service) que creamos en el paso 1. Rellene los detalles siguientes proporcionando un nombre para el servicio y el nombre del clúster de proceso de AKS (creado en el paso 1). También se recomienda aumentar la __capacidad de reserva de CPU__ a 1 (a partir de 0,1) y la __capacidad de reserva de memoria__ a 1 (a partir de 0,5); para ello, haga clic en __Avanzado__ y rellene los detalles. A continuación, haga clic en __Implementar__.

![detalles de la implementación](./media/how-to-track-experiments/mlflow-experiments-5.png)

### <a name="step-3-consume"></a>Paso 3: Consumo

Una vez implementado el modelo correctamente, debería ver lo siguiente (para obtener acceso a esta página, haga clic en Puntos de conexión en el menú de la izquierda y, a continuación, haga clic en el nombre del servicio implementado):

![Consumir modelo](./media/how-to-track-experiments/mlflow-experiments-8.png)

Tenga en cuenta que el estado de implementación va de __en transición__ a __correcto__. Además, esta sección de detalles proporciona las direcciones URL de Swagger y el punto de conexión REST que un desarrollador de aplicaciones puede usar para integrar el modelo de Machine Learning en sus aplicaciones.

Puede probar el punto de conexión mediante [Postman](https://www.postman.com/); también puede usar el SDK de AzureML:

```python
from azureml.core import Webservice
import json

# if you called your service differently then change the name below
service = Webservice(ws, name="diabetes-service")

input_payload = json.dumps({
    'data': X_test[0:2].tolist(),
    'method': 'predict'  # If you have a classification model, you can get probabilities by changing this to 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

### <a name="step-4-clean-up"></a>Paso 4: Limpieza

Elimine el proceso de inferencia que creó en el paso 1 para no incurrir en gastos de proceso continuos. En el menú de la izquierda, en Azure Machine Learning Studio, haga clic en Proceso > Clústeres de inferencia > Seleccione el proceso > Eliminar.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre la [implementación de modelos en AzureML](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-and-where)
