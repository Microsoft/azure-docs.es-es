---
title: Actualización de firmware de dispositivos mediante Azure IoT Hub | Microsoft Docs
description: Aprenda a implementar un proceso de actualización de firmware de dispositivos que se pueda desencadenar desde una aplicación de back-end conectada al centro de IoT.
services: iot-hub
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 06/28/2019
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-javascript
ms.openlocfilehash: 2e2d66e113c855830f841761cb11a70e4f26c19d
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87415081"
---
# <a name="tutorial-implement-a-device-firmware-update-process"></a>Tutorial: Implementación de un proceso de actualización de firmware de dispositivos

Puede que tenga que actualizar el firmware de los dispositivos conectados a IoT Hub. Por ejemplo, puede que desee agregar nuevas características al firmware o aplicar revisiones de seguridad. En muchos escenarios de IoT, resulta poco práctico visitar físicamente y, después, aplicar manualmente las actualizaciones de firmware en los dispositivos. En este tutorial se muestra cómo puede iniciar y supervisar el proceso de actualización del firmware de forma remota mediante una aplicación back-end conectada al centro.

Para crear y supervisar el proceso de actualización del firmware, la aplicación back-end de este tutorial crea una _configuración_ en IoT Hub. La [administración automática de dispositivos](iot-hub-auto-device-config.md) de IoT Hub usa esta configuración para actualizar un conjunto de _propiedades deseadas de los dispositivos gemelos_ en todos los dispositivos de refrigerador. Las propiedades deseadas especifican los detalles de la actualización del firmware que se necesitan. Mientras se ejecuta en los dispositivos de refrigerador el proceso de actualización del firmware, estos notifican su estado a la aplicación back-end mediante las _propiedades notificadas de los dispositivos gemelos_. La aplicación de back-end puede usar la configuración para supervisar las propiedades notificadas enviadas desde el dispositivo y seguir el proceso de actualización del firmware hasta su finalización:

![Proceso de actualización del firmware](media/tutorial-firmware-update/Process.png)

En este tutorial, va a completar las siguientes tareas:

> [!div class="checklist"]
> * Creación de un centro de IoT e incorporación de un dispositivo de prueba en el registro de identidades del dispositivo.
> * Creación de una configuración para definir la actualización del firmware.
> * Simulación del proceso de actualización del firmware en un dispositivo.
> * Recepción de las actualizaciones de estado del dispositivo a medida que avanza la actualización del firmware.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Las dos aplicaciones de ejemplo que se ejecutan en este inicio rápido se escriben con Node.js. Necesitará Node.js v10.x.x o una versión posterior en la máquina de desarrollo.

Puede descargar Node.js para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

Descargue el proyecto de muestra de Node.js desde https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip y extraiga el archivo ZIP.

Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivos de ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

Para completar este tutorial, su suscripción de Azure debe tener un centro de IoT con un dispositivo agregado en el registro de identidades de dispositivo. La entrada en el registro de identidad del dispositivo permite que el dispositivo simulado que se ejecuta en este tutorial se conecte con el centro.

Si aún no tiene un centro de IoT configurado en su suscripción, puede configurar uno con el siguiente script de la CLI. Este script utiliza el nombre **tutorial-iot-hub** para el centro de IoT, cuando lo ejecute, tiene que reemplazar este nombre por otro que sea exclusivo. El script crea el grupo de recursos y el centro en la región **Centro de EE. UU.** , que usted puede cambiar a otra región más cercana. El script recupera la cadena de conexión de servicio del centro de IoT, el cual se utiliza en la aplicación de ejemplo de back-end para conectar con el centro de IoT:

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed
az extension add --name azure-iot

# Create a resource group
az group create --name tutorial-iot-hub-rg --location $location

# Create your free-tier IoT Hub. You can only have one free IoT Hub per subscription
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --sku F1

# Make a note of the service connection string, you need it later
az iot hub show-connection-string --name $hubname --policy-name service -o table

```

Este tutorial usa un dispositivo simulado llamado **MyFirmwareUpdateDevice**. El siguiente script agrega este dispositivo al registro de identidades de dispositivo, establece un valor de etiqueta y recupera su cadena de conexión:

```azurecli-interactive
# Set the name of your IoT hub
hubname=tutorial-iot-hub

# Create the device in the identity registry
az iot hub device-identity create --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Add a device type tag
az iot hub device-twin update --device-id MyFirmwareUpdateDevice --hub-name $hubname --set tags='{"devicetype":"chiller"}'

# Retrieve the device connection string, you need this later
az iot hub device-identity show-connection-string --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

> [!TIP]
> Si ejecuta estos comandos en un símbolo del sistema de Windows o de Powershell, consulte la página de [sugerencias de azure-iot-cli-extension](https://github.com/Azure/azure-iot-cli-extension/wiki/Tips) para más información acerca de cómo escribir cadenas JSON.

## <a name="start-the-firmware-update"></a>Inicio de la actualización del firmware

Puede crear una [configuración automática de administración de dispositivos](iot-hub-automatic-device-management.md#create-a-configuration) en la aplicación back-end para empezar el proceso de actualización del firmware en todos los dispositivos etiquetados con un **devicetype** de refrigerador. En esta sección verá como:

* Cree una configuración en una aplicación back-end.
* Supervise el trabajo hasta su finalización.

### <a name="use-desired-properties-to-start-the-firmware-upgrade-from-the-back-end-application"></a>Uso de propiedades deseadas para empezar la actualización del firmware desde la aplicación back-end

Para ver el código de la aplicación back-end que crea la configuración, vaya a la carpeta **iot-hub/Tutorials/FirmwareUpdate** del proyecto de ejemplo de Node.js que descargó. Luego abra el archivo ServiceClient.js. con un editor de texto.

La aplicación back-end crea la siguiente configuración:

[!code-javascript[Automatic device management configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=configuration "Automatic device management configuration")]

La configuración incluye las siguientes secciones:

* `content` especifica las propiedades deseadas del firmware que se envían a los dispositivos seleccionados.
* `metrics` especifica las consultas que se deben ejecutar para notificar el estado de la actualización del firmware.
* `targetCondition` selecciona los dispositivos que van a recibir la actualización del firmware.
* `priorty` establece la prioridad relativa de esta configuración frente a otras configuraciones.

La aplicación back-end utiliza el código siguiente para crear la configuración que establece las propiedades deseadas:

[!code-javascript[Create configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=createConfiguration "Create configuration")]

Después de crear la configuración, la aplicación supervisa la actualización del firmware:

[!code-javascript[Monitor firmware update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=monitorConfiguration "Monitor firmware update")]

La configuración notifica sobre dos tipos de métricas:

* Las métricas del sistema que notifican cuántos son los dispositivos de destino y en cuántos se ha aplicado la actualización.
* Métricas personalizadas que generan las consultas que agregue a la configuración. En este tutorial las consultas notifican cuántos dispositivos hay en cada etapa del proceso de actualización.

### <a name="respond-to-the-firmware-upgrade-request-on-the-device"></a>Respuesta a la solicitud de actualización del firmware en el dispositivo

Para ver el código del dispositivo simulado que controla las propiedades deseadas del firmware que se envían desde la aplicación back-end, vaya a la carpeta **iot-hub/Tutorials/FirmwareUpdate** del proyecto de ejemplo de Node.js que descargó. Luego abra el archivo SimulatedDevice.js. con un editor de texto.

La aplicación del dispositivo simulado crea un controlador para las actualizaciones de las propiedades deseadas de **properties.desired.firmware** en el dispositivo gemelo. El controlador lleva a cabo algunas comprobaciones básicas en las propiedades deseadas antes de iniciar el proceso de actualización:

[!code-javascript[Handle desired property update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=initiateUpdate "Handle desired property update")]

## <a name="update-the-firmware"></a>Actualización del firmware

La función **initiateFirmwareUpdateFlow** ejecuta la actualización. Esta función utiliza la función **waterfall** para ejecutar las fases del proceso de actualización en orden. En este ejemplo, la actualización del firmware tiene cuatro fases. La primera fase descarga la imagen, la segunda la comprueba mediante una suma de comprobación, la tercera aplica la imagen y la última fase reinicia el dispositivo:

[!code-javascript[Firmware update flow](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=firmwareupdateflow "Firmware update flow")]

Durante el proceso de actualización, los dispositivos simulados informan sobre el progreso mediante propiedades notificadas:

[!code-javascript[Reported properties](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=reportedProperties "Reported properties")]

En el siguiente fragmento de código se muestra la implementación de la fase de descarga. Durante la fase de descarga, el dispositivo simulado usa propiedades notificadas para enviar información de estado a la aplicación back-end:

[!code-javascript[Download image phase](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=downloadimagephase "Download image phase")]

## <a name="run-the-sample"></a>Ejecución del ejemplo

En esta sección, va a ejecutar dos aplicaciones de ejemplo para observar como una aplicación back-end crea la configuración para administrar el proceso de actualización del firmware en el dispositivo simulado.

Para ejecutar el dispositivo simulado y las aplicaciones back-end, necesita las cadenas de conexión de dispositivo y servicio. Cuando creó los recursos al principio de este tutorial, tomó nota de las cadenas de conexión.

Para ejecutar la aplicación de dispositivo simulado, abra una ventana del shell o del símbolo del sistema y vaya a la carpeta **iot-hub/Tutorials/FirmwareUpdate** del proyecto de Node.js que descargó. Luego, ejecute los siguientes comandos:

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Para ejecutar la aplicación back-end, abra otra ventana del shell o del símbolo del sistema. A continuación, vaya a la carpeta **iot-hub/Tutorials/FirmwareUpdate** del proyecto de Node.js que ha descargado. Luego, ejecute los siguientes comandos:

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

La siguiente captura de pantalla muestra la salida de la aplicación del dispositivo simulado y muestra cómo responde a la actualización de las propiedades deseadas del firmware desde la aplicación back-end:

![Dispositivo simulado](./media/tutorial-firmware-update/SimulatedDevice.png)

La siguiente captura de pantalla muestra la salida de la aplicación back-end y resalta cómo crea la configuración para actualizar las propiedades deseadas del firmware:

![Aplicación back-end](./media/tutorial-firmware-update/BackEnd1.png)

La siguiente captura de pantalla muestra la salida de la aplicación back-end y resalta cómo supervisa las métricas de actualización del firmware desde el dispositivo simulado:

![Aplicación back-end](./media/tutorial-firmware-update/BackEnd2.png)

Debido a que las configuraciones automáticas de los dispositivos se ejecutan en el momento de la creación y después cada cinco minutos, es posible que no vea todas las actualizaciones de estado enviadas a la aplicación de back-end. También puede ver las métricas en el portal en la sección **Administración automática de dispositivos -> Configuración de dispositivo IoT** del centro de IoT:

![Visualización de la configuración en el portal](./media/tutorial-firmware-update/portalview.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene pensado completar el siguiente tutorial, deje el centro de IoT y el grupo de recursos y vuelva a usarlos más adelante.

Si ya no los va a necesitar más, elimínelos en el portal. Para ello, seleccione el grupo de recursos **tutorial-iot-hub-rg** que contiene el centro de IoT y haga clic en **Eliminar**.

Como alternativa, use la CLI:

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a implementar un proceso de actualización de firmware para los dispositivos conectados. Continúe en el siguiente tutorial para aprender a usar herramientas del portal de Azure IoT Hub y comandos de la CLI de Azure para probar la conectividad de los dispositivos.

> [!div class="nextstepaction"]
> [Uso de un dispositivo simulado para probar la conectividad con IoT Hubs](tutorial-connectivity.md)
