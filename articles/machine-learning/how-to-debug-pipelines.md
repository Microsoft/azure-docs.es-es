---
title: Depuración y solución de problemas de canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Depure las canalizaciones de Azure Machine Learning en Python. Obtenga información sobre los errores comunes en el desarrollo de las canalizaciones y sugerencias que le ayudarán a depurar los scripts antes y durante la ejecución remota. Obtenga información sobre cómo usar Visual Studio Code para depurar interactivamente las canalizaciones de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: lobrien
ms.author: laobri
ms.date: 08/28/2020
ms.topic: conceptual
ms.custom: troubleshooting, devx-track-python
ms.openlocfilehash: 616cdb1d0940ea6f64c3be3d687adaa9c2a98cc2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90889962"
---
# <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>Depuración y solución de problemas de canalizaciones de aprendizaje automático

En este artículo se obtiene información sobre cómo depurar y resolver los problemas de las [canalizaciones de aprendizaje automático](concept-ml-pipelines.md) en el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) y el [diseñador de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-designer). Se proporciona información sobre cómo:

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

La tabla siguiente contiene problemas comunes que se pueden producir durante el desarrollo de canalizaciones y posibles soluciones a los mismo.

| Problema | Posible solución |
|--|--|
| No se pueden pasar datos al directorio `PipelineData` | Asegúrese de haber creado un directorio en el script que se corresponda con el lugar en el que la canalización espera los datos de salida del paso. En la mayoría de los casos, un argumento de entrada definirá el directorio de salida, a continuación se crea el directorio de forma explícita. Use `os.makedirs(args.output_dir, exist_ok=True)` para crear el directorio de salida. Si necesita un ejemplo de script de puntuación que muestra este patrón de diseño, acuda al [tutorial](tutorial-pipeline-batch-scoring-classification.md#write-a-scoring-script) correspondiente. |
| Errores de dependencia | Si ve errores de dependencia en la canalización remota que no se produjeron durante las pruebas locales, confirme que las dependencias y las versiones del entorno remoto coinciden con las de su entorno de prueba. Consulte el artículo [Compilación, almacenamiento en caché y reutilización de entornos](https://docs.microsoft.com/azure/machine-learning/concept-environments#environment-building-caching-and-reuse).|
| Errores ambiguos con destinos de proceso | Pruebe a eliminar y volver a crear los destinos de proceso. Volver a crear los destinos de proceso es un proceso rápido y puede resolver algunos problemas transitorios. |
| La canalización no reutiliza los pasos | La reutilización de pasos está habilitada de forma predeterminada, asegúrese de que no la ha deshabilitado en un paso de la canalización. Si la reutilización está deshabilitada, el parámetro `allow_reuse` del paso se establecerá en `False`. |
| La canalización se está volviendo a ejecutar innecesariamente | Para asegurarse de que los pasos solo se vuelven a ejecutar cuando sus datos o scripts subyacentes cambian, desacople los directorios de código fuente en cada paso. Si usa el mismo directorio de origen para varios pasos, puede experimentar la repetición innecesaria de ejecuciones. Use el parámetro `source_directory` en un objeto de paso de canalización para apuntar a su directorio aislado para ese paso, y asegúrese de que no está usando la misma ruta de acceso `source_directory` para varios pasos. |
| Paso más lento en los tiempos de entrenamiento u otro comportamiento de bucle | Intente cambiar cualquier escritura de archivo, incluido el registro, de `as_mount()` a `as_upload()`. El modo de **montaje** usa un sistema de archivos virtualizado remoto y carga todo el archivo cada vez que se anexa a este. |

## <a name="debugging-techniques"></a>Técnicas de depuración

Hay tres técnicas principales para depurar las canalizaciones: 

* Depurar pasos de canalización individuales en el equipo local
* Usar el registro y Application Insights para aislar y diagnosticar el origen del problema
* Conectar un depurador remoto a una canalización que se ejecute en Azure

### <a name="debug-scripts-locally"></a>Depuración de scripts de forma local

Uno de los errores más comunes en una canalización es que el script de dominio no se ejecute según lo previsto, o que contenga errores en tiempo de ejecución difíciles de depurar en el contexto de proceso remoto.

Las canalizaciones no se pueden ejecutar localmente, pero la ejecución de los scripts de forma aislada en la máquina local le permite depurar más rápido, ya que no tiene que esperar al proceso de compilación de proceso y de entorno. Para ello, se necesita realizar algo de trabajo de desarrollo:

* Si los datos están en un almacén de datos en la nube, tendrá que descargarlos y ponerlos a disposición del script. El uso de una pequeña muestra de los datos es una buena forma de reducir el tiempo de ejecución y obtener rápidamente comentarios sobre el comportamiento del script
* Si intenta simular un paso de canalización intermedio, es posible que tenga que compilar manualmente los tipos de objeto que el script específico espera del paso anterior
* También tendrá que definir su propio entorno y replicar las dependencias definidas en el entorno de proceso remoto

Una vez que tenga una configuración de script que se ejecute en su entorno local, es mucho más fácil realizar tareas de depuración como:

* Incorporación de una configuración de depuración personalizada
* Pausa de la ejecución e inspección del estado del objeto
* Detección de errores lógicos o de tipo que no se expondrían hasta el tiempo de ejecución

> [!TIP] 
> Una vez que pueda comprobar que el script se ejecuta según lo previsto, un buen paso posterior es ejecutar el script en una canalización de un solo paso antes de intentar ejecutarlo en una canalización con varios pasos.

## <a name="configure-write-to-and-review-pipeline-logs"></a>Configuración, escritura y revisión de los registros de canalización

La prueba de scripts de forma local es una excelente manera de depurar fragmentos de código principales y lógica compleja antes de empezar a crear una canalización, de todas formas, es probable que en algún momento tenga que depurar scripts durante la propia ejecución de canalización real, especialmente al diagnosticar el comportamiento que se produce durante la interacción entre los pasos de canalización. Se recomienda el uso generoso de las instrucciones `print()` en los scripts de paso, con el fin de ver el estado del objeto y los valores esperados durante la ejecución remota, de manera similar a como se depuraría el código de JavaScript.

### <a name="logging-options-and-behavior"></a>Opciones de registro y comportamiento

En la tabla siguiente se proporciona información sobre las distintas opciones de depuración para las canalizaciones. No se trata de una lista exhaustiva, ya que existen otras opciones aparte de las de Azure Machine Learning, Python y OpenCensus que se muestran aquí.

| Biblioteca                    | Tipo   | Ejemplo                                                          | Destination                                  | Recursos                                                                                                                                                                                                                                                                                                                    |
|----------------------------|--------|------------------------------------------------------------------|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SDK de Azure Machine Learning | Métrica | `run.log(name, val)`                                             | UI del portal de Azure Machine Learning             | [Seguimiento de experimentos](how-to-track-experiments.md)<br>[Clase azureml.core.Run](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run(class)?view=experimental)                                                                                                                                                 |
| Impresión/registro de Python    | Log    | `print(val)`<br>`logging.info(message)`                          | Registros de controladores, el diseñador de Azure Machine Learning | [Seguimiento de experimentos](how-to-track-experiments.md)<br><br>[Registro de Python](https://docs.python.org/2/library/logging.html)                                                                                                                                                                       |
| Python para OpenCensus          | Log    | `logger.addHandler(AzureLogHandler())`<br>`logging.log(message)` | Application Insights: seguimientos                | [Depuración de canalizaciones en Application Insights](how-to-debug-pipelines-application-insights.md)<br><br>[Exportadores de Azure Monitor de OpenCensus](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)<br>[Guía de registro de Python](https://docs.python.org/3/howto/logging-cookbook.html) |

#### <a name="logging-options-example"></a>Ejemplo de opciones de registro

```python
import logging

from azureml.core.run import Run
from opencensus.ext.azure.log_exporter import AzureLogHandler

run = Run.get_context()

# Azure ML Scalar value logging
run.log("scalar_value", 0.95)

# Python print statement
print("I am a python print statement, I will be sent to the driver logs.")

# Initialize python logger
logger = logging.getLogger(__name__)
logger.setLevel(args.log_level)

# Plain python logging statements
logger.debug("I am a plain debug statement, I will be sent to the driver logs.")
logger.info("I am a plain info statement, I will be sent to the driver logs.")

handler = AzureLogHandler(connection_string='<connection string>')
logger.addHandler(handler)

# Python logging with OpenCensus AzureLogHandler
logger.warning("I am an OpenCensus warning statement, find me in Application Insights!")
logger.error("I am an OpenCensus error statement with custom dimensions", {'step_id': run.id})
``` 

## <a name="azure-machine-learning-designer"></a>Diseñador de Azure Machine Learning

En el caso de las canalizaciones creadas en el diseñador, puede encontrar el archivo **70_driver_log** en la página de creación o en la página de detalles de ejecución de la canalización.

### <a name="enable-logging-for-real-time-endpoints"></a>Habilitación del registro para puntos de conexión en tiempo real

Para solucionar problemas y depurar puntos de conexión en tiempo real en el diseñador, tiene que habilitar el registro de Application Insights mediante el SDK. El registro le permite depurar y solucionar problemas de uso e implementación de modelos. Para más información, consulte [Registro para modelos implementados](how-to-enable-logging.md#logging-for-deployed-models). 

### <a name="get-logs-from-the-authoring-page"></a>Obtención de los registros desde la página de creación

Cuando envía una ejecución de canalización y permanece en la página de creación, puede encontrar los archivos de registro generados para cada módulo a medida que cada módulo finaliza su ejecución.

1. Seleccione un módulo que haya terminado de ejecutarse en el lienzo de creación.
1. En el panel derecho del módulo, vaya a la pestaña **Resultados y registros**.
1. Expanda el panel derecho y seleccione el archivo **70_driver_log. txt** para verlo en el explorador. También puede descargar registros localmente.

    ![Panel de salida expandido en el diseñador](./media/how-to-debug-pipelines/designer-logs.png)?view=azure-ml-py&preserve-view=true)?view=azure-ml-py&preserve-view=true)

### <a name="get-logs-from-pipeline-runs"></a>Obtención de registros desde las ejecuciones de canalización

También puede buscar los archivos de registro de ejecuciones específicas en la página de detalles de ejecución de canalización en las secciones **Canalizaciones** o **Experimentos** de Studio.

1. Seleccione una ejecución de canalización creada en el diseñador.

    ![Página de ejecución de la canalización](./media/how-to-debug-pipelines/designer-pipelines.png)

1. Seleccione un módulo en el panel de vista previa.
1. En el panel derecho del módulo, vaya a la pestaña **Resultados y registros**.
1. Expanda el panel derecho para ver el archivo **70_driver_log. txt** en el explorador, o seleccione el archivo para descargar los registros de forma local.

> [!IMPORTANT]
> Para actualizar una canalización desde la página de detalles de ejecución de la canalización, tiene que **clonar** la ejecución de la canalización en un nuevo borrador de canalización. Una ejecución de canalización es una instantánea de la canalización. Es similar a un archivo de registro y no se puede modificar. 

## <a name="application-insights"></a>Application Insights
Para obtener más información sobre el uso de la biblioteca de Python para OpenCensus de esta manera, consulte esta guía: [Depuración y solución de problemas de canalizaciones de aprendizaje automático en Application Insights](how-to-debug-pipelines-application-insights.md)

## <a name="interactive-debugging-with-visual-studio-code"></a>Depuración interactiva con Visual Studio Code

En algunos casos, es posible que tenga que depurar interactivamente el código de Python usado en la canalización de ML. Mediante Visual Studio Code (VS Code) y debugpy, se puede conectar al código que se ejecuta en el entorno de entrenamiento. Para más información, visite la [guía de depuración interactiva de VS Code](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-machine-learning-pipelines).

## <a name="next-steps"></a>Pasos siguientes

* Consulte la referencia del SDK para encontrar ayuda para los paquetes [azureml-pipelines-core](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py&preserve-view=true) y [azureml-pipelines-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py&preserve-view=true).

* Vea la lista de [excepciones y códigos de error del diseñador](algorithm-module-reference/designer-error-codes.md).
