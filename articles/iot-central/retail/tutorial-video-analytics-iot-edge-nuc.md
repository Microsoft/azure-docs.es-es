---
title: 'Tutorial: Creación de una instancia de IoT Edge para análisis de vídeo en Azure IoT Central (Intel NUC)'
description: En este tutorial se muestra cómo crear una instancia de IoT Edge para análisis de vídeo con la plantilla de aplicación de análisis de vídeo con detección de objetos y movimiento.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/27/2020
ms.openlocfilehash: 4ecce689e287673a3b08f8f90f87c28e021106d6
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88038061"
---
# <a name="tutorial-create-an-iot-edge-instance-for-video-analytics-intel-nuc"></a>Tutorial: Creación de una instancia de IoT Edge para análisis de vídeo (Intel NUC)

Azure IoT Edge es un servicio totalmente administrado que ofrece inteligencia en la nube de forma local mediante la implementación y ejecución de:

* Lógica personalizada
* Servicios de Azure
* Inteligencia artificial

En IoT Edge, estos servicios se ejecutan directamente en dispositivos IoT multiplataforma, lo que le permite ejecutar la solución IoT de forma segura y a escala en la nube o sin conexión.

En este tutorial se muestra cómo instalar y configurar el entorno de ejecución de IoT Edge en un dispositivo NUC de Intel.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Actualizar y configurar IoT Edge
> * Configurar la puerta de enlace de IoT Edge
> * Conectar una cámara local compatible con ONVIF al dispositivo NUC de Intel

## <a name="prerequisites"></a>Requisitos previos

* Antes de empezar, debe completar el tutorial anterior [Creación de una aplicación de análisis de vídeo en directo en Azure IoT Central](./tutorial-video-analytics-create-app.md).
* Un dispositivo, como un Intel NUC, que ejecute Linux, que pueda ejecutar contenedores de Docker y que tenga suficiente capacidad de procesamiento para ejecutar análisis de vídeo.
* El [entorno de ejecución de IoT Edge instalado](../../iot-edge/how-to-install-iot-edge-linux.md) y en ejecución en el dispositivo.
* Poder conectarse al dispositivo IoT Edge desde la máquina Windows, para ello se necesita el [cliente SSH de PuTTy](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) o una utilidad equivalente.
* También necesita una suscripción de Azure. Si no tiene una suscripción a Azure, puede crear la de forma gratuita en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).

## <a name="configure-the-iot-edge-device"></a>Configuración del dispositivo de IoT Edge

Si no tiene el entorno de ejecución de IoT Edge instalado en la máquina Intel NUC, consulte las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en sistemas Linux basados en Debian](../../iot-edge/how-to-install-iot-edge-linux.md).

Para actualizar el entorno de ejecución de IoT Edge:

1. Use la utilidad PuTTy para conectarse al dispositivo IoT Edge.

1. Ejecute los siguientes comandos para actualizar y comprobar la versión del entorno de ejecución de IoT Edge. En el momento de escribir este documento, la versión es la 1.0.9:

    ```bash
    sudo apt-get update
    sudo apt-get install libiothsm iotedge
    sudo iotedge --version
    ```

1. Use los comandos siguientes para crear las carpetas que utiliza la implementación con los permisos necesarios:

    ```bash
    sudo mkdir -p /data/storage
    sudo mkdir -p /data/media
    sudo chmod -R 777 /data
    ```

Para agregar el archivo de configuración *state.json* a la carpeta */data/storage* en el dispositivo IoT Edge:

1. Use un editor de texto para abrir el archivo *state.json* de la carpeta *lva-configuration* del equipo local.

1. Actualice los marcadores de posición `system` e `iotCentral > properties` con los valores de cadena que describen el dispositivo de puerta de enlace. Puede ver estos valores más adelante en el panel de la aplicación de IoT Central.

1. Actualice los marcadores de posición `iotCentral > appKeys` con los valores que anotó en el archivo *scratchpad.txt* en el tutorial anterior. Guarde los cambios.

1. Use la utilidad `scp` de PuTTy en un símbolo del sistema para copiar el archivo *state.json* que acaba de editar en la carpeta */data/storage* del dispositivo IoT Edge. En este ejemplo se usa `192.168.0.144` como dirección IP de ejemplo. Reemplácela por la dirección IP del dispositivo IoT Edge:

    ```cmd
    scp state.json YourUserName@192.168.0.144:/data/storage/state.json`
    ```

Configure IoT Edge para registrarse y conectarse a la aplicación de IoT Central:

1. Use la utilidad PuTTy para conectarse al dispositivo IoT Edge.

1. Use un editor de texto, como `nano`, para abrir el archivo config.yaml de IoT Edge.

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

    > [!WARNING]
    > Los archivos YAML no pueden usar tabuladores para la sangría. En su lugar, use dos espacios. Los elementos de nivel superior no pueden tener espacios en blanco iniciales.

1. Desplácese hacia abajo hasta que vea `# Manual provisioning configuration`. Convierta en comentario las tres líneas siguientes, tal como se muestra en el siguiente fragmento de código:

    ```yaml
    # Manual provisioning configuration
    #provisioning:
    #  source: "manual"
    #  device_connection_string: "temp"
    ```

1. Desplácese hacia abajo hasta que vea `# DPS symmetric key provisioning configuration`. Quite las marcas de comentario de las siguientes ocho líneas, como se muestra en el siguiente fragmento de código:

    ```yaml
    # DPS symmetric key provisioning configuration
    provisioning:
      source: "dps"
      global_endpoint: "https://global.azure-devices-provisioning.net"
      scope_id: "{scope_id}"
      attestation:
        method: "symmetric_key"
        registration_id: "{registration_id}"
        symmetric_key: "{symmetric_key}"
    ```

1. Reemplace `{scope_id}` por el **ámbito de identificador** que anotó en el archivo *scratchpad.txt* en el tutorial anterior.

1. Reemplace `{registration_id}` por *lva-gateway-001*, el dispositivo que creó en el tutorial anterior.

1. Reemplace `{symmetric_key}` por la **Clave principal** del dispositivo **lva-gateway-001** que anotó en el archivo *scratchpad.txt* en el tutorial anterior.

1. Ejecute el siguiente comando para reiniciar el demonio de IoT Edge:

    ```bash
    sudo systemctl restart iotedge
    ```

1. Ejecute el siguiente comando para comprobar el estado de los módulos de IoT Edge:

    ```bash
    iotedge list
    ```

    La salida del comando anterior muestra cinco módulos en ejecución. También puede ver el estado de los módulos en ejecución en la aplicación de IoT Central.

    > [!TIP]
    > Puede volver a ejecutar este comando para comprobar el estado. Es posible que tenga que esperar a que todos los módulos empiecen a ejecutarse.

Si los módulos IoT Edge no se inician correctamente, consulte [Solución de problemas del dispositivo IoT Edge](../../iot-edge/troubleshoot.md).

## <a name="collect-the-rstp-stream-from-your-camera"></a>Recopilación de la secuencia RSTP desde la cámara

Identifique las direcciones URL de la secuencia RTSP de las cámaras conectadas al dispositivo IoT Edge, por ejemplo:

`rtsp://192.168.1.64:554/Streaming/Channels/101/`

> [!TIP]
> Intente ver la secuencia de la cámara en el equipo de IoT Edge con un reproductor multimedia, como VLC.

## <a name="next-steps"></a>Pasos siguientes

Ahora ha implementado el entorno de ejecución de IoT Edge y los módulos LVA en el dispositivo de puerta de enlace con Intel NUC.

Para administrar las cámaras, consulte el tutorial siguiente:

> [!div class="nextstepaction"]
> [Supervisión y administración de una aplicación de análisis de vídeo con detección de objetos y movimiento](./tutorial-video-analytics-manage.md)
