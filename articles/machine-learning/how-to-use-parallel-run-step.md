---
title: Ejecución de predicciones por lotes en macrodatos
titleSuffix: Azure Machine Learning
description: Aprenda a obtener inferencias por lotes de forma asincrónica en grandes cantidades de datos mediante ParallelRunStep en Azure Machine Learning. ParallelRunStep proporciona funcionalidades de procesamiento en paralelo de forma integrada y optimiza la inferencia de alto rendimiento y de tipo "fire-and-forget" (dispara y olvida) para casos de uso de macrodatos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.reviewer: jmartens, larryfr
ms.author: tracych
author: tracychms
ms.date: 08/14/2020
ms.custom: Build2020, devx-track-python
ms.openlocfilehash: 8e8d0a13bc01e95311345154648ecb00b624c4bf
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905608"
---
# <a name="run-batch-inference-on-large-amounts-of-data-by-using-azure-machine-learning"></a>Ejecución de la inferencia por lotes en grandes cantidades de datos mediante Azure Machine Learning


En este artículo se muestra cómo ejecutar el modelo de Azure Machine Learning en paralelo y evaluar rápidamente grandes cantidades de datos. 

La inferencia de grandes conjuntos de datos o con modelos complicados puede llevar mucho tiempo. La clase `ParallelRunStep` permite realizar el procesamiento en paralelo, lo que puede acelerar los resultados generales. Aunque la ejecución de una sola evaluación sea bastante rápida, muchos escenarios (detección de objetos, procesamiento de vídeo, procesamiento de lenguaje natural, etc.) conllevan la ejecución de numerosas evaluaciones. 

Con `ParallelRunStep`, es fácil escalar las inferencias por lotes a grandes grupos de máquinas. Tales grupos pueden administrar terabytes de datos estructurados o no estructurados con una mayor productividad y mejores costos.

En este artículo, obtendrá información sobre las siguientes tareas:

> 1. Configurar los recursos de aprendizaje automático.
> 1. Configurar las entradas y salidas de datos de inferencia por lotes.
> 1. Preparar el modelo de clasificación de imágenes previamente entrenado basado en el conjunto de datos de [MNIST](https://publicdataset.azurewebsites.net/dataDetail/mnist/). 
> 1.  Escriba el script de inferencia.
> 1. Cree una [canalización de aprendizaje automático](concept-ml-pipelines.md) que contenga ParallelRunStep y ejecute la inferencia por lotes en las imágenes de prueba de MNIST. 
> 1. Vuelva a enviar una ejecución de inferencia por lotes con nuevos parámetros y entrada de datos. 
> 1. Vea los resultados.

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Para un inicio rápido guiado, complete el [tutorial de instalación](tutorial-1st-experiment-sdk-setup.md) si aún no tiene un área de trabajo de Azure Machine Learning. 

* Para administrar el entorno y las dependencias propias, consulte la [guía paso a paso](how-to-configure-environment.md) sobre la configuración de su propio entorno local.

## <a name="set-up-machine-learning-resources"></a>Configurar los recursos de aprendizaje automático

Las siguientes acciones configuran los recursos de aprendizaje automático necesarios que necesita para ejecutar una canalización de inferencias por lotes:

- Conexión a un área de trabajo.
- Creación o asociación de un recurso de proceso existente.

### <a name="configure-workspace"></a>Configuración del área de trabajo

Cree un objeto de área de trabajo desde el área de trabajo existente. `Workspace.from_config()` lee el archivo config.json y carga los detalles en un objeto denominado ws.

```python
from azureml.core import Workspace

ws = Workspace.from_config()
```

> [!IMPORTANT]
> Este fragmento de código espera que la configuración del área de trabajo se guarde en el directorio actual o en su elemento primario. Para más información sobre cómo crear un área de trabajo, consulte [Creación y administración de áreas de trabajo de Azure Machine Learning](how-to-manage-workspace.md). Para más información sobre cómo guardar la configuración en un archivo, consulte [Creación de un archivo de configuración de área de trabajo](how-to-configure-environment.md#workspace).

### <a name="create-a-compute-target"></a>Creación de un destino de proceso

En Azure Machine Learning, el *proceso* (o *destino de proceso*) se refiere a las máquinas o clústeres que realizarán los pasos del cálculo en su canal de aprendizaje automático. Ejecute el código siguiente para crear un destino [AmlCompute](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py&preserve-view=true) basado en CPU.

```python
from azureml.core.compute import AmlCompute, ComputeTarget
from azureml.core.compute_target import ComputeTargetException

# choose a name for your cluster
compute_name = os.environ.get("AML_COMPUTE_CLUSTER_NAME", "cpucluster")
compute_min_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MIN_NODES", 0)
compute_max_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MAX_NODES", 4)

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
vm_size = os.environ.get("AML_COMPUTE_CLUSTER_SKU", "STANDARD_D2_V2")


if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('found compute target. just use it. ' + compute_name)
else:
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = vm_size,
                                                                min_nodes = compute_min_nodes, 
                                                                max_nodes = compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(ws, compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

## <a name="configure-inputs-and-output"></a>Configuración de las entradas y salidas

### <a name="create-a-datastore-with-sample-images"></a>Creación de un almacén de datos con imágenes de ejemplo

Obtenga el conjunto de evaluación de MNIST del contenedor de blobs público `sampledata` en una cuenta llamada `pipelinedata`. Cree un almacén de datos con el nombre `mnist_datastore` que apunte a este contenedor. En la siguiente llamada a `register_azure_blob_container`, si se establece la marca `overwrite` en `True` se sobrescribe cualquier almacén de datos que se haya creado anteriormente con ese nombre. 

Puede cambiar este paso para que apunte al contenedor de blobs si proporciona sus propios valores para `datastore_name`, `container_name` y `account_name`.

```python
from azureml.core import Datastore
from azureml.core import Workspace

mnist_blob = Datastore.register_azure_blob_container(ws, 
                      datastore_name="mnist_datastore", 
                      container_name="sampledata", 
                      account_name="pipelinedata",
                      overwrite=True)
```

A continuación, especifique el almacén de datos predeterminado del área de trabajo como almacén de datos de salida. Lo usará para la salida de la inferencia.

Cuando cree su área de trabajo, [Azure Files](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)  y [Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)  se adjuntan al área de trabajo de forma predeterminada. Azure Files es el almacén de datos predeterminado para un área de trabajo, pero también puede usar Blob Storage como un almacén de datos. Para obtener más información, consulte [Opciones de almacenamiento de Azure](https://docs.microsoft.com/azure/storage/common/storage-decide-blobs-files-disks).

```python
def_data_store = ws.get_default_datastore()
```

### <a name="create-the-data-inputs"></a>Creación de las entradas de datos

Las entradas de la inferencia por lotes son los datos que desea dividir en particiones para el procesamiento en paralelo. Una canalización de inferencia por lotes acepta entradas de datos con [`Dataset`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py&preserve-view=true).

`Dataset` es una clase para explorar, transformar y administrar datos en Azure Machine Learning. Hay dos tipos: [`TabularDataset`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py&preserve-view=true) y [`FileDataset`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.filedataset?view=azure-ml-py&preserve-view=true). En este ejemplo, se usa `FileDataset` como entradas. `FileDataset` permite descargar o montar los archivos en el proceso. Al crear un conjunto de datos, se crea una referencia a la ubicación del origen de datos. Si aplicó alguna transformación de subconjunto al conjunto de datos, también se almacenará en el conjunto de datos. Los datos se mantienen en la ubicación existente, por lo que no se genera ningún costo de almacenamiento adicional.

Para más información sobre los conjuntos de datos de Azure Machine Learning, consulte [Creación de conjuntos de datos y acceso a ellos (versión preliminar)](https://docs.microsoft.com/azure/machine-learning/how-to-create-register-datasets).

```python
from azureml.core.dataset import Dataset

path_on_datastore = mnist_blob.path('mnist/')
input_mnist_ds = Dataset.File.from_files(path=path_on_datastore, validate=False)
```

Para usar entradas de datos dinámicas al ejecutar la canalización de inferencia por lotes, puede definir las entradas `Dataset` como [`PipelineParameter`](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.pipelineparameter?view=azure-ml-py&preserve-view=true). Puede especificar el conjunto de datos de entradas cada vez que vuelva a enviar una ejecución de canalización de inferencia por lotes.

```python
from azureml.data.dataset_consumption_config import DatasetConsumptionConfig
from azureml.pipeline.core import PipelineParameter

pipeline_param = PipelineParameter(name="mnist_param", default_value=input_mnist_ds)
input_mnist_ds_consumption = DatasetConsumptionConfig("minist_param_config", pipeline_param).as_mount()
```

### <a name="create-the-output"></a>Creación de la salida

Los objetos [`PipelineData`](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py&preserve-view=true) se usan para transferir los datos intermedios entre los pasos de la canalización. En este ejemplo, se usa para la salida de la inferencia.

```python
from azureml.pipeline.core import Pipeline, PipelineData

output_dir = PipelineData(name="inferences", datastore=def_data_store)
```

## <a name="prepare-the-model"></a>Preparar el modelo

[Descargue el modelo de clasificación de imágenes previamente entrenado](https://pipelinedata.blob.core.windows.net/mnist-model/mnist-tf.tar.gz) y extráigalo en el directorio `models`.

```python
import os
import tarfile
import urllib.request

model_dir = 'models'
if not os.path.isdir(model_dir):
    os.mkdir(model_dir)

url="https://pipelinedata.blob.core.windows.net/mnist-model/mnist-tf.tar.gz"
response = urllib.request.urlretrieve(url, "model.tar.gz")
tar = tarfile.open("model.tar.gz", "r:gz")
tar.extractall(model_dir)
```

Después, registre el modelo con el área de trabajo de modelo que esté disponible para el recurso de proceso.

```python
from azureml.core.model import Model

# Register the downloaded model 
model = Model.register(model_path="models/",
                       model_name="mnist",
                       tags={'pretrained': "mnist"},
                       description="Mnist trained tensorflow model",
                       workspace=ws)
```

## <a name="write-your-inference-script"></a>Escritura del script de inferencia

>[!Warning]
>El código siguiente es solo un ejemplo que utiliza el [cuaderno de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines/parallel-run). Deberá crear su propio script para el escenario.

El script *debe contener* dos funciones:
- `init()`: utilice esta función para cualquier preparación costosa o común para la inferencia posterior. Por ejemplo, para cargar el modelo en un objeto global. Solo se llamará a esta función una vez al principio del proceso.
-  `run(mini_batch)`: la función se ejecutará para cada instancia de `mini_batch`.
    -  `mini_batch`: `ParallelRunStep` invocará el método run y pasará una lista o `DataFrame` de Pandas como argumento al método. Cada entrada de mini_batch será una ruta de acceso de archivo si la entrada es `FileDataset`, o `DataFrame` de Pandas si la entrada es `TabularDataset`.
    -  `response`: El método run() debe devolver `DataFrame` de Pandas o una matriz. Para append_row output_action, estos elementos devueltos se anexan al archivo de salida común. Para summary_only, se omite el contenido de los elementos. Para todas las acciones de salida, cada elemento de salida devuelto indica una ejecución correcta del elemento de entrada en el minilote de entrada. Asegúrese de que se incluyen suficientes datos en el resultado de la ejecución para asignar la entrada al resultado de la salida de la ejecución. La salida de la ejecución se escribirá en el archivo de salida y no se garantiza que esté en orden, por lo que debe usar alguna clave en la salida para asignarla a la entrada.

```python
%%writefile digit_identification.py
# Snippets from a sample script.
# Refer to the accompanying digit_identification.py
# (https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines/parallel-run)
# for the implementation script.

import os
import numpy as np
import tensorflow as tf
from PIL import Image
from azureml.core import Model


def init():
    global g_tf_sess

    # Pull down the model from the workspace
    model_path = Model.get_model_path("mnist")

    # Construct a graph to execute
    tf.reset_default_graph()
    saver = tf.train.import_meta_graph(os.path.join(model_path, 'mnist-tf.model.meta'))
    g_tf_sess = tf.Session()
    saver.restore(g_tf_sess, os.path.join(model_path, 'mnist-tf.model'))


def run(mini_batch):
    print(f'run method start: {__file__}, run({mini_batch})')
    resultList = []
    in_tensor = g_tf_sess.graph.get_tensor_by_name("network/X:0")
    output = g_tf_sess.graph.get_tensor_by_name("network/output/MatMul:0")

    for image in mini_batch:
        # Prepare each image
        data = Image.open(image)
        np_im = np.array(data).reshape((1, 784))
        # Perform inference
        inference_result = output.eval(feed_dict={in_tensor: np_im}, session=g_tf_sess)
        # Find the best probability, and add it to the result list
        best_result = np.argmax(inference_result)
        resultList.append("{}: {}".format(os.path.basename(image), best_result))

    return resultList
```

Si tiene otro archivo o carpeta en el mismo directorio que el script de inferencia, puede hacer referencia a él buscando el directorio de trabajo actual.

```python
script_dir = os.path.realpath(os.path.join(__file__, '..',))
file_path = os.path.join(script_dir, "<file_name>")
```

## <a name="build-and-run-the-pipeline-containing-parallelrunstep"></a>Compilación y ejecución de la canalización que contiene ParallelRunStep

Ahora tiene todo lo que necesita: las entradas de datos, el modelo, la salida y el script de inferencia. Vamos a compilar la canalización de inferencia por lotes que contiene ParallelRunStep.

### <a name="prepare-the-environment"></a>Preparación del entorno

Primero, especifique las dependencias para el script. Esto le permite instalar paquetes pip, así como configurar el entorno.

Incluya siempre **azureml-core** y **azureml-dataset-runtime[pandas, fuse]** en la lista de paquetes de pip. Si usa una imagen de Docker personalizada (user_managed_dependencies=true), también debe tener instalado Conda.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_GPU_IMAGE

batch_conda_deps = CondaDependencies.create(pip_packages=["tensorflow==1.15.2", "pillow", 
                                                          "azureml-core", "azureml-dataset-runtime[pandas, fuse]"])

batch_env = Environment(name="batch_environment")
batch_env.python.conda_dependencies = batch_conda_deps
batch_env.docker.enabled = True
batch_env.docker.base_image = DEFAULT_GPU_IMAGE
```

### <a name="specify-the-parameters-using-parallelrunconfig"></a>Especificación de los parámetros mediante ParallelRunConfig

`ParallelRunConfig` es la configuración principal de la instancia de `ParallelRunStep` dentro de la canalización de Azure Machine Learning. Se usa para encapsular el script y configurar los parámetros necesarios, incluidos los de las siguientes entradas:
- `entry_script`: script de usuario como ruta de acceso de archivo local que se ejecutará en paralelo en varios nodos. Si `source_directory` está presente, utilice una ruta de acceso relativa. De lo contrario, use cualquier ruta de acceso accesible desde la máquina.
- `mini_batch_size`: tamaño del minilote que se pasa a una sola llamada de `run()`. (Opcional; el valor predeterminado es `10` archivos para `FileDataset`, y `1MB` para `TabularDataset`).
    - En el caso de `FileDataset`, es el número de archivos con un valor mínimo de `1`. Puede combinar varios archivos en un solo minilote.
    - En el caso de `TabularDataset`, es el tamaño de los datos. Los valores posibles son `1024`, `1024KB`, `10MB` y `1GB`. `1MB` es el valor recomendado. El minilote de `TabularDataset` nunca cruzará los límites de los archivos. Por ejemplo, si tiene archivos. csv de varios tamaños, el menor es de 100 KB y el mayor es de 10 MB. Si establece `mini_batch_size = 1MB`, los archivos con un tamaño menor que 1 MB se tratarán como un solo minilote. Los archivos de tamaño mayor que 1 MB se dividirán en varios minilotes.
- `error_threshold`: número de errores de registro para `TabularDataset` y errores de archivo para `FileDataset` que se deben omitir durante el procesamiento. Si el recuento de errores de la entrada supera este valor, el trabajo se anulará. El umbral de error es para toda la entrada y no para los minilotes individuales que se envían al método `run()`. El intervalo es `[-1, int.max]`. La parte `-1` indica que se omitirán todos los errores durante el procesamiento.
- `output_action`: uno de los valores siguientes indica cómo se organizará la salida:
    - `summary_only`: el script de usuario almacenará la salida. `ParallelRunStep` usará la salida solo para el cálculo del umbral de error.
    - `append_row`: en las entradas, solo se creará un archivo en la carpeta de salida para anexar todas las salidas separadas por líneas.
- `append_row_file_name`: para personalizar el nombre del archivo de salida de append_row output_action (opcional; el valor predeterminado es `parallel_run_step.txt`).
- `source_directory`: rutas de acceso a las carpetas que contienen todos los archivos que se van a ejecutar en el destino de proceso (opcional).
- `compute_target`: Solo se admite `AmlCompute`.
- `node_count`: número de nodos de proceso que se usarán para ejecutar el script de usuario.
- `process_count_per_node`: número de procesos por nodo. El procedimiento recomendado es establecerlo en el número de GPU o CPU que tenga un nodo (opcional; el valor predeterminado es `1`).
- `environment`: definición del entorno de Python. Puede configurarla para usar un entorno de Python existente o un entorno temporal. La definición también es responsable de establecer las dependencias de la aplicación necesarias (opcional).
- `logging_level`: nivel de detalle del registro. Los valores con nivel de detalle en aumento son: `WARNING`, `INFO` y `DEBUG`. (Opcional; el valor predeterminado es `INFO`).
- `run_invocation_timeout`: tiempo de espera de invocación del método `run()` en segundos. (Opcional; el valor predeterminado es `60`).
- `run_max_try`: número máximo de intentos de `run()` para un minilote. Se produce un error en `run()` si se genera una excepción o no se devuelve nada cuando se alcanza `run_invocation_timeout` (opcional; el valor predeterminado es `3`). 

Puede especificar `mini_batch_size`, `node_count`, `process_count_per_node`, `logging_level`, `run_invocation_timeout` y `run_max_try` como `PipelineParameter`, de modo que, cuando vuelva a enviar una ejecución de la canalización, pueda optimizar los valores de los parámetros. En este ejemplo, se usa `PipelineParameter` para `mini_batch_size` y `Process_count_per_node`, y se cambiarán estos valores cuando se vuelva a enviar una ejecución más tarde. 

En este ejemplo se da por supuesto que usa el script `digit_identification.py` analizado anteriormente. Si usa su propio script, cambie los parámetros `source_directory` y `entry_script` en consecuencia.

```python
from azureml.pipeline.core import PipelineParameter
from azureml.pipeline.steps import ParallelRunConfig

parallel_run_config = ParallelRunConfig(
    source_directory='.',
    entry_script="digit_identification.py",
    mini_batch_size=PipelineParameter(name="batch_size_param", default_value="5"),
    error_threshold=10,
    output_action="append_row",
    append_row_file_name="mnist_outputs.txt",
    environment=batch_env,
    compute_target=compute_target,
    process_count_per_node=PipelineParameter(name="process_count_param", default_value=2),
    node_count=2)
```

### <a name="create-the-parallelrunstep"></a>Creación de ParallelRunStep

Cree ParallelRunStep mediante el script, la configuración del entorno y los parámetros. Especifique el destino de proceso que ya adjuntó a su área de trabajo como destino de ejecución del script de inferencia. Use `ParallelRunStep` para crear el paso de canalización de inferencias por lotes, que toma todos los parámetros siguientes:
- `name`: nombre del paso, con las siguientes restricciones de nomenclatura: unique, 3-32 caracteres y regex ^\[a-z\]([-a-z0-9]*[a-z0-9])?$.
- `parallel_run_config`: objeto `ParallelRunConfig`, tal y como se definió anteriormente.
- `inputs`: uno o varios conjuntos de datos de Azure Machine Learning de tipo único que se van a particionar para el procesamiento en paralelo.
- `side_inputs`: uno o varios datos de referencia o conjuntos de datos que se usan como entradas laterales sin necesidad de crear particiones.
- `output`: objeto `PipelineData` que corresponde al directorio de salida.
- `arguments`: lista de los argumentos pasados al script de usuario. Use unknown_args para recuperarlos en el script de entrada (opcional).
- `allow_reuse`: sirve para decidir si el paso debe volver a usar los resultados anteriores cuando se ejecuta con la misma configuración o entrada. Si este parámetro es `False`, siempre se generará una nueva ejecución para este paso durante la ejecución de la canalización. (Opcional; el valor predeterminado es `True`).

```python
from azureml.pipeline.steps import ParallelRunStep

parallelrun_step = ParallelRunStep(
    name="predict-digits-mnist",
    parallel_run_config=parallel_run_config,
    inputs=[input_mnist_ds_consumption],
    output=output_dir,
    allow_reuse=True
)
```
### <a name="create-and-run-the-pipeline"></a>Creación y ejecución de la canalización

Ahora ejecute la canalización. En primer lugar, cree un objeto [`Pipeline`](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline%28class%29?view=azure-ml-py&preserve-view=true) con la referencia al área de trabajo y el paso de canalización que creó. El parámetro `steps` es una matriz de pasos. En este caso, la inferencia por lotes consta de un solo paso. Para compilar canalizaciones con varios pasos, colóquelos en orden en esta matriz.

Luego, use la función `Experiment.submit()` para enviar la canalización para su ejecución.

```python
from azureml.pipeline.core import Pipeline
from azureml.core.experiment import Experiment

pipeline = Pipeline(workspace=ws, steps=[parallelrun_step])
experiment = Experiment(ws, 'digit_identification')
pipeline_run = experiment.submit(pipeline)
```

## <a name="monitor-the-batch-inference-job"></a>Supervisión del trabajo de inferencias por lotes

Un trabajo de inferencias por lotes puede tardar mucho tiempo. En este ejemplo se supervisa el progreso mediante un widget de Jupyter. También puede administrar el progreso del trabajo mediante:

* Azure Machine Learning Studio. 
* La salida de la consola del objeto [`PipelineRun`](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.run.pipelinerun?view=azure-ml-py&preserve-view=true).

```python
from azureml.widgets import RunDetails
RunDetails(pipeline_run).show()

pipeline_run.wait_for_completion(show_output=True)
```

## <a name="resubmit-a-run-with-new-data-inputs-and-parameters"></a>Nuevo envío de una ejecución con nuevos parámetros y entradas de datos

Dado que ha realizado las entradas y varias configuraciones como `PipelineParameter`, puede volver a enviar una ejecución de inferencia por lotes con otra entrada de conjunto de datos y ajustar los parámetros sin tener que crear una canalización completamente nueva. Usará el mismo almacén de datos, pero una sola imagen como entrada de datos.

```python
path_on_datastore = mnist_blob.path('mnist/0.png')
single_image_ds = Dataset.File.from_files(path=path_on_datastore, validate=False)

pipeline_run_2 = experiment.submit(pipeline, 
                                   pipeline_parameters={"mnist_param": single_image_ds, 
                                                        "batch_size_param": "1",
                                                        "process_count_param": 1}
)

pipeline_run_2.wait_for_completion(show_output=True)
```
## <a name="view-the-results"></a>View the results

Los resultados de la ejecución anterior se escriben en el objeto `DataStore` especificado en el objeto `PipelineData` como los datos de salida, que en este caso se llama *inferences*. Los resultados se almacenan en el contenedor de blobs predeterminado; puede ir a la cuenta de almacenamiento y verlos con el Explorador de Storage. La ruta de acceso del archivo es azureml-blobstore-*GUID*/azureml/*RunId*/*output_dir*.

También puede descargar estos datos para ver los resultados. A continuación se muestra el código de ejemplo para ver las 10 primeras filas.

```python
import pandas as pd
import tempfile

batch_run = pipeline_run.find_step_run(parallelrun_step.name)[0]
batch_output = batch_run.get_output_data(output_dir.name)

target_dir = tempfile.mkdtemp()
batch_output.download(local_path=target_dir)
result_file = os.path.join(target_dir, batch_output.path_on_datastore, parallel_run_config.append_row_file_name)

df = pd.read_csv(result_file, delimiter=":", header=None)
df.columns = ["Filename", "Prediction"]
print("Prediction has ", df.shape[0], " rows")
df.head(10) 
```

## <a name="next-steps"></a>Pasos siguientes

Para ver cómo funciona esto de un extremo a otro, pruebe el [cuaderno de inferencias por lotes](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines/parallel-run). 

Para instrucciones sobre depuración y solución de problemas de ParallelRunStep, consulte la [guía de procedimientos](how-to-debug-parallel-run-step.md).

Para instrucciones sobre depuración y solución de problemas para las canalizaciones, consulte la [guía paso a paso](how-to-debug-pipelines.md).

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

