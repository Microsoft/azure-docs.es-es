---
title: Agregación de datos
description: Obtenga más información sobre la agregación de eventos de Defender para IoT.
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
ms.date: 09/26/2019
ms.author: mlottner
ms.openlocfilehash: aec750d246ce99fa65431e23ef68e70418db0017
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932656"
---
# <a name="defender-for-iot-event-aggregation"></a>Agregación de eventos de Defender para IoT

Los agentes de seguridad de Defender para IoT recopilan datos y eventos del sistema del dispositivo local, y envían estos datos a la nube de Azure para su procesamiento y análisis. El agente de seguridad recopila muchos tipos de eventos de dispositivo, incluidos los eventos de nuevo proceso y nueva conexión. En un dispositivo se pueden producir de forma legítima eventos tanto de nuevo proceso como de nueva conexión en un segundo, y si bien es importante para una seguridad sólida y completa, el número de mensajes que obliga a enviar a los agentes de seguridad puede alcanzar o superar rápidamente los límites de cuota y costo de su instancia de IoT Hub. Sin embargo, estos eventos contienen información de seguridad muy valiosa que es crucial para proteger el dispositivo.

Para reducir la cuota y los costos adicionales, a la vez que se mantienen los dispositivos protegidos, los agentes de Defender para IoT agregan estos tipos de eventos.

La agregación de eventos está **activada** de forma predeterminada y, aunque no se recomienda, puede **desactivarse** manualmente en cualquier momento.

La agregación está disponible actualmente para los siguientes tipos de eventos:

* ProcessCreate
* ConnectionCreate
* ProcessTerminate (solo Windows)

## <a name="how-does-event-aggregation-work"></a>¿Cómo funciona la agregación de eventos?

Cuando la agregación de eventos se deja **activada**, los agentes de Defender para IoT agregan eventos para el período de intervalo o ventana de tiempo.
Una vez transcurrido el período de intervalo, el agente envía los eventos agregados a la nube de Azure para su posterior análisis.
Los eventos agregados se almacenan en memoria hasta que se envían a la nube de Azure.

Para reducir la superficie de memoria del agente, cada vez que este recopila un evento idéntico a otro que ya está guardado en la memoria, el agente aumenta el número de llamadas de este evento específico. Cuando ha transcurrido la ventana de tiempo de agregación, el agente envía el número de llamadas de cada tipo específico de evento que se ha producido. La agregación de eventos es simplemente la agregación de los números de llamadas de cada tipo de evento recopilado.

Los eventos se consideran idénticos solo cuando se cumplen las condiciones siguientes:

* Eventos ProcessCreate: cuando **commandLine**, **executable**, **username** y **userid** son idénticos
* Eventos ConnectionCreate: cuando **commandLine**, **userId**, **direction**, **local address**, **remote address**, **protocol y **destination port** son idénticos
* Eventos de ProcessTerminate: cuando el **ejecutable** y el **estado de salida** son idénticos

### <a name="working-with-aggregated-events"></a>Trabajo con eventos agregados

Durante la agregación, se descartan las propiedades de evento que no se están agregadas y aparecen en el análisis de registros con un valor de 0.

* Eventos ProcessCreate: **processId**  y **parentProcessId** se establecen en 0
* Eventos ConnectionCreate: **processId** y **puerto de origen** se establecen en 0

## <a name="event-aggregation-based-alerts"></a>Alertas basadas en la agregación de eventos

Después del análisis, Defender para IoT crea alertas de seguridad para los eventos agregados sospechosos. Las alertas que se crean a partir de eventos agregados aparecen solo una vez para cada evento agregado.

La hora de inicio de la agregación, la hora de finalización y el número de llamadas para cada evento se registran en el campo **ExtraDetails** del evento en Log Analytics para su uso durante las investigaciones.

Cada evento agregado representa un período de 24 horas de alertas recopiladas. Con el menú de opciones de evento de la parte superior izquierda de cada evento, puede **descartar** cada evento agregado individual.

## <a name="event-aggregation-twin-configuration"></a>Configuración gemela de agregación de eventos

Realice cambios en la configuración de la agregación de eventos de Defender para IoT dentro del [objeto de configuración del agente](how-to-agent-configuration.md) de la identidad gemela del módulo **azureiotsecurity**.

| Nombre de la configuración | Valores posibles | Detalles | Observaciones |
|:-----------|:---------------|:--------|:--------|
| aggregationEnabledProcessCreate | boolean | Habilitar o deshabilitar la agregación de eventos para los eventos de creación de procesos |
| aggregationIntervalProcessCreate | Cadena de intervalo de tiempo ISO8601 | Intervalo de agregación para eventos de creación de proceso |
| aggregationEnabledConnectionCreate | boolean| Habilitar o deshabilitar la agregación de eventos para los eventos de creación de conexión |
| aggregationIntervalConnectionCreate | Cadena de intervalo de tiempo ISO8601 | Intervalo de agregación para eventos de creación de conexión |
| aggregationEnabledProcessTerminate | boolean | Habilitar o deshabilitar la agregación de eventos para los eventos de finalización de proceso | Solo Windows|
| aggregationIntervalProcessTerminate | Cadena de intervalo de tiempo ISO8601 | Intervalo de agregación para eventos de finalización de proceso | Solo Windows|
|

## <a name="default-configurations-settings"></a>Valores de configuración predeterminados

| Nombre de la configuración | Valores predeterminados |
|:-----------|:---------------|
| aggregationEnabledProcessCreate | true |
| aggregationIntervalProcessCreate | "PT1H"|
| aggregationEnabledConnectionCreate | true |
| aggregationIntervalConnectionCreate | "PT1H"|
| aggregationEnabledProcessTerminate | true |
| aggregationIntervalProcessTerminate | "PT1H"|
|

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre la agregación de agentes a Defender para IoT y las opciones de configuración de evento disponibles.

Para seguir obteniendo información sobre la implementación de Defender para IoT, lea los siguientes artículos:

- Conozca los [métodos de autenticación del agente de seguridad](concept-security-agent-authentication-methods.md)
- Seleccione e implemente un [agente de seguridad](how-to-deploy-agent.md)
- Repase los [requisitos previos de servicio](service-prerequisites.md) de Defender para IoT.
- Obtenga información sobre cómo [habilitar el servicio Defender para IoT en IoT Hub](quickstart-onboard-iot-hub.md).
- Obtenga información sobre el servicio en las [preguntas más frecuentes de Defender para IoT](resources-frequently-asked-questions.md).
