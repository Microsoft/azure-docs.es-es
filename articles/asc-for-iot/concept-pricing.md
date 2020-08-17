---
title: Precios y costos asociados
description: Sepa qué costos conlleva Azure Security Center para IoT y cómo mantenerlos a raya.
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: ef839708-4574-4a40-bc45-07005f8e9daf
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/27/2019
ms.author: mlottner
ms.openlocfilehash: 98afd1eb1f8a73c2d6dd3f293a07cdb70780d238
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87004773"
---
# <a name="pricing-and-associated-costs"></a>Precios y costos asociados

En este artículo se explica el modelo de precios de Azure Security Center para IoT, se resumen todos los costos asociados y se explica cómo administrarlos.

## <a name="pricing"></a>Precios

El modelo de precios de Azure Security Center para IoT está formado por dos partes, y se factura una vez que haya un IoT Hub [habilitado](quickstart-onboard-iot-hub.md) en Azure Security Center para IoT:

- Coste por dispositivo: capacidades de seguridad integradas en función de los análisis de registros de IoT Hub.

- Coste por mensaje: capacidades de seguridad mejoradas en función de los mensajes de seguridad de dispositivos IoT Edge u hoja.

Para obtener más información, vea la [página de precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="associated-costs"></a>Costes asociados

Azure Security Center para IoT tiene costos asociados que no forman parte de los precios directos:

- Costes de almacenamiento de Log Analytics

Puede reducir estos costos asociados si decide no utilizar determinadas características. Opte por no cambiar su configuración.

Para cambiar la configuración:

1. Abra IoT Hub.

1. En **Security** (Seguridad), haga clic en **Overview** (Introducción).

1. Haga clic en **Configuración**.

En la siguiente tabla se proporciona un resumen de los costes asociados y lo que cada opción conlleva.

| Opción | Uso | Comentario |
| --- | --- | --- |
| **Almacenamiento de Log Analytics** |  |
| Alertas y recomendaciones de dispositivo| Alertas y recomendaciones de dispositivo generadas por el servicio | No opcional. |
| Datos de seguridad sin procesar| Datos de seguridad sin procesar de dispositivos IoT recopilados por agentes de seguridad | Se deshabilita el _almacenamiento de eventos de seguridad de dispositivos sin procesar_. |
|

>[!Important]
> Dejar de participar tiene graves implicaciones en Azure Security Center para IoT en relación con la disponibilidad de las características de seguridad.

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
