---
title: Componentes y requisitos previos
description: Detalles de todo lo necesario para empezar a conocer los requisitos previos del servicio Azure Defender para IoT.
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
ms.custom: references_regions
ms.openlocfilehash: 19263f8db58c8d20288d3ae74c24efd85667bc33
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931988"
---
# <a name="azure-defender-for-iot-prerequisites"></a>Requisitos previos de Azure Defender para IoT

En este artículo se proporciona una explicación sobre los diferentes componentes del servicio Defender para IoT, lo que se necesita para comenzar, y explica igualmente los conceptos básicos para ayudar a conocer el servicio.

## <a name="minimum-requirements"></a>Requisitos mínimos

- Nivel estándar de IoT Hub
  - Privilegios de nivel **Propietario** de roles de Azure
- [Área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)
- Azure Security Center (recomendado)
  - El uso de Azure Security Center es una recomendación, no un requisito. Sin Azure Security Center, no podrá ver los demás recursos de Azure en IoT Hub.

## <a name="working-with-defender-for-iot-service"></a>Trabajar con el servicio Defender para IoT

La información y los informes de Defender para IoT están disponibles mediante Azure IoT Hub y Azure Security Center. Para habilitar Defender para IoT en Azure IoT Hub, se necesita una cuenta con privilegios de nivel **Propietario**. Después de habilitar ASC for IoT en IoT Hub, la información de Defender para IoT se muestra como la característica **Seguridad** en Azure IoT Hub y como **IoT** en Azure Security Center.

## <a name="supported-service-regions"></a>Regiones de servicio admitidas

Actualmente, Defender para IoT es compatible con IoT Hub de las siguientes regiones de Azure:

- Centro de EE. UU.
- Este de EE. UU.
- Este de EE. UU. 2
- Centro-Oeste de EE. UU.
- Oeste de EE. UU.
- Oeste de EE. UU. 2
- Centro-Sur de EE. UU.
- Centro-Norte de EE. UU
- Centro de Canadá
- Este de Canadá
- Norte de Europa
- Sur de Brasil
- Centro de Francia
- Oeste de Reino Unido
- Sur de Reino Unido 2
- Oeste de Europa
- Norte de Europa
- Japón Occidental
- Japón Oriental
- Sudeste de Australia
- Este de Australia
- Este de Asia
- Sudeste de Asia
- Centro de Corea del Sur
- Corea del Sur
- Centro de la India
- Sur de la India

Defender para IoT enruta todo el tráfico desde todas las regiones europeas hasta el centro de datos de Oeste de Europa, y desde todas las regiones restantes hasta el centro de datos regional de Centro de EE. UU.

## <a name="wheres-my-iot-hub"></a>¿Dónde está mi IoT Hub?

Antes de comenzar, compruebe la ubicación del IoT Hub para confirmar la disponibilidad del servicio.

1. Abra IoT Hub.
1. Haga clic en **Descripción general**.
1. Compruebe que la ubicación que aparece coincide con una de las [regiones de servicio admitidas](#supported-service-regions).

## <a name="supported-platforms-for-agents"></a>Plataformas compatibles con los agentes

Los agentes de Defender para IoT admiten una lista cada vez mayor de dispositivos y plataformas. Vea la [lista de plataformas admitidas](how-to-deploy-agent.md) para consultar su biblioteca de dispositivos prevista o existente.

## <a name="next-steps"></a>Pasos siguientes

- Lea la [introducción](overview.md) a la seguridad de Azure IoT
- Aprenda a [habilitar el servicio](quickstart-onboard-iot-hub.md)
- Lea las [preguntas más frecuentes de Defender para IoT](resources-frequently-asked-questions.md).
- Explore la [información sobre las alertas de Defender para IoT](concept-security-alerts.md).
