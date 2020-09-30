---
title: Provisión de detalles de contacto de seguridad en Azure Security Center | Microsoft Docs
description: En este documento se muestra cómo proporcionar detalles de contacto de seguridad en Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 26b5dcb4-ce3f-4f22-8d56-d2bf743cfc90
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/11/2020
ms.author: memildin
ms.openlocfilehash: 9fbd63e1b46b837350be720fadf68777927f9bff
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904756"
---
# <a name="set-up-email-notifications-for-security-alerts"></a>Configuración de notificaciones de alertas de seguridad por correo electrónico 

Para garantizar que las personas adecuadas de la organización reciban notificaciones sobre las alertas de seguridad de su entorno, escriba sus direcciones de correo electrónico en la página de configuración **Notificaciones por correo electrónico**.

Al configurar las notificaciones, puede configurar los correos electrónicos para que se envíen a usuarios específicos o a cualquier persona con un rol de Azure específico para una suscripción. 

Para evitar un exceso de alertas, Security Center limita el volumen de correos salientes. Para cada suscripción, Security Center envía:

- un máximo de **cuatro** correos electrónicos al día para alertas de **gravedad alta**
- un máximo de **dos** correos electrónicos al día para alertas de **gravedad media**
- un máximo de **un** correo electrónico al día para alertas de **gravedad baja**


:::image type="content" source="./media/security-center-provide-security-contacts/email-notification-settings.png" alt-text="Configuración de los detalles de los contactos que recibirán correos electrónicos sobre las alertas de seguridad" :::

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Gratuito|
|Roles y permisos necesarios:|**Administrador de seguridad**<br>**Propietario de la suscripción** |
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov (parcial)<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos|
|||


## <a name="set-up-email-notifications-for-alerts"></a>Configuración de notificaciones de correo electrónico para alertas <a name="email"></a>

Puede enviar notificaciones por correo electrónico a individuos o a todos los usuarios con roles de Azure específicos.

1. En el área **Precios y configuración** de Security Center, seleccione la suscripción correspondiente y **Notificaciones de correo electrónico**.

1. Defina los destinatarios de las notificaciones:

    - En la lista desplegable, seleccione entre los roles disponibles.
    - También puede escribir direcciones de correo electrónico separadas por comas. No hay ningún límite en el número de direcciones de correo electrónico que se pueden escribir.

1. Seleccione **Guardar** para aplicar la información de contacto de seguridad a su suscripción.


## <a name="see-also"></a>Consulte también
Para más información sobre las alertas de seguridad, consulte los siguientes recursos:

* [Alertas de seguridad: una guía de referencia](alerts-reference.md): obtenga información sobre las alertas de seguridad que puede ver en el módulo de protección contra amenazas de Azure Security Center.
* [Administración y respuesta a las alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md): aprenda a administrar y responder a alertas de seguridad.
* [Automatización de flujos de trabajo](workflow-automation.md): automatice respuestas a alertas con lógica de notificación personalizada