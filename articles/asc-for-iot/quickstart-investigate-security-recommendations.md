---
title: 'Inicio rápido: Investigación de recomendaciones de seguridad'
description: Investigue las recomendaciones de seguridad con el servicio de seguridad Azure Security Center para IoT.
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/12/2020
ms.author: mlottner
ms.openlocfilehash: aa241a9108be32f88357065732c6f283d312be6c
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88144049"
---
# <a name="quickstart-investigate-security-recommendations"></a>Inicio rápido: Investigación de recomendaciones de seguridad


El análisis y la mitigación puntual de las recomendaciones por parte de Azure Security Center para IoT es la manera ideal de mejorar la posición de seguridad y reducir la superficie expuesta a ataques de la solución de IoT.

En este inicio rápido, se explorará la información disponible en cada recomendación de seguridad de IoT y se explicará cómo explorar en profundidad y usar los detalles de cada recomendación y los dispositivos relacionados, a fin de reducir el riesgo.

Comencemos.

## <a name="investigate-new-recommendations"></a>Investigación de nuevas recomendaciones

En la lista de recomendaciones de IoT Hub se muestran todas las recomendaciones de seguridad agregadas para el centro de IoT.

1.  En Azure Portal, abra el  **centro de IoT**  en el que quiere investigar nuevas recomendaciones.

1.  En el menú  **Seguridad** , seleccione  **Recomendaciones**. Se mostrarán todas las recomendaciones de seguridad para el centro de IoT y las recomendaciones con una marca  **Nueva**  indican las recomendaciones de las últimas 24 horas. 

    [ ![Investigación de las recomendaciones de seguridad con ASC para IoT](media/quickstart/investigate-security-recommendations-inline.png)](media/quickstart/investigate-security-recommendations-expanded.png#lightbox)


1.  Seleccione y abra cualquier recomendación de la lista para ver sus detalles y explorar en profundidad aspectos específicos.

## <a name="security-recommendation-details"></a>Detalles de la recomendación de seguridad

Abra cada recomendación agregada para mostrar su descripción detallada, los pasos de corrección y el identificador de cada dispositivo que desencadenó una recomendación. También se muestra la gravedad de la recomendación y el acceso a la investigación directa mediante Log Analytics.

1.  Seleccione y abra una recomendación de seguridad en la lista  **IoT Hub** \> **Seguridad** \> **Recomendaciones** .

1.  Revise la **descripción** de la recomendación, la  **gravedad** y los  **detalles**  de todos los dispositivos que emitieron esta recomendación en el período de agregación. 

1.  Después de revisar los pormenores de las recomendaciones, use las instrucciones del  **paso de corrección manual** para solucionar y resolver el problema que provocó la recomendación. 

    [ ![Corrección de recomendaciones de seguridad con ASC para IoT](media/quickstart/remediate-security-recommendations-inline.png)](media/quickstart/remediate-security-recommendations-expanded.png#lightbox)


1.  Explore los detalles de la recomendación de un dispositivo específico seleccionando el dispositivo deseado en la página de exploración en profundidad.

    [ ![Investigación de las recomendaciones de seguridad específicas de un dispositivo con ASC para IoT](media/quickstart/explore-security-recommendation-detail-inline.png)](media/quickstart/explore-security-recommendation-detail-expanded.png#lightbox)


1.  Si se requiere una investigación más a fondo,  **investigue la recomendación en Log Analytics**  mediante el vínculo. 


## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para aprender a crear alertas personalizadas...

> [!div class="nextstepaction"]
> [Creación de alertas personalizadas](quickstart-create-custom-alerts.md)
