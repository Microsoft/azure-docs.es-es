---
title: Crear, ejecutar y realizar un seguimiento de las canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Cree y ejecute una canalización de aprendizaje automático con el SDK de Azure Machine Learning para Python. Use canalizaciones de ML para crear y administrar flujos de trabajo que unen las fases de Machine Learning (ML). Estas fases incluyen la preparación de los datos, el entrenamiento del modelo, la implementación de modelo y la inferencia o puntuación.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: nilsp
author: NilsPohlmann
ms.date: 12/05/2019
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 0a8bb3ff3d1fc36d4213c6d1a8ea402833bd915e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852946"
---
# <a name="create-and-run-machine-learning-pipelines-with-azure-machine-learning-sdk"></a>Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a crear, publicar, ejecutar y realizar un seguimiento de una [canalización de aprendizaje automático](concept-ml-pipelines.md) mediante el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).  Use **canalizaciones de Machine Learning** para crear un flujo de trabajo que reúna varias fases de Machine Learning y, luego, publique esa canalización en el área de trabajo de Azure Machine Learning para acceder a ella más adelante o compartirla con los demás.  Las canalizaciones de Machine Learning son ideales para puntuar escenarios por lotes, para usar varios procesos, para reutilizar pasos en lugar de volver a ejecutarlos y para compartir flujos de trabajo de Machine Learning con otros usuarios.

Si bien se puede usar un tipo distinto de canalización, denominado [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/targets/azure-machine-learning?context=azure%2Fmachine-learning%2Fservice%2Fcontext%2Fml-context&view=azure-devops&tabs=yaml), para la automatización de CI/CD de tareas de Machine Learning, este tipo de canalización nunca se almacena en el área de trabajo. [Compare ambas canalizaciones](concept-ml-pipelines.md#which-azure-pipeline-technology-should-i-use).

Cada fase de una canalización de Machine Learning, como la preparación de datos y el entrenamiento de modelos, puede incluir uno o más pasos.

Las canalizaciones de Machine Learning que cree serán visibles para los miembros de su [área de trabajo](how-to-manage-workspace.md) de Azure Machine Learning. 

Las canalizaciones de Machine Learning usan destinos de proceso remotos para el cálculo y el almacenamiento de los datos intermedios y finales asociados a esa canalización. Asimismo, pueden leer y escribir datos en y desde las ubicaciones de[Azure Storage](https://docs.microsoft.com/azure/storage/).

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

Cargue archivos de datos o directorios en el almacén de datos para que pueda obtener acceso a ellos desde sus canalizaciones. En este ejemplo se usa Blob Storage como almacén de datos:

```python
def_blob_store.upload_files(
    ["iris.csv"],
    target_path="train-dataset",
    overwrite=True)
```

Una canalización consta de uno o varios pasos. Un paso es una unidad que se ejecuta en un destino de proceso. Los pasos pueden consumir orígenes de datos y producir datos "intermedios". Igualmente, un paso puede crear datos como un modelo, un directorio con archivos dependientes o datos temporales. Estos datos están disponibles para otros pasos más adelante en la canalización.

Para más información sobre cómo conectar la canalización a los datos, consulte los artículos [Cómo tener acceso a los datos](how-to-access-data.md) y [Cómo registrar conjuntos de datos](how-to-create-register-datasets.md). 

### <a name="configure-data-using-dataset-and-pipelinedata-objects"></a>Configuración de datos mediante los objetos `Dataset` y `PipelineData`

Acaba de crear un origen de datos al que se puede hacer referencia en una canalización como una entrada a un paso. La mejor manera de proporcionar datos a una canalización es un objeto [Dataset](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.Dataset). El objeto `Dataset` apunta a datos que están en un almacén de datos o que son accesibles desde este o en una dirección URL web. La clase `Dataset` es abstracta, así que creará una instancia de un objeto `FileDataset` (que hace referencia a uno o varios archivos) o de un objeto `TabularDataset` que se crea a partir de uno o varios archivos con columnas delimitadas de datos.

Los objetos de `Dataset` admiten control de versiones, diferencias y estadísticas de resumen. Los elementos de `Dataset` se evalúan de forma diferida (como los generadores de Python) y resulta eficaz crear subconjuntos mediante la división o el filtrado. 

Creará un objeto `Dataset` con métodos como [from_file](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory?view=azure-ml-py#from-files-path--validate-true-) o [from_delimited_files](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory?view=azure-ml-py#from-delimited-files-path--validate-true--include-path-false--infer-column-types-true--set-column-types-none--separator------header-true--partition-format-none--support-multi-line-false-).

```python
from azureml.core import Dataset

iris_tabular_dataset = Dataset.Tabular.from_delimited_files([(def_blob_store, 'train-dataset/iris.csv')])
```

Los datos intermedios (o salida de un paso) se representan mediante un objeto [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py). `output_data1` se crea como la salida de un paso y se usa como la entrada de uno o más pasos futuros. `PipelineData` introduce una dependencia de datos entre los pasos y crea un orden de ejecución implícito en la canalización. Este objeto se usará más adelante al crear los pasos de la canalización.

```python
from azureml.pipeline.core import PipelineData

output_data1 = PipelineData(
    "output_data1",
    datastore=def_blob_store,
    output_name="output_data1")
```

Puede encontrar más información y código de ejemplo para trabajar con conjuntos de datos y datos de canalización en [Movimiento de datos a los pasos de canalización de ML (Python) o entre ellos](how-to-move-data-in-out-of-pipelines.md).

## <a name="set-up-a-compute-target"></a>Configuración de un destino de proceso

En Azure Machine Learning, el término __proceso__ (o __destino de proceso__) se refiere a las máquinas o clústeres que realizarán los pasos del cálculo en su canal de aprendizaje automático.   Consulte los [destinos de proceso del entrenamiento del modelo](how-to-set-up-training-targets.md) para obtener una lista completa de destinos de proceso y cómo crearlos y adjuntarlos a su área de trabajo.  El proceso para crear o adjuntar un destino de proceso es el mismo independientemente de si entrena un modelo o ejecuta un paso de la canalización. Después de crear y adjuntar el destino de proceso, utilice el objeto `ComputeTarget` en su [paso de canalización](#steps).

> [!IMPORTANT]
> No se admite la realización de operaciones de administración en destinos de proceso desde dentro de trabajos remotos. Puesto que las canalizaciones de aprendizaje automático se envían como un trabajo remoto, no use operaciones de administración en destinos de proceso desde dentro de la canalización.

A continuación se muestran ejemplos de creación y asociación de los destinos de proceso para:

* Proceso de Azure Machine Learning
* Azure Databricks 
* Análisis con Azure Data Lake

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="azure-machine-learning-compute"></a>Proceso de Azure Machine Learning

Puede crear un proceso de Azure Machine Learning para ejecutar sus pasos.

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

### <a name="azure-databricks"></a><a id="databricks"></a>Azure Databricks

Azure Databricks es un entorno basado en Apache Spark de la nube de Azure. Se puede usar como destino de proceso con una canalización de Azure Machine Learning.

Cree un área de trabajo de Azure Databricks antes de usarlo. Para crear un recurso de área de trabajo, consulte el documento [Ejecución de un trabajo de Spark en Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal).

Para adjuntar Azure Databricks como destino de proceso, proporcione la información siguiente:

* __Nombre del proceso de Databricks__: el nombre que desea asignar a este recurso de proceso.
* __Nombre de área de trabajo de Databricks__: el nombre del área de trabajo de Azure Databricks.
* __Token de acceso de Databricks__: el token de acceso usado para autenticarse en Azure Databricks. Para generar un token de acceso, consulte el documento [Autenticación](https://docs.azuredatabricks.net/dev-tools/api/latest/authentication.html).

En el código siguiente se muestra cómo adjuntar Azure Databricks como un destino de proceso con el SDK de Azure Machine Learning (__el área de trabajo de Databricks debe estar presente en la misma suscripción que el área de trabajo de AML__):

```python
import os
from azureml.core.compute import ComputeTarget, DatabricksCompute
from azureml.exceptions import ComputeTargetException

databricks_compute_name = os.environ.get(
    "AML_DATABRICKS_COMPUTE_NAME", "<databricks_compute_name>")
databricks_workspace_name = os.environ.get(
    "AML_DATABRICKS_WORKSPACE", "<databricks_workspace_name>")
databricks_resource_group = os.environ.get(
    "AML_DATABRICKS_RESOURCE_GROUP", "<databricks_resource_group>")
databricks_access_token = os.environ.get(
    "AML_DATABRICKS_ACCESS_TOKEN", "<databricks_access_token>")

try:
    databricks_compute = ComputeTarget(
        workspace=ws, name=databricks_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('databricks_compute_name {}'.format(databricks_compute_name))
    print('databricks_workspace_name {}'.format(databricks_workspace_name))
    print('databricks_access_token {}'.format(databricks_access_token))

    # Create attach config
    attach_config = DatabricksCompute.attach_configuration(resource_group=databricks_resource_group,
                                                           workspace_name=databricks_workspace_name,
                                                           access_token=databricks_access_token)
    databricks_compute = ComputeTarget.attach(
        ws,
        databricks_compute_name,
        attach_config
    )

    databricks_compute.wait_for_completion(True)
```

Para consultar un ejemplo más detallado, vea un [cuaderno de ejemplo](https://aka.ms/pl-databricks) en GitHub.

### <a name="azure-data-lake-analytics"></a><a id="adla"></a>Azure Data Lake Analytics

Azure Data Lake Analytics es una plataforma de análisis de macrodatos de la nube de Azure. Se puede usar como destino de proceso con una canalización de Azure Machine Learning.

Cree una cuenta de Azure Data Lake Analytics antes de usarlo. Para crear este recurso, consulte la [Introducción a Azure Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-get-started-portal) documento.

Para asociar Data Lake Analytics como destino de proceso, debe usar el SDK de Azure Machine Learning y proporcionar la siguiente información:

* __Nombre de proceso__: el nombre que desea asignar a este recurso de proceso.
* __Grupo de recursos__: el grupo de recursos que contiene la cuenta de Data Lake Analytics.
* __Nombre de cuenta__: El nombre de la cuenta de Data Lake Analytics.

El código siguiente muestra cómo asociar Data Lake Analytics como destino de proceso:

```python
import os
from azureml.core.compute import ComputeTarget, AdlaCompute
from azureml.exceptions import ComputeTargetException


adla_compute_name = os.environ.get(
    "AML_ADLA_COMPUTE_NAME", "<adla_compute_name>")
adla_resource_group = os.environ.get(
    "AML_ADLA_RESOURCE_GROUP", "<adla_resource_group>")
adla_account_name = os.environ.get(
    "AML_ADLA_ACCOUNT_NAME", "<adla_account_name>")

try:
    adla_compute = ComputeTarget(workspace=ws, name=adla_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('adla_compute_name {}'.format(adla_compute_name))
    print('adla_resource_id {}'.format(adla_resource_group))
    print('adla_account_name {}'.format(adla_account_name))
    # create attach config
    attach_config = AdlaCompute.attach_configuration(resource_group=adla_resource_group,
                                                     account_name=adla_account_name)
    # Attach ADLA
    adla_compute = ComputeTarget.attach(
        ws,
        adla_compute_name,
        attach_config
    )

    adla_compute.wait_for_completion(True)
```

Para consultar un ejemplo más detallado, vea un [cuaderno de ejemplo](https://aka.ms/pl-adla) en GitHub.

> [!TIP]
> Las canalizaciones de Azure Machine Learning solo pueden trabajar con datos almacenados en el almacén de datos predeterminado de la cuenta de Data Lake Analytics. Si los datos con los que necesita trabajar están en un almacén no predeterminado, puede usar [`DataTransferStep`](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep?view=azure-ml-py) para copiar los datos antes del entrenamiento.

## <a name="construct-your-pipeline-steps"></a><a id="steps"></a>Construir los pasos de la canalización

Después de crear y adjuntar un destino de proceso al área de trabajo, está listo para definir un paso de la canalización. Hay muchos pasos integrados disponibles a través del SDK de Azure Machine Learning. El más básico de estos pasos es [PythonScriptStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?view=azure-ml-py), que ejecuta un script de Python en un destino de proceso específico.

```python
from azureml.pipeline.steps import PythonScriptStep

ds_input = my_dataset.as_named_input('input1')

trainStep = PythonScriptStep(
    script_name="train.py",
    arguments=["--input", ds_input.as_download(), "--output", output_data1],
    inputs=[ds_input],
    outputs=[output_data1],
    compute_target=compute_target,
    source_directory=project_folder,
    allow_reuse=True
)
```

La reutilización de los resultados anteriores (`allow_reuse`) es clave cuando se usan canalizaciones en un entorno de colaboración, ya que eliminar las repeticiones innecesarias ofrece agilidad. La reutilización es el comportamiento predeterminado cuando el nombre del script, las entradas y los parámetros de un paso siguen siendo los mismos. Cuando se reutiliza la salida del paso indicado, el trabajo no se envía al proceso; en cambio, los resultados de la ejecución anterior están disponibles de inmediato para la ejecución del siguiente paso. Si `allow_reuse` se establece en "false", siempre se generará una nueva ejecución para este paso durante la ejecución de la canalización. 

Después de definir sus pasos, debe compilar la canalización mediante algunos o todos ellos.

> [!NOTE]
> No se carga ningún archivo o dato en Azure Machine Learning cuando define los pasos o compila la canalización.

```python
# list of steps to run
compareModels = [trainStep, extractStep, compareStep]

from azureml.pipeline.core import Pipeline

# Build the pipeline
pipeline1 = Pipeline(workspace=ws, steps=[compareModels])
```

En el ejemplo siguiente se usa el destino de proceso de Azure Databricks que creó anteriormente: 

```python
from azureml.pipeline.steps import DatabricksStep

dbStep = DatabricksStep(
    name="databricksmodule",
    inputs=[step_1_input],
    outputs=[step_1_output],
    num_workers=1,
    notebook_path=notebook_path,
    notebook_params={'myparam': 'testparam'},
    run_name='demo run name',
    compute_target=databricks_compute,
    allow_reuse=False
)
# List of steps to run
steps = [dbStep]

# Build the pipeline
pipeline1 = Pipeline(workspace=ws, steps=steps)
```

### <a name="use-a-dataset"></a>Uso de un conjunto de datos 

Los conjuntos de datos creados desde Azure Blob Storage, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database y Azure Database for PostgreSQL se pueden usar como entrada en cualquier paso de una canalización. Puede escribir la salida en [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?view=azure-ml-py), [DatabricksStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep?view=azure-ml-py) o bien, si quiere escribir los datos en un almacén de datos específico, use [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py). 

> [!IMPORTANT]
> La escritura de datos de salida en un almacén de datos con PipelineData solo se admite para los almacenes de datos de Azure Blob y del recurso compartido de archivos de Azure. Esta funcionalidad no se admite para los [almacén de datos de generación 2 de ADLS](https://docs.microsoft.com/python/api/azureml-core/azureml.data.azure_data_lake_datastore.azuredatalakegen2datastore?view=azure-ml-py) en este momento.

```python
dataset_consuming_step = PythonScriptStep(
    script_name="iris_train.py",
    inputs=[iris_tabular_dataset.as_named_input("iris_data")],
    compute_target=compute_target,
    source_directory=project_folder
)
```

A continuación, recupere el conjunto de datos de la canalización mediante el diccionario [Run.input_datasets](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py#input-datasets).

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

Para obtener más información, consulte la documentación de referencia de la [clase Experimento](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment.experiment?view=azure-ml-py).

### <a name="view-results-of-a-pipeline"></a>Visualización de los resultados de una canalización

Consulte la lista de todas las canalizaciones y sus detalles de ejecución en Studio:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. [Vea su área de trabajo](how-to-manage-workspace.md#view).

1. A la izquierda, seleccione **Canalizaciones** para ver todas las ejecuciones de la canalización.
 ![lista de canalizaciones de Machine Learning](./media/how-to-create-your-first-pipeline/pipelines.png)
 
1. Seleccione una canalización específica para ver los resultados de la ejecución.

## <a name="git-tracking-and-integration"></a>Integración y seguimiento de Git

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="publish-a-pipeline"></a>Publicar una canalización

Puede publicar una canalización para ejecutarla con diferentes entradas más adelante. Para que el punto de conexión REST de una canalización ya publicada acepte los parámetros, debe parametrizar la canalización antes de publicarla.

1. Para crear un parámetro de canalización, use un objeto [PipelineParameter](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.pipelineparameter?view=azure-ml-py) con un valor predeterminado.

   ```python
   from azureml.pipeline.core.graph import PipelineParameter
   
   pipeline_param = PipelineParameter(
     name="pipeline_arg",
     default_value=10)
   ```

2. Agregue este objeto `PipelineParameter` como parámetro a cualquiera de los pasos de la canalización tal como se muestra a continuación:

   ```python
   compareStep = PythonScriptStep(
     script_name="compare.py",
     arguments=["--comp_data1", comp_data1, "--comp_data2", comp_data2, "--output_data", out_data3, "--param1", pipeline_param],
     inputs=[ comp_data1, comp_data2],
     outputs=[out_data3],
     compute_target=compute_target,
     source_directory=project_folder)
   ```

3. Publique esta canalización para que acepte un parámetro cuando se invoque.

   ```python
   published_pipeline1 = pipeline_run1.publish_pipeline(
        name="My_Published_Pipeline",
        description="My Published Pipeline Description",
        version="1.0")
   ```

### <a name="run-a-published-pipeline"></a>Ejecutar una canalización publicada

Todas las canalizaciones publicadas tienen un punto de conexión REST. Con el punto de conexión de canalización, puede desencadenar una ejecución de la canalización desde cualquier sistema externo, incluidos los clientes que no sean de Python. Este punto de conexión habilita la "repetibilidad administrada" en los escenarios de puntuación y nuevo entrenamiento.

Para invocar la ejecución de la canalización anterior, necesitará un token de encabezado de autenticación de Azure Active Directory, tal como se describe en la referencia a la [clase AzureCliAuthentication](https://docs.microsoft.com/python/api/azureml-core/azureml.core.authentication.azurecliauthentication?view=azure-ml-py) u obtener más detalles en el notebook [Autenticación en Azure Machine Learning](https://aka.ms/pl-restep-auth).

```python
from azureml.pipeline.core import PublishedPipeline
import requests

response = requests.post(published_pipeline1.endpoint,
                         headers=aad_token,
                         json={"ExperimentName": "My_Pipeline",
                               "ParameterAssignments": {"pipeline_arg": 20}})
```

## <a name="create-a-versioned-pipeline-endpoint"></a>Creación de un punto de conexión de canalización con versiones

Puede crear un punto de conexión de canalización con varias canalizaciones publicadas detrás. Puede usarse como una canalización publicada, pero le proporciona un punto de conexión de REST fijo a medida que recorre en iteración y actualiza las canalizaciones de aprendizaje automático.

```python
from azureml.pipeline.core import PipelineEndpoint

published_pipeline = PipelineEndpoint.get(workspace=ws, name="My_Published_Pipeline")
pipeline_endpoint = PipelineEndpoint.publish(workspace=ws, name="PipelineEndpointTest",
                                            pipeline=published_pipeline, description="Test description Notebook")
```

### <a name="submit-a-job-to-a-pipeline-endpoint"></a>Envío de un trabajo a un punto de conexión de canalización

Puede enviar un trabajo a la versión predeterminada de un punto de conexión de canalización:

```python
pipeline_endpoint_by_name = PipelineEndpoint.get(workspace=ws, name="PipelineEndpointTest")
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment")
print(run_id)
```

También puede enviar un trabajo a una versión específica:

```python
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment", pipeline_version="0")
print(run_id)
```

Lo mismo puede hacerse mediante la API REST:

```python
rest_endpoint = pipeline_endpoint_by_name.endpoint
response = requests.post(rest_endpoint, 
                         headers=aad_token, 
                         json={"ExperimentName": "PipelineEndpointExperiment",
                               "RunSource": "API",
                               "ParameterAssignments": {"1": "united", "2":"city"}})
```

### <a name="use-published-pipelines-in-the-studio"></a>Uso de canalizaciones publicadas en Studio

También puede ejecutar una canalización publicada desde Studio:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. [Vea su área de trabajo](how-to-manage-workspace.md#view).

1. En el panel izquierdo, seleccione **Puntos de conexión**.

1. En la parte superior, seleccione **Pipeline endpoints** (Puntos de conexión de la canalización).
 ![lista de las canalizaciones publicadas de Machine Learning](./media/how-to-create-your-first-pipeline/pipeline-endpoints.png)

1. Seleccione una canalización específica para ejecutar, utilizar o revisar los resultados de las ejecuciones anteriores del punto de conexión de la canalización.

### <a name="disable-a-published-pipeline"></a>Deshabilitar una canalización publicada

Para ocultar una canalización de la lista de canalizaciones publicadas hay que deshabilitarla, ya sea en Studio o en el SDK:

```python
# Get the pipeline by using its ID from Azure Machine Learning studio
p = PublishedPipeline.get(ws, id="068f4885-7088-424b-8ce2-eeb9ba5381a6")
p.disable()
```

Puede habilitarla de nuevo con `p.enable()`. Para más información, consulte la referencia de la [clase PublishedPipeline](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.publishedpipeline?view=azure-ml-py).

## <a name="caching--reuse"></a>Almacenamiento en caché y reutilización  

Para optimizar y personalizar el comportamiento de las canalizaciones, puede hacer algunas cosas en relación con el almacenamiento en caché y la reutilización. Por ejemplo, puede hacer lo siguiente:
+ **Desactivar la reutilización predeterminada de la salida de ejecución de pasos** configurando `allow_reuse=False` durante la [definición de pasos](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py). La reutilización es clave cuando se usan canalizaciones en un entorno de colaboración, ya que eliminar las repeticiones innecesarias ofrece agilidad. Sin embargo, puede optar por ignorar esta reutilización.
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

## <a name="next-steps"></a>Pasos siguientes

- Use [estos cuadernos de Jupyter en GitHub](https://aka.ms/aml-pipeline-readme) para explorar aún más canalizaciones de Machine Learning.
- Consulte la ayuda de referencia del SDK para el paquete [azureml-pipelines-core](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py) y el paquete [azureml-pipelines-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py).
- Consulte los [procedimientos](how-to-debug-pipelines.md) para obtener sugerencias sobre la depuración y la solución de problemas de canalizaciones.

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]
