---
title: Arquitectura del agente de seguridad
description: Conozca la arquitectura de agente de seguridad de los agentes que se usan en el servicio Azure Defender para IoT.
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
ms.date: 07/26/2019
ms.author: mlottner
ms.openlocfilehash: 64d90b4d9dc3efbe877230bbc20780b1c4f2d213
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932348"
---
# <a name="security-agent-reference-architecture"></a>Arquitectura de referencia del agente de seguridad

Azure Defender para IoT proporciona una arquitectura de referencia a agentes de seguridad que registran, procesan, agregan y envían datos de seguridad mediante IoT Hub.

Los agentes de seguridad están diseñados para funcionar en un entorno de IoT restringido, y son tremendamente personalizables en cuanto a todo el valor que aportan en comparación con los recursos que consumen.

Los agentes de seguridad permiten realizar lo siguiente:

- Recopilar eventos de seguridad sin procesar del sistema operativo subyacente (Linux o Windows). Vea [Configuración del agente de Defender para IoT](how-to-agent-configuration.md) para obtener más información sobre los recopiladores de datos de seguridad disponibles.

- Agregar eventos de seguridad sin procesar a los mensajes enviados mediante IoT Hub.

- Autenticarse con la identidad del dispositivo existente, o con una identidad de módulo dedicada. Vea [Métodos de autenticación del agente de seguridad](concept-security-agent-authentication-methods.md) para obtener más información.

- Establecer configuraciones de forma remota mediante el módulo gemelo **azureiotsecurity**. Vea [Configuración de un agente de Defender para IoT](how-to-agent-configuration.md) para obtener más información.

Los agentes de seguridad de Defender para IoT se desarrollan como proyectos de código abierto y están disponibles en GitHub:

- [Agente basado en C de Defender para IoT](https://github.com/Azure/Azure-IoT-Security-Agent-C)
- [Agente basado en C# de Defender para IoT](https://github.com/Azure/Azure-IoT-Security-Agent-CS)

## <a name="agent-supported-platforms"></a>Plataformas compatibles con el agente

Defender para IoT ofrece diferentes instaladores de agente de 32 y 64 bits de Windows, y lo mismo para 32 bits y 64 bits de Linux. Asegúrese de que tiene el instalador de agente correcto para cada uno de los dispositivos, según la siguiente tabla:

| Architecture | Linux | Windows |    Detalles|
|----------|----------------------------------------------|-------------|-------------------------------------------|
| 32 bits  | C  | C#  ||
| 64 bits  | C# o C           | C#      | Se recomienda usar el agente de C para dispositivos con recursos de dispositivo mínimos o más restringidos.|
|

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha conocido la arquitectura de agentes de seguridad de Defender para IoT, así como los instaladores disponibles.

Para seguir obteniendo información sobre la implementación de Defender para IoT, lea los siguientes artículos:

- Información acerca de los [métodos de autenticación del agente de seguridad](concept-security-agent-authentication-methods.md)
- Seleccione e implemente un [agente de seguridad](how-to-deploy-agent.md)
- Repase los [requisitos previos de servicio](service-prerequisites.md) de Defender para IoT.
- Obtenga información sobre cómo [habilitar el servicio Defender para IoT en IoT Hub](quickstart-onboard-iot-hub.md).
- Obtenga información sobre el servicio en las [preguntas más frecuentes de Defender para IoT](resources-frequently-asked-questions.md).
