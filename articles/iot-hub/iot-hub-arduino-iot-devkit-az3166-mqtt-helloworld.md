---
title: Enviar mensajes a un servidor de MQTT mediante la biblioteca de cliente de Azure MQTT
description: Aprenda a usar la biblioteca cliente de MQTT para enviar mensajes a un agente MQTT. También aprenderá a configurar el mXChip IoT DevKit para que sea un cliente MQTT.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/02/2018
ms.author: liydu
ms.custom: mqtt
ms.openlocfilehash: a3a8c065c226d1cc94d79b44d16f288fab205a56
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337864"
---
# <a name="send-messages-to-an-mqtt-server"></a>Envío de mensajes a un servidor MQTT

Los sistemas de Internet de las cosas (IoT) se ven afectados a menudo por conexiones a Internet intermitentes, lentas o de baja calidad. MQTT es un protocolo de conectividad de equipo a equipo (M2M) desarrollado teniendo en cuenta estos desafíos. 

La biblioteca cliente de MQTT utilizada aquí forma parte del proyecto [Eclipse Paho](https://www.eclipse.org/paho/), que proporciona las API para usar MQTT en varios medios de transporte.

## <a name="what-you-learn"></a>Conocimientos que adquirirá

En este proyecto, aprenderá a:
- Usar la biblioteca cliente de MQTT para enviar mensajes a un agente MQTT.
- Configurar su MXChip Iot DevKit como un cliente MQTT.

## <a name="what-you-need"></a>Lo que necesita

Siga la [Guía de introducción](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started) para:

* Conexión de DevKit con Wi-Fi
* Preparación del entorno de desarrollo

## <a name="open-the-project-folder"></a>Apertura de la carpeta del proyecto

1. Si DevKit ya está conectado a su equipo, desconéctelo.

2. Inicie VS Code.

3. Conecte el DevKit al equipo.

## <a name="open-the-mqttclient-sample"></a>Apertura del ejemplo de MQTTClient

Expanda la sección de la izquierda **EJEMPLOS DE ARDUINO**, vaya a **Ejemplos para MXCHIP AZ3166 > MQTT** y seleccione **MQTTClient**. Se abre una nueva ventana de VS Code que contiene la carpeta del proyecto.

> [!NOTE]
> También puede abrir el ejemplo desde la paleta de comandos. Utilice `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`) para abrir la paleta de comandos, escriba **Arduino** y, a continuación, busque y seleccione **Arduino: ejemplos** .

## <a name="build-and-upload-the-arduino-sketch-to-the-devkit"></a>Creación y carga del boceto de Arduino en DevKit

Escriba `Ctrl+P` (macOS: `Cmd+P`) para ejecutar `task device-upload`. Una vez completada la carga, DevKit se reinicia y ejecuta el boceto.

![carga-dispositivo](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/device-upload.jpg)

> [!NOTE]
> Puede aparecer el mensaje de error "Error: AZ3166: paquete desconocido". Este error se produce cuando el índice del paquete de la placa no se actualiza correctamente. Para resolver este error, consulte la [sección de desarrollo de las preguntas frecuentes de IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#development).

## <a name="test-the-project"></a>Prueba del proyecto

En VS Code, siga este procedimiento para abrir y configurar el monitor serie:

1. Haga clic en la palabra `COM[X]` de la barra de estado para establecer el puerto COM correcto con `STMicroelectronics`: ![establecer-puerto-com](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/set-com-port.jpg)

2. Haga clic en el icono de interruptor de alimentación de la barra de estado para abrir el monitor serie: ![monitor-serie](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/serial-monitor.jpg)
  
3. En la barra de estado, haga clic en el número que representa la velocidad en baudios y configúrelo en `115200`: ![establecer-velocidad-baudios](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/set-baud-rate.jpg)

El monitor serie muestra todos los mensajes enviados por el boceto de ejemplo. El boceto conecta el DevKit a la red Wi-Fi. Una vez establecida correctamente la conexión Wi-Fi, el boceto envía un mensaje al agente MQTT. Después, el ejemplo envía repetidamente dos mensajes "iot.eclipse.org" mediante QoS 0 y QoS 1, respectivamente.

![salida-serie](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/serial-output.jpg)

## <a name="problems-and-feedback"></a>Problemas y comentarios

Si tiene algún problema, consulte las [preguntas más frecuentes de DevKit de IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) o póngase en contacto con nosotros mediante los siguientes canales:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="see-also"></a>Consulte también

* [Conexión de IoT DevKit AZ3166 a Azure IoT Hub en la nube](iot-hub-arduino-iot-devkit-az3166-get-started.md)
* [Agite para un Tweet](iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message.md)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a configurar su MXChip Iot DevKit como un cliente MQTT y a usar la biblioteca cliente de MQTT para enviar mensajes a un agente MQTT, este es el siguiente paso recomendado: [Información general sobre el acelerador de la solución de supervisión remota de Azure IoT](https://docs.microsoft.com/azure/iot-suite/)
