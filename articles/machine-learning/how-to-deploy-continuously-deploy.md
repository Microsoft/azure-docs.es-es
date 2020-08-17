---
title: Implementación continua de modelos de Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo implementar modelos continuamente con la extensión DevOps de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 08/03/2020
ms.topic: conceptual
ms.custom: how-to, tracking-python
ms.openlocfilehash: c95e1012decf1e3ae4ee3fbab8aae1a4c1a71be9
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87544497"
---
# <a name="continuously-deploy-models"></a>Implementación de modelos de forma continuada

En este artículo se muestra cómo usar la implementación continua en Azure DevOps para buscar automáticamente nuevas versiones de los modelos registrados e incorporarlos en producción.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya ha registrado un modelo en el área de trabajo de Azure Machine Learning. Consulte [este tutorial](how-to-train-scikit-learn.md) para obtener un ejemplo de entrenamiento y registro de un modelo de scikit-learn.

## <a name="continuously-deploy-models"></a>Implementación de modelos de forma continuada

Puede implementar continuamente modelos mediante la extensión de Machine Learning para [Azure DevOps](https://azure.microsoft.com/services/devops/). Puede usar la extensión de Machine Learning para Azure DevOps para desencadenar una canalización de implementación cuando se registra un nuevo modelo de aprendizaje automático en el área de trabajo de Azure Machine Learning.

1. Regístrese en [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops), lo que hace posible la integración continua y la entrega de la aplicación en cualquier plataforma o nube. (Tenga en cuenta que Azure Pipelines no es lo mismo que las [canalizaciones de Machine Learning](concept-ml-pipelines.md#compare)).

1. [Cree un proyecto de Azure DevOps.](https://docs.microsoft.com/azure/devops/organizations/projects/create-project?view=azure-devops)

1. Instale la [extensión de Machine Learning para Azure Pipelines](https://marketplace.visualstudio.com/items?itemName=ms-air-aiagility.vss-services-azureml&targetId=6756afbe-7032-4a36-9cb6-2771710cadc2&utm_source=vstsproduct&utm_medium=ExtHubManageList).

1. Use las conexiones de servicio para configurar una conexión de entidad de servicio para el área de trabajo de Azure Machine Learning para tener acceso a los artefactos. Vaya a configuración del proyecto, seleccione **Conexiones de servicio** y, luego, seleccione **Azure Resource Manager**:

    [![Selección de Azure Resource Manager](media/how-to-deploy-and-where/view-service-connection.png)](media/how-to-deploy-and-where/view-service-connection-expanded.png)

1. En la lista **Nivel de ámbito**, seleccione **AzureMLWorkspace** y luego escriba el resto de los valores:

    ![Selección de AzureMLWorkspace](media/how-to-deploy-and-where/resource-manager-connection.png)

1. Para implementar continuamente el modelo de aprendizaje automático mediante Azure Pipelines, seleccione **Versión** en Canalizaciones. Agregue un artefacto nuevo y seleccione el artefacto del **modelo de AzureML** y la conexión de servicio que creó en el paso anterior. Seleccione el modelo y la versión para desencadenar una implementación:

    [![Selección del modelo de AzureML](media/how-to-deploy-and-where/enable-modeltrigger-artifact.png)](media/how-to-deploy-and-where/enable-modeltrigger-artifact-expanded.png)

1. Habilite el desencadenador del modelo en el artefacto del modelo. Al activar el desencadenador, cada vez que se registra la versión especificada (es decir, la versión más reciente) de ese modelo en el área de trabajo, se desencadena una canalización de versión de Azure DevOps.

    [![Habilitación del desencadenador de modelo](media/how-to-deploy-and-where/set-modeltrigger.png)](media/how-to-deploy-and-where/set-modeltrigger-expanded.png)

## <a name="next-steps"></a>Pasos siguientes

Consulte los proyectos siguientes en GitHub para obtener más ejemplos de implementación continua de modelos de ML.

* [Microsoft/MLOps](https://github.com/Microsoft/MLOps)
* [Microsoft/MLOpsPython](https://github.com/microsoft/MLOpsPython)