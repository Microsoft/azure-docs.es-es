---
title: Alertas de seguridad personalizables
description: Obtenga información sobre las alertas de seguridad personalizables y la corrección recomendada mediante las características y el servicio de Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/04/2020
ms.author: mlottner
ms.openlocfilehash: 5a5b9182081e267f8e20cb0818202b9cb5ecd904
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932311"
---
# <a name="defender-for-iot-customizable-security-alerts"></a>Alertas de seguridad personalizables de Defender para IoT

Defender para IoT analiza continuamente la solución de IoT mediante análisis avanzados e inteligencia de amenazas para alertarle de cualquier actividad malintencionada.

Se recomienda crear alertas personalizadas en función de su conocimiento sobre el comportamiento esperado del dispositivo. De este modo, se asegura de que las alertas actúan como indicadores eficaces del riesgo potencial en la implementación e infraestructura únicos de la organización.

La siguiente lista de alertas de Defender para IoT se puede definir en función del comportamiento esperado de su instancia de IoT Hub o dispositivos. Para obtener más información acerca de cómo personalizar cada alerta, consulte [cómo crear alertas personalizadas](quickstart-create-custom-alerts.md).

## <a name="iot-hub-alerts-available-for-customization"></a>Alertas de IoT Hub disponibles para la personalización

| severity | Nombre de la alerta | Origen de datos | Descripción | Corrección sugerida|
|---|---|---|---|---|
| Bajo      | Alerta personalizada: el número de mensajes de la nube al dispositivo en el protocolo AMQP está fuera del intervalo permitido.          | IoT Hub     | El número de mensajes de nube a dispositivo (protocolo AMQP) dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.||
| Bajo      | Alerta personalizada: el número de mensajes de la nube al dispositivo rechazados en el protocolo AMQP está fuera del intervalo permitido. | IoT Hub     | El número de mensajes de nube a dispositivo (protocolo AMQP) rechazados por el dispositivo dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.||
| Bajo      | Alerta personalizada: el número de mensajes del dispositivo a la nube en el protocolo AMQP está fuera del intervalo permitido.      | IoT Hub     | La cantidad de mensajes de dispositivo a nube (protocolo AMQP) dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|   |
| Bajo      | Alerta personalizada: el número de invocaciones del método directo está fuera del intervalo permitido. | IoT Hub     | La cantidad de invocaciones del método directo dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.||
| Bajo      | Alerta personalizada: el número de cargas de archivos está fuera del intervalo permitido. | IoT Hub     | La cantidad de cargas de archivos dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.| |
| Bajo      | Alerta personalizada: el número de mensajes de la nube al dispositivo en el protocolo HTTP está fuera del intervalo permitido. | IoT Hub     | La cantidad de mensajes de la nube al dispositivo (protocolo HTTP) en un período de tiempo no está en el intervalo permitido configurado                                  |
| Bajo      | Alerta personalizada: el número de mensajes de la nube al dispositivo rechazados en el protocolo HTTP no se encuentra en el intervalo permitido. | IoT Hub     | La cantidad de mensajes de nube a dispositivo (protocolo HTTP) dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente. |
| Bajo      | Alerta personalizada: el número de mensajes del dispositivo a la nube en el protocolo HTTP está fuera del intervalo permitido. | IoT Hub| La cantidad de mensajes de dispositivo a nube (protocolo HTTP) dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|    |
| Bajo      | Alerta personalizada: el número de mensajes de la nube al dispositivo en el protocolo MQTT está fuera del intervalo permitido. | IoT Hub     | La cantidad de mensajes de nube a dispositivo (protocolo MQTT) dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|   |
| Bajo      | Alerta personalizada: el número de mensajes de la nube al dispositivo rechazados en el protocolo MQTT está fuera del intervalo permitido. | IoT Hub     | La cantidad de mensajes de nube a dispositivo (protocolo MQTT) rechazados por el dispositivo dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente. |
| Bajo      | Alerta personalizada: el número de mensajes del dispositivo a la nube en el protocolo MQTT está fuera del intervalo permitido.          | IoT Hub     | La cantidad de mensajes de dispositivo a nube (protocolo MQTT) dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|
| Bajo      | Alerta personalizada: el número de purgas de la cola de comandos está fuera del intervalo permitido.                               | IoT Hub     | La cantidad de purgas de la cola de comandos dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.||
| Bajo      | Alerta personalizada: el número de actualizaciones de módulos gemelos está fuera del intervalo permitido.                                       | IoT Hub     | La cantidad de actualizaciones de módulos gemelos dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|
| Bajo      | Alerta personalizada: el número de operaciones sin autorizar está fuera del intervalo permitido.  | IoT Hub     | La cantidad de operaciones sin autorizar dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|
|

## <a name="agent-alerts-available-for-customization"></a>Alertas de agente disponibles para la personalización

| severity | Nombre de la alerta | Origen de datos | Descripción | Corrección sugerida|
|---|---|---|---|---|
| Bajo      | Alerta personalizada: el número de conexiones activas está fuera del intervalo permitido.  | Agente       | El número de conexiones activas dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente.|  Investigue los registros del dispositivo. Averigüe dónde se originó la conexión y determine si es inofensiva o malintencionada. Si es malintencionada, quite el posible malware y reconozca el origen. Si es inofensiva, agregue el origen a la lista de conexiones permitidas.  |
| Bajo      | Alerta personalizada: se ha creado una conexión saliente a una dirección IP que no está permitida.                             | Agente       | Se ha creado una conexión saliente a una dirección IP que está fuera de la lista de direcciones IP permitidas. |Investigue los registros del dispositivo. Averigüe dónde se originó la conexión y determine si es inofensiva o malintencionada. Si es malintencionada, quite el posible malware y reconozca el origen. Si es inofensiva, agregue el origen a la lista de direcciones IP permitidas.                        |
| Bajo      | Alerta personalizada: el número de inicios de sesión locales con errores está fuera del intervalo permitido.                               | Agente       | La cantidad de inicios de sesión locales con errores dentro de un período de tiempo específico está fuera del intervalo configurado y permitido actualmente. |   |
| Bajo      | Alerta personalizada: inicio de sesión de un usuario que no está en la lista de usuarios permitidos. | Agente       | Un usuario local no incluido en la lista de usuarios permitidos ha iniciado sesión en el dispositivo.|  Si va a guardar datos sin procesar, vaya a la cuenta de Log Analytics y use los datos para investigar el dispositivo, identifique el origen y, a continuación, corrija la lista de permitidos o bloqueados para esos valores. Si no va a guardar datos sin procesar, vaya al dispositivo y corrija la lista de permitidos o bloqueados para esos valores.|
| Bajo      | Alerta personalizada: se ha ejecutado un proceso no permitido. | Agente       | Se ha ejecutado en el dispositivo un proceso que no se permite. |Si va a guardar datos sin procesar, vaya a la cuenta de Log Analytics y use los datos para investigar el dispositivo, identifique el origen y, a continuación, corrija la lista de permitidos o bloqueados para esos valores. Si no va a guardar datos sin procesar, vaya al dispositivo y corrija la lista de permitidos o bloqueados para esos valores.  |
|

## <a name="next-steps"></a>Pasos siguientes

- Averigüe cómo puede [personalizar una alerta](quickstart-create-custom-alerts.md)
- [Información general](overview.md) del servicio Defender para IoT
- Aprenda a [acceder a los datos de seguridad](how-to-security-data-access.md)
- Más información sobre cómo [investigar un dispositivo](how-to-investigate-device.md)
