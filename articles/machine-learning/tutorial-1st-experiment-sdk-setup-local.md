---
title: 'Tutorial:  Introducción a Machine Learning: Python'
titleSuffix: Azure Machine Learning
description: En este tutorial empezará a trabajar con el SDK de Python para Azure Machine Learning que se ejecuta en el entorno de desarrollo personal.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/15/2020
ms.custom: devx-track-python
ms.openlocfilehash: 48edc138e7ab16e712339d6291db52a4a2b36b32
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90929709"
---
# <a name="tutorial-get-started-with-azure-machine-learning-on-your-development-environment-part-1-of-4"></a>Tutorial: Introducción a Azure Machine Learning en el entorno de desarrollo (parte 1 de 4)

En esta **serie de tutoriales de cuatro partes**, aprenderá los aspectos básicos de Azure Machine Learning y completará tareas de ML de Python basadas en trabajos en la nube de Azure, entre las que se incluyen las siguientes:

1. Configuración de un área de trabajo y el entorno de desarrollador de aprendizaje automático local.
2. Ejecución del código en la nube con el SDK de Python de Azure Machine Learning.
3. Administración del entorno de Python que se usa para el entrenamiento del modelo.
4. Carga de datos en Azure y consumo de esos datos en el entrenamiento.

En la **parte 1 de esta serie de tutoriales**, deberá realizar lo siguiente:

> [!div class="checklist"]
> * Instalación del SDK de Azure Machine Learning
> * Configuración de la estructura de directorio del código
> * Creación de un área de trabajo de Azure Machine Learning
> * Configuración del entorno de desarrollo local
> * Configuración de un clúster de proceso

>[!NOTE]
> Esta serie de tutoriales se centra en los conceptos de Azure Machine Learning que se adaptan a las tareas de aprendizaje automático __basadas en trabajos__ de Python que tienen muchos procesos o que requieren reproducibilidad. Si las tareas de aprendizaje automático no se ajustan a este perfil, use la [funcionalidad de Jupyter o RStudio en una instancia de proceso de Azure Machine Learning](tutorial-1st-experiment-sdk-setup.md) para incorporarse a Azure Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción de Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe [Azure Machine Learning](https://aka.ms/AMLFree) hoy mismo.
- Familiaridad con los [conceptos de Machine Learning](concept-azure-machine-learning-architecture.md) y Python. Por ejemplo, entornos, entrenamiento, puntuación, etc.
- Un entorno de desarrollo local: un portátil con Python instalado y su IDE favorito (por ejemplo: VSCode, Pycharm, Jupyter, etc.).

## <a name="install-the-azure-machine-learning-sdk"></a>Instalación del SDK de Azure Machine Learning

En este tutorial se usa el SDK de Python de Azure Machine Learning.

Puede usar las herramientas más conocidas (por ejemplo: Conda, PIP, etc.) para configurar un entorno que se usará en este tutorial. Instale en el entorno el SDK de Python de Azure Machine Learning a través de PIP:

```bash
pip install azureml-sdk
```

## <a name="create-directory-structure-for-code"></a>Creación de la estructura de directorio del código
Se recomienda configurar el siguiente directorio sencillo para este tutorial:

```markdown
tutorial
└──.azureml
```

- **tutorial** (directorio de nivel superior del proyecto)
- **.azureml** (subdirectorio oculto del tutorial):  El directorio `.azureml` se usa para almacenar archivos de configuración de Azure Machine Learning.

## <a name="create-an-azure-machine-learning-workspace"></a>Crear un área de trabajo de Azure Machine Learning

Un área de trabajo es un recurso de nivel superior de Azure Machine Learning y es un lugar centralizado para:

- Administrar recursos, como procesos.
- Almacenar recursos, como cuadernos, entornos, conjuntos de datos, canalizaciones, modelos, extremos, etc.
- Colaborar con otros miembros del equipo.

En el directorio superior primario, `tutorial`, agregue un nuevo archivo de Python denominado `01-create-workspace.py` con el código siguiente. Adapte los parámetros (nombre, id. de suscripción, grupo de recursos y [ubicación](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service)) con sus preferencias.

Puede ejecutar el código en una sesión interactiva o como un archivo de Python.

>[!NOTE]
> Cuando se usa un entorno de desarrollo local (por ejemplo, un portátil), se le pedirá que se autentique en el área de trabajo mediante un *código de dispositivo* la primera vez que ejecute el código siguiente. Sigue las instrucciones en pantalla.

```python
# tutorial/01-create-workspace.py
from azureml.core import Workspace

ws = Workspace.create(name='<my_workspace_name>', # provide a name for your workspace
                      subscription_id='<azure-subscription-id>', # provide your subscription ID
                      resource_group='<myresourcegroup>', # provide a resource group name
                      create_resource_group=True,
                      location='<NAME_OF_REGION>') # For example: 'westeurope' or 'eastus2' or 'westus2' or 'southeastasia'.

# write out the workspace details to a configuration file: .azureml/config.json
ws.write_config(path='.azureml')
```

Ejecute este código desde el directorio de `tutorial`:

```bash
cd <path/to/tutorial>
python ./01-create-workspace.py
```

Una vez que se haya ejecutado el fragmento de código anterior, la estructura de carpetas tendrá el siguiente aspecto:

```markdown
tutorial
└──.azureml
|  └──config.json
└──01-create-workspace.py
```

El archivo `.azureml/config.json` contiene los metadatos necesarios para conectarse al área de trabajo de Azure Machine Learning, es decir, el id. suscripción, el grupo de recursos y el nombre del área de trabajo. 

> [!NOTE]
> El contenido de `config.json` no es secreto: se pueden compartir estos detalles.
> Todavía se requiere autenticación para interactuar con el área de trabajo de Azure Machine Learning.

## <a name="create-an-azure-machine-learning-compute-cluster"></a>Creación de un clúster de proceso de Azure Machine Learning

Cree un script de Python en el directorio de nivel superior `tutorial` denominado `02-create-compute.py` y rellene con el código siguiente para crear un clúster de proceso de Azure Machine Learning que se escalará automáticamente entre cero y cuatro nodos:

```python
# tutorial/02-create-compute.py
from azureml.core import Workspace
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

ws = Workspace.from_config() # this automatically looks for a directory .azureml

# Choose a name for your CPU cluster
cpu_cluster_name = "cpu-cluster"

# Verify that cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print('Found existing cluster, use it.')
except ComputeTargetException:
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=4, 
                                                            idle_seconds_before_scaledown=2400)
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

cpu_cluster.wait_for_completion(show_output=True)
```

Ejecución del archivo de Python:

```bash
python ./02-create-compute.py
```


> [!NOTE]
> Cuando se haya creado el clúster, no tendrá ningún nodo aprovisionado. Por lo tanto, el clúster **no** genera costos hasta que envíe un trabajo. Este clúster se reducirá verticalmente cuando haya estado inactivo durante 2400 segundos (40 minutos).

Ahora la estructura de carpetas tendrá el siguiente aspecto:

```bash
tutorial
└──.azureml
|  └──config.json
└──01-create-workspace.py
└──02-create-compute.py
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial de configuración, hizo lo siguiente:

- Crear un área de trabajo de Azure Machine Learning
- Configurar el entorno de desarrollo local
- Ha creado un clúster de proceso de Azure Machine Learning.

En el siguiente tutorial, le servirá como guía para el envío de un script al clúster de proceso de Azure Machine Learning.

> [!div class="nextstepaction"]
> [Tutorial: Ejecución del script de Python "Hola mundo" en Azure](tutorial-1st-experiment-hello-world.md)
