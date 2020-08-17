---
title: Conexión de Raspberry Pi a Azure IoT Hub en la nube (Node.js)
description: En este tutorial aprenderá a configurar y conectar Raspberry Pi a Azure IoT Hub para que envíe datos a la plataforma en la nube de Azure.
author: wesmc7777
manager: eliotgra
keywords: azure iot raspberry pi, raspberry pi iot hub, raspberry pi envía datos a la nube, raspberry pi a la nube
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: wesmc
ms.custom:
- 'Role: Cloud Development'
- devx-track-javascript
ms.openlocfilehash: e1df4387b5617e9aebb4045bdf19c8df628b56ef
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423667"
---
# <a name="connect-raspberry-pi-to-azure-iot-hub-nodejs"></a>Conectar Raspberry Pi a Azure IoT Hub (Node.js)

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

En este tutorial, empezará por aprender los principios básicos del uso de Raspberry Pi que ejecuta Raspbian. A continuación, aprenderá a conectar sin problemas los dispositivos en la nube con [Azure IoT Hub](about-iot-hub.md). Para obtener ejemplos de Windows 10 IoT Core, vaya al [Centro de desarrollo de Windows](https://www.windowsondevices.com/).

¿Aún no tiene un kit? Pruebe el [simulador en línea de Rapsberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md). También puede comprar un nuevo kit [aquí](https://azure.microsoft.com/develop/iot/starter-kits).

## <a name="what-you-do"></a>Qué debe hacer

* Cree un Centro de IoT.

* Registre un dispositivo para Pi en IoT Hub.

* Configure Raspberry Pi.

* Ejecute una aplicación de ejemplo en Pi para enviar datos de sensor a IoT Hub.

## <a name="what-you-learn"></a>Conocimientos que adquirirá

* Cómo crear Azure IoT Hub y obtener la cadena de conexión del nuevo dispositivo.

* Cómo conectar Pi con un sensor BME280.

* Cómo recopilar datos del sensor al ejecutar una aplicación de ejemplo en Pi.

* Cómo enviar los datos del sensor a IoT Hub.

## <a name="what-you-need"></a>Lo que necesita

![Lo que necesita](./media/iot-hub-raspberry-pi-kit-node-get-started/0-starter-kit.png)

* Una placa de Raspberry Pi 2 o Raspberry Pi 3.

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* Un monitor, un teclado USB y un mouse que se conecten a Pi.

* Un equipo PC o Mac con Windows o Linux.

* Una conexión a Internet.

* Una tarjeta microSD de 16 GB o más.

* Un adaptador de USB a SD o una tarjeta microSD para grabar la imagen del sistema operativo en la tarjeta microSD.

* Una fuente de alimentación de 5 V y 2 A con un cable microUSB de 6 pies.

Los elementos siguientes son opcionales:

* Un sensor de temperatura, presión y humedad Adafruit BME280 ensamblado.

* La placa de pruebas.

* Cables de puente M/6 F.

* Un LED difuso de 10 mm.

> [!NOTE]
> Si no tiene los elementos opcionales, puede usar datos del sensor simulados.

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="set-up-raspberry-pi"></a>Configuración de Raspberry Pi

### <a name="install-the-raspbian-operating-system-for-pi"></a>Instalar el sistema operativo Raspbian para Pi

Prepare la tarjeta microSD para instalar la imagen de Raspbian.

1. Descargue Raspbian.

   a. [Raspbian Buster con escritorio](https://www.raspberrypi.org/downloads/raspbian/) (el archivo .zip).

   b. Extraiga la imagen de Raspbian en una carpeta del equipo.

2. Instale Raspbian en la tarjeta microSD.

   a. [Descargue e instale la utilidad de grabadora de tarjetas SD Etcher](https://etcher.io/).

   b. Ejecute Etcher y seleccione la imagen de Raspbian que extrajo en el paso 1.

   c. Seleccione la unidad de la tarjeta microSD. Puede que Etcher ya haya seleccionado la unidad correcta.

   d. Haga clic en Flash para instalar Raspbian en la tarjeta microSD.

   e. Quite la tarjeta microSD del equipo cuando se complete la instalación. Es seguro quitar la tarjeta microSD directamente porque Etcher expulsa o desmonta la tarjeta microSD automáticamente al acabar.

   f. Inserte la tarjeta microSD en la Pi.

### <a name="enable-ssh-and-i2c"></a>Habilitar SSH e I2C

1. Conecte Pi al monitor, el teclado y el mouse.

2. Inicie Pi y luego inicie sesión en Raspbian con `pi` como nombre de usuario y `raspberry` como contraseña.

3. Haga clic en el icono de Raspberry > **Preferencias** > **Configuración de Raspberry Pi**.

   ![Menú Preferencias de Raspbian](./media/iot-hub-raspberry-pi-kit-node-get-started/1-raspbian-preferences-menu.png)

4. En la pestaña **Interfaces**, establezca **I2C** y **SSH** en **Habilitar** y luego haga clic en **Aceptar**. Si no tiene sensores físicos y desea usar datos de detección simulados, este paso es opcional.

   ![Habilitar I2C y SSH en Raspberry Pi](./media/iot-hub-raspberry-pi-kit-node-get-started/2-enable-i2c-ssh-on-raspberry-pi.png)

> [!NOTE]
> Para habilitar SSH e I2C, puede buscar más documentos de referencia en [raspberrypi.org](https://www.raspberrypi.org/documentation/remote-access/ssh/) y [Adafruit.com](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup/configuring-i2c).

### <a name="connect-the-sensor-to-pi"></a>Conectar el sensor a Pi

Use la placa de pruebas y los cables de puente para conectar un LED y un BME280 a Pi como se indica a continuación. Si no tiene el sensor, [omita esta sección](#connect-pi-to-the-network).

![Conexión de Raspberry Pi y el sensor](./media/iot-hub-raspberry-pi-kit-node-get-started/3-raspberry-pi-sensor-connection.png)

El sensor BME280 recopila datos sobre la temperatura y la humedad, El LED parpadea cuando el dispositivo envía un mensaje a la nube.

En las patillas del sensor, use el siguiente cableado:

| Inicio (sensor y LED)     | Fin (placa)            | Color de cable   |
| -----------------------  | ---------------------- | ------------: |
| VDD (patilla 5G)             | Potencia 3,3 V (patilla 1)       | Cable blanco   |
| GND (patilla 7G)             | GND (patilla 6)            | Cable marrón   |
| SDI (patilla 10G)            | I2C1 SDA (patilla 3)       | Cable rojo     |
| SCK (patilla 8G)             | I2C1 SCL (patilla 5)       | Cable naranja  |
| LED VDD (patilla 18F)        | GPIO 24 (patilla 18)       | Cable blanco   |
| LED GND (patilla 17F)        | GND (patilla 20)           | Cable negro   |

Haga clic para ver [Raspberry Pi 2 & 3 Pin mappings](/windows/iot-core/learn-about-hardware/pinmappings/pinmappingsrpi) (Asignaciones de patillas de Raspberry Pi 2 y 3) para su referencia.

Una vez que haya conectado correctamente BME280 a Raspberry Pi, su aspecto debería ser el de la imagen siguiente.

![Pi y BME280 conectados](./media/iot-hub-raspberry-pi-kit-node-get-started/4-connected-pi.png)

### <a name="connect-pi-to-the-network"></a>Conexión de Pi a la red

Encienda la Pi mediante un cable microUSB y la fuente de alimentación. Use el cable Ethernet para conectar Pi a la red cableada o siga las [instrucciones de Raspberry Pi Foundation](https://www.raspberrypi.org/documentation/configuration/wireless/) para conectar Pi a la red inalámbrica. Después de que Pi se ha conectado correctamente a la red, debe anotar la [dirección IP de su Pi](https://www.raspberrypi.org/documentation/remote-access/ip-address.md).

![Conectado a la red cableada](./media/iot-hub-raspberry-pi-kit-node-get-started/5-power-on-pi.png)

> [!NOTE]
> Asegúrese de que la Pi se conecta a la misma red que el equipo. Por ejemplo, si el equipo está conectado a una red inalámbrica mientras Pi está conectada a una red cableada, es posible que no vea la dirección IP en la salida de devdisco.

## <a name="run-a-sample-application-on-pi"></a>Ejecutar una aplicación de ejemplo en Pi

### <a name="clone-sample-application-and-install-the-prerequisite-packages"></a>Clonar una aplicación de ejemplo e instalar los paquetes de requisitos previos

1. Conéctese a Raspberry Pi con uno de los siguientes clientes SSH del equipo host:

   **Usuarios de Windows**

   a. Descargue e instale [PuTTY](https://www.putty.org/) para Windows.

   b. Copie la dirección IP de Pi en la sección de nombre de host (o dirección IP) y seleccione SSH como el tipo de conexión.

   ![PuTTy](./media/iot-hub-raspberry-pi-kit-node-get-started/7-putty-windows.png)

   **Usuarios de Mac y Ubuntu**

   Use el cliente de SSH integrado en Ubuntu o macOS. Quizás deba ejecutar `ssh pi@<ip address of pi>` para conectar Pi mediante SSH.

   > [!NOTE]
   > El nombre de usuario predeterminado es `pi` y la contraseña es `raspberry`.

2. Instale Node.js y NPM en Pi.

   En primer lugar, compruebe la versión de Node.js.

   ```bash
   node -v
   ```

   Si la versión es anterior a la 10.x, o bien si no hay ninguna versión de Node.js en Pi, instale la versión más reciente.

   ```bash
   curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash
   sudo apt-get -y install nodejs
   ```

3. Clone la aplicación de ejemplo.

   ```bash
   git clone https://github.com/Azure-Samples/azure-iot-samples-node.git
   ```

4. Instale todos los paquetes para el ejemplo. La instalación incluye el SDK de dispositivo IoT de Azure, la biblioteca del sensor BME280 y la biblioteca de cableado de Pi.

   ```bash
   cd azure-iot-samples-node/iot-hub/Tutorials/RaspberryPiApp
   npm install
   ```

   > [!NOTE]
   >Este proceso de instalación puede llevar varios minutos en función de la conexión de red.

### <a name="configure-the-sample-application"></a>Configurar la aplicación de ejemplo

1. Abra el archivo config mediante la ejecución de los comandos siguientes:

   ```bash
   nano config.json
   ```

   ![Archivo config](./media/iot-hub-raspberry-pi-kit-node-get-started/6-config-file.png)

   Hay dos elementos en este archivo que se pueden configurar. El primero es `interval`, que define el intervalo de tiempo (en milisegundos) entre los mensajes que se envían a la nube. El segundo es `simulatedData`, un valor booleano que indica si se usan los datos de sensor simulados o no.

   Si **no tiene el sensor**, establezca el valor `simulatedData` en `true` para que la aplicación de ejemplo cree y use datos de sensor simulados.

   *Nota: La dirección de I2C usada en este tutorial es 0x77 de forma predeterminada. En función de su configuración, también podría ser 0x76: si encuentra un error de I2C, intente cambiar el valor a 118 y compruebe si se resuelve. Para ver qué dirección usa el sensor, ejecute `sudo i2cdetect -y 1` en un shell de Raspberry Pi*.

2. Guarde y salga al presionar Control-O > Entrar > Control-X.

### <a name="run-the-sample-application"></a>Ejecutar la aplicación de ejemplo

Ejecute la aplicación de ejemplo mediante el comando siguiente:

   ```bash
   sudo node index.js '<YOUR AZURE IOT HUB DEVICE CONNECTION STRING>'
   ```

   > [!NOTE]
   > Asegúrese de que copia y pega la cadena de conexión del dispositivo entre las comillas simples.

Debería ver el resultado siguiente, que muestra los datos del sensor y los mensajes que se envían a IoT Hub.

![Resultado: datos de sensor enviados desde Raspberry Pi a IoT Hub](./media/iot-hub-raspberry-pi-kit-node-get-started/8-run-output.png)

## <a name="read-the-messages-received-by-your-hub"></a>Lectura de los mensajes recibidos por IoT Hub

Una forma de supervisar los mensajes recibidos por la instancia de IoT Hub desde el dispositivo consiste en usar Azure IoT Tools para Visual Studio Code. Para más información, vea [Uso de Azure IoT Tools para Visual Studio Code a fin de enviar y recibir mensajes entre el dispositivo e IoT Hub](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

Para obtener más formas de procesar los datos enviados por el dispositivo, continúe con la sección siguiente.

## <a name="next-steps"></a>Pasos siguientes

Ha ejecutado una aplicación de ejemplo para recopilar datos de sensor y enviarlos a IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
