---
title: Movimiento de datos en canalizaciones de Machine Learning
titleSuffix: Azure Machine Learning
description: Obtenga información sobre la entrada y salida de datos en las canalizaciones Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 07/20/2020
ms.topic: conceptual
ms.custom: how-to, contperfq4, devx-track-python
ms.openlocfilehash: 740ca2d991f9447e8a3a04c7795c8a6f3011fd39
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843035"
---
# <a name="moving-data-into-and-between-ml-pipeline-steps-python"></a>Movimiento de datos a los pasos de canalización de Machine Learning (Python) y entre ellos

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo se proporciona código para importar, transformar y mover datos entre los pasos de una canalización de Azure Machine Learning. Para obtener información general sobre cómo funcionan los datos en Azure Machine Learning, consulte [Acceso a los datos en los servicios de Azure Storage](how-to-access-data.md). Para obtener información sobre las ventajas y la estructura de las canalizaciones de Azure Machine Learning, consulte [¿Qué son las canalizaciones de Azure Machine Learning?](concept-ml-pipelines.md).

Este artículo le mostrará cómo realizar los siguientes procedimientos:

- Usar objetos `Dataset` para los datos ya existentes
- Acceder a los datos en los pasos
- Dividir los datos de `Dataset` en subconjuntos, como los subconjuntos de entrenamiento y validación
- Crear objetos `PipelineData` para transferir datos al siguiente paso de la canalización
- Usar objetos `PipelineData` como entrada para los pasos de la canalización
- Crear nuevos objetos `Dataset` a partir de los objetos `PipelineData` que desea conservar

## <a name="prerequisites"></a>Prerrequisitos

Necesitará:

- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

- [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) o acceso a [Azure Machine Learning Studio](https://ml.azure.com/).

- Un área de trabajo de Azure Machine Learning.
  
  [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) o use una existente mediante el SDK de Python. Importe las clases `Workspace` y `Datastore`, y cargue la información de la suscripción desde el archivo `config.json` con la función `from_config()`. De forma predeterminada, esta función busca el archivo JSON en el directorio actual, pero también puede especificar un parámetro de ruta de acceso para que apunte al archivo mediante `from_config(path="your/file/path")`.

   ```python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

- Algunos datos ya existentes. En este artículo se muestra brevemente el uso de un [contenedor de blobs de Azure](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview).

- Opcional: Una canalización de aprendizaje automático existente, como la descrita en [Creación y ejecución de canalizaciones de aprendizaje automático con el SDK de Azure Machine Learning](how-to-create-your-first-pipeline.md).

## <a name="use-dataset-objects-for-pre-existing-data"></a>Uso de objetos `Dataset` para los datos ya existentes 

La mejor manera de ingerir datos en una canalización es utilizar un objeto [Dataset](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py). Los objetos `Dataset` representan datos persistentes disponibles en todo el área de trabajo.

Hay muchas maneras de crear y registrar objetos `Dataset`. Los conjuntos de datos tabulares son para los datos delimitados disponibles en uno o varios archivos. Los conjuntos de datos de archivos son para datos binarios (como imágenes) o para datos que se van a analizar. Las formas más sencillas de crear objetos `Dataset` mediante programación consisten en usar los blobs existentes en el almacenamiento del área de trabajo o en direcciones URL públicas:

```python
datastore = Datastore.get(workspace, 'training_data')
iris_dataset = Dataset.Tabular.from_delimited_files(DataPath(datastore, 'iris.csv'))

cats_dogs_dataset = Dataset.File.from_files(
    paths='https://download.microsoft.com/download/3/E/1/3E1C3F21-ECDB-4869-8368-6DEBA77B919F/kagglecatsanddogs_3367a.zip',
    archive_options=ArchiveOptions(archive_type=ArchiveType.ZIP, entry_glob='**/*.jpg')
)
```

Para más opciones sobre cómo crear conjuntos de datos con distintas opciones y desde distintos orígenes, registrarlos y revisarlos en la interfaz de usuario de Azure Machine Learning, comprender cómo interactúa el tamaño de los datos con la capacidad de proceso y el control de versiones, consulte [Creación de conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md). 

### <a name="pass-datasets-to-your-script"></a>Paso de los conjuntos de datos al script

Para pasar la ruta de acceso del conjunto de datos al script, use el método `as_named_input()` del objeto `Dataset`. Puede pasar el objeto `DatasetConsumptionConfig` resultante al script como argumento o, mediante el uso del argumento `inputs` en el script de la canalización, puede recuperar el conjunto de resultados mediante `Run.get_context().input_datasets[]`.

Una vez que haya creado una entrada con nombre, puede elegir su modo de acceso: `as_mount()` o `as_download()`. Si el script procesa todos los archivos del conjunto de datos y el disco del recurso de proceso es lo suficientemente grande para el conjunto de datos, el modo de acceso de descarga es la mejor opción. El modo de acceso de descarga evitará la sobrecarga de transmitir los datos en tiempo de ejecución. Si el script accede a un subconjunto del conjunto de datos o es demasiado grande para el recurso de proceso, use el modo de acceso de montaje. Para más información, consulte [Montaje frente a descarga](https://docs.microsoft.com/azure/machine-learning/how-to-train-with-datasets#mount-vs-download)

Para pasar un conjunto de datos al paso de la canalización:

1. Use `TabularDataset.as_named_inputs()` o `FileDataset.as_named_input()` (sin "s" al final) para crear un objeto `DatasetConsumptionConfig`.
1. Use `as_mount()` o `as_download()` para establecer el modo de acceso.
1. Pase los conjuntos de datos a los pasos de la canalización mediante `arguments` o el argumento `inputs`.

En el fragmento de código siguiente se muestra el patrón común de combinación de estos pasos dentro del constructor `PythonScriptStep`: 

```python

train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    inputs=[iris_dataset.as_named_inputs('iris').as_mount()]
)
```

También puede usar métodos como `random_split()` y `take_sample()` para crear varias entradas o reducir la cantidad de datos que se pasan al paso de la canalización:

```python
seed = 42 # PRNG seed
smaller_dataset = iris_dataset.take_sample(0.1, seed=seed) # 10%
train, test = smaller_dataset.random_split(percentage=0.8, seed=seed)

train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    inputs=[train.as_named_inputs('train').as_download(), test.as_named_inputs('test').as_download()]
)
```

### <a name="access-datasets-within-your-script"></a>Acceso a los conjuntos de datos del script

Las entradas con nombre para el script del paso de la canalización están disponibles como un diccionario dentro del objeto `Run`. Recupere el objeto `Run` activo mediante `Run.get_context()` y, a continuación, recupere el diccionario de entradas con nombre mediante `input_datasets`. Si pasó el objeto `DatasetConsumptionConfig` con el argumento `arguments` en lugar del argumento `inputs`, acceda a los datos mediante el código `ArgParser`. En el siguiente fragmento de código se muestran ambas técnicas.

```python
# In pipeline definition script:
# Code for demonstration only: It would be very confusing to split datasets between `arguments` and `inputs`
train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    arguments=['--training-folder', train.as_named_inputs('train').as_download()]
    inputs=[test.as_named_inputs('test').as_download()]
)

# In pipeline script
parser = argparse.ArgumentParser()
parser.add_argument('--training-folder', type=str, dest='train_folder', help='training data folder mounting point')
args = parser.parse_args()
training_data_folder = args.train_folder

testing_data_folder = Run.get_context().input_datasets['test']
```

El valor pasado será la ruta de acceso a los archivos del conjunto de datos.

También es posible acceder directamente a un objeto `Dataset` registrado. Dado que los conjuntos de datos registrados son persistentes y se comparten en un área de trabajo, puede recuperarlos directamente:

```python
run = Run.get_context()
ws = run.experiment.workspace
ds = Dataset.get_by_name(workspace=ws, name='mnist_opendataset')
```

## <a name="use-pipelinedata-for-intermediate-data"></a>Uso de `PipelineData` para los datos intermedios

Mientras que los objetos `Dataset` representan datos persistentes, los objetos [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) se utilizan para los datos temporales que son la salida de los pasos de la canalización. Dado que la duración de un objeto `PipelineData` es mayor que un solo paso de la canalización, los definirá en el script de definición de la canalización. Al crear un objeto `PipelineData`, debe proporcionar un nombre y un almacén de datos en el que vayan a residir los datos. Pase los objetos `PipelineData` al elemento `PythonScriptStep` mediante _ambos_ argumentos, `arguments` y `outputs`:

```python
default_datastore = workspace.get_default_datastore()
dataprep_output = PipelineData("clean_data", datastore=default_datastore)

dataprep_step = PythonScriptStep(
    name="prep_data",
    script_name="dataprep.py",
    compute_target=cluster,
    arguments=["--output-path", dataprep_output]
    inputs=[Dataset.get_by_name(workspace, 'raw_data')],
    outputs=[dataprep_output]
)
```

Puede optar por crear el objeto `PipelineData` con un modo de acceso que proporcione una carga inmediata. En ese caso, cuando cree el objeto `PipelineData`, establezca `upload_mode` en `"upload"` y use el argumento `output_path_on_compute` para especificar la ruta de acceso en la que se van a escribir los datos:

```python
PipelineData("clean_data", datastore=def_blob_store, output_mode="upload", output_path_on_compute="clean_data_output/")
```

### <a name="use-pipelinedata-as-outputs-of-a-training-step"></a>Uso de `PipelineData` como salidas de un paso de entrenamiento

En el elemento `PythonScriptStep` de la canalización, puede recuperar las rutas de acceso de salida disponibles mediante los argumentos del programa. Si este paso es el primero y va a inicializar los datos de salida, debe crear el directorio en la ruta de acceso especificada. Después, puede escribir los archivos que desee incluir en el objeto `PipelineData`.

```python
parser = argparse.ArgumentParser()
parser.add_argument('--output_path', dest='output_path', required=True)
args = parser.parse_args()

# Make directory for file
os.makedirs(os.path.dirname(args.output_path), exist_ok=True)
with open(args.output_path, 'w') as f:
    f.write("Step 1's output")
```

Si ha creado el objeto `PipelineData` con el argumento `is_directory` establecido en `True`, bastará con realizar la llamada a `os.makedirs()` y, a continuación, podrá escribir los archivos que desee en la ruta de acceso. Para más información, consulte la documentación de referencia de [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py).

### <a name="read-pipelinedata-as-inputs-to-non-initial-steps"></a>Lectura de `PipelineData` como entradas en pasos no iniciales

Después de que el paso de canalización inicial escriba algunos datos en la ruta de acceso de `PipelineData` y se convierta en una salida de ese paso inicial, se puede usar como entrada para un paso posterior:

```python
step1_output_data = PipelineData("processed_data", datastore=def_blob_store, output_mode="upload")

step1 = PythonScriptStep(
    name="generate_data",
    script_name="step1.py",
    runconfig = aml_run_config,
    arguments = ["--output_path", step1_output_data],
    inputs=[],
    outputs=[step1_output_data]
)

step2 = PythonScriptStep(
    name="read_pipeline_data",
    script_name="step2.py",
    compute_target=compute,
    runconfig = aml_run_config,
    arguments = ["--pd", step1_output_data],
    inputs=[step1_output_data]
)

pipeline = Pipeline(workspace=ws, steps=[step1, step2])
```

El valor de una entrada `PipelineData` es la ruta de acceso a la salida anterior. Si, como se mostró anteriormente, el primer paso escribió un único archivo, consumirlo podría ser algo similar a: 

```python
parser = argparse.ArgumentParser()
parser.add_argument('--pd', dest='pd', required=True)
args = parser.parse_args()

with open(args.pd) as f:
    print(f.read())
```

## <a name="convert-pipelinedata-objects-to-datasets"></a>Conversión de objetos `PipelineData` en objetos `Dataset`

Si desea que el objeto `PipelineData` esté disponible durante más tiempo que la duración de una ejecución, use su función `as_dataset()` para convertirlo en un objeto `Dataset`. Después, puede registrar el objeto `Dataset`, convirtiéndolo en un ciudadano de primera clase del área de trabajo. Dado que el objeto `PipelineData` tendrá una ruta de acceso diferente cada vez que se ejecute la canalización, se recomienda establecer `create_new_version` en `True` al registrar un objeto `Dataset` creado a partir de un objeto `PipelineData`.

```python
step1_output_ds = step1_output_data.as_dataset()
step1_output_ds.register(name="processed_data", create_new_version=True)
```

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md)
* [Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning](how-to-create-your-first-pipeline.md)
