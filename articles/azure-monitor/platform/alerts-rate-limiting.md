---
title: Limitación del número de SMS, correos electrónicos y notificaciones push
description: Conozca cómo limita Azure el número de notificaciones de webhook o notificaciones push de App de Azure, mensajes de correo electrónico o SMS posibles de un grupo de acciones.
author: dkamstra
ms.author: dukek
ms.topic: conceptual
ms.date: 3/12/2018
ms.subservice: alerts
ms.openlocfilehash: 39b9fd8396601d777aa802a147bee3acc4a22cd0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045301"
---
# <a name="rate-limiting-for-voice-sms-emails-azure-app-push-notifications-and-webhook-posts"></a>Limitación del número de SMS, llamadas de voz, correos electrónicos, notificaciones push de App de Azure y publicaciones de webhooks
La limitación de número supone una suspensión de las notificaciones que se produce cuando se envían demasiadas a un número de teléfono, dirección de correo electrónico o dispositivo determinados. La limitación del índice asegura que las alertas son fáciles de administrar y procesar.

Los umbrales del límite de número son:

- **SMS**: no más de 1 SMS cada 5 minutos.
- **Voz**: no más de 1 llamada de voz cada 5 minutos.
- **Correo electrónico**: no más de 100 mensajes de correo electrónico en una hora.
 
  Otras acciones no tienen velocidad limitada.

## <a name="rate-limit-rules"></a>Reglas de limitación de número
- Un número de teléfono o correo electrónico particular se limitan cuando reciben más mensajes de los que permite el umbral.
- Un número de teléfono o correo electrónico puede formar parte de grupos de acciones de muchas suscripciones. La limitación de número se aplica en todas las suscripciones. En cuanto se alcanza el umbral, se aplica incluso si se envían los mensajes desde varias suscripciones.
- Cuando una dirección de correo electrónico está limitada, se envía una notificación adicional del mismo tipo para comunicar la limitación. El correo electrónico indica cuándo expira la limitación de número.

## <a name="next-steps"></a>Pasos siguientes ##
* Más información sobre el [comportamiento de las alertas por SMS](alerts-sms-behavior.md).
* Consulte la [introducción a las alertas del registro de actividad](alerts-overview.md) y aprenda cómo puede recibir alertas.  
* Aprenda a [configurar alertas siempre que se publique una notificación de mantenimiento de un servicio](../../service-health/alerts-activity-log-service-notifications-portal.md).
