---
title: Instalación de Azure IoT Edge en Linux | Microsoft Docs
description: Instrucciones de instalación de Azure IoT Edge en dispositivos Linux que ejecutan Ubuntu o Raspbian
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: kgremban
ms.openlocfilehash: 4078d7e6c20571db2387cfd138ecb325fc3469e7
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022095"
---
# <a name="install-the-azure-iot-edge-runtime-on-debian-based-linux-systems"></a>Instalación del entorno de ejecución de Azure IoT Edge en sistemas Linux basados en Debian

El entorno de ejecución de Azure IoT Edge es lo que convierte a un dispositivo en un dispositivo IoT Edge. El entorno de ejecución se puede implementar en dispositivos tan pequeños como un Raspberry Pi o tan grandes como un servidor industrial. Una vez que un dispositivo está configurado con el entorno de ejecución de IoT Edge, puede empezar a implementar lógica de negocios en él desde la nube. Para obtener más información, consulte [Información del entorno de ejecución de Azure IoT Edge y su arquitectura](iot-edge-runtime.md).

En este artículo se enumeran los pasos para instalar el entorno de ejecución de Azure IoT Edge en un dispositivo Linux X64, ARM32 o ARM64. Se proporcionan paquetes de instalación para Ubuntu Server 16.04, Ubuntu Server 18.04 y Raspbian Stretch. Para obtener una lista de las arquitecturas y sistemas operativos Linux compatibles, consulte [Sistemas compatibles con Azure IoT Edge](support.md#operating-systems).

> [!NOTE]
> Los paquetes en los repositorios de software de Linux están sujetos a los términos de licencia que se encuentran en cada paquete (/usr/share/doc/*nombre-de-paquete*). Lea los términos de licencia antes de usar el paquete. La instalación y uso del paquete constituye la aceptación de estos términos. Si no acepta los términos de licencia, no utilice el paquete.

## <a name="install-iot-edge-and-container-runtimes"></a>Instalación de IoT Edge y tiempos de ejecución de contenedor

Utilice las secciones siguientes para instalar la versión más reciente del entorno de ejecución de Azure IoT Edge en sus dispositivos.

>[!NOTE]
>La compatibilidad con dispositivos ARM64 está en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="register-microsoft-key-and-software-repository-feed"></a>Registro de la clave y la fuente del repositorio de software de Microsoft

Prepare el dispositivo para la instalación del entorno de ejecución de Azure IoT Edge.

Instale la configuración del repositorio. Elija el comando **16.04** o **18.04** que coincida con el sistema operativo del dispositivo:

* **Ubuntu Server 16.04**:

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/16.04/multiarch/prod.list > ./microsoft-prod.list
   ```

* **Ubuntu Server 18.04**:

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
   ```

* **Raspbian Stretch**:

   ```bash
   curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
   ```

Copie la lista generada.

   ```bash
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

Instalación de la clave pública de GPG de Microsoft.

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

### <a name="install-a-container-runtime"></a>Instalación de un entorno de ejecución del contenedor

Azure IoT Edge utiliza un runtime de contenedor [compatible con OCI](https://www.opencontainers.org/). En los escenarios de producción, se recomienda utilizar el motor [basado en Moby](https://mobyproject.org/) que se proporciona a continuación. El motor Moby es el único motor de contenedor compatible oficialmente con Azure IoT Edge. Las imágenes de contenedor de Docker CE/EE son totalmente compatibles con el entorno de ejecución de Moby.

Actualice las listas de paquetes en el dispositivo.

   ```bash
   sudo apt-get update
   ```

Instale el motor de Moby.

   ```bash
   sudo apt-get install moby-engine
   ```

Instale la interfaz de la línea de comandos (CLI) de Moby. La CLI es útil para el desarrollo pero opcional para implementaciones de producción.

   ```bash
   sudo apt-get install moby-cli
   ```

Si se producen errores al instalar el entorno de ejecución del contenedor de Moby, siga los pasos que se proporcionan más adelante en este artículo para [comprobar la compatibilidad del kernel de Linux con Moby](#verify-your-linux-kernel-for-moby-compatibility).

### <a name="install-the-azure-iot-edge-security-daemon"></a>Instalación del demonio de seguridad de Azure IoT Edge

El **demonio de seguridad de IoT Edge** proporciona y mantiene los estándares de seguridad en el dispositivo IoT Edge. El demonio se inicia en cada arranque e inicia el resto del entorno de ejecución de IoT Edge para arrancar el dispositivo.

Actualice las listas de paquetes en el dispositivo.

   ```bash
   sudo apt-get update
   ```

Compruebe qué versiones de IoT Edge están disponibles.

   ```bash
   apt list -a iotedge
   ```

Si desea instalar la versión más reciente del demonio de seguridad, use el siguiente comando, que también instala la versión más reciente de **libiothsm-std**:

   ```bash
   sudo apt-get install iotedge
   ```

Si desea instalar una versión específica del demonio de seguridad, especifique la versión en la salida de la lista apt. Especifique también la misma versión para el paquete **libiothsm-std**, que de lo contrario instalaría su versión más reciente. Por ejemplo, el siguiente comando instala la versión más reciente de la versión 1.0.8:

   ```bash
   sudo apt-get install iotedge=1.0.8* libiothsm-std=1.0.8*
   ```

Si la versión que desea instalar no aparece en la lista, siga los pasos descritos en [Instalación de tiempo de ejecución mediante recursos de versión](#install-runtime-using-release-assets). En esta sección se muestra cómo seleccionar cualquier versión anterior del demonio de seguridad de IoT Edge o versiones candidatas para lanzamiento.

Una vez que IoT Edge se ha instalado correctamente en `/etc/iotedge/`, se le solicita que actualice el archivo de configuración. Siga los pasos de la sección siguiente para completar el aprovisionamiento de dispositivos.

## <a name="configure-the-security-daemon"></a>Configuración del demonio de seguridad

Configure el entorno de ejecución de Azure IoT Edge para vincular el dispositivo físico con una identidad de dispositivo que exista en un centro de Azure IoT.

El demonio se puede configurar mediante el archivo de configuración en `/etc/iotedge/config.yaml`. De forma predeterminada el archivo está protegido contra escritura, puede que necesite permisos elevados para editarlo.

Un único dispositivo de IoT Edge se puede aprovisionar manualmente mediante una cadena de conexiones de dispositivo proporcionada por IoT Hub. O bien, puede usar el servicio Device Provisioning para aprovisionar dispositivos automáticamente, lo que resulta útil cuando tiene muchos dispositivos para aprovisionar. Según la elección de aprovisionamiento, elija el script de instalación adecuado.

### <a name="option-1-manual-provisioning"></a>Opción 1: Aprovisionamiento manual

Para aprovisionar manualmente un dispositivo, debe proporcionarle una [cadena de conexión](how-to-register-device.md#register-in-the-azure-portal), que puede crear mediante el registro de un dispositivo nuevo en su instancia de IoT Hub.

Abra el archivo de configuración.

```bash
sudo nano /etc/iotedge/config.yaml
```

Busque las configuraciones de aprovisionamiento del archivo y quite la marca de comentario de la sección **Manual provisioning configuration**. Actualice el valor de **device_connection_string** con la cadena de conexión desde un dispositivo IoT Edge. Asegúrese de que las demás secciones de aprovisionamiento estén comentadas. Asegúrese de que la línea **provisioning:** no tiene ningún espacio en blanco delante y de que los elementos anidados muestran una sangría de dos espacios.

```yml
# Manual provisioning configuration
provisioning:
  source: "manual"
  device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
```

Para pegar el contenido del portapapeles en Nano: `Shift+Right Click` o presione `Shift+Insert`.

Guarde y cierre el archivo.

   `CTRL + X`, `Y`, `Enter`

Después de especificar la información de aprovisionamiento en el archivo de configuración, reinicie el demonio:

```bash
sudo systemctl restart iotedge
```

### <a name="option-2-automatic-provisioning"></a>Opción 2: Aprovisionamiento automático

Los dispositivos IoT Edge pueden aprovisionarse automáticamente con [Azure IoT Hub Device Provisioning Service (DPS)](../iot-dps/index.yml). Actualmente, IoT Edge admite tres mecanismos de atestación cuando se usa el aprovisionamiento automático, pero los requisitos de hardware pueden afectar a sus elecciones. Por ejemplo, de forma predeterminada, los dispositivos Raspberry Pi no cuentan con un chip Módulo de plataforma segura (TPM). Para más información, consulte los siguientes artículos.

* [Creación y aprovisionamiento de un dispositivo IoT Edge con un TPM virtual en una máquina virtual Linux](how-to-auto-provision-simulated-device-linux.md)
* [Creación y aprovisionamiento de un dispositivo IoT Edge mediante certificados X.509](how-to-auto-provision-x509-certs.md)
* [Creación y aprovisionamiento de un dispositivo IoT Edge mediante la atestación de clave simétrica](how-to-auto-provision-symmetric-keys.md)

Estos artículos le guían por los pasos para configurar inscripciones en DPS y generar los certificados o claves adecuados para la atestación. Con independencia del mecanismo de atestación que elija, la información de aprovisionamiento se agrega al archivo de configuración de IoT Edge en el dispositivo IoT Edge.

Abra el archivo de configuración.

```bash
sudo nano /etc/iotedge/config.yaml
```

Busque las configuraciones de aprovisionamiento del archivo y quite la marca de comentario de la sección adecuada para el mecanismo de atestación. Asegúrese de que las demás secciones de aprovisionamiento estén comentadas. La línea **provisioning:** no debe ir precedida por espacios en blanco y se debe aplicar una sangría de dos espacios a los elementos anidados. Actualice el valor de **scope_id** con el valor de la instancia de IoT Hub Device Provisioning Service y proporcione los valores adecuados en los campos de atestación.

Atestación de TPM:

```yml
# DPS TPM provisioning configuration
provisioning:
  source: "dps"
  global_endpoint: "https://global.azure-devices-provisioning.net"
  scope_id: "<SCOPE_ID>"
  attestation:
    method: "tpm"
    registration_id: "<REGISTRATION_ID>"
```

Atestación de X.509:

```yml
# DPS X.509 provisioning configuration
provisioning:
  source: "dps"
  global_endpoint: "https://global.azure-devices-provisioning.net"
  scope_id: "<SCOPE_ID>"
  attestation:
    method: "x509"
#   registration_id: "<OPTIONAL REGISTRATION ID. LEAVE COMMENTED OUT TO REGISTER WITH CN OF identity_cert>"
    identity_cert: "<REQUIRED URI TO DEVICE IDENTITY CERTIFICATE>"
    identity_pk: "<REQUIRED URI TO DEVICE IDENTITY PRIVATE KEY>"
```

Atestación de clave simétrica:

```yml
# DPS symmetric key provisioning configuration
provisioning:
  source: "dps"
  global_endpoint: "https://global.azure-devices-provisioning.net"
  scope_id: "<SCOPE_ID>"
  attestation:
    method: "symmetric_key"
    registration_id: "<REGISTRATION_ID>"
    symmetric_key: "<SYMMETRIC_KEY>"
```

Para pegar el contenido del portapapeles en Nano: `Shift+Right Click` o presione `Shift+Insert`.

Guarde y cierre el archivo. `CTRL + X`, `Y`, `Enter`

Después de especificar la información de aprovisionamiento en el archivo de configuración, reinicie el demonio:

```bash
sudo systemctl restart iotedge
```

## <a name="verify-successful-installation"></a>Comprobación de instalación correcta

Si uso los pasos de **configuración manual** de la sección anterior, el entorno de ejecución de IoT Edge debe haberse aprovisionado correctamente y estar en ejecución en el dispositivo. Si ha usado los pasos de **configuración automática**, deberá completar algunos pasos adicionales para que el entorno de ejecución pueda registrar el dispositivo con IoT Hub en su nombre. Para los pasos siguientes, consulte [Creación y aprovisionamiento de un dispositivo de IoT Edge con un TPM virtual en una máquina virtual Linux](how-to-auto-provision-simulated-device-linux.md#give-iot-edge-access-to-the-tpm).

Puede comprobar el estado del demonio de IoT Edge:

```bash
systemctl status iotedge
```

Examine los registros del demonio:

```bash
journalctl -u iotedge --no-pager --no-full
```

Ejecute la [herramienta para la solución de problemas](troubleshoot.md#run-the-check-command) para consultar los errores de configuración y redes más comunes:

```bash
sudo iotedge check
```

Hasta que implemente el primer módulo en IoT Edge en el dispositivo, el módulo del sistema **$edgeHub** no se implementará en el dispositivo. Como resultado, la comprobación automatizada devolverá un error para la comprobación de conectividad de `Edge Hub can bind to ports on host`. Este error se puede omitir a menos que se produzca después de implementar un módulo en el dispositivo.

Por último, enumere los módulos en ejecución:

```bash
sudo iotedge list
```

Después de una instalación de IoT Edge en el dispositivo, el único módulo que debería en ejecución es **edgeAgent**. Una vez creada la primera implementación, el otro módulo del sistema **$edgeHub** se iniciará también en el dispositivo. Para más información, consulte [Implementación de módulos IoT Edge](how-to-deploy-modules-portal.md).

## <a name="tips-and-troubleshooting"></a>Sugerencias y solución de problemas

Necesita privilegios elevados para ejecutar comandos `iotedge`. Después de instalar el entorno de ejecución, cierre la sesión en la máquina e iníciela de nuevo para actualizar sus permisos automáticamente. Hasta entonces, use **sudo** delante de los comandos `iotedge`.

En dispositivos con recursos limitados, se recomienda encarecidamente establecer la variable de entorno *OptimizeForPerformance* en *false*, tal como se indica en las instrucciones de la [guía para la solución de problemas](troubleshoot.md).

Si la red tiene un servidor proxy, siga los pasos descritos en [Configure your IoT Edge device to communicate through a proxy server](how-to-configure-proxy-support.md) (Configuración del dispositivo IoT Edge para comunicarse mediante un servidor proxy).

### <a name="verify-your-linux-kernel-for-moby-compatibility"></a>Verificar si el kernel de Linux es compatible con Moby

Muchos fabricantes de dispositivos incrustados distribuyen imágenes de dispositivo que contienen kernels de Linux personalizados sin las características necesarias para la compatibilidad del entorno de ejecución del contenedor. Si tiene problemas al instalar el entorno de ejecución del contenedor recomendado de Moby, quizá pueda solucionar los problemas de configuración del kernel de Linux mediante el script [check-config](https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh) del [repositorio Moby de GitHub](https://github.com/moby/moby) oficial. Ejecute los siguientes comandos en el dispositivo para comprobar la configuración del kernel:

   ```bash
   curl -sSL https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh -o check-config.sh
   chmod +x check-config.sh
   ./check-config.sh
   ```

Este comando proporciona una salida detallada que contiene el estado de las características del kernel que utiliza el entorno de ejecución de Moby. Debe asegurarse de que todos los elementos de `Generally Necessary` y `Network Drivers` están habilitados para garantizar que el kernel es totalmente compatible con el runtime de Moby.  Si ha identificado alguna característica que falta, puede habilitarla volviendo a generar el kernel a partir del origen y seleccionando los módulos asociados para incluirlos en el archivo .config de kernel adecuado.  Igualmente, si usa un generador de configuración de kernel como `defconfig` o `menuconfig`, busque y habilite las características correspondientes y vuelva a generar el kernel como corresponda.  Una vez que haya implementado el kernel recién modificado, vuelva a ejecutar el script check-config para comprobar que se han habilitado correctamente todas las características necesarias.

## <a name="install-runtime-using-release-assets"></a>Instalación de tiempo de ejecución mediante recursos de versión

Siga los pasos de esta sección si quiere instalar una versión específica del entorno de ejecución de Azure IoT Edge que no esté disponible mediante `apt-get install`. La lista de paquetes de Microsoft solo contiene un conjunto limitado de versiones recientes y sus subversiones, por lo que estos pasos son para cualquiera que quiera instalar una versión anterior o una versión candidata para lanzamiento.

Con los comandos curl, puede dirigirse a los archivos de componentes directamente desde el repositorio de GitHub de IoT Edge. Siga los pasos siguientes para instalar libiothsm y el demonio de seguridad de IoT Edge. Instale el motor de Moby y la CLI mediante los pasos de la sección [Instalación de un entorno de ejecución del contenedor](#install-a-container-runtime).

1. Vaya a [Versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases) y busque la versión de lanzamiento que desea tener como destino.

2. Expanda la sección **Recursos** para esa versión.

3. Cada versión debe tener archivos nuevos para el demonio de seguridad de IoT Edge y hsmlib. Use los comandos siguientes para actualizar esos componentes.

   1. Busque el archivo **libiothsm-std** que coincida con la arquitectura del dispositivo IoT Edge. Haga clic con el botón derecho en el vínculo del archivo y copie la dirección del vínculo.

   2. Use el vínculo copiado en el comando siguiente para instalar esa versión de hsmlib:

      ```bash
      curl -L <libiothsm-std link> -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb
      ```

   3. Busque el archivo **iotedge** que coincida con la arquitectura del dispositivo IoT Edge. Haga clic con el botón derecho en el vínculo del archivo y copie la dirección del vínculo.

   4. Use el vínculo copiado en el comando siguiente para instalar esa versión del demonio de seguridad de IoT Edge.

      ```bash
      curl -L <iotedge link> -o iotedge.deb && sudo dpkg -i ./iotedge.deb
      ```

Una vez que IoT Edge se ha instalado correctamente en `/etc/iotedge`, se le solicita que actualice el archivo de configuración. Siga los pasos descritos en la sección [Configuración del demonio de seguridad](#configure-the-security-daemon) para terminar de aprovisionar el dispositivo.

## <a name="uninstall-iot-edge"></a>Desinstalación de IoT Edge

Si quiere quitar la instalación de IoT Edge del dispositivo de Linux, use los siguientes comandos de la línea de comandos.

Quite el entorno de ejecución de Azure IoT Edge.

```bash
sudo apt-get remove --purge iotedge
```

Cuando se quita el entorno de ejecución de IoT Edge, los contenedores que se han creado se detienen, pero siguen existiendo en el dispositivo. Observe todos los contenedores para ver cuáles permanecen.

```bash
sudo docker ps -a
```

Elimine los contenedores del dispositivo, incluidos los dos contenedores del entorno en tiempo de ejecución.

```bash
sudo docker rm -f <container name>
```

Por último, quite el entorno en tiempo de ejecución del contenedor del dispositivo.

```bash
sudo apt-get remove --purge moby-cli
sudo apt-get remove --purge moby-engine
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un dispositivo IoT Edge aprovisionado con el entorno de ejecución instalado, puede [implementar módulos de IoT Edge](how-to-deploy-modules-portal.md).

Si tiene problemas con la instalación correcta del entorno de ejecución de Azure IoT Edge, consulte la página de [solución de problemas](troubleshoot.md).

Para actualizar una instalación existente a la versión más reciente de IoT Edge, vea [Actualice el archivo de configuración del demonio de seguridad y el entorno de ejecución de IoT Edge](how-to-update-iot-edge.md).
