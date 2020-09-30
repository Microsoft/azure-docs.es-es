---
title: Módulo de seguridad para IoT Edge de Defender para IoT
description: Conozca la arquitectura y las funcionalidades del módulo de seguridad para IoT Edge de Azure Defender para IoT.
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
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 1933e60892430b907e070ea04f39d8acc86ddac2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931989"
---
# <a name="azure-defender-for-iot-edge-security-module"></a>Módulo de seguridad de Azure Defender para IoT Edge

[Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/) proporciona capacidades de enorme eficacia para administrar y procesar flujos de trabajo empresariales en el perímetro.
El papel esencial que IoT Edge desempeña en entornos de IoT lo hace especialmente atrayente para atacantes malintencionados.

El módulo de seguridad de Defender para IoT proporciona una solución de seguridad completa para dispositivos IoT Edge.
El módulo de Defender para IoT recopila, agrega y analiza los datos de seguridad sin procesar del sistema operativo y del sistema de contenedores para generar alertas y recomendaciones de seguridad que requieren acción.

Al igual que los agentes de seguridad de Defender para IoT de los dispositivos IoT, el módulo de Defender para IoT Edge se puede personalizar profundamente a través de su módulo gemelo.
Vea [Configurar agentes de seguridad](how-to-agent-configuration.md) para obtener más información.

El módulo de seguridad de Defender para IoT para IoT Edge ofrece las siguientes características:

- Recopila eventos de seguridad sin procesar del sistema operativo subyacente (Linux) y de los sistemas de contenedor de IoT Edge.

  Consulte [Configuración del agente de Defender para IoT](how-to-agent-configuration.md) para obtener más información sobre los recopiladores de datos de seguridad disponibles.

- Análisis de manifiestos de implementación de IoT Edge.

- Agrega eventos de seguridad sin procesar a los mensajes enviados a través de [IoT Edge Hub](https://docs.microsoft.com/azure/iot-edge/iot-edge-runtime#iot-edge-hub).

- Quita la configuración usando el módulo de seguridad gemelo.

  Consulte [Configuración de un agente de Defender para IoT](how-to-agent-configuration.md) para obtener más información.

El módulo de seguridad de Defender para IoT para IoT Edge se ejecuta en un modo con privilegios en IoT Edge.
El modo con privilegios es necesario para permitir que el módulo supervise el sistema operativo y otros módulos de IoT Edge.

## <a name="module-supported-platforms"></a>Plataformas admitidas por el módulo

El módulo de seguridad de Defender para IoT para IoT Edge solo está disponible actualmente en Linux.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha conocido la arquitectura y las funcionalidades del módulo de seguridad de Defender para IoT para IoT Edge.

Para seguir obteniendo información sobre la implementación de Defender para IoT, lea los siguientes artículos:

- Implementar el [módulo de seguridad de IoT Edge](how-to-deploy-edge.md)
- Obtenga información sobre cómo [configurar el módulo de seguridad](how-to-agent-configuration.md)
- Repase los [requisitos previos de servicio](service-prerequisites.md) de Defender para IoT
- Obtenga información sobre cómo [habilitar el servicio Defender para IoT en IoT Hub](quickstart-onboard-iot-hub.md)
- Obtenga más información sobre el servicio en las [preguntas más frecuentes de Defender para IoT](resources-frequently-asked-questions.md)
