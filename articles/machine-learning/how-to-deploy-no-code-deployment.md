---
title: Implementación sin código (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a implementar un modelo sin un script de entrada.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
ms.openlocfilehash: d6c70f27eebc9d502ce7275603e99975c2efc267
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87544496"
---
# <a name="preview-no-code-model-deployment"></a>(Versión preliminar) Implementación de modelo sin código

La implementación del modelo sin código se encuentra actualmente en versión preliminar y admite los siguientes marcos de aprendizaje automático:

## <a name="tensorflow-savedmodel-format"></a>Formato Tensorflow SavedModel
Los modelos Tensorflow deben registrarse en **formato de SavedModel** para que funcionen con la implementación de modelos sin código.

Consulte [este vínculo](https://www.tensorflow.org/guide/saved_model) para obtener información sobre cómo crear un formato SavedModel.

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='flowers',                        # Name of the registered model in your workspace.
                       model_path='./flowers_model',                # Local Tensorflow SavedModel folder to upload and register as a model.
                       model_framework=Model.Framework.TENSORFLOW,  # Framework used to create the model.
                       model_framework_version='1.14.0',            # Version of Tensorflow used to create the model.
                       description='Flowers model')

service_name = 'tensorflow-flower-service'
service = Model.deploy(ws, service_name, [model])
```

## <a name="onnx-models"></a>Modelos de ONNX

El registro y la implementación del modelo ONNX se admiten en cualquier gráfico de inferencia de ONNX. Los pasos de preprocesamiento y postproceso no se admiten actualmente.

Este es un ejemplo de cómo registrar e implementar un modelo de MNIST ONNX:

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='mnist-sample',                  # Name of the registered model in your workspace.
                       model_path='mnist-model.onnx',              # Local ONNX model to upload and register as a model.
                       model_framework=Model.Framework.ONNX ,      # Framework used to create the model.
                       model_framework_version='1.3',              # Version of ONNX used to create the model.
                       description='Onnx MNIST model')

service_name = 'onnx-mnist-service'
service = Model.deploy(ws, service_name, [model])
```

Para puntuar un modelo, consulte [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](https://docs.microsoft.com/azure/machine-learning/how-to-consume-web-service). Muchos proyectos de ONNX usan archivos protobuf para almacenar de forma compacta datos de entrenamiento y validación, lo que puede hacer que sea difícil saber cuál es el formato de datos esperado por el servicio. Como desarrollador de modelos, debe documentar para los desarrolladores:

* El formato de entrada (JSON o binario)
* La forma y el tipo de los datos de entrada (por ejemplo, una matriz de floats de forma [100,100,3])
* Información de dominio (por ejemplo, para una imagen, el espacio de colores, el orden de los componentes y si los valores están normalizados)

Si usa PyTorch, el artículo [Exportación de modelos de PyTorch a ONNX](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb) tiene los detalles sobre la conversión y las limitaciones. 

## <a name="scikit-learn-models"></a>Modelos de Scikit-learn

Se admite la implementación de modelo sin código para todos los tipos de modelos integrados de Scikit-learn.

Este es un ejemplo de cómo registrar e implementar un modelo de sklearn sin código adicional:

```python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = Model.register(workspace=ws,
                       model_name='my-sklearn-model',                # Name of the registered model in your workspace.
                       model_path='./sklearn_regression_model.pkl',  # Local file to upload and register as a model.
                       model_framework=Model.Framework.SCIKITLEARN,  # Framework used to create the model.
                       model_framework_version='0.19.1',             # Version of scikit-learn used to create the model.
                       resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5),
                       description='Ridge regression model to predict diabetes progression.',
                       tags={'area': 'diabetes', 'type': 'regression'})
                       
service_name = 'my-sklearn-service'
service = Model.deploy(ws, service_name, [model])
```

> [!NOTE]
> Los modelos que admiten predict_proba usarán ese método de forma predeterminada. Para invalidar esta opción para usar la predicción, puede modificar el cuerpo de POST como se indica a continuación:

```python
import json


input_payload = json.dumps({
    'data': [
        [ 0.03807591,  0.05068012,  0.06169621, 0.02187235, -0.0442235,
         -0.03482076, -0.04340085, -0.00259226, 0.01990842, -0.01764613]
    ],
    'method': 'predict'  # If you have a classification model, the default behavior is to run 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

> [!NOTE]
> Estas dependencias se incluyen en el contenedor de inferencia Scikit-learn integrado previamente:

```yaml
    - dill
    - azureml-defaults
    - inference-schema[numpy-support]
    - scikit-learn
    - numpy
    - joblib
    - pandas
    - scipy
    - sklearn_pandas
```
## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Creación de aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md)
* [Actualización de servicios web](how-to-deploy-update-web-service.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)
