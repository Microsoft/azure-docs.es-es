---
title: Registro de métricas y experimentos de ML
titleSuffix: Azure Machine Learning
description: Supervise sus experimentos de Azure ML y supervise las métricas de ejecución para mejorar el proceso de creación de modelos. Agregue el registro al script de entrenamiento mediante run.log, Run.start_logging o ScriptRunConfig.
services: machine-learning
author: likebupt
ms.author: keli19
ms.reviewer: peterlu
ms.service: machine-learning
ms.subservice: core
ms.date: 07/30/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 44fe71f575a32ccc1a687bc87793cb6a8b6508a9
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89650621"
---
# <a name="enable-logging-in-azure-ml-training-runs"></a>Habilitación del registro en ejecuciones de entrenamiento de Azure ML
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

El SDK de Python para Azure Machine Learning permite registrar información en tiempo real mediante el paquete de registro de Python predeterminado y la funcionalidad específica del SDK. Puede anotar registros de forma local y enviarlos al área de trabajo en el portal.

Los registros pueden ayudarle a diagnosticar errores y advertencias, o a realizar un seguimiento de las métricas de rendimiento, como los parámetros y el rendimiento del modelo. En este artículo, aprenderá a habilitar el registro en los siguientes escenarios:

> [!div class="checklist"]
> * Sesiones de aprendizaje interactivo
> * Envío de trabajos de entrenamiento mediante ScriptRunConfig
> * Configuración nativa de `logging` de Python
> * Registro desde otros orígenes


> [!TIP]
> En este artículo se muestra cómo supervisar el proceso de entrenamiento del modelo. Si está interesado en la supervisión del uso de los recursos y eventos desde Azure Machine Learning, como las cuotas o las ejecuciones de entrenamiento o implementaciones de modelos completadas, puede consultar [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="data-types"></a>Tipos de datos

Puede registrar varios tipos de datos, como valores escalares, listas, tablas, imágenes, directorios, etc. Para más información y ver ejemplos de código de Python para diferentes tipos de datos, consulte la [página de referencia de la clase Run](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run(class)?view=azure-ml-py&preserve-view=true).

## <a name="interactive-logging-session"></a>Sesión de registro interactivo

Las sesiones de registro interactivo se utilizan normalmente en entornos de cuaderno. El método [Experiment.start_logging()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment(class)?view=azure-ml-py#&preserve-view=truestart-logging--args----kwargs-) comienza con una sesión de registro interactivo. Todas las métricas que se registran durante la sesión se agregan al registro de ejecución del experimento. El método [run.complete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run(class)?view=azure-ml-py#&preserve-view=truecomplete--set-status-true-) finaliza las sesiones y marca la ejecución como completada.

## <a name="scriptrunconfig-logs"></a>Registros de ScriptRunConfig

En esta sección, aprenderá a agregar código de registro dentro de las ejecuciones de ScriptConfig. Puede usar la clase [**ScriptRunConfig**](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrunconfig?view=azure-ml-py&preserve-view=true) para encapsular los scripts y entornos y permitir así ejecuciones repetibles. También puede usar esta opción para mostrar un widget visual de instancias de Jupyter Notebook para la supervisión.

En este ejemplo se realiza un barrido de parámetros sobre valores alfa y se capturan los resultados mediante el método [run.log()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run(class)?view=azure-ml-py#&preserve-view=truelog-name--value--description----).

1. Cree un script de entrenamiento que incluya la lógica de registro `train.py`.

   [!code-python[] (~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train.py)]


1. Envíe el script ```train.py``` para ejecutarlo en un entorno administrado por el usuario. La carpeta de scripts completa se envía para el entrenamiento.

   [!notebook-python[] (~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train-on-local.ipynb?name=src)] [!notebook-python[] (~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train-on-local.ipynb?name=run)]

    El parámetro `show_output` activa el registro detallado, que le permite ver los detalles del proceso de entrenamiento, así como información sobre los recursos remotos o los destinos de proceso. Use el código siguiente para activar el registro detallado al enviar el experimento.

```python
run = exp.submit(src, show_output=True)
```

También puede usar el mismo parámetro en la función `wait_for_completion` de la ejecución resultante.

```python
run.wait_for_completion(show_output=True)
```

## <a name="native-python-logging"></a>Registro nativo de Python

Algunos registros del SDK pueden contener un error que le insta a establecer el nivel de registro en DEBUG. Para establecer el nivel de registro, agregue el código siguiente al script.

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## <a name="additional-logging-sources"></a>Orígenes de registro adicionales

Azure Machine Learning también puede registrar información de otros orígenes durante el entrenamiento, como las ejecuciones de aprendizaje automático automatizado o los contenedores de Docker que ejecutan los trabajos. Estos registros no se documentan, pero si encuentra problemas y se pone en contacto con el soporte técnico de Microsoft, es posible que puedan usar estos registros como ayuda para resolverlos.

Para información sobre el registro de métricas en el diseñador de Azure Machine Learning (versión preliminar), consulte [Registro de las métricas en el diseñador (versión preliminar)](how-to-track-designer-experiments.md)

## <a name="example-notebooks"></a>Cuadernos de ejemplo

Los cuadernos siguientes muestran los conceptos de este artículo:
* [how-to-use-azureml/training/train-on-local](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-on-local)
* [how-to-use-azureml/track-and-monitor-experiments/logging-api](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/logging-api)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

Consulte estos artículos para aprender a usar Machine Learning:

* Aprenda a [registrar métricas en el diseñador de Azure Machine Learning (versión preliminar)](how-to-track-designer-experiments.md).

* Vea un ejemplo de cómo registrar el mejor modelo e implementarlo en el tutorial, [Entrenar un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).
