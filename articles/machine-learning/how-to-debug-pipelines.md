---
title: Depuración y solución de problemas de canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Depure las canalizaciones de Azure Machine Learning en Python. Obtenga información sobre los errores comunes en el desarrollo de las canalizaciones y sugerencias que le ayudarán a depurar los scripts antes y durante la ejecución remota. Obtenga información sobre cómo usar Visual Studio Code para depurar interactivamente las canalizaciones de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: likebupt
ms.author: keli19
ms.date: 03/18/2020
ms.topic: conceptual
ms.custom: troubleshooting, tracking-python
ms.openlocfilehash: 21254f5d7171d67614b212007aa33f4429505574
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553060"
---
# <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>Depuración y solución de problemas de canalizaciones de aprendizaje automático
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a depurar y resolver los problemas de las [canalizaciones de aprendizaje automático](concept-ml-pipelines.md) con el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) y el [diseñador de Azure Machine Learning (versión preliminar)](https://docs.microsoft.com/azure/machine-learning/concept-designer). Se proporciona información sobre cómo:

* Depurar con el SDK de Azure Machine Learning
* Depurar con el diseñador de Azure Machine Learning
* Depurar con Application Insights
* Depurar interactivamente con Visual Studio Code (VS Code) y Herramientas de Python para Visual Studio (PTVSD)

## <a name="azure-machine-learning-sdk"></a>SDK de Azure Machine Learning
En la siguientes secciones se da una información general sobre los errores comunes durante la creación de canalizaciones, y sobre las diferentes estrategias para depurar el código que se ejecuta en una canalización. Cuando aparezcan problemas y una canalización no se ejecute según lo esperado, utilice las sugerencias que se ofrecen a continuación.

### <a name="testing-scripts-locally"></a>Prueba de scripts de forma local

Uno de los errores más comunes en una canalización es que uno de los scripts adjuntos (script de limpieza de datos, script de puntuación, etc.) no se ejecute según lo previsto, o que contenga errores en tiempo de ejecución en el contexto de proceso remoto que son difíciles de depurar en el área de trabajo en Azure Machine Learning Studio. 

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

### <a name="debugging-scripts-from-remote-context"></a>Depuración de scripts desde un contexto remoto

La prueba de scripts de forma local es una excelente manera de depurar fragmentos de código principales y lógica compleja antes de empezar a crear una canalización, de todas formas, es probable que en algún momento tenga que depurar scripts durante la propia ejecución de canalización real, especialmente al diagnosticar el comportamiento que se produce durante la interacción entre los pasos de canalización. Se recomienda el uso generoso de las instrucciones `print()` en los scripts de paso, con el fin de ver el estado del objeto y los valores esperados durante la ejecución remota, de manera similar a como se depuraría el código de JavaScript.

El archivo de registro `70_driver_log.txt` contiene: 

* Todas las instrucciones impresas durante la ejecución del script
* El seguimiento de la pila para el script 

Para encontrar este y otros archivos de registro en el portal, primero haga clic en la canalización en el área de trabajo.

![Página de lista de ejecuciones de canalización](./media/how-to-debug-pipelines/pipelinerun-01.png)

Vaya a la página de detalles de ejecución de la canalización.

![Página de detalles de ejecución de canalización](./media/how-to-debug-pipelines/pipelinerun-02.png)

Haga clic en el identificador de ejecución del paso específico. Vaya a la pestaña **Registros**. Otros registros incluyen información sobre el proceso de generación de imágenes de entorno y scripts de preparación de pasos.

![Pestaña de registro de la página de detalles de ejecución de canalización](./media/how-to-debug-pipelines/pipelinerun-03.png)

> [!TIP]
> Las ejecuciones de las *canalizaciones publicadas* están en la pestaña **Puntos de conexión** del área de trabajo del portal. Las ejecuciones de las *canalizaciones no publicadas* se encuentran en **Experimentos** o **Canalizaciones**.

### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

La tabla siguiente contiene problemas comunes que se pueden producir durante el desarrollo de canalizaciones y posibles soluciones a los mismo.

| Problema | Posible solución |
|--|--|
| No se pueden pasar datos al directorio `PipelineData` | Asegúrese de haber creado un directorio en el script que se corresponda con el lugar en el que la canalización espera los datos de salida del paso. En la mayoría de los casos, un argumento de entrada definirá el directorio de salida, a continuación se crea el directorio de forma explícita. Use `os.makedirs(args.output_dir, exist_ok=True)` para crear el directorio de salida. Si necesita un ejemplo de script de puntuación que muestra este patrón de diseño, acuda al [tutorial](tutorial-pipeline-batch-scoring-classification.md#write-a-scoring-script) correspondiente. |
| Errores de dependencia | Si ha desarrollado y probado los scripts localmente, pero se encuentra problemas de dependencia cuando la ejecución se realiza en un proceso remoto en la canalización, asegúrese de que las dependencias y las versiones del entorno de proceso coinciden con el entorno de prueba. Consulte el artículo [Compilación, almacenamiento en caché y reutilización de entornos](https://docs.microsoft.com/azure/machine-learning/concept-environments#environment-building-caching-and-reuse).|
| Errores ambiguos con destinos de proceso | Eliminar y volver a crear los destinos de proceso puede resolver determinados problemas relacionados con ellos. |
| La canalización no reutiliza los pasos | La reutilización de pasos está habilitada de forma predeterminada, asegúrese de que no la ha deshabilitado en un paso de la canalización. Si la reutilización está deshabilitada, el parámetro `allow_reuse` del paso se establecerá en `False`. |
| La canalización se está volviendo a ejecutar innecesariamente | Para asegurarse de que los pasos solo se vuelven a ejecutar cuando sus datos o scripts subyacentes cambian, desacople los directorios de cada paso. Si usa el mismo directorio de origen para varios pasos, puede experimentar la repetición innecesaria de ejecuciones. Use el parámetro `source_directory` en un objeto de paso de canalización para apuntar a su directorio aislado para ese paso, y asegúrese de que no está usando la misma ruta de acceso `source_directory` para varios pasos. |

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

## <a name="azure-machine-learning-designer-preview"></a>Diseñador de Azure Machine Learning (versión preliminar)

En esta sección se proporciona información general sobre cómo solucionar los problemas de las canalizaciones en el diseñador. En el caso de las canalizaciones creadas en el diseñador, puede encontrar el archivo **70_driver_log** en la página de creación o en la página de detalles de ejecución de la canalización.

### <a name="enable-logging-for-real-time-endpoints"></a>Habilitación del registro para puntos de conexión en tiempo real

Para solucionar problemas y depurar puntos de conexión en tiempo real en el diseñador, tiene que habilitar el registro de Application Insights mediante el SDK. El registro le permite depurar y solucionar problemas de uso e implementación de modelos. Para más información, consulte [Registro para modelos implementados](how-to-enable-logging.md#logging-for-deployed-models). 

### <a name="get-logs-from-the-authoring-page"></a>Obtención de los registros desde la página de creación

Cuando envía una ejecución de canalización y permanece en la página de creación, puede encontrar los archivos de registro generados para cada módulo a medida que cada módulo finaliza su ejecución.

1. Seleccione un módulo que haya terminado de ejecutarse en el lienzo de creación.
1. En el panel derecho del módulo, vaya a la pestaña **Resultados y registros**.
1. Expanda el panel derecho y seleccione el archivo **70_driver_log. txt** para verlo en el explorador. También puede descargar registros localmente.

    ![Panel de salida expandido en el diseñador](./media/how-to-debug-pipelines/designer-logs.png)

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

## <a name="visual-studio-code"></a>Visual Studio Code

En algunos casos, es posible que tenga que depurar interactivamente el código de Python usado en la canalización de ML. Mediante el uso de Visual Studio Code(VS Code) y Herramientas de Python para Visual Studio (PTVSD), se puede conectar al código que se ejecuta en el entorno de aprendizaje.

### <a name="prerequisites"></a>Prerrequisitos

* Un __área de trabajo de Azure Machine Learning__ configurada para usar __Azure Virtual Network__.
* Una __canalización de Azure Machine Learning__ que use scripts de Python como parte de los pasos de la canalización. Por ejemplo, PythonScriptStep.
* Un clúster de Proceso de Azure Machine Learning, que se encuentre __en la red virtual__ y lo __use la canalización para el aprendizaje__.
* Un __entorno de desarrollo__ que se encuentre __en la red virtual__. El entorno de desarrollo puede ser uno de los siguientes:

    * Una máquina virtual de Azure en la red virtual
    * Una instancia de proceso de VM de cuadernos en la red virtual
    * Un equipo cliente conectado a la red virtual mediante una red privada virtual (VPN).

Para más información sobre el uso de Azure Virtual Network con Azure Machine Learning, consulte [Protección de los trabajos de experimentación e inferencia de ML en una instancia de Azure Virtual Network](how-to-enable-virtual-network.md).

### <a name="how-it-works"></a>Funcionamiento

Los pasos de la canalización de ML ejecutan scripts de Python. Estos scripts se modifican para realizar las siguientes acciones:
    
1. Registrar la dirección IP del host en el que se ejecutan. Se usa la dirección IP para conectar el depurador al script.

2. Iniciar el componente de depuración PTVSD y esperar a que se conecte un depurador.

3. En el entorno de desarrollo, se supervisan los registros creados por el proceso de aprendizaje para encontrar la dirección IP en la que se ejecuta el script.

4. Se indica a VS Code la dirección IP a la que se debe conectar el depurador mediante un archivo `launch.json`.

5. Se conecta el depurador y se recorre el script de forma interactiva.

### <a name="configure-python-scripts"></a>Configuración de scripts de Python

Para habilitar la depuración, realice los cambios siguientes en los scripts de Python que se usan en los pasos de la canalización de ML:

1. Agregue las siguientes instrucciones import:

    ```python
    import ptvsd
    import socket
    from azureml.core import Run
    ```

1. Agregue los siguientes argumentos. Estos argumentos permiten habilitar el depurador según sea necesario y establecer el tiempo de espera para la conexión del depurador:

    ```python
    parser.add_argument('--remote_debug', action='store_true')
    parser.add_argument('--remote_debug_connection_timeout', type=int,
                    default=300,
                    help=f'Defines how much time the Azure ML compute target '
                    f'will await a connection from a debugger client (VSCODE).')
    ```

1. Agregue las siguientes instrucciones. Estas instrucciones cargan el contexto de ejecución actual para que pueda registrar la dirección IP del nodo en el que se está ejecutando el código:

    ```python
    global run
    run = Run.get_context()
    ```

1. Agregue una instrucción `if` que inicie PTVSD y espere a que se conecte un depurador. Si no se conecta ningún depurador antes de que finalice el tiempo de espera, el script continúa de la forma habitual.

    ```python
    if args.remote_debug:
        print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
        # Log the IP and port
        ip = socket.gethostbyname(socket.gethostname())
        print(f'ip_address: {ip}')
        ptvsd.enable_attach(address=('0.0.0.0', 5678),
                            redirect_output=True)
        # Wait for the timeout for debugger to attach
        ptvsd.wait_for_attach(timeout=args.remote_debug_connection_timeout)
        print(f'Debugger attached = {ptvsd.is_attached()}')
    ```

En el siguiente ejemplo de Python se muestra un archivo `train.py` básico que habilita la depuración:

```python
# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT license.

import argparse
import os
import ptvsd
import socket
from azureml.core import Run

print("In train.py")
print("As a data scientist, this is where I use my training code.")

parser = argparse.ArgumentParser("train")

parser.add_argument("--input_data", type=str, help="input data")
parser.add_argument("--output_train", type=str, help="output_train directory")

# Argument check for remote debugging
parser.add_argument('--remote_debug', action='store_true')
parser.add_argument('--remote_debug_connection_timeout', type=int,
                    default=300,
                    help=f'Defines how much time the AML compute target '
                    f'will await a connection from a debugger client (VSCODE).')
# Get run object, so we can find and log the IP of the host instance
global run
run = Run.get_context()

args = parser.parse_args()

# Start debugger if remote_debug is enabled
if args.remote_debug:
    print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
    # Log the IP and port
    ip = socket.gethostbyname(socket.gethostname())
    print(f'ip_address: {ip}')
    ptvsd.enable_attach(address=('0.0.0.0', 5678),
                        redirect_output=True)
    # Wait for the timeout for debugger to attach
    ptvsd.wait_for_attach(timeout=args.remote_debug_connection_timeout)
    print(f'Debugger attached = {ptvsd.is_attached()}')

print("Argument 1: %s" % args.input_data)
print("Argument 2: %s" % args.output_train)

if not (args.output_train is None):
    os.makedirs(args.output_train, exist_ok=True)
    print("%s created" % args.output_train)
```

### <a name="configure-ml-pipeline"></a>Configuración de la canalización de ML

Para proporcionar los paquetes de Python necesarios para iniciar PTVSD y obtener el contexto de ejecución, cree un entorno y establezca `pip_packages=['ptvsd', 'azureml-sdk==1.0.83']`. Cambie la versión del SDK para que coincida con la que está usando. El fragmento de código siguiente muestra cómo crear un entorno:

```python
# Use a RunConfiguration to specify some additional requirements for this step.
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_CPU_IMAGE

# create a new runconfig object
run_config = RunConfiguration()

# enable Docker 
run_config.environment.docker.enabled = True

# set Docker base image to the default CPU-based image
run_config.environment.docker.base_image = DEFAULT_CPU_IMAGE

# use conda_dependencies.yml to create a conda environment in the Docker image for execution
run_config.environment.python.user_managed_dependencies = False

# specify CondaDependencies obj
run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
                                                                           pip_packages=['ptvsd', 'azureml-sdk==1.0.83'])
```

En la sección [Configuración de scripts de Python](#configure-python-scripts), se agregaron dos nuevos argumentos a los scripts usados por los pasos de la canalización de ML. En el fragmento de código siguiente se muestra cómo usar estos argumentos para habilitar la depuración del componente y establecer un tiempo de espera. También se muestra cómo usar el entorno creado anteriormente estableciendo `runconfig=run_config`:

```python
# Use RunConfig from a pipeline step
step1 = PythonScriptStep(name="train_step",
                         script_name="train.py",
                         arguments=['--remote_debug', '--remote_debug_connection_timeout', 300],
                         compute_target=aml_compute, 
                         source_directory=source_directory,
                         runconfig=run_config,
                         allow_reuse=False)
```

Cuando se ejecuta la canalización, cada paso crea una ejecución secundaria. Si está habilitada la depuración, el script modificado registra información similar al siguiente texto en `70_driver_log.txt` para la ejecución secundaria:

```text
Timeout for debug connection: 300
ip_address: 10.3.0.5
```

Guarde el valor de `ip_address`. Se usa en la siguiente sección.

> [!TIP]
> También puede encontrar la dirección IP en los registros de la ejecución secundaria para este paso de canalización. Para obtener más información sobre cómo ver esta información, vea [Supervisión de métricas y ejecuciones de experimentos de Azure ML](how-to-track-experiments.md).

### <a name="configure-development-environment"></a>Configuración del entorno de desarrollo

1. Para instalar Herramientas de Python para Visual Studio (PTVSD) en el entorno de desarrollo de VS Code, use el comando siguiente:

    ```
    python -m pip install --upgrade ptvsd
    ```

    Para más información sobre cómo usar PTVSD con VS Code, consulte [Depuración remota](https://code.visualstudio.com/docs/python/debugging#_remote-debugging).

1. Para configurar VS Code para comunicarse con el proceso de Azure Machine Learning que ejecuta el depurador, cree una nueva configuración de depuración:

    1. En VS Code, seleccione el menú __Depurar__ y, luego, seleccione __Abrir configuraciones__. Se abre un archivo denominado __launch.json__.

    1. En el archivo __launch.json__, busque la línea que contiene `"configurations": [` e inserte el texto siguiente después de ella. Cambie la entrada `"host": "10.3.0.5"` a la dirección IP devuelta en los registros de la sección anterior. Cambie la entrada `"localRoot": "${workspaceFolder}/code/step"` a un directorio local que contenga una copia del script que se está depurando:

        ```json
        {
            "name": "Azure Machine Learning Compute: remote debug",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "10.3.0.5",
            "redirectOutput": true,
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}/code/step1",
                    "remoteRoot": "."
                }
            ]
        }
        ```

        > [!IMPORTANT]
        > Si ya hay otras entradas en la sección de configuraciones, agregue una coma (,) después del código que insertó.

        > [!TIP]
        > El procedimiento recomendado es mantener los recursos para los scripts en directorios independientes, es por lo que el valor del ejemplo `localRoot` hace referencia a `/code/step1`.
        >
        > Si va a depurar varios scripts en directorios diferentes, cree una sección de configuración independiente para cada script.

    1. Guarde el archivo __launch.json__.

### <a name="connect-the-debugger"></a>Conexión del depurador

1. Abra VS Code y abra una copia local del script.
2. Establezca puntos de interrupción donde desee que se detenga el script una vez que se haya conectado.
3. Mientras el proceso secundario ejecuta el script y se muestra `Timeout for debug connection` en los registros, use la tecla F5 o seleccione __Depurar__. Cuando se le solicite, seleccione la configuración __Azure Machine Learning Compute: remote debug__ (Proceso de Azure Machine Learning: depuración remota). También puede seleccionar el icono de depuración en la barra lateral, la entrada __Azure Machine Learning: remote debug__ (Azure Machine Learning: depuración remota) en el menú desplegable Depurar y, a continuación, usar la flecha verde para conectar el depurador.

    En este punto, VS Code se conecta a PTVSD en el nodo de ejecución y se detiene en el punto de interrupción que se estableció anteriormente. Ahora puede recorrer el código a medida que se ejecuta, ver variables, etc.

    > [!NOTE]
    > Si el registro muestra una entrada que indica `Debugger attached = False`, el tiempo de espera ha expirado y el script ha continuado sin el depurador. Vuelva a enviar la canalización y conecte el depurador después del mensaje de `Timeout for debug connection` y antes de que expire el tiempo de espera.

## <a name="next-steps"></a>Pasos siguientes

* Consulte la referencia del SDK para encontrar ayuda para los paquetes [azureml-pipelines-core](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py) y [azureml-pipelines-steps](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py).

* Vea la lista de [excepciones y códigos de error del diseñador](algorithm-module-reference/designer-error-codes.md).
