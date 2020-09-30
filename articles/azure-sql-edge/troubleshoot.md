---
title: Solución de problemas de implementaciones de Azure SQL Edge
description: Obtenga información sobre los posibles errores al implementar Azure SQL Edge
keywords: SQL Edge, solución de problemas, errores de implementación
services: sql-edge
ms.service: sql-edge
ms.topic: troubleshooting
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: d8da8bcf3d2bb6b2af2b5c69ce003289d83d3884
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932283"
---
# <a name="troubleshooting-azure-sql-edge-deployments"></a>Solución de problemas de implementaciones de Azure SQL Edge 

En este artículo se proporciona información sobre los posibles errores que se pueden ver al implementar y usar contenedores de Azure SQL Edge y se ofrecen técnicas de solución de problemas para ayudar a resolver estas incidencias. 

Azure SQL Edge admite dos modelos de implementación: 
- Implementación conectada a través de Azure IoT Edge: Azure SQL Edge está disponible en Azure Marketplace y se puede implementar como un módulo para [Azure IoT Edge](../iot-edge/about-iot-edge.md). Para obtener más información, consulte [Implementación de Azure SQL Edge](deploy-portal.md).<br>

- Implementación desconectada: Las imágenes de contenedor de Azure SQL Edge se pueden extraer de Docker Hub e implementarse como un contenedor de Docker independiente o en un clúster de Kubernetes. Para obtener más información, vea [Implementación de Azure SQL Edge con Docker](disconnected-deployment.md) e [Implementación de un contenedor de Azure SQL Edge en Kubernetes](deploy-kubernetes.md).

## <a name="troubleshooting-iot-edge-device-and-deployments"></a>Solución de problemas de dispositivos e implementaciones de IoT Edge

Si se produce un error durante la implementación de SQL Edge a través de Azure IoT Edge, asegúrese de que el servicio `iotedge` esté correctamente configurado y en ejecución. Los siguientes documentos pueden ser útiles a la hora de solucionar incidencias relacionadas con Azure IoT Edge:
- [Incidencias habituales y soluciones para Azure IoT Edge](../iot-edge/troubleshoot-common-errors.md).
- [Solución de problemas del dispositivo IoT Edge](../iot-edge/troubleshoot.md)

## <a name="docker-command-errors"></a>Errores de comandos de Docker

Si obtiene errores para los comandos `docker`, asegúrese de que el servicio Docker se está ejecutando e intente ejecutar con permisos elevados.

Por ejemplo, en Linux, podría recibir el siguiente error al ejecutar los comandos `docker`:

```output
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
```

Si recibe este error en Linux, intente ejecutar los mismos comandos precedidos de `sudo`. Si se produce un error, compruebe que el servicio Docker se está ejecutando e inícielo si es necesario.

```bash
sudo systemctl status docker
sudo systemctl start docker
```

En Windows, compruebe que está iniciando PowerShell o el símbolo del sistema como administrador.

## <a name="azure-sql-edge-container-startup-errors"></a>Errores de inicio del contenedor de Azure SQL Edge

Si no se puede ejecutar el contenedor de SQL Edge, pruebe lo siguiente:

- Si está utilizando Azure IoT Edge, asegúrese de que las imágenes del módulo se han descargado correctamente y de que las variables de entorno y las opciones de creación del contenedor están especificadas correctamente en el manifiesto del módulo.

- Si usa la implementación basada en Docker o Kubernetes, asegúrese de que el comando `docker run` tiene el formato correcto. Para obtener más información, consulte [Implementación de Azure SQL Edge con Docker](disconnected-deployment.md) e [Implementación de un contenedor de Azure SQL Edge en Kubernetes](deploy-kubernetes.md).

- Si recibe un error como `failed to create endpoint CONTAINER_NAME on network bridge. Error starting proxy: listen tcp 0.0.0.0:1433 bind: address already in use.`, está intentando asignar el puerto del contenedor 1433 a un puerto que ya está en uso. Esto puede ocurrir si está ejecutando SQL Edge localmente en la máquina host. También puede ocurrir si inicia dos contenedores de SQL Edge e intenta asignarlos al mismo puerto de host. Si esto ocurre, use el parámetro `-p` para asignar el puerto del contenedor 1433 a otro puerto de host. Por ejemplo: 

    ```bash
    sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge-developer.
    ```

- Si recibe un error, como `Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.30tdout=1&tail=all: dial unix /var/run/docker.sock: connect: permission denied` al intentar iniciar un contenedor, agregue el usuario al grupo de Docker en Ubuntu. Después, cierre la sesión y vuelva a iniciarla, ya que este cambio afectará a las nuevas sesiones. 

   ```bash
    usermod -aG docker $USER
   ```

- Compruebe si hay algún mensaje de error del contenedor.

   ```bash
   docker logs e69e056c702d
   ```

- Si usa software de administración de contenedores, asegúrese de que admite procesos de contenedor que se ejecutan como raíz. El proceso sqlservr en el contenedor se ejecuta como raíz.

- De forma predeterminada, los contenedores de Azure SQL Edge se ejecutan como un usuario no raíz denominado `mssql`. Si usa puntos de montaje o volúmenes de datos para conservar los datos, asegúrese de que el usuario `mssql` tenga los permisos adecuados en el volumen. Para obtener más información, vea [Ejecución como usuario no raíz](configure.md#run-azure-sql-edge-as-non-root-user) y [Conservación de datos](configure.md#persist-your-data).

- Si el contenedor de Docker de SQL Edge sale inmediatamente después de iniciarse, compruebe los registros de Docker. Si usa PowerShell en Windows con el comando `docker run`, use comillas dobles en lugar de comillas simples. Con PowerShell Core, use comillas simples.

- Revise los [registros de errores de SQL Edge](#errorlogs).

## <a name="sql-edge-connection-failures"></a>Errores de conexión de SQL Edge

Si no puede conectarse a la instancia de SQL Edge que se ejecuta en el contenedor, pruebe lo siguiente:

- Asegúrese de que el contenedor de SQL Edge se está ejecutando examinando la columna **STATUS** de la salida de `docker ps -a`. Si no es así, use `docker start <Container ID>` para iniciarlo.

- Si ha asignado a un puerto de host no predeterminado (no 1433), asegúrese de que está especificando el puerto en la cadena de conexión. Puede ver la asignación de puertos en la columna **PORTS** de la salida de `docker ps -a`. Para obtener más información sobre cómo conectarse a Azure SQL Edge, consulte [Conexión a Azure SQL Edge y consulta](connect.md).

- Si anteriormente se ha implementado SQL Edge con un volumen de datos asignado o un contenedor de volúmenes de datos, y ahora usa el volumen de datos asignado existente o el contenedor de volúmenes de datos, SQL Edge omite el valor de la variable de entorno `MSSQL_SA_PASSWORD`. En su lugar, se usa la contraseña de usuario SA configurada anteriormente. Esto sucede porque SQL Edge reutiliza los archivos de bases de datos maestras existentes en el volumen asignado o en el contenedor de volúmenes de datos. Si experimenta esta incidencia, puede usar las opciones siguientes:

    - Conéctese con la contraseña usada anteriormente, si todavía sigue disponible.
    - Configure SQL Edge para usar otro volumen asignado o contenedor de volúmenes de datos.
    - Quite los archivos existentes de la base de datos maestra (master.mdf y mastlog.mdf) del volumen asignado o del contenedor de volúmenes de datos.

- Revise los [registros de errores de SQL Edge](#errorlogs).

## <a name="sql-edge-setup-and-error-logs"></a><a id="errorlogs"></a> Configuración y registros de errores de SQL Edge

De forma predeterminada, los registros de errores de SQL Edge están presentes en el directorio **/var/opt/mssql/log** en el contenedor y se puede acceder a ellos con cualquiera de las formas siguientes:

- Si montó un directorio host en **/var/opt/mssql** al crear el contenedor, puede buscar en el subdirectorio de **registro** de la ruta de acceso asignada en el host en su lugar.
- Mediante el uso de un símbolo del sistema interactivo para conectarse al contenedor. Si el contenedor no se está ejecutando, primero inicie el contenedor. Después, use un símbolo del sistema interactivo para inspeccionar los registros. Puede obtener el id. de contenedor si ejecuta el comando `docker ps`.

    ```bash
    docker start <ContainerID>
    docker exec -it <ContainerID> "/bin/bash"
    ```

    En la sesión de Bash dentro del contenedor, ejecute los siguientes comandos:

    ```bash
    cd /var/opt/mssql/log
    cat errorlog
    ```
- Si el contenedor de SQL Edge está en funcionamiento y puede conectarse a la instancia mediante herramientas de cliente, puede utilizar el procedimiento almacenado `sp_readerrorlog` para leer el contenido del registro de errores de SQL Edge.

## <a name="execute-commands-in-a-container"></a>Ejecución de comandos en un contenedor

Si tiene un contenedor en ejecución, puede ejecutar comandos dentro del contenedor desde un terminal del host.

Para obtener el identificador de contenedor, ejecute:

```bash
docker ps -a
```

Para iniciar un terminal de Bash en el contenedor, ejecute:

```bash
docker exec -it <Container ID> /bin/bash
```

Ahora puede ejecutar comandos como si los estuviera ejecutando en el terminal dentro del contenedor. Cuando termine, escriba `exit`. Esto cierra la sesión de comandos interactiva, pero el contenedor continúa ejecutándose.

## <a name="troubleshooting-issues-with-data-streaming"></a>Solución de problemas con el streaming de datos

De forma predeterminada, los registros del motor de streaming de Azure SQL Edge se escriben en un archivo denominado `current` en el directorio **/var/opt/mssql/log/services/00000001-0000-0000-0000-000000000000**. Se puede acceder al archivo directamente a través del volumen asignado o del contenedor de volúmenes de datos, o mediante el inicio de una sesión interactiva del símbolo del sistema en el contenedor de SQL Edge. 

Además, si puede conectarse a la instancia de SQL Edge con las herramientas de cliente, puede usar el comando siguiente de T-SQL para acceder al registro del motor de streaming actual. 

```sql

select value as log, try_convert(DATETIME2, substring(value, 0, 26)) as timestamp 
from 
    STRING_SPLIT
    (
        (
            select BulkColumn as logs
            FROM OPENROWSET (BULK '/var/opt/mssql/log/services/00000001-0000-0000-0000-000000000000/current', SINGLE_CLOB) MyFile
        ),
        CHAR(10)
    ) 
where datalength(value) > 0

```

### <a name="enabling-verbose-logging"></a>Habilitación del registro detallado

Si el nivel de registro predeterminado para el motor de streaming no proporciona suficiente información, el registro de depuración del motor de streaming se puede habilitar en SQL Edge. Para habilitar el registro de depuración, agregue la variable de entorno `RuntimeLogLevel=debug` a la implementación de SQL Edge. Después de habilitar el registro de depuración, intente reproducir el problema y compruebe si hay mensajes o excepciones relevantes en los registros. 



## <a name="next-steps"></a>Pasos siguientes

- [Aprendizaje automático e IA con ONNX en SQL Edge](onnx-overview.md).
- [Streaming de datos en Azure SQL Edge](stream-data.md)
- [Retención y limpieza de datos](data-retention-overview.md).
- [Relleno de intervalos de tiempo e imputación de los valores que faltan](imputing-missing-values.md).







