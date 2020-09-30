---
title: 'Inicio rápido: Investigación de alertas de seguridad'
description: Conozca, explore en profundidad e investigue las alertas de seguridad de Defender para IoT en sus dispositivos de IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/30/2020
ms.author: mlottner
ms.openlocfilehash: 172ae82288c2cb948839b69955b9491715eb4690
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945171"
---
# <a name="quickstart-investigate-security-alerts"></a>Inicio rápido: Investigación de alertas de seguridad

La mejor manera de garantizar el cumplimiento y la protección de una solución de IoT es la investigación programada y la corrección de las alertas que emite Defender para IoT.

En esta guía de inicio rápido, exploraremos la información disponible en cada alerta de seguridad de IoT y explicaremos cómo explorar en profundidad y usar los detalles de cada alerta y dispositivo relacionado, para responderlas y corregirlas. 

Comencemos. 


## <a name="investigate-new-security-alerts"></a>Investigación de nuevas alertas de seguridad

La lista de alertas de seguridad de IoT Hub muestra todas las alertas de seguridad agregadas para una instancia de IoT Hub. 

1. En Azure Portal, abra la instancia de **IoT Hub** en el que desee investigar nuevas alertas.
1. En el menú **Seguridad**, seleccione **Alertas**. Se mostrarán todas las alertas de seguridad para la instancia de IoT Hub y se agregará la marca **Nueva** a las alertas de las últimas 24 horas.
:::image type="content" source="media/quickstart/investigate-new-security-alerts.png" alt-text="Investigación de las nuevas alertas de seguridad de IoT con la marca de nueva alerta":::
1. Seleccione y abra cualquier alerta de la lista para abrir sus detalles y explorarlos en profundidad. 

## <a name="security-alert-details"></a>Detalles de alertas de seguridad

Al abrir cada alerta agregada, se muestra su descripción detallada, los pasos de corrección, el identificador de dispositivo de cada dispositivo que desencadenó una alerta, así como la gravedad de la alerta y el acceso a la investigación directa mediante Log Analytics. 

1. Seleccione y abra cualquier alerta de seguridad de la lista **IoT Hub** > **Seguridad** > **Alertas**. 
1. Revise la **descripción**, **gravedad**, **origen de la detección** y **detalles del dispositivo** de la alerta para todos los dispositivos que la emitieron en el período de agregación.
:::image type="content" source="media/quickstart/drill-down-iot-alert-details.png" alt-text="Exploración en profundidad y revisión de los detalles de cada dispositivo en una alerta agregada "::: 
1. Después de revisar los detalles de la alerta, siga las instrucciones del **paso de corrección manual** para corregir o resolver el problema que provocó la alerta. 
:::image type="content" source="media/quickstart/iot-alert-manual-remediation-steps.png" alt-text="Siga los pasos de corrección manual para ayudar a resolver o corregir las alertas de seguridad del dispositivo.":::

1. Si se requiere una investigación adicional, **investigue las alertas en Log Analytics** mediante el vínculo. 
:::image type="content" source="media/quickstart/investigate-iot-alert-log-analytics.png" alt-text="Para investigar más una alerta, use el vínculo para investigar con Log Analytics que aparece en la pantalla":::

## <a name="next-steps"></a>Pasos siguientes

Continúe con el siguiente artículo para obtener más información sobre los tipos de alerta de Defender y las posibles personalizaciones.

> [!div class="nextstepaction"]
> [Descripción de las alertas de seguridad de IoT](concept-security-alerts.md)
