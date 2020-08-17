---
title: Depuración y solución de problemas de ParallelRunStep
titleSuffix: Azure Machine Learning
description: Depure y solucione los problemas de ParallelRunStep en las canalizaciones de aprendizaje automático del SDK de Azure Machine Learning para Python. Aprenda sobre los errores comunes en el desarrollo de las canalizaciones, y consulte sugerencias que le ayudarán a depurar los scripts antes y durante la ejecución remota.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: troubleshooting
ms.reviewer: jmartens, larryfr, vaidyas, laobri, tracych
ms.author: trmccorm
author: tmccrmck
ms.date: 07/16/2020
ms.openlocfilehash: 16366d9f3be1144a7588ceb9133fb4e2e60db95c
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373715"
---
# <a name="debug-and-troubleshoot-parallelrunstep"></a>Depuración y solución de problemas de ParallelRunStep
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a depurar y resolver los problemas de la clase [ParallelRunStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallel_run_step.parallelrunstep?view=azure-ml-py) con el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).

## <a name="testing-scripts-locally"></a>Prueba de scripts de forma local

Consulte la [sección Prueba de scripts de forma local](how-to-debug-pipelines.md#testing-scripts-locally) en el artículo sobre canalizaciones de aprendizaje automático. La instancia de ParallelRunStep se ejecuta como un paso en las canalizaciones de Machine Learning, por lo que la misma respuesta se aplica en ambos casos.

## <a name="debugging-scripts-from-remote-context"></a>Depuración de scripts desde un contexto remoto

La transición de la depuración de un script de puntuación de forma local, a la depuración de un script de puntuación en una canalización real puede resultar difícil. Para información sobre cómo buscar los registros en el portal, consulte la [sección de canalizaciones de aprendizaje automático en la depuración de scripts desde un contexto remoto](how-to-debug-pipelines.md#debugging-scripts-from-remote-context). La información de esa sección también se aplica a ParallelRunStep.

Por ejemplo, el archivo de registro `70_driver_log.txt` contiene información del controlador que inicia el código de ParallelRunStep.

Dada la naturaleza distribuida de los trabajos de ParallelRunStep, hay registros de distintos orígenes. Sin embargo, se crean dos archivos consolidados que proporcionan información de alto nivel:

- `~/logs/overview.txt`: Este archivo proporciona información de alto nivel sobre el número de minilotes (también conocidos como tareas) que se han creado hasta el momento y el número de minilotes que se han procesado hasta ahora. En este extremo, se muestra el resultado del trabajo. Si se produjo un error en el trabajo, se mostrará el mensaje de error y dónde se debe iniciar la solución de problemas.

- `~/logs/sys/master.txt`: Este archivo proporciona la vista del nodo principal (también conocido como orquestador) del trabajo en ejecución. Incluye la creación de tareas, la supervisión del progreso y el resultado de la ejecución.

Los registros generados a partir del script de entrada mediante el asistente EntryScript y las instrucciones print se encuentran en los siguientes archivos:

- `~/logs/user/<ip_address>/<node_name>.log.txt`: estos archivos son los registros escritos desde entry_script con el asistente EntryScript. También contiene la instrucción print (stdout) de entry_script.

Para obtener una descripción concisa de los errores del script, hay lo siguiente:

- `~/logs/user/error.txt`: este archivo intentará resumir los errores del script.

Para obtener más información sobre los errores del script, hay lo siguiente:

- `~/logs/user/error/`: contiene todos los errores producidos y los seguimientos de la pila completos organizados por nodo.

Cuando necesite comprender en detalle cómo ejecuta cada nodo el script de puntuación, examine los registros de proceso individuales para cada nodo. Los registros de proceso se pueden encontrar en la carpeta `sys/node`, agrupados por nodos de trabajo:

- `~/logs/sys/node/<node_name>.txt`: este archivo proporciona información detallada sobre cada uno de los minilotes a medida que un trabajo lo elige o lo completa. Para cada minilote, este archivo incluye:

    - La dirección IP y el PID del proceso de trabajo. 
    - El número total de elementos, el número de elementos procesados correctamente y el número de elementos con errores.
    - Hora de inicio, duración, tiempo de proceso y tiempo del método de ejecución.

También puede encontrar información sobre el uso de recursos de los procesos para cada trabajo. Esta información está en formato CSV y se encuentra en `~/logs/sys/perf/overview.csv`. En `~logs/sys/processes.csv` encontrará información sobre cada proceso.

### <a name="how-do-i-log-from-my-user-script-from-a-remote-context"></a>¿Cómo me puedo registrar desde mi script de usuario en un contexto remoto?
ParallelRunStep puede ejecutar varios procesos en un nodo basado en process_count_per_node. Para organizar los registros de cada proceso en el nodo y combinar las instrucciones print y log, se recomienda usar el registrador ParallelRunStep como se muestra a continuación. Puede obtener un registrador de EntryScript, y hacer que los registros aparezcan en la carpeta **logs/user** del portal.

**Uso del registrador por un script de entrada de ejemplo:**
```python
from azureml_user.parallel_run import EntryScript

def init():
    """ Initialize the node."""
    entry_script = EntryScript()
    logger = entry_script.logger
    logger.debug("This will show up in files under logs/user on the Azure portal.")


def run(mini_batch):
    """ Accept and return the list back."""
    # This class is in singleton pattern and will return same instance as the one in init()
    entry_script = EntryScript()
    logger = entry_script.logger
    logger.debug(f"{__file__}: {mini_batch}.")
    ...

    return mini_batch
```

### <a name="how-could-i-pass-a-side-input-such-as-a-file-or-files-containing-a-lookup-table-to-all-my-workers"></a>¿Cómo puedo pasar una entrada lateral, como un archivo o archivos que contienen una tabla de búsqueda, a todos los trabajadores?

El usuario puede pasar datos de referencia al script mediante el parámetro side_inputs de ParalleRunStep. Todos los conjuntos de datos proporcionados como side_inputs se montarán en cada nodo de trabajo. El usuario puede obtener la ubicación del montaje pasando el argumento.

Construya un [conjunto de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py) que contenga los datos de referencia y regístrelo con su área de trabajo. Páselo al parámetro `side_inputs` de `ParallelRunStep`. Además, puede agregar su ruta de acceso en la sección `arguments` para acceder fácilmente a su ruta de acceso montada:

```python
label_config = label_ds.as_named_input("labels_input")
batch_score_step = ParallelRunStep(
    name=parallel_step_name,
    inputs=[input_images.as_named_input("input_images")],
    output=output_dir,
    arguments=["--labels_dir", label_config],
    side_inputs=[label_config],
    parallel_run_config=parallel_run_config,
)
```

Una vez que pueda acceder a él en su script inferencia [por ejemplo, en el método init()] como sigue:

```python
parser = argparse.ArgumentParser()
parser.add_argument('--labels_dir', dest="labels_dir", required=True)
args, _ = parser.parse_known_args()

labels_path = args.labels_dir
```

## <a name="next-steps"></a>Pasos siguientes

* Vea la referencia del SDK para obtener ayuda con el paquete [azureml-pipeline-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps?view=azure-ml-py). Vea la [documentación](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallelrunstep?view=azure-ml-py) de referencia de la clase ParallelRunStep.

* Realice el [tutorial avanzado](tutorial-pipeline-batch-scoring-classification.md) sobre cómo usar canalizaciones con ParallelRunStep. En este tutorial se explica cómo pasar otro archivo como entrada lateral. 
