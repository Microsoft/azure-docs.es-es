---
title: 'Solución de problemas: Azure IoT Edge | Microsoft Docs'
description: Use este artículo para obtener información sobre las aptitudes de diagnóstico estándar para Azure IoT Edge, como la recuperación de los registros y el estado del componente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/27/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 0e4ec7127df288ec1818df307da1ea9824141309
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87902463"
---
# <a name="troubleshoot-your-iot-edge-device"></a>Solución de problemas del dispositivo IoT Edge

Si experimenta problemas al ejecutar Azure IoT Edge en el entorno, use este artículo como guía para solucionar problemas y de diagnóstico.

## <a name="run-the-check-command"></a>Ejecución del comando "check"

El primer paso a la hora de solucionar problemas de IoT Edge debe ser usar el comando `check`, que ejecuta una serie de pruebas de configuración y conectividad para problemas comunes. El comando `check` está disponible en la [versión 1.0.7](https://github.com/Azure/azure-iotedge/releases/tag/1.0.7) y posteriores.

>[!NOTE]
>La herramienta de solución de problemas no puede ejecutar comprobaciones de conectividad si el dispositivo IoT Edge está detrás de un servidor proxy.

Puede ejecutar el comando `check` como se indica a continuación, o bien incluir la marca `--help` para ver una lista completa de opciones:

En Linux:

```bash
sudo iotedge check
```

En Windows:

```powershell
iotedge check
```

La herramienta de solución de problemas ejecuta muchas comprobaciones que se clasifican en estas tres categorías:

* Las comprobaciones de seguridad examinan detalles que podrían evitar que los dispositivos de IoT Edge se conectaran a la nube, incluidos problemas con *config.yaml* y el motor del contenedor *.*
* Las *comprobaciones de conexión* comprueban que el entorno en tiempo de ejecución de IoT Edge pueda acceder a los puertos en el dispositivo de host y que todos los componentes de IoT Edge puedan conectarse a IoT Hub. Este conjunto de comprobaciones devuelve errores si el dispositivo IoT Edge está detrás de un proxy.
* Las *comprobaciones de preparación para producción* buscan procedimientos recomendados de producción, como el estado de los certificados de la entidad de certificación (CA) del dispositivo y la configuración del archivo de registro de módulo.

Para información sobre cada una de las comprobaciones de diagnóstico que ejecuta esta herramienta, como qué hacer si se recibe un error o una advertencia, consulte [Comprobaciones de solución de problemas de IoT Edge](https://github.com/Azure/iotedge/blob/master/doc/troubleshoot-checks.md).

## <a name="gather-debug-information-with-support-bundle-command"></a>Recopilación de información de depuración con el comando "support-bundle"

Cuando necesite recopilar registros de un dispositivo IoT Edge, la manera más cómoda es usar el comando `support-bundle`. Este comando recopila de forma predeterminada los registros del módulo, del administrador de seguridad de IoT Edge y del motor de contenedor, la salida JSON `iotedge check` y otra información de depuración de utilidad. y, luego, lo comprime todo en un solo archivo para poder compartirlo fácilmente. El comando `support-bundle` está disponible en la [versión 1.0.9](https://github.com/Azure/azure-iotedge/releases/tag/1.0.9) y posteriores.

Ejecute el comando `support-bundle` con la marca `--since` para especificar hasta qué momento en el pasado deben remontarse los registros. Por ejemplo, con `6h` se obtendrán registros de las últimas seis horas, con `6d`, de los últimos seis días y con `6m`, de los últimos seis minutos, etc. Incluya la marca `--help` para ver una lista completa de opciones.

En Linux:

```bash
sudo iotedge support-bundle --since 6h
```

En Windows:

```powershell
iotedge support-bundle --since 6h
```

> [!WARNING]
> La salida del comando `support-bundle` puede contener nombres de hosts, de dispositivos y de módulos, información registrada por los módulos, etc. Tenga esto en cuenta si comparte la salida en un foro público.

## <a name="check-your-iot-edge-version"></a>Comprobación de la versión de IoT Edge

Si está ejecutando una versión anterior de IoT Edge, la actualización puede resolver el problema. La herramienta `iotedge check` comprueba que la versión del demonio de seguridad de IoT Edge es la más reciente, pero no comprueba las versiones de los módulos del concentrador y del agente de IoT Edge. Para comprobar la versión de los módulos del entorno de ejecución en el dispositivo, use los comandos `iotedge logs edgeAgent` y `iotedge logs edgeHub`. El número de versión se declara en los registros cuando se inicia el módulo.

Para obtener instrucciones sobre cómo actualizar el dispositivo, consulte [Actualización del demonio de seguridad y el entorno de ejecución de IoT Edge](how-to-update-iot-edge.md).

## <a name="check-the-status-of-the-iot-edge-security-manager-and-its-logs"></a>Comprobación del estado del administrador de seguridad de IoT Edge y sus registros

El [administrador de seguridad de IoT Edge](iot-edge-security-manager.md) es responsable de operaciones como la inicialización del sistema de IoT Edge en los dispositivos de inicio y aprovisionamiento. Si IoT Edge no se inicia, los registros del administrador de seguridad pueden proporcionar información útil.

En Linux:

* Vea el estado del administrador de seguridad de IoT Edge:

   ```bash
   sudo systemctl status iotedge
   ```

* Vea los registros del administrador de seguridad de IoT Edge:

    ```bash
    sudo journalctl -u iotedge -f
    ```

* Vea los registros más detallados del administrador de seguridad de IoT Edge:

  * Edite la configuración del demonio de IoT Edge:

      ```bash
      sudo systemctl edit iotedge.service
      ```

  * Actualice las líneas siguientes:

      ```bash
      [Service]
      Environment=IOTEDGE_LOG=edgelet=debug
      ```

  * Reinicie el demonio de seguridad de IoT Edge:

      ```bash
      sudo systemctl cat iotedge.service
      sudo systemctl daemon-reload
      sudo systemctl restart iotedge
      ```

En Windows:

* Vea el estado del administrador de seguridad de IoT Edge:

   ```powershell
   Get-Service iotedge
   ```

* Vea los registros del administrador de seguridad de IoT Edge:

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
   ```

* Vea solamente los últimos cinco minutos de los registros del administrador de seguridad de IoT Edge:

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog -StartTime ([datetime]::Now.AddMinutes(-5))
   ```

* Vea los registros más detallados del administrador de seguridad de IoT Edge:

  * Agregue una variable de entorno del sistema

      ```powershell
      [Environment]::SetEnvironmentVariable("IOTEDGE_LOG", "debug", [EnvironmentVariableTarget]::Machine)
      ```

  * Reinicie el demonio de seguridad de IoT Edge:

      ```powershell
      Restart-Service iotedge
      ```

### <a name="if-the-iot-edge-security-manager-is-not-running-verify-your-yaml-configuration-file"></a>Comprobación del archivo de configuración de YAML si no se está ejecutando el administrador de seguridad de IoT Edge

> [!WARNING]
> Los archivos de YAML no pueden contener tabulaciones como sangría. Utilice en su lugar dos espacios. Los elementos de nivel superior no deben tener espacios iniciales.

En Linux:

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

En Windows:

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
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

## <a name="check-container-logs-for-issues"></a>Comprobación de los registros del contenedor para detectar problemas

Cuando el demonio de seguridad de IoT Edge se esté ejecutando, examine los registros de los contenedores para detectar problemas. Empiece por los contenedores implementados y, luego, examine los contenedores que componen el runtime de IoT Edge: edgeAgent y edgeHub. Los registros del agente de IoT Edge normalmente proporcionan información sobre el ciclo de vida de cada contenedor. Los registros del centro de IoT Edge proporcionan información sobre mensajería y enrutamiento.

```cmd
iotedge logs <container name>
```

## <a name="view-the-messages-going-through-the-iot-edge-hub"></a>Visualización de mensajes que se envían a través del centro de IoT Edge

Puede ver los mensajes que se envían a través del centro de IoT Edge y recopilar la información que se encuentra en los registros detallados procedentes de los contenedores del runtime. Para activar los registros detallados en estos contenedores, establezca `RuntimeLogLevel` en el archivo de configuración yaml. Para abrir el archivo:

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
   > Los archivos de YAML no pueden contener tabulaciones como sangría. Utilice en su lugar dos espacios. Los elementos de nivel superior no pueden tener espacios en blanco iniciales.

Guarde el archivo y reinicie el administrador de seguridad de IoT Edge.

También puede comprobar los mensajes que se envían entre los dispositivos de IoT Hub e IoT Edge. Vea estos mensajes mediante la extensión de [Azure IoT Hub para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit). Para más información, consulte [Handy tool when you develop with Azure IoT](https://blogs.msdn.microsoft.com/iotdev/2017/09/01/handy-tool-when-you-develop-with-azure-iot/) (Herramienta práctica para desarrollar con Azure IoT).

## <a name="restart-containers"></a>Reinicio de los contenedores

Después de investigar los registros y mensajes para obtener información, puede intentar reiniciar los contenedores:

```cmd
iotedge restart <container name>
```

Reinicie los contenedores del entorno de ejecución de IoT Edge:

```cmd
iotedge restart edgeAgent && iotedge restart edgeHub
```

## <a name="check-your-firewall-and-port-configuration-rules"></a>Comprobación de las reglas de configuración de los puertos y el firewall

Azure IoT Edge permite la comunicación desde un servidor local a la nube de Azure mediante protocolos de IoT Hub compatibles; vea [Elección de un protocolo de comunicación](../iot-hub/iot-hub-devguide-protocols.md). Para mejorar la seguridad, los canales de comunicación entre Azure IoT Edge y Azure IoT Hub siempre están configurados para que sea la salida. Esta configuración se basa en el [patrón de comunicación asistida de servicios](https://blogs.msdn.microsoft.com/clemensv/2014/02/09/service-assisted-communication-for-connected-devices/), que minimiza la superficie de ataque que una entidad malintencionada puede explorar. La comunicación entrante solo es necesaria para escenarios específicos donde Azure IoT Hub necesita insertar mensajes en el dispositivo de Azure IoT Edge. Los mensajes de nube a dispositivo están protegidos mediante canales TLS seguros y pueden protegerse aún más mediante los certificados X.509 y los módulos de dispositivos TPM. El Administrador de seguridad de Azure IoT Edge rige cómo se puede establecer esta comunicación; consulte [Administrador de seguridad de IoT Edge](../iot-edge/iot-edge-security-manager.md).

Aunque IoT Edge permite una mejor configuración para proteger el entorno de ejecución de Azure IoT Edge y los módulos implementados, todavía depende de la configuración de la máquina y la red subyacentes. Por lo tanto, es fundamental garantizar que existan reglas adecuadas de red y firewall configuradas para una comunicación segura entre Edge y la nube. La siguiente tabla se puede usar como guía al configurar reglas de firewall para los servidores subyacentes donde se hospeda el runtime de Azure IoT Edge:

|Protocolo|Port|Entrante|Saliente|Guía|
|--|--|--|--|--|
|MQTT|8883|BLOQUEADO (valor predeterminado)|BLOQUEADO (valor predeterminado)|<ul> <li>Configure el valor de Saliente (de salida) para que sea Abierto cuando se usa MQTT como protocolo de comunicación.<li>1883 para MQTT no es compatible con IoT Edge. <li>Se deben bloquear las conexiones entrantes (de entrada).</ul>|
|AMQP|5671|BLOQUEADO (valor predeterminado)|ABIERTO (valor predeterminado)|<ul> <li>Protocolo de comunicación predeterminado de IoT Edge. <li> Debe configurarse para ser Abierto si Azure IoT Edge no está configurado para otros protocolos compatibles o AMQP es el protocolo de comunicación que se desea.<li>5672 para AMQP no es compatible con IoT Edge.<li>Bloquee este puerto cuando Azure IoT Edge use un protocolo compatible de IoT Hub diferente.<li>Se deben bloquear las conexiones entrantes (de entrada).</ul></ul>|
|HTTPS|443|BLOQUEADO (valor predeterminado)|ABIERTO (valor predeterminado)|<ul> <li>Configure el valor de las conexiones salientes (de salida) para que sea abierto en 443 para el aprovisionamiento de IoT Edge. Esta configuración es necesaria cuando se usen scripts manuales o Azure IoT Device Provisioning Service (DPS). <li>La conexión entrante (de entrada) solo debe ser Abierta en escenarios concretos: <ul> <li>  Si tiene una puerta de enlace transparente con dispositivos de hoja que puedan enviar solicitudes de método. En este caso, no es necesario que el puerto 443 esté abierto a las redes externas para conectarse a IOT Hub ni proporcionar servicios de IoTHub mediante Azure IoT Edge. Por lo tanto, la regla de entrada podría limitarse solo a los puertos entrantes (de entrada) abiertos desde la red interna. <li> En el caso de escenarios de cliente a dispositivo (C2D).</ul><li>80 para HTTP no es compatible con IoT Edge.<li>Si no se pueden configurar los protocolos que no sean HTTP (por ejemplo, AMQP o MQTT) en la empresa; los mensajes pueden enviarse a través de WebSockets. En ese caso, el puerto 443 se utilizará para la comunicación de WebSocket.</ul>|

## <a name="next-steps"></a>Pasos siguientes

¿Cree que encontró un error en la plataforma de IoT Edge? [Envíe un problema](https://github.com/Azure/iotedge/issues) para que podamos seguir mejorando.

Si tiene más preguntas, cree una [solicitud de soporte técnico](https://portal.azure.com/#create/Microsoft.Support) para obtener ayuda.
