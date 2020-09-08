---
title: Depuración interactiva con Visual Studio Code
titleSuffix: Azure Machine Learning
description: Depure de manera interactiva el código, las canalizaciones e implementaciones de Azure Machine Learning mediante Visual Studio Code
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: troubleshooting
author: luisquintanilla
ms.author: luquinta
ms.date: 08/06/2020
ms.openlocfilehash: 73cb8396876a5baad74190ec9a86237362037c36
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907610"
---
# <a name="interactive-debugging-with-visual-studio-code"></a>Depuración interactiva con Visual Studio Code

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Obtenga información sobre cómo depurar interactivamente canalizaciones e implementaciones de Azure Machine Learning con Visual Studio Code (VS Code) y [depugpy](https://github.com/microsoft/debugpy/).

## <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>Depuración y solución de problemas de canalizaciones de aprendizaje automático

En algunos casos, es posible que tenga que depurar interactivamente el código de Python usado en la canalización de ML. Mediante VS Code y debugpy, se puede conectar al código que se ejecuta en el entorno de entrenamiento.

### <a name="prerequisites"></a>Prerrequisitos

* Un __área de trabajo de Azure Machine Learning__ configurada para usar __Azure Virtual Network__.
* Una __canalización de Azure Machine Learning__ que use scripts de Python como parte de los pasos de la canalización. Por ejemplo, PythonScriptStep.
* Un clúster de Proceso de Azure Machine Learning, que se encuentre __en la red virtual__ y lo __use la canalización para el aprendizaje__.
* Un __entorno de desarrollo__ que se encuentre __en la red virtual__. El entorno de desarrollo puede ser uno de los siguientes:

  * Una máquina virtual de Azure en la red virtual
  * Una instancia de proceso de VM de cuadernos en la red virtual
  * Una máquina cliente que tenga conectividad de red privada con la red virtual, ya sea por VPN o a través de ExpressRoute.

Para más información sobre el uso de Azure Virtual Network con Azure Machine Learning, consulte [Protección de los trabajos de experimentación e inferencia de ML en una instancia de Azure Virtual Network](how-to-enable-virtual-network.md).

> [!TIP]
> Aunque puede trabajar con recursos de Azure Machine Learning que no están detrás de una red virtual, se recomienda usar una red virtual.

### <a name="how-it-works"></a>Funcionamiento

Los pasos de la canalización de ML ejecutan scripts de Python. Estos scripts se modifican para realizar las siguientes acciones:

1. Registrar la dirección IP del host en el que se ejecutan. Se usa la dirección IP para conectar el depurador al script.

2. Iniciar el componente de depuración debugpy y esperar a que se conecte un depurador.

3. En el entorno de desarrollo, se supervisan los registros creados por el proceso de aprendizaje para encontrar la dirección IP en la que se ejecuta el script.

4. Se indica a VS Code la dirección IP a la que se debe conectar el depurador mediante un archivo `launch.json`.

5. Se conecta el depurador y se recorre el script de forma interactiva.

### <a name="configure-python-scripts"></a>Configuración de scripts de Python

Para habilitar la depuración, realice los cambios siguientes en los scripts de Python que se usan en los pasos de la canalización de ML:

1. Agregue las siguientes instrucciones import:

    ```python
    import argparse
    import os
    import debugpy
    import socket
    from azureml.core import Run
    ```

1. Agregue los siguientes argumentos. Estos argumentos permiten habilitar el depurador según sea necesario y establecer el tiempo de espera para la conexión del depurador:

    ```python
    parser.add_argument('--remote_debug', action='store_true')
    parser.add_argument('--remote_debug_connection_timeout', type=int,
                        default=300,
                        help=f'Defines how much time the AML compute target '
                        f'will await a connection from a debugger client (VSCODE).')
    parser.add_argument('--remote_debug_client_ip', type=str,
                        help=f'Defines IP Address of VS Code client')
    parser.add_argument('--remote_debug_port', type=int,
                        default=5678,
                        help=f'Defines Port of VS Code client')
    ```

1. Agregue las siguientes instrucciones. Estas instrucciones cargan el contexto de ejecución actual para que pueda registrar la dirección IP del nodo en el que se está ejecutando el código:

    ```python
    global run
    run = Run.get_context()
    ```

1. Agregue una instrucción `if` que inicie debugpy y espere a que se conecte un depurador. Si no se conecta ningún depurador antes de que finalice el tiempo de espera, el script continúa de la forma habitual. Asegúrese de reemplazar los valores `HOST` y `PORT` de la función `listen` por los propios.

    ```python
    if args.remote_debug:
        print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
        # Log the IP and port
        try:
            ip = args.remote_debug_client_ip
        except:
            print("Need to supply IP address for VS Code client")
        print(f'ip_address: {ip}')
        debugpy.listen(address=(ip, args.remote_debug_port))
        # Wait for the timeout for debugger to attach
        debugpy.wait_for_client()
        print(f'Debugger attached = {debugpy.is_client_connected()}')
    ```

En el siguiente ejemplo de Python se muestra un archivo `train.py` básico que habilita la depuración:

```python
# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT license.

import argparse
import os
import debugpy
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
parser.add_argument('--remote_debug_client_ip', type=str,
                    help=f'Defines IP Address of VS Code client')
parser.add_argument('--remote_debug_port', type=int,
                    default=5678,
                    help=f'Defines Port of VS Code client')

# Get run object, so we can find and log the IP of the host instance
global run
run = Run.get_context()

args = parser.parse_args()

# Start debugger if remote_debug is enabled
if args.remote_debug:
    print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
    # Log the IP and port
    # ip = socket.gethostbyname(socket.gethostname())
    try:
        ip = args.remote_debug_client_ip
    except:
        print("Need to supply IP address for VS Code client")
    print(f'ip_address: {ip}')
    debugpy.listen(address=(ip, args.remote_debug_port))
    # Wait for the timeout for debugger to attach
    debugpy.wait_for_client()
    print(f'Debugger attached = {debugpy.is_client_connected()}')

print("Argument 1: %s" % args.input_data)
print("Argument 2: %s" % args.output_train)

if not (args.output_train is None):
    os.makedirs(args.output_train, exist_ok=True)
    print("%s created" % args.output_train)
```

### <a name="configure-ml-pipeline"></a>Configuración de la canalización de ML

Para proporcionar los paquetes de Python necesarios para iniciar debugpy y obtener el contexto de ejecución, cree un entorno y establezca `pip_packages=['debugpy', 'azureml-sdk==<SDK-VERSION>']`. Cambie la versión del SDK para que coincida con la que está usando. El fragmento de código siguiente muestra cómo crear un entorno:

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
                                                                           pip_packages=['debugpy', 'azureml-sdk==<SDK-VERSION>'])
```

En la sección [Configuración de scripts de Python](#configure-python-scripts), se agregaron nuevos argumentos a los scripts usados por los pasos de la canalización de ML. En el fragmento de código siguiente se muestra cómo usar estos argumentos para habilitar la depuración del componente y establecer un tiempo de espera. También se muestra cómo usar el entorno creado anteriormente estableciendo `runconfig=run_config`:

```python
# Use RunConfig from a pipeline step
step1 = PythonScriptStep(name="train_step",
                         script_name="train.py",
                         arguments=['--remote_debug', '--remote_debug_connection_timeout', 300,'--remote_debug_client_ip','<VS-CODE-CLIENT-IP>','--remote_debug_port',5678],
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

1. Para instalar debugpy en el entorno de desarrollo de VS Code, use el comando siguiente:

    ```bash
    python -m pip install --upgrade debugpy
    ```

    Para obtener más información sobre cómo usar debugpy con VS Code, consulte [Depuración remota](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection).

1. Para configurar VS Code para comunicarse con el proceso de Azure Machine Learning que ejecuta el depurador, cree una nueva configuración de depuración:

    1. En VS Code, seleccione el menú __Depurar__ y, luego, seleccione __Abrir configuraciones__. Se abre un archivo denominado __launch.json__.

    1. En el archivo __launch.json__, busque la línea que contiene `"configurations": [` e inserte el texto siguiente después de ella. Cambie la entrada `"host": "<IP-ADDRESS>"` a la dirección IP devuelta en los registros de la sección anterior. Cambie la entrada `"localRoot": "${workspaceFolder}/code/step"` a un directorio local que contenga una copia del script que se está depurando:

        ```json
        {
            "name": "Azure Machine Learning Compute: remote debug",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "<IP-ADDRESS>",
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
        > El procedimiento recomendado, especialmente en el caso de las canalizaciones, es mantener los recursos de los scripts en directorios independientes para que el código sea pertinente solo para cada uno de los pasos. En este ejemplo, el valor de ejemplo `localRoot` hace referencia a `/code/step1`.
        >
        > Si va a depurar varios scripts en directorios diferentes, cree una sección de configuración independiente para cada script.

    1. Guarde el archivo __launch.json__.

### <a name="connect-the-debugger"></a>Conexión del depurador

1. Abra VS Code y abra una copia local del script.
2. Establezca puntos de interrupción donde desee que se detenga el script una vez que se haya conectado.
3. Mientras el proceso secundario ejecuta el script y se muestra `Timeout for debug connection` en los registros, use la tecla F5 o seleccione __Depurar__. Cuando se le solicite, seleccione la configuración __Azure Machine Learning Compute: remote debug__ (Proceso de Azure Machine Learning: depuración remota). También puede seleccionar el icono de depuración en la barra lateral, la entrada __Azure Machine Learning: remote debug__ (Azure Machine Learning: depuración remota) en el menú desplegable Depurar y, a continuación, usar la flecha verde para conectar el depurador.

    En este punto, VS Code se conecta a debugpy en el nodo de ejecución y se detiene en el punto de interrupción que se estableció anteriormente. Ahora puede recorrer el código a medida que se ejecuta, ver variables, etc.

    > [!NOTE]
    > Si el registro muestra una entrada que indica `Debugger attached = False`, el tiempo de espera ha expirado y el script ha continuado sin el depurador. Vuelva a enviar la canalización y conecte el depurador después del mensaje de `Timeout for debug connection` y antes de que expire el tiempo de espera.

## <a name="debug-and-troubleshoot-deployments"></a>Depuración y solución de problemas de implementaciones

En algunos casos, es posible que tenga que depurar interactivamente el código de Python incluido en la implementación de modelo. Por ejemplo, si el script de entrada presenta errores y no se puede determinar el motivo mediante un registro adicional. Mediante VS Code y debugpy, puede conectarse al código que se ejecuta en el contenedor de Docker.

> [!IMPORTANT]
> Este método de depuración no funciona cuando se usa `Model.deploy()` y `LocalWebservice.deploy_configuration` para implementar un modelo de manera local. En su lugar, debe crear una imagen con el método [Model.package()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#package-workspace--models--inference-config-none--generate-dockerfile-false-).

Las implementaciones de servicios web locales requieren una instalación de Docker en funcionamiento en el sistema local. Para obtener más información sobre el uso de Docker, consulte la [Documentación de Docker](https://docs.docker.com/). Tenga en cuenta que, al trabajar con instancias de proceso, Docker ya está instalado.

### <a name="configure-development-environment"></a>Configuración del entorno de desarrollo

1. Para instalar debugpy en el entorno de desarrollo de VS Code local, use el comando siguiente:

    ```bash
    python -m pip install --upgrade debugpy
    ```

    Para obtener más información sobre cómo usar debugpy con VS Code, consulte [Depuración remota](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection).

1. Para configurar VS Code para comunicarse con la imagen de Docker, cree una configuración de depuración nueva:

    1. En VS Code, seleccione el menú __Depurar__ y, luego, seleccione __Abrir configuraciones__. Se abre un archivo denominado __launch.json__.

    1. En el archivo __launch.json__, busque la línea que contiene `"configurations": [` e inserte el texto siguiente después de ella:

        ```json
        {
            "name": "Azure Machine Learning Deployment: Docker Debug",
            "type": "python",
            "request": "attach",
            "connect": {
                "port": 5678,
                "host": "0.0.0.0",
            },
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}",
                    "remoteRoot": "/var/azureml-app"
                }
            ]
        }
        ```

        > [!IMPORTANT]
        > Si ya hay otras entradas en la sección de configuraciones, agregue una coma (,) después del código que insertó.

        Esta sección se adjunta al contenedor de Docker mediante el puerto 5678.

    1. Guarde el archivo __launch.json__.

### <a name="create-an-image-that-includes-debugpy"></a>Creación de una imagen que incluye debugpy

1. Modifique el entorno de conda para la implementación de manera que incluya debugpy. En el ejemplo siguiente se muestra cómo se agrega con el parámetro `pip_packages`:

    ```python
    from azureml.core.conda_dependencies import CondaDependencies 


    # Usually a good idea to choose specific version numbers
    # so training is made on same packages as scoring
    myenv = CondaDependencies.create(conda_packages=['numpy==1.15.4',
                                'scikit-learn==0.19.1', 'pandas==0.23.4'],
                                 pip_packages = ['azureml-defaults==1.0.83', 'debugpy'])

    with open("myenv.yml","w") as f:
        f.write(myenv.serialize_to_string())
    ```

1. Para iniciar debugpy y esperar una conexión cuando se inicia el servicio, agregue lo siguiente en la parte superior del archivo `score.py`:

    ```python
    import debugpy
    # Allows other computers to attach to debugpy on this IP address and port.
    debugpy.listen(('0.0.0.0', 5678))
    # Wait 30 seconds for a debugger to attach. If none attaches, the script continues as normal.
    debugpy.wait_for_client()
    print("Debugger attached...")
    ```

1. Cree una imagen basada en la definición de entorno y extráigala en el Registro local. 

    > [!NOTE]
    > En este ejemplo se da por supuesto que `ws` apunta al área de trabajo de Azure Machine Learning y que `model` es el modelo que se está implementando. El archivo `myenv.yml` contiene las dependencias de conda creadas en el paso 1.

    ```python
    from azureml.core.conda_dependencies import CondaDependencies
    from azureml.core.model import InferenceConfig
    from azureml.core.environment import Environment


    myenv = Environment.from_conda_specification(name="env", file_path="myenv.yml")
    myenv.docker.base_image = None
    myenv.docker.base_dockerfile = "FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04"
    inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
    package = Model.package(ws, [model], inference_config)
    package.wait_for_creation(show_output=True)  # Or show_output=False to hide the Docker build logs.
    package.pull()
    ```

    Una vez que se haya creado y descargado la imagen, la ruta de acceso de la misma (incluye el repositorio, el nombre y la etiqueta, que en este caso también es su resumen) se mostrará en un mensaje similar al siguiente:

    ```text
    Status: Downloaded newer image for myregistry.azurecr.io/package@sha256:<image-digest>
    ```

1. Para facilitar el trabajo con la imagen, use el comando siguiente para agregar una etiqueta. Reemplace `myimagepath` por el valor de la ubicación del paso anterior.

    ```bash
    docker tag myimagepath debug:1
    ```

    Para el resto de los pasos, puede hacer referencia a la imagen local como `debug:1` en lugar del valor de la ruta de acceso completa a la imagen.

### <a name="debug-the-service"></a>Depuración del servicio

> [!TIP]
> Si establece un tiempo de expiración para la conexión de debugpy en el archivo `score.py`, debe conectar VS Code a la sesión de depuración antes de que se cumpla el tiempo de expiración. Inicie VS Code, abra la copia local de `score.py`, establezca un punto de interrupción y prepárelo antes de seguir los pasos de esta sección.
>
> Para más información sobre la depuración y el establecimiento de puntos de interrupción, consulte [Depuración](https://code.visualstudio.com/Docs/editor/debugging).

1. Para iniciar un contenedor de Docker con la imagen, use el comando siguiente:

    ```bash
    docker run -it --name debug -p 8000:5001 -p 5678:5678 -v <my_path_to_score.py>:/var/azureml-apps/score.py debug:1 /bin/bash
    ```

    Esto asocia el archivo `score.py` localmente al del contenedor. Por lo tanto, los cambios realizados en el editor se verán reflejados automáticamente en el contenedor.

1. En el contenedor, ejecute el comando siguiente en el shell.

    ```bash
    runsvdir /var/runit
    ```

1. Para adjuntar VS Code a debugpy dentro del contenedor, abra VS Code y use la tecla F5 o seleccione __Depurar__. Cuando se le solicite, seleccione la configuración __Azure Machine Learning Deployment: Docker Debug__ (Azure Machine Learning Service: depuración de Docker). También puede seleccionar el icono de depuración en la barra lateral, la entrada __Azure Machine Learning Deployment: Docker Debug__ (Azure Machine Learning Service: depuración de Docker) en el menú desplegable Depurar y, luego, use la flecha verde para adjuntar el depurador.

    ![El icono de depuración, el botón de inicio de la depuración y el selector de configuración](./media/how-to-troubleshoot-deployment/start-debugging.png)

En este punto, VS Code se conecta a debugpy dentro del contenedor de Docker y se detiene en el punto de interrupción que se estableció anteriormente. Ahora puede recorrer el código a medida que se ejecuta, ver variables, etc.

Para más información sobre cómo usar VS Code para implementar Python, consulte [Depurar el código de Python](https://docs.microsoft.com/visualstudio/python/debugging-python-in-visual-studio?view=vs-2019).

### <a name="stop-the-container"></a>Detención del contenedor

Para detener el contenedor, use el comando siguiente:

```bash
docker stop debug
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado Visual Studio Code remoto, puede usar una instancia de proceso como proceso remoto desde Visual Studio Code para depurar el código de forma interactiva. 

[Tutorial: Formar su primer modelo de ML](tutorial-1st-experiment-sdk-train.md) muestra cómo usar una instancia de proceso con un cuaderno integrado.
