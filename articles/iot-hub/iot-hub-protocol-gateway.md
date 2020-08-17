---
title: Puerta de enlace de protocolos de IoT de Azure | Microsoft Docs
description: Describe cómo usar una puerta de enlace de protocolo de IoT de Azure para extender el protocolo y las funcionalidades de IoT Hub para permitir que los dispositivos se conecten al centro que usa los protocolos no compatibles con IoT de forma nativa.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/11/2017
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 315d76715973b79a971f3ca15bcc5186d20135a2
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325039"
---
# <a name="support-additional-protocols-for-iot-hub"></a>Compatibilidad con protocolos adicionales para centro de IoT Hub

Azure IoT Hub admite de manera nativa la comunicación a través de los protocolos MQTT, AMQP y HTTPS. En algunos casos, puede que los dispositivos o las puertas de enlace de campo no puedan usar uno de estos protocolos estándar y requieran la adaptación del mismo. En esos casos, puede usar una puerta de enlace personalizada. Una puerta de enlace personalizada habilita la adaptación de protocolos para los puntos de conexión de IoT Hub puenteando el tráfico que se origina o finaliza en IoT Hub. Puede usar la [Puerta de enlace de protocolos de IoT de Azure](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md) como puerta de enlace personalizada para permitir la adaptación de protocolos para Azure IoT Hub.

## <a name="azure-iot-protocol-gateway"></a>Puerta de enlace de protocolos de IoT de Azure

La puerta de enlace de protocolos de IoT de Azure es un marco para la adaptación de protocolos diseñado para la comunicación bidireccional a gran escala de dispositivos con Azure IoT Hub. La puerta de enlace de protocolo es un componente de acceso directo que acepta conexiones de dispositivos a través de un protocolo específico. Une el tráfico a IoT Hub sobre AMQP 1.0.

Puede implementar la puerta de enlace de protocolo en Azure de forma muy escalable con Azure Service Fabric, los roles de trabajo de Azure Cloud Services o Windows Virtual Machines. Además, la puerta de enlace de protocolos puede implementarse en entornos locales como puertas de enlace de campo.

La puerta de enlace de protocolos de IoT de Azure incluye un adaptador de protocolos de MQTT que le permite personalizar el comportamiento del protocolo de MQTT si es necesario. Puesto que IoT Hub proporciona compatibilidad integrada para el protocolo de MQTT v3.1.1, solo debe considerar el uso del adaptador de protocolo MQTT si necesita personalizaciones de protocolo o requisitos específicos para una funcionalidad adicional.

El adaptador de MQTT también muestra el modelo de programación para la creación de adaptadores de protocolo para otros protocolos. Además, el modelo de programación de puerta de enlace de protocolo de IoT de Azure le permite conectar componentes personalizados para su procesamiento especializado como, por ejemplo, autenticación personalizada, transformaciones de mensajes, compresión y descompresión o cifrado y descifrado de tráfico entre los dispositivos y el IoT Hub.

Para mayor flexibilidad, la puerta de enlace de protocolos de Azure IoT y la implementación de MQTT se ofrecen como proyecto de software de código abierto. Puede usar el proyecto de código abierto para agregar compatibilidad con distintos protocolos y versiones de protocolo o personalizar la implementación para su escenario. 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la puerta de enlace de protocolos de IoT de Azure y cómo usarlo e implementarlo como parte de su solución de IoT, vea:

* [Repositorio de puerta de enlace de protocolos de IoT de Azure en GitHub](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)

* [Guía para desarrolladores de puerta de enlace de protocolos de IoT de Azure](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/docs/DeveloperGuide.md)

Para más información acerca de planificación de la implementación de IoT Hub, consulte:

* [Comparación con Event Hubs](iot-hub-compare-event-hubs.md)

* [Escalado, alta disponibilidad y recuperación ante desastres](iot-hub-scaling.md)

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)