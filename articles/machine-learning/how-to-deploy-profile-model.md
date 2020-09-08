---
title: Uso de CPU y memoria del modelo de perfil
titleSuffix: Azure Machine Learning
description: Aprenda a generar perfiles de uso de CPU y memoria del modelo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
zone_pivot_groups: aml-control-methods
ms.openlocfilehash: a3aed23441df225316f52eb3acb1387cbba6d807
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935592"
---
# <a name="profile-your-model-to-determine-resource-utilization"></a>Generar perfiles del modelo para determinar el uso de recursos

En este artículo se muestra cómo generar perfiles de modelo de aprendizaje automático para determinar la cantidad de CPU y memoria que se debe asignar al modelo cuando se implementa como servicio web.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya ha entrenado y registrado un modelo con Azure Machine Learning. Vea [este tutorial de muestra](how-to-train-scikit-learn.md) para obtener un ejemplo de entrenamiento y registro de un modelo scikit-learn con Azure Machine Learning.

## <a name="run-the-profiler"></a>Ejecución del generador de perfiles

Una vez que haya registrado el modelo y preparado los otros componentes necesarios para su implementación, puede determinar la CPU y la memoria que necesitará el servicio implementado. La generación de perfiles prueba el servicio que ejecuta el modelo y devuelve información como el uso de la CPU, el uso de memoria y la latencia de respuesta. También proporciona una recomendación para la CPU y la memoria en función del uso de recursos.

Para generar un perfil del modelo, necesitará lo siguiente:
* Un modelo registrado.
* Una configuración de inferencia basada en el script de entrada y en la definición de entorno de inferencia.
* Un conjunto de datos tabular de una sola columna, donde cada fila contiene una cadena que representa datos de solicitud de ejemplo.

> [!IMPORTANT]
> En este momento, solo se admite la generación de perfiles de servicios que esperan que sus datos de solicitud sean una cadena, por ejemplo: JSON serializado en cadena, texto, imagen serializada en cadena, etc. El contenido de cada fila del conjunto de resultados (cadena) se colocará en el cuerpo de la solicitud HTTP y se enviará al servicio que encapsula el modelo para la puntuación.

> [!IMPORTANT]
> Solo se admite la generación de perfiles de 2 CPU como máximo en la región USGovArizona y ChinaEast2.

A continuación se muestra un ejemplo de cómo se puede construir un conjunto de datos de entrada para generar perfiles de un servicio que espera que los datos de solicitudes entrantes contengan JSON serializado. En este caso, creamos un conjunto de datos basado en 100 instancias del mismo contenido de datos de la solicitud. En escenarios reales, se recomienda usar conjuntos de datos más grandes que contengan varias entradas, especialmente si el comportamiento o el uso de recursos del modelo depende de la entrada.

::: zone pivot="py-sdk"

```python
import json
from azureml.core import Datastore
from azureml.core.dataset import Dataset
from azureml.data import dataset_type_definitions

input_json = {'data': [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
                       [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]]}
# create a string that can be utf-8 encoded and
# put in the body of the request
serialized_input_json = json.dumps(input_json)
dataset_content = []
for i in range(100):
    dataset_content.append(serialized_input_json)
dataset_content = '\n'.join(dataset_content)
file_name = 'sample_request_data.txt'
f = open(file_name, 'w')
f.write(dataset_content)
f.close()

# upload the txt file created above to the Datastore and create a dataset from it
data_store = Datastore.get_default(ws)
data_store.upload_files(['./' + file_name], target_path='sample_request_data')
datastore_path = [(data_store, 'sample_request_data' +'/' + file_name)]
sample_request_data = Dataset.Tabular.from_delimited_files(
    datastore_path, separator='\n',
    infer_column_types=True,
    header=dataset_type_definitions.PromoteHeadersBehavior.NO_HEADERS)
sample_request_data = sample_request_data.register(workspace=ws,
                                                   name='sample_request_data',
                                                   create_new_version=True)
```

Una vez que tenga listo el conjunto de datos que contiene los datos de solicitud de ejemplo, cree una configuración de inferencia. La configuración de inferencia se basa en score.py y en la definición de entorno. En el siguiente ejemplo le indicaremos cómo crear una configuración de inferencia y ejecutar la generación de perfiles:

```python
from azureml.core.model import InferenceConfig, Model
from azureml.core.dataset import Dataset


model = Model(ws, id=model_id)
inference_config = InferenceConfig(entry_script='path-to-score.py',
                                   environment=myenv)
input_dataset = Dataset.get_by_name(workspace=ws, name='sample_request_data')
profile = Model.profile(ws,
            'unique_name',
            [model],
            inference_config,
            input_dataset=input_dataset)

profile.wait_for_completion(True)

# see the result
details = profile.get_details()
```

::: zone-end

::: zone pivot="cli"


El comando siguiente muestra cómo generar el perfil de un modelo mediante la CLI:

```azurecli-interactive
az ml model profile -g <resource-group-name> -w <workspace-name> --inference-config-file <path-to-inf-config.json> -m <model-id> --idi <input-dataset-id> -n <unique-name>
```

> [!TIP]
> Para conservar la información que devuelve la generación de perfiles, use las etiquetas o las propiedades del modelo. El uso de etiquetas o propiedades almacena los datos con el modelo en el registro del modelo. En los siguientes ejemplos se muestra cómo agregar una nueva etiqueta que contiene la información de `requestedCpu` y `requestedMemoryInGb`:
>
> ```python
> model.add_tags({'requestedCpu': details['requestedCpu'],
>                 'requestedMemoryInGb': details['requestedMemoryInGb']})
> ```
>
> ```azurecli-interactive
> az ml model profile -g <resource-group-name> -w <workspace-name> --i <model-id> --add-tag requestedCpu=1 --add-tag requestedMemoryInGb=0.5
> ```

::: zone-end

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

