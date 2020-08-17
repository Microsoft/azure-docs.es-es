---
title: Compatibilidad con Azure RTOS
description: Obtenga información sobre la compatibilidad de Azure RTOS en el servicio Azure Security Center para IoT.
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/15/2020
ms.author: mlottner
ms.openlocfilehash: af4579d25b94fc12f67cfc91d0f6a5af4b35c810
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095125"
---
# <a name="azure-security-center-for-iot-security-solution-for-azure-rtos"></a>Solución de seguridad Azure Security Center para IoT para Azure RTOS 

El módulo de seguridad de Azure Security Center para IoT proporciona una solución de seguridad completa para dispositivos Azure RTOS. Azure RTOS se suministra con un módulo de seguridad integrado que cubre amenazas comunes en dispositivos de sistema operativo en tiempo real. 

![Azure RTOS de Azure Security Center para IoT](./media/architecture/azure-rtos-security-monitoring.png)


El módulo de seguridad de Azure Security Center para IoT con Azure RTOS ofrece las siguientes características: 
- Detección de actividad de red malintencionada
- Línea de base del comportamiento de los dispositivos basada en alertas personalizadas
- Mejora de la higiene de seguridad de los dispositivos

### <a name="detection-of-malicious-network-activities"></a>Detección de actividades de red malintencionadas

Se supervisa la actividad de red entrante y saliente de cada dispositivo (protocolos admitidos: TCP, UDP, ICMP en IPv4 e IPv6). Azure Security Center para IoT inspecciona cada una de estas actividades de red en la fuente de inteligencia sobre amenazas de Microsoft. La fuente se actualiza en tiempo real con millones de indicadores de amenazas únicos recopilados en todo el mundo. 

### <a name="device-behavior-baselining-based-on-custom-alerts"></a>Línea de base del comportamiento de los dispositivos basada en alertas personalizadas

La línea de base permite la agrupación en clústeres de dispositivos en grupos de seguridad y la definición del comportamiento esperado de cada grupo. Dado que los dispositivos IoT suelen estar diseñados para funcionar en escenarios bien definidos y limitados, es fácil crear una línea de base que defina su comportamiento esperado mediante un conjunto de parámetros. Cualquier desviación de la línea de base desencadena una alerta. 

### <a name="improve-your-device-security-hygiene"></a>Mejora de la higiene de seguridad de los dispositivos

Al aprovechar la infraestructura recomendada que proporciona Azure Security Center para IoT, adquiere conocimientos e información sobre los problemas de su entorno que afectan y dañan la postura de seguridad de los dispositivos. Una postura de seguridad deficiente de los dispositivos IoT puede permitir que ataques potenciales se realicen correctamente si permanecen sin cambios, ya que la seguridad siempre se mide por el vínculo más débil dentro de cualquier organización. 

## <a name="get-started-protecting-azure-rtos-devices"></a>Introducción a la protección de dispositivos Azure RTOS

- El módulo de seguridad de Azure Security Center para IoT para Azure RTOS se proporciona como una descarga gratuita para sus dispositivos. El servicio en la nube Azure Security Center para IoT está disponible con una versión de prueba de 30 días por suscripción de Azure. Descargue el [módulo de seguridad de Azure Security Center para IoT para Azure RTOS](https://github.com/azure-rtos/iot-security-module-preview) para comenzar. 


## <a name="next-steps"></a>Pasos siguientes

En este artículo ha adquirido información sobre la compatibilidad de Azure Security Center para IoT para Azure RTOS. Para más información sobre cómo empezar a trabajar y habilitar la solución de seguridad en IoT Hub, consulte los artículos siguientes:

- [Requisitos previos del servicio](service-prerequisites.md)
- [Introducción](getting-started.md)
- [Configuración de una solución](quickstart-configure-your-solution.md)
- [Habilitar la seguridad en IoT Hub](quickstart-onboard-iot-hub.md)
- [Preguntas más frecuentes de Azure Security Center para IoT](resources-frequently-asked-questions.md)
- [Alertas de seguridad de Azure Security Center para IoT](concept-security-alerts.md)
