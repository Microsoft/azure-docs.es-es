---
title: Uso de métricas para supervisar Azure IoT Hub | Microsoft Docs
description: Describe cómo usar las métricas de IoT Hub de Azure para evaluar y supervisar el estado general de los centros de IoT Hub.
author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/24/2019
ms.author: jlian
ms.openlocfilehash: 8eac70db0c563f102dfa0e3fcece9d4604582cce
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595645"
---
# <a name="understand-iot-hub-metrics"></a>Comprender las métricas de IoT Hub

Las métricas de IoT Hub ofrecen mejores datos sobre el estado de los recursos de Azure en la suscripción de Azure. Las métricas de IoT Hub permiten evaluar el estado general del servicio IoT Hub y de los dispositivos conectados a él. Las estadísticas orientadas al usuario son importantes porque ayudan a ver lo que está ocurriendo con su Centro de IoT y ayudan a determinar la causa raíz de los problemas sin necesidad de ponerse en contacto con el soporte técnico de Azure.

Las métricas están habilitadas de forma predeterminada. Puede ver las métricas de IoT Hub desde Azure Portal.

## <a name="how-to-view-iot-hub-metrics"></a>Cómo ver las métricas de IoT Hub

1. Cree un Centro de IoT. Puede encontrar instrucciones sobre cómo crear una instancia de IoT hub en la guía [Enviar datos de telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-dotnet.md).

2. Abra la hoja de su centro de IoT. Desde aquí, haga clic en **Métricas**.
   
    ![Captura de pantalla que muestra dónde está la opción de métricas en la página de recursos de IoT Hub](./media/iot-hub-metrics/enable-metrics-1.png)

3. En la hoja de métricas, puede ver las métricas para su centro de IoT y crear vistas personalizadas de ellas. 
   
    ![Captura de pantalla que muestra la página de métricas de IoT Hub](./media/iot-hub-metrics/enable-metrics-2.png)
    
4. Puede elegir entre enviar los datos de las métricas a un punto de conexión de Event Hubs o a una cuenta de Azure Storage haciendo clic en **Configuración de diagnóstico** y luego en **Agregar configuración de diagnóstico**.

   ![Captura de pantalla muestra dónde está el botón de configuración de diagnóstico](./media/iot-hub-metrics/enable-metrics-3.png)

## <a name="iot-hub-metrics-and-how-to-use-them"></a>Las métricas de IoT Hub y su uso

IoT Hub proporciona varias métricas para ofrecerle una visión general del estado de su centro y el número total de dispositivos conectados. Puede combinar información de varias métricas para conseguir una imagen más amplia del estado del Centro de IoT. La tabla siguiente describe las métricas de las que cada Centro de IoT realiza un seguimiento y cómo se relaciona cada métrica con el estado general del Centro de IoT.

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|DESCRIPCIÓN|Dimensiones|
|---|---|---|---|---|---|
|d2c<br>.telemetry<br>.ingress.<br>allProtocol|Intentos de envío de mensajes de telemetría|Count|Total|Número de mensajes de telemetría de dispositivo a la nube para enviar a IoT Hub|Sin dimensiones|
|d2c<br>.telemetry<br>.ingress<br>.success|Mensajes de telemetría enviados|Count|Total|Número de mensajes de telemetría de dispositivo a la nube enviados correctamente a IoT Hub|Sin dimensiones|
|c2d<br>.commands<br>.egress<br>.complete<br>.success|Comandos completados|Count|Total|Número de comandos de la nube al dispositivo que el dispositivo ha completado correctamente|Sin dimensiones|
|c2d<br>.commands<br>.egress<br>.abandon<br>.success|Comandos abandonados|Count|Total|Número de comandos de la nube al dispositivo que el dispositivo ha abandonado|Sin dimensiones|
|c2d<br>.commands<br>.egress<br>.reject<br>.success|Comandos rechazados|Count|Total|Número de comandos de la nube al dispositivo que el dispositivo ha rechazado|Sin dimensiones|
|dispositivos<br>.totalDevices|Total de dispositivos (en desuso)|Count|Total|Número de dispositivos registrados en IoT Hub|Sin dimensiones|
|dispositivos<br>.connectedDevices<br>.allProtocol|Dispositivos conectados (en desuso) |Count|Total|Número de dispositivos conectados a IoT Hub|Sin dimensiones|
|d2c<br>.telemetry<br>.egress<br>.success|Enrutamiento: mensajes de telemetría entregados|Count|Total|Número de veces que los mensajes se han entregado correctamente a todos los puntos de conexión mediante el enrutamiento de IoT Hub. Si un mensaje se enruta a varios puntos de conexión, este valor aumenta en uno por cada entrega correcta. Si un mensaje se enruta al mismo punto de conexión varias veces, este valor aumenta en uno por cada entrega correcta.|Sin dimensiones|
|d2c<br>.telemetry<br>.egress<br>.dropped|Enrutamiento: mensajes de telemetría quitados |Count|Total|Número de veces que se han quitado los mensajes con el enrutamiento de IoT Hub a causa de puntos de conexión inactivos. Este valor no cuenta los mensajes entregados a la ruta de reserva porque los mensajes quitados no se entregan allí.|Sin dimensiones|
|d2c<br>.telemetry<br>.egress<br>.orphaned|Enrutamiento: mensajes de telemetría huérfanos |Count|Total|Número de veces que los mensajes eran huérfanos a causa del enrutamiento de IoT Hub porque no coincidían con las reglas de enrutamiento (incluida la regla de reserva). |Sin dimensiones|
|d2c<br>.telemetry<br>.egress<br>.invalid|Enrutamiento: mensajes de telemetría incompatibles|Count|Total|Número de veces que el enrutamiento de IoT Hub no ha podido entregar mensajes debido a una incompatibilidad con el punto de conexión. Este valor no incluye los reintentos.|Sin dimensiones|
|d2c<br>.telemetry<br>.egress<br>.fallback|Enrutamiento: mensajes entregados a la reserva|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes al punto de conexión asociado con la ruta de reserva.|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.eventHubs|Enrutamiento: mensajes entregados al centro de eventos|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión del centro de eventos.|Sin dimensiones|
|d2c<br>.endpoints<br>.latency<br>.eventHubs|Enrutamiento: latencia de mensajes para el centro de eventos|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes a un punto de conexión del centro de eventos.|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.serviceBusQueues|Enrutamiento: mensajes entregados a la cola de Service Bus|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión de la cola de Service Bus.|Sin dimensiones|
|d2c<br>.endpoints<br>.latency<br>.serviceBusQueues|Enrutamiento: latencia de mensajes de la cola de Service Bus|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría a un punto de conexión de la cola de Service Bus.|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.serviceBusTopics|Enrutamiento: mensajes entregados al tema de Service Bus|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión del tema de Service Bus.|Sin dimensiones|
|d2c<br>.endpoints<br>.latency<br>.serviceBusTopics|Enrutamiento: latencia de mensajes del tema de Service Bus|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría a un punto de conexión del tema de Service Bus.|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.builtIn<br>.events|Enrutamiento: mensajes entregados a mensajes/eventos|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente al punto de conexión integrado (mensajes/eventos).|Sin dimensiones|
|d2c<br>.endpoints<br>.latency<br>.builtIn.events|Enrutamiento: latencia de mensajes para mensajes/eventos|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría al punto de conexión integrado (mensajes/eventos).|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.storage|Enrutamiento: mensajes entregados al almacenamiento|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado mensajes correctamente a los puntos de conexión de almacenamiento.|Sin dimensiones|
|d2c<br>.endpoints<br>.latency<br>.storage|Enrutamiento: latencia de mensajes para almacenamiento|Milisegundos|Promedio|Latencia media (milisegundos) entre la entrada de mensajes a IoT Hub y la entrada de mensajes de telemetría a un punto de conexión de almacenamiento.|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.storage<br>.bytes|Enrutamiento: datos entregados al almacenamiento|Bytes|Total|Cantidad de datos (bytes) que el enrutamiento de IoT Hub ha entregado a los punto de conexión de almacenamiento.|Sin dimensiones|
|d2c<br>.endpoints<br>.egress<br>.storage<br>.blobs|Enrutamiento: blobs entregados al almacenamiento|Count|Total|Número de veces que el enrutamiento de IoT Hub ha entregado blobs a los puntos de conexión de almacenamiento.|Sin dimensiones|
|EventGridDeliveries|Entregas de cuadrícula de eventos (versión preliminar)|Count|Total|El número de solicitudes de IoT Hub emitir eventos a Event Grid. Este número incluye solicitudes correctas e incorrectas. Usar el resultado de la dimensión para el número de diferentes tipos de respuesta. Para ver el lugar en las solicitudes proceden de, use la dimensión EventType.|Resultado, EventType|
|EventGridLatency|Latencia de la cuadrícula de eventos (versión preliminar)|Milisegundos|Promedio|La latencia media (en milisegundos) entre la entrada de eventos a IoT Hub y entrada de eventos en Event Grid. Este número es un promedio entre todos los tipos de evento. Usar la dimensión EventType para ver la latencia de un tipo específico de evento.|Tipo de evento|
|d2c<br>.twin<br>.read<br>.success|Lecturas gemelas correctas de los dispositivos|Count|Total|El recuento de todas las lecturas gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|d2c<br>.twin<br>.read<br>.failure|Lecturas gemelas con error de los dispositivos|Count|Total|El recuento de todas las lecturas gemelas con error iniciadas por el dispositivo.|Sin dimensiones|
|d2c<br>.twin<br>.read<br>.size|Tamaño de la respuesta de las lecturas gemelas de dispositivos|Bytes|Promedio|El valor medio, mínimo y máximo de todas las lecturas gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|d2c<br>.twin<br>.update<br>.success|Actualizaciones gemelas correctas de los dispositivos|Count|Total|El recuento de todas las actualizaciones gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|d2c<br>.twin<br>.update<br>.failure|Actualizaciones gemelas con error de los dispositivos|Count|Total|El recuento de todas las actualizaciones gemelas con error iniciadas por el dispositivo.|Sin dimensiones|
|d2c<br>.twin<br>.update<br>.size|Tamaño de las actualizaciones gemelas de los dispositivos|Bytes|Promedio|El tamaño medio, mínimo y máximo de todas las actualizaciones gemelas correctas iniciadas por el dispositivo.|Sin dimensiones|
|c2d<br>.methods<br>.success|Invocaciones correctas al método directo|Count|Total|El número de todas las llamadas correctas al método directo.|Sin dimensiones|
|c2d<br>.methods<br>.failure|Invocaciones al método directo con error|Count|Total|El número de todas las llamadas al método directo con error.|Sin dimensiones|
|c2d<br>.methods<br>.requestSize|Tamaño de la solicitud de las invocaciones a métodos directos|Bytes|Promedio|El valor medio, mínimo y máximo de todas las solicitudes correctas de método directo.|Sin dimensiones|
|c2d<br>.methods<br>.responseSize|Tamaño de la respuesta de las invocaciones a métodos directos|Bytes|Promedio|El valor medio, mínimo y máximo de todas las respuestas correctas de método directo.|Sin dimensiones|
|c2d<br>.twin<br>.read<br>.success|Lecturas gemelas correctas del back-end|Count|Total|El recuento de todas las lecturas gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|c2d<br>.twin<br>.read<br>.failure|Lecturas gemelas con error del back-end|Count|Total|El recuento de todas las lecturas gemelas con error iniciadas por el back-end.|Sin dimensiones|
|c2d<br>.twin<br>.read<br>.size|Tamaño de la respuesta de las lecturas gemelas del back-end|Bytes|Promedio|El valor medio, mínimo y máximo de todas las lecturas gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|c2d<br>.twin<br>.update<br>.success|Actualizaciones gemelas correctas del back-end|Count|Total|El recuento de todas las actualizaciones gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|c2d<br>.twin<br>.update<br>.failure|Actualizaciones gemelas con error del back-end|Count|Total|El recuento de todas las actualizaciones gemelas con error iniciadas por el back-end.|Sin dimensiones|
|c2d<br>.twin<br>.update<br>.size|Tamaño de las actualizaciones gemelas del back-end|Bytes|Promedio|El tamaño medio, mínimo y máximo de todas las actualizaciones gemelas correctas iniciadas por el back-end.|Sin dimensiones|
|twinQueries<br>.success|Consultas gemelas correctas|Count|Total|El recuento de todas las consultas gemelas correctas.|Sin dimensiones|
|twinQueries<br>.failure|Consultas gemelas con error|Count|Total|El recuento de todas las consultas gemelas con error.|Sin dimensiones|
|twinQueries<br>.resultSize|Tamaño de resultado de consultas gemelas|Bytes|Promedio|El valor medio, mínimo y máximo del tamaño del resultado de todas las consultas gemelas correctas.|Sin dimensiones|
|trabajos<br>.createTwinUpdateJob<br>.success|Creaciones correctas de trabajos de actualización gemelos|Count|Total|El recuento de todas las creaciones correctas de trabajos de actualización gemelos.|Sin dimensiones|
|trabajos<br>.createTwinUpdateJob<br>.failure|Creaciones con error de trabajos de actualización gemelos|Count|Total|El recuento de todas las creaciones con error de trabajos de actualización gemelos.|Sin dimensiones|
|trabajos<br>.createDirectMethodJob<br>.success|Creaciones correctas de trabajos de invocación de método|Count|Total|El recuento de todas las creaciones correctas de los trabajos de invocación de método directos.|Sin dimensiones|
|trabajos<br>.createDirectMethodJob<br>.failure|Creaciones con error de trabajos de invocación de método|Count|Total|El recuento de todas las creaciones con error de los trabajos de invocación de método directos.|Sin dimensiones|
|trabajos<br>.listJobs<br>.success|Llamadas correctas para enumerar trabajos|Count|Total|El recuento de todas las llamadas correctas para enumerar trabajos.|Sin dimensiones|
|trabajos<br>.listJobs<br>.failure|Llamadas con error para enumerar trabajos|Count|Total|El recuento de todas las llamadas con error para enumerar trabajos.|Sin dimensiones|
|trabajos<br>.cancelJob<br>.success|Cancelaciones de trabajos correctas|Count|Total|El recuento de todas las llamadas correctas para cancelar un trabajo.|Sin dimensiones|
|trabajos<br>.cancelJob<br>.failure|Cancelaciones de trabajos con error|Count|Total|El recuento de todas las llamadas con error para cancelar un trabajo.|Sin dimensiones|
|trabajos<br>.queryJobs<br>.success|Consultas de trabajo correctas|Count|Total|El recuento de todas las llamadas correctas para consultar trabajos.|Sin dimensiones|
|trabajos<br>.queryJobs<br>.failure|Consultas de trabajo con error|Count|Total|El recuento de todas las llamadas con error para consultar trabajos.|Sin dimensiones|
|trabajos<br>.completed|Trabajos completados|Count|Total|El recuento de todos los trabajos completados.|Sin dimensiones|
|trabajos<br>.failed|Trabajos con error|Count|Total|El recuento de todos los trabajos con error.|Sin dimensiones|
|d2c<br>.telemetry<br>.ingress<br>.sendThrottle|Número de errores de limitación|Count|Total|Número de errores de limitación debido a las limitaciones de rendimiento del dispositivo|Sin dimensiones|
|dailyMessage<br>QuotaUsed|Número total de mensajes usados|Count|Promedio|Número de mensajes totales usados hoy. Se trata de un valor acumulado que se restablece en cero a las 00:00 UTC cada día.|Sin dimensiones|
|deviceDataUsage|Uso de datos total del dispositivo|Bytes|Total|Bytes transferidos hacia y desde cualquier dispositivo conectado al Centro de IOT|Sin dimensiones|
|totalDeviceCount|Total de dispositivos (versión preliminar)|Count|Promedio|Número de dispositivos registrados en IoT Hub|Sin dimensiones|
|connected<br>N.º de dispositivos|Dispositivos conectados (versión preliminar)|Count|Promedio|Número de dispositivos conectados a IoT Hub|Sin dimensiones|
|configuraciones|Métricas de configuración|Count|Total|Métricas de las operaciones de configuración|Sin dimensiones|

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha visto la información general sobre las métricas de IoT Hub, siga este vínculo para más información sobre la administración de IoT Hub de Azure:

* [Supervisión de operaciones](iot-hub-operations-monitoring.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
