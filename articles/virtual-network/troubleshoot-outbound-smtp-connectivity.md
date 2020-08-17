---
title: Solución de problemas de conectividad SMTP saliente en Azure | Microsoft Docs
description: Sepa cuál es el método recomendado para enviar correo electrónico y cómo solucionar problemas de conectividad SMTP saliente en Azure.
services: virtual-network
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/20/2018
ms.author: genli
ms.openlocfilehash: e58f498ca254378354fb0e9e21d04fe9969588c9
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265269"
---
# <a name="troubleshoot-outbound-smtp-connectivity-issues-in-azure"></a>Solución de problemas de conectividad SMTP saliente en Azure

A partir del 15 de noviembre de 2017, los mensajes de correo electrónico salientes que se envían directamente a dominios externos (como outlook.com y gmail.com) desde una máquina virtual (VM) solo están disponibles a ciertos tipos de suscripción de Microsoft Azure. Las conexiones SMTP salientes que usan el puerto TCP 25 se bloquearon. (El puerto 25 se usa principalmente para la entrega de correo electrónico sin autenticación).

Este cambio de comportamiento solo se aplica a suscripciones e implementaciones nuevas desde el 15 de noviembre de 2017.

## <a name="recommended-method-of-sending-email"></a>Método recomendado para el envío de correo electrónico
Se recomienda usar servicios de retransmisión SMTP autenticados (que habitualmente se conectan a través del puerto TCP 587 o 443, pero que también admiten otros puertos) para enviar correo electrónico desde máquinas virtuales de Azure o desde Azure App Service. Estos servicios se usan para mantener la reputación de IP o dominio para minimizar la posibilidad de que proveedores de correo electrónico de terceros rechacen el mensaje. Dichos servicios de retransmisión SMTP incluyen, entre otros, [SendGrid](https://sendgrid.com/partners/azure/). También es posible que tenga un servicio de retransmisión SMTP seguro que se ejecute de manera local que puede usar.

El uso de estos servicios de entrega de correo electrónico no está restringido en Azure, independientemente del tipo de suscripción.

## <a name="enterprise-agreement"></a>Contrato Enterprise
Para los usuarios de Azure de Contrato Enterprise, no hay ningún cambio en la capacidad técnica para enviar correo electrónico sin usar una retransmisión autenticada. Tanto los usuarios nuevos como existentes de Contrato Enterprise puede probar la entrega de correo electrónico saliente desde máquinas virtuales de Azure directamente a proveedores de correo electrónico externos sin ninguna restricción desde la plataforma de Azure. Si bien no se garantiza que los proveedores de correo electrónico acepten correo electrónico entrante de un usuario cualquiera, los intentos de entrega no se bloquearán por medio de la plataforma de Azure desde máquinas virtuales dentro de las suscripciones a Contrato Enterprise. Tendrá que trabajar directamente con proveedores de correo electrónico para corregir cualquier problema de entrega de mensaje o filtrado de correo no deseado que implique a proveedores específicos.

## <a name="pay-as-you-go"></a>Pay-As-You-Go
Si se suscribió antes del 15 de noviembre de 2017 en las ofertas de suscripción de Pago por uso o de Microsoft Partner Network, no habrá ningún cambio en la capacidad técnica de probar la entrega de correo electrónico saliente. Seguirá teniendo la posibilidad de probar la entrega de correo electrónico saliente desde máquinas virtuales de Azure dentro de estas suscripciones directamente a proveedores de correo electrónico externos sin ninguna restricción desde la plataforma de Azure. Como ya se indicó, no se garantiza que los proveedores de correo electrónico acepten el correo electrónico entrante de un usuario cualquiera y los usuarios deberán trabajar directamente con proveedores de correo electrónico para corregir cualquier problema de entrega de mensaje o filtrado de correo no deseado que implique a proveedores específicos.

Para las suscripciones a Pago por uso o Microsoft Partner Network creadas después del 15 de noviembre de 2017, habrá restricciones técnicas para bloquear el correo electrónico que se envía directamente desde máquinas virtuales dentro de estas suscripciones. Si quiere poder enviar correo electrónico desde máquinas virtuales de Azure directamente a proveedores de correo electrónico externos (sin usar retransmisión SMTP autenticada), puede hacer una solicitud para quitar la restricción. Las solicitudes se revisarán y aprobarán a discreción de Microsoft y solo se concederán una vez que se hayan realizado comprobaciones adicionales contra fraudes. Para realizar una solicitud, abra un caso de soporte técnico mediante el siguiente tipo de incidencia: **Técnico** > **Red virtual** > **Conectividad** > **No se puede enviar correo (SMTP/Puerto 25)** . Asegúrese de agregar detalles sobre por qué la implementación tiene que enviar correo directamente a los proveedores de correo en lugar de usar una retransmisión autenticada.

Después de que una suscripción de pago por uso o de Microsoft Partner Network esté exenta y las VM hayan sido "detenidas" e "iniciadas" desde Azure Portal, todas las VM dentro de esa suscripción quedarán exentas en el futuro. La exención solo es aplicable a la suscripción solicitada y solo se aplica al tráfico de la máquina virtual que se enruta directamente a Internet. No se admite el tráfico del puerto 25 de enrutamiento mediante servicios de PaaS de Azure, como [Azure Firewall](https://azure.microsoft.com/services/azure-firewall/).

> [!NOTE]
> Microsoft se reserva el derecho a revocar este exención si se determina que se produjo una infracción en los términos del servicio.

## <a name="msdn-azure-pass-azure-in-open-education-bizspark-and-free-trial"></a>MSDN, Pase para Azure, Azure bajo licencia Open, Education, BizSpark y evaluación gratuita
Si creó una suscripción de MSDN, Pase para Azure, Azure bajo licencia Open, Education, BizSpark, Patrocinio de Azure, Azure para alumnos, Evaluación gratuita o cualquier suscripción a Visual Studio después del 15 de noviembre de 2017, tendrá restricciones técnicas que bloquearán el correo electrónico que se envía desde máquinas virtuales dentro de estas suscripciones directamente a los proveedores de correo electrónico. Las restricciones existen para evitar abusos. No se concederá ninguna solicitud para quitar esta restricción.

Si usa estos tipos de suscripción, se recomienda que use los servicios de retransmisión SMTP tal como se indicó anteriormente en este artículo, o que cambie el tipo de suscripción.

## <a name="cloud-service-provider-csp"></a>Proveedor de servicios en la nube (CSP)

Si usa los recursos de Azure a través de CSP, puede solicitar que CSP cree en su nombre una solicitud de exención de desbloqueo con Microsoft si no se puede usar una retransmisión de SMTP segura.

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si aún necesita ayuda, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver su problema rápidamente; para ello, use el siguiente tipo de problema: Tipo de problema **Administración de suscripciones**: **Solicitud para habilitar el flujo de correo electrónico del puerto 25**.
