---
title: Integración de Git con Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo se integra Azure Machine Learning con un repositorio de Git local. Al enviar una ejecución de entrenamiento desde un directorio local que es un repositorio de Git, se realiza un seguimiento de la información sobre el repositorio, la rama y la confirmación actual como parte de la ejecución.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.date: 03/05/2020
ms.openlocfilehash: bd77af133b88e1ba93054dbb7e0f896d8d418f89
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893554"
---
# <a name="git-integration-for-azure-machine-learning"></a>Integración de Git con Azure Machine Learning

[Git](https://git-scm.com/) es un conocido sistema de control de versiones que le permite compartir sus proyectos y colaborar con otros usuarios en ellos. 

Azure Machine Learning es totalmente compatible con los repositorios de Git para supervisar el trabajo; puede clonar repositorios directamente al sistema de archivos compartido de su área de trabajo, usar Git en su estación de trabajo local o usarlo desde una canalización de CI/CD.

Al enviar un trabajo a Azure Machine Learning, si los archivos de origen se almacenan en un repositorio de Git local, se realiza un seguimiento de la información sobre el repositorio como parte del proceso de entrenamiento.

Dado que Azure Machine Learning realiza un seguimiento de la información desde un repositorio de Git local, no está vinculado a ningún repositorio central específico. El repositorio se puede clonar desde GitHub, GitLab, Bitbucket, Azure DevOps o cualquier otro servicio compatible con Git.

## <a name="clone-git-repositories-into-your-workspace-file-system"></a>Clonación de repositorios de Git en el sistema de archivos del área de trabajo
Azure Machine Learning ofrece un sistema de archivos compartido para todos los usuarios del área de trabajo.
Para clonar un repositorio de Git a este recurso compartido de archivos, se recomienda crear una instancia de proceso y abrir un terminal.
Una vez abierto el terminal, tendrá acceso a un cliente completo de Git y podrá clonar y trabajar con Git a través de la experiencia de CLI de Git.

Se recomienda clonar el repositorio en el directorio de los usuarios, de modo que otros no generen conflictos directamente en la rama en funcionamiento.

Puede clonar cualquier repositorio de Git en el que pueda autenticarse (GitHub, Azure Repos, BitBucket, etc.).

Para consultar una guía sobre cómo usar la CLI de Git, lea [este vínculo](https://guides.github.com/introduction/git-handbook/).

## <a name="track-code-that-comes-from-git-repositories"></a>Seguimiento del código proveniente de repositorios de Git

Cuando se envía una ejecución de entrenamiento desde el SDK de Python o la CLI de Machine Learning, los archivos necesarios para entrenar el modelo se cargan en el área de trabajo. Si el comando `git` está disponible en el entorno de desarrollo, el proceso de carga lo usa para comprobar si los archivos se almacenan en un repositorio Git. Si es así, la información del repositorio de Git también se carga como parte de la ejecución de entrenamiento. Esta información se almacena en las siguientes propiedades para la ejecución de entrenamiento:

| Propiedad | Comando de Git que se usa para obtener el valor | Descripción |
| ----- | ----- | ----- |
| `azureml.git.repository_uri` | `git ls-remote --get-url` | El URI desde el que se clonó el repositorio. |
| `mlflow.source.git.repoURL` | `git ls-remote --get-url` | El URI desde el que se clonó el repositorio. |
| `azureml.git.branch` | `git symbolic-ref --short HEAD` | Rama activa cuando se envió la ejecución. |
| `mlflow.source.git.branch` | `git symbolic-ref --short HEAD` | Rama activa cuando se envió la ejecución. |
| `azureml.git.commit` | `git rev-parse HEAD` | Hash de confirmación del código enviado para la ejecución. |
| `mlflow.source.git.commit` | `git rev-parse HEAD` | Hash de confirmación del código enviado para la ejecución. |
| `azureml.git.dirty` | `git status --porcelain .` | `True`, si la rama/confirmación se ha modificado; de lo contrario, `false`. |

Esta información se envía para las ejecuciones que usan un estimador, una canalización de aprendizaje automático o una ejecución de scripts.

Si los archivos de entrenamiento no se encuentran en un repositorio de Git en el entorno de desarrollo, o el comando `git` no está disponible, no se realiza el seguimiento de ninguna información relacionada con Git.

> [!TIP]
> Para comprobar si el comando GIT está disponible en el entorno de desarrollo, abra una sesión del shell, un símbolo del sistema, PowerShell u otra interfaz de la línea de comandos y escriba el siguiente comando:
>
> ```
> git --version
> ```
>
> Si está instalado y, en la ruta de acceso, recibirá una respuesta similar a `git version 2.4.1`. Para obtener más información sobre la instalación de GIT en el entorno de desarrollo, consulte el [sitio web de GIT](https://git-scm.com/).

## <a name="view-the-logged-information"></a>Visualización de la información registrada

La información de Git se almacena en las propiedades para la ejecución de entrenamiento. Puede ver esta información mediante Azure Portal, el SDK de Python y la CLI. 

### <a name="azure-portal"></a>Portal de Azure

1. En el [portal de Studio](https://ml.azure.com), seleccione su área de trabajo.
1. Seleccione __Experimentos__ y, a continuación, seleccione uno de los experimentos.
1. Seleccione una de las ejecuciones de la columna __NÚMERO DE EJECUCIÓN__.
1. Seleccione __Resultados y registros__ y, a continuación, expanda las entradas __logs__ y __azureml__. Seleccione el vínculo que comienza por __###\_azure__.

La información registrada contiene texto similar al siguiente JSON:

```json
"properties": {
    "_azureml.ComputeTargetType": "batchai",
    "ContentSnapshotId": "5ca66406-cbac-4d7d-bc95-f5a51dd3e57e",
    "azureml.git.repository_uri": "git@github.com:azure/machinelearningnotebooks",
    "mlflow.source.git.repoURL": "git@github.com:azure/machinelearningnotebooks",
    "azureml.git.branch": "master",
    "mlflow.source.git.branch": "master",
    "azureml.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "mlflow.source.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "azureml.git.dirty": "True",
    "AzureML.DerivedImageName": "azureml/azureml_9d3568242c6bfef9631879915768deaf",
    "ProcessInfoFile": "azureml-logs/process_info.json",
    "ProcessStatusFile": "azureml-logs/process_status.json"
}
```

### <a name="python-sdk"></a>SDK de Python

Después de enviar una ejecución de entrenamiento, se devuelve un objeto [Run](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run%28class%29?view=azure-ml-py&preserve-view=true). El atributo `properties` de este objeto contiene la información de Git registrada. Por ejemplo, el código siguiente recupera el hash de confirmación:

```python
run.properties['azureml.git.commit']
```

### <a name="cli"></a>CLI

El comando de la CLI `az ml run` se puede usar para recuperar las propiedades de una ejecución. Por ejemplo, el comando siguiente devuelve las propiedades de la última ejecución en el experimento denominado `train-on-amlcompute`:

```azurecli-interactive
az ml run list -e train-on-amlcompute --last 1 -w myworkspace -g myresourcegroup --query '[].properties'
```

Para más información, consulte la documentación de referencia de [az ml run](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/run?view=azure-cli-latest).

## <a name="next-steps"></a>Pasos siguientes

* [Uso de los destinos de proceso para el entrenamiento de modelos](how-to-set-up-training-targets.md)
