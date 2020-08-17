---
title: Comparación de Azure IoT Hub con Azure Event Hubs | Microsoft Docs
description: Comparación de los servicios de IoT Hub y los servicios de Event Hubs de Azure en la que se resaltan diferencias funcionales y casos de uso. La comparación incluye protocolos admitidos, administración de dispositivos, supervisión y cargas de archivos.
author: kgremban
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: kgremban
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: System Architecture'
ms.openlocfilehash: 1b51e2338dabd34ec69d311a9eb1a5c4a644f628
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322761"
---
# <a name="connecting-iot-devices-to-azure-iot-hub-and-event-hubs"></a>Conexión de dispositivos IoT a Azure: IoT Hub y Event Hubs

Azure proporciona servicios desarrollados específicamente para diversos tipos de conectividad y comunicación para ayudar a conectar los datos a la tecnología de la nube. Tanto Azure IoT Hub como Azure Event Hubs son servicios en la nube que pueden ingerir grandes cantidades de datos y procesar o almacenar esos datos para obtener perspectivas empresariales. Los dos servicios se asemejan en que ambos admiten la ingesta de datos con baja latencia y alta fiabilidad, pero están diseñados para propósitos diferentes. IoT Hub se desarrolló para satisfacer los requisitos únicos de la conexión de dispositivos IoT a la nube de Azure, mientras que Event Hubs se diseñó para el flujo de macrodatos. Microsoft recomienda el uso de Azure IoT Hub para conectar dispositivos IoT a Azure.

Azure IoT Hub es la puerta de enlace en la nube que conecta dispositivos IoT para recopilar los datos y dirigir las perspectivas y automatización empresariales. Además, IoT Hub incluye características que enriquecen la relación entre los dispositivos y los sistemas back-end. Las capacidades de comunicación bidireccional implican que al tiempo que se reciben datos de los dispositivos, también es posible devolver comandos y directivas a los dispositivos. Por ejemplo, utilice la mensajería de la nube al dispositivo para actualizar las propiedades o invocar acciones de administración de dispositivos. La comunicación de la nube al dispositivo también le permite enviar inteligencia de nube a los dispositivos perimetrales con Azure IoT Edge. La identidad única de nivel de dispositivo proporcionada por IoT Hub ayuda a proteger mejor la solución IoT frente a ataques potenciales. 

[Azure Event Hubs](../event-hubs/event-hubs-what-is-event-hubs.md) es el servicio de flujo de macrodatos de Azure. Está diseñado para los escenarios de flujo de datos de alto rendimiento en los que los clientes pueden enviar miles de millones de solicitudes al día. Event Hubs usa un modelo de consumidor con particiones para escalar horizontalmente el flujo de datos y se integra en los servicios de macrodatos y análisis de Azure, entre ellos, Databricks, Stream Analytics, ADLS y HDInsight. Gracias a características como Event Hubs Capture y el inflado automático, el diseño de este servicio admite soluciones y aplicaciones de macrodatos. Además, IoT Hub utiliza Event Hubs para la ruta de acceso del flujo de telemetría, por lo que la solución IoT también se beneficia de la gran eficacia de Event Hubs.

En resumen, ambas soluciones están diseñadas para la ingesta de datos a escala masiva. Solo IoT Hub proporciona las abundantes capacidades específicas de IoT diseñadas para maximizar el valor empresarial de conectar los dispositivos IoT a la nube de Azure.  Si acaba de iniciar su experiencia en IoT, al empezar por IoT Hub para admitir los escenarios de ingesta de datos garantizará el acceso inmediato a las capacidades de IoT con todas las características cuando las necesidades empresariales y técnicas las requieran.

En la tabla siguiente se proporcionan detalles de la comparación entre dos niveles de IoT Hub y Event Hubs al valorarlas en función de su capacidad de IoT. Para más información acerca de los niveles básico y estándar de IoT Hub, consulte [Elección del nivel adecuado de IoT Hub](iot-hub-scaling.md).

| Funcionalidad de IoT | Nivel estándar de IoT Hub | Nivel básico de IoT Hub | Event Hubs |
| --- | --- | --- | --- |
| Mensajería de un dispositivo a la nube | ![Comprobar][checkmark] | ![Comprobar][checkmark] | ![Comprobar][checkmark] |
| Protocolos: HTTPS, AMQP, AMQP sobre WebSockets | ![Comprobar][checkmark] | ![Comprobar][checkmark] | ![Comprobar][checkmark] |
| Protocolos: MQTT, MQTT sobre WebSockets | ![Comprobar][checkmark] | ![Comprobar][checkmark] |  |
| Identidad por dispositivo | ![Comprobar][checkmark] | ![Comprobar][checkmark] |  |
| Carga de archivos desde dispositivos | ![Comprobar][checkmark] | ![Comprobar][checkmark] |  |
| Servicio Device Provisioning | ![Comprobar][checkmark] | ![Comprobar][checkmark] |  |
| Mensajería de la nube a un dispositivo | ![Comprobar][checkmark] |  |  |
| Administración de dispositivos y dispositivos gemelos | ![Comprobar][checkmark] |  |  |
| Flujos de dispositivos (versión preliminar) | ![Comprobar][checkmark] |  |  |
| IoT Edge | ![Comprobar][checkmark] |  |  |

Aunque el único uso sea la ingesta de datos de dispositivo a nube, es muy recomendable utilizar IoT Hub porque ofrece un servicio que está diseñado para la conectividad de dispositivos IoT. 

### <a name="next-steps"></a>Pasos siguientes

Para explorar aún más las funcionalidades de IoT Hub, consulte la [Guía del desarrollador de Azure IoT Hub](iot-hub-devguide.md).

<!-- This one reference link is used over and over. --robinsh -->
[checkmark]: ./media/iot-hub-compare-event-hubs/ic195031.png
