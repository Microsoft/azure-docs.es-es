---
title: Telemetría y registro de contenedores de análisis espacial
titleSuffix: Azure Cognitive Services
description: El análisis espacial proporciona a cada contenedor una configuración común de información, registro y seguridad del marco de trabajo de configuración.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/11/2020
ms.author: aahi
ms.openlocfilehash: 2d19c061ad1e5cf033d2801df64a0ae37736c418
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90983022"
---
# <a name="telemetry-and-troubleshooting"></a>Telemetría y solución de problemas

El análisis espacial incluye un conjunto de características para supervisar el mantenimiento del sistema y ayudar a diagnosticar problemas.

## <a name="enable-visualizations"></a>Habilitación de visualizaciones

Para habilitar una visualización de eventos de Conclusiones de IA en un fotograma de vídeo, debe usar la versión `.debug` de una [operación de análisis espacial](spatial-analysis-operations.md). Hay cuatro operaciones de depuración disponibles.

Edite el [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179) para usar el valor correcto para la variable de entorno `DISPLAY`. Debe coincidir con la variable `$DISPLAY` del equipo host. Después de actualizar el manifiesto de implementación, vuelva a implementar el contenedor.

Una vez finalizada la implementación, es posible que tenga que copiar el archivo `.Xauthority` del equipo host en el contenedor y reiniciarlo. En el ejemplo siguiente, `peopleanalytics` es el nombre del contenedor del equipo host.

```bash
sudo docker cp $XAUTHORITY peopleanalytics:/root/.Xauthority
sudo docker stop peopleanalytics
sudo docker start peopleanalytics
xhost +
```


## <a name="collect-system-health-telemetry"></a>Recopilación de telemetría de mantenimiento del sistema

Telegraf es una imagen de código abierto que funciona con el análisis espacial y está disponible en Microsoft Container Registry. Toma las siguientes entradas y las envía a Azure Monitor. El módulo Telegraf se puede compilar con las entradas y salidas personalizadas que se quiera. La configuración del módulo Telegraf en el análisis espacial forma parte del [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179). Este módulo es opcional y se puede quitar del manifiesto si no se necesita. 

Entradas: 
1. Métricas de análisis espacial
2. Métricas de disco
3. Métricas de CPU
4. Métricas de Docker
5. Métricas de GPU

Salidas:
1. Azure Monitor

El módulo Telegraf de análisis espacial proporcionado publicará todos los datos de telemetría emitidos por el contenedor de análisis espacial en Azure Monitor. Consulte [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview) para más información sobre cómo agregar Azure Monitor a su suscripción.

Después de configurar Azure Monitor, tendrá que crear credenciales que habiliten el módulo para enviar telemetría. Puede usar Azure Portal para crear una entidad de servicio o usar el siguiente comando de la CLI de Azure para crear una.

> [!NOTE] 
> Este comando requiere que tenga privilegios de propietario en la suscripción. 

```bash
# Find your Azure IoT Hub resource ID by running this command. The resource ID  should start with something like 
# "/subscriptions/b60d6458-1234-4be4-9885-c7e73af9ced8/resourceGroups/...”
az iot hub list

# Create a Service Principal with `Monitoring Metrics Publisher` role in the IoTHub resource:
# Save the output from this command. The values will be used in the deployment manifest. The password won't be shown again so make sure to write it down
az ad sp create-for-rbac --role="Monitoring Metrics Publisher" --name "<principal name>" --scopes="<resource ID of IoT Hub>"
```

En el [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179), busque el módulo *Telegraf* y reemplace los siguientes valores por la información de la entidad de servicio del paso anterior y vuelva a implementarlo.

```json

"telegraf": { 
  "settings": {
  "image":   "mcr.microsoft.com/azure-cognitive-services/vision/spatial-analysis/telegraf:1.0",
  "createOptions":   "{\"HostConfig\":{\"Runtime\":\"nvidia\",\"NetworkMode\":\"azure-iot-edge\",\"Memory\":33554432,\"Binds\":[\"/var/run/docker.sock:/var/run/docker.sock\"]}}"
},
"type": "docker",
"env": {
    "AZURE_TENANT_ID": {
        "value": "<Tenant Id>"
    },
    "AZURE_CLIENT_ID": {
        "value": "Application Id"
    },
    "AZURE_CLIENT_SECRET": {
        "value": "<Password>"
    },
    "region": {
        "value": "<Region>"
    },
    "resource_id": {
        "value": "/subscriptions/{subscriptionId}/resourceGroups/{resoureGroupName}/providers/Microsoft.Devices/IotHubs/{IotHub}"
    },
...
```

Cuando haya implementado el módulo Telegraf, se puede acceder a las métricas notificadas a través del servicio Azure Monitor o seleccionando **Supervisión** en la instancia de IoT Hub en Azure Portal.

:::image type="content" source="./media/spatial-analysis/iot-hub-telemetry.png" alt-text="Informe de telemetría de Azure Monitor":::

### <a name="system-health-events"></a>Eventos de mantenimiento del sistema

| Nombre del evento | Descripción|
|------|---------|
|archon_exit    |Se envía cuando un usuario cambia el estado del módulo de análisis espacial de *en ejecución* a *detenido*.  |
|archon_error   |Se envía cuando alguno de los procesos se bloquea en el contenedor. Se trata de un error crítico.  |
|InputRate  |Velocidad a la que el gráfico procesa la entrada de vídeo. Se notifica cada cinco minutos. | 
|OutputRate     |Velocidad a la que el gráfico genera Conclusiones de IA. Se notifica cada cinco minutos. |
|archon_allGraphsStarted | Se envía cuando todos los gráficos han terminado de iniciarse. |
|archon_configchange    | Se envía cuando cambia la configuración de un gráfico. |
|archon_graphCreationFailed     |Se envía cuando no se puede iniciar el gráfico con la `graphId` notificada. |
|archon_graphCreationSuccess    |Se envía cuando el gráfico con la `graphId` notificada se inicia correctamente. |
|archon_graphCleanup    | Se envía cuando el gráfico con la `graphId` notificada se limpia y se cierra. |
|archon_graphHeartbeat  |Latido que se envía cada minuto por cada gráfico de una aptitud. |
|archon_apiKeyAuthFail |Se envía cuando se produce un error en la clave de recurso de Computer Vision para autenticar el contenedor durante más de 24 horas, debido a los siguientes motivos: Cuota agotada, no válido, sin conexión. |
|VideoIngesterHeartbeat     |Se envía cada hora para indicar que el vídeo se transmite desde el origen de vídeo, con el número de errores en esa hora. Se notifica para cada gráfico. |
|VideoIngesterState | Notifica el estado *Detenido* o *Iniciado* para el streaming de vídeo. Se notifica para cada gráfico. |

##  <a name="troubleshooting-an-iot-edge-device"></a>Solución de problemas de un dispositivo IoT Edge

Puede usar la herramienta de línea de comandos `iotedge` para comprobar el estado y los registros de los módulos en ejecución. Por ejemplo:
* `iotedge list`: Notifica una lista de módulos en ejecución. 
  También puede comprobar si hay errores con `iotedge logs edgeAgent`. Si `iotedge` se bloquea, puede tratar de reiniciarlo con `iotedge restart edgeAgent`
* `iotedge logs <module-name>`
* `iotedge restart <module-name>` para reiniciar un módulo específico 

## <a name="collect-log-files-with-the-diagnostics-container"></a>Recopilación de archivos de registro con el contenedor de diagnósticos

El análisis espacial genera registros de depuración de Docker que puede usar para diagnosticar problemas en tiempo de ejecución o incluir en incidencias de soporte técnico. El módulo de diagnóstico de análisis espacial está disponible en Microsoft Container Registry para su descarga. En el [manifiesto de implementación](https://go.microsoft.com/fwlink/?linkid=2142179) de ejemplo, busque el módulo de *diagnóstico*.

En la sección "env", agregue la siguiente configuración:

```json
"diagnostics": {  
  "settings": {
  "image":   "mcr.microsoft.com/azure-cognitive-services/vision/spatial-analysis/diagnostics:1.0",
  "createOptions":   "{\"HostConfig\":{\"Mounts\":[{\"Target\":\"/usr/bin/docker\",\"Source\":\"/home/data/docker\",\"Type\":\"bind\"},{\"Target\":\"/var/run\",\"Source\":\"/run\",\"Type\":\"bind\"}],\"LogConfig\":{\"Config\":{\"max-size\":\"500m\"}}}}"
  }
```    

>[!NOTE]
> Si no está ejecutando en un entorno de Kubernetes de ASE, reemplace las opciones de creación del contenedor del módulo de registro por lo siguiente:
>
>`"createOptions": "{\"HostConfig\": {\"Binds\": [\"/var/run/docker.sock:/var/run/docker.sock\",\"/usr/bin/docker:/usr/bin/docker\"],\"LogConfig\": {\"Config\": {\"max-size\": \"500m\"}}}}"`

Para optimizar los registros cargados en un punto de conexión remoto, como Azure Blob Storage, se recomienda mantener un tamaño de archivo pequeño. Consulte el ejemplo siguiente para ver la configuración recomendada de los registros de Docker.

```json
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "500m",
                "max-file": "1000"
            }
        }
    }
}
```

### <a name="configure-the-log-level"></a>Configuración del nivel de registro

La configuración del nivel de registro permite controlar el nivel de detalle de los registros generados. Los niveles de registro admitidos son: `none`, `verbose`, `info`, `warning` y `error`. El nivel detallado de registro predeterminado para los nodos y la plataforma es `info`. 

Los niveles de registro se pueden modificar globalmente si se establece la variable de entorno `ARCHON_LOG_LEVEL` en uno de los valores permitidos.
También pueden establecerse a través del documento del módulo gemelo IoT Edge, ya sea globalmente, para todas las aptitudes implementadas, o para cada aptitud específica estableciendo los valores de `platformLogLevel` y `nodeLogLevel` tal y como se muestra a continuación.

```json
{
    "version": 1,
    "properties": {
        "desired": {
            "globalSettings": {
                "platformLogLevel": "verbose"
            },
            "graphs": {
                "samplegraph": {
                    "nodeLogLevel": "verbose",
                    "platformLogLevel": "verbose"
                }
            }
        }
    }
}
```

### <a name="collecting-logs"></a>Recopilación de registros

> [!NOTE]
> El módulo `diagnostics` no afecta al contenido del registro, solo sirve para recopilar, filtrar y cargar los registros existentes.
> Para usar este módulo, debe tener la versión 1.40 o superior de la API de Docker.

El archivo de [manifiesto de implementación de ejemplo](https://go.microsoft.com/fwlink/?linkid=2142179) incluye un módulo denominado `diagnostics` que recopila y carga registros. Este módulo está deshabilitado de forma predeterminada y debe habilitarse a través de la configuración del módulo IoT Edge cuando tenga que acceder a los registros. 

La recopilación de `diagnostics` se realiza a petición y se controla a través de un método directo de IoT Edge, y puede enviar registros a una instancia de Azure Blob Storage.

### <a name="configure-diagnostics-upload-targets"></a>Configuración de destinos de carga de diagnósticos

En el portal de IoT Edge, seleccione el dispositivo y luego el módulo de **diagnóstico**. En el archivo de ejemplo [*DeploymentManifest.json*](https://go.microsoft.com/fwlink/?linkid=2142179), busque la sección de **variables de entorno** de diagnóstico, denominada "env", y agregue la siguiente información:

**Configuración de la carga en Azure Blob Storage**

1. Cree su propia cuenta de Azure Blob Storage, si todavía no lo ha hecho.
2. Obtenga la **Cadena de conexión** de la cuenta de almacenamiento en Azure Portal. La encontrará en **Claves de acceso**.
3. Los registros de análisis espacial se cargarán automáticamente en un contenedor de Blob Storage denominado *rtcvlogs* con el siguiente formato de nombre de archivo: `{CONTAINER_NAME}/{START_TIME}-{END_TIME}-{QUERY_TIME}.log`.

```json
"env":{
    "IOTEDGE_WORKLOADURI":"fd://iotedge.socket",
    "AZURE_STORAGE_CONNECTION_STRING":"XXXXXX",   //from the Azure Blob Storage account
    "ARCHON_LOG_LEVEL":"info"
}
```

### <a name="uploading-spatial-analysis-logs"></a>Carga de registros de análisis espacial

Los registros se cargan a petición con el método `getRTCVLogs` de IoT Edge, en el módulo `diagnostics`. 


1. Vaya a la página del portal de IoT Hub, seleccione **dispositivos perimetrales** y elija el dispositivo y el módulo de diagnóstico. 
2. Vaya a la página de detalles del módulo y haga clic en la pestaña ***método directo***.
3. Escriba `getRTCVLogs` como Nombre del método y una cadena de formato JSON en la carga. Puede escribir `{}`, que es una carga vacía. 
4. Establezca los tiempos de espera de conexión y método y haga clic en **Invocar método**.
5. Seleccione el contenedor de destino y cree una cadena JSON de carga con los parámetros descritos en la sección **Sintaxis de registro**. Haga clic en **Invocar método** para realizar la solicitud.

>[!NOTE]
> Al invocar el método `getRTCVLogs` con una carga vacía, se devolverá una lista de todos los contenedores implementados en el dispositivo. El nombre del método distingue mayúsculas de minúsculas. Obtendrá un error 501 si se especifica un nombre de método incorrecto.

:::image type="content" source="./media/spatial-analysis/direct-log-collection.png" alt-text="Invocación del método getRTCVLogs ":::
![Página de método directo getRTCVLogs](./media/spatial-analysis/direct-log-collection.png)

 
### <a name="logging-syntax"></a>Sintaxis de registro

En la tabla siguiente se muestran los parámetros que puede usar al consultar registros.

| Palabra clave | Descripción | Valor predeterminado |
|--|--|--|
| StartTime | Hora de inicio de los registros deseados, en milisegundos UTC. | `-1`, el inicio del tiempo de ejecución del contenedor. Cuando se usa `[-1.-1]` como intervalo de tiempo, la API devuelve los registros de la última hora.|
| EndTime | Hora de finalización de los registros deseados, en milisegundos UTC. | `-1`, la hora actual. Cuando se usa el intervalo de tiempo `[-1.-1]`, la API devuelve los registros de la última hora. |
| ContainerId | Contenedor de destino para la captura de registros.| `null`, cuando no hay ningún identificador de contenedor. La API devuelve toda la información de contenedores disponibles con identificadores.|
| DoPost | Realice la operación de carga. Cuando se establece en `false`, realiza la operación solicitada y devuelve el tamaño de carga sin realizar la carga. Cuando se establece en `true`, se iniciará la carga asincrónica de los registros seleccionados. | `false`, no cargar.|
| Limitación | Se indica el número de líneas de registros que se van a cargar por lote. | `1000`, use este parámetro para ajustar la velocidad de POST. |
| Filtros | Filtra los registros que se van a cargar. | `null`, los filtros se pueden especificar como pares de clave y valor en función de la estructura de registros de análisis espacial: `[UTC, LocalTime, LOGLEVEL,PID, CLASS, DATA]`. Por ejemplo: `{"TimeFilter":[-1,1573255761112]}, {"TimeFilter":[-1,1573255761112]}, {"CLASS":["myNode"]`|

En la tabla siguiente se muestran los atributos de la respuesta de la consulta.

| Palabra clave | Descripción|
|--|--|
|DoPost| Puede ser *true* o *false*. Indica si los registros se han cargado o no. Cuando elige no cargar los registros, la API devuelve información ***sincrónicamente***. Cuando elige cargar registros, la API devuelve 200, si la solicitud es válida, e inicia la carga de registros ***asincrónicamente***.|
|TimeFilter| Filtro de tiempo aplicado a los registros.|
|ValueFilters| Filtros de palabra clave aplicados a los registros. |
|TimeStamp| Hora de inicio de ejecución del método. |
|ContainerId| Identificador del contenedor de destino. |
|FetchCounter| Número total de líneas de registro. |
|FetchSizeInByte| Cantidad total de datos de registro en bytes. |
|MatchCounter| Número válido de líneas de registro. |
|MatchSizeInByte| Cantidad válida de datos de registro en bytes. |
|FilterCount| Número total de líneas de registro después de aplicar el filtro. |
|FilterSizeInByte| Cantidad total de datos de registro en bytes después de aplicar el filtro. |
|FetchLogsDurationInMiliSec| Duración de la operación FETCH. |
|PaseLogsDurationInMiliSec| Duración de la operación de filtrado. |
|PostLogsDurationInMiliSec| Duración de la operación POST. |

#### <a name="example-request"></a>Solicitud de ejemplo 

```json
{
    "StartTime": -1,
    "EndTime": -1,
    "ContainerId": "5fa17e4d8056e8d16a5a998318716a77becc01b36fde25b3de9fde98a64bf29b",
    "DoPost": false,
    "Filters": null
}
```

#### <a name="example-response"></a>Respuesta de ejemplo 

```json
{
    "status": 200,
    "payload": {
        "DoPost": false,
        "TimeFilter": [-1, 1581310339411],
        "ValueFilters": {},
        "Metas": {
            "TimeStamp": "2020-02-10T04:52:19.4365389+00:00",
            "ContainerId": "5fa17e4d8056e8d16a5a998318716a77becc01b36fde25b3de9fde98a64bf29b",
            "FetchCounter": 61,
            "FetchSizeInByte": 20470,
            "MatchCounter": 61,
            "MatchSizeInByte": 20470,
            "FilterCount": 61,
            "FilterSizeInByte": 20470,
            "FetchLogsDurationInMiliSec": 0,
            "PaseLogsDurationInMiliSec": 0,
            "PostLogsDurationInMiliSec": 0
        }
    }
}
```

Compruebe las líneas, las horas y los tamaños del registro de captura, y si la configuración es correcta, reemplace ***DoPost*** por `true` y eso insertará los registros con los mismos filtros en los destinos. 

Puede exportar registros desde la instancia de Azure Blob Storage al solucionar problemas. 

## <a name="common-issues"></a>Problemas comunes

Si ve el siguiente mensaje en los registros del módulo, es posible que se deba aprobar su suscripción de Azure: 

"El contenedor no tiene un estado válido. Error de validación de la suscripción con el estado 'No coincidente'. La clave de API no está pensada para el tipo de contenedor especificado".

Para obtener más información, consulte [Solicitud de aprobación para ejecutar el contenedor](spatial-analysis-container.md#request-approval-to-run-the-container). 


## <a name="troubleshooting-the-azure-stack-edge-device"></a>Solución de problemas del dispositivo de Azure Stack Edge

La siguiente sección se ofrece como ayuda con la depuración y comprobación del estado del dispositivo de Azure Stack Edge.

### <a name="access-the-kubernetes-api-endpoint"></a>Acceda al punto de conexión de API de Kubernetes. 

1. En la interfaz de usuario local del dispositivo, vaya a la página **Dispositivos**. 
2. En **Puntos de conexión del dispositivo**, copie el punto de conexión de servicio API de Kubernetes. Este punto de conexión es una cadena en el formato siguiente: `https://compute..[device-IP-address]`.
3. Guarde la cadena de punto de conexión. La usará más adelante al configurar `kubectl` para acceder al clúster de Kubernetes.

### <a name="connect-to-powershell-interface"></a>Conexión a la interfaz de PowerShell

Conéctese de forma remota desde un cliente de Windows. Una vez creado el clúster de Kubernetes, puede administrar las aplicaciones a través de este clúster. Tendrá que conectarse a la interfaz de PowerShell del dispositivo. Según el sistema operativo del cliente, es posible que los procedimientos para conectarse de forma remota al dispositivo sean distintos. Los pasos siguientes corresponden a un cliente de Windows que ejecuta PowerShell.

> [!TIP]
> * Antes de comenzar, asegúrese de que el cliente de Windows ejecuta Windows PowerShell 5.0 o posterior.
> * PowerShell también está [disponible en Linux](https://docs.microsoft.com/powershell/scripting/install/installing-powershell-core-on-linux).

1. Ejecute una sesión de Windows PowerShell como administrador. 
    1. Asegúrese de que el servicio Administración remota de Windows se ejecuta en el cliente. En el símbolo del sistema, escriba `winrm quickconfig`.

2. Asigne una variable a la dirección IP del dispositivo. Por ejemplo, `$ip = "<device-ip-address>"`.

3. Use el siguiente comando para agregar la dirección IP del dispositivo a la lista de hosts de confianza del cliente. 

    ```powershell
    Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force
    ```
 
4. Inicie una sesión de Windows PowerShell en el dispositivo. 

    ```powershell
    Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell
    ```

5. Proporcione la contraseña cuando se le solicite. Use la misma contraseña que se emplea para iniciar sesión en la interfaz web local. La contraseña predeterminada de la interfaz web local es `Password1`. 

### <a name="access-the-kubernetes-cluster"></a>Acceso al clúster de Kubernetes

Una vez creado el clúster de Kubernetes, puede usar la herramienta de línea de comandos `kubectl` para acceder al clúster.

1. Cree un espacio de nombres. 

    ```powershell
    New-HcsKubernetesNamespace -Namespace
    ```

2. Cree un usuario y obtenga un archivo de configuración. Este comando generará información de configuración del clúster de Kubernetes. Copie esta información y guárdela en un archivo denominado *config*. No guarde el archivo con una extensión de archivo.
    
    ```powershell
    New-HcsKubernetesUser -UserName
    ```

3. Agregue el archivo *config* a la carpeta *.kube* del perfil de usuario en el equipo local.   

4. Asocie el espacio de nombres con el usuario que ha creado.

    ```powershell
    Grant-HcsKubernetesNamespaceAccess -Namespace -UserName
    ```

5. Instale `kubectl` en el cliente de Windows con el siguiente comando:
    
    ```powershell
    curl https://storage.googleapis.com/kubernetesrelease/release/v1.15.2/bin/windows/amd64/kubectl.exe -O kubectl.exe
    ```

6. Agregue una entrada DNS al archivo de hosts en el sistema. 
    1. Ejecute el Bloc de notas como administrador y abra el archivo *hosts* que se encuentra en `C:\windows\system32\drivers\etc\hosts`. 
    2. Cree una entrada en el archivo hosts con la dirección IP del dispositivo y el dominio DNS que obtuvo en la página **Dispositivo** de la interfaz de usuario local. El punto de conexión que debe usar tendrá un aspecto similar al siguiente: `https://compute.asedevice.microsoftdatabox.com/10.100.10.10`.

7. Compruebe que puede conectarse a los pods de Kubernetes.

    ```powershell
    kubectl get pods -n "iotedge"
    ```

Para obtener los registros de contenedor, ejecute el siguiente comando:

```powershell
kubectl logs <pod-name> -n <namespace> --all-containers
```

### <a name="useful-commands"></a>Comandos útiles

|Get-Help  |Descripción  |
|---------|---------|
|`Get-HcsKubernetesUserConfig -AseUser`     | Genera un archivo de configuración de Kubernetes. Al usar el comando, copie la información en un archivo denominado *config*. No guarde el archivo con una extensión de archivo.        |
| `Get-HcsApplianceInfo` | Devuelve información básica sobre el dispositivo. |
| `Enable-HcsSupportAccess` | Genera las credenciales de acceso para iniciar una sesión de soporte técnico. |

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de una aplicación web de recuento de personas](spatial-analysis-web-app.md)
* [Configuración de operaciones de análisis espaciales](./spatial-analysis-operations.md)
* [Guía de selección de ubicación de la cámara](spatial-analysis-camera-placement.md)
* [Guía de selección de ubicación de zonas y líneas](spatial-analysis-zone-line-placement.md)
