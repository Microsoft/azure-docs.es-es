---
title: Información de los puntos de conexión de Azure IoT Hub | Microsoft Docs
description: 'Guía para desarrolladores: información de referencia sobre los puntos de conexión orientados a dispositivos y servicios IoT Hub.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 06/10/2019
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: System Architecture'
ms.openlocfilehash: 9ae21b2a6d470b959f5e869e8a9edda9c56bec73
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90019570"
---
# <a name="reference---iot-hub-endpoints"></a>Referencia: Puntos de conexión de IoT Hub

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="iot-hub-names"></a>Nombres de IoT Hub

Puede encontrar el nombre de host de la instancia de IoT Hub que hospeda los puntos de conexión en el portal en la página **Información general** de la central. De forma predeterminada, el nombre DNS de una instancia de IoT Hub es similar a lo siguiente: `{your iot hub name}.azure-devices.net`.

## <a name="list-of-built-in-iot-hub-endpoints"></a>Lista de puntos de conexión de IoT Hub integrados

Azure IoT Hub es un servicio multiinquilino que muestra su funcionalidad a diversos actores. En el siguiente diagrama se muestran los diferentes puntos de conexión expuestos por IoT Hub.

![Puntos de conexión de IoT Hub](./media/iot-hub-devguide-endpoints/endpoints.png)

En la lista siguiente se describen los puntos de conexión:

* **Proveedor de recursos**. El proveedor de recursos de IoT Hub expone una interfaz de [Azure Resource Manager](../azure-resource-manager/management/overview.md). Esta interfaz permite que los propietarios de suscripciones de Azure creen y eliminen instancias de IoT Hub para actualizar sus propiedades. Las propiedades de IoT Hub controlan las [directivas de seguridad a nivel de centro](iot-hub-devguide-security.md#access-control-and-permissions), a diferencia del control de acceso de nivel de dispositivo y las opciones funcionales para la mensajería de nube a dispositivo y de dispositivo a nube. El proveedor de recursos también permite [exportar identidades de dispositivo](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

* **Administración de identidades de dispositivo**. Cada instancia de IoT Hub muestra un conjunto de puntos de conexión HTTPS REST para administrar las identidades de dispositivo (crear, recuperar, actualizar y eliminar). Las [identidades del dispositivo](iot-hub-devguide-identity-registry.md) se usan para autenticación de dispositivos y control de acceso.

* **Administración de dispositivo gemelo**. Cada instancia de IoT Hub muestra un conjunto de punto de conexión de HTTPS REST orientado a servicios para consultar y actualizar [dispositivos gemelos](iot-hub-devguide-device-twins.md) (actualización de etiquetas y propiedades). 

* **Administración de trabajos**. Cada instancia de IoT Hub muestra un conjunto de punto de conexión de HTTPS REST orientado a servicios para consultar y administrar [trabajos](iot-hub-devguide-jobs.md).

* **Puntos de conexión de dispositivo**. Por cada dispositivo del registro de identidad, IoT Hub muestra un conjunto de puntos de conexión. Salvo cuando se indique lo contrario, estos puntos de conexión se exponen a través de los protocolos [MQTT v3.1.1](https://mqtt.org/), HTTPS 1.1 y [AMQP 1.0](https://www.amqp.org/). AMQP y MQTT también están disponibles sobre [WebSockets](https://tools.ietf.org/html/rfc6455) en el puerto 443.

  * *Envío de mensajes de dispositivo a nube*. Un dispositivo usa este punto de conexión para [enviar mensajes de dispositivo a la nube](iot-hub-devguide-messages-d2c.md).

  * *Recepción de mensajes de nube a dispositivo*. El dispositivo usa ese punto de conexión para recibir [mensajes de nube a dispositivos](iot-hub-devguide-messages-c2d.md) dirigidos.

  * *Iniciar cargas de archivos*. Un dispositivo usa este punto de conexión para recibir un URI de SAS de Azure Storage de IoT Hub para [cargar un archivo](iot-hub-devguide-file-upload.md).

  * *Recuperación y actualización de las propiedades del dispositivo gemelo*. Un dispositivo usa este punto de conexión para tener acceso a las propiedades del [dispositivo gemelo](iot-hub-devguide-device-twins.md). HTTPS no es compatible.

  * *Recepción de solicitudes de métodos directos*. Un dispositivo usa este punto de conexión para escuchar solicitudes de [métodos directos](iot-hub-devguide-direct-methods.md). HTTPS no es compatible.

  [!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

* **Puntos de conexión de servicio**. Cada instancia de IoT Hub muestra un conjunto de puntos de conexión que el back-end de solución para comunicarse con los dispositivos. Con una excepción, estos puntos de conexión solo se muestran con los protocolo [AMQP](https://www.amqp.org/) y AMQP sobre WebSockets. El punto de conexión de invocación de método directo se muestra en el protocolo HTTPS.
  
  * *Recepción de mensajes de dispositivo a nube*. Este punto de conexión es compatible con [Azure Event Hubs](https://azure.microsoft.com/documentation/services/event-hubs/). Un servicio back-end puede usarse para leer los [mensajes de dispositivo a nube](iot-hub-devguide-messages-d2c.md) enviados por los dispositivos. Además de este punto de conexión integrado, puede crear puntos de conexión personalizados en el centro de IoT.
  
  * *Envío de mensajes de nube a dispositivo y recepción de confirmaciones de entrega*. Estos puntos de conexión permiten al back-end de aplicaciones enviar mensajes confiables [de nube a dispositivo](iot-hub-devguide-messages-c2d.md) y recibir las confirmaciones de entrega o expiración correspondientes.
  
  * *Recepción de notificaciones de archivos*. Este punto de conexión de mensajería le permite recibir notificaciones del momento en que los dispositivos cargan correctamente un archivo. 
  
  * *Invocación del método director*. Este punto de conexión permite que un servicio back-end invoque un [método directo](iot-hub-devguide-direct-methods.md) en un dispositivo.
  
  * *Recepción de eventos de supervisión de operaciones*. Este punto de conexión le permite recibir eventos de supervisión de operaciones si su centro de IoT se ha configurado para emitirlos. Para más información, consulte [Supervisión de operaciones de IoT Hub](iot-hub-operations-monitoring.md).

En el artículo [SDK IoT de Azure](iot-hub-devguide-sdks.md) se describen las distintas formas con las que se puede acceder a estos puntos de conexión.

Todos los puntos de conexión de IoT Hub usan el protocolo [TLS](https://tools.ietf.org/html/rfc5246) y ningún punto de conexión se expone en canales sin cifrar o no seguros.

## <a name="custom-endpoints"></a>Puntos de conexión personalizados

Puede vincular los servicios de Azure existentes en su centro de IoT para usarlos como puntos de conexión para el enrutamiento de mensajes. Estos puntos de conexión hacen de puntos de conexión de servicio y se usan como receptores para las rutas de los mensajes. Los dispositivos no pueden escribir directamente en los puntos de conexión adicionales. Más información sobre el [enrutamiento de mensajes](../iot-hub/iot-hub-devguide-messages-d2c.md).

IoT Hub admite actualmente los siguientes servicios de Azure como puntos de conexión adicionales:

* Contenedores de Azure Storage
* Event Hubs
* Colas de Service Bus
* Temas de Service Bus

Para conocer los límites del número de puntos de conexión que se pueden agregar, consulte [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md).

## <a name="endpoint-health"></a>Mantenimiento del punto de conexión

[!INCLUDE [iot-hub-endpoint-health](../../includes/iot-hub-include-endpoint-health.md)]

## <a name="field-gateways"></a>Puertas de enlace de campo

En una solución de IoT, un *puerta de enlace de campo* se encuentra entre los dispositivos y los puntos de conexión de IoT Hub. Suele encontrarse cerca de los dispositivos. Los dispositivos se comunican directamente con la puerta de enlace de campo mediante un protocolo compatible con los dispositivos. La puerta de enlace de campo se conecta al punto de conexión de IoT Hub con un protocolo que es compatible con IoT Hub. Una puerta de enlace de campo puede ser un dispositivo de hardware dedicado o un equipo de bajo consumo de energía que ejecuta el software de puerta de enlace personalizado.

Puede usar [Azure IoT Edge](/azure/iot-edge/) para implementar una puerta de enlace de campo. IoT Edge ofrece la funcionalidad de multiplexar la comunicación desde varios dispositivos en la misma conexión de IoT Hub.

## <a name="next-steps"></a>Pasos siguientes

Otros temas de referencia en la Guía del desarrollador de IoT Hub son:

* [Lenguaje de consulta de IoT Hub para dispositivos gemelos, trabajos y enrutamiento de mensajes](iot-hub-devguide-query-language.md)
* [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md)
* [Compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md)
* [Información sobre la dirección IP de IoT Hub](iot-hub-understand-ip-address.md)