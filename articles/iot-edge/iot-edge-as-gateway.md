---
title: 'Puertas de enlace para dispositivos de bajada: Azure IoT Edge | Microsoft Docs'
description: Use Azure IoT Edge para crear un dispositivo de puerta de enlace transparente, opaco o proxy que envíe datos desde varios dispositivos de bajada a la nube y los procese localmente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/21/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 0589779de2ddb0bc75dde3b57d6444634b879f86
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89017029"
---
# <a name="how-an-iot-edge-device-can-be-used-as-a-gateway"></a>Uso de un dispositivo IoT Edge como puerta de enlace

Las puertas de enlace en las soluciones de IoT Edge proporcionan conectividad de dispositivo y análisis perimetral a los dispositivos de IoT que, de lo contrario, no tendrían esas funcionalidades. Azure IoT Edge se puede usar para satisfacer las necesidades de una puerta de enlace de IoT, tanto si está relacionada con la conectividad, la identidad o el análisis perimetral. Los patrones de puerta de enlace de este artículo únicamente hacen referencia a las características de la conectividad de los dispositivos de nivel inferior y la identidad de los dispositivos, y no a cómo los datos del dispositivo se procesan en la puerta de enlace.

## <a name="patterns"></a>Patrones

Existen tres patrones para usar un dispositivo IoT Edge como puerta de enlace: transparente, traducción de protocolo y traducción de identidad.

Una diferencia clave entre los patrones es que una puerta de enlace transparente pasa mensajes entre dispositivos de bajada y IoT Hub sin necesidad de procesamiento adicional. Sin embargo, la traducción de protocolo y la traducción de identidad requieren procesamiento en la puerta de enlace para habilitar la comunicación.

Cualquier puerta de enlace puede usar módulos de IoT Edge para realizar análisis o preprocesamiento en el perímetro antes de pasar mensajes de dispositivos de bajada a IoT Hub.

![Diagrama: patrones de puerta de enlace transparente, de protocolo y de identidad](./media/iot-edge-as-gateway/edge-as-gateway.png)

### <a name="transparent-pattern"></a>Patrón transparente

En un patrón de puerta de enlace *transparente*, los dispositivos que podrían conectarse teóricamente a IoT Hub pueden conectarse en su lugar a un dispositivo de puerta de enlace. Los dispositivos de bajada tienen sus propias identidades IoT Hub y usan cualquiera de los protocolos MQTT, AMQP o HTTP. La puerta de enlace simplemente pasa las comunicaciones entre los dispositivos e IoT Hub. Tanto los dispositivos como los usuarios que interactúan con ellos mediante IoT Hub no saben que una puerta de enlace está arbitrando sus comunicaciones. Esta falta de conocimiento significa que la puerta de enlace se considera *transparente*.

El entorno de ejecución de Azure IoT Edge incluye funcionalidades de puerta de enlace transparente. Para más información, consulte [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md).

### <a name="protocol-translation-pattern"></a>Patrón de traducción de protocolo

Una puerta de enlace de *traducción de protocolo* también se conoce como puerta de enlace *opaca*, a diferencia del patrón de puerta de enlace transparente. En este patrón, los dispositivos que no admiten MQTT, AMQP o HTTP pueden usar un dispositivo de puerta de enlace para enviar datos a IoT Hub en su nombre. La puerta de enlace comprende el protocolo utilizado por los dispositivos de nivel inferior y es el único dispositivo que tiene identidad en IoT Hub. Toda la información parece proceder de un dispositivo, la puerta de enlace. Los dispositivos de nivel inferior deben insertar información de identificación adicional en sus mensajes si las aplicaciones de nube quieren analizar los datos de cada dispositivo. Además, los primitivos de IoT Hub, como los gemelos y los métodos, solo están disponibles para el dispositivo de puerta de enlace, no para los dispositivos de bajada.

El entorno de ejecución de Azure IoT Edge no incluye funcionalidades de traducción de protocolo. Este patrón requiere módulos personalizados o de terceros que suelen ser específicos del hardware y el protocolo usados. [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) contiene varios módulos de traducción de protocolo para elegir.

### <a name="identity-translation-pattern"></a>Patrón de traducción de identidad

En un patrón de *traducción de identidad*, los dispositivos que no se pueden conectar a IoT Hub en su lugar pueden conectarse a un dispositivo de puerta de enlace. La puerta de enlace proporciona a IoT Hub la traducción de protocolos e identidad en nombre de los dispositivos de bajada. La puerta de enlace es lo suficientemente inteligente como para comprender el protocolo utilizado por los dispositivos de nivel inferior, proporcionarles una identidad y traducir los primitivos de IoT Hub. Los dispositivos de nivel inferior aparecen en IoT Hub como dispositivos de primera clase con gemelos y métodos. Un usuario puede interactuar con los dispositivos en IoT Hub y no ser consciente del dispositivo de puerta de enlace intermedio.

El entorno de ejecución de Azure IoT Edge no incluye funcionalidades de traducción de identidad. Este patrón requiere módulos personalizados o de terceros que suelen ser específicos del hardware y el protocolo usados. Para obtener un ejemplo que usa el patrón de traducción de identidad, vea [Starter Kit de LoRaWAN para Azure IoT Edge](https://github.com/Azure/iotedge-lorawan-starterkit).

## <a name="use-cases"></a>Casos de uso

Todos los patrones de puerta de enlace proporcionan las siguientes ventajas:

* **Análisis en el perimetral**: use los servicios de IA en el entorno local para procesar los datos que proceden de dispositivos de nivel inferior sin enviar datos de telemetría con total fidelidad a la nube. Busca y reacciona a la información localmente y solo envía un subconjunto de datos a IoT Hub.
* **Aislamiento de dispositivos de nivel inferior**: el dispositivo de puerta de enlace puede proteger todos los dispositivos de nivel inferior de la exposición a Internet. Se puede situar entre una red de tecnología operativa (OT) que no tiene conectividad y una red de tecnología de la información (TI) que proporciona acceso a Internet.
* **Multiplexación de conexiones**: todos los dispositivos que se conectan a IoT Hub mediante una puerta de enlace IoT Edge usan la misma conexión subyacente.
* **Suavizado de tráfico**: el dispositivo IoT Edge implementará automáticamente un retroceso exponencial si IoT Hub limita el tráfico, al tiempo que se conservan los mensajes localmente. Esta ventaja hace que la solución sea resistente a los picos de tráfico.
* **Compatibilidad sin conexión**: el dispositivo de puerta de enlace almacena los mensajes y las actualizaciones gemelas que no se puedan entregar a IoT Hub.

Una puerta de enlace que realiza la traducción de protocolo puede admitir dispositivos existentes y dispositivos nuevos que tienen restricciones de recursos. Muchos dispositivos existentes producen datos que pueden favorecer el conocimiento del negocio; sin embargo, no están diseñados teniendo en mente la conectividad a la nube. Las puertas de enlace opacas permiten que estos datos se desbloqueen y utilicen en una solución IoT.

Una puerta de enlace que realiza la traducción de la identidad proporciona las ventajas de la traducción del protocolo y además permite una manejabilidad completa de los dispositivos de nivel inferior desde la nube. Todos los dispositivos de la solución IoT se muestran en IoT Hub con independencia del protocolo que usen.

## <a name="cheat-sheet"></a>Hoja de referencia rápida

Se trata de una hoja de características clave rápida que compara los primitivos de IoT Hub cuando se usan puertas de enlace transparentes, opacas (protocolo) y proxy.

| Primitivo | Puerta de enlace transparente | Traducción del protocolo | Traducción de la identidad |
|--------|-------------|--------|--------|
| Identidades almacenadas en el registro de identidades de IoT Hub | Identidades de todos los dispositivos conectados | Solo la identidad del dispositivo de puerta de enlace | Identidades de todos los dispositivos conectados |
| Dispositivo gemelo | Cada dispositivo conectado tiene su propio dispositivo gemelo | Solo la puerta de enlace tiene dispositivo y un módulo gemelos | Cada dispositivo conectado tiene su propio dispositivo gemelo |
| Métodos directos y mensajes de nube a dispositivo | La nube puede tratar individualmente cada dispositivo conectado | La nube solo puede tratar el dispositivo de puerta de enlace | La nube puede tratar individualmente cada dispositivo conectado |
| [Limitaciones y cuotas de IoT Hub](../iot-hub/iot-hub-devguide-quotas-throttling.md) | Aplicación a cada dispositivo | Aplicación al dispositivo de puerta de enlace | Aplicación a cada dispositivo |

Cuando se usa un patrón de puerta de enlace opaca (traducción de protocolo), todos los dispositivos que se conectan a través de esa puerta de enlace comparten la misma cola de nube a dispositivo, que puede contener 50 mensajes como máximo. De ello se desprende que el patrón de puerta de enlace opaca debe usarse solo cuando pocos dispositivos están conectados a través de cada puerta de enlace de campo y su tráfico de nube a dispositivo sea bajo.

## <a name="next-steps"></a>Pasos siguientes

Aprenda los tres pasos para configurar una puerta de enlace transparente:

* [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md)
* [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md)
* [Conexión de un dispositivo de bajada a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-device.md)
