---
title: 'Tutorial: Uso de sus propios datos'
titleSuffix: Azure Machine Learning
description: La parte 4 de la serie de introducción de Azure ML muestra cómo usar sus propios datos en una ejecución de entrenamiento remoto.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/15/2020
ms.custom: tracking-python
ms.openlocfilehash: 876ba76655572979a1d831a1ca07e5f3871a3283
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90929714"
---
# <a name="tutorial-use-your-own-data-part-4-of-4"></a>Tutorial: Uso de sus propios datos (Parte 4 de 4)

En este tutorial se muestra cómo cargar y usar sus propios datos para entrenar modelos de Machine Learning en Azure Machine Learning.

Este tutorial es la **cuarta parte de una serie de tutoriales de cuatro partes** en la que aprenderá los aspectos básicos de Azure Machine Learning y completará tareas de aprendizaje automático basadas en trabajos en Azure. Este tutorial se basa en el trabajo completado en [Parte 1: Configuración](tutorial-1st-experiment-sdk-setup-local.md), [Parte 2: Ejecución de "Hola mundo"](tutorial-1st-experiment-hello-world.md) y [parte 3: Entrenamiento de un modelo](tutorial-1st-experiment-sdk-train.md).

En [Parte 3: Entrenamiento de un modelo](tutorial-1st-experiment-sdk-train.md), los datos se descargaron mediante el método `torchvision.datasets.CIFAR10` integrado de la API de PyTorch. Sin embargo, en muchos casos querrá usar sus propios datos en una ejecución de entrenamiento remota. En este artículo se muestra el flujo de trabajo que puede usar para trabajar con sus propios datos en Azure Machine Learning.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Configurar el script de entrenamiento para usar datos en un directorio local.
> * Probar el script de entrenamiento localmente.
> * Cargar datos en Azure.
> * Crear el script de control.
> * Comprender los nuevos conceptos de Azure Machine Learning (pasar parámetros, conjuntos de datos, almacenes de datos).
> * Enviar y ejecutar el script de entrenamiento.
> * Ver la salida del código en la nube.

## <a name="prerequisites"></a>Requisitos previos

* Completar la [parte 3](tutorial-1st-experiment-sdk-train.md) de la serie.
* Conocimiento introductorio del lenguaje Python y los flujos de trabajo de aprendizaje automático.
* Entorno de desarrollo local. Esto incluye, entre otros, Visual Studio Code, Jupyter o PyCharm.
* Python (versión 3.5-3.7).

## <a name="adjust-the-training-script"></a>Ajuste del script de entrenamiento
Ahora tiene el script de entrenamiento (tutorial/src/train.py) en ejecución en Azure Machine Learning y puede supervisar el rendimiento del modelo. Vamos a parametrizar el script de entrenamiento mediante la introducción de los argumentos. El uso de argumentos le permitirá comparar fácilmente diferentes hiperparámetros.

En la actualidad, el script de entrenamiento está configurado para descargar el conjunto de datos de CIFAR10 en cada ejecución. El código de Python siguiente se ha ajustado para leer los datos de un directorio.

>[!NOTE] 
> Uso de `argparse` para parametrizar el script.

```python
# tutorial/src/train.py
import os
import argparse
import torch
import torch.optim as optim
import torchvision
import torchvision.transforms as transforms

from model import Net
from azureml.core import Run

run = Run.get_context()

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--data_path', type=str, help='Path to the training data')
    parser.add_argument('--learning_rate', type=float, default=0.001, help='Learning rate for SGD')
    parser.add_argument('--momentum', type=float, default=0.9, help='Momentum for SGD')
    args = parser.parse_args()
    
    print("===== DATA =====")
    print("DATA PATH: " + args.data_path)
    print("LIST FILES IN DATA PATH...")
    print(os.listdir(args.data_path))
    print("================")
    
    # prepare DataLoader for CIFAR10 data
    transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])
    trainset = torchvision.datasets.CIFAR10(
        root=args.data_path,
        train=True,
        download=False,
        transform=transform,
    )
    trainloader = torch.utils.data.DataLoader(trainset, batch_size=4, shuffle=True, num_workers=2)

    # define convolutional network
    net = Net()

    # set up pytorch loss /  optimizer
    criterion = torch.nn.CrossEntropyLoss()
    optimizer = optim.SGD(
        net.parameters(),
        lr=args.learning_rate,
        momentum=args.momentum,
    )

    # train the network
    for epoch in range(2):

        running_loss = 0.0
        for i, data in enumerate(trainloader, 0):
            # unpack the data
            inputs, labels = data

            # zero the parameter gradients
            optimizer.zero_grad()

            # forward + backward + optimize
            outputs = net(inputs)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()

            # print statistics
            running_loss += loss.item()
            if i % 2000 == 1999:
                loss = running_loss / 2000
                run.log('loss', loss) # log loss metric to AML
                print(f'epoch={epoch + 1}, batch={i + 1:5}: loss {loss:.2f}')
                running_loss = 0.0

    print('Finished Training')
```

### <a name="understanding-the-code-changes"></a>Comprensión de los cambios de código

El código utilizado en `train.py` ha aprovechado la biblioteca de `argparse` para configurar `data_path`, `learning_rate` y `momentum`.

```python
# .... other code
parser = argparse.ArgumentParser()
parser.add_argument('--data_path', type=str, help='Path to the training data')
parser.add_argument('--learning_rate', type=float, default=0.001, help='Learning rate for SGD')
parser.add_argument('--momentum', type=float, default=0.9, help='Momentum for SGD')
args = parser.parse_args()
# ... other code
```

También se ha adaptado el script de `train.py` para actualizar el optimizador para usar los parámetros definidos por el usuario:

```python
optimizer = optim.SGD(
    net.parameters(),
    lr=args.learning_rate,     # get learning rate from command-line argument
    momentum=args.momentum,    # get momentum from command-line argument
)
```

## <a name="test-the-script-locally"></a>Prueba local del script

El script ahora acepta las _rutas de acceso de datos_ como argumento. Para empezar, pruébelo localmente. Agregue a la estructura de directorios del tutorial una carpeta denominada `data`. La estructura de directorios debe tener el siguiente aspecto:

```txt
tutorial
└──.azureml
|  └──config.json
|  └──pytorch-env.yml
└──data
└──src
|  └──hello.py
|  └──model.py
|  └──train.py
└──01-create-workspace.py
└──02-create-compute.py
└──03-run-hello.py
└──04-run-pytorch.py
```

Si no ejecutó `train.py` localmente en el tutorial anterior, no tendrá el directorio `data/`. En este caso, ejecute el método `torchvision.datasets.CIFAR10` localmente con `download=True` en el script de `train.py`.

Para ejecutar el script de entrenamiento modificado localmente, llame a:

```bash
python src/train.py --data_path ./data --learning_rate 0.003 --momentum 0.92
```

Para evitar tener que descargar el conjunto de datos de CIFAR10, pase una ruta de acceso local a los datos. También puede experimentar con valores diferentes para los hiperparámetros de _velocidad de aprendizaje_ y _momento_ sin necesidad de codificarlos de forma rígida en el script de entrenamiento.

## <a name="upload-the-data-to-azure"></a>Carga de los datos en Azure

Para ejecutar este script en Azure Machine Learning, es necesario que los datos de entrenamiento estén disponibles en Azure. El área de trabajo de Azure Machine Learning incluye un **almacén de datos** _predeterminado_ (una cuenta de Azure Blob Storage) que puede usar para almacenar los datos de entrenamiento.

>[!NOTE] 
> Azure Machine Learning permite conectar otros almacenes de datos basados en la nube que almacenan los datos. Para más información, vea la [documentación de los almacenes de datos](./concept-data.md).  

Cree un nuevo script de control de Python llamado `05-upload-data.py` en el directorio `tutorial`:

```python
# tutorial/05-upload-data.py
from azureml.core import Workspace
ws = Workspace.from_config()
datastore = ws.get_default_datastore()
datastore.upload(src_dir='./data', target_path='datasets/cifar10', overwrite=True)
```

`target_path` especifica la ruta de acceso en el almacén de datos donde se cargarán los datos de CIFAR10.

>[!TIP] 
> Mientras usa Azure Machine Learning para cargar los datos, puede usar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para cargar archivos ad hoc. Si necesita una herramienta de extracción, transformación y carga de datos, se puede usar [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) para ingerir los datos en Azure.

Ejecute el archivo de Python para cargar los datos (tenga en cuenta que la carga debe ser rápida, menos de 60 segundos).

```bash
python 05-upload-data.py
```
Debería ver la siguiente salida estándar:
```txt
Uploading ./data\cifar-10-batches-py\data_batch_2
Uploaded ./data\cifar-10-batches-py\data_batch_2, 4 files out of an estimated total of 9
.
.
Uploading ./data\cifar-10-batches-py\data_batch_5
Uploaded ./data\cifar-10-batches-py\data_batch_5, 9 files out of an estimated total of 9
Uploaded 9 files
```


## <a name="create-a-control-script"></a>Creación de un script de control

Como ha hecho anteriormente, cree un nuevo script de control de Python llamado `06-run-pytorch-data.py`:

```python
# tutorial/06-run-pytorch-data.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig
from azureml.core import Dataset

if __name__ == "__main__":
    ws = Workspace.from_config()
    
    datastore = ws.get_default_datastore()
    dataset = Dataset.File.from_files(path=(datastore, 'datasets/cifar10'))

    experiment = Experiment(workspace=ws, name='day1-experiment-data')

    config = ScriptRunConfig(
        source_directory='./src',
        script='train.py',
        compute_target='cpu-cluster',
        arguments=[
            '--data_path', dataset.as_named_input('input').as_mount(),
            '--learning_rate', 0.003,
            '--momentum', 0.92],
        )
    
    # set up pytorch environment
    env = Environment.from_conda_specification(name='pytorch-env',file_path='.azureml/pytorch-env.yml')
    config.run_config.environment = env

    run = experiment.submit(config)
    aml_url = run.get_portal_url()
    print("Submitted to an Azure Machine Learning compute cluster. Click on the link below")
    print("")
    print(aml_url)
```

### <a name="understand-the-code-changes"></a>Comprensión de los cambios de código

El script de control es similar al de la [parte 3 de esta serie](tutorial-1st-experiment-sdk-train.md) con las siguientes líneas nuevas:

:::row:::
   :::column span="":::
      `dataset = Dataset.File.from_files( ... )`
   :::column-end:::
   :::column span="2":::
      Un [conjunto de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py&preserve-view=true) se usa para hacer referencia a los datos cargados en el almacén de blobs de Azure. Los conjuntos de datos son una capa de abstracción sobre los datos que están diseñados para mejorar la confiabilidad.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config = ScriptRunConfig(...)`
   :::column-end:::
   :::column span="2":::
      [ScriptRunConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrunconfig?view=azure-ml-py&preserve-view=true) se modifica para incluir una lista de argumentos que se pasarán a `train.py`. El argumento `dataset.as_named_input('input').as_mount()` significa que el directorio especificado se _montará_ en el destino de proceso.
   :::column-end:::
:::row-end:::

## <a name="submit-run-to-azure-machine-learning"></a>Envío de la ejecución a Azure Machine Learning

Ahora vuelva a enviar la ejecución para usar la nueva configuración:

```bash
python 06-run-pytorch-data.py
```

De esta forma, se imprimirá una dirección URL en el experimento de Azure Machine Learning Studio. Si navega a ese vínculo, podrá ver el código en ejecución.

### <a name="inspect-the-70_driver_log-log-file"></a>Inspección del archivo de registro 70_driver_log

En Azure Machine Learning Studio, navegue hasta la ejecución del experimento (para ello, haga clic en el resultado de dirección URL de la celda anterior) seguido de **Resultados y registros**. Haga clic en el archivo 70_driver_log.txt. Debería ver el siguiente resultado:

```txt
Processing 'input'.
Processing dataset FileDataset
{
  "source": [
    "('workspaceblobstore', 'datasets/cifar10')"
  ],
  "definition": [
    "GetDatastoreFiles"
  ],
  "registration": {
    "id": "XXXXX",
    "name": null,
    "version": null,
    "workspace": "Workspace.create(name='XXXX', subscription_id='XXXX', resource_group='X')"
  }
}
Mounting input to /tmp/tmp9kituvp3.
Mounted input to /tmp/tmp9kituvp3 as folder.
Exit __enter__ of DatasetContextManager
Entering Run History Context Manager.
Current directory:  /mnt/batch/tasks/shared/LS_root/jobs/dsvm-aml/azureml/tutorial-session-3_1600171983_763c5381/mounts/workspaceblobstore/azureml/tutorial-session-3_1600171983_763c5381
Preparing to call script [ train.py ] with arguments: ['--data_path', '$input', '--learning_rate', '0.003', '--momentum', '0.92']
After variable expansion, calling script [ train.py ] with arguments: ['--data_path', '/tmp/tmp9kituvp3', '--learning_rate', '0.003', '--momentum', '0.92']

Script type = None
===== DATA =====
DATA PATH: /tmp/tmp9kituvp3
LIST FILES IN DATA PATH...
['cifar-10-batches-py', 'cifar-10-python.tar.gz']
```

Aviso:

1. Azure Machine Learning ha montado el almacén de blobs en el clúster de proceso automáticamente.
2. Los ``dataset.as_named_input('input').as_mount()`` usados en el script de control se resuelven en el punto de montaje.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, vimos cómo cargar datos en Azure con un `Datastore`. El almacén de datos sirvió como almacenamiento en la nube para el área de trabajo, lo que le proporcionó un lugar persistente y flexible para conservar los datos.

Vio cómo modificar el script de entrenamiento para aceptar una ruta de acceso de datos a través de la línea de comandos. Mediante el uso de `Dataset`, pudo montar un directorio en la ejecución remota. 

Ahora que tiene un modelo, aprenda a realizar la:

* [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).
