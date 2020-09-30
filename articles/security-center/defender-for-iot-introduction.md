---
title: Azure Defender para IoT
description: Más información sobre Azure Defender para IoT
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 67bb01092cb6171ae56b8ab8b308e0c2ea3bece0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932370"
---
# <a name="introduction-to-azure-defender-for-iot"></a>Introducción a Azure Defender para IoT

Unifique la administración de seguridad y habilite el análisis y la detección de amenazas de un extremo a otro en todas las cargas de trabajo de la nube híbrida y en la solución Azure IoT.

Azure Defender para IoT ofrece:

- **Detección de recursos y asignación de red**, incluidos detalles como el fabricante del dispositivo, el tipo de dispositivo y cómo se comunican los dispositivos en la red
- **Administración de vulnerabilidades**, incluida la información sobre CVE, los puertos abiertos y las conexiones a Internet no autorizadas
- **Supervisión continua de amenazas**, con alertas en tiempo real que indican cualquier actividad anómala o no autorizada, como ataques dirigidos o malware

Puede encontrar todos los detalles en la [documentación dedicada](https://docs.microsoft.com/azure/asc-for-iot/overview).

## <a name="availability"></a>Disponibilidad
|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Requiere [Azure Defender](security-center-pricing.md).|
|Roles y permisos necesarios:|Permisos de escritura en los NSG de la máquina|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="what-devices-can-azure-defender-for-iot-secure"></a>¿Qué dispositivos puede proteger Azure Defender para IoT?
Al combinar las funcionalidades de Security Center, Azure Defender y la tecnología sin agente de CyberX, puede proteger:

- **Dispositivos Greenfield** conectados a través de IoT Hub. Esto permite a las organizaciones acelerar sus iniciativas de IoT mediante la protección de dispositivos modernos y nativos de Internet y dispositivos ICS/SCADA tradicionales con una única solución unificada.
    - Incorpore nuevos dispositivos y aplique directivas de seguridad a las cargas de trabajo (dispositivos Leaf, Microsoft Edge e IOT Hub) para garantizar el cumplimiento de los estándares de seguridad y una posición de seguridad mejorada.

- Los **dispositivos Brownfield no administrados** utilizados en entornos de tecnología operativa (TO), como fabricación, creación de sistemas de administración (BMS), ciencias biológicas, servicios de energía y agua, petróleo y gas y logística. 
    - Para proteger los dispositivos no administrados, Azure Defender para IoT usa un sensor local para el análisis de tráfico de red pasivo y no invasivo (NTA) que no tiene ningún impacto en el rendimiento de los entornos de OT. 
    - También incorpora un conocimiento exhaustivo de los protocolos IoT/OT especializados, combinado con el aprendizaje automático y el análisis de comportamiento patentado que reconoce IoT/OT, para eliminar la necesidad de configurar cualquier regla o firma, lo que permite realizar las implementaciones típicas en cuestión de minutos u horas, en lugar de en días o semanas. 


## <a name="azure-defender-for-iot-integration-with-azure-sentinel"></a>Integración de Azure Defender para IoT con Azure Sentinel
Para habilitar la supervisión y la gobernanza de la seguridad unificada de TI/OT, Azure Defender para IoT se integra de forma nativa con [Azure Sentinel](../sentinel/overview.md).

Los equipos de SecOps pueden:

- Detectar las amenazas de IoT/OT y responder a ellas rápidamente mediante cuadernos de estrategias SOAR específicos de OT incluidos con Sentinel
- Beneficiarse de la inteligencia de amenazas específica de IoT/OT actualizada constantemente, proporcionada por la sección 52, el equipo de inteligencia de amenazas de IoT/OT interno de Microsoft
- Integrar Azure Defender para IoT con flujos de trabajo existentes de SOC y herramientas de seguridad de terceros, como Splunk, IBM QRadar y ServiceNow


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para IoT en Azure Security Center. Para más información, consulte:

- [Presentación de Azure Security Center for IoT](../asc-for-iot/overview.md)
