---
title: Desarrollo de dispositivos para Azure IoT Central | Microsoft Docs
description: Azure IoT Central es una plataforma de aplicaciones de IoT que simplifica la creación de soluciones de IoT. En este artículo se proporciona una introducción al desarrollo de dispositivos para conectarse a la aplicación IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 05/05/2020
ms.topic: overview
ms.service: iot-central
services: iot-central
ms.custom:
- mvc
- device-developer
ms.openlocfilehash: 6fabd7d8cf5c19f05bd31c2d0b12863fd6e25382
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90017530"
---
# <a name="iot-central-device-development-overview"></a>Introducción al desarrollo de dispositivos IoT Central

*Este artículo se aplica a los desarrolladores de dispositivos.*

Una aplicación IoT Central le permite supervisar y administrar millones de dispositivos a lo largo de su ciclo de vida. Esta introducción está dirigida a los desarrolladores de dispositivos que implementan código para ejecutarlo en dispositivos que se conectan a IoT Central.

Los dispositivos interactúan con una aplicación IoT Central con los siguientes primitivos:

- _Telemetría_: son datos que un dispositivo envía a IoT Central. Por ejemplo, una secuencia de valores de temperatura de un sensor incorporado.
- _Propiedades_: son valores de estado que un dispositivo notifica a IoT Central. Por ejemplo, la versión actual del firmware del dispositivo. También puede tener propiedades de escritura que IoT Central puede actualizar en el dispositivo.
- _Comandos_: se llaman desde IoT Central para controlar el comportamiento de un dispositivo. Por ejemplo, la aplicación IoT Central podría llamar a un comando para reiniciar un dispositivo.

Un generador de soluciones es responsable de configurar paneles y vistas en la interfaz de usuario web de IoT Central para visualizar la telemetría, administrar las propiedades y llamar a los comandos.

## <a name="types-of-device"></a>Tipos de dispositivos

En las secciones siguientes se describen los principales tipos de dispositivos que se pueden conectar a una aplicación IoT Central:

### <a name="standalone-device"></a>Dispositivo independiente

Un dispositivo independiente se conecta directamente a IoT Central. Normalmente, un dispositivo independiente envía telemetría desde sus sensores incorporados o conectados a la aplicación IoT Central. Los dispositivos independientes también pueden notificar valores de propiedad, recibir valores de propiedad de escritura y responder a los comandos.

### <a name="gateway-device"></a>Dispositivo de puerta de enlace

Un dispositivo de puerta de enlace administra uno o varios dispositivos de nivel inferior que se conectan a la aplicación IoT Central. IoT Central se usa para configurar las relaciones entre los dispositivos de nivel inferior y el dispositivo de puerta de enlace. Para más información, consulte [Definición de un nuevo tipo de dispositivo de puerta de enlace de IoT en la aplicación de Azure IoT Central](./tutorial-define-gateway-device-type.md).

### <a name="edge-device"></a>Dispositivo perimetral

Un dispositivo perimetral se conecta directamente a IoT Central, pero actúa como intermediario para otros dispositivos conocidos como _dispositivos hoja_. Un dispositivo perimetral normalmente se encuentra cerca de los dispositivos hoja para los que actúa como intermediario. Algunos de los escenarios en los que se usan dispositivos perimetrales son:

- Habilitación de dispositivos que no se pueden conectar directamente a IoT Central y lo hacen a través del dispositivo perimetral. Por ejemplo, un dispositivo hoja podría usar Bluetooth para conectarse al dispositivo perimetral, que luego se conecta a IoT Central a través de Internet.
- Adición de telemetría antes de que se envíe a IoT Central. Este enfoque puede ayudar a reducir los costos de enviar datos a IoT Central.
- Control local de los dispositivos hoja para evitar la latencia asociada con la conexión a IoT Central a través de Internet.

Un dispositivo perimetral también puede enviar su propia telemetría, notificar sus propiedades y responder a los comandos y las actualizaciones de propiedades de escritura.

IoT Central solo ve el dispositivo perimetral, no los dispositivos hoja conectados a él.

Para más información, consulte [Incorporación de un dispositivo Azure IoT Edge a la aplicación Azure IoT Central](./tutorial-add-edge-as-leaf-device.md).

## <a name="connect-a-device"></a>Conexión de un dispositivo

Azure IoT Central usa Azure [IoT Device Provisioning Service (DPS)](../../iot-dps/about-iot-dps.md) para administrar el registro y la conexión de todos los dispositivos.

El uso de DPS permite:

- Que IoT Central admite la incorporación y la conexión de dispositivos a escala.
- Que usted pueda generar credenciales de dispositivo y configurar los dispositivos sin conexión sin tener que registrarlos primero en la interfaz de usuario de IoT Central.
- Que usted pueda usar identificadores de dispositivo propios para registrar estos en IoT Central. Esto simplifica la integración con sistemas del área de operaciones existentes.
- Una manera sencilla y coherente de conectar dispositivos a IoT Central.

Para más información, consulte [Conexión a Azure IoT Central](./concepts-get-connected.md).

### <a name="security"></a>Seguridad

La conexión entre un dispositivo y la aplicación IoT Central está protegida mediante [firmas de acceso compartido](./concepts-get-connected.md#connect-devices-at-scale-using-sas) o [certificados X.509](./concepts-get-connected.md#connect-devices-using-x509-certificates) estándar del sector.

### <a name="communication-protocols"></a>Protocolos de comunicación

Los protocolos de comunicación que un dispositivo puede usar para conectarse a IoT Central incluyen MQTT, AMQP y HTTPS. A nivel interno, IoT Central usa una instancia de IoT Hub para permitir la conectividad del dispositivo. Para más información sobre los protocolos de comunicación que admite IoT Hub para la conectividad de los dispositivos, consulte [Selección de un protocolo de comunicación](../../iot-hub/iot-hub-devguide-protocols.md).

## <a name="implement-the-device"></a>Implementación del dispositivo

Use uno de los [SDK de dispositivo Azure IoT](#languages-and-sdks) para implementar el comportamiento del dispositivo. El código debe:

- Registrar el dispositivo en DPS y usar la información de DPS para conectarse al centro de IoT Hub interno en la aplicación IoT Central.
- Enviar telemetría en el formato que especifica la plantilla de dispositivo en IoT Central. IoT Central usa la plantilla de dispositivo para determinar cómo utilizar la telemetría para las visualizaciones y el análisis.
- Sincronizar los valores de propiedad entre el dispositivo e IoT Central. La plantilla de dispositivo especifica los nombres de propiedad y los tipos de datos de modo que IoT Central pueda mostrar la información.
- Implementar controladores de comandos para los comandos especificados en la plantilla de dispositivo. La plantilla de dispositivo especifica los nombres de comandos y los parámetros que el dispositivo debe usar.

Para obtener más información sobre el rol de las plantillas de dispositivo, consulte [¿Qué son las plantillas de dispositivo?](./concepts-device-templates.md)

Para obtener el código de ejemplo, consulte [Creación y conexión de una aplicación cliente de Node.js](./tutorial-connect-device-nodejs.md) o [Creación y conexión de una aplicación cliente de Python](./tutorial-connect-device-python.md).

### <a name="languages-and-sdks"></a>Lenguajes y SDK

Para más información sobre los lenguajes y SDK admitidos, consulte [Uso de los SDK de dispositivos de Azure IoT Hub](../../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks).

## <a name="next-steps"></a>Pasos siguientes

Si es un desarrollador de dispositivos y desea profundizar en algún código, el paso siguiente que se sugiere se indica en [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central](./tutorial-connect-device-nodejs.md).

Para más información sobre el uso de IoT Central, los siguientes pasos sugeridos son probar los inicios rápidos, empezando por [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md).
