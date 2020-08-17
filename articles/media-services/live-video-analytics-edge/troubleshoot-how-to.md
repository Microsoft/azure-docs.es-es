---
title: Solución de problemas de Live Video Analytics on IoT Edge - Azure
description: En este artículo se describen los pasos de solución de problemas para Live Video Analytics on IoT Edge.
author: IngridAtMicrosoft
ms.topic: how-to
ms.author: inhenkel
ms.date: 05/24/2020
ms.openlocfilehash: bbd3cb88b017209adff58a646e274caf31ab425f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87486449"
---
# <a name="troubleshoot-live-video-analytics-on-iot-edge"></a>Solución de problemas de Live Video Analytics on IoT Edge

En este artículo se describen los pasos de solución de problemas para Live Video Analytics on Azure IoT Edge.

## <a name="troubleshoot-deployment-issues"></a>Solución de problemas de implementación

### <a name="diagnostics"></a>Diagnóstico

Como parte de la implementación de Live Video Analytics, va a configurar recursos de Azure como IoT Hub y dispositivos IoT Edge. Como primer paso para diagnosticar problemas, siga estas instrucciones para asegurarse de que el dispositivo Edge esté bien configurado:

1. [Ejecute el comando `check`](../../iot-edge/troubleshoot.md#run-the-check-command).
1. [Compruebe la versión de IoT Edge](../../iot-edge/troubleshoot.md#check-your-iot-edge-version).
1. [Compruebe el estado del administrador de seguridad de IoT Edge y sus registros](../../iot-edge/troubleshoot.md#check-the-status-of-the-iot-edge-security-manager-and-its-logs).
1. [Vea los mensajes que se envían a través del centro de IoT Edge](../../iot-edge/troubleshoot.md#view-the-messages-going-through-the-iot-edge-hub).
1. [Reinicie los contenedores](../../iot-edge/troubleshoot.md#restart-containers).
1. [Compruebe las reglas de configuración de los puertos y el firewall](../../iot-edge/troubleshoot.md#check-your-firewall-and-port-configuration-rules).

### <a name="pre-deployment-issues"></a>Problemas anteriores a la implementación

Si la infraestructura perimetral está bien, puede comprobar si existen problemas con el archivo de manifiesto de implementación. Para implementar el módulo Live Video Analytics on IoT Edge en el dispositivo IoT Edge junto con cualquier otro módulo de IoT, puede usar un manifiesto de implementación que contenga el centro de IoT Edge, el agente de IoT Edge y otros módulos con sus propiedades. Si el código JSON no tiene el formato correcto, podría recibir el siguiente error: 

```
az iot edge set-modules --hub-name <iot-hub-name> --device-id lva-sample-device --content <path-to-deployment_manifest.json>
```

No se pudo analizar el código JSON del archivo: "<deployment manifest.json>" para el argumento "content" con la excepción: "Datos adicionales: línea 101, columna 1 (char 5325)"

Si se produce este error, se recomienda comprobar en el archivo JSON si faltan corchetes o existe algún otro problema con la estructura del archivo. Puede usar un cliente como [Notepad++ con el complemento del visor de JSON](https://riptutorial.com/notepadplusplus/example/18201/json-viewer) o una herramienta en línea como el [formateador y validador de JSON](https://jsonformatter.curiousconcept.com/) para validar la estructura de archivos.

### <a name="during-deployment-diagnose-with-media-graph-direct-methods"></a>Durante la implementación: Diagnóstico con los métodos directos de grafo multimedia 

Una vez implementado correctamente el módulo Live Video Analytics on IoT Edge en el dispositivo IoT Edge, puede crear y ejecutar el grafo multimedia invocando [métodos directos](direct-methods.md). Puede usar Azure Portal para ejecutar el diagnóstico del grafo multimedia mediante métodos directos:

1. En Azure Portal, vaya al centro de IoT conectado al dispositivo IoT Edge.

1. Busque **Administración automática de dispositivos** y, después, seleccione **IoT Edge**.  

1. En la lista de dispositivos Edge, seleccione el dispositivo que desea diagnosticar.  
         
    ![Captura de pantalla de Azure Portal que muestra una lista de los dispositivos Edge.](./media/troubleshoot-how-to/lva-sample-device.png)

1. Compruebe si el código de respuesta es *200-OK*. Existen otros códigos de respuesta para el [entorno de ejecución de IoT Edge](../../iot-edge/iot-edge-runtime.md), entre los que se incluyen:
    * 400 - La configuración de implementación tiene un formato incorrecto o no es válida.
    * 417 - El dispositivo no tiene un conjunto de configuración de implementación.
    * 412 - La versión del esquema de la configuración de implementación no es válida.
    * 406 - El dispositivo de IoT Edge está sin conexión o no envía informes de estado.
    * 500 - Error en el entorno en tiempo de ejecución de Azure IoT Edge.

1. Si recibe un código de estado 501, compruebe si el nombre del método directo es adecuado. Si el nombre del método y la carga útil de la solicitud son adecuados, debería obtener resultados junto con el código 200 de ejecución correcta. Si la carga de la solicitud es incorrecta, obtendrá un estado 400 y una carga de respuesta que indica el código de error, así como un mensaje que debería resultar útil para diagnosticar el problema con la llamada de método directo.
    * La comprobación de las propiedades notificadas y deseadas puede ayudarle a entender si las propiedades del módulo se han sincronizado con la implementación. Si no es así, puede reiniciar el dispositivo IoT Edge. 
    * Use la guía de [métodos directos](direct-methods.md) para invocar algunos métodos, especialmente los sencillos, como GraphTopologyList. En la guía también se especifican las cargas de solicitud y respuesta esperadas, así como los códigos de error. Si los métodos directos sencillos funcionan correctamente, puede estar seguro de que el módulo Live Video Analytics on IoT Edge está operativo.
        
       ![Captura de pantalla del panel "Método directo" del módulo de IoT Edge.](./media/troubleshoot-how-to/direct-method.png) 

1. Si las columnas **Especificado en la implementación** y **Notificado por el dispositivo** indican *Sí*, puede invocar métodos directos en el módulo Live Video Analytics on IoT Edge. Seleccione el módulo para ir a una página en la que puede comprobar las propiedades deseadas y notificadas, así como invocar métodos directos. Tenga en cuenta lo siguiente: 

### <a name="post-deployment-diagnose-logs-for-issues-during-the-run"></a>Después de la implementación: Registros de diagnóstico de problemas durante la ejecución 

Los registros de contenedor del módulo de IoT Edge deben contener información de diagnóstico que ayude a depurar los problemas durante el tiempo de ejecución del módulo. Puede [comprobar si hay problemas en los registros del contenedor](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) y realizar un diagnóstico automático de estos. 

Si ha ejecutado todas las comprobaciones anteriores y todavía hay problemas, recopile los registros del dispositivo IoT Edge [con el comando `support bundle`](../../iot-edge/troubleshoot.md#gather-debug-information-with-support-bundle-command) para su posterior análisis por parte del equipo de Azure. Puede [ponerse en contacto](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) con nosotros para obtener soporte técnico y enviar los registros recopilados.

## <a name="common-error-resolutions"></a>Resoluciones de errores comunes

Live Video Analytics se implementa como un módulo de IoT Edge en el dispositivo IoT Edge y funciona en colaboración con el agente de IoT Edge y los módulos del centro de conectividad. Algunos de los errores comunes que se encontrará con la implementación de Live Video Analytics se deben a los problemas con la infraestructura de IoT subyacente. Entre los errores se incluyen:

* [El agente de IoT Edge se detiene después de, aproximadamente, un minuto](../../iot-edge/troubleshoot-common-errors.md#iot-edge-agent-stops-after-about-a-minute).
* [El agente de IoT Edge no puede acceder a la imagen de un módulo (403)](../../iot-edge/troubleshoot-common-errors.md#iot-edge-agent-cant-access-a-modules-image-403).
* [El módulo Agente de IoT Edge notifica "archivo de configuración vacío" y no se inicia ningún módulo en el dispositivo](../../iot-edge/troubleshoot-common-errors.md#edge-agent-module-reports-empty-config-file-and-no-modules-start-on-the-device).
* [No se puede iniciar el centro de IoT Edge](../../iot-edge/troubleshoot-common-errors.md#iot-edge-hub-fails-to-start).
* [Error de nombre de host no válido del demonio de seguridad de IoT Edge](../../iot-edge/troubleshoot-common-errors.md#iot-edge-security-daemon-fails-with-an-invalid-hostname).
* [Live Video Analytics o cualquier otro módulo de IoT Edge personalizado no puede enviar un mensaje al centro de IoT Edge con el error 404](../../iot-edge/troubleshoot-common-errors.md#iot-edge-module-fails-to-send-a-message-to-edgehub-with-404-error).
* [El módulo de IoT Edge se implementa correctamente y, a continuación, desaparece del dispositivo](../../iot-edge/troubleshoot-common-errors.md#iot-edge-module-deploys-successfully-then-disappears-from-device).

### <a name="edge-setup-script-issues"></a>Problemas de scripts de configuración de Edge

Como parte de la documentación, hemos proporcionado un [script de configuración](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) para implementar los recursos perimetrales y en la nube para empezar a usar Live Video Analytics on IoT Edge. En esta sección se presentan algunos errores de scripts que pueden surgir, junto con soluciones para depurarlos.

Problema: El script se ejecuta y crea parcialmente algunos recursos, pero devuelve un error con el siguiente mensaje:

```
registering device...

Unable to load extension 'eventgrid: unrecognized kwargs: ['min_profile']'. Use --debug for more information.
The command failed with an unexpected error. Here is the traceback:

No module named 'azure.mgmt.iothub.iot_hub_client'
Traceback (most recent call last):
File "/opt/az/lib/python3.6/site-packages/knack/cli.py", line 215, in invoke
  cmd_result = self.invocation.execute(args)
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 631, in execute
  raise ex
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 695, in _run_jobs_serially
  results.append(self._run_job(expanded_arg, cmd_copy))
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 688, in _run_job
  six.reraise(*sys.exc_info())
File "/opt/az/lib/python3.6/site-packages/six.py", line 693, in reraise
  raise value
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 665, in _run_job
  result = cmd_copy(params)
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 324, in __call__
  return self.handler(*args, **kwargs)
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/__init__.py", line 574, in default_command_handler
  return op(**command_args)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/operations/hub.py", line 75, in iot_device_list
  result = iot_query(cmd, query, hub_name, top, resource_group_name, login=login)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/operations/hub.py", line 45, in iot_query
  target = get_iot_hub_connection_string(cmd, hub_name, resource_group_name, login=login)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/common/_azure.py", line 112, in get_iot_hub_connection_string
  client = iot_hub_service_factory(cmd.cli_ctx)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/_factory.py", line 28, in iot_hub_service_factory
  from azure.mgmt.iothub.iot_hub_client import IotHubClient
ModuleNotFoundError: No module named 'azure.mgmt.iothub.iot_hub_client'
```
    
Para corregir este problema:

1. Ejecute el siguiente comando:

    ```
    az --version
    ```
1. Asegúrese de que tiene instaladas las siguientes extensiones. En el momento de la publicación de este artículo, las extensiones y sus versiones son:

    | Extensión | Versión |
    |---|---|
    |azure-cli   |      2.5.1*|
    |command-modules-nspkg         |   2.0.3|
    |core  |    2.5.1*|
    |nspkg    | 3.0.4|
    |telemetry| 1.0.4|
    |storage-preview          |     0.2.10|
    |azure-cli-iot-ext          |    0.8.9|
    |eventgrid| 0.4.9|
    |azure-iot                       | 0.9.2|
1. Si tiene una extensión instalada cuya versión es anterior al número de versión que aparece aquí, actualice la extensión con el siguiente comando:

    ```
    az extension update --name <Extension name>
    ```

    Por ejemplo, puede ejecutar `az extension update --name azure-iot`.

### <a name="sample-app-issues"></a>Problemas con la aplicación de muestra

Como parte de nuestra versión, hemos proporcionado cierto código de ejemplo en .NET para impulsar el arranque de la comunidad de desarrolladores. En esta sección se presentan algunos errores de scripts que pueden surgir cuando se ejecuta el código de ejemplo, junto con soluciones para depurarlos.

Problema: Program.cs devuelve el siguiente error en la invocación del método directo:

```
Unhandled exception. Microsoft.Azure.Devices.Common.Exceptions.UnauthorizedException: {"Message":"{\"errorCode\":401002,\"trackingId\":\"b1da85801b2e4faf951a2291a2c467c3-G:32-TimeStamp:04/06/2020 17:15:11\",\"message\":\"Unauthorized\",\"timestampUtc\":\"2020-04-06T17:15:11.6990676Z\"}","ExceptionMessage":""}
    
        at Microsoft.Azure.Devices.HttpClientHelper.ExecuteAsync(HttpClient httpClient, HttpMethod httpMethod, Uri requestUri, Func`3 modifyRequestMessageAsync, Func`2 isMappedToException, Func`3 processResponseMessageAsync, IDictionary`2 errorMappingOverrides, CancellationToken cancellationToken)
    
        at Microsoft.Azure.Devices.HttpClientHelper.ExecuteAsync(HttpMethod httpMethod, Uri requestUri, Func`3 modifyRequestMessageAsync, Func`3 processResponseMessageAsync, IDictionary`2 errorMappingOverrides, CancellationToken cancellationToken)
        
        at Microsoft.Azure.Devices.HttpClientHelper.PostAsync[T,T2](Uri requestUri, T entity, TimeSpan operationTimeout, IDictionary`2 errorMappingOverrides, IDictionary`2 customHeaders, CancellationToken cancellationToken)…
```

1. Asegúrese de que dispone de [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) instalado en el entorno de Visual Studio Code y que ha configurado la conexión con el centro de IoT. Para ello, seleccione Ctrl + Mayús + P y, a continuación, elija **Select IoT Hub method** (Seleccionar método de IoT Hub).

1. Compruebe si puede invocar un método directo en el módulo de IoT Edge a través de Visual Studio Code. Por ejemplo, llame a GraphTopologyList con la siguiente carga {&nbsp;"@apiVersion": "1.0"}. Debe recibir la respuesta siguiente: 

    ```
    {
      "status": 200,
      "payload": {
        "values": [
          {…
    …}
          ]
        }
    }
    ```

    ![Captura de pantalla de la respuesta en Visual Studio Code.](./media/troubleshoot-how-to/visual-studio-code1.png)
1. Si se produce un error en la solución anterior, intente lo siguiente:

    a. Vaya al símbolo del sistema en el dispositivo IoT Edge y ejecute el siguiente comando:
    
      ```
      sudo systemctl restart iotedge
      ```

      Este comando reinicia el dispositivo IoT Edge y todos los módulos. Espere unos minutos y, a continuación, antes de intentar volver a usar el método directo, confirme que los módulos se están ejecutando mediante el comando siguiente:

      ```
      sudo iotedge list
      ```

    b. Si también se produce un error en el enfoque anterior, intente reiniciar el equipo o la máquina virtual.

    c. Si se produce un error en todos los enfoques, ejecute el siguiente comando para obtener un archivo ZIP con todos los [registros pertinentes](../../iot-edge/troubleshoot.md#gather-debug-information-with-support-bundle-command) y adjúntelo a una [incidencia de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

    ```
    sudo iotedge support-bundle --since 2h
    ```

1. Si recibe un código *400* de respuesta de error, asegúrese de que la carga de invocación del método tenga un formato correcto según la guía de [métodos directos](direct-methods.md).
1. Si obtiene el código de estado *200*, esto significa que el centro de conectividad funciona bien y que la implementación del módulo es correcta y responde. 

1. Compruebe si la configuración de la aplicación es correcta. La configuración de la aplicación consta de los siguientes campos en el archivo *appsettings.json*. Asegúrese de que los valores de deviceId y moduleId sean adecuados. Una manera sencilla de comprobarlo es ir a la sección de la extensión de Azure IoT Hub en Visual Studio Code. Los valores del archivo *appsettings.json* y la sección de IoT Hub deben coincidir.
    
    ```
    {
        "IoThubConnectionString" : 
        "deviceId" : 
        "moduleId" : 
    }
    ```

1. Asegúrese de que, en el archivo *appsettings.json*, ha proporcionado la cadena de conexión de IoT Hub y *no* la cadena de conexión del dispositivo IoT Hub, ya que los [formatos de las cadenas de conexión son diferentes](https://devblogs.microsoft.com/iotdev/understand-different-connection-strings-in-azure-iot-hub/).

### <a name="live-video-analytics-working-with-external-modules"></a>Live Video Analytics trabajando con módulos externos

Live Video Analytics, mediante el procesador de extensiones HTTP, puede ampliar el grafo multimedia para enviar y recibir datos de otros módulos de IoT Edge por HTTP mediante REST. Como [ejemplo específico](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/httpExtension), el grafo multimedia puede enviar fotogramas de vídeo como imágenes a un módulo de inferencia externo, como Yolo v3, y recibir resultados de análisis basados en JSON. En esta topología, el destino final de los eventos es, principalmente, el centro de IoT. En situaciones en las que no vea los eventos de inferencia en el centro de conectividad, compruebe lo siguiente:

* Compruebe si el centro de conectividad en el que publica el grafo multimedia es el mismo que está examinando. Cuando crea varias implementaciones, puede que haya muchos centros de conectividad y podría estar comprobando el centro incorrecto para los eventos.
* En Visual Studio Code, compruebe si el módulo externo está implementado y en ejecución. En la imagen de ejemplo que se muestra aquí, rtspsim y cv son módulos de IoT Edge que se ejecutan fuera del módulo lvaEdge.

    ![Captura de pantalla que muestra el estado de ejecución de los módulos en Azure IoT Hub.](./media/troubleshoot-how-to/iot-hub.png)

* Compruebe que está enviando eventos al punto de conexión de URL correcto. El contenedor de inteligencia artificial externo expone una dirección URL y un puerto a través del cual recibe y devuelve los datos de las solicitudes POST. Esta dirección URL se especifica como una propiedad `endpoint: url` para el procesador de extensión HTTP. Como se aprecia en la [dirección URL de la topología](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/httpExtension/topology.json), el punto de conexión se establece en el parámetro URL de inferencia. Asegúrese de que el valor predeterminado del parámetro o el valor pasado son correctos. Puede realizar una prueba para ver si funciona mediante una dirección URL del cliente (cURL).  

    Por ejemplo, este es un contenedor de Yolo v3 que se ejecuta en una máquina local con esta dirección IP: 172.17.0.3. Use docker inspect para encontrar la dirección IP.

    ```
    curl -X POST http://172.17.0.3/score -H "Content-Type: image/jpeg" --data-binary @<fullpath to jpg>
    ```

    Resultado que se devuelve:

    ```
    {"inferences": [{"type": "entity", "entity": {"tag": {"value": "car", "confidence": 0.8668569922447205}, "box": {"l": 0.3853073438008626, "t": 0.6063712999658677, "w": 0.04174524943033854, "h": 0.02989496027381675}}}]}
    ```

* Si está ejecutando una o varias instancias de un grafo que usa el procesador de extensiones HTTP, debe tener un filtro de velocidad de fotogramas antes de cada procesador de extensión HTTP para administrar los fotogramas por segundo (FPS) de la fuente de vídeo. 

   En determinadas situaciones en las que la CPU o la memoria de la máquina perimetral tienen un uso muy elevado, puede perder ciertos eventos de inferencia. Para solucionar este problema, establezca un valor bajo para la propiedad maximumFps en el filtro de velocidad de fotogramas. Puede establecerlo en 0,5 ("maximumFps": 0,5) en cada instancia del grafo y volver a ejecutar la instancia para obtener los eventos de inferencia del centro de conectividad.

   Como alternativa, puede obtener una máquina perimetral más eficaz con una mayor CPU y memoria.
    
### <a name="multiple-direct-methods-in-parallel--timeout-failure"></a>Varios métodos directos en paralelo: error de tiempo de espera 

Live Video Analytics on IoT Edge proporciona un modelo de programación basado en métodos directos que permite configurar varias topologías y varias instancias de grafos. Como parte de la configuración de la topología y el grafo, puede invocar varias llamadas de métodos directos en el módulo de IoT Edge. Si invoca estas llamadas de métodos múltiples en paralelo, especialmente las que inician y detienen los grafos, puede experimentar un error de tiempo de espera como el siguiente: 

Assembly Initialization method Microsoft.Media.LiveVideoAnalytics.Test.Feature.Edge.AssemblyInitializer.InitializeAssemblyAsync threw exception. Microsoft.Azure.Devices.Common.Exceptions.IotHubException: Microsoft.Azure.Devices.Common.Exceptions.IotHubException:<br/> `{"Message":"{\"errorCode\":504101,\"trackingId\":\"55b1d7845498428593c2738d94442607-G:32-TimeStamp:05/15/2020 20:43:10-G:10-TimeStamp:05/15/2020 20:43:10\",\"message\":\"Timed out waiting for the response from device.\",\"info\":{},\"timestampUtc\":\"2020-05-15T20:43:10.3899553Z\"}","ExceptionMessage":""}. Aborting test execution. `

Se recomienda *no* llamar a métodos directos en paralelo. Llámelos de manera secuencial (es decir, realice una llamada de método directo solo después de que finalice la anterior).

### <a name="collect-logs-for-submitting-a-support-ticket"></a>Recopilación de registros para enviar una incidencia de soporte técnico

Si los pasos de solución de problemas autoguiados no resuelven el problema, debe ir a Azure Portal y [abrir una incidencia de soporte técnico](../../azure-portal/supportability/how-to-create-azure-support-request.md).

> [!WARNING]
> Los registros pueden contener información de identificación personal (PII) como su dirección IP. Se eliminarán todas las copias locales de los registros en cuanto terminemos de examinarlos y cerremos la incidencia de soporte técnico.  

Para recopilar los registros pertinentes que se deben agregar a la incidencia, siga las instrucciones de las secciones siguientes. Puede cargar los archivos de registro en el panel **Detalles** de la solicitud de soporte técnico.

### <a name="use-the-support-bundle-command"></a>Uso del comando support-bundle

Cuando tenga que recopilar registros de un dispositivo IoT Edge, la manera más sencilla es usar el comando `support-bundle`. Este comando recopila:

- Registros del módulo
- Administrador de seguridad de IoT Edge y registros del motor de contenedor
- Salida JSON iotedge check
- Información de depuración útil

1. Ejecute el comando `support-bundle` con la marca *--since* para especificar cuánto tiempo desea que abarquen los registros. Por ejemplo, 2 h obtendrá los registros de las últimas dos horas. Puede cambiar el valor de esta marca para incluir los registros de un período diferente.

    ```
    sudo iotedge support-bundle --since 2h
    ```

   Este comando crea un archivo denominado *support_bundle.zip* en el directorio donde se ejecutó el comando. 
   
1. Adjunte el archivo *support_bundle.zip* a la incidencia de soporte técnico.

### <a name="live-video-analytics-debug-logs"></a>Registros de depuración de Live Video Analytics

Para configurar el módulo de Live Video Analytics on IoT Edge para que genere registros de depuración, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al centro de IoT.
1. En el panel izquierdo, seleccione **IoT Edge**.
1. Seleccione el identificador del dispositivo de destino en la lista de dispositivos.
1. En la parte superior del panel, seleccione **Establecer módulos**.

   ![Captura de pantalla del botón "Establecer módulos" en Azure Portal.](media/troubleshoot-how-to/set-modules.png)

1. En la sección **módulos de IoT Edge**, busque y seleccione **lvaEdge**.
1. Seleccione **Opciones de creación del contenedor**.
1. En la sección **Enlaces**, agregue el comando siguiente:

    `/var/local/mediaservices/logs:/var/lib/azuremediaservices/logs`

    > [!NOTE] 
    > Este comando enlaza las carpetas de registros entre el dispositivo Edge y el contenedor. Si desea recopilar los registros en una ubicación diferente, use el comando siguiente, reemplazando **$LOG_LOCATION_ON_EDGE_DEVICE** por la ubicación que desea usar: `/var/$LOG_LOCATION_ON_EDGE_DEVICE:/var/lib/azuremediaservices/logs`

1. Seleccione **Actualizar**.
1. Seleccione **Revisar + crear**. Se publica un mensaje de validación correcto en un banner verde.
1. Seleccione **Crear**.
1. Actualice **Identidad de módulo gemela** para que apunte al parámetro DebugLogsDirectory a fin de seleccionar el directorio en el que se recopilarán los registros:

    a. En la tabla **Módulos**, seleccione **IvaEdge**.  
    b. En la parte superior del panel, seleccione **Identidad de módulo gemela**. Se abre un panel editable.  
    c. Agregue el siguiente par clave-valor en la **clave que desee**:  
    `"DebugLogsDirectory": "/var/lib/azuremediaservices/logs"`

    > [!NOTE] 
    > Este comando enlaza las carpetas de registros entre el dispositivo Edge y el contenedor. Si desea recopilar los registros en una ubicación diferente, use el comando siguiente, reemplazando **$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE** por la ubicación que desea usar:  
    > `"DebugLogsDirectory": "/var/$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE"`  

    d. Seleccione **Guardar**.

1. Reproduzca el problema.
1. Conéctese a la máquina virtual desde la página **IoT Hub** del portal.
1. Comprima todos los archivos de la carpeta *debugLogs*.

   > [!NOTE]
   > Estos archivos de registro no están diseñados para el autodiagnóstico, sino para que el equipo de ingeniería de Azure analice sus problemas.

   a. En el siguiente comando, asegúrese de reemplazar **$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE** por la ubicación de los registros de depuración en el dispositivo Edge que configuró anteriormente.  

   ```
   sudo apt install zip unzip  
   zip -r debugLogs.zip $DEBUG_LOG_LOCATION_ON_EDGE_DEVICE 
   ```

   b. Adjunte el archivo *debugLogs.zip* a la incidencia de soporte técnico.

1. La recopilación de registros se puede detener estableciendo ese valor en **Identidad de módulo gemela** en *NULL*. Vuelva a la página **Identidad de módulo gemela** y actualice los siguientes parámetros como:

    `"DebugLogsDirectory": ""`

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: Grabación de vídeo basada en eventos en la nube y reproducción desde la nube](event-based-video-recording-tutorial.md)
