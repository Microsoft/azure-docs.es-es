---
title: Supervisión y seguimiento del uso de los servicios gratuitos de Azure
description: Aprenda a comprobar el uso de los servicios gratuitos en Azure Portal. No se aplica ningún cargo por los servicios incluidos en una cuenta gratuita, a menos que supere los límites de servicio.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: 03efb7e4d0aa06e4c33a94f15621d58330e52e50
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461874"
---
# <a name="check-usage-of-free-services-included-with-your-azure-free-account"></a>Comprobación del uso de los servicios gratuitos incluidos con la cuenta gratuita de Azure

No se le cobrarán los servicios que se incluyen de forma gratuita en la cuenta gratuita de Azure, a menos que supere los límites de los mismos. Para no superarlos, puede usar Azure Portal para realizar el seguimiento del uso de los servicios gratuitos.

## <a name="check-usage-in-the-azure-portal"></a>Comprobación del uso en Azure Portal

1.  Inicie sesión en [Azure Portal](https://portal.azure.com).

2.  Busque **Suscripciones**.

    ![Captura de pantalla que muestra la búsqueda de suscripciones en el portal](./media/check-free-service-usage/billing-search-subscriptions.png)

3.  Seleccione la suscripción que se creó cuando se registró para obtener la cuenta gratuita de Azure.

4.  Desplácese hacia abajo para encontrar la tabla que muestra el uso de los servicios gratuitos.

    ![Captura de pantalla que muestra el uso de los servicios gratuitos](./media/check-free-service-usage/subscription-usage-free-services.png)

    La tabla tiene las columnas siguientes.

* **Medidor:** identifica la unidad de medida del recurso que se consume.
* **Uso y límite:** uso y límite del mes actual para el medidor.
* **Estado:** estado de uso del servicio. En función del uso, puede tener uno de los siguientes estados:
  * **No está en uso:** no ha usado el medidor o el uso de este no llega al sistema de facturación.
  * **Lo ha superado el \<Date>:** Ha superado el límite del medidor el \<Date>.
  * **Superación improbable:** es poco probable que supere el límite del medidor.
  * **Lo superará el \<Date>:** Es probable que supere el límite del medidor el \<Date>.

> [!IMPORTANT]
>
> Servicios gratuitos que solo están disponibles para la suscripción que se creó cuando se registró para obtener la cuenta gratuita de Azure. Si no puede ver la tabla de servicios gratuitos en la página de información general de la suscripción, no estarán disponibles para la suscripción.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes
- [Actualización de una cuenta gratuita de Azure](upgrade-azure-subscription.md)
