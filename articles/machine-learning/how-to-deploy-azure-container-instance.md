---
title: Implementación de modelos en Azure Container Instances
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo implementar modelos de Azure Machine Learning como un servicio web con Azure Container Instances.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 06/12/2020
ms.openlocfilehash: cbba0dd5341ad148831ac3b1f94685bf2beddd5a
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855275"
---
# <a name="deploy-a-model-to-azure-container-instances"></a>Implementación de un modelo en Azure Container Instances
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Obtenga información sobre cómo usar Azure Machine Learning para implementar un modelo como un servicio web en Azure Container Instances (ACI). Use Azure Container Instances si se cumple una de las condiciones siguientes:

- Debe implementar y validar rápidamente el modelo. No es necesario crear contenedores ACI de antemano. Se crean como parte del proceso de implementación.
- Está probando un modelo que está en desarrollo. 

Para información de la disponibilidad de cuotas y regiones de ACI, consulte el artículo [Disponibilidad de cuotas y regiones en Azure Container Instances](https://docs.microsoft.com/azure/container-instances/container-instances-quotas).

> [!IMPORTANT]
> Se recomienda realizar una depuración local antes de la implementación en el servicio web. Para obtener más información, vea [Depuración local](https://docs.microsoft.com/azure/machine-learning/how-to-troubleshoot-deployment#debug-locally).
>
> También puede consultar Azure Machine Learning: [Implementación en el cuaderno local](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/deploy-to-local).

## <a name="prerequisites"></a>Requisitos previos

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

- Un modelo de Machine Learning registrado en el área de trabajo. Si no tiene un modelo registrado, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

- En los fragmentos de código de __Python__ de este artículo se supone que se han establecido las siguientes variables:

    * `ws`: establézcalo en su área de trabajo.
    * `model`: establézcalo en el modelo registrado.
    * `inference_config`: establézcalo en la configuración de inferencia del modelo.

    Para más información acerca de cómo establecer estas variables, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

- En los fragmentos de código de la __CLI__ de este artículo se supone que ha creado un documento `inferenceconfig.json`. Para más información acerca cómo crear este documento, consulte el artículo en el que se explica [cómo y dónde se implementan los modelos](how-to-deploy-and-where.md).

## <a name="deploy-to-aci"></a>Implementación en ACI

Para implementar un modelo en Azure Container Instances, cree una __configuración de implementación__ que describa los recursos de proceso necesarios. Por ejemplo, el número de núcleos y la memoria. También necesita una __configuración de inferencia__, que describe el entorno necesario para hospedar el modelo y el servicio web. Para más información sobre cómo crear la configuración de inferencia, consulte [Cómo y dónde implementar modelos](how-to-deploy-and-where.md).

> [!NOTE]
> * ACI solo es adecuado para pequeños modelos con un tamaño inferior a 1 GB. 
> * Se recomienda el uso de AKS de un solo nodo para modelos de desarrollo y pruebas más grandes.
> * El número de modelos que se implementará se limita a 1000 modelos por implementación (por contenedor). 

### <a name="using-the-sdk"></a>Uso del SDK

```python
from azureml.core.webservice import AciWebservice, Webservice
from azureml.core.model import Model

deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "aciservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Para más información acerca de las clases, los métodos y los parámetros que se usan en este ejemplo, consulte los siguientes documentos de referencia:

* [AciWebservice.deploy_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-)
* [Model.deploy](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Webservice.wait_for_deployment](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py#wait-for-deployment-show-output-false-)

### <a name="using-the-cli"></a>Uso de la CLI

Para realizar una implementación con la CLI, use el siguiente comando. Reemplace `mymodel:1` por el nombre y la versión del modelo registrado. Reemplace `myservice` por el nombre que quiere asignar a este servicio:

```azurecli-interactive
az ml model deploy -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aci-deploy-config.md)]

Para obtener más información, consulte la referencia [az ml model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy). 

## <a name="using-vs-code"></a>Uso de Visual Studio Code

Consulte [Implementación de modelos con VS Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

> [!IMPORTANT]
> No es necesario crear un contenedor ACI para probar de antemano. Los contenedores ACI se crean según sea necesario.


## <a name="next-steps"></a>Pasos siguientes

* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Solución de problemas de implementación](how-to-troubleshoot-deployment.md)
* [Actualización del servicio web](how-to-deploy-update-web-service.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Consumir un modelo de ML que está implementado como un servicio web](how-to-consume-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
