---
title: Programación de canalizaciones de Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Programe canalizaciones de Azure Machine Learning con el SDK de Azure Machine Learning para Python. Las canalizaciones programadas le permiten automatizar las tareas rutinarias y lentas, como el procesamiento de datos, el entrenamiento y la supervisión.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 11/12/2019
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 351d52128660c25c557c536f2835e0e71c0115f7
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87842967"
---
# <a name="schedule-machine-learning-pipelines-with-azure-machine-learning-sdk-for-python"></a>Programación de canalizaciones de aprendizaje automático con el SDK de Azure Machine Learning para Python

En este artículo, aprenderá a programar mediante programación una canalización para que se ejecute en Azure. Puede optar por crear una programación basada en el tiempo transcurrido o en los cambios en el sistema de archivos. Las programaciones basadas en el tiempo se pueden usar para encargarse de tareas rutinarias, como la supervisión del desfase de datos. Las programaciones basadas en los cambios se pueden usar para reaccionar ante cambios irregulares o imprevisibles, como la carga de nuevos datos o la edición de datos antiguos. Después de aprender a crear programaciones, aprenderá a recuperarlas y desactivarlas.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree).

* Entorno de Python en el que está instalado el SDK de Azure Machine Learning para Python. Para más información, consulte [Creación y administración de entornos reutilizables para aprendizaje e implementación con Azure Machine Learning](how-to-use-environments.md).

* Un área de trabajo Machine Learning con una canalización publicada. Puede usar la creada en [Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning](how-to-create-your-first-pipeline.md).

## <a name="initialize-the-workspace--get-data"></a>Inicialización del área de trabajo y obtención de los datos

Para programar una canalización, necesitará una referencia al área de trabajo, el identificador de la canalización publicada y el nombre del experimento en el que quiere crear la programación. Puede obtener estos valores con el código siguiente:

```Python
import azureml.core
from azureml.core import Workspace
from azureml.pipeline.core import Pipeline, PublishedPipeline
from azureml.core.experiment import Experiment

ws = Workspace.from_config()

experiments = Experiment.list(ws)
for experiment in experiments:
    print(experiment.name)

published_pipelines = PublishedPipeline.list(ws)
for published_pipeline in  published_pipelines:
    print(f"{published_pipeline.name},'{published_pipeline.id}'")

experiment_name = "MyExperiment" 
pipeline_id = "aaaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" 
```

## <a name="create-a-schedule"></a>Crear una programación

Para ejecutar una canalización de forma periódica, creará una programación. Un elemento `Schedule` asocia una canalización, un experimento y un desencadenador. El desencadenador puede ser un elemento `ScheduleRecurrence` que describa la espera entre ejecuciones o una ruta de acceso a un almacén de datos que especifique un directorio para inspeccionar los cambios. En cualquier caso, necesitará el identificador de la canalización y el nombre del experimento en el que se va a crear la programación.

En la parte superior del archivo de Python, importe las clases `Schedule` y `ScheduleRecurrence`:

```python

from azureml.pipeline.core.schedule import ScheduleRecurrence, Schedule
```

### <a name="create-a-time-based-schedule"></a>Creación de una programación basada en el tiempo

El constructor `ScheduleRecurrence` tiene un argumento `frequency` necesario que debe ser una de las cadenas siguientes: "Minute", "Hour", "Day" "Week" o "Month". También requiere un argumento `interval` de entero que especifique el número de unidades de `frequency` que debe transcurrir entre inicios de programación. Los argumentos opcionales le permiten ser más específico sobre las horas de inicio, como se detalla en los [documentos del SDK de ScheduleRecurrence](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.schedule.schedulerecurrence?view=azure-ml-py).

Cree un elemento `Schedule` que inicie una ejecución cada 15 minutos:

```python
recurrence = ScheduleRecurrence(frequency="Minute", interval=15)
recurring_schedule = Schedule.create(ws, name="MyRecurringSchedule", 
                            description="Based on time",
                            pipeline_id=pipeline_id, 
                            experiment_name=experiment_name, 
                            recurrence=recurrence)
```

### <a name="create-a-change-based-schedule"></a>Creación de una programación basada en los cambios

Las canalizaciones que se desencadenan por cambios en los archivos pueden ser más eficaces que las basadas en el tiempo. Por ejemplo, puede realizar un paso de preprocesamiento cuando cambie un archivo o cuando se agrega un nuevo archivo a un directorio de datos. Puede supervisar los cambios en un almacén de datos o aquellos que tienen lugar dentro de un directorio específico del almacén de datos. Si supervisa un directorio específico, los cambios en los subdirectorios de ese directorio _no_ desencadenarán una ejecución.

Para crear un elemento `Schedule` reactivo a los archivos, debe establecer el parámetro `datastore` en la llamada a [Schedule.create](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.schedule.schedule?view=azure-ml-py#create-workspace--name--pipeline-id--experiment-name--recurrence-none--description-none--pipeline-parameters-none--wait-for-provisioning-false--wait-timeout-3600--datastore-none--polling-interval-5--data-path-parameter-name-none--continue-on-step-failure-none--path-on-datastore-none---workflow-provider-none---service-endpoint-none-). Para supervisar una carpeta, establezca el argumento `path_on_datastore`.

El argumento `polling_interval` permite expresar (en minutos) la frecuencia con la que se comprueba si hay cambios en el almacén de datos.

Si la canalización se construyó con [DataPath](https://docs.microsoft.com/python/api/azureml-core/azureml.data.datapath.datapath?view=azure-ml-py) [PipelineParameter](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelineparameter?view=azure-ml-py), puede establecer esa variable en el nombre del archivo cambiado mediante el argumento `data_path_parameter_name`.

```python
datastore = Datastore(workspace=ws, name="workspaceblobstore")

reactive_schedule = Schedule.create(ws, name="MyReactiveSchedule", description="Based on input file change.",
                            pipeline_id=pipeline_id, experiment_name=experiment_name, datastore=datastore, data_path_parameter_name="input_data")
```

### <a name="optional-arguments-when-creating-a-schedule"></a>Argumentos opcionales al crear una programación

Además de los argumentos descritos anteriormente, puede establecer el argumento `status` en `"Disabled"` para crear una programación inactiva. Por último, `continue_on_step_failure` permite pasar un valor booleano que invalidará el comportamiento de error predeterminado de la canalización.

### <a name="use-azure-logic-apps-for-more-complex-workflows"></a>Uso de Azure Logic Apps para flujos de trabajo más complejos

Azure Logic Apps admite flujos de trabajo más complejos y se integra mucho más ampliamente que las canalizaciones de Azure Machine Learning. Consulte [Desencadenamiento de una ejecución de una canalización de Machine Learning desde una aplicación lógica](how-to-trigger-published-pipeline.md) para más información.

## <a name="view-your-scheduled-pipelines"></a>Visualización de las canalizaciones programadas

En el explorador web, vaya a Azure Machine Learning. En la sección **Endpoints** (Puntos de conexión) del panel de navegación, elija **Pipeline endpoints** (Puntos de conexión de la canalización). Esto le llevará a una lista de las canalizaciones publicadas en el área de trabajo.

![Página de canalizaciones de Azure Machine Learning](./media/how-to-schedule-pipelines/scheduled-pipelines.png)

En esta página puede ver información de resumen sobre todas las canalizaciones del área de trabajo: nombres, descripciones, estado, etc. Profundice haciendo clic en la canalización. En la página resultante, hay más detalles sobre la canalización y puede profundizar en cada una de las ejecuciones.

## <a name="deactivate-the-pipeline"></a>Desactivación de la canalización

Si tiene un elemento `Pipeline` publicado pero no programado, puede deshabilitarlo con:

```python
pipeline = PublishedPipeline.get(ws, id=pipeline_id)
pipeline.disable()
```

Si la canalización está programada, primero debe cancelar la programación. Recupere el identificador de la programación del portal o mediante la ejecución de:

```python
ss = Schedule.list(ws)
for s in ss:
    print(s)
```

Una vez que tenga el elemento `schedule_id` que quiere deshabilitar, ejecute:

```python
def stop_by_schedule_id(ws, schedule_id):
    s = next(s for s in Schedule.list(ws) if s.id == schedule_id)
    s.disable()
    return s

stop_by_schedule_id(ws, schedule_id)
```

Si vuelve a ejecutar `Schedule.list(ws)`, obtendrá una lista vacía.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha usado el SDK de Azure Machine Learning para Python para programar una canalización de dos maneras diferentes. Una programación se repite en función de la hora de reloj transcurrida. La otra programación se ejecuta si se modifica un archivo en un elemento `Datastore` especificado o en un directorio de ese almacén. Ha visto cómo usar el portal para examinar la canalización y las ejecuciones individuales. Por último, ha aprendido a deshabilitar una programación para que la canalización deje de ejecutarse.

Para más información, consulte:

> [!div class="nextstepaction"]
> [Uso de canalizaciones de Azure Machine Learning para la puntuación por lotes](tutorial-pipeline-batch-scoring-classification.md)

* Más información sobre las [canalizaciones](concept-ml-pipelines.md)
* Más información sobre la [exploración de Azure Machine Learning con Jupyter](samples-notebooks.md)

