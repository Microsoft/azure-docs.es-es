---
title: Supervisión y visualización de métricas y registros de ejecución de ML
titleSuffix: Azure Machine Learning
description: Supervise sus experimentos de Azure ML y consulte las métricas de ejecución para mejorar el proceso de creación de modelos. Use widgets y el portal de Studio para explorar el estado de ejecución y ver los registros de ejecución.
services: machine-learning
author: likebupt
ms.author: keli19
ms.reviewer: peterlu
ms.service: machine-learning
ms.subservice: core
ms.date: 07/30/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: d06186b2ce4d8bb9143663d41f03b9508e4bd00e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005962"
---
# <a name="monitor-and-view-ml-run-logs-and-metrics"></a>Supervisión y visualización de métricas y registros de ejecución de ML

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, aprenderá a supervisar las ejecuciones de Azure Machine Learning y a consultar sus registros. Para poder ver los registros, antes debe habilitarlos. Para obtener más información, consulte [Habilitación del registro en las ejecuciones de entrenamiento de Azure ML](how-to-track-experiments.md).

Los registros pueden ayudarle a diagnosticar los errores y advertencias, o a realizar un seguimiento de las métricas de rendimiento, como los parámetros y la precisión del modelo. En este artículo, aprenderá a ver los registros con los métodos siguientes:

> [!div class="checklist"]
> * Supervisar las ejecuciones de Studio
> * Supervisar las ejecuciones con el widget de Jupyter Notebook
> * Supervisar las ejecuciones automatizadas de aprendizaje automático
> * Ver registros de salida al finalizar
> * Ver registros de salida en Studio

Para obtener información general sobre cómo administrar los experimentos, consulte [Inicio, supervisión y cancelación de las ejecuciones de entrenamiento](how-to-manage-runs.md).

## <a name="monitor-runs-in-the-studio"></a>Supervisar las ejecuciones de Studio

Para supervisar las ejecuciones de un destino de proceso específico desde el explorador, siga estos pasos:

1. En [Azure Machine Learning Studio](https://ml.azure.com/), seleccione el área de trabajo y, a continuación, seleccione __Proceso__ en el lado izquierdo de la página.

1. Seleccione __Training Clusters__ (Clústeres de entrenamiento) para mostrar una lista de los destinos de proceso usados para el entrenamiento. Después, seleccione el clúster.

    ![Selección del clúster de entrenamiento](./media/how-to-track-experiments/select-training-compute.png)

1. Seleccione __Runs__ (Ejecuciones). Se muestra la lista de ejecuciones que usan este clúster. Para ver los detalles de una ejecución específica, use el vínculo de la columna __Run__ (Ejecución). Para ver los detalles del experimento, use el vínculo de la columna __Experiment__ (Experimento).

    ![Selección de ejecuciones para el clúster de entrenamiento](./media/how-to-track-experiments/show-runs-for-compute.png)
    
    > [!TIP]
    > Dado que los destinos de proceso de entrenamiento son un recurso compartido, pueden tener varias ejecuciones en cola o activas en un momento dado.
    > 
    > Una ejecución puede contener ejecuciones secundarias, por lo que un trabajo de entrenamiento puede dar lugar a varias entradas.

Una vez finalizada la ejecución, ya no se muestra en esta página. Para ver información sobre las ejecuciones completadas, visite la sección de __experimentos__ de Studio, seleccione el experimento y ejecútelo. Para obtener más información, consulte la sección [Visualización de métricas para las ejecuciones completadas](#view-the-experiment-in-the-web-portal).

## <a name="monitor-runs-using-the-jupyter-notebook-widget"></a>Supervisar las ejecuciones con el widget de Jupyter Notebooks

Cuando se usa el método **ScriptRunConfig** para enviar ejecuciones, se puede ver el progreso de la ejecución mediante el [widget de Jupyter](https://docs.microsoft.com/python/api/azureml-widgets/azureml.widgets?view=azure-ml-py). Al igual que el envío de ejecución, el widget es asincrónico y proporciona las actualizaciones directas cada 10 a 15 segundos hasta que se completa el trabajo.

Vea el widget de Jupyter mientras espera a que finalice la ejecución.
    
```python
from azureml.widgets import RunDetails
RunDetails(run).show()
```

![Captura de pantalla del widget de cuaderno de Jupyter](./media/how-to-track-experiments/run-details-widget.png)

También puede obtener un vínculo a la misma pantalla en el área de trabajo.

```python
print(run.get_portal_url())
```

## <a name="monitor-automated-machine-learning-runs"></a>Supervisar las ejecuciones automatizadas de aprendizaje automático

En el caso de las ejecuciones automatizadas de aprendizaje automático, para acceder a los gráficos de una ejecución anterior, sustituya `<<experiment_name>>` con el nombre del experimento correspondiente.

```python
from azureml.widgets import RunDetails
from azureml.core.run import Run

experiment = Experiment (workspace, <<experiment_name>>)
run_id = 'autoML_my_runID' #replace with run_ID
run = Run(experiment, run_id)
RunDetails(run).show()
```

![Widget de cuaderno de Jupyter Notebooks para aprendizaje automático automatizado](./media/how-to-track-experiments/azure-machine-learning-auto-ml-widget.png)

## <a name="show-output-upon-completion"></a>Visualización de la salida al finalizar

Cuando se usa **ScriptRunConfig**, se puede usar ```run.wait_for_completion(show_output = True)``` para mostrar cuándo finaliza el entrenamiento del modelo. La marca ```show_output``` le ofrece una salida detallada. Para obtener más información, consulte la sección ScriptRunConfig en [Cómo habilitar el registro](how-to-track-experiments.md#scriptrunconfig-logs).

<a id="queryrunmetrics"></a>
## <a name="query-run-metrics"></a>Métricas de ejecución de consulta

Puede ver las métricas de un modelo entrenado con ```run.get_metrics()```. Podría usar esto con el ejemplo anterior para determinar el mejor modelo buscando el modelo con el menor valor de error cuadrático medio (MSE).

<a name="view-the-experiment-in-the-web-portal"></a>
## <a name="view-run-records-in-the-studio"></a>Visualización de los registros de ejecución en Studio

Puede examinar los registros de ejecución completados, incluidas las métricas registradas, en [Azure Machine Learning Studio](https://ml.azure.com).

Vaya a la pestaña **Experimentos** y seleccione el experimento. En el panel de ejecución del experimento, puede ver las métricas y los registros del seguimiento de cada ejecución. 

Explore en profundidad una ejecución específica para ver sus salidas o registros, o descargar la instantánea del experimento para que pueda compartir la carpeta del experimento con otros usuarios.

También puede editar la tabla de la lista de ejecución para seleccionar varias ejecuciones y mostrar el último valor registrado, el mínimo o el máximo para las ejecuciones. Personalice los gráficos para comparar los valores de las métricas registradas y los agregados en varias ejecuciones.

![Detalles de la ejecución en Azure Machine Learning Studio](media/how-to-track-experiments/experimentation-tab.gif)

### <a name="format-charts-in-the-studio"></a>Dar formato a los gráficos en Studio

Use los métodos siguientes en las API de registro para influir en cómo visualiza las métricas Studio.

|Valor registrado|Ejemplo de código| Formato en el portal|
|----|----|----|
|Registrar una matriz de valores numéricos| `run.log_list(name='Fibonacci', value=[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89])`|Gráfico de líneas de una sola variable|
|Registrar un único valor numérico con el mismo nombre de métrica usado repetidamente (como desde dentro de un bucle)| `for i in tqdm(range(-10, 10)):    run.log(name='Sigmoid', value=1 / (1 + np.exp(-i))) angle = i / 2.0`| Gráfico de líneas de una sola variable|
|Registrar una fila con dos columnas numéricas repetidamente|`run.log_row(name='Cosine Wave', angle=angle, cos=np.cos(angle))   sines['angle'].append(angle)      sines['sine'].append(np.sin(angle))`|Gráfico de líneas de dos variables|
|Registrar tabla con dos columnas numéricas|`run.log_table(name='Sine Wave', value=sines)`|Gráfico de líneas de dos variables|


## <a name="next-steps"></a>Pasos siguientes

Pruebe estos pasos para aprender a usar Azure Machine Learning:

* Obtenga información sobre cómo [realizar un seguimiento de los experimentos y habilitar los registros en el diseñador de Azure Machine Learning (versión preliminar)](how-to-track-designer-experiments.md).

* Vea un ejemplo de cómo registrar el mejor modelo e implementarlo en el tutorial, [Entrenar un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

