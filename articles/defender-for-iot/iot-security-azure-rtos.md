---
title: Información general del módulo de seguridad para Azure RTOS
description: Obtenga más información sobre la compatibilidad y la implementación del módulo de seguridad para Azure RTOS, integrado en el servicio Defender para IoT.
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
ms.date: 09/07/2020
ms.author: mlottner
ms.openlocfilehash: 0cfd1e0ce16008c6f7fd128d561ad361bcc53b87
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932298"
---
# <a name="overview-defender-for-iot-security-module-for-azure-rtos-preview"></a>Introducción: Módulo de seguridad para Azure RTOS de Defender para IoT

El módulo de seguridad de Defender para IoT para RTOS proporciona una solución de seguridad completa para dispositivos Azure RTOS. Azure RTOS ahora se incluye con el módulo de seguridad de IoT de Azure integrado y proporciona cobertura para amenazas comunes y posibles actividades malintencionadas en los dispositivos del sistema operativo en tiempo real. 

![Azure RTOS en Defender para IoT](./media/architecture/azure-rtos-security-monitoring.png)


El módulo de seguridad para Azure RTOS ofrece las siguientes características: 
- Detección de actividad de red malintencionada
- Línea de base del comportamiento de los dispositivos basada en alertas personalizadas
- Mejora de la higiene de seguridad de los dispositivos

## <a name="detection-of-malicious-network-activities"></a>Detección de actividades de red malintencionadas

Se supervisa la actividad de red entrante y saliente de cada dispositivo (protocolos admitidos: TCP, UDP, ICMP en IPv4 e IPv6). Defender para IoT inspecciona cada una de estas actividades de red en la fuente de inteligencia sobre amenazas de Microsoft. La fuente se actualiza en tiempo real con millones de indicadores de amenazas únicos recopilados en todo el mundo. 

## <a name="device-behavior-baselining-based-on-custom-alerts"></a>Línea de base del comportamiento de los dispositivos basada en alertas personalizadas

La línea de base permite la agrupación en clústeres de dispositivos en grupos de seguridad y la definición del comportamiento esperado de cada grupo. Dado que los dispositivos IoT suelen estar diseñados para funcionar en escenarios bien definidos y limitados, es fácil crear una línea de base que defina su comportamiento esperado mediante un conjunto de parámetros. Cualquier desviación de la línea de base desencadena una alerta. 

## <a name="improve-your-device-security-hygiene"></a>Mejora de la higiene de seguridad de los dispositivos

Al aprovechar la infraestructura recomendada que proporciona Defender para IoT, adquiere conocimientos e información sobre los problemas de su entorno que afectan y dañan la postura de seguridad de los dispositivos. Una postura de seguridad deficiente de los dispositivos IoT puede permitir que ataques potenciales se realicen correctamente si permanecen sin cambios, ya que la seguridad siempre se mide por el vínculo más débil dentro de cualquier organización. 

## <a name="get-started-protecting-azure-rtos-devices"></a>Introducción a la protección de dispositivos Azure RTOS

El módulo de seguridad para Azure RTOS se proporciona como una descarga gratuita para los dispositivos. El servicio en la nube Defender para IoT está disponible con una versión de prueba de 30 días por suscripción de Azure. Para comenzar, descargue el [módulo de seguridad para Azure RTOS](https://github.com/azure-rtos/iot-security-module-preview). 


## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió sobre el servicio del módulo de seguridad para Azure RTOS. Para más información sobre el módulo de seguridad y empezar a usarlo, consulte los siguientes artículos:

- [Conceptos del módulo de seguridad de IoT de Azure RTOS](concept-rtos-security-module.md)
- [Inicio rápido: Módulo de seguridad de IoT de Azure RTOS](quickstart-azure-rtos-security-module.md)
