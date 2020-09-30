---
title: Publicación de canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Ejecute flujos de trabajo de aprendizaje automático con canalizaciones de aprendizaje automático y el SDK de Azure Machine Learning para Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: laobri
author: lobrien
ms.date: 8/25/2020
ms.topic: conceptual
ms.custom: how-to, contperfq1
ms.openlocfilehash: ddc8186e85001a2a3ed2ed9f57b8f025133ef16a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897759"
---
# <a name="publish-and-track-machine-learning-pipelines"></a>Publicación y seguimiento de canalizaciones de aprendizaje automático



En este artículo se muestra cómo compartir una canalización de aprendizaje automático con sus compañeros o clientes.

Las canalizaciones de aprendizaje automático son flujos de trabajo reutilizables para las tareas de aprendizaje automático. Una ventaja de las canalizaciones es aumentar la colaboración. También puede usar canalizaciones de versión, lo que permite a los clientes utilizar el modelo actual mientras trabaja en una nueva versión. 

## <a name="prerequisites"></a>Requisitos previos

* Cree un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) que contenga todos los recursos de canalización.

* [Configure su entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning o use una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md) con el SDK ya instalado.

* Cree y ejecute una canalización de aprendizaje automático, como en el siguiente [Tutorial: Compilación de una canalización de Azure Machine Learning para la puntuación por lotes](tutorial-pipeline-batch-scoring-classification.md). Para otras opciones, consulte [Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning](how-to-create-your-first-pipeline.md).

## <a name="publish-a-pipeline"></a>Publicar una canalización

Una vez que tenga una canalización en funcionamiento, puede publicarla para que se ejecute con diferentes entradas. Para que el punto de conexión REST de una canalización ya publicada acepte parámetros, tiene que configurar la canalización para que use objetos `PipelineParameter` para los argumentos que varían.

1. Para crear un parámetro de canalización, use un objeto [PipelineParameter](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.pipelineparameter?view=azure-ml-py&preserve-view=true) con un valor predeterminado.

   ```python
   from azureml.pipeline.core.graph import PipelineParameter
   
   pipeline_param = PipelineParameter(
     name="pipeline_arg",
     default_value=10)
   ```

2. Agregue este objeto `PipelineParameter` como parámetro a cualquiera de los pasos de la canalización tal como se muestra a continuación:

   ```python
   compareStep = PythonScriptStep(
     script_name="compare.py",
     arguments=["--comp_data1", comp_data1, "--comp_data2", comp_data2, "--output_data", out_data3, "--param1", pipeline_param],
     inputs=[ comp_data1, comp_data2],
     outputs=[out_data3],
     compute_target=compute_target,
     source_directory=project_folder)
   ```

3. Publique esta canalización para que acepte un parámetro cuando se invoque.

   ```python
   published_pipeline1 = pipeline_run1.publish_pipeline(
        name="My_Published_Pipeline",
        description="My Published Pipeline Description",
        version="1.0")
   ```

## <a name="run-a-published-pipeline"></a>Ejecutar una canalización publicada

Todas las canalizaciones publicadas tienen un punto de conexión REST. Con el punto de conexión de canalización, puede desencadenar una ejecución de la canalización desde cualquier sistema externo, incluidos los clientes que no sean de Python. Este punto de conexión habilita la "repetibilidad administrada" en los escenarios de puntuación y nuevo entrenamiento.

Para invocar el proceso de ejecución de la canalización anterior, necesitará un token de encabezado de autenticación de Azure Active Directory. La obtención de este tipo de token se describe en la referencia de [clase AzureCliAuthentication](https://docs.microsoft.com/python/api/azureml-core/azureml.core.authentication.azurecliauthentication?view=azure-ml-py&preserve-view=true) y en el cuaderno [Autenticación en Azure Machine Learning](https://aka.ms/pl-restep-auth).

```python
from azureml.pipeline.core import PublishedPipeline
import requests

response = requests.post(published_pipeline1.endpoint,
                         headers=aad_token,
                         json={"ExperimentName": "My_Pipeline",
                               "ParameterAssignments": {"pipeline_arg": 20}})
```

## <a name="create-a-versioned-pipeline-endpoint"></a>Creación de un punto de conexión de canalización con versiones

Puede crear un punto de conexión de canalización con varias canalizaciones publicadas detrás. Esto le proporciona un punto de conexión de REST fijo a medida que se recorre en iteración y se actualizan las canalizaciones de ML.

```python
from azureml.pipeline.core import PipelineEndpoint

published_pipeline = PipelineEndpoint.get(workspace=ws, name="My_Published_Pipeline")
pipeline_endpoint = PipelineEndpoint.publish(workspace=ws, name="PipelineEndpointTest",
                                            pipeline=published_pipeline, description="Test description Notebook")
```

## <a name="submit-a-job-to-a-pipeline-endpoint"></a>Envío de un trabajo a un punto de conexión de canalización

Puede enviar un trabajo a la versión predeterminada de un punto de conexión de canalización:

```python
pipeline_endpoint_by_name = PipelineEndpoint.get(workspace=ws, name="PipelineEndpointTest")
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment")
print(run_id)
```

También puede enviar un trabajo a una versión específica:

```python
run_id = pipeline_endpoint_by_name.submit("PipelineEndpointExperiment", pipeline_version="0")
print(run_id)
```

Lo mismo puede hacerse mediante la API REST:

```python
rest_endpoint = pipeline_endpoint_by_name.endpoint
response = requests.post(rest_endpoint, 
                         headers=aad_token, 
                         json={"ExperimentName": "PipelineEndpointExperiment",
                               "RunSource": "API",
                               "ParameterAssignments": {"1": "united", "2":"city"}})
```

## <a name="use-published-pipelines-in-the-studio"></a>Uso de canalizaciones publicadas en Studio

También puede ejecutar una canalización publicada desde Studio:

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. [Vea su área de trabajo](how-to-manage-workspace.md#view).

1. En el panel izquierdo, seleccione **Puntos de conexión**.

1. En la parte superior, seleccione **Pipeline endpoints** (Puntos de conexión de la canalización).
 ![lista de las canalizaciones publicadas de Machine Learning](./media/how-to-create-your-first-pipeline/pipeline-endpoints.png)

1. Seleccione una canalización específica para ejecutar, utilizar o revisar los resultados de las ejecuciones anteriores del punto de conexión de la canalización.

## <a name="disable-a-published-pipeline"></a>Deshabilitar una canalización publicada

Para ocultar una canalización de la lista de canalizaciones publicadas hay que deshabilitarla, ya sea en Studio o en el SDK:

```python
# Get the pipeline by using its ID from Azure Machine Learning studio
p = PublishedPipeline.get(ws, id="068f4885-7088-424b-8ce2-eeb9ba5381a6")
p.disable()
```

Puede habilitarla de nuevo con `p.enable()`. Para más información, consulte la referencia de la [clase PublishedPipeline](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.publishedpipeline?view=azure-ml-py&preserve-view=true).

## <a name="next-steps"></a>Pasos siguientes

- Use [estos cuadernos de Jupyter en GitHub](https://aka.ms/aml-pipeline-readme) para explorar aún más canalizaciones de Machine Learning.
- Consulte la ayuda de referencia del SDK para el paquete [azureml-pipelines-core](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py&preserve-view=true) y el paquete [azureml-pipelines-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py&preserve-view=true).
- Consulte los [procedimientos](how-to-debug-pipelines.md) para obtener sugerencias sobre la depuración y la solución de problemas de canalizaciones.
