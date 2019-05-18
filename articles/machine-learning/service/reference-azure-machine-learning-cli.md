---
title: Extensión de la CLI de Machine Learning
titleSuffix: Azure Machine Learning service
description: Información acerca de la extensión de la CLI de Azure Machine Learning para la CLI de Azure. La CLI de Azure es una utilidad de la línea de comandos multiplataforma que le permite trabajar con recursos de la nube de Azure. La extensión de Machine Learning permite trabajar con el servicio Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: jordane
author: jpe316
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 163b8e1f68b8d5a102465022c67f7d0da57a7215
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596963"
---
# <a name="use-the-cli-extension-for-azure-machine-learning-service"></a>Uso de la extensión de la CLI para Azure Machine Learning Service

La CLI de Azure Machine Learning es una extensión de la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest), una interfaz de la línea de comandos multiplataforma para la plataforma Azure. Esta extensión proporciona comandos para trabajar con el servicio de Azure Machine Learning. Permite automatizar sus actividades de aprendizaje automático. La siguiente lista proporciona algunas acciones de ejemplo que puede hacer con la extensión de la CLI:

+ Realizar experimentos para crear modelos de aprendizaje automático

+ Registrar los modelos de aprendizaje automático para el uso del cliente

+ Empaquetar, implementar y realizar un seguimiento del ciclo de vida de los modelos de aprendizaje automático

La CLI no sustituye al SDK de Azure Machine Learning. Es una herramienta complementaria que está optimizada para controlar tareas muy con parámetros que adaptarse a ellos mismos apropiado para la automatización.

## <a name="prerequisites"></a>Requisitos previos

* Para usar la CLI, debe tener una suscripción de Azure. Si no tiene una suscripción a Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning Service](https://aka.ms/AMLFree).

* La[CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).

## <a name="full-reference-docs"></a>Documentos de referencia completa

Buscar el [completa de documentos de referencia para la extensión de azure-cli-ml de CLI de Azure](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/?view=azure-cli-latest).

## <a name="install-the-extension"></a>Instale la extensión

Para instalar la extensión de la CLI de Machine Learning, use el siguiente comando:

```azurecli-interactive
az extension add -n azure-cli-ml
```

> [!TIP]
> Pueden encontrar los archivos de ejemplo que puede usar con los siguientes comandos [aquí](https://aka.ms/azml-deploy-cloud).

Cuando se le solicite, seleccione `y` para instalar la extensión.

Para comprobar que se ha instalado la extensión, utilice el siguiente comando para mostrar una lista de los subcomandos específicos de ML:

```azurecli-interactive
az ml -h
```

## <a name="remove-the-extension"></a>Eliminación de la extensión

Para eliminar la extensión de la CLI, use el siguiente comando:

```azurecli-interactive
az extension remove -n azure-cli-ml
```

## <a name="resource-management"></a>Administración de recursos

Los siguientes comandos muestran cómo utilizar la CLI para administrar los recursos que usa Azure Machine Learning.

+ Si ya tiene uno, cree un grupo de recursos:

    ```azurecli-interactive
    az group create -n myresourcegroup -l westus2
    ```

+ Cree un área de trabajo de Azure Machine Learning:

    ```azurecli-interactive
    az ml workspace create -w myworkspace -g myresourcegroup
    ```

    Para obtener más información, consulte [Crear área de trabajo de az ml](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-create).

+ Adjuntar una configuración de área de trabajo a una carpeta para habilitar el reconocimiento contextual de CLI.

    ```azurecli-interactive
    az ml folder attach -w myworkspace -g myresourcegroup
    ```

    Este comando crea un `.azureml` subdirectorio que contiene los archivos del entorno de ejemplo /runconfig y de conda. También contiene un `config.json` archivo que se utiliza para comunicarse con el área de trabajo de Azure Machine Learning.

    Para obtener más información, consulte [adjuntar de la carpeta de az ml](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/folder?view=azure-cli-latest#ext-azure-cli-ml-az-ml-folder-attach).

+ Adjuntar un contenedor de blobs de Azure como un almacén de datos.

    ```azurecli-interactive
    az ml datastore attach-blob  -n datastorename -a accountname -c containername
    ```

    Para obtener más información, consulte [az ml datastore adjuntar blob](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/datastore?view=azure-cli-latest#ext-azure-cli-ml-az-ml-datastore-attach-blob).

    
+ Conectar un clúster de AKS como un destino de proceso.

    ```azurecli-interactive
    az ml computetarget attach aks -n myaks -i myaksresourceid -g myresourcegroup -w myworkspace
    ```

    Para obtener más información, consulte [az ml computetarget attach aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-attach-aks)

+ Cree un nuevo destino AMLcompute.

    ```azurecli-interactive
    az ml computetarget create amlcompute -n cpu --min-nodes 1 --max-nodes 1 -s STANDARD_D3_V2
    ```

    Para obtener más información, consulte [az ml computetarget crear amlcompute](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-amlcompute).

## <a id="experiments"></a>Ejecutar experimentos

* Inicie la ejecución del experimento. Cuando se usa este comando, especifique el nombre del archivo /runconfig (el texto delante \*.runconfig si se trata de su sistema de archivos) con respecto al parámetro - c.

    ```azurecli-interactive
    az ml run submit-script -c sklearn -e testexperiment train.py
    ```

    > [!TIP]
    > El `az ml folder attach` comando crea un `.azureml` subdirectorio, que contiene dos archivos de ejemplo /runconfig. 
    >
    > Si tiene un script de Python que crea un objeto de configuración de ejecución mediante programación, puede usar [RunConfig.save()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py#save-path-none--name-none--separate-environment-yaml-false-) para guardarlo como un archivo .runconfig.
    >
    > Para obtener más archivos de ejemplo /runconfig, consulte [ https://github.com/MicrosoftDocs/pipelines-azureml/tree/master/.azureml ](https://github.com/MicrosoftDocs/pipelines-azureml/tree/master/.azureml).

    Para obtener más información, consulte [az ml ejecutar enviar script](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/run?view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-submit-script).

* Ver una lista de experimentos:

    ```azurecli-interactive
    az ml experiment list
    ```

    Para obtener más información, consulte [az ml experimentar lista](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/experiment?view=azure-cli-latest#ext-azure-cli-ml-az-ml-experiment-list).

## <a name="model-registration-profiling-deployment"></a>Registro del modelo, generación de perfiles, implementación

Los siguientes comandos muestran cómo registrar un modelo entrenado e implementarlo como servicio de producción:

+ Registre un modelo con Azure Machine Learning:

    ```azurecli-interactive
    az ml model register -n mymodel -p sklearn_regression_model.pkl
    ```

    Para obtener más información, consulte [register de az ml modelo](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-register).

+ **OPCIONAL** el modelo para obtener los valores óptimos de CPU y memoria para la implementación de perfil.
    ```azurecli-interactive
    az ml model profile -n myprofile -m mymodel:1 --ic inferenceconfig.json -d "{\"data\": [[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}" -t myprofileresult.json
    ```

    Para obtener más información, consulte [perfil de az ml modelo](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-profile).

+ Implementa el modelo en AKS

    ```azurecli-interactive
    az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json
    ```

    El siguiente es un ejemplo `inferenceconfig.json` documento:

    ```json
    {
    "entryScript": "score.py",
    "runtime": "python",
    "condaFile": "myenv.yml",
    "extraDockerfileSteps": null,
    "sourceDirectory": null,
    "enableGpu": false,
    "baseImage": null,
    "baseImageRegistry": null
    }
    ```

    Para obtener más información, consulte [implementar el modelo de aprendizaje automático de az](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).


## <a name="next-steps"></a>Pasos siguientes

* [Referencia para la extensión de la CLI de Machine Learning del comando](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml?view=azure-cli-latest).

* [Entrenar e implementar modelos de aprendizaje automático mediante canalizaciones de Azure](/azure/devops/pipelines/targets/azure-machine-learning)