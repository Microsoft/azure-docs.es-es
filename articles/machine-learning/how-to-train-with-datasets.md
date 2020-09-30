---
title: Entrenamiento con azureml-datasets
titleSuffix: Azure Machine Learning
description: Aprenda a usar conjuntos de datos en el entrenamiento.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 7a3f839a676723942af2e669839457ed3246aabd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885885"
---
# <a name="train-with-datasets-in-azure-machine-learning"></a>Entrenamiento con conjuntos de datos en Azure Machine Learning


En este artículo obtendrá información acerca de cómo usar [conjuntos de datos de Azure Machine Learning](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py&preserve-view=true) en sus experimentos de entrenamiento.  Puede usar conjuntos de datos en el destino de proceso local o remoto sin preocuparse por las cadenas de conexión o las rutas de acceso a datos.

Los conjuntos de datos de Azure Machine Learning proporcionan una integración perfecta con productos de entrenamiento de Azure Machine Learning, como [ScriptRun](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrun?view=azure-ml-py&preserve-view=true), [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py&preserve-view=true), [HyperDrive](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive?view=azure-ml-py&preserve-view=true) y [canalizaciones de Azure Machine Learning](how-to-create-your-first-pipeline.md).

## <a name="prerequisites"></a>Prerrequisitos

Para crear conjuntos de datos y entrenar con ellos, necesita:

* Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* El [SDK de Azure Machine Learning para Python instalado](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true), que incluye el paquete azureml-datasets.

> [!Note]
> Algunas clases Dataset tienen dependencias del paquete [azureml dataprep](https://docs.microsoft.com/python/api/azureml-dataprep/?view=azure-ml-py&preserve-view=true). Para los usuarios de Linux, estas clases solo se admiten en las siguientes distribuciones:  Red Hat Enterprise Linux, Ubuntu, Fedora y CentOS.

## <a name="access-and-explore-input-datasets"></a>Acceso y exploración de conjuntos de datos de entrada

Puede acceder a un conjunto de datos TabularDataset existente desde el script de entrenamiento de un experimento en su área de trabajo y cargar ese conjunto de datos en un dataframe de Pandas para realizar más exploraciones en su entorno local.

En el código siguiente se usa el método [`get_context()`]() de la clase [`Run`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py&preserve-view=true) para acceder al conjunto de datos TabularDataset de entrada existente, `titanic`, en el script de entrenamiento. A continuación, se usa el método [`to_pandas_dataframe()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) para cargar ese conjunto de datos en un dataframe de Pandas para la exploración y preparación de datos adicionales antes del entrenamiento.

> [!Note]
> Si el origen de datos original contiene NaN, cadenas vacías o valores en blanco, cuando se utiliza el to_pandas_dataframe(), esos valores se reemplazan como un valor de *Null*. 

```Python
%%writefile $script_folder/train_titanic.py

from azureml.core import Dataset, Run

run = Run.get_context()
# get the input dataset by name
dataset = run.input_datasets['titanic']

# load the TabularDataset to pandas DataFrame
df = dataset.to_pandas_dataframe()
```

Si necesita cargar los datos preparados en un nuevo conjunto de datos a partir de un dataframe de Pandas en memoria, escriba los datos en un archivo local, como un parquet, y cree el nuevo conjunto de datos a partir de ese archivo. También puede crear conjuntos de datos a partir de archivos o rutas de acceso locales en almacenes de datos. Más información sobre [cómo crear conjuntos de datos](how-to-create-register-datasets.md).

## <a name="use-datasets-directly-in-training-scripts"></a>Usar los conjuntos de datos directamente en los scripts de entrenamiento

Si tiene datos estructurados que aún no se han registrado como un conjunto de datos, cree un conjunto de datos TabularDataset y úselo directamente en el script de entrenamiento para su experimento local o remoto.

En este ejemplo, creará un objeto [TabularDataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py&preserve-view=true) no registrado que usará como entrada directa para el objeto `estimator` para entrenamiento. Si desea volver a usar este conjunto de datos TabularDataset con otros experimentos en su área de trabajo, consulte [Registro de conjuntos de registros en su área de trabajo](how-to-create-register-datasets.md#register-datasets).

### <a name="create-a-tabulardataset"></a>Creación de un objeto TabularDataset

En el código siguiente se crea un objeto TabularDataset no registrado a partir de una dirección URL web.  

```Python
from azureml.core.dataset import Dataset

web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path)
```

Los objetos TabularDataset ofrecen la posibilidad de cargar los datos de su conjunto de datos TabularDataset en un DataFrame de Pandas o Spark para que pueda trabajar con bibliotecas conocidas de entrenamiento y preparación de datos sin tener que salir del cuaderno. Para aprovechar esta funcionalidad, consulte [Acceso y exploración de conjuntos de datos de entrada](#access-and-explore-input-datasets).

### <a name="configure-the-estimator"></a>Configuración del estimador

Un objeto [estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py&preserve-view=true) se usa para enviar la ejecución del experimento. Azure Machine Learning tiene estimadores preconfigurados para marcos de aprendizaje automático comunes, así como un estimador genérico.

Este código crea un objeto estimator genérico, `est`, que especifica:

* Un directorio de script para los scripts. Todos los archivos de este directorio se cargan en los nodos del clúster para su ejecución.
* El script de entrenamiento, *train_titanic.* .
* El conjunto de datos de entrada para el entrenamiento, `titanic_ds`. `as_named_input()` es necesario para que el nombre asignado `titanic` pueda hacer referencia al conjunto de datos de entrada en el script de entrenamiento. 
* El destino de proceso del experimento.
* La definición de entorno del experimento.

```Python
est = Estimator(source_directory=script_folder,
                entry_script='train_titanic.py',
                # pass dataset object as an input with name 'titanic'
                inputs=[titanic_ds.as_named_input('titanic')],
                compute_target=compute_target,
                environment_definition= conda_env)

# Submit the estimator as part of your experiment run
experiment_run = experiment.submit(est)
experiment_run.wait_for_completion(show_output=True)
```

## <a name="mount-files-to-remote-compute-targets"></a>Montar los archivos en destinos de proceso remotos

Si tiene datos no estructurados, cree un conjunto de datos [FileDataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.filedataset?view=azure-ml-py&preserve-view=true) y monte o descargue los archivos de datos para que estén disponibles para el destino de proceso remoto para el entrenamiento. Obtenga información acerca de cuándo usar [el montaje frente a la descarga](#mount-vs-download) para los experimentos de entrenamiento remoto. 

En el ejemplo siguiente se crea un conjunto de datos FileDataset y se monta el conjunto de datos en el destino de proceso, para lo cual se pasa como un argumento en el estimador del entrenamiento. 

> [!Note]
> Si usa una imagen base de Docker personalizada, tendrá que instalar Fuse a través de `apt-get install -y fuse` como una dependencia para que el montaje del conjunto de datos funcione. Obtenga información sobre cómo [compilar una imagen de compilación personalizada](how-to-deploy-custom-docker-image.md#build-a-custom-base-image).

### <a name="create-a-filedataset"></a>Creación de un objeto FileDataset

En el ejemplo siguiente se crea un objeto FileDataset no registrado a partir de direcciones URL web. Más información sobre [cómo crear conjuntos de datos](how-to-create-register-datasets.md) desde otros orígenes.

```Python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
mnist_ds = Dataset.File.from_files(path = web_paths)
```

### <a name="configure-the-estimator"></a>Configuración del estimador

Se recomienda pasar el conjunto de datos como un argumento al realizar el montaje. Además de pasar el conjunto de datos mediante el parámetro `inputs` en el estimador, también puede pasarlo mediante `script_params` y obtener la ruta de acceso a los datos (punto de montaje) en el script de entrenamiento por medio de argumentos. En otras palabras, podrá usar el mismo script de entrenamiento para la depuración local y el entrenamiento remoto en todas las plataformas en la nube.

Un objeto estimator [SKLearn](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.sklearn.sklearn?view=azure-ml-py&preserve-view=true) se usa para enviar la ejecución de experimentos scikit-learn. Después de enviar la ejecución, los archivos de datos a los que el conjunto de datos `mnist` hace referencia se montarán en el destino de proceso. Obtenga más información sobre el entrenamiento con el [calculador de SKlearn](how-to-train-scikit-learn.md).

```Python
from azureml.train.sklearn import SKLearn

script_params = {
    # mount the dataset on the remote compute and pass the mounted path as an argument to the training script
    '--data-folder': mnist_ds.as_named_input('mnist').as_mount(),
    '--regularization': 0.5
}

est = SKLearn(source_directory=script_folder,
              script_params=script_params,
              compute_target=compute_target,
              environment_definition=env,
              entry_script='train_mnist.py')

# Run the experiment
run = experiment.submit(est)
run.wait_for_completion(show_output=True)
```

### <a name="retrieve-the-data-in-your-training-script"></a>Recuperación de los datos en el script de entrenamiento

En el código siguiente se muestra cómo recuperar los datos en el script.

```Python
%%writefile $script_folder/train_mnist.py

import argparse
import os
import numpy as np
import glob

from utils import load_data

# retrieve the 2 arguments configured through script_params in estimator
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# get the file paths on the compute
X_train_path = glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0]
X_test_path = glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0]
y_train_path = glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0]
y_test = glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0]

# load train and test set into numpy arrays
X_train = load_data(X_train_path, False) / 255.0
X_test = load_data(X_test_path, False) / 255.0
y_train = load_data(y_train_path, True).reshape(-1)
y_test = load_data(y_test, True).reshape(-1)
```

## <a name="mount-vs-download"></a>Montaje frente a descarga

Tanto el montaje como la descarga de archivos de cualquier formato se admiten en los conjuntos de datos creados desde Azure Blob Storage, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database y Azure Database for PostgreSQL. 

Al **montar** un conjunto de datos, se asocian a un directorio (punto de montaje) los archivos a los que hace referencia en el conjunto de datos, y este se vuelve disponible en el destino de proceso. El montaje se admite en procesos basados en Linux, como Proceso de Azure Machine Learning, máquinas virtuales y HDInsight. 

Al **descargar** un conjunto de datos, todos los archivos a los que se hace referencia en él se descargarán en el destino de proceso. La descarga se admite con todos los tipos de proceso. 

Si el script procesa todos los archivos a los que se hace referencia en el conjunto de datos y el disco de proceso puede caber en el conjunto de datos completo, se recomienda la descarga para evitar la sobrecarga de la transmisión de datos desde los servicios de almacenamiento. Si el tamaño de los datos supera el tamaño del disco de proceso, no es posible realizar la descarga. En este escenario, se recomienda el montaje, ya que en el momento del procesamiento solo se cargan los archivos de datos usados por el script.

El siguiente código monta el elemento `dataset` en el directorio temporal de `mounted_path`.

```python
import tempfile
mounted_path = tempfile.mkdtemp()

# mount dataset onto the mounted_path of a Linux-based compute
mount_context = dataset.mount(mounted_path)

mount_context.start()

import os
print(os.listdir(mounted_path))
print (mounted_path)
```

## <a name="access-datasets-in-your-script"></a>Obtener acceso a los conjuntos de datos del script

A los conjuntos de bases de datos registrados se puede acceder de local y remota en clústeres de proceso como el proceso de Azure Machine Learning. Para acceder a un conjunto de datos registrado en los experimentos, use el siguiente código para acceder al área de trabajo y al conjunto de datos registrado por nombre. De forma predeterminada, el método [`get_by_name()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#&preserve-view=trueget-by-name-workspace--name--version--latest--) de la clase `Dataset` devuelve la versión más reciente del conjunto de datos registrada en el área de trabajo.

```Python
%%writefile $script_folder/train.py

from azureml.core import Dataset, Run

run = Run.get_context()
workspace = run.experiment.workspace

dataset_name = 'titanic_ds'

# Get a dataset by name
titanic_ds = Dataset.get_by_name(workspace=workspace, name=dataset_name)

# Load a TabularDataset into pandas DataFrame
df = titanic_ds.to_pandas_dataframe()
```

## <a name="accessing-source-code-during-training"></a>Acceso al código fuente durante el entrenamiento

Azure Blob Storage presenta mayores velocidades de rendimiento que un recurso compartido de archivos de Azure y se escala a un gran número de trabajos iniciados en paralelo. Por esta razón, se recomienda configurar las ejecuciones de manera que usen Blob Storage para transferir archivos de código fuente.

En el ejemplo de código siguiente se especifica en la configuración de ejecución el almacén de datos de blobs que se usará para las transferencias de código fuente.

```python 
# workspaceblobstore is the default blob storage
run_config.source_directory_data_store = "workspaceblobstore" 
```

## <a name="notebook-examples"></a>Ejemplos de cuadernos

Los [cuadernos de conjunto de datos](https://aka.ms/dataset-tutorial) se demuestran y se analizan con mayor profundidad sobre los conceptos en este artículo.

## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento automático de modelos de aprendizaje automático](how-to-auto-train-remote.md) con TabularDatasets.

* [Entrenamiento de modelos de clasificación de imágenes](https://aka.ms/filedataset-samplenotebook) con FileDatasets.

* [Entrenamiento con conjuntos de datos mediante canalizaciones](how-to-create-your-first-pipeline.md).
