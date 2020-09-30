---
title: Supervisión y recopilación de archivos de registro de canalizaciones
titleSuffix: Azure Machine Learning
description: Agregue el registro a las canalizaciones de entrenamiento y puntuación por lotes y vea los resultados registrados en Application Insights.
services: machine-learning
author: NilsPohlmann
ms.author: nilsp
ms.service: machine-learning
ms.subservice: core
ms.date: 08/11/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 8a3a82e091791b39ddf36e39987590dcddea320f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897504"
---
# <a name="collect-machine-learning-pipeline-log-files-in-application-insights-for-alerts-and-debugging"></a>Recopilación de archivos de registro de canalización de aprendizaje automático en Application Insights para alertas y depuración


La biblioteca [OpenCensus](https://opencensus.io/quickstart/python/) de Python se puede usar para enrutar los registros a Application Insights desde sus scripts. La agregación de registros de ejecuciones de canalización en un solo lugar le permite generar consultas y diagnosticar problemas. El uso de Application Insights le permitirá realizar un seguimiento de los registros a lo largo del tiempo y la comparación de los registros de la canalización entre las ejecuciones.

El hecho de tener los registros en un lugar permite proporcionar un historial de excepciones y mensajes de error. Como Application Insights se integra con alertas de Azure, también puede crear alertas basadas en consultas de Application Insights.

## <a name="prerequisites"></a>Requisitos previos

* Siga los pasos para crear un área de trabajo de [Azure Machine Learning](./how-to-manage-workspace.md) y [su primera canalización](./how-to-create-your-first-pipeline.md).
* [Configure un entorno de desarrollo](./how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning.
* Instale el paquete del [exportador OpenCensus de Azure Monitor](https://pypi.org/project/opencensus-ext-azure/) en un entorno local:
  ```python
  pip install opencensus-ext-azure
  ```
* Cree una [instancia de Application Insights](../azure-monitor/app/opencensus-python.md) (este documento también contiene información sobre cómo obtener la cadena de conexión para el recurso)

## <a name="getting-started"></a>Introducción

Esta sección es una introducción específica al uso de OpenCensus desde una canalización de Azure Machine Learning. Para ver un tutorial detallado, consulte [Exportadores de Azure Monitor de OpenCensus](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure).

Agregue un paso con PythonScriptStep a la canalización de Azure Machine Learning. Configure [RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py&preserve-view=true) con la dependencia en opencensus-ext-azure. Configure la variable de entorno `APPLICATIONINSIGHTS_CONNECTION_STRING`.

```python
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import RunConfiguration
from azureml.pipeline.core import Pipeline
from azureml.pipeline.steps import PythonScriptStep

# Connecting to the workspace and compute target not shown

# Add pip dependency on OpenCensus
dependencies = CondaDependencies()
dependencies.add_pip_package("opencensus-ext-azure>=1.0.1")
run_config = RunConfiguration(conda_dependencies=dependencies)

# Add environment variable with Application Insights Connection String
# Replace the value with your own connection string
run_config.environment.environment_variables = {
    "APPLICATIONINSIGHTS_CONNECTION_STRING": 'InstrumentationKey=00000000-0000-0000-0000-000000000000'
}

# Configure step with runconfig
sample_step = PythonScriptStep(
        script_name="sample_step.py",
        compute_target=compute_target,
        runconfig=run_config
)

# Submit new pipeline run
pipeline = Pipeline(workspace=ws, steps=[sample_step])
pipeline.submit(experiment_name="Logging_Experiment")
```

Cree un archivo llamado `sample_step.py`. Importe la clase AzureLogHandler para enrutar los registros a Application Insights. También tendrá que importar la biblioteca de registro de Python.

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
import logging
```

A continuación, agregue AzureLogHandler al registrador de Python.

```python
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler())

# Assumes the environment variable APPLICATIONINSIGHTS_CONNECTION_STRING is already set
logger.addHandler(AzureLogHandler())
logger.warning("I will be sent to Application Insights")
```

## <a name="logging-with-custom-dimensions"></a>Registro con dimensiones personalizadas
 
De forma predeterminada, los registros reenviados a Application Insights no dispondrán de suficiente contexto para realizar el seguimiento de la ejecución o el experimento. Para hacer que los registros sean útiles para el diagnóstico de problemas, se necesitan campos adicionales. 

Para agregar estos campos, se pueden agregar dimensiones personalizadas para proporcionar contexto a un mensaje de registro. Un ejemplo es cuando alguien desea ver los registros en varios pasos en la misma ejecución de canalización.

Las dimensiones personalizadas componen un diccionario de pares clave-valor (almacenado como cadena, cadena). Posteriormente, el diccionario se envía a Application Insights y se muestra como una columna en los resultados de la consulta. Sus dimensiones individuales se pueden usar como [parámetros de consulta](#additional-helpful-queries).

### <a name="helpful-context-to-include"></a>Contexto útil para incluir

| Campo                          | Razonamiento y ejemplo                                                                                                                                                                       |
|--------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| parent_run_id                  | Puede consultar aquellos registros con el mismo parent_run_id para ver los registros de todos los pasos durante un período de tiempo, en lugar de tener que profundizar en cada paso individual.                                        |
| step_id                        | Puede consultar aquellos registros con el mismo step_id para ver dónde se produjo un problema con un ámbito reducido a solo un paso individual.                                                        |
| step_name                      | Puede consultar los registros para ver el rendimiento del paso a lo largo del tiempo. También ayuda a encontrar una step_id para las ejecuciones recientes sin tener que profundizar en la interfaz de usuario del portal.                                          |
| experiment_name                | Puede realizar consultas en los registros para ver el rendimiento de los experimentos a lo largo del tiempo. También ayuda a encontrar un parent_run_id o step_id para las ejecuciones recientes sin tener que profundizar en la interfaz de usuario del portal.                   |
| run_url                 | Puede proporcionar un vínculo directamente a la ejecución con fines de investigación. |

**Otros campos útiles**

Estos campos pueden requerir instrumentación de código adicional y el contexto de ejecución no los proporciona.

| Campo                   | Razonamiento y ejemplo                                                                                                                                                                                                           |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| build_url/build_version | Si usa CI/CD para implementar, este campo puede correlacionar los registros con la versión del código que proporcionó el paso y la lógica de la canalización. Este vínculo puede ayudarle a diagnosticar problemas o identificar modelos con rasgos específicos (valores de registro o métricas). |
| run_type                       | Puede diferenciar entre distintos tipos de modelos o entre las ejecuciones de entrenamiento o de puntuación. |

### <a name="creating-a-custom-dimensions-dictionary"></a>Creación de un diccionario de dimensiones personalizadas

```python
from azureml.core import Run

run = Run.get_context(allow_offline=False)

custom_dimensions = {
    "parent_run_id": run.parent.id,
    "step_id": run.id,
    "step_name": run.name,
    "experiment_name": run.experiment.name,
    "run_url": run.parent.get_portal_url(),
    "run_type": "training"
}

# Assumes AzureLogHandler was already registered above
logger.info("I will be sent to Application Insights with Custom Dimensions", custom_dimensions)
```

## <a name="opencensus-python-logging-considerations"></a>Consideraciones sobre el registro de Python para OpenCensus

OpenCensus AzureLogHandler se usa para enrutar los registros de Python a Application Insights. Como resultado, se deben tener en cuenta los matices del registro de Python. Cuando se crea un registrador, tiene un nivel de registro predeterminado y muestra los registros que son mayores o iguales a ese nivel. Una buena referencia para el uso de las características de registro de Python es la [guía de registro](https://docs.python.org/3/howto/logging-cookbook.html).

La variable de entorno `APPLICATIONINSIGHTS_CONNECTION_STRING` es necesaria para la biblioteca OpenCensus. Se recomienda establecer esta variable de entorno en lugar de pasarla como parámetro de la canalización para evitar pasar cadenas de conexión de texto no cifrado.

## <a name="querying-logs-in-application-insights"></a>Consulta de registros en Application Insights

Los registros enrutados a Application Insights aparecerán en "seguimientos" o en "excepciones". Asegúrese de ajustar el período de tiempo para que incluya la ejecución de la canalización.

![Resultado de la consulta en Application Insights](./media/how-to-debug-pipelines-application-insights/traces-application-insights-query.png)

El resultado en Application Insights mostrará el mensaje y el nivel del registro, la ruta de acceso del archivo y el número de línea del código. También mostrará las dimensiones personalizadas incluidas. En esta imagen, el diccionario customDimensions muestra los pares clave-valor del [ejemplo de código](#creating-a-custom-dimensions-dictionary) anterior.

### <a name="additional-helpful-queries"></a>Consultas útiles adicionales

Algunas de las consultas siguientes usan "customDimensions.Level". Estos niveles de gravedad se corresponden con el nivel con el que se envió originalmente el registro de Python. Para obtener información adicional sobre consultas, vea [Consultas de registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/query-language).

| Caso de uso                                                               | Consultar                                                                                              |
|------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Resultados del registro para una dimensión personalizada específica, por ejemplo "parent_run_id" | <pre>traces \| <br>where customDimensions.parent_run_id == '931024c2-3720-11ea-b247-c49deda841c1</pre> |
| Resultados de registros para todas las ejecuciones de entrenamiento de los últimos 7 días                     | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.run_type == 'training'</pre>           |
| Resultados de registros con un valor en severityLevel de "Error" de los últimos 7 días              | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.Level == 'ERROR'                     |
| Recuento de resultados del registro con un valor en severityLevel de "Error" de los últimos 7 días     | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.Level == 'ERROR' \| <br>summarize count()</pre> |

## <a name="next-steps"></a>Pasos siguientes

Una vez que tenga registros en la instancia de Application Insights, se pueden usar para establecer las [alertas de Azure Monitor](../azure-monitor/platform/alerts-overview.md#what-you-can-alert-on) según los resultados de la consulta.

También puede agregar resultados de consultas a un [panel de Azure](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-app-dashboards#add-logs-analytics-query) para obtener información adicional.
