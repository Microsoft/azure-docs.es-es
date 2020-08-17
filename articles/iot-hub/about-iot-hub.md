---
title: Introducción a Azure IoT Hub | Microsoft Docs
description: Obtenga más información acerca de Azure IoT Hub. Este servicio de IoT está pensado para la ingesta de datos escalables, la administración de dispositivos y la seguridad.
author: nberdy
ms.author: nberdy
ms.date: 08/08/2019
ms.topic: overview
ms.custom:
- mvc
- amqp
- mqtt
- 'role: Direction'
- 'role: System Architecture'
ms.service: iot-hub
services: iot-hub
ms.openlocfilehash: cde9d8614883168c3fbb3ab8219cdbef6087c9d0
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87428977"
---
# <a name="what-is-azure-iot-hub"></a>¿Qué es Azure IoT Hub?

IoT Hub es un servicio administrado, hospedado en la nube, que actúa como centro de mensajes para comunicaciones bidireccionales entre la aplicación de IoT y los dispositivos que administra. Puede usar Azure IoT Hub para compilar soluciones de IoT con comunicaciones confiables y seguras entre millones de dispositivos de IoT y un back-end de solución hospedado en la nube. Puede conectar virtualmente cualquier dispositivo a IoT Hub.

IoT Hub es compatible con comunicaciones del dispositivo a la nube y viceversa. IoT Hub es compatible con varios patrones de mensajes, como telemetría de dispositivo a la nube, carga de archivos desde dispositivos y métodos de respuesta a solicitudes para controlar los dispositivos desde la nube. La supervisión de IoT Hub le ayuda a conservar el estado de la solución, ya que realiza el seguimiento de eventos, como la creación, los errores y las conexiones de dispositivos.

Las funcionalidades de IoT Hub le ayudan a compilar soluciones de IoT escalables y completas, como la administración de equipos industriales que se usan en fabricación, el seguimiento de activos valiosos en el sector sanitario y la supervisión del uso de compilaciones en oficina.

## <a name="scale-your-solution"></a>Escalado de la solución

IoT Hub escala a millones de dispositivos conectados de manera simultánea y a millones de eventos por segundo para admitir las cargas de trabajo de IoT. Para obtener más información sobre cómo escalar el IoT Hub, vea [Escalado de IOT Hub](iot-hub-scaling.md?branch=release-iotbasic). Para obtener más información sobre los distintos niveles de servicio ofrecidos por IoT Hub y cómo ajustarse mejor a sus necesidades de escalabilidad, vea la [página de precios](https://azure.microsoft.com/pricing/details/iot-hub/).

## <a name="secure-your-communications"></a>Proteger las comunicaciones

IoT Hub ofrece un canal de comunicación seguro para que los dispositivos envíen datos.

* La autenticación por dispositivo permite que cada dispositivo se conecte a IoT Hub y se administre de manera segura.

* Tiene todo el control sobre el acceso de los dispositivos y puede controlar las conexiones por dispositivo.

* [IoT Hub Device Provisioning](https://docs.microsoft.com/azure/iot-dps/) aprovisiona automáticamente dispositivos en IoT Hub de manera correcta cuando el dispositivo se inicia por primera vez.

* Varios tipos de autenticación son compatibles con varias funcionalidades de dispositivos:

  * Autenticación basada en token de SAS para empezar rápidamente con la solución de IoT.

  * Autenticación de certificado X.509 individual para una autenticación segura basada en estándares.

  * Autenticación X.509 CA para una inscripción básica basada en estándares.

## <a name="route-device-data"></a>Enrutar datos del dispositivo

La funcionalidad integrada de redirección de mensajes le ofrece flexibilidad para configurar distribuciones ramificadas de mensajes basadas en reglas de manera automática:

* Use el [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md) para controlar dónde envía el centro los datos de telemetría del dispositivo.

* Enrutar mensajes a varios puntos de conexión no tiene ningún costo adicional.

* Las reglas de enrutamiento sin código sustituyen los códigos de distribuidor de mensajes personalizados.

## <a name="integrate-with-other-services"></a>Integración con otros servicios

Puede integrar IoT Hub con otros servicios de Azure para compilar soluciones completas de un extremo a otro. Por ejemplo, use:

* [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/) para permitir que la empresa responda rápidamente a eventos críticos de forma confiable, escalable y segura.

* [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/) para automatizar procesos de negocio.

* [Azure Machine Learning](iot-hub-weather-forecast-machine-learning.md) para agregar aprendizaje automático y modelos de AI a la solución.

* [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/) para ejecutar cálculos de análisis en tiempo real en los flujos de datos de los dispositivos.

## <a name="configure-and-control-your-devices"></a>Configurar y controlar dispositivos

Puede administrar los dispositivos conectados a IoT Hub con una matriz de funcionalidades integradas.

* Almacene, sincronice y consulte metadatos e información de estado de todos los dispositivos.

* Establezca el estado de los dispositivos por dispositivo o en función de características comunes de los dispositivos.

* Responda automáticamente a un cambio de estado notificado por el dispositivo con la integración de enrutamiento de mensajes.

## <a name="make-your-solution-highly-available"></a>Hacer que la solución tenga alta disponibilidad

Hay un [Acuerdo de Nivel de Servicio para IoT Hub](https://azure.microsoft.com/support/legal/sla/iot-hub/) al 99,9 %. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

## <a name="connect-your-devices"></a>Conectar dispositivos

Use las bibliotecas de [SDK de dispositivo IoT de Azure](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-sdks) bibliotecas para compilar aplicaciones que se ejecutan en los dispositivos e interactúan con IoT Hub. Las plataformas compatibles incluyen varias distribuciones de Linux, Windows y sistemas operativos en tiempo real. Los lenguajes compatibles incluyen:

* C
* C#
* Java
* Python
* Node.js.

IoT Hub y los SDK de dispositivo admiten los siguientes protocolos de conexión de dispositivos:

* HTTPS
* AMQP
* AMQP sobre WebSockets
* MQTT
* MQTT sobre WebSockets

Si la solución no puede usar las bibliotecas de dispositivos, los dispositivos pueden usar los protocolos MQTT v3.1.1, HTTPS 1.1 o AMQP 1.0 para conectarse de manera nativa al centro.

Si la solución no puede usar ninguno de los protocolos compatibles, puede ampliar IoT Hub para admitir protocolos personalizados:

* Use [Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/) para crear una puerta de enlace de campo para realizar la traslación de protocolos en dispositivos perimetrales.

* Personalice la [puerta de enlace de protocolo de Azure IoT](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md) para realizar la traslación de protocolos en la nube.

## <a name="quotas-and-limits"></a>Cuotas y límites

Cada suscripción de Azure tiene límites de cuota predeterminados para impedir el abuso del servicio y pueden afectar al ámbito de la solución de IoT. El límite actual es de 50 de centros de IoT Hub por suscripción. Puede solicitar un aumento de la cuota si se pone en contacto con el soporte técnico. Para obtener más información, consulte las [cuotas y limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md). Para obtener más información sobre los límites de cuota, vea alguno de los siguientes artículos:

* [Límites de servicio de suscripción de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md)

* [IoT Hub throttling and you](https://azure.microsoft.com/blog/iot-hub-throttling-and-you/) (Limitación de IoT Hub)

## <a name="next-steps"></a>Pasos siguientes

Para probar una solución de IoT de un extremo a otro, consulte los siguientes inicios rápidos de IoT Hub:

* [Inicio rápido: Enviar telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-node.md)

Para más información sobre las distintas formas en que puede compilar e implementar soluciones de IoT con IoT de Azure, visite:

* [Aspectos básicos: Soluciones y tecnologías de IoT de Azure](../iot-fundamentals/iot-services-and-technologies.md).
