---
title: Uso de Studio para implementar modelos entrenados en el diseñador
titleSuffix: Azure Machine Learning
description: Use Azure Machine Learning Studio para implementar modelos entrenados en el diseñador.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: keli19
author: likebupt
ms.reviewer: peterlu
ms.date: 09/04/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 95b41723d3cb398caad3a0cf388b7810deda78dc
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932121"
---
# <a name="use-the-studio-to-deploy-models-trained-in-the-designer"></a>Uso de Studio para implementar modelos entrenados en el diseñador

En este artículo, aprenderá a implementar un modelo entrenado desde el diseñador como un punto de conexión en tiempo real en Azure Machine Learning Studio.

La implementación en Studio consta de los siguientes pasos:

1. Registro del modelo entrenado.
1. Descarga del script de entrada y el archivo de dependencias de Conda para el modelo.
1. Implementación del modelo en el destino de proceso.

También puede implementar modelos directamente en el diseñador para omitir los pasos de registro de modelos y descarga de archivos. Esto puede ser útil para una rápida implementación. Para más información, consulte [Implementación de un modelo con el diseñador](tutorial-designer-automobile-price-deploy.md).

Los modelos entrenados en el diseñador también se pueden implementar a través del SDK o la interfaz de la línea de comandos (CLI). Para más información, consulte [Implementación del modelo existente con Azure Machine Learning](how-to-deploy-existing-model.md).

## <a name="prerequisites"></a>Requisitos previos

* [Un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md)

* Una canalización de entrenamiento completada que contenga un [módulo Entrenar modelo](./algorithm-module-reference/train-model.md)

## <a name="register-the-model"></a>Registro del modelo

Una vez finalizada la canalización de entrenamiento, registre el modelo entrenado en el área de trabajo de Azure Machine Learning para tener acceso al modelo en otros proyectos.

1. Seleccione el módulo [Entrenar modelo](./algorithm-module-reference/train-model.md).
1. Seleccione la pestaña **Resultados y registros**.
1. Seleccione el icono **Registrar modelo** ![Captura de pantalla del icono de engranaje](./media/how-to-deploy-model-designer/register-model-icon.png).

    ![Captura de pantalla del panel derecho del módulo Entrenar modelo](./media/how-to-deploy-model-designer/train-model-right-pane.png)

1. Escriba un nombre para el modelo y, a continuación, seleccione **Guardar**.

Después de registrar el modelo, puede encontrarlo en la página de recursos de **Modelos** en Studio.
    
![Captura de pantalla del modelo registrado en la página de recursos de Modelos](./media/how-to-deploy-model-designer/models-asset-page.png)


## <a name="download-the-entry-script-file-and-conda-dependencies-file"></a>Descarga del archivo de script de entrada y el archivo de dependencias de Conda

Necesita los siguientes archivos para implementar un modelo en Azure Machine Learning Studio:

- **Archivo de script de entrada**: carga el modelo entrenado, procesa los datos de entrada de las solicitudes, realiza inferencias en tiempo real y devuelve el resultado. El diseñador genera automáticamente un archivo de script de entrada `score.py` cuando se completa el módulo **Entrenar modelo**.

- **Archivo de dependencias de Conda**: especifica de qué paquetes de PIP y Conda depende su servicio web. El diseñador crea automáticamente un archivo `conda_env.yaml` cuando se completa el módulo **Entrenar modelo**.

Puede descargar estos dos archivos en el panel derecho del módulo **Entrenar modelo**:

1. Seleccione el módulo **Train Model** (Entrenar modelo).
1. En la pestaña **Resultados y registros**, seleccione la carpeta `trained_model_outputs`.
1. Descargue los archivos `conda_env.yaml` y `score.py`.

    ![Captura de pantalla de archivos de descarga para la implementación en el panel derecho](./media/how-to-deploy-model-designer/download-artifacts-in-right-pane.png)

Como alternativa, puede descargar los archivos desde la página de recursos de **Modelos** después de registrar el modelo:

1. Vaya a la página de recursos de **Modelos**.
1. Seleccione el modelo que quiere implementar.
1. Seleccione la pestaña **Artefactos**.
1. Seleccione la carpeta `trained_model_outputs` .
1. Descargue los archivos `conda_env.yaml` y `score.py`.  

    ![Captura de pantalla de archivos de descarga para la implementación en la página de detalles del modelo](./media/how-to-deploy-model-designer/download-artifacts-in-models-page.png)

> [!NOTE]
> El archivo `score.py` proporciona prácticamente la misma funcionalidad que los módulos del **Modelo de puntuación**. Sin embargo, algunos módulos, como [Puntuación del recomendador SVD](./algorithm-module-reference/score-svd-recommender.md), [Puntuación del recomendador Wide and Deep](./algorithm-module-reference/score-wide-and-deep-recommender.md) y [Puntuación del modelo de Vowpal Wabbit](./algorithm-module-reference/score-vowpal-wabbit-model.md) tienen parámetros para diferentes modos de puntuación. También puede cambiar esos parámetros en el script de entrada.
>
>Para obtener más información sobre la configuración de parámetros en el archivo `score.py`, vea la sección [Configuración del script de entrada](#configure-the-entry-script).

## <a name="deploy-the-model"></a>Implementación del modelo

Después de descargar los archivos necesarios, ya puede implementar el modelo.

1. En la página de recursos de **Modelos**, seleccione el modelo registrado.
1. Seleccione el botón **Implementar**.
1. En el menú de configuración, escriba la siguiente información:

    - Escriba un nombre para el punto de conexión.
    - Seleccione implementar el modelo en [Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md) o [Azure Container Instances](how-to-deploy-azure-container-instance.md).
    - Cargue `score.py` para el **archivo de script de entrada**.
    - Cargue `conda_env.yml` para el **archivo de dependencias de Conda**. 

    >[!TIP]
    > En la configuración **avanzada**, puede establecer la capacidad de CPU y memoria y otros parámetros para la implementación. Esta configuración es importante para ciertos modelos, como los modelos PyTorch, que consumen una cantidad considerable de memoria (aproximadamente 4 GB).

1. Seleccione **Implementar** para implementar el modelo como un punto de conexión en tiempo real.

    ![Captura de pantalla de la página de implementación del modelo en el recurso del modelo](./media/how-to-deploy-model-designer/deploy-model.png)

## <a name="consume-the-real-time-endpoint"></a>Consumo del punto de conexión en tiempo real

Una vez que la implementación se realiza correctamente, puede encontrar el punto de conexión en tiempo real en la página de recursos de **Puntos de conexión**. Una vez allí, encontrará un punto de conexión de REST, que los clientes pueden usar para enviar solicitudes al punto de conexión en tiempo real. 

> [!NOTE]
> El diseñador también genera un archivo JSON de datos de ejemplo para las pruebas. Puede descargar `_samples.json` en la carpeta **trained_model_outputs**.

Use el siguiente ejemplo de código para consumir un punto de conexión en tiempo real.

```python

import json
from pathlib import Path
from azureml.core.workspace import Workspace, Webservice
 
service_name = 'YOUR_SERVICE_NAME'
ws = Workspace.get(
    name='WORKSPACE_NAME',
    subscription_id='SUBSCRIPTION_ID',
    resource_group='RESOURCEGROUP_NAME'
)
service = Webservice(ws, service_name)
sample_file_path = '_samples.json'
 
with open(sample_file_path, 'r') as f:
    sample_data = json.load(f)
score_result = service.run(json.dumps(sample_data))
print(f'Inference result = {score_result}')
```

## <a name="configure-the-entry-script"></a>Configuración del script de entrada

Algunos módulos, como [Puntuación del recomendador SVD](./algorithm-module-reference/score-svd-recommender.md), [Puntuación del recomendador Wide and Deep](./algorithm-module-reference/score-wide-and-deep-recommender.md) y [Puntuación del modelo de Vowpal Wabbit](./algorithm-module-reference/score-vowpal-wabbit-model.md) tienen parámetros para diferentes modos de puntuación. En esta sección, aprenderá a actualizar estos parámetros también en el archivo de script de entrada.

En el ejemplo siguiente se actualiza el comportamiento predeterminado de un modelo de **Recomendador Wide & Deep**. De manera predeterminada, el archivo `score.py` indica al servicio web que prediga las clasificaciones entre los usuarios y los elementos. 

Puede modificar el archivo de script de entrada para hacer recomendaciones de elementos y devolver los elementos recomendados cambiando el parámetro `recommender_prediction_kind`.


```python
import os
import json
from pathlib import Path
from collections import defaultdict
from azureml.studio.core.io.model_directory import ModelDirectory
from azureml.designer.modules.recommendation.dnn.wide_and_deep.score. \
    score_wide_and_deep_recommender import ScoreWideAndDeepRecommenderModule
from azureml.designer.serving.dagengine.utils import decode_nan
from azureml.designer.serving.dagengine.converter import create_dfd_from_dict

model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'trained_model_outputs')
schema_file_path = Path(model_path) / '_schema.json'
with open(schema_file_path) as fp:
    schema_data = json.load(fp)


def init():
    global model
    model = ModelDirectory.load(load_from_dir=model_path)


def run(data):
    data = json.loads(data)
    input_entry = defaultdict(list)
    for row in data:
        for key, val in row.items():
            input_entry[key].append(decode_nan(val))

    data_frame_directory = create_dfd_from_dict(input_entry, schema_data)

    # The parameter names can be inferred from Score Wide and Deep Recommender module parameters:
    # convert the letters to lower cases and replace whitespaces to underscores.
    score_params = dict(
        trained_wide_and_deep_recommendation_model=model,
        dataset_to_score=data_frame_directory,
        training_data=None,
        user_features=None,
        item_features=None,
        ################### Note #################
        # Set 'Recommender prediction kind' parameter to enable item recommendation model
        recommender_prediction_kind='Item Recommendation',
        recommended_item_selection='From All Items',
        maximum_number_of_items_to_recommend_to_a_user=5,
        whether_to_return_the_predicted_ratings_of_the_items_along_with_the_labels='True')
    result_dfd, = ScoreWideAndDeepRecommenderModule().run(**score_params)
    result_df = result_dfd.data
    return json.dumps(result_df.to_dict("list"))
```

En el caso de los modelos de **Recomendador Wide & Deep** y **Vowpal Wabbit**, puede configurar el parámetro de modo de puntuación mediante los métodos siguientes:

- Los nombres de parámetro son las combinaciones de minúsculas y caracteres de subrayado de los nombres de parámetro de [puntuación del modelo de Vowpal Wabbit](./algorithm-module-reference/score-vowpal-wabbit-model.md) y [puntuación del recomendador Wide and Deep](./algorithm-module-reference/score-wide-and-deep-recommender.md).
- Los valores de parámetro de tipo de modo son cadenas de los nombres de opción correspondientes. Tome como ejemplo el **tipo de predicción de recomendador** en los códigos anteriores. El valor puede ser `'Rating Prediction'`o `'Item Recommendation'`. No se permiten otros valores.

Para el modelo entrenado **Recomendador SVD**, los nombres y valores de parámetro quizás sean menos obvios. Puede buscar las tablas siguientes para decidir cómo establecer los parámetros.

| Nombre del parámetro de [Puntuación del recomendador SVD](./algorithm-module-reference/score-svd-recommender.md)                           | Nombre del parámetro del archivo de script de entrada |
| ------------------------------------------------------------ | --------------------------------------- |
| Tipo de predicción de recomendador                                  | prediction_kind                         |
| Selección de elementos recomendados                                   | recommended_item_selection              |
| Tamaño mínimo de la agrupación de recomendación para un solo usuario    | min_recommendation_pool_size            |
| Número máximo de elementos para recomendar a un usuario               | max_recommended_item_count              |
| Si se van a devolver las clasificaciones previstas de los elementos junto con las etiquetas | return_ratings                          |

En el código siguiente se muestra cómo establecer los parámetros de un recomendador SVD que usa los seis parámetros para recomendar elementos clasificados con las clasificaciones previstas adjuntas.

```python
score_params = dict(
        learner=model,
        test_data=DataTable.from_dfd(data_frame_directory),
        training_data=None,
        # RecommenderPredictionKind has 2 members, 'RatingPrediction' and 'ItemRecommendation'. You
        # can specify prediction_kind parameter with one of them.
        prediction_kind=RecommenderPredictionKind.ItemRecommendation,
        # RecommendedItemSelection has 3 members, 'FromAllItems', 'FromRatedItems', 'FromUndatedItems'.
        # You can specify recommended_item_selection parameter with one of them.
        recommended_item_selection=RecommendedItemSelection.FromRatedItems,
        min_recommendation_pool_size=1,
        max_recommended_item_count=3,
        return_ratings=True,
    )
```


## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento de un modelo en el diseñador](tutorial-designer-automobile-price-train-score.md)
* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Creación de aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md)
* [Actualización de servicios web](how-to-deploy-update-web-service.md)
