---
title: Arquitectura de la solución
description: Obtenga información sobre el flujo de información en el servicio Azure Defender para IoT.
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
ms.date: 07/23/2019
ms.author: mlottner
ms.openlocfilehash: 3d26d9e3d686ad7c34e7493dc1413b7a9e7a2f6b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932669"
---
# <a name="azure-defender-for-iot-architecture"></a>Arquitectura de Azure Defender para IoT

En este artículo se explica la arquitectura del sistema funcional de la solución Azure Defender para IoT.

## <a name="defender-for-iot-components"></a>Componentes de Azure Defender para IoT

Azure Defender para IoT consta de los siguientes componentes:

- Integración de IoT Hub
- Agentes de dispositivos (opcional)
- SDK de Send Security Message
- Canalización de Analytics

### <a name="defender-for-iot-workflows"></a>Flujos de trabajo de Azure Defender para IoT

Azure Defender para IoT funciona en uno de dos flujos de trabajo de características: Integrado y mejorado

### <a name="built-in"></a>Integrada

En el modo **Integrado**, Azure Defender para IoT se habilita al elegir la activación de la opción **Seguridad** en IoT Hub. Con la oferta de alertas, recomendaciones y supervisión en tiempo real, el modo integrado proporciona una seguridad sin precedentes y una visibilidad de los dispositivos en un solo paso. El modo de integración no requiere la instalación del agente en ningún dispositivo y usa el análisis avanzado en las actividades registradas para analizar y proteger el dispositivo de campo.

### <a name="enhanced"></a>Mejorada

En el modo **mejorado**, después de activar la opción **Seguridad** en IoT Hub e instalar los agentes de dispositivos de Azure Defender para IoT en los dispositivos, los agentes recopilan, agregan y analizan eventos de seguridad sin procesar de los dispositivos. Los eventos de seguridad sin procesar pueden incluir las conexiones de IP, la creación de procesos, los inicios de sesión de los usuarios y otra información relacionada con la seguridad. Además, los agentes de dispositivo de Azure Defender para IoT controlan la agregación de eventos para ayudar a evitar un alta tasa de rendimiento de la red. Los agentes son altamente personalizables, lo que le permite usarlos para tareas específicas, como enviar solo información importante a los SLA más rápidos, o para agregar información de seguridad exhaustiva y contexto en segmentos más grandes, lo que evita mayores costos de servicio.

![Arquitectura de Defender para IoT](./media/architecture/azure-iot-security-architecture.png)

Los agentes de dispositivos y otra aplicaciones usan el **SDK de Send Security Message de Azure** para enviar información de seguridad a Azure IoT Hub. IoT Hub recoge esta información y la reenvía al servicio de Azure Defender para IoT.

Una vez que el servicio de Azure Defender para IoT está habilitado, además de los datos reenviados, IoT Hub envía también todos los datos internos para el análisis por parte de ASC para IoT. Estos datos incluyen los registros de operaciones de dispositivos a la nube, las identidades de los dispositivos la y configuración del Hub. Toda esta información ayuda a crear la canalización de análisis de Azure Defender para IoT.

La canalización de análisis de Azure Defender para IoT también recibe flujos adicionales de inteligencia de amenazas desde varios orígenes dentro de Microsoft y asociados de Microsoft. La canalización de análisis completa de Azure Defender para IoT funciona con cada configuración del cliente realizada en el servicio (como alertas personalizadas y uso del SDK de Send Security Message).

Con la canalización de análisis, Azure Defender para IoT combina todos los flujos de información para generar alertas y recomendaciones viables. La canalización contiene reglas personalizadas creadas tanto por investigadores de seguridad como por expertos, así como modelos de Machine Learning en busca de desviaciones del comportamiento estándar del dispositivo y análisis de riesgos.

Las recomendaciones y alertas de Azure Defender para IoT (salida de la canalización de análisis) se escriben en el área de trabajo de Log Analytics de cada cliente. Incluir los eventos sin procesar en el área de trabajo, así como las alertas y recomendaciones, permite investigaciones y consultas de profundización con los detalles exactos de las actividades sospechosas detectadas.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre la arquitectura y el flujo de trabajo básicos de la solución Azure Defender para IoT. Para más información sobre los requisitos previos, cómo empezar a trabajar y habilitar la solución de seguridad en IoT Hub, consulte los artículos siguientes:

- [Requisitos previos del servicio](service-prerequisites.md)
- [Introducción](getting-started.md)
- [Configuración de una solución](quickstart-configure-your-solution.md)
- [Habilitar la seguridad en IoT Hub](quickstart-onboard-iot-hub.md)
- [Preguntas más frecuentes de Defender para IoT](resources-frequently-asked-questions.md)
- [Alertas de seguridad de Defender para IoT](concept-security-alerts.md)
