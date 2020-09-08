---
title: 'Tutorial: Canalizaciones de ML para la puntuación por lotes'
titleSuffix: Azure Machine Learning
description: En este tutorial, compilará una canalización de aprendizaje automático para realizar la puntuación por lotes en un modelo de clasificación de imágenes. Azure Machine Learning le permiten centrarse en el aprendizaje automático, en lugar de en la infraestructura y la automatización.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: lobrien
ms.author: laobri
ms.reviewer: laobri
ms.date: 03/11/2020
ms.custom: contperfq4, devx-track-python
ms.openlocfilehash: 600b19ffac61f8f7c7336f114c6b52c6bc88b5ad
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89489517"
---
# <a name="tutorial-build-an-azure-machine-learning-pipeline-for-batch-scoring"></a>Tutorial: Compilación de una canalización de Azure Machine Learning para la puntuación por lotes

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este tutorial avanzado, aprenderá a crear una canalización en Azure Machine Learning para ejecutar un trabajo de puntuación por lotes. Las canalizaciones de Machine Learning optimizan el flujo de trabajo con velocidad, portabilidad y reutilización, con el fin de que pueda centrarse en el aprendizaje automático, en lugar de en la infraestructura y la automatización. Después de compilar y publicar una canalización, configurará un punto de conexión REST para desencadenar la canalización desde cualquier biblioteca HTTP en cualquier plataforma. 

En el ejemplo se usa un modelo de red neuronal convolucional [Inception-V3](https://arxiv.org/abs/1512.00567) entrenado previamente implementado en Tensorflow para clasificar las imágenes sin etiquetar. [Más información sobre las canalizaciones de aprendizaje automático](concept-ml-pipelines.md).

En este tutorial, va a completar las siguientes tareas:

> [!div class="checklist"]
> * Configuración del área de trabajo 
> * Descargar y almacenar datos de muestra
> * Crear objetos de un conjunto de datos para capturar y generar datos
> * Descargar, preparar y registrar el modelo en el área de trabajo
> * Aprovisionar destinos de proceso y crear un script de puntuación
> * Usar la clase `ParallelRunStep` para la puntuación de lotes asincrónicos
> * Compilar, ejecutar y publicar una canalización
> * Habilitar un punto de conexión REST para la canalización

Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

## <a name="prerequisites"></a>Requisitos previos

* Complete la [parte 1 del tutorial de instalación](tutorial-1st-experiment-sdk-setup.md) si aún no tiene un área de trabajo de Azure Machine Learning o una máquina virtual de cuadernos.
* Una vez terminado el tutorial de instalación, use el mismo servidor de cuadernos para abrir el cuaderno *tutorials/machine-learning-pipelines-advanced/tutorial-pipeline-batch-scoring-classification.ipynb*.

Si desea ejecutar el tutorial de instalación en su propio [entorno local](how-to-configure-environment.md#local) puede acceder al tutorial en [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials). Ejecute `pip install azureml-sdk[notebooks] azureml-pipeline-core azureml-pipeline-steps pandas requests` para obtener los paquetes requeridos.

## <a name="configure-workspace-and-create-a-datastore"></a>Configuración del área de trabajo y creación de un almacén de datos

Cree un objeto de área de trabajo a partir del área de trabajo de Azure Machine Learning existente.

```python
from azureml.core import Workspace
ws = Workspace.from_config()
```

> [!IMPORTANT]
> Este fragmento de código espera que la configuración del área de trabajo se guarde en el directorio actual o en su elemento primario. Para más información sobre cómo crear un área de trabajo, consulte [Creación y administración de áreas de trabajo de Azure Machine Learning](how-to-manage-workspace.md). Para más información sobre cómo guardar la configuración en un archivo, consulte [Creación de un archivo de configuración de área de trabajo](how-to-configure-environment.md#workspace).

## <a name="create-a-datastore-for-sample-images"></a>Creación de un almacén de datos para las imágenes de ejemplo

En la cuenta `pipelinedata`, obtenga el ejemplo de datos públicos de evaluación de ImageNet del contenedor de blobs público `sampledata`. Una llamada a `register_azure_blob_container()` hace que los datos estén disponibles en el área de trabajo con el nombre `images_datastore`. A continuación, establezca el almacén de datos predeterminado del área de trabajo como almacén de datos de salida y úselo para puntuar las salidas de la canalización.

Para obtener más información sobre el acceso a los datos, consulte [cómo acceder a los datos](https://docs.microsoft.com/azure/machine-learning/how-to-access-data#python-sdk).

```python
from azureml.core.datastore import Datastore

batchscore_blob = Datastore.register_azure_blob_container(ws, 
                      datastore_name="images_datastore", 
                      container_name="sampledata", 
                      account_name="pipelinedata", 
                      overwrite=True)

def_data_store = ws.get_default_datastore()
```

## <a name="create-dataset-objects"></a>Crear objetos de conjunto de datos

Al compilar canalizaciones, se usan objetos `Dataset` para leer datos de los almacenes de datos del área de trabajo, y objetos `PipelineData` para transferir los datos intermedios entre los pasos de la canalización.

> [!Important]
> En el ejemplo de puntuación por lotes de este tutorial solo se usa un paso de canalización. En los casos de uso con varios pasos, el flujo típico incluirá los siguientes:
>
> 1. Use objetos `Dataset` como *entradas* para capturar datos sin procesar, realizar algunas transformaciones y generar un objeto `PipelineData` de *salida*.
>
> 2. Use el `PipelineData` *objeto de salida* del paso anterior como *objeto de entrada*. Repítalo en pasos posteriores.

En este escenario se crean objetos `Dataset` correspondientes a los directorios del almacén de datos para las imágenes de entrada y las etiquetas de clasificación (valores de la prueba de la y). También se crea un objeto `PipelineData` para los datos de salida de puntuación por lotes.

```python
from azureml.core.dataset import Dataset
from azureml.pipeline.core import PipelineData

input_images = Dataset.File.from_files((batchscore_blob, "batchscoring/images/"))
label_ds = Dataset.File.from_files((batchscore_blob, "batchscoring/labels/"))
output_dir = PipelineData(name="scores", 
                          datastore=def_data_store, 
                          output_path_on_compute="batchscoring/results")
```

A continuación, registre los conjuntos de datos en el área de trabajo.

```python

input_images = input_images.register(workspace = ws, name = "input_images")
label_ds = label_ds.register(workspace = ws, name = "label_ds")
```

## <a name="download-and-register-the-model"></a>Descarga y registro del modelo

Descargue el modelo Tensorflow previamente entrenado para usarlo en la puntuación por lotes de una canalización. En primer lugar, cree un directorio local donde almacenar el modelo y, luego, descárguelo y extráigalo.

```python
import os
import tarfile
import urllib.request

if not os.path.isdir("models"):
    os.mkdir("models")
    
response = urllib.request.urlretrieve("http://download.tensorflow.org/models/inception_v3_2016_08_28.tar.gz", "model.tar.gz")
tar = tarfile.open("model.tar.gz", "r:gz")
tar.extractall("models")
```

A continuación registrará el modelo en el área de trabajo para poder recuperarlo fácilmente en el proceso de canalización. En la función estática `register()`, el parámetro `model_name` es la clave que se usa para buscar el modelo en todo el SDK.

```python
from azureml.core.model import Model
 
model = Model.register(model_path="models/inception_v3.ckpt",
                       model_name="inception",
                       tags={"pretrained": "inception"},
                       description="Imagenet trained tensorflow inception",
                       workspace=ws)
```

## <a name="create-and-attach-the-remote-compute-target"></a>Creación y asociación del destino de proceso remoto

Las canalizaciones de aprendizaje automático no se pueden ejecutar en un entorno local, por lo que debe ejecutarlas en recursos en la nube o *destinos de proceso remotos*. Un destino de proceso remoto es un entorno de proceso virtual reutilizable en el que se ejecutan los experimentos y los flujos de trabajo de aprendizaje automático. 

Ejecute el código siguiente para crear un destino [`AmlCompute`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py) habilitado para GPU y asócielo al área de trabajo. Consulte el [artículo conceptual](https://docs.microsoft.com/azure/machine-learning/concept-compute-target) para más información sobre los destinos de proceso.


```python
from azureml.core.compute import AmlCompute, ComputeTarget
from azureml.exceptions import ComputeTargetException
compute_name = "gpu-cluster"

# checks to see if compute target already exists in workspace, else create it
try:
    compute_target = ComputeTarget(workspace=ws, name=compute_name)
except ComputeTargetException:
    config = AmlCompute.provisioning_configuration(vm_size="STANDARD_NC6",
                                                   vm_priority="lowpriority", 
                                                   min_nodes=0, 
                                                   max_nodes=1)

    compute_target = ComputeTarget.create(workspace=ws, name=compute_name, provisioning_configuration=config)
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

## <a name="write-a-scoring-script"></a>Escritura de un script de puntuación

Para realizar la puntuación, cree un script de puntuación por lotes llamado `batch_scoring.py` y escríbalo en el directorio actual. El script toma imágenes de entrada, aplica el modelo de clasificación y envía las predicciones a un archivo de resultados.

El script `batch_scoring.py` toma los siguientes parámetros, que se pasan desde el paso `ParallelRunStep` que se crea más adelante:

- `--model_name`: nombre del modelo que se usa.
- `--labels_dir`: ubicación del archivo `labels.txt`.

La infraestructura de la canalización usa la clase `ArgumentParser` para pasar los parámetros a los pasos de canalización. Por ejemplo, en el código siguiente, al primer argumento `--model_name` se le da el identificador de propiedad `model_name`. En la función `init()` se utiliza `Model.get_model_path(args.model_name)` para acceder a esta propiedad.


```python
%%writefile batch_scoring.py

import os
import argparse
import datetime
import time
import tensorflow as tf
from math import ceil
import numpy as np
import shutil
from tensorflow.contrib.slim.python.slim.nets import inception_v3

from azureml.core import Run
from azureml.core.model import Model
from azureml.core.dataset import Dataset

slim = tf.contrib.slim

image_size = 299
num_channel = 3


def get_class_label_dict(labels_dir):
    label = []
    labels_path = os.path.join(labels_dir, 'labels.txt')
    proto_as_ascii_lines = tf.gfile.GFile(labels_path).readlines()
    for l in proto_as_ascii_lines:
        label.append(l.rstrip())
    return label


def init():
    global g_tf_sess, probabilities, label_dict, input_images

    parser = argparse.ArgumentParser(description="Start a tensorflow model serving")
    parser.add_argument('--model_name', dest="model_name", required=True)
    parser.add_argument('--labels_dir', dest="labels_dir", required=True)
    args, _ = parser.parse_known_args()

    label_dict = get_class_label_dict(args.labels_dir)
    classes_num = len(label_dict)

    with slim.arg_scope(inception_v3.inception_v3_arg_scope()):
        input_images = tf.placeholder(tf.float32, [1, image_size, image_size, num_channel])
        logits, _ = inception_v3.inception_v3(input_images,
                                              num_classes=classes_num,
                                              is_training=False)
        probabilities = tf.argmax(logits, 1)

    config = tf.ConfigProto()
    config.gpu_options.allow_growth = True
    g_tf_sess = tf.Session(config=config)
    g_tf_sess.run(tf.global_variables_initializer())
    g_tf_sess.run(tf.local_variables_initializer())

    model_path = Model.get_model_path(args.model_name)
    saver = tf.train.Saver()
    saver.restore(g_tf_sess, model_path)


def file_to_tensor(file_path):
    image_string = tf.read_file(file_path)
    image = tf.image.decode_image(image_string, channels=3)

    image.set_shape([None, None, None])
    image = tf.image.resize_images(image, [image_size, image_size])
    image = tf.divide(tf.subtract(image, [0]), [255])
    image.set_shape([image_size, image_size, num_channel])
    return image


def run(mini_batch):
    result_list = []
    for file_path in mini_batch:
        test_image = file_to_tensor(file_path)
        out = g_tf_sess.run(test_image)
        result = g_tf_sess.run(probabilities, feed_dict={input_images: [out]})
        result_list.append(os.path.basename(file_path) + ": " + label_dict[result[0]])
    return result_list
```

> [!TIP]
> La canalización de este tutorial solo tiene un paso y escribe la salida en un archivo. En el caso de las canalizaciones de varios pasos, también se usa `ArgumentParser` para definir un directorio en el que escribir los datos de salida para la entrada en pasos posteriores. Consulte el [cuaderno`ArgumentParser` para ver un ejemplo de cómo pasar datos entre varios pasos de la canalización mediante el modelo de diseño ](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb).

## <a name="build-the-pipeline"></a>Creación de la canalización

Antes de ejecutar la canalización, cree un objeto que defina el entorno de Python y cree las dependencias que necesita el script `batch_scoring.py`. La dependencia principal requerida es Tensorflow, pero también se instalan `azureml-core` y `azureml-dataprep[fuse]`, que se necesitan en ParallelRunStep. Además, especifique la compatibilidad con Docker y Docker-GPU.

```python
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_GPU_IMAGE

cd = CondaDependencies.create(pip_packages=["tensorflow-gpu==1.15.2",
                                            "azureml-core", "azureml-dataprep[fuse]"])
env = Environment(name="parallelenv")
env.python.conda_dependencies = cd
env.docker.base_image = DEFAULT_GPU_IMAGE
```

### <a name="create-the-configuration-to-wrap-the-script"></a>Creación de la configuración para encapsular el script

Cree el paso de canalización mediante el script, la configuración del entorno y los parámetros. Especifique el destino de proceso que ya adjuntó a su área de trabajo.

```python
from azureml.pipeline.steps import ParallelRunConfig

parallel_run_config = ParallelRunConfig(
    environment=env,
    entry_script="batch_scoring.py",
    source_directory=".",
    output_action="append_row",
    mini_batch_size="20",
    error_threshold=1,
    compute_target=compute_target,
    process_count_per_node=2,
    node_count=1
)
```

### <a name="create-the-pipeline-step"></a>Creación del paso de canalización

Un paso de canalización es un objeto que encapsula todo lo que necesita para ejecutar una canalización, por ejemplo:

* Configuración del entorno y las dependencias
* El recurso de proceso en el que se ejecutará la canalización
* Los datos de entrada y salida, y cualquier parámetro personalizado
* Referencia a un script o a una lógica del SDK que se ejecutará durante el paso

Varias clases heredan de la clase primaria [`PipelineStep`](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep?view=azure-ml-py). Puede elegir clases para usar marcos o pilas concretos para compilar un paso. En este ejemplo, se usa la clase `ParallelRunStep` para definir la lógica del paso con un script de Python personalizado. Si un argumento del script es una entrada al paso o una salida del paso, se debe definir el argumento *tanto* en la matriz `arguments`, *como* en el parámetro `input` o `output`, respectivamente. 

En escenarios con más de un paso, las referencias a un objeto de la matriz `outputs` estarán disponibles como *entrada* en un paso posterior de la canalización.

```python
from azureml.pipeline.steps import ParallelRunStep
from datetime import datetime

parallel_step_name = "batchscoring-" + datetime.now().strftime("%Y%m%d%H%M")

label_config = label_ds.as_named_input("labels_input")

batch_score_step = ParallelRunStep(
    name=parallel_step_name,
    inputs=[input_images.as_named_input("input_images")],
    output=output_dir,
    arguments=["--model_name", "inception",
               "--labels_dir", label_config],
    side_inputs=[label_config],
    parallel_run_config=parallel_run_config,
    allow_reuse=False
)
```

Para ver una lista de todas las clases que se pueden usar para los diferentes tipos de pasos, consulte el [paquete de pasos](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps?view=azure-ml-py).

## <a name="submit-the-pipeline"></a>Enviar la canalización

Ahora ejecute la canalización. En primer lugar, cree un objeto `Pipeline` con la referencia al área de trabajo y el paso de canalización que creó. El parámetro `steps` es una matriz de pasos. En este caso, la puntuación por lotes consta de un solo paso. Para compilar canalizaciones con varios pasos, colóquelos en orden en esta matriz.

Luego, use la función `Experiment.submit()` para enviar la canalización para su ejecución. La función `wait_for_completion` genera registros durante el proceso de compilación de la canalización que puede usar para ver el progreso.

> [!IMPORTANT]
> La primera ejecución de canalización tarda aproximadamente *15 minutos*. Se deben descargar todas las dependencias, se crea una imagen de Docker y se aprovisiona y se crea el entorno de Python. La repetición de la ejecución de la canalización tarda mucho menos, ya que esos recursos se reutilizan en lugar de crearse. Sin embargo, el tiempo de ejecución total de la canalización depende de la carga de trabajo de los scripts y de los procesos que se ejecutan en cada paso de la canalización.

```python
from azureml.core import Experiment
from azureml.pipeline.core import Pipeline

pipeline = Pipeline(workspace=ws, steps=[batch_score_step])
pipeline_run = Experiment(ws, 'batch_scoring').submit(pipeline)
pipeline_run.wait_for_completion(show_output=True)
```

### <a name="download-and-review-output"></a>Descarga y revisión de la salida

Ejecute el siguiente código para descargar el archivo de salida creado a partir del script `batch_scoring.py` y, luego, explore los resultados de la puntuación.

```python
import pandas as pd

batch_run = next(pipeline_run.get_children())
batch_output = batch_run.get_output_data("scores")
batch_output.download(local_path="inception_results")

for root, dirs, files in os.walk("inception_results"):
    for file in files:
        if file.endswith("parallel_run_step.txt"):
            result_file = os.path.join(root, file)

df = pd.read_csv(result_file, delimiter=":", header=None)
df.columns = ["Filename", "Prediction"]
print("Prediction has ", df.shape[0], " rows")
df.head(10)
```

## <a name="publish-and-run-from-a-rest-endpoint"></a>Publicación y ejecución desde un punto de conexión REST

Ejecute el código siguiente para publicar la canalización en el área de trabajo. En el área de trabajo de Azure Machine Learning Studio puede ver los metadatos de la canalización, incluidos el historial de ejecución y las duraciones. También puede ejecutar la canalización manualmente desde Studio.

La publicación de la canalización permite a un punto de conexión REST volver a ejecutar la canalización desde cualquier biblioteca HTTP en cualquier plataforma.

```python
published_pipeline = pipeline_run.publish_pipeline(
    name="Inception_v3_scoring", description="Batch scoring using Inception v3 model", version="1.0")

published_pipeline
```

Para ejecutar la canalización desde el punto de conexión REST, necesita un encabezado de autenticación de tipo portador de OAuth2. En el ejemplo siguiente se usa la autenticación interactiva, con fines ilustrativos; en la mayoría de los escenarios de producción que requieren autenticación automatizada o desatendida, use la autenticación de la entidad de servicio tal y como se [describe en este artículo](how-to-setup-authentication.md).

La autenticación de la entidad de servicio implica la creación de un *Registro de aplicación* en *Azure Active Directory*. Primero genere un secreto de cliente y, a continuación, conceda a la entidad de servicio el *acceso de rol* al área de trabajo de aprendizaje automático. Use la clase [`ServicePrincipalAuthentication`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.authentication.serviceprincipalauthentication?view=azure-ml-py) para administrar el flujo de autenticación. 

Tanto [`InteractiveLoginAuthentication`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.authentication.interactiveloginauthentication?view=azure-ml-py) como `ServicePrincipalAuthentication` heredan de `AbstractAuthentication`. En ambos caso, use la función [`get_authentication_header()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.authentication.abstractauthentication?view=azure-ml-py#get-authentication-header--) de la misma manera para capturar el encabezado:

```python
from azureml.core.authentication import InteractiveLoginAuthentication

interactive_auth = InteractiveLoginAuthentication()
auth_header = interactive_auth.get_authentication_header()
```

Obtenga la dirección URL de REST de la propiedad `endpoint` del objeto de canalización publicado. También puede encontrar la dirección URL de REST en el área de trabajo de Azure Machine Learning Studio. 

Compile una solicitud HTTP POST para el punto de conexión. Especifique el encabezado de autenticación en la solicitud. Agregue un objeto de carga JSON con el nombre del experimento.

Realice la solicitud para desencadenar la ejecución. Incluya código para acceder a la clave `Id` desde el diccionario de respuestas para obtener el valor del identificador de ejecución.

```python
import requests

rest_endpoint = published_pipeline.endpoint
response = requests.post(rest_endpoint, 
                         headers=auth_header, 
                         json={"ExperimentName": "batch_scoring",
                               "ParameterAssignments": {"process_count_per_node": 6}})
run_id = response.json()["Id"]
```

Use el identificador de ejecución para supervisar el estado de la nueva ejecución. La nueva ejecución tardará otros 10-15 minutos 

y su aspecto será similar al de la canalización que se ejecutó anteriormente en el tutorial. Puede elegir no ver la salida completa.

```python
from azureml.pipeline.core.run import PipelineRun
from azureml.widgets import RunDetails

published_pipeline_run = PipelineRun(ws.experiments["batch_scoring"], run_id)
RunDetails(published_pipeline_run).show()
```

## <a name="clean-up-resources"></a>Limpieza de recursos

No complete esta sección si planea ejecutar otros tutoriales de Azure Machine Learning.

### <a name="stop-the-compute-instance"></a>Detención de la instancia de proceso

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]

### <a name="delete-everything"></a>Eliminar todo el contenido

Si no va a usar los recursos creados, elimínelos para no incurrir en cargos:

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
1. En la lista de grupos de recursos, seleccione el que ha creado.
1. Seleccione **Eliminar grupo de recursos**.
1. Escriba el nombre del grupo de recursos. A continuación, seleccione **Eliminar**.

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial de canalizaciones de aprendizaje automático, realizó las siguientes tareas:

> [!div class="checklist"]
> * Compiló una canalización con dependencias del entorno para ejecutarse en un recurso de proceso de GPU remota.
> * Creó un script de puntuación para ejecutar predicciones por lotes con un modelo Tensorflow entrenado previamente.
> * Publicó una canalización y la habilitó para ejecutarla desde un punto de conexión REST.

Para más ejemplos de cómo crear canalizaciones mediante el SDK de aprendizaje automático, consulte el [repositorio de cuadernos](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines).
