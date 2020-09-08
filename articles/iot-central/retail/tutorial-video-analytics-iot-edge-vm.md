---
title: 'Tutorial: Creación de una instancia de IoT Edge para análisis de vídeo en Azure IoT Central (máquina virtual Linux)'
description: En este tutorial se muestra cómo crear una instancia de IoT Edge para análisis de vídeo con la plantilla de aplicación de análisis de vídeo con detección de objetos y movimiento.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/31/2020
ms.openlocfilehash: 69e5b757036a2d68fa779e3fc232cc42a034e33c
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88038086"
---
# <a name="tutorial-create-an-iot-edge-instance-for-video-analytics-linux-vm"></a>Tutorial: Creación de una instancia de IoT Edge para análisis de vídeo (máquina virtual Linux)

Azure IoT Edge es un servicio totalmente administrado que ofrece inteligencia en la nube de forma local mediante la implementación y ejecución de:

* Lógica personalizada
* Servicios de Azure
* Inteligencia artificial

En IoT Edge, estos servicios se ejecutan directamente en dispositivos IoT multiplataforma, lo que le permite ejecutar la solución IoT de forma segura y a escala en la nube o sin conexión.

En este tutorial se muestra cómo preparar un dispositivo IoT Edge en una máquina virtual de Azure. La instancia de IoT Edge ejecuta los módulos de análisis de vídeo en directo que usa la plantilla de aplicación de análisis de vídeo con detección de objetos y movimiento de Azure IoT Central.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear una máquina virtual de Azure con el entorno de ejecución de Azure IoT Edge instalado
> * Preparación de la instalación de IoT Edge para hospedar el módulo de análisis de vídeo en directo y conectarse a IoT Central

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, debe completar el tutorial anterior [Creación de una aplicación de análisis de vídeo en Azure IoT Central](./tutorial-video-analytics-create-app.md).

También necesita una suscripción de Azure. Si no tiene una suscripción a Azure, puede crear la de forma gratuita en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).

## <a name="deploy-azure-iot-edge"></a>Implementación de Azure IoT Edge

Para crear una máquina virtual de Azure con el entorno de ejecución de IoT Edge más reciente y los módulos de análisis de vídeo en directo instalados, seleccione el botón siguiente:

[![Botón Implementar en Azure para iotedge-VM-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Flive-video-analytics%2Fmaster%2Fref-apps%2Flva-edge-iot-central-gateway%2Fvm_deploy%2FedgeModuleVMDeploy.json)

Utilice la información de la tabla siguiente para completar el formulario de **Implementación personalizada**:

| Campo | Value |
| ----- | ----- |
| Suscripción | Seleccione su suscripción a Azure. |
| Resource group | *lva-rg*: grupo de recursos que ha creado en el tutorial anterior. |
| Region       | *Este de EE. UU.* |
| Prefijo de etiqueta DNS | Elija un prefijo DNS único para la máquina virtual. |
| Nombre de usuario administrador | *AzureUser* |
| Contraseña de administrador | Escriba una contraseña. Tome nota de la contraseña en el archivo *scratchpad.txt*, la utilizará más adelante. |
| El identificador de ámbito | **Identificador de ámbito** que anotó en el archivo *scratchpad.txt* en el tutorial anterior al agregar el dispositivo de puerta de enlace. |
| Id. de dispositivo | *lva-gateway-001*: dispositivo de puerta de enlace que creó en el tutorial anterior. |
| Clave de dispositivo | Clave principal del dispositivo que anotó en el archivo *scratchpad.txt* en el tutorial anterior al agregar el dispositivo de puerta de enlace. |
| Host de la aplicación de IoT Central | **Dirección URL de la aplicación** que anotó en el archivo *scratchpad.txt* en el tutorial anterior. Por ejemplo, *traders.azureiotcentral.com*. |
| Token de la API de la aplicación de IoT Central | Token de la API del operador que anotó en el tutorial anterior. |
| Clave de aprovisionamiento de dispositivos de IoT Central | Token de la firma de acceso compartido del grupo principal que anotó en el archivo *scratchpad.txt* en el tutorial anterior. |
| Tamaño de VM | *Standard_DS1_v2* |
| Versión del sistema operativo Ubuntu | *18.04-LTS* |
| Location | *[resourceGroup().location]* |

Seleccione **Revisar + crear**. Una vez completada la validación, seleccione **Create** (Crear). Normalmente, la implementación tarda unos tres minutos en completarse. Una vez finalizada la implementación, vaya al grupo de recursos **lva-rg** en Azure Portal.

## <a name="ensure-the-iot-edge-runtime-loads-the-modules"></a>Asegurarse de que el entorno de ejecución de IoT Edge carga los módulos

En Azure Portal, vaya al grupo de recursos **lva-rg** y seleccione la máquina virtual. A continuación, en la sección **Soporte técnico y solución de problemas**, seleccione **Consola serie**.

Presione **Entrar** para obtener un símbolo del sistema `login:`. Use *AzureUser* como nombre de usuario y la contraseña que eligió al crear la máquina virtual.

Ejecute el siguiente comando para comprobar la versión del entorno de ejecución de IoT Edge. En el momento de escribir este documento, la versión es la 1.0.9:

```bash
sudo iotedge --version
```

Enumere los módulos IoT Edge mediante el comando:

```bash
sudo iotedge list
```

La implementación ha configurado los siguientes cinco módulos IoT Edge para ejecutar:

* LvaEdgeGatewayModule
* edgeAgent
* edgeHub
* lvaEdge
* lvaYolov3

La implementación ha creado un entorno de IoT Edge personalizado con los módulos necesarios para el análisis de vídeo en directo. La implementación ha actualizado el archivo **config.yaml** predeterminado para asegurarse de que el entorno de ejecución de IoT Edge utiliza IoT Hub Device Provisioning Service para conectarse a IoT Central. La implementación también ha creado un archivo llamado **state.json** en la carpeta **/data/storage** para proporcionar a los módulos los datos de configuración adicionales. Para más información, consulte el tutorial [Creación de una instancia de IoT Edge para análisis de vídeo (Intel NUC)](./tutorial-video-analytics-iot-edge-nuc.md).

Para solucionar problemas del dispositivo IoT Edge, consulte [Solución de problemas del dispositivo IoT Edge](https://docs.microsoft.com/azure/iot-edge/troubleshoot).

## <a name="use-the-rtsp-simulator"></a>Uso del simulador RTSP

Si no tiene dispositivos de cámara reales para conectar al dispositivo IoT Edge, puede usar los dos dispositivos de cámara simulada de la plantilla de aplicación de análisis de vídeo. En esta sección se muestra cómo usar una secuencia de vídeo simulada en el dispositivo IoT Edge.

En estas instrucciones se usa [Live555 Media Server](http://www.live555.com/mediaServer/) como simulador RTSP en un contenedor de Docker.

> [!NOTE]
> Las referencias a software de terceros en este repositorio son solo para fines informativos y por comodidad. Microsoft no aprueba ni proporciona derechos para el software de terceros. Para más información, consulte [Live555 Media Server](http://www.live555.com/mediaServer/).

Use el comando siguiente para ejecutar la utilidad **rtspvideo** en un contenedor de Docker en la máquina virtual de IoT Edge. El contenedor de Docker crea una secuencia RTSP en segundo plano:

```bash
sudo docker run -d --name live555 --rm -p 554:554 mcr.microsoft.com/lva-utilities/rtspsim-live555:1.2
```

Use el comando siguiente para enumerar los contenedores de Docker:

```bash
sudo docker ps
```

La lista incluye un contenedor llamado **live555**.

## <a name="next-steps"></a>Pasos siguientes

Ahora ha implementado el entorno de ejecución de IoT Edge, los módulos de LVA y la secuencia simulada de Live555 en una máquina virtual Linux que se ejecuta en Azure.

Para administrar las cámaras, consulte el tutorial siguiente:

> [!div class="nextstepaction"]
> [Supervisión y administración de una aplicación de análisis de vídeo con detección de objetos y movimiento](./tutorial-video-analytics-manage.md)
