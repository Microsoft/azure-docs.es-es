---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/30/2020
ms.author: alkohli
ms.openlocfilehash: 92ccb6127e624ace9e719ffd23324b3a1b971f72
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89272156"
---
En un dispositivo Azure Stack Edge que tenga configurado el rol de proceso, puede solucionar problemas o supervisar el dispositivo con dos conjuntos diferentes de comandos.

- Uso de comandos de `iotedge` Estos comandos están disponibles para las operaciones básicas del dispositivo.
- Uso de comandos de `dkrdbe` Estos comandos están disponibles para un amplio conjunto de operaciones para el dispositivo.

Para ejecutar cualquiera de los comandos anteriores, debe [conectarse a la interfaz de PowerShell](#connect-to-the-powershell-interface).

### <a name="use-iotedge-commands"></a>Usar comandos `iotedge`

Para ver una lista de comandos disponibles, [conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface) y utilice la función `iotedge`.

```powershell
[10.100.10.10]: PS>iotedge -?                                                                                                                                                                                                 Usage: iotedge COMMAND

Commands:
   check
   list
   logs
   restart

[10.100.10.10]: PS>
```

En la tabla siguiente se proporciona una breve descripción de los comandos disponibles para `iotedge`:

|command  |Descripción |
|---------|---------|
|`check`     | Realiza comprobaciones automatizadas de problemas comunes de configuración y conectividad.       |
|`list`     | Enumere los módulos         |
|`logs`     | Captura los registros de un módulo.        |
|`restart`     | Detiene y reinicia un módulo.         |


### <a name="use-dkrdbe-commands"></a>Usar comandos `dkrdbe`

Para ver una lista de comandos disponibles, [conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface) y utilice la función `dkrdbe`.

```powershell
[10.100.10.10]: PS>dkrdbe -?
Usage: dkrdbe COMMAND

Commands:
   image [prune]
   images
   inspect
   login
   logout
   logs
   port
   ps
   pull
   start
   stats
   stop
   system [df]
   top

[10.100.10.10]: PS>
```
En la tabla siguiente se proporciona una breve descripción de los comandos disponibles para `dkrdbe`:

|command  |Descripción |
|---------|---------|
|`image`     | Administrar imágenes. Para quitar imágenes sin usar, utilice: `dkrdbe image prune -a -f`       |
|`images`     | Lista de imágenes         |
|`inspect`     | Devolver información de bajo nivel sobre los objetos de Docker         |
|`login`     | Iniciar sesión en un registro de Docker         |
|`logout`     | Cerrar sesión de un registro de Docker         |
|`logs`     | Capturar los registros de un contenedor        |
|`port`     | Enumerar asignaciones de puerto o una asignación específica para el contenedor        |
|`ps`     | Enumerar contenedores        |
|`pull`     | Extraer una imagen o un repositorio de un registro         |
|`start`     | Iniciar uno o varios contenedores detenidos         |
|`stats`     | Mostrar streaming en vivo de estadísticas de uso de recursos de los contenedores         |
|`stop`     | Detener uno o varios contenedores en ejecución        |
|`system`     | Administrar Docker         |
|`top`     | Mostrar los procesos en ejecución de un contenedor         |

Para obtener ayuda sobre cualquier comando disponible, use `dkrdbe <command-name> --help`.

Por ejemplo, para comprender el uso del comando `port`, escriba:

```powershell
[10.100.10.10]: P> dkrdbe port --help

Usage:  dkr port CONTAINER [PRIVATE_PORT[/PROTO]]

List port mappings or a specific mapping for the container
[10.100.10.10]: P> dkrdbe login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
[10.100.10.10]: PS>
```

Los comandos disponibles para la función `dkrdbe` utilizan los mismos parámetros que los que se utilizan para los comandos de Docker normales. Para las opciones y los parámetros utilizados con el comando de Docker, vaya a [Uso de la línea de comandos de Docker](https://docs.docker.com/engine/reference/commandline/docker/).

### <a name="to-check-if-the-module-deployed-successfully"></a>Comprobación de si el módulo se ha implementado correctamente

Los módulos de proceso son contenedores que tienen una lógica de negocios implementada. Para comprobar si un módulo de proceso se ha implementado correctamente, ejecute el comando `ps` y compruebe si el contenedor (correspondiente al módulo de proceso) se ejecuta.

Para obtener la lista de todos los contenedores (incluidos aquellos que están en pausa), ejecute el comando `ps -a`.

```powershell
[10.100.10.10]: P> dkrdbe ps -a
CONTAINER ID        IMAGE                                                COMMAND                   CREATED             STATUS              PORTS                                                                  NAMES
d99e2f91d9a8        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  movefile
0a06f6d605e9        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  filemove
2f8a36e629db        mcr.microsoft.com/azureiotedge-hub:1.0               "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days           0.0.0.0:443->443/tcp, 0.0.0.0:5671->5671/tcp, 0.0.0.0:8883->8883/tcp   edgeHub
acce59f70d60        mcr.microsoft.com/azureiotedge-agent:1.0             "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days                                                                                  edgeAgent
[10.100.10.10]: PS>
```

Si se ha producido un error al crear la imagen de contenedor o durante la extracción de la imagen, ejecute `logs edgeAgent`.  `EdgeAgent` es el contenedor de tiempo de ejecución de IoT Edge, que es responsable del aprovisionamiento de otros contenedores.

Dado que `logs edgeAgent` hace un volcado de todos los registros, una buena forma de ver los errores recientes consiste en usar la opción `--tail 20`.


```powershell
[10.100.10.10]: PS>dkrdbe logs edgeAgent --tail 20
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Util.Uds.HttpUdsMessageHandler] - Connected socket /var/run/iotedge/mgmt.sock
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Util.Uds.HttpUdsMessageHandler] - Sending request http://mgmt.sock/modules?api-version=2018-06-28
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Core.Agent] - Getting edge agent config...
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Core.Agent] - Obtained edge agent config
2019-02-28 23:38:23.469 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Edgelet.ModuleManagementHttpClient] - Received a valid Http response from unix:///var/run/iotedge/mgmt.soc
k for List modules
--------------------CUT---------------------
--------------------CUT---------------------
08:28.1007774+00:00","restartCount":0,"lastRestartTimeUtc":"2019-02-26T20:08:28.1007774+00:00","runtimeStatus":"running","version":"1.0","status":"running","restartPolicy":"always
","type":"docker","settings":{"image":"edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64","imageHash":"sha256:47778be0602fb077d7bc2aaae9b0760fbfc7c058bf4df192f207ad6cbb96f7cc","c
reateOptions":"{\"HostConfig\":{\"Binds\":[\"/home/hcsshares/share4-dl460:/home/input\",\"/home/hcsshares/share4-iot:/home/output\"]}}"},"env":{}}
2019-02-28 23:38:28.480 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Core.Planners.HealthRestartPlanner] - HealthRestartPlanner created Plan, with 0 command(s).
```

### <a name="to-get-container-logs"></a>Para obtener registros de contenedores

Para obtener los registros de un contenedor específico, primero enumere el contenedor y después obtenga los registros del contenedor en el que está interesado.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Para obtener la lista de contenedores en ejecución, ejecute el comando `ps`.

    ```powershell
    [10.100.10.10]: P> dkrdbe ps
    CONTAINER ID        IMAGE                                                COMMAND                   CREATED             STATUS              PORTS                                                                  NAMES
    d99e2f91d9a8        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  movefile
    0a06f6d605e9        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  filemove
    2f8a36e629db        mcr.microsoft.com/azureiotedge-hub:1.0               "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days           0.0.0.0:443->443/tcp, 0.0.0.0:5671->5671/tcp, 0.0.0.0:8883->8883/tcp   edgeHub
    acce59f70d60        mcr.microsoft.com/azureiotedge-agent:1.0             "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days                                                                                  edgeAgent
    ```

3. Anote el identificador del contenedor para el que necesita los registros.

4. Para obtener los registros de un contenedor específico, ejecute el comando `logs` proporcionando el identificador del contenedor.

    ```powershell
    [10.100.10.10]: PS>dkrdbe logs d99e2f91d9a8
    02/26/2019 18:21:45: Info: Opening module client connection.
    02/26/2019 18:21:46: Info: Initializing with input: /home/input, output: /home/output.
    02/26/2019 18:21:46: Info: IoT Hub module client initialized.
    02/26/2019 18:22:24: Info: Received message: 1, SequenceNumber: 0 CorrelationId: , MessageId: 081886a07e694c4c8f245a80b96a252a Body: [{"ChangeType":"Created","ShareRelativeFilePath":"\\__Microsoft Data Box Edge__\\Upload\\Errors.xml","ShareName":"share4-dl460"}]
    02/26/2019 18:22:24: Info: Moving input file: /home/input/__Microsoft Data Box Edge__/Upload/Errors.xml to /home/output/__Microsoft Data Box Edge__/Upload/Errors.xml
    02/26/2019 18:22:24: Info: Processed event.
    02/26/2019 18:23:38: Info: Received message: 2, SequenceNumber: 0 CorrelationId: , MessageId: 30714d005eb048e7a4e7e3c22048cf20 Body: [{"ChangeType":"Created","ShareRelativeFilePath":"\\f [10]","ShareName":"share4-dl460"}]
    02/26/2019 18:23:38: Info: Moving input file: /home/input/f [10] to /home/output/f [10]
    02/26/2019 18:23:38: Info: Processed event.
    ```

### <a name="to-monitor-the-usage-statistics-of-the-device"></a>Supervisión de las estadísticas de uso del dispositivo

Para supervisar la memoria, el uso de CPU y la E/S en el dispositivo, use el comando `stats`.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Ejecute el comando `stats` para deshabilitar el streaming en vivo y extraer solo el primer resultado.

   ```powershell
   dkrdbe stats --no-stream
   ```

   En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```
    [10.100.10.10]: P> dkrdbe stats --no-stream
    CONTAINER ID        NAME          CPU %         MEM USAGE / LIMIT     MEM %         NET I/O             BLOCK I/O           PIDS
    d99e2f91d9a8        movefile      0.0           24.4MiB / 62.89GiB    0.04%         751kB / 497kB       299kB / 0B          14
    0a06f6d605e9        filemove      0.00%         24.11MiB / 62.89GiB   0.04%         679kB / 481kB       49.5MB / 0B         14
    2f8a36e629db        edgeHub       0.18%         173.8MiB / 62.89GiB   0.27%         4.58MB / 5.49MB     25.7MB / 2.19MB     241
    acce59f70d60        edgeAgent     0.00%         35.55MiB / 62.89GiB   0.06%         2.23MB / 2.31MB     55.7MB / 332kB      14
    [10.100.10.10]: PS>
    ```

