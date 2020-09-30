---
title: Conceptos de SMS en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre los conceptos de SMS de Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: e5cfc1e27bae10a1c67e4506afe9db825664785f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945307"
---
# <a name="sms-concepts"></a>Conceptos de SMS

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

Azure Communication Services le permite enviar y recibir mensajes de texto SMS mediante las bibliotecas cliente de SMS de Communication Services. Estas bibliotecas cliente se pueden usar para admitir escenarios de servicio al cliente, recordatorios de citas, autenticación en dos fases y otros requisitos de comunicación en tiempo real. Los SMS de Communication Services permiten enviar mensajes de manera confiable a la vez que exponen las conclusiones sobre la entrega y tasa de respuesta de las campañas.

Entre las principales características de las bibliotecas cliente de SMS de Azure Communication Services se incluyen:

-  Experiencia de instalación **sencilla** para agregar la capacidad de SMS a las aplicaciones.
- Compatibilidad con mensajes de **alta velocidad** mediante números gratuitos para los casos de uso de A2P (aplicación a persona) en Estados Unidos.
- Conversaciones **bidireccionales** para admitir escenarios como la asistencia al cliente, las alertas y los recordatorios de citas.
- **Entrega confiable** con informes de entrega en tiempo real para los mensajes que se envían desde la aplicación.
- **Análisis** realizar un seguimiento de los patrones de uso y la involucración de los clientes.
- Compatibilidad con la administración de **cancelaciones** para detectar automáticamente y respetar las cancelaciones de números gratuitos. Communication Services detecta los mensajes de detención e inicio (STOP y START) y envía las siguientes respuestas predeterminadas a los usuarios finales: 
  - STOP: *"Se canceló correctamente la suscripción a los mensajes de este número. Responda START para volver a suscribirse".*
  - START: *"Se volvió a suscribir correctamente a los mensajes de este número. Responda STOP para cancelar la suscripción".*
  - Los mensajes de detención e inicio (STOP y START) se retransmitirán de vuelta. Azure Communication Services recomienda supervisar e implementar estas cancelaciones para garantizar que no se envíen más mensajes a los destinatarios que hayan optado por no recibir las comunicaciones.


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción al envío de SMS](../../quickstarts/telephony-sms/send.md)

Puede que los siguientes documentos le resulten interesantes:

- Familiarícese con la [biblioteca cliente de SMS](../telephony-sms/sdk-features.md)
- Obtención de un [número de teléfono](../../quickstarts/telephony-sms/get-phone-number.md) compatible con SMS
- [Planeación de una solución de SMS](../telephony-sms/plan-solution.md)
