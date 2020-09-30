---
title: Visualización de experimentos con TensorBoard
titleSuffix: Azure Machine Learning
description: Inicie TensorBoard para visualizar los historiales de ejecución de los experimentos e identificar posibles áreas para el ajuste y el reentrenamiento de hiperparámetros.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: maxluk
ms.author: maxluk
ms.date: 02/27/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: da5c128b9e0befd69e1ded6b47644a3c64b8f657
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905054"
---
# <a name="visualize-experiment-runs-and-metrics-with-tensorboard-and-azure-machine-learning"></a>Visualización de las ejecuciones y las métricas de los experimentos con TensorBoard y Azure Machine Learning


En este artículo, obtendrá información sobre cómo ver las métricas y ejecuciones de los experimentos en TensorBoard mediante [el paquete `tensorboard`](https://docs.microsoft.com/python/api/azureml-tensorboard/?view=azure-ml-py&preserve-view=true) del SDK principal de Azure Machine Learning. Una vez que haya inspeccionado las ejecuciones de sus experimentos, puede ajustar y reentrenar mejor sus modelos de Machine Learning.

[TensorBoard](https://www.tensorflow.org/tensorboard/r1/overview) es un conjunto de aplicaciones web para inspeccionar y comprender el rendimiento y la estructura de sus experimentos.

La forma de iniciar TensorBoard con experimentos de Azure Machine Learning depende del tipo de experimento:
+ Si el experimento genera archivos de registro de forma nativa que puede utilizar TensorBoard, como experimentos de PyTorch, Chainer y TensorFlow, puede [iniciar directamente TensorBoard](#direct) desde el historial de ejecución del experimento. 

+ En el caso de los experimentos que no generan de forma nativa archivos que pueda usar TensorBoard, tales como los experimentos de Scikit-learn o Azure Machine Learning, utilice [el método `export_to_tensorboard()`](#export) para exportar los historiales de ejecución como registros de TensorBoard e iniciar TensorBoard desde allí. 

> [!TIP]
> La información de este documento va destinada principalmente a aquellos científicos de datos y desarrolladores que deseen supervisar el proceso de entrenamiento del modelo. Si es administrador y está interesado en la supervisión del uso de recursos y eventos desde Azure Machine Learning, como cuotas, ejecuciones de entrenamiento completadas o implementaciones de modelos completadas, consulte [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="prerequisites"></a>Prerrequisitos

* Para iniciar TensorBoard y ver los historiales de ejecución de sus experimentos, sus experimentos deben haber habilitado previamente el registro para realizar un seguimiento de sus métricas y rendimiento.  

* El código de este documento se puede ejecutar en cualquiera de los siguientes entornos: 

    * Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación

        * Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.

        * En la carpeta de muestras del servidor de cuadernos, vaya a estos directorios para encontrar dos cuadernos completados y expandidos:
            * **how-to-use-azureml > training-with-deep-learning > export-run-history-to-tensorboard > export-run-history-to-tensorboard.ipynb**

            * **how-to-use-azureml > track-and-monitor-experiments > tensorboard.ipynb**

    * Su propio servidor de Jupyter Notebook
       * [Instalar el SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true) con el elemento `tensorboard` adicional
        * [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).  
        * [Cree un archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
  
<a name="direct"></a>

## <a name="option-1-directly-view-run-history-in-tensorboard"></a>Opción 1: Ver directamente el historial de ejecución en TensorBoard

Esta opción funciona en experimentos que generan de forma nativa archivos de registro que puede consumir TensorBoard, como los experimentos de PyTorch, Chainer y TensorFlow. Si no es así en el caso de su experimento, use [el método `export_to_tensorboard()`](#export) en su lugar.

El código de ejemplo siguiente usa el [experimento de demostración MNIST](https://raw.githubusercontent.com/tensorflow/tensorflow/r1.8/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py) del repositorio de TensorFlow en un destino de proceso remoto, Proceso de Azure Machine Learning. A continuación, entrenamos nuestro modelo con el [estimador de TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py&preserve-view=true) personalizado del SDK, y, a continuación, iniciamos TensorBoard con este experimento de TensorFlow, es decir, un experimento que genera archivos de eventos de TensorBoard de forma nativa.

### <a name="set-experiment-name-and-create-project-folder"></a>Establecimiento del nombre del experimento y creación de la carpeta del proyecto

Aquí asignamos un nombre al experimento y creamos su carpeta. 
 
```python
from os import path, makedirs
experiment_name = 'tensorboard-demo'

# experiment folder
exp_dir = './sample_projects/' + experiment_name

if not path.exists(exp_dir):
    makedirs(exp_dir)

```

### <a name="download-tensorflow-demo-experiment-code"></a>Descarga del código del experimento de demostración de TensorFlow

El repositorio de TensorFlow tiene una demostración MNIST con una amplia instrumentación de TensorBoard. No modificamos nada de este código de demostración (y tampoco es necesario hacerlo) para que funcione con Azure Machine Learning. En el código siguiente, descargamos el código MNIST y lo guardamos en la carpeta recién creada del experimento.

```python
import requests
import os

tf_code = requests.get("https://raw.githubusercontent.com/tensorflow/tensorflow/r1.8/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py")
with open(os.path.join(exp_dir, "mnist_with_summaries.py"), "w") as file:
    file.write(tf_code.text)
```
En el archivo del código MNIST, mnist_with_summaries.py, tenga en cuenta que hay líneas que llaman a `tf.summary.scalar()`, `tf.summary.histogram()`, `tf.summary.FileWriter()`, etc. Estos métodos agrupan, registran y etiquetan métricas clave de sus experimentos en el historial de ejecución. El método `tf.summary.FileWriter()` es especialmente importante, ya que serializa los datos de las métricas de sus experimentos registrados, lo que permite a TensorBoard generar visualizaciones fuera de ellos.

 ### <a name="configure-experiment"></a>Configuración del experimento

En la siguiente tabla, configuramos nuestro experimento y los directorios para los registros y datos. Estos registros se cargarán en el servicio de artefacto, al que TensorBoard accede más adelante.

>[!Note]
> En este ejemplo de TensorFlow, deberá instalar TensorFlow en la máquina local. Además, el módulo de TensorBoard (es decir, el que se incluye con TensorFlow) debe ser accesible para el kernel de este cuaderno, ya que la máquina local es la que ejecuta TensorBoard.

```Python
import azureml.core
from azureml.core import Workspace
from azureml.core import Experiment

ws = Workspace.from_config()

# create directories for experiment logs and dataset
logs_dir = os.path.join(os.curdir, "logs")
data_dir = os.path.abspath(os.path.join(os.curdir, "mnist_data"))

if not path.exists(data_dir):
    makedirs(data_dir)

os.environ["TEST_TMPDIR"] = data_dir

# Writing logs to ./logs results in their being uploaded to Artifact Service,
# and thus, made accessible to our TensorBoard instance.
script_params = ["--log_dir", logs_dir]

# Create an experiment
exp = Experiment(ws, experiment_name)
```

### <a name="create-a-cluster-for-your-experiment"></a>Creación de un clúster para el experimento
Crearemos un clúster de AmlCompute para este experimento, sin embargo, puede crear sus experimentos en cualquier entorno y podrá iniciar TensorBoard igualmente con el historial de ejecución del experimento. 

```Python
from azureml.core.compute import ComputeTarget, AmlCompute

cluster_name = "cpucluster"

cts = ws.compute_targets
found = False
if cluster_name in cts and cts[cluster_name].type == 'AmlCompute':
   found = True
   print('Found existing compute target.')
   compute_target = cts[cluster_name]
if not found:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2', 
                                                           max_nodes=4)

    # create the cluster
    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

compute_target.wait_for_completion(show_output=True, min_node_count=None)

# use get_status() to get a detailed status for the current cluster. 
# print(compute_target.get_status().serialize())
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="submit-run-with-tensorflow-estimator"></a>Envío de la ejecución con el estimador de TensorFlow

El estimador de TensorFlow proporciona una manera sencilla de iniciar un trabajo de entrenamiento de TensorFlow en un destino de proceso. Se implementa a través de la clase genérica [`estimator`](https://docs.microsoft.com//python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py&preserve-view=true), que se puede usar para admitir cualquier marco. Para obtener más información sobre cómo entrenar modelos con el estimador genérico, consulte [Entrenamiento de modelos con Azure Machine Learning mediante un objeto Estimator](how-to-train-ml-models.md).

```Python
from azureml.train.dnn import TensorFlow

tf_estimator = TensorFlow(source_directory=exp_dir,
                          compute_target=compute_target,
                          entry_script='mnist_with_summaries.py',
                          script_params=script_params)

run = exp.submit(tf_estimator)
```

### <a name="launch-tensorboard"></a>Inicio de TensorBoard

Puede iniciar TensorBoard durante su ejecución o después de que se complete. A continuación, creamos una instancia de objeto de TensorBoard, `tb`, que toma el historial de ejecución del experimento cargado en `run` y, luego, inicia TensorBoard con el método `start()`. 
  
El [constructor de TensorBoard](https://docs.microsoft.com/python/api/azureml-tensorboard/azureml.tensorboard.tensorboard?view=azure-ml-py&preserve-view=true) toma una matriz de ejecuciones, así que asegúrese de pasarla como una matriz de un solo elemento.

```python
from azureml.tensorboard import Tensorboard

tb = Tensorboard([run])

# If successful, start() returns a string with the URI of the instance.
tb.start()

# After your job completes, be sure to stop() the streaming otherwise it will continue to run. 
tb.stop()
```

>[!Note]
 Aunque en este ejemplo se ha usado TensorFlow, se puede usar TensorBoard con la misma facilidad con los modelos de PyTorch o Chainer. TensorFlow debe estar disponible en el equipo que ejecuta TensorBoard, pero no es necesario en el que realiza cálculos de PyTorch o Chainer. 


<a name="export"></a>

## <a name="option-2-export-history-as-log-to-view-in-tensorboard"></a>Opción 2: Exportación del historial como registro para ver en TensorBoard

El código siguiente configura un experimento de ejemplo, comienza el proceso de registro mediante las API del historial de ejecución de Azure Machine Learning y exporta el historial de ejecución del experimento en registros que puede usar TensorBoard para la visualización. 

### <a name="set-up-experiment"></a>Configuración del experimento

El código siguiente configura un nuevo experimento y asigna un nombre al directorio de ejecución `root_run`. 

```python
from azureml.core import Workspace, Experiment
import azureml.core

# set experiment name and run name
ws = Workspace.from_config()
experiment_name = 'export-to-tensorboard'
exp = Experiment(ws, experiment_name)
root_run = exp.start_logging()
```

A continuación, cargamos el conjunto de datos diabetes, que es un conjunto de datos pequeño integrado que se incluye con scikit-learn, y lo dividimos en conjuntos de entrenamiento y prueba.

```Python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
X, y = load_diabetes(return_X_y=True)
columns = ['age', 'gender', 'bmi', 'bp', 's1', 's2', 's3', 's4', 's5', 's6']
x_train, x_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)
data = {
    "train":{"x":x_train, "y":y_train},        
    "test":{"x":x_test, "y":y_test}
}
```

### <a name="run-experiment-and-log-metrics"></a>Ejecución del experimento y registro de las métricas

Para este código, entrenamos un modelo de regresión lineal y registramos en el historial de ejecución las métricas clave, el coeficiente de alfa `alpha` y el error cuadrático medio`mse`.

```Python
from tqdm import tqdm
alphas = [.1, .2, .3, .4, .5, .6 , .7]
# try a bunch of alpha values in a Linear Regression (aka Ridge regression) mode
for alpha in tqdm(alphas):
  # create child runs and fit lines for the resulting models
  with root_run.child_run("alpha" + str(alpha)) as run:
 
   reg = Ridge(alpha=alpha)
   reg.fit(data["train"]["x"], data["train"]["y"])    
 
   preds = reg.predict(data["test"]["x"])
   mse = mean_squared_error(preds, data["test"]["y"])
   # End train and eval

# log alpha, mean_squared_error and feature names in run history
   root_run.log("alpha", alpha)
   root_run.log("mse", mse)
```

### <a name="export-runs-to-tensorboard"></a>Exportación de las ejecuciones a TensorBoard

Con el método [export_to_tensorboard()](https://docs.microsoft.com/python/api/azureml-tensorboard/azureml.tensorboard.export?view=azure-ml-py&preserve-view=true) del SDK, podemos exportar el historial de ejecución de nuestro experimento de Azure Machine Learning en los registros de TensorBoard, por lo que podemos verlos mediante TensorBoard.  

En el código siguiente, creamos la carpeta `logdir` en nuestro directorio de trabajo actual. Esta carpeta es donde exportaremos el historial de ejecución de nuestro experimento y los registros de `root_run` y, a continuación, marcaremos dicha ejecución como completada. 

```Python
from azureml.tensorboard.export import export_to_tensorboard
import os

logdir = 'exportedTBlogs'
log_path = os.path.join(os.getcwd(), logdir)
try:
    os.stat(log_path)
except os.error:
    os.mkdir(log_path)
print(logdir)

# export run history for the project
export_to_tensorboard(root_run, logdir)

root_run.complete()
```

>[!Note]
 También puede exportar una ejecución concreta para TensorBoard si especifica el nombre de la ejecución `export_to_tensorboard(run_name, logdir)`.

### <a name="start-and-stop-tensorboard"></a>Inicio y detención de TensorBoard:
una vez que se haya exportado nuestro historial de ejecución para este experimento, podemos iniciar TensorBoard con el método [start ()](https://docs.microsoft.com/python/api/azureml-tensorboard/azureml.tensorboard.tensorboard?view=azure-ml-py#&preserve-view=truestart-start-browser-false-). 

```Python
from azureml.tensorboard import Tensorboard

# The TensorBoard constructor takes an array of runs, so be sure and pass it in as a single-element array here
tb = Tensorboard([], local_root=logdir, port=6006)

# If successful, start() returns a string with the URI of the instance.
tb.start()
```

Cuando haya terminado, asegúrese de llamar al método [stop()](https://docs.microsoft.com/python/api/azureml-tensorboard/azureml.tensorboard.tensorboard?view=azure-ml-py#&preserve-view=truestop--) del objeto TensorBoard. De lo contrario, TensorBoard seguirá ejecutándose hasta que apague el kernel del cuaderno. 

```python
tb.stop()
```

## <a name="next-steps"></a>Pasos siguientes

En este procedimiento, creó dos experiencias y aprendió a iniciar TensorBoard con sus historiales de ejecución a fin de identificar las áreas de posible ajuste y reentrenamiento. 

* Si está satisfecho con el modelo, diríjase al artículo [Cómo implementar un modelo](how-to-deploy-and-where.md). 
* Obtenga más información sobre [el ajuste de hiperparámetros](how-to-tune-hyperparameters.md). 
