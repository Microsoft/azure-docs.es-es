---
title: Información sobre la mensajería de IoT Hub de Azure | Microsoft Docs
description: 'Guía del desarrollador: mensajería de dispositivo a nube y de nube a dispositivo con IoT Hub Incluye información sobre los formatos de mensaje y protocolos de comunicación compatibles.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: e0466aa3497edf5f271c01944438c033bba152f4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327674"
---
# <a name="send-device-to-cloud-and-cloud-to-device-messages-with-iot-hub"></a>Envío de mensajes de dispositivo a nube y de nube a dispositivo con IoT Hub

IoT Hub permite la comunicación segura y bidireccional con los dispositivos. Use la mensajería de IoT Hub para comunicar con los dispositivos mediante el envío de mensajes desde los dispositivos a las soluciones back-end y para enviar comandos desde las soluciones back-end de IoT a los dispositivos. Más información acerca del [formato de mensajes de IoT Hub](iot-hub-devguide-messages-construct.md).

## <a name="sending-device-to-cloud-messages-to-iot-hub"></a>Envío de mensajes de dispositivos a la nube para IoT Hub

IoT Hub tiene un punto de conexión de servicio integrado que puede utilizarse en los servicios back-end para leer los mensajes de telemetría de los dispositivos. Este punto de conexión es compatible con [Event Hubs](https://docs.microsoft.com/azure/event-hubs/) y el SDK de IoT Hub estándar se puede usar para [leer desde este punto de conexión integrado](iot-hub-devguide-messages-read-builtin.md).

IoT Hub también admite [puntos de conexión personalizados](iot-hub-devguide-endpoints.md#custom-endpoints) que los usuarios pueden definir para enviar eventos y datos de telemetría de dispositivo a los servicios de Azure mediante el [enrutamiento de los mensajes](iot-hub-devguide-messages-d2c.md).

## <a name="sending-cloud-to-device-messages-from-iot-hub"></a>Envío de mensajes de nube a dispositivo desde IoT Hub

Puede enviar mensajes [de nube a dispositivo](iot-hub-devguide-messages-c2d.md) desde la solución de back-end a sus dispositivos.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Las propiedades básicas de la funcionalidad de mensajería de IoT Hub son la confiabilidad y durabilidad de los mensajes. Estas propiedades permiten la resistencia a la conectividad intermitente en el dispositivo y a los picos de carga del procesamiento de eventos en la nube. IoT Hub implementa *al menos una vez* garantías de entrega para la mensajería del dispositivo a la nube y de la nube al dispositivo.

## <a name="choosing-the-right-type-of-iot-hub-messaging"></a>Elección del tipo correcto de mensajería de IoT Hub

Los mensajes de dispositivo a nube se utilizan para enviar telemetría y alertas de series temporales desde la aplicación para dispositivo y los mensajes de nube a dispositivo, para las notificaciones unidireccionales a su aplicación para dispositivo.

* Para elegir entre los mensajes de dispositivo a nube, las propiedades notificadas o la carga de archivos, consulte la [Guía de comunicación de dispositivo a nube](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-d2c-guidance).

* Para elegir entre los mensajes de nube a dispositivo, las propiedades deseadas o los métodos directos, consulte la [Guía de comunicación de nube a dispositivo](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-c2d-guidance).

## <a name="next-steps"></a>Pasos siguientes

* Información acerca del [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md) de IoT Hub.

* Más información acerca de la [mensajería de nube a dispositivo](iot-hub-devguide-messages-c2d.md) de IoT Hub.