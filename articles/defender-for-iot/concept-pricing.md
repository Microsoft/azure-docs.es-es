---
title: Precios y costos asociados
description: Sepa qué costos conlleva Defender para IoT y cómo mantenerlos bajo control.
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
ms.date: 09/04/2020
ms.author: mlottner
ms.openlocfilehash: 24ae6c4014948639aa737a0d2d88ec15f98a7cb4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932310"
---
# <a name="pricing-and-associated-costs"></a>Precios y costos asociados

En este artículo se explica el modelo de precios de Defender para IoT, se resumen todos los costos asociados y se explica cómo administrarlos.

## <a name="pricing"></a>Precios

El modelo de precios de Defender para IoT está formado por dos partes, y se factura una vez que haya un IoT Hub [habilitado](quickstart-onboard-iot-hub.md) en Defender para IoT:

- Coste por dispositivo: capacidades de seguridad integradas en función de los análisis de registros de IoT Hub.

- Coste por mensaje: capacidades de seguridad mejoradas en función de los mensajes de seguridad de dispositivos IoT Edge u hoja.

Para obtener más información, vea la [página de precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="associated-costs"></a>Costes asociados

Defender para IoT tiene costos asociados que no forman parte de los precios directos:

- Costes de almacenamiento de Log Analytics

Puede reducir estos costos asociados si decide no utilizar determinadas características. Opte por no cambiar su configuración.

Para cambiar la configuración:

1. Abra IoT Hub.

1. En **Seguridad**, haga clic en **Configuración**.

1. Haga clic en **Recopilación de datos**.

En la siguiente tabla se proporciona un resumen de los costes asociados y lo que cada opción conlleva.

| Opción | Uso | Comentario |
| --- | --- | --- |
| **Almacenamiento de Log Analytics** |  |
| Alertas y recomendaciones de dispositivo| Alertas y recomendaciones de dispositivo generadas por el servicio | No opcional. |
| Datos de seguridad sin procesar| Datos de seguridad sin procesar de dispositivos IoT recopilados por agentes de seguridad | Se deshabilita el _almacenamiento de eventos de seguridad de dispositivos sin procesar_. |
|

>[!Important]
> Dejar de participar tiene graves implicaciones en Defender para IoT en relación con la disponibilidad de las características de seguridad.

| Si se deshabilita | Implicaciones |
| --- | --- |
| _Recopilación de metadatos gemelos_ | Se deshabilitan las [alertas personalizadas](quickstart-create-custom-alerts.md). |
| | Se deshabilitan las recomendaciones de manifiesto de IoT Edge. |
| | Se deshabilitan las alertas y recomendaciones basadas en identidades de dispositivo. |
| _Almacenamiento de eventos de seguridad de dispositivos sin procesar_ | No hay disponible información detallada sobre las recomendaciones de línea base de sistema operativo de dispositivo. |
| | No hay disponible información detallada sobre las investigaciones de [alertas](concept-security-alerts.md) y [recomendaciones](concept-recommendations.md). |
|

## <a name="see-also"></a>Consulte también

- Acceso a los [datos de seguridad sin procesar](how-to-security-data-access.md)
- [Investigación de un dispositivo](how-to-investigate-device.md)
- Conozca y explore las [recomendaciones de seguridad](concept-recommendations.md)
- Conozca y explore las [alertas de seguridad](concept-security-alerts.md)
