---
title: 'Preparación para implementar una solución en producción: Azure IoT Edge'
description: Obtenga más información sobre cómo llevar su solución Azure IoT Edge desde el desarrollo hasta la producción, incluida la configuración de sus dispositivos con los certificados apropiados y la realización de un plan de implementación para futuras actualizaciones de código.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 7cabae837656611813d44017ce2e1112f06066ef
ms.sourcegitcommit: 5d7f8c57eaae91f7d9cf1f4da059006521ed4f9f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89669608"
---
# <a name="prepare-to-deploy-your-iot-edge-solution-in-production"></a>Preparación para implementar la solución IoT Edge en producción

Cuando esté listo para llevar su solución IoT Edge desde el desarrollo hasta la producción, asegúrese de que está configurada para un rendimiento continuo.

La información proporcionada en este artículo no es la misma para todo. Para ayudarle con la prioridad, cada sección comienza con listas que dividen el trabajo en dos secciones: **importante** para completar antes de pasar a producción, o  **útil** para que se sepa.

## <a name="device-configuration"></a>Configuración del dispositivo

Los dispositivos IoT Edge pueden ser cualquier cosa, desde un dispositivo Raspberry Pi a un portátil o una máquina virtual que se ejecuta en un servidor. Es posible que tenga acceso al dispositivo ya sea físicamente o mediante una conexión virtual, o que esté aislado durante períodos prolongados de tiempo. En cualquier caso, debe asegurarse de que está configurado para funcionar correctamente.

* **Importante**
  * Instalar certificados de producción
  * Disponer de un plan de administración de dispositivos
  * Utilizar Moby como motor de contenedor

* **Útil**
  * Elegir el protocolo ascendente

### <a name="install-production-certificates"></a>Instalar certificados de producción

Cada dispositivo IoT Edge en producción necesita un certificado de la entidad de certificación (CA) instalado en él. Este certificado de entidad de certificación se declara en el entorno de ejecución de Azure IoT Edge en el archivo config.yaml. En escenarios de desarrollo y prueba, el entorno de ejecución de IoT Edge crea certificados temporales si no se declara ningún certificado en el archivo config.yaml. Sin embargo, estos certificados temporales expiran después de tres meses y no son seguros para escenarios de producción. En el caso de los escenarios de producción, debe proporcionar su propio certificado de CA de dispositivo, ya sea de una entidad de certificación autofirmada o adquirido de una entidad de certificación comercial.

> [!NOTE]
> Actualmente, una limitación en libiothsm impide el uso de certificados que expiran el 1 de enero de 2038 o en una fecha posterior.

Para comprender el rol del certificado de entidad de certificación del dispositivo, consulte [Información de uso de los certificados de Azure IoT Edge](iot-edge-certs.md).

Para obtener más información sobre cómo instalar certificados en un dispositivo IoT Edge y hacer referencia a él desde el archivo config.yaml, consulte [Administración de certificados en un dispositivo IoT Edge](how-to-manage-device-certificates.md).

### <a name="have-a-device-management-plan"></a>Disponer de un plan de administración de dispositivos

Antes de poner cualquier dispositivo en producción, debe saber cómo va a administrar las actualizaciones futuras. Para un dispositivo IoT Edge, la lista de componentes que se van a actualizar puede incluir:

* Firmware del dispositivo
* Bibliotecas de sistema operativo
* Motor de contenedor, como Moby
* Demonio de IoT Edge
* Certificados de entidad de certificación

Para obtener más información, vea [Actualización del entorno de ejecución de IoT Edge](how-to-update-iot-edge.md). Los métodos actuales para actualizar el demonio de IoT Edge requieren acceso físico o SSH al dispositivo IoT Edge. Si tiene muchos dispositivos que actualizar, considere la posibilidad de agregar los pasos de actualización a un script o de utilizar una herramienta de automatización como Ansible.

### <a name="use-moby-as-the-container-engine"></a>Utilizar Moby como motor de contenedor

Tener un motor de contenedor es un requisito previo para cualquier dispositivo IoT Edge. Solo se admite en producción el motor Moby. Otros motores de contenedor, como Docker, funcionan con IoT Edge y es correcto usar estos motores para el desarrollo. El motor Moby se puede redistribuir cuando se usa con Azure IoT Edge, y Microsoft proporciona servicio para este motor.

### <a name="choose-upstream-protocol"></a>Elegir el protocolo ascendente

Puede configurar el protocolo (que determina el puerto usado) para la comunicación ascendente con IoT Hub tanto para el agente de IoT Edge como para el centro de IoT Edge. El protocolo predeterminado es AMQP, pero es posible que desee cambiarlo en función de la configuración de la red.

Los dos módulos del entorno de ejecución tienen la variable de entorno **UpstreamProtocol**. Los valores válidos para la variable son:

* MQTT
* AMQP
* MQTTWS
* AMQPWS

Configure la variable UpstreamProtocol para el agente de IoT Edge en el archivo config.yaml del propio dispositivo. Por ejemplo, si el dispositivo IoT Edge está detrás de un servidor proxy que bloquea los puertos AMQP, puede que necesite configurar el agente de IoT Edge para que utilice AMQP sobre WebSocket (AMQPWS) para establecer la conexión inicial con IoT Hub.

Una vez que el dispositivo IoT Edge se conecte, asegúrese de continuar con la configuración de la variable UpstreamProtocol para ambos módulos del entorno de ejecución en futuras implementaciones. Un ejemplo de este proceso se muestra en [Configuración de un dispositivo de IoT Edge para que se comunique a través de un servidor proxy](how-to-configure-proxy-support.md).

## <a name="deployment"></a>Implementación

* **Útil**
  * Ser coherente con el protocolo ascendente
  * Configuración de almacenamiento de host para módulos del sistema
  * Reducir el espacio de memoria utilizado por el centro de IoT Edge
  * No utilizar versiones de depuración de las imágenes de módulo

### <a name="be-consistent-with-upstream-protocol"></a>Ser coherente con el protocolo ascendente

Si ha configurado el agente de IoT Edge en su dispositivo IoT Edge para utilizar un protocolo diferente al AMQP predeterminado, debe declarar el mismo protocolo en todas las futuras implementaciones. Por ejemplo, si el dispositivo IoT Edge está detrás de un servidor proxy que bloquea los puertos AMQP, probablemente ha configurado el dispositivo para que se conecte a través de AMQP sobre WebSocket (AMQPWS). Cuando implemente módulos en el dispositivo, configure el mismo protocolo AMQPWS para el agente de IoT Edge y el centro de IoT Edge, ya que, de lo contrario, el AMQPP predeterminado invalidará la configuración e impedirá que se vuelva a conectar.

Solo tiene que configurar la variable de entorno UpstreamProtocol para el agente de IoT Edge y el centro de IoT Edge. Cualquier módulo adicional adopta cualquier protocolo que se establezca en los módulos del entorno de ejecución.

Un ejemplo de este proceso se muestra en [Configuración de un dispositivo de IoT Edge para que se comunique a través de un servidor proxy](how-to-configure-proxy-support.md).

### <a name="set-up-host-storage-for-system-modules"></a>Configuración de almacenamiento de host para módulos del sistema

Los módulos del agente y del centro de IoT Edge usan almacenamiento local para mantener el estado y permiten la mensajería entre módulos, dispositivos y la nube. Para mejorar la confiabilidad y el rendimiento, configure los módulos del sistema para usar el almacenamiento en el sistema de archivos del host.

Para obtener más información, vea [Almacenamiento de host para módulos del sistema](how-to-access-host-storage-from-module.md).

### <a name="reduce-memory-space-used-by-iot-edge-hub"></a>Reducir el espacio de memoria utilizado por el centro de IoT Edge

Si va a implementar dispositivos restringidos con memoria disponible limitada, puede configurar el centro de IoT Edge para que se ejecute con una capacidad más optimizada y utilice menos espacio en disco. Sin embargo, estas configuraciones limitan el rendimiento del centro de IoT Edge, por lo que debe encontrar el equilibrio adecuado para su solución.

#### <a name="dont-optimize-for-performance-on-constrained-devices"></a>No optimizar el rendimiento en dispositivos restringidos

El centro de IoT Edge está optimizado para el rendimiento de manera predeterminada e intenta asignar grandes fragmentos de memoria. Esta configuración puede causar problemas de estabilidad en dispositivos más pequeños como el Raspberry Pi. Si va a implementar dispositivos con recursos limitados, es posible que quiera establecer la variable de entorno **OptimizeForPerformance** en **false** en el centro de IoT Edge.

Cuando **OptimizeForPerformance** se establece en **true**, el encabezado del protocolo MQTT usa PooledByteBufferAllocator, que tiene un mejor rendimiento, pero asigna más memoria. El asignador no funciona bien en sistemas operativos de 32 bits o en dispositivos con poca memoria. Además, cuando se optimiza para el rendimiento, RocksDb asigna más memoria para su rol como proveedor de almacenamiento local.

Para más información, consulte [Problemas de estabilidad en dispositivos más pequeños](troubleshoot-common-errors.md#stability-issues-on-smaller-devices).

#### <a name="disable-unused-protocols"></a>Deshabilitar los protocolos no utilizados

Otra forma de optimizar el rendimiento del centro de IoT Edge y reducir su uso de memoria es desactivar los encabezados del protocolo de cualquier protocolo que no utilicen su solución.

Los encabezados del protocolo se configuran mediante el establecimiento de variables de entorno booleanas para el módulo del centro de IoT Edge en los manifiestos de implementación. Las tres variables son:

* **amqpSettings__enabled**
* **mqttSettings__enabled**
* **httpSettings__enabled**

Las tres variables tienen *dos caracteres de subrayado* y se pueden establecer como verdadero o falso.

#### <a name="reduce-storage-time-for-messages"></a>Reducir el tiempo de almacenamiento para los mensajes

El módulo del centro de IoT Edge almacena mensajes temporalmente si no se pueden entregar a IoT Hub por cualquier razón. Puede configurar el tiempo que el centro de IoT Edge conserva los mensajes no entregados antes de dejarlos expirar. Si tiene problemas de memoria en el dispositivo, puede reducir el valor **timeToLiveSecs** en el módulo gemelo del centro de IoT Edge.

El valor predeterminado del parámetro timeToLiveSecs es de 7200 segundos, es decir, dos horas.

### <a name="do-not-use-debug-versions-of-module-images"></a>No utilizar versiones de depuración de las imágenes de módulo

Al pasar de escenarios de prueba a escenarios de producción, recuerde quitar las configuraciones de depuración de los manifiestos de implementación. Compruebe que ninguna de las imágenes de módulo en los manifiestos de implementación tiene el sufijo **\.debug**. Si agregó opciones de creación para exponer puertos en los módulos de depuración, quite también estas opciones.

## <a name="container-management"></a>Administración de contenedores

* **Importante**
  * Administrar el acceso al registro de contenedor
  * Usar etiquetas para administrar versiones
* **Útil**
  * Almacenar los contenedores del entorno de ejecución en el registro privado

### <a name="manage-access-to-your-container-registry"></a>Administrar el acceso al registro de contenedor

Antes de implementar módulos en dispositivos de producción IoT Edge, asegúrese de controlar el acceso al registro de contenedor para que las personas ajenas no puedan acceder a las imágenes de contenedor o realizar cambios en ellas. Utilice un registro de contenedores privado, no público, para administrar las imágenes de contenedor.

En los tutoriales y otra documentación, le indicamos que utilice las mismas credenciales de registro de contenedor en el dispositivo IoT Edge que en la máquina de desarrollo. Estas instrucciones solo pretenden ayudarle a configurar entornos de pruebas y desarrollo con mayor facilidad, y no deben seguirse en un escenario de producción.

Para obtener un acceso más seguro al registro, tiene una selección de [opciones de autenticación](../container-registry/container-registry-authentication.md). Un método de autenticación conocido y recomendado consiste en usar una entidad de servicio de Active Directory que sea adecuada para que las aplicaciones o los servicios extraigan imágenes de contenedor de forma automática o cualquier otra forma desatendida (sin supervisión directa), como lo hacen los dispositivos IoT Edge.

Para crear una entidad de servicio, ejecute los dos scripts como se describe en [Creación de una entidad de servicio](../container-registry/container-registry-auth-service-principal.md#create-a-service-principal). Estos scripts realizan las siguientes tareas:

* El primer script crea la entidad de servicio. Devuelve el id. de entidad de servicio y la contraseña de la entidad de servicio. Almacene estos valores de forma segura en los registros.

* El segundo script crea asignaciones de roles para concederlos a la entidad de servicio, que se puede ejecutar posteriormente si es necesario. Se recomienda aplicar el rol de usuario **acrPull** para el parámetro `role`. Para obtener una lista de roles, consulte [Roles y permisos de Azure Container Registry](../container-registry/container-registry-roles.md).

Para autenticarse con una entidad de servicio, proporcione el id y la contraseña de la entidad de servicio que obtuvo del primer script. Especifique estas credenciales en el manifiesto de implementación.

* Como nombre de usuario o id. de cliente, especifique el identificador de la entidad de servicio.

* Como contraseña o secreto de cliente, especifique la contraseña de la entidad de servicio.

> [!NOTE]
> Después de implementar una autenticación de seguridad mejorada, deshabilite el valor **Usuario administrador** para que el acceso de nombre de usuario y contraseña predeterminados deje de estar disponible. En el registro de contenedor en Azure Portal, seleccione del menú del panel izquierdo, dentro de **Configuración**, la opción **Claves de acceso**.

### <a name="use-tags-to-manage-versions"></a>Usar etiquetas para administrar versiones

Una etiqueta es un concepto de Docker que puede utilizar para distinguir entre versiones de contenedores Docker. Las etiquetas son sufijos como **1.0** que van al final de un repositorio de contenedores. Por ejemplo, **mcr.microsoft.com/azureiotedge-agent:1.0**. Las etiquetas son mutables y pueden cambiarse para que apunten a otro contenedor en cualquier momento, por lo que su equipo debe acordar una convención a seguir a medida que actualice las imágenes de sus módulos.

Las etiquetas también le ayudan a aplicar las actualizaciones en sus dispositivos IoT Edge. Cuando inserte una versión actualizada de un módulo en el registro de contenedor, aumente la etiqueta. A continuación, inserte una nueva implementación a sus dispositivos con la etiqueta incrementada. El motor de contenedor reconocerá la etiqueta de incremento como una nueva versión y extraerá la versión más reciente del módulo en el dispositivo.

Para ver un ejemplo de una convención de etiquetas, consulte [Actualización del entorno de ejecución de Azure IoT Edge](how-to-update-iot-edge.md#understand-iot-edge-tags) para aprender cómo IoT Edge utiliza etiquetas rodantes y etiquetas específicas para realizar el seguimiento de las versiones.

### <a name="store-runtime-containers-in-your-private-registry"></a>Almacenar los contenedores del entorno de ejecución en el registro privado

Sabe cómo almacenar las imágenes de contenedor para los módulos de código personalizados en el registro privado de Azure, aunque también puede usarlo para almacenar imágenes de contenedor públicas, por ejemplo, para los módulos de entorno de ejecución de edgeAgent y edgeHub. Esto puede ser necesario si tiene restricciones de firewall muy estrictas, ya que estos contenedores del entorno de ejecución se almacenan en Microsoft Container Registry (MCR).

Obtenga las imágenes con el comando docker pull para colocarlas en el registro privado. Tenga en cuenta que tendrá que actualizar las imágenes con cada nueva versión del entorno de ejecución de Azure IoT Edge.

| Contenedor del entorno de ejecución de Azure IoT Edge | Comando docker pull |
| --- | --- |
| [Agente de Azure IoT Edge](https://hub.docker.com/_/microsoft-azureiotedge-agent) | `docker pull mcr.microsoft.com/azureiotedge-agent` |
| [Centro de Azure IoT Edge](https://hub.docker.com/_/microsoft-azureiotedge-hub) | `docker pull mcr.microsoft.com/azureiotedge-hub` |

Después, asegúrese de actualizar las referencias de imagen en el archivo deployment.template.json para los módulos del sistema edgeAgent y edgeHub. Reemplace `mcr.microsoft.com` por el nombre del registro y el servidor de ambos módulos.

* edgeAgent:

    `"image": "<registry name and server>/azureiotedge-agent:1.0",`

* edgeHub:

    `"image": "<registry name and server>/azureiotedge-hub:1.0",`

## <a name="networking"></a>Redes

* **Útil**
  * Revisar configuración de entrada y salida
  * Permitir conexiones desde dispositivos IoT Edge
  * Configurar la comunicación a través un servidor proxy

### <a name="review-outboundinbound-configuration"></a>Revisar configuración de entrada y salida

Los canales de comunicación entre Azure IoT Hub e IoT Edge siempre están configurados para que sean la salida. Para la mayoría de los escenarios de IoT Edge, solo se necesitan tres conexiones. El motor de contenedor necesita conectarse con el registro (o registros) de contenedor que contiene las imágenes del módulo. El entorno de ejecución de Azure IoT Edge necesita conectarse con IoT Hub para recuperar la información de configuración del dispositivo y para enviar mensajes y telemetría. Y si utiliza el aprovisionamiento automático, el demonio de IoT Edge necesita conectarse al servicio de aprovisionamiento de dispositivos. Para más información, consulte [Reglas de configuración de puertos y firewall](troubleshoot.md#check-your-firewall-and-port-configuration-rules).

### <a name="allow-connections-from-iot-edge-devices"></a>Permitir conexiones desde dispositivos IoT Edge

Si la configuración de red requiere agregar en la lista de permitidos las conexiones hechas desde dispositivos IoT Edge, revise la siguiente lista de componentes de IoT Edge:

* El **agente de IoT Edge** abre una conexión AMQP/MQTT persistente con IoT Hub, posiblemente a través de WebSockets.
* El **centro de IoT Edge** abre una única conexión AMQP persistente o varias conexiones MQTT con IoT Hub, posiblemente a través de WebSockets.
* El **demonio de IoT Edge** realiza las llamadas HTTPS intermitentes a IoT Hub.

En los tres casos, el nombre DNS coincidiría con el patrón \*.azure-devices.net.

Además, el **motor de contenedor** realiza llamadas a los registros de contenedor mediante HTTPS. Para recuperar las imágenes de contenedor del entorno de ejecución de Azure IoT Edge, el nombre DNS es mcr.microsoft.com. El motor de contenedor se conecta a otros registros tal y como se ha configurado en la implementación.

Esta lista de comprobación es un punto de partida para las reglas de firewall:

   | URL (\* = comodín) | Puertos TCP salientes | Uso |
   | ----- | ----- | ----- |
   | mcr.microsoft.com  | 443 | Registro de contenedor de Microsoft |
   | global.azure-devices-provisioning.net  | 443 | Acceso a DPS (opcional) |
   | \*.azurecr.io | 443 | Registros de contenedores personales y de terceros |
   | \*blob.core.windows.net | 443 | Descargar deltas de imágenes de Azure Container Registry desde Blob Storage |
   | \*.azure-devices.net | 5671, 8883, 443 | Acceso de IoT Hub |
   | \*.docker.io  | 443 | Acceso a Docker Hub (opcional) |

Algunas de estas reglas de firewall se heredan de Azure Container Registry. Para más información, consulte [Configuración de reglas para acceder a un registro de contenedor de Azure desde detrás de un firewall](../container-registry/container-registry-firewall-access-rules.md).

> [!NOTE]
> Para proporcionar un FQDN coherente entre los puntos de conexión de REST y de datos, a partir del **15 de junio de 2020** el punto de conexión de datos de Microsoft Container Registry cambiará de `*.cdn.mscr.io` a `*.data.mcr.microsoft.com`  
> Para obtener más información, consulte [Configuración de las reglas de firewall de cliente de Microsoft Container Registry](https://github.com/microsoft/containerregistry/blob/master/client-firewall-rules.md)

Si no quiere configurar el firewall para permitir el acceso a los registros de contenedores públicos, puede almacenar las imágenes en el registro de contenedor privado, como se describe en [Almacenar los contenedores del entorno de ejecución en el registro privado](#store-runtime-containers-in-your-private-registry).

### <a name="configure-communication-through-a-proxy"></a>Configurar la comunicación a través un servidor proxy

Si sus dispositivos se van a implementar en una red que utiliza un servidor proxy, deben ser capaces de comunicarse a través del proxy para llegar a IoT Hub y a los registros de contenedor. Para más información, consulte [Configuración de un dispositivo de IoT Edge para que se comunique a través de un servidor proxy](how-to-configure-proxy-support.md).

## <a name="solution-management"></a>Administración de soluciones

* **Útil**
  * Configurar los registros y diagnósticos
  * Considerar la posibilidad de pruebas y canalizaciones de CI/CD

### <a name="set-up-logs-and-diagnostics"></a>Configurar los registros y diagnósticos

En Linux, el demonio de IoT Edge utiliza journals como controlador de registro predeterminado. Puede usar la herramienta de línea de comandos `journalctl` para consultar los registros del demonio. En Windows, el demonio de IoT Edge utiliza diagnósticos de PowerShell. Use `Get-IoTEdgeLog` para consultar los registros del demonio. Los módulos de IoT Edge utilizan el controlador JSON para el registro, que es el valor predeterminado.  

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

Cuando está probando una implementación de IoT Edge, normalmente puede acceder a sus dispositivos para recuperar registros y solucionar problemas. En un escenario de implementación, es posible que no tenga esa opción. Considere cómo va a recopilar información sobre los dispositivos en producción. Una opción es usar un módulo de registro que recoge información de los otros módulos y la envía a la nube. Un ejemplo de un módulo de registro es [logspout-loganalytics](https://github.com/veyalla/logspout-loganalytics) o puede diseñar el suyo propio.

### <a name="place-limits-on-log-size"></a>Colocar límites en el tamaño de registro

De forma predeterminada el motor del contenedor Moby no establece límites de tamaño de registro de contenedor. Con el tiempo, puede que el dispositivo se llene con registros y se quede sin espacio en disco. Considere las opciones siguientes para evitarlo:

#### <a name="option-set-global-limits-that-apply-to-all-container-modules"></a>Opción: establecer límites globales que se aplican a todos los módulos del contenedor

Puede limitar el tamaño de todos los archivos de registro del contenedor en las opciones de registro del motor del contenedor. En el ejemplo siguiente se establece el controlador de registro en `json-file` (recomendado) con límites de tamaño y número de archivos:

```JSON
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    }
}
```

Agregue (o anexe) esta información a un archivo denominado `daemon.json` y colóquelo en la ubicación adecuada para su plataforma del dispositivo.

| Plataforma | Location |
| -------- | -------- |
| Linux | `/etc/docker/` |
| Windows | `C:\ProgramData\iotedge-moby\config\` |

El motor del contenedor debe reiniciarse para que los cambios surtan efecto.

#### <a name="option-adjust-log-settings-for-each-container-module"></a>Opción: ajustar la configuración de registro de cada módulo del contenedor

Puede hacerlo en el elemento **createOptions** de cada módulo. Por ejemplo:

```yml
"createOptions": {
    "HostConfig": {
        "LogConfig": {
            "Type": "json-file",
            "Config": {
                "max-size": "10m",
                "max-file": "3"
            }
        }
    }
}
```

#### <a name="additional-options-on-linux-systems"></a>Opciones adicionales en sistemas Linux

* Configure el motor del contenedor para enviar registros al [diario](https://docs.docker.com/config/containers/logging/journald/) de `systemd` estableciendo `journald` como controlador de registro predeterminado.

* Quite periódicamente los registros antiguos del dispositivo mediante la instalación de una herramienta de logrotate. Utilice la siguiente especificación de archivo:

   ```txt
   /var/lib/docker/containers/*/*-json.log{
       copytruncate
       daily
       rotate7
       delaycompress
       compress
       notifempty
       missingok
   }
   ```

### <a name="consider-tests-and-cicd-pipelines"></a>Considerar la posibilidad de pruebas y canalizaciones de CI/CD

Para el escenario de implementación más eficiente de IoT Edge, considere la posibilidad de integrar la implementación de producción en las canalizaciones de pruebas y de CI/CD. Azure IoT Edge admite varias plataformas de CI/CD, incluida Azure DevOps. Para más información, consulte [Integración continua e implementación continua en Azure IoT Edge](how-to-continuous-integration-continuous-deployment.md).

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de la [implementación automática de IoT Edge](module-deployment-monitoring.md).
* Consulte como IoT Edge admite la [integración continua e implementación continua](how-to-continuous-integration-continuous-deployment.md).
