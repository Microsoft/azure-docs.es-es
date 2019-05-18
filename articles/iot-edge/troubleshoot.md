---
title: 'Solución de problemas: Azure IoT Edge | Microsoft Docs'
description: Use este artículo para obtener información sobre las aptitudes de diagnóstico estándar para Azure IoT Edge, como la recuperación de los registros y el estado del componente, así como la resolución de problemas comunes.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: 00147002317f15345f01c88e81973837d16e6669
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797621"
---
# <a name="common-issues-and-resolutions-for-azure-iot-edge"></a>Problemas habituales y soluciones para Azure IoT Edge

Si experimenta problemas al ejecutar Azure IoT Edge en el entorno, use este artículo como guía para solucionarlos.

## <a name="run-the-iotedge-check-command"></a>Ejecute el iotedge 'comprobar' (comando)

Debe ser el primer paso al solucionar problemas de IoT Edge usar el `check` comando, que lleva a cabo una serie de pruebas de configuración y la conectividad para problemas comunes. El `check` comando está disponible en [versión 1.0.7](https://github.com/Azure/azure-iotedge/releases/tag/1.0.7) y versiones posteriores.

Puede ejecutar el `check` comando como se indica a continuación, o bien incluir el `--help` marca para ver una lista completa de opciones:

* En Linux:

  ```bash
  sudo iotedge check
  ```

* En Windows:

  ```powershell
  iotedge check
  ```

Los tipos de comprobaciones de ejecutar la herramienta se pueden clasificar como:

* Comprobaciones de configuración: Examina los detalles que podrían impedir que se conecten a la nube, incluidos los problemas con dispositivos perimetrales *config.yaml* y el motor de contenedor.
* Comprobaciones de conexión: Comprueba el tiempo de ejecución de IoT Edge puede tener acceso a los puertos en el dispositivo de host y todos los componentes de IoT Edge pueden conectarse a IoT Hub.
* Comprobaciones de disponibilidad de producción: Busca las mejores prácticas recomendadas de producción, como el estado de certificados de entidad de certificación (CA) de dispositivo y la configuración del archivo de registro de módulo.

Para obtener una lista completa de las comprobaciones de diagnóstico, consulte [integradas de solución de problemas de funcionalidad](https://github.com/Azure/iotedge/blob/master/doc/troubleshoot-checks.md).

## <a name="standard-diagnostic-steps"></a>Pasos de diagnóstico estándar

Si encuentra algún problema, puede aprender más sobre el estado del dispositivo de IoT Edge revisando los registros del contenedor y los mensajes que pasan a y desde el dispositivo. Use los comandos y las herramientas de esta sección para recopilar información.

### <a name="check-the-status-of-the-iot-edge-security-manager-and-its-logs"></a>Comprobar el estado de sus registros y el Administrador de seguridad de IoT Edge

En Linux:
- Para ver el estado del administrador de seguridad de IoT Edge:

   ```bash
   sudo systemctl status iotedge
   ```

- Para ver los registros del administrador de seguridad de IoT Edge:

    ```bash
    sudo journalctl -u iotedge -f
    ```

- Para ver registros más detallados del administrador de seguridad de IoT Edge:

   - Edite la configuración del demonio de iotedge:

      ```bash
      sudo systemctl edit iotedge.service
      ```
   
   - Actualice las líneas siguientes:
    
      ```
      [Service]
      Environment=IOTEDGE_LOG=edgelet=debug
      ```
    
   - Reinicie el demonio de seguridad de IoT Edge:
    
      ```bash
      sudo systemctl cat iotedge.service
      sudo systemctl daemon-reload
      sudo systemctl restart iotedge
      ```

En Windows:
- Para ver el estado del administrador de seguridad de IoT Edge:

   ```powershell
   Get-Service iotedge
   ```

- Para ver los registros del administrador de seguridad de IoT Edge:

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
   ```

### <a name="if-the-iot-edge-security-manager-is-not-running-verify-your-yaml-configuration-file"></a>Comprobación del archivo de configuración de YAML si no se está ejecutando el administrador de seguridad de IoT Edge

> [!WARNING]
> Archivos YAML no pueden contener tabuladores como sangría. Utilice en su lugar dos espacios.

En Linux:

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

En Windows:

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

### <a name="check-container-logs-for-issues"></a>Comprobación de los registros del contenedor para detectar problemas

Cuando el demonio de seguridad de IoT Edge se esté ejecutando, examine los registros de los contenedores para detectar problemas. Comience con los contenedores implementados y, después, examine los contenedores que componen el entorno de ejecución de IoT Edge: edgeAgent y edgeHub. Los registros del agente de IoT Edge normalmente proporcionan información sobre el ciclo de vida de cada contenedor. Los registros del centro de IoT Edge proporcionan información sobre la mensajería y enrutamiento. 

   ```cmd
   iotedge logs <container name>
   ```

### <a name="view-the-messages-going-through-the-iot-edge-hub"></a>Ver los mensajes que pasen a través del centro de IoT Edge

Puede ver los mensajes que pasen a través del centro de IoT Edge y recopilar información de los registros detallados de los contenedores en tiempo de ejecución. Para activar los registros detallados en estos contenedores, establezca `RuntimeLogLevel` en el archivo de configuración yaml. Para abrir el archivo:

En Linux:

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

En Windows:

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

De forma predeterminada, el elemento `agent` tendrá un aspecto similar al del ejemplo siguiente:

   ```yaml
   agent:
     name: edgeAgent
     type: docker
     env: {}
     config:
       image: mcr.microsoft.com/azureiotedge-agent:1.0
       auth: {}
   ```

Reemplace `env: {}` por:

   ```yaml
   env:
     RuntimeLogLevel: debug
   ```

   > [!WARNING]
   > Los archivos de YAML no pueden contener tabulaciones como sangría. Utilice en su lugar dos espacios.

Guarde el archivo y reinicie el administrador de seguridad de IoT Edge.

También puede comprobar los mensajes que se envían entre los dispositivos de IoT Hub e IoT Edge. Consulte estos mensajes mediante la extensión [Azure IoT Hub Toolkit](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) (anteriormente extensión Azure IoT Toolkit) para Visual Studio Code. Para más información, consulte [Handy tool when you develop with Azure IoT](https://blogs.msdn.microsoft.com/iotdev/2017/09/01/handy-tool-when-you-develop-with-azure-iot/) (Herramienta práctica para desarrollar con Azure IoT).

### <a name="restart-containers"></a>Reinicio de los contenedores
Después de investigar los registros y mensajes para obtener información, puede intentar reiniciar los contenedores:

```
iotedge restart <container name>
```

Reinicie los contenedores del entorno de ejecución de IoT Edge:

```
iotedge restart edgeAgent && iotedge restart edgeHub
```

### <a name="restart-the-iot-edge-security-manager"></a>Reinicio del administrador de seguridad de IoT Edge

Si el problema persiste, puede probar a reiniciar el administrador de seguridad de IoT Edge.

En Linux:

   ```cmd
   sudo systemctl restart iotedge
   ```

En Windows:

   ```powershell
   Stop-Service iotedge -NoWait
   sleep 5
   Start-Service iotedge
   ```

## <a name="iot-edge-agent-stops-after-about-a-minute"></a>Agente de IoT Edge se detiene después de aproximadamente un minuto

El módulo de edgeAgent se inicia y se ejecuta correctamente durante aproximadamente un minuto, a continuación, se detiene. Los registros indican que el agente de IoT Edge intenta conectarse a IoT Hub a través de AMQP y, a continuación, intenta conectarse mediante AMQP sobre WebSocket. Cuando se produce un error, se cierra el agente de IoT Edge. 

Registros de edgeAgent de ejemplo:

```output
2017-11-28 18:46:19 [INF] - Starting module management agent. 
2017-11-28 18:46:19 [INF] - Version - 1.0.7516610 (03c94f85d0833a861a43c669842f0817924911d5) 
2017-11-28 18:46:19 [INF] - Edge agent attempting to connect to IoT Hub via AMQP... 
2017-11-28 18:46:49 [INF] - Edge agent attempting to connect to IoT Hub via AMQP over WebSocket... 
```

### <a name="root-cause"></a>Causa principal
Una configuración de red en la red de host está impidiendo que el agente de IoT Edge alcance la red. El agente intentará conectarse en primer lugar a través de AMQP (puerto 5671). Si se produce un error en la conexión, lo intentará mediante los protocolos WebSocket (puerto 443).

El entorno de ejecución de IoT Edge configura una red para cada uno de los módulos en los que se comunica. En Linux, esta red es una red de puente. En Windows, se usa NAT. Este problema es más frecuente en los dispositivos de Windows que usan contenedores de Windows que usan la red NAT. 

### <a name="resolution"></a>Resolución
Asegúrese de que hay una ruta a Internet para las direcciones IP asignadas a esta red de puente o NAT. A veces, una configuración de VPN en el host invalida la red de IoT Edge. 

## <a name="iot-edge-hub-fails-to-start"></a>No se puede iniciar el centro de IoT Edge

Se produce un error de módulo de edgeHub al inicio e imprime el siguiente mensaje en los registros: 

```output
One or more errors occurred. 
(Docker API responded with status code=InternalServerError, response=
{\"message\":\"driver failed programming external connectivity on endpoint edgeHub (6a82e5e994bab5187939049684fb64efe07606d2bb8a4cc5655b2a9bad5f8c80): 
Error starting userland proxy: Bind for 0.0.0.0:443 failed: port is already allocated\"}\n) 
```

### <a name="root-cause"></a>Causa principal
Algún otro proceso del equipo host se ha enlazado con el puerto 443. El centro de IoT Edge asigna los puertos 5671 y 443 para su uso en escenarios de puerta de enlace. Se producirá un error en la asignación de puertos si hay algún otro proceso enlazado con este puerto. 

### <a name="resolution"></a>Resolución
Busque y detenga el proceso que está utilizando el puerto 443. Este proceso suele ser un servidor web.

## <a name="iot-edge-agent-cant-access-a-modules-image-403"></a>Agente de IoT Edge no puede tener acceso a una imagen de módulo (403)
No se puede ejecutar un contenedor, y los registros de edgeAgent muestran un error 403. 

### <a name="root-cause"></a>Causa principal
El agente de Iot Edge no tiene permisos para acceder a la imagen de un módulo. 

### <a name="resolution"></a>Resolución
Asegúrese de que las credenciales del registro se hayan especificado correctamente en el manifiesto de implementación.

## <a name="iot-edge-security-daemon-fails-with-an-invalid-hostname"></a>Error de nombre de host no válido del demonio de seguridad de IoT Edge

El comando `sudo journalctl -u iotedge` produce un error e imprime el mensaje siguiente: 

```output
Error parsing user input data: invalid hostname. Hostname cannot be empty or greater than 64 characters
```

### <a name="root-cause"></a>Causa principal
El tiempo de ejecución de IoT Edge solo puede admitir los nombres de host que tienen menos de 64 caracteres. Las máquinas físicas no suelen tener nombres de host largos, pero el problema es más común en una máquina virtual. Los nombres de host generados automáticamente para las máquinas virtuales Windows hospedadas en Azure, en particular, suelen ser largos. 

### <a name="resolution"></a>Resolución
Cuando vea este error, puede resolverlo configurando el nombre DNS de la máquina virtual y, a continuación, estableciendo el nombre DNS como nombre de host en el comando de configuración.

1. En Azure Portal, navegue a la hoja de introducción de la máquina virtual. 
2. Seleccione **configurar** en el nombre DNS. Si la máquina virtual ya tiene configurado un nombre DNS, no es necesario configurar uno nuevo. 

   ![Configuración del nombre DNS de la máquina virtual](./media/troubleshoot/configure-dns.png)

3. Proporcione un valor para **Etiqueta de nombre DNS** y seleccione **Guardar**.
4. Copie el nuevo nombre DNS, que debe tener el formato **\<etiquetanombreDNS\>.\< ubicaciónvm\>.cloudapp.azure.com**.
5. Dentro de la máquina virtual, use el comando siguiente para configurar el tiempo de ejecución de IoT Edge con el nombre DNS:

   - En Linux:

      ```bash
      sudo nano /etc/iotedge/config.yaml
      ```

   - En Windows:

      ```cmd
      notepad C:\ProgramData\iotedge\config.yaml
      ```

## <a name="stability-issues-on-resource-constrained-devices"></a>Problemas de estabilidad en dispositivos con recursos limitados 
Es posible que encuentre problemas de estabilidad en dispositivos restringidos como Raspberry Pi, especialmente cuando se utiliza como puerta de enlace. Los síntomas incluyen excepciones de memoria insuficiente en el módulo del concentrador perimetral, los dispositivos de nivel inferior no pueden conectarse o el dispositivo deja de enviar mensajes de telemetría después de unas horas.

### <a name="root-cause"></a>Causa principal
El centro de IoT Edge, que forma parte del tiempo de ejecución de IoT Edge, está optimizado para rendimiento predeterminada y se intenta asignar grandes fragmentos de memoria. Esta optimización no es ideal para dispositivos con perímetro limitado y puede causar problemas de estabilidad.

### <a name="resolution"></a>Resolución
Para el centro de IoT Edge, establecer una variable de entorno **OptimizeForPerformance** a **false**. Existen dos formas de hacerlo:

En la interfaz de usuario: 

En el portal, vaya a **detalles del dispositivo** > **establecer módulos** > **configurar opciones de tiempo de ejecución de Edge avanzadas**. Crear una variable de entorno para el módulo de centro de Edge llamado *OptimizeForPerformance* que se establece en *false*.

![OptimizeForPerformance establecido en false](./media/troubleshoot/optimizeforperformance-false.png)

**O bien**

En el manifiesto de implementación:

```json
  "edgeHub": {
    "type": "docker",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
      "createOptions": <snipped>
    },
    "env": {
      "OptimizeForPerformance": {
          "value": "false"
      }
    },
```
## <a name="cant-get-the-iot-edge-daemon-logs-on-windows"></a>No se obtienen los registros del demonio de IoT Edge en Windows
Si aparece EventLogException al usar`Get-WinEvent` en Windows, compruebe las entradas del registro.

### <a name="root-cause"></a>Causa principal
El comando de PowerShell `Get-WinEvent` se basa en que esté presente una entrada del Registro para encontrar los registros por un elemento `ProviderName` concreto.

### <a name="resolution"></a>Resolución
Establezca una entrada del registro para el demonio de IoT Edge. Cree un archivo **iotedge.reg** con el siguiente contenido e impórtelo en el Registro de Windows haciendo doble clic en él o mediante el comando `reg import iotedge.reg`:

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\iotedged]
"CustomSource"=dword:00000001
"EventMessageFile"="C:\\ProgramData\\iotedge\\iotedged.exe"
"TypesSupported"=dword:00000007
```

## <a name="iot-edge-module-fails-to-send-a-message-to-the-edgehub-with-404-error"></a>El módulo de IoT Edge no puede enviar un mensaje a edgeHub y se produce el error 404

El módulo de IoT Edge personalizado no puede enviar un mensaje a edgeHub y se produce el error 404 `Module not found`. El demonio de IoT Edge imprime el mensaje siguiente en los registros: 

```output
Error: Time:Thu Jun  4 19:44:58 2018 File:/usr/sdk/src/c/provisioning_client/adapters/hsm_client_http_edge.c Func:on_edge_hsm_http_recv Line:364 executing HTTP request fails, status=404, response_buffer={"message":"Module not found"}u, 04 ) 
```

### <a name="root-cause"></a>Causa principal
El demonio de IoT Edge aplica la identificación de proceso para todos los módulos que se conectan a edgeHub por motivos de seguridad. Comprueba que todos los mensajes enviados por un módulo proceden del identificador de proceso principal del módulo. Si un módulo envía un mensaje desde un identificador de proceso diferente del que se estableció inicialmente, se rechazará el mensaje y se generará el mensaje de error 404.

### <a name="resolution"></a>Resolución
Asegúrese de que el módulo de IoT Edge personalizado siempre usa el mismo identificador de proceso para enviar los mensajes a edgeHub. Por ejemplo, no olvide `ENTRYPOINT` en lugar de `CMD` comando en el archivo de Docker, ya que `CMD` dará lugar a un proceso de identificador para el módulo y otro Id. de proceso para ejecutar el programa principal, mientras que el comando de bash `ENTRYPOINT` dará lugar a un identificador de proceso único.


## <a name="firewall-and-port-configuration-rules-for-iot-edge-deployment"></a>Reglas de configuración de puertos y firewall para la implementación de IoT Edge
Azure IoT Edge permite la comunicación desde un servidor local a Azure en la nube mediante protocolos de IoT Hub compatibles, consulte [elegir un protocolo de comunicación](../iot-hub/iot-hub-devguide-protocols.md). Para mejorar la seguridad, los canales de comunicación entre Azure IoT Edge y Azure IoT Hub siempre están configurados para que sea la salida. Esta configuración se basa en el [patrón de comunicación asistida de servicios](https://blogs.msdn.microsoft.com/clemensv/2014/02/09/service-assisted-communication-for-connected-devices/), que minimiza la superficie de ataque que una entidad malintencionada puede explorar. La comunicación entrante solo es necesaria para escenarios específicos donde Azure IoT Hub necesita insertar mensajes en el dispositivo de Azure IoT Edge. Los mensajes de nube a dispositivo están protegidos mediante canales TLS seguros y pueden protegerse aún más mediante los certificados X.509 y los módulos de dispositivos TPM. El Administrador de seguridad de Azure IoT Edge rige cómo se puede establecer esta comunicación; consulte [Administrador de seguridad de IoT Edge](../iot-edge/iot-edge-security-manager.md).

Aunque IoT Edge permite una mejor configuración para proteger el entorno de ejecución de Azure IoT Edge y los módulos implementados, todavía depende de la configuración de la máquina y la red subyacentes. Por lo tanto, es imprescindible asegurarse de que la red adecuada y las reglas de firewall se configuran para que edge segura para la comunicación de nube. En la tabla siguiente puede usarse como guía cuando las reglas de firewall de la configuración de los servidores subyacentes donde se hospeda en tiempo de ejecución de Azure IoT Edge:

|Protocol|Port|Entrante|Saliente|Guía|
|--|--|--|--|--|
|MQTT|8883|BLOQUEADO (valor predeterminado)|BLOQUEADO (valor predeterminado)|<ul> <li>Configure el valor de Saliente (de salida) para que sea Abierto cuando se usa MQTT como protocolo de comunicación.<li>1883 para MQTT no es compatible con IoT Edge. <li>Se deben bloquear las conexiones entrantes (de entrada).</ul>|
|AMQP|5671|BLOQUEADO (valor predeterminado)|ABIERTO (valor predeterminado)|<ul> <li>Protocolo de comunicación predeterminado de IoT Edge. <li> Debe configurarse para ser Abierto si Azure IoT Edge no está configurado para otros protocolos compatibles o AMQP es el protocolo de comunicación que se desea.<li>5672 para AMQP no es compatible con IoT Edge.<li>Bloquee este puerto cuando Azure IoT Edge use un protocolo compatible de IoT Hub diferente.<li>Se deben bloquear las conexiones entrantes (de entrada).</ul></ul>|
|HTTPS|443|BLOQUEADO (valor predeterminado)|ABIERTO (valor predeterminado)|<ul> <li>Configure el valor de las conexiones salientes (de salida) para que sea abierto en 443 para el aprovisionamiento de IoT Edge. Esta configuración es necesaria cuando se usen scripts manuales o Azure IoT Device Provisioning Service (DPS). <li>La conexión entrante (de entrada) solo debe ser Abierta en escenarios concretos: <ul> <li>  Si tiene una puerta de enlace transparente con dispositivos de hoja que puedan enviar solicitudes de método. En este caso, no es necesario que el puerto 443 esté abierto a las redes externas para conectarse a IOT Hub ni proporcionar servicios de IoTHub mediante Azure IoT Edge. Por lo tanto, la regla de entrada podría limitarse solo a los puertos entrantes (de entrada) abiertos desde la red interna. <li> En el caso de escenarios de cliente a dispositivo (C2D).</ul><li>80 para HTTP no es compatible con IoT Edge.<li>Si no se pueden configurar los protocolos que no sean HTTP (por ejemplo, AMQP o MQTT) en la empresa; los mensajes pueden enviarse a través de WebSockets. En ese caso, el puerto 443 se utilizará para la comunicación de WebSocket.</ul>|

## <a name="edge-agent-module-continually-reports-empty-config-file-and-no-modules-start-on-the-device"></a>Módulo de agente de Edge continuamente informes "archivo de configuración vacía' y no hay módulos de inician en el dispositivo

El dispositivo tiene problemas para iniciar los módulos definidos en la implementación. Solo el edgeAgent se está ejecutando pero continuamente reporting 'archivo de configuración vacía'.

### <a name="potential-root-cause"></a>Posible causa principal
De forma predeterminada, IoT Edge inicia módulos en su propia red de contenedor aislado. El dispositivo pueda tener problemas con la resolución de nombres DNS dentro de esta red privada.

### <a name="resolution"></a>Resolución

**Opción 1: Establecer el servidor DNS en el contenedor de configuración del motor**

Especifique el servidor DNS para su entorno en la configuración del motor de contenedor que se aplicará a todos los módulos de contenedor iniciados por el motor. Cree un archivo denominado `daemon.json` especificando el servidor DNS para usar. Por ejemplo:

```
{
    "dns": ["1.1.1.1"]
}
```

El ejemplo anterior establece el servidor DNS a un servicio DNS accesible públicamente. Si el dispositivo perimetral no puede acceder a esta dirección IP de su entorno, reemplácelo por la dirección del servidor DNS que sea accesible.

Lugar `daemon.json` en la ubicación correcta para su plataforma: 

| Plataforma | Ubicación |
| --------- | -------- |
| Linux | `/etc/docker` |
| Host de Windows con contenedores de Windows | `C:\ProgramData\iotedge-moby\config` |

Si la ubicación ya contiene `daemon.json` , agregue el **dns** clave a él y guarde el archivo.

*Reiniciar el motor de contenedor para que las actualizaciones surtan efecto*

| Plataforma | Get-Help |
| --------- | -------- |
| Linux | `sudo systemctl restart docker` |
| Windows (Administrador de Powershell) | `Restart-Service iotedge-moby -Force` |

**Opción 2: Establecer el servidor DNS en la implementación de IoT Edge por cada módulo.**

Puede establecer el servidor DNS para cada módulo *createOptions* en la implementación de IoT Edge. Por ejemplo:

```
"createOptions": {
  "HostConfig": {
    "Dns": [
      "x.x.x.x"
    ]
  }
}
```

No olvide establecer esto para el *edgeAgent* y *edgeHub* módulos también. 

## <a name="next-steps"></a>Pasos siguientes
¿Cree que encontró un error en la plataforma de IoT Edge? [Envíe un problema](https://github.com/Azure/iotedge/issues) para que podamos seguir mejorando. 

Si tiene más preguntas, cree una [solicitud de soporte técnico](https://portal.azure.com/#create/Microsoft.Support) para obtener ayuda. 

