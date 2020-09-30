---
title: Creación y ejecución de canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Cree y ejecute una canalización de aprendizaje automático con el SDK de Azure Machine Learning para Python. Use canalizaciones de ML para crear y administrar flujos de trabajo que unen las fases de Machine Learning (ML). Estas fases incluyen la preparación de los datos, el entrenamiento del modelo, la implementación de modelo y la inferencia o puntuación.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: nilsp
author: NilsPohlmann
ms.date: 8/14/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperfq1
ms.openlocfilehash: 15e1af35def6a3cb6ffaf5df2db53326fba60bc0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883072"
---
# <a name="create-and-run-machine-learning-pipelines-with-azure-machine-learning-sdk"></a>Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning



En este artículo, aprenderá a crear y ejecutar una [canalización de aprendizaje automático](concept-ml-pipelines.md) mediante el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true). Use **canalizaciones de ML** para crear un flujo de trabajo que reúna varias fases de ML. A continuación, publique esa canalización para acceder a ella posteriormente o compartirla con otras personas. Realice un seguimiento de las canalizaciones de ML para ver cómo funciona el modelo en el mundo real y detectar el desfase de datos. Las canalizaciones de Machine Learning son ideales para puntuar escenarios por lotes, para usar varios procesos, para reutilizar pasos en lugar de volver a ejecutarlos y para compartir flujos de trabajo de Machine Learning con otros usuarios.

Si bien se puede usar un tipo distinto de canalización, denominado [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/targets/azure-machine-learning?context=azure%2Fmachine-learning%2Fservice%2Fcontext%2Fml-context&view=azure-devops&tabs=yaml), para la automatización de CI/CD de tareas de Machine Learning, este tipo de canalización nunca se almacena en el área de trabajo. [Compare ambas canalizaciones](concept-ml-pipelines.md#which-azure-pipeline-technology-should-i-use).

Las canalizaciones de Machine Learning que cree serán visibles para los miembros de su [área de trabajo](how-to-manage-workspace.md) de Azure Machine Learning. 

Las canalizaciones de ML se ejecutan en destinos de proceso (consulte [¿Qué son los destinos de proceso en Azure Machine Learning?](https://docs.microsoft.com/azure/machine-learning/concept-compute-target)). Asimismo, las canalizaciones pueden leer y escribir datos en y desde las ubicaciones de[Azure Storage](https://docs.microsoft.com/azure/storage/).

Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

## <a name="prerequisites"></a>Requisitos previos

* Cree un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) que contendrá todos los recursos de la canalización.

* [Configure un entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning o use una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md) con el SDK ya instalado.

Empiece asociando el área de trabajo:

```Python
import azureml.core
from azureml.core import Workspace, Datastore

ws = Workspace.from_config()
```

## <a name="set-up-machine-learning-resources"></a>Configurar los recursos de aprendizaje automático

Cree los recursos necesarios para ejecutar una canalización de Machine Learning:

* Configure un almacén de datos que puede usar para obtener acceso a los datos necesarios en los pasos de la canalización.

* Configure un objeto `Dataset` para que apunte a los datos que se guardan o a los que puede acceder en un almacén de datos. Configure un objeto de `PipelineData` para los datos temporales pasados entre los pasos de una canalización. 

    > [!TIP]
    > Una experiencia mejorada para pasar datos temporales entre los pasos de la canalización se encuentra disponible en la clase de versión preliminar pública, [`OutputFileDatasetConfig`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.outputfiledatasetconfig?view=azure-ml-py&preserve-view=true).  Esta clase es una característica en vista previa (GB) [experimental](https://docs.microsoft.com/python/api/overview/azure/ml/?view=azure-ml-py#&preserve-view=truestable-vs-experimental) y puede cambiar en cualquier momento.

* Configure los [destinos de proceso](concept-azure-machine-learning-architecture.md#compute-targets) en los que se ejecutarán los pasos de su canalización.

### <a name="set-up-a-datastore"></a>Configurar un almacén de datos

Un almacén de datos almacena los datos a los que la canalización puede obtener acceso. Cada área de trabajo tiene un almacén de datos predeterminado. Asimismo, puede registrar almacenes de datos adicionales. 

Cuando crea su área de trabajo, una instancia de [Azure Files](https://docs.microsoft.com/azure/storage/files/storage-files-introduction) y una de [Azure Blob Storage ](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction) se adjuntan al área de trabajo. Se registra un almacén de datos predeterminado para conectarse al almacenamiento de blobs de Azure. Para obtener más información, consulte [Decisión sobre cuándo usar Azure Files, Azure Blobs o Azure Disks](https://docs.microsoft.com/azure/storage/common/storage-decide-blobs-files-disks). 

```python
# Default datastore 
def_data_store = ws.get_default_datastore()

# Get the blob storage associated with the workspace
def_blob_store = Datastore(ws, "workspaceblobstore")

# Get file storage associated with the workspace
def_file_store = Datastore(ws, "workspacefilestore")

```

Los pasos suelen consumir datos y generar datos de salida. Igualmente, un paso puede crear datos como un modelo, un directorio con archivos dependientes o datos temporales. Estos datos están disponibles para otros pasos más adelante en la canalización. Para más información sobre cómo conectar la canalización a los datos, consulte los artículos [Cómo tener acceso a los datos](how-to-access-data.md) y [Cómo registrar conjuntos de datos](how-to-create-register-datasets.md). 

### <a name="configure-data-with-dataset-and-pipelinedata-objects"></a>Configuración de datos con los objetos `Dataset` y `PipelineData`

La mejor manera de proporcionar datos a una canalización es un objeto [Dataset](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.Dataset). El objeto `Dataset` apunta a datos que están en un almacén de datos o que son accesibles desde este o en una dirección URL web. La clase `Dataset` es abstracta, así que creará una instancia de un objeto `FileDataset` (que hace referencia a uno o varios archivos) o de un objeto `TabularDataset` que se crea a partir de uno o varios archivos con columnas delimitadas de datos.


Creará un objeto `Dataset` con métodos como [from_file](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory?view=azure-ml-py#&preserve-view=truefrom-files-path--validate-true-) o [from_delimited_files](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory?view=azure-ml-py#&preserve-view=truefrom-delimited-files-path--validate-true--include-path-false--infer-column-types-true--set-column-types-none--separator------header-true--partition-format-none--support-multi-line-false-).

```python
from azureml.core import Dataset

my_dataset = Dataset.File.from_files([(def_blob_store, 'train-images/')])
```
Los datos intermedios (o salida de un paso) se representan mediante un objeto [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py&preserve-view=true). `output_data1` se crea como la salida de un paso y se usa como la entrada de uno o más pasos futuros. `PipelineData` introduce una dependencia de datos entre los pasos y crea un orden de ejecución implícito en la canalización. Este objeto se usará más adelante al crear los pasos de la canalización.

```python
from azureml.pipeline.core import PipelineData

output_data1 = PipelineData(
    "output_data1",
    datastore=def_blob_store,
    output_name="output_data1")

```

> [!TIP]
> La conservación de datos intermedios entre los pasos de la canalización también se puede realizar con la clase de versión preliminar pública, [`OutputFileDatasetConfig`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.outputfiledatasetconfig?view=azure-ml-py&preserve-view=true). Para obtener un ejemplo de código con la clase `OutputFileDatasetConfig`, consulte cómo [compilar una canalización de ML de dos pasos](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/pipeline-with-datasets/pipeline-for-image-classification.ipynb).

## <a name="set-up-a-compute-target"></a>Configuración de un destino de proceso


En Azure Machine Learning, el término __proceso__ (o __destino de proceso__) se refiere a las máquinas o clústeres que realizarán los pasos del cálculo en su canal de aprendizaje automático.   En el apartado sobre los [destinos de proceso para el entrenamiento de modelos](concept-compute-target.md#train) encontrará una lista completa de destinos de proceso, mientras que en el apartado sobre la [creación de destinos de proceso](how-to-create-attach-compute-sdk.md) obtendrá información sobre cómo crearlos y adjuntarlos a un área de trabajo.   El proceso para crear o adjuntar un destino de proceso es el mismo independientemente de si entrena un modelo o ejecuta un paso de la canalización. Después de crear y adjuntar el destino de proceso, utilice el objeto `ComputeTarget` en su [paso de canalización](#steps).

> [!IMPORTANT]
> No se admite la realización de operaciones de administración en destinos de proceso desde dentro de trabajos remotos. Puesto que las canalizaciones de aprendizaje automático se envían como un trabajo remoto, no use operaciones de administración en destinos de proceso desde dentro de la canalización.

### <a name="azure-machine-learning-compute"></a>Proceso de Azure Machine Learning

Puede crear un proceso de Azure Machine Learning para ejecutar sus pasos. El código de otros destinos de proceso es muy similar, con parámetros ligeramente diferentes, en función del tipo. 

```python
from azureml.core.compute import ComputeTarget, AmlCompute

compute_name = "aml-compute"
vm_size = "STANDARD_NC6"
if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('Found compute target: ' + compute_name)
else:
    print('Creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size=vm_size,  # STANDARD_NC6 is GPU-enabled
                                                                min_nodes=0,
                                                                max_nodes=4)
    # create the compute target
    compute_target = ComputeTarget.create(
        ws, compute_name, provisioning_config)

    # Can poll for a minimum number of nodes and for a specific timeout.
    # If no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(
        show_output=True, min_node_count=None, timeout_in_minutes=20)

    # For a more detailed view of current cluster status, use the 'status' property
    print(compute_target.status.serialize())
```

## <a name="configure-the-training-runs-environment"></a>Configuración del entorno de ejecución de entrenamiento

El siguiente paso consiste en asegurarse de que la ejecución del entrenamiento remoto tiene todas las dependencias necesarias para los pasos de entrenamiento. Las dependencias y el contexto del entorno de ejecución se establecen creando y configurando un objeto `RunConfiguration`. 

```python
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core import Environment 

aml_run_config = RunConfiguration()
# `compute_target` as defined in "Azure Machine Learning compute" section above
aml_run_config.target = compute_target

USE_CURATED_ENV = True
if USE_CURATED_ENV :
    curated_environment = Environment.get(workspace=ws, name="AzureML-Tutorial")
    aml_run_config.environment = curated_environment
else:
    aml_run_config.environment.python.user_managed_dependencies = False
    
    # Add some packages relied on by data prep step
    aml_run_config.environment.python.conda_dependencies = CondaDependencies.create(
        conda_packages=['pandas','scikit-learn'], 
        pip_packages=['azureml-sdk', 'azureml-dataprep[fuse,pandas]'], 
        pin_sdk_version=False)
```

El código anterior muestra dos opciones para administrar las dependencias. Tal como se presenta, con `USE_CURATED_ENV = True`, la configuración se basa en un entorno seleccionado. Los entornos seleccionados están preparados con bibliotecas interdependientes comunes y pueden ser mucho más rápidos para ponerlos en línea. Los entornos seleccionados tienen imágenes de Docker creadas previamente en [Microsoft Container Registry](https://hub.docker.com/publishers/microsoftowner). Para más información, consulte [Entornos mantenidos de Azure Machine Learning](resource-curated-environments.md).

La ruta de acceso que se toma si cambia `USE_CURATED_ENV` a `False` muestra el patrón para establecer explícitamente las dependencias. En ese escenario, se creará una nueva imagen de Docker personalizada y se registrará en Azure Container Registry dentro del grupo de recursos (consulte [Introducción a los registros de contenedores privados de Docker en Azure](https://docs.microsoft.com/azure/container-registry/container-registry-intro)). La creación y el registro de esta imagen pueden tardar unos minutos.

## <a name="construct-your-pipeline-steps"></a><a id="steps"></a>Construir los pasos de la canalización

Una vez creado el recurso de proceso y el entorno, está listo para definir los pasos de la canalización. Hay muchos pasos integrados disponibles a través del SDK de Azure Machine Learning, como puede ver en la [documentación de referencia del paquete de `azureml.pipeline.steps`](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps?view=azure-ml-py&preserve-view=true). La clase más flexible es [PythonScriptStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?view=azure-ml-py&preserve-view=true), que ejecuta un script de Python.

```python
from azureml.pipeline.steps import PythonScriptStep
dataprep_source_dir = "./dataprep_src"
entry_point = "prepare.py"
# `my_dataset` as defined above
ds_input = my_dataset.as_named_input('input1')

# `output_data1`, `compute_target`, `aml_run_config` as defined above
data_prep_step = PythonScriptStep(
    script_name=entry_point,
    source_directory=dataprep_source_dir,
    arguments=["--input", ds_input.as_download(), "--output", output_data1],
    inputs=[ds_input],
    outputs=[output_data1],
    compute_target=compute_target,
    runconfig=aml_run_config,
    allow_reuse=True
)
```

En el código anterior se muestra un paso típico de canalización inicial. El código de preparación de los datos está en un subdirectorio (en este ejemplo, `"prepare.py"` en el directorio `"./dataprep.src"`). Como parte del proceso de creación de la canalización, este directorio se comprime y se carga en el elemento `compute_target` y el paso ejecuta el script especificado como el valor de `script_name`.

Los valores `arguments`, `inputs` y `outputs` especifican las entradas y salidas del paso. En el ejemplo anterior, los datos de línea base son el conjunto de datos `my_dataset`. Los datos correspondientes se descargarán en el recurso de proceso, ya que el código lo especifica como `as_download()`. El script `prepare.py` realiza las tareas de transformación de datos adecuadas para la tarea a mano y envía los datos a `output_data1`, de tipo `PipelineData`. Para más información, consulte [Movimiento de datos a los pasos de canalización de Machine Learning (Python) y entre ellos](how-to-move-data-in-out-of-pipelines.md). 

El paso se ejecutará en el equipo definido por `compute_target`, con la configuración `aml_run_config`. 

La reutilización de los resultados anteriores (`allow_reuse`) es clave cuando se usan canalizaciones en un entorno de colaboración, ya que eliminar las repeticiones innecesarias ofrece agilidad. La reutilización es el comportamiento predeterminado cuando el nombre del script, las entradas y los parámetros de un paso siguen siendo los mismos. Si se permite la reutilización, los resultados de la ejecución anterior se envían inmediatamente al paso siguiente. Si `allow_reuse` se establece en `False`, siempre se generará una nueva ejecución para este paso durante la ejecución de la canalización.

Es posible crear una canalización con un solo paso, pero casi siempre tendrá elegirá dividir el proceso general en varios pasos. Por ejemplo, puede tener pasos para la preparación de los datos, el entrenamiento, la comparación de modelos y la implementación. Por ejemplo, puede imaginarse que, después del elemento `data_prep_step` especificado anteriormente, el paso siguiente sería el entrenamiento:

```python
train_source_dir = "./train_src"
train_entry_point = "train.py"

training_results = PipelineData(name = "training_results", 
                                datastore=def_blob_store,
                                output_name="training_results")

    
train_step = PythonScriptStep(
    script_name=train_entry_point,
    source_directory=train_source_dir,
    arguments=["--prepped_data", output_data1.as_input(), "--training_results", training_results],
    compute_target=compute_target,
    runconfig=aml_run_config,
    allow_reuse=True
)
```

El código anterior es muy similar al del paso de preparación de datos. El código de entrenamiento está en un directorio independiente del código de preparación de datos. La salida de `PipelineData` del paso de preparación de datos, `output_data1`, se usa como _entrada_ al paso de entrenamiento. Un nuevo objeto `PipelineData`, `training_results`, se crea para almacenar los resultados de un paso posterior de comparación o implementación. 


> [!TIP]
> Para mejorar la experiencia del usuario y la capacidad para escribir datos intermedios en el almacén de datos al final de la ejecución de la canalización, use la clase de versión preliminar pública, [`OutputFileDatasetConfig`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.outputfiledatasetconfig?view=azure-ml-py&preserve-view=true). Para obtener ejemplos de código, vea cómo [compilar una canalización de Machine Learning en dos pasos](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/pipeline-with-datasets/pipeline-for-image-classification.ipynb) y [cómo escribir datos en los almacenes de datos tras la finalización de la ejecución](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/scriptrun-with-data-input-output/how-to-use-scriptrun.ipynb).

Después de definir sus pasos, debe compilar la canalización mediante algunos o todos ellos.

> [!NOTE]
> No se carga ningún archivo o dato en Azure Machine Learning cuando define los pasos o compila la canalización. Los archivos se cargan al llamar a [Experiment.submit()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment.experiment?view=azure-ml-py#&preserve-view=truesubmit-config--tags-none----kwargs-).

```python
# list of steps to run (`compare_step` definition not shown)
compare_models = [data_prep_step, train_step, compare_step]

from azureml.pipeline.core import Pipeline

# Build the pipeline
pipeline1 = Pipeline(workspace=ws, steps=[compare_models])
```

### <a name="use-a-dataset"></a>Uso de un conjunto de datos 

Los conjuntos de datos creados desde Azure Blob Storage, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database y Azure Database for PostgreSQL se pueden usar como entrada en cualquier paso de una canalización. Puede escribir la salida en [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?view=azure-ml-py&preserve-view=true), [DatabricksStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep?view=azure-ml-py&preserve-view=true) o bien, si quiere escribir los datos en un almacén de datos específico, use [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py&preserve-view=true). 

> [!IMPORTANT]
> La escritura de datos de salida en un almacén de datos con PipelineData solo se admite para los almacenes de datos de Azure Blob y del recurso compartido de archivos de Azure. 
>
> Para reescribir los datos de salida en los almacenes de datos de Azure Blob, Azure File Share, ADLS Gen 1 y ADLS Gen 2 use la clase de versión preliminar pública, [`OutputFileDatasetConfig`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.output_dataset_config.outputfiledatasetconfig?view=azure-ml-py&preserve-view=true).

```python
dataset_consuming_step = PythonScriptStep(
    script_name="iris_train.py",
    inputs=[iris_tabular_dataset.as_named_input("iris_data")],
    compute_target=compute_target,
    source_directory=project_folder
)
```

A continuación, recupere el conjunto de datos de la canalización mediante el diccionario [Run.input_datasets](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py#&preserve-view=trueinput-datasets).

```python
# iris_train.py
from azureml.core import Run, Dataset

run_context = Run.get_context()
iris_dataset = run_context.input_datasets['iris_data']
dataframe = iris_dataset.to_pandas_dataframe()
```

Merece la pena resaltar la línea `Run.get_context()`. Esta función recupera un objeto `Run` que representa la ejecución experimental actual. En el ejemplo anterior, se usa para recuperar un conjunto de datos registrado. Otro uso común del objeto `Run` es recuperar el propio experimento y el área de trabajo en el que reside el experimento: 

```python
# Within a PythonScriptStep

ws = Run.get_context().experiment.workspace
```

Para más información, incluidas formas alternativas de pasar datos y acceder a ellos, consulte [Movimiento de datos a los pasos de canalización de ML (Python) o entre ellos](how-to-move-data-in-out-of-pipelines.md).

## <a name="caching--reuse"></a>Almacenamiento en caché y reutilización  

Para optimizar y personalizar el comportamiento de las canalizaciones, puede hacer algunas cosas en relación con el almacenamiento en caché y la reutilización. Por ejemplo, puede hacer lo siguiente:
+ **Desactivar la reutilización predeterminada de la salida de ejecución de pasos** configurando `allow_reuse=False` durante la [definición de pasos](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py&preserve-view=true). La reutilización es clave cuando se usan canalizaciones en un entorno de colaboración, ya que eliminar las repeticiones innecesarias ofrece agilidad. Sin embargo, puede optar por ignorar esta reutilización.
+ **Forzar la regeneración de salida de todos los pasos de una ejecución** con `pipeline_run = exp.submit(pipeline, regenerate_outputs=False)`

De forma predeterminada, se habilita `allow_reuse` para los pasos y se crea un hash para el valor de `source_directory` especificados en la definición del paso. Así pues, si el script de un paso determinado permanece igual (`script_name`, entradas y parámetros) y no ha cambiado nada más de ` source_directory`, se reutiliza la salida de un paso anterior ejecutado, el trabajo no se envía al proceso y los resultados de la ejecución anterior están inmediatamente disponibles para el siguiente paso.

```python
step = PythonScriptStep(name="Hello World",
                        script_name="hello_world.py",
                        compute_target=aml_compute,
                        source_directory=source_directory,
                        allow_reuse=False,
                        hash_paths=['hello_world.ipynb'])
```

## <a name="submit-the-pipeline"></a>Enviar la canalización

Cuando envía la canalización, Azure Machine Learning comprueba las dependencias para cada paso y carga una instantánea del directorio de origen especificado. Si no se especifica ningún directorio de origen, se carga el directorio local actual. La instantánea también se almacena como parte del experimento del área de trabajo.

> [!IMPORTANT]
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
>
> Para más información, consulte [Instantánea](concept-azure-machine-learning-architecture.md#snapshots).

```python
from azureml.core import Experiment

# Submit the pipeline to be run
pipeline_run1 = Experiment(ws, 'Compare_Models_Exp').submit(pipeline1)
pipeline_run1.wait_for_completion()
```

Cuando se ejecuta por primera vez una canalización, Azure Machine Learning:

* Descarga la instantánea del proyecto en el destino de proceso de la instancia de Blob Storage asociada al área de trabajo.
* Crea una imagen de docker correspondiente a cada paso en la canalización.
* Descarga la imagen de docker para cada paso en el destino de proceso del registro de contenedor.
* Configura el acceso a los objetos `Dataset` y `PipelineData`. En modo de acceso `as_mount()`, se usa FUSE para proporcionar acceso virtual. Si no se admite el montaje o si el usuario especificó el acceso como `as_download()`, los datos se copian en su lugar en el destino de proceso.

* Ejecuta el paso en el destino de proceso especificado en la definición del paso. 
* Crea artefactos como registros, las propiedades stdout y stderr, métricas y resultados que especifica el paso. Estos artefactos se cargan y se guardan en el almacén de datos predeterminado del usuario.

![Diagrama de ejecución de un experimento como una canalización](./media/how-to-create-your-first-pipeline/run_an_experiment_as_a_pipeline.png)

Para obtener más información, consulte la documentación de referencia de la [clase Experimento](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment.experiment?view=azure-ml-py&preserve-view=true).

## <a name="view-results-of-a-pipeline"></a>Visualización de los resultados de una canalización

Consulte la lista de todas las canalizaciones y sus detalles de ejecución en Studio:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. [Vea su área de trabajo](how-to-manage-workspace.md#view).

1. A la izquierda, seleccione **Canalizaciones** para ver todas las ejecuciones de la canalización.
 ![lista de canalizaciones de Machine Learning](./media/how-to-create-your-first-pipeline/pipelines.png)
 
1. Seleccione una canalización específica para ver los resultados de la ejecución.

### <a name="git-tracking-and-integration"></a>Integración y seguimiento de Git

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="next-steps"></a>Pasos siguientes

- Para compartir su canalización con compañeros o clientes, consulte [Publicación de canalizaciones de aprendizaje automático](how-to-deploy-pipelines.md).
- Use [estos cuadernos de Jupyter Notebook en GitHub](https://aka.ms/aml-pipeline-readme) para explorar en profundidad las canalizaciones de aprendizaje automático.
- Consulte la referencia del SDK para obtener ayuda con el paquete [azureml-pipelines-core](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py&preserve-view=true) y el paquete [azureml-pipelines-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py&preserve-view=true).
- Consulte los [procedimientos](how-to-debug-pipelines.md) para obtener sugerencias sobre la depuración y la solución de problemas de canalizaciones.
- Siga las instrucciones del artículo [Exploración de Azure Machine Learning con cuadernos de Jupyter](samples-notebooks.md) para aprender a ejecutar cuadernos.
